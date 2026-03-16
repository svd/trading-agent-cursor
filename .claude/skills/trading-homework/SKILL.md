---
name: trading-homework
description: Use when user requests homework completion, trading session preparation, or instrument analysis. Phrases: "домашка", "homework", "подготовься к сессии", "разбери AAPL", "проведи скрининг", "подготовить список". Works for any number of instruments — single or full list.
---

# Trading Homework — Orchestrator

## Overview

Mandatory session preparation. **Always dispatches sub-agents via Task tool** — never fetches market data or runs analysis inline. Your role: orchestrate, aggregate, summarize.

**Without homework completion, trading is not allowed.**

## What You Do vs Sub-Agents

| Orchestrator (you) | Sub-agents |
|-------------------|------------|
| Create output directory | Fetch OHLCV + quotes |
| Read templates, fill placeholders | Screen instruments (volume, ATR, trend) |
| Launch Task sub-agents in parallel | Analyze levels, trends, prerequisites |
| Read `RECOMMENDATION:` lines from outputs | Write homework files and charts |
| Build and update Summary table | — |

## Sub-Agent Templates

Templates live in `prompts/subagents/` (part of the agent repo; workspace gets this via symlink at `{WORKSPACE}/prompts`):

| Template | Use for |
|----------|---------|
| `screening-stocks.md` | Bulk stock screening |
| `screening-crypto.md` | Bulk crypto screening per exchange |
| `instrument-analysis.md` | Single instrument full analysis |

**Usage:** Read template from `{WORKSPACE}/prompts/subagents/<template>` → substitute `{{PLACEHOLDERS}}` → pass filled string as `prompt` to Task tool (`subagent_type="general-purpose"`)

## File-Based Data Flow (tmp/)

OHLCV data and chart input JSON are written to `{WORKSPACE}/tmp/` — **not passed through the orchestrator**.

```
Sub-agent instrument flow:
  1. Fetch OHLCV → save {WORKSPACE}/tmp/SYMBOL_DATE_ohlcv_1d.json
  2. Analyze levels/trends from json (not via LLM context)
  3. generate_chart_from_file(input_path=json_path) → homework/DATE/charts/SYMBOL_1D.png
  4. Write homework/DATE/SYMBOL.md with RECOMMENDATION: line
  5. Orchestrator reads only RECOMMENDATION: — OHLCV never passes through orchestrator
```

**tmp/ TTL rules:**
- File naming: `SYMBOL_DATE_ohlcv_1d.json` (e.g. `AAPL_2026-03-16_ohlcv_1d.json`)
- If DATE == today → reuse (skip re-fetch)
- Clean manually or at session end; gitignored

### Placeholders

**screening-stocks.md**
- `{{TICKERS}}` — comma-separated: `AAPL, META, NFLX`
- `{{DATE}}` — `YYYY-MM-DD`
- `{{OUTPUT_FILE}}` — absolute path: `{workspace}/homework/{DATE}/screening/screening-stocks.md`
- `{{WORKSPACE}}` — workspace root absolute path

**screening-crypto.md**
- `{{PAIRS}}` — e.g. `BTCUSDT, ETHUSDT`
- `{{EXCHANGE}}` — `binance` or `bybit`
- `{{DATE}}`, `{{OUTPUT_FILE}}` (e.g. `homework/{DATE}/screening/screening-crypto-binance.md`), `{{WORKSPACE}}`

**instrument-analysis.md**
- `{{SYMBOL}}` — `AAPL` or `BTCUSDT`
- `{{INSTRUMENT_TYPE}}` — `stock` or `crypto`
- `{{EXCHANGE}}` — exchange name for crypto; empty string for stocks
- `{{DATE}}`, `{{WORKSPACE}}`
- `{{SCREENING_DATA}}` — brief snippet from screening (price, ATR, trend, phase); empty string if no screening

Use **absolute paths** for `{{OUTPUT_FILE}}` and `{{WORKSPACE}}`.

## Workflow

### 1. Create Directory

```
homework/YYYY-MM-DD/
homework/YYYY-MM-DD/screening/
homework/YYYY-MM-DD/data/
homework/YYYY-MM-DD/charts/
```

### 2. Screening — Determine Source

**First, check which mode applies:**

| User input | Action |
|------------|--------|
| "по META, AAPL, LLY" (explicit tickers) | Skip screening → Step 3 directly |
| "по вотчлисту" / "из watchlist" | Use `watchlist/{TODAY}/longs.md` + `shorts.md` tickers |
| "по акциям" / "скрининг акций" (no tickers) | **Prompt with AskUserQuestion** |
| "сделай домашку" (no asset class) | Ask: "Какие инструменты?" |

