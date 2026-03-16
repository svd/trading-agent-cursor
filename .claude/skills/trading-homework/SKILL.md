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
  3. generate_chart_from_file(input_path=json_path) → homework/DATE/SYMBOL_1D.png
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
- `{{OUTPUT_FILE}}` — absolute path: `{workspace}/homework/{DATE}/screening-stocks.md`
- `{{WORKSPACE}}` — workspace root absolute path

**screening-crypto.md**
- `{{PAIRS}}` — e.g. `BTCUSDT, ETHUSDT`
- `{{EXCHANGE}}` — `binance` or `bybit`
- `{{DATE}}`, `{{OUTPUT_FILE}}` (e.g. `homework/{DATE}/screening-crypto-binance.md`), `{{WORKSPACE}}`

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
```

### 2. Screening (skip if user provided specific instruments)

- **Stocks**: fill `screening-stocks.md` → Task → `homework/{DATE}/screening-stocks.md`
- **Crypto per exchange**: fill `screening-crypto.md` → Task → `homework/{DATE}/screening-crypto-{exchange}.md`
- Run up to 4 screening Tasks in parallel.
- After completion: check `STATUS: success` or `STATUS: error`. Present candidate table, optionally confirm with user before proceeding.

### 3. Instrument Analysis (batched, 2–4 parallel)

For each candidate (or user-specified instruments directly):

1. Read `prompts/subagents/instrument-analysis.md`
2. Substitute all placeholders; include `{{SCREENING_DATA}}` snippet if available from step 2
3. Launch Task
4. After Task completes: read output file for `RECOMMENDATION:` line

### 4. Update Summary

After each analysis Task, append/update row in `homework/{DATE}/Summary-{DATE}.md`:

```markdown
# Summary - YYYY-MM-DD

| Тикер | Рекомендация | Комментарий |
|-------|--------------|-------------|
| [AAPL](./AAPL.md) | ✅ Добавить на лонг-лист | Поджатие к 185.50, сильный тренд |
| [BTCUSDT (Binance)](./BTCUSDT-Binance.md) | ✅ Добавить на лонг-лист | Отскок от уровня излома 90,363 |
| [LLY](./LLY.md) | ❌ Пропустить | Боковик, нет чёткого уровня |
```

### 5. Present Results

Show Summary table and list of homework files + charts.

## File Naming

| Type | Path |
|------|------|
| Screening stocks | `homework/{DATE}/screening-stocks.md` |
| Screening crypto | `homework/{DATE}/screening-crypto-{exchange}.md` |
| Homework stock | `homework/{DATE}/SYMBOL.md` |
| Homework crypto | `homework/{DATE}/SYMBOL-EXCHANGE.md` |
| Chart | `homework/{DATE}/SYMBOL_1D.png` or `SYMBOL-EXCHANGE_1D.png` |
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