**AskUserQuestion when user says "по акциям" without specifics:**

Use the AskUserQuestion tool to present these options:

```python
AskUserQuestion(
    questions=[{
        "question": "Какие акции проанализировать?",
        "header": "Тикеры",
        "multiSelect": true,
        "options": [
            {"label": "Mega-cap (FAAMG)", "description": "AAPL, MSFT, GOOGL, AMZN, META"},
            {"label": "Tech / Internet", "description": "Batch 1 — 25 tech tickers"},
            {"label": "Semiconductors", "description": "Batch 2 — NVDA, AMD, INTC, ..."},
            {"label": "Tech Software", "description": "Batch 3 — CRM, ADBE, ORCL, ..."},
            {"label": "Energy / Materials", "description": "Batch 4 — XOM, CVX, ..."},
            {"label": "Financials", "description": "Batches 5-6 — Banks, payments"},
            {"label": "Industrials", "description": "Batch 7 — BA, CAT, GE, ..."},
            {"label": "Consumer", "description": "Batches 8-9 — Retail, services"},
            {"label": "Healthcare", "description": "Batches 10-11 — Pharma, devices"},
            {"label": "REITs / Utilities", "description": "Batch 12 — AMT, NEE, ..."},
            {"label": "All 300 stocks", "description": "Full S&P500 + NASDAQ100 (slow)"},
            {"label": "From watchlist", "description": "Use today's longs/shorts"},
            {"label": "Custom", "description": "Enter your own tickers"}
        ]
    }]
)
```

**After user selection:**
- If user selected sector batch(es): read tickers from `prompts/stocks-universe.md` for those batches
- If user selected "Custom": prompt for ticker list via follow-up question
- If user selected "From watchlist": read today's watchlist
- Then proceed with screening using selected tickers

### 3. Run Screening

- **Stocks**: fill `screening-stocks.md` → Task → `homework/{DATE}/screening/screening-stocks.md`
- For batch selections from universe: create separate screening tasks per batch (e.g., `batch-tech-internet.md`)
- **Crypto per exchange**: fill `screening-crypto.md` → Task → `homework/{DATE}/screening/screening-crypto-{exchange}.md`
- Run up to 4 screening Tasks in parallel.
- After completion: check `STATUS: success` or `STATUS: error`. Present candidate table, optionally confirm with user before proceeding.

### 4. Instrument Analysis (batched, 2–4 parallel)

For each candidate (or user-specified instruments directly):

1. Read `prompts/subagents/instrument-analysis.md`
2. Substitute all placeholders; include `{{SCREENING_DATA}}` snippet if available from step 3
3. Launch Task
4. After Task completes: read output file for `RECOMMENDATION:` line

### 5. Update Summary

After each analysis Task, append/update row in `homework/{DATE}/Summary-{DATE}.md`:

```markdown
# Summary - YYYY-MM-DD

| Тикер | Рекомендация | Комментарий |
|-------|--------------|-------------|
| [AAPL](./AAPL.md) | ✅ Добавить на лонг-лист | Поджатие к 185.50, сильный тренд |
| [BTCUSDT (Binance)](./BTCUSDT-Binance.md) | ✅ Добавить на лонг-лист | Отскок от уровня излома 90,363 |
| [LLY](./LLY.md) | ❌ Пропустить | Боковик, нет чёткого уровня |
```

### 6. Present Results

Show Summary table and list of homework files + charts.

## File Naming

| Type | Path |
|------|------|
| Screening stocks | `homework/{DATE}/screening/screening-stocks.md` |
| Screening crypto | `homework/{DATE}/screening/screening-crypto-{exchange}.md` |
| Homework stock | `homework/{DATE}/SYMBOL.md` |
| Homework crypto | `homework/{DATE}/SYMBOL-EXCHANGE.md` |
| Chart JSON | `homework/{DATE}/data/SYMBOL_1D_chart_input.json` or `SYMBOL-EXCHANGE_1D_chart_input.json` |
| Chart PNG | `homework/{DATE}/charts/SYMBOL_1D.png` or `SYMBOL-EXCHANGE_1D.png` |
| Summary | `homework/{DATE}/Summary-{DATE}.md` |

Exchange name with capital first letter: Binance, Bybit, Coinbase.

## Error Handling

- `STATUS: error` in screening → report, optionally retry or skip group
- Analysis sub-agent fails / no output file → note in summary, continue with others
- Never block the whole run on one failure

## What You MUST NOT Do

- Call `get_stock_history`, `get_crypto_history`, or any MCP market data tool directly
- Run level analysis, trend analysis, or scenario formation in your own context
- Keep OHLCV tables in context — read only `RECOMMENDATION:` lines and summary files
