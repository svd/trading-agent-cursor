---
name: orchestrator-homework
description: Use when user requests full homework or screening for many instruments. Dispatches screening and instrument-analysis sub-agents via Task tool using prompt templates from prompts/subagents/. Prevents context overflow; do not run screening or full analysis inline for 10+ instruments.
---

# Orchestrator Homework — Sub-Agent Dispatch

Use this skill when the user asks for **homework**, **screening**, or **домашка** and the list of instruments is large (e.g. 5+ stocks and/or crypto). Your role is **orchestration only**: you dispatch sub-agents and aggregate results. You do **not** fetch OHLCV or run screening/level logic in your own context.

## When to Use

- User: "Сделай домашку на [list of tickers]", "Проведи скрининг акций/крипты", "Подготовься к сессии" with many instruments.
- Prefer this over the full inline homework skill when there are **5+ instruments** or when you want to avoid context overflow.

## Prompt Templates Location

All sub-agent prompts are **file-based** in the workspace:

- **Stocks screening:** `prompts/subagents/screening-stocks.md`
- **Crypto screening:** `prompts/subagents/screening-crypto.md`
- **Single instrument analysis:** `prompts/subagents/instrument-analysis.md`

Read the template with `read_file`, substitute the placeholders (see below), then pass the **resulting string** as the `prompt` argument to the **Task** tool (`subagent_type="generalPurpose"`). Do not load the full template into your context for execution — only for substitution and dispatch.

## Placeholders to Substitute

### screening-stocks.md

- `{{TICKERS}}` — **single ticker**, e.g. `AAPL` (one ticker per subagent — see Workflow below)
- `{{DATE}}` — YYYY-MM-DD (today or user date)
- `{{OUTPUT_FILE}}` — per-ticker staging file: `{workspace}/homework/{DATE}/screening-stocks-{TICKER}.md` (e.g. `screening-stocks-AAPL.md`)
- `{{WORKSPACE}}` — workspace root absolute path

### screening-crypto.md

- `{{PAIRS}}` — **single pair**, e.g. `BTCUSDT` (one pair per subagent — see Workflow below)
- `{{EXCHANGE}}` — **always required**; extract from the user's message (e.g. `binance`, `bybit`). If the user did not specify an exchange, default to `binance`. Never leave this placeholder unsubstituted.
- `{{DATE}}`, `{{WORKSPACE}}` — same idea
- `{{OUTPUT_FILE}}` — per-pair staging file: `{workspace}/homework/{DATE}/screening-crypto-{exchange}-{PAIR}.md` (e.g. `screening-crypto-binance-BTCUSDT.md`)

### instrument-analysis.md

- `{{SYMBOL}}` — e.g. `AAPL` or `BTCUSDT`
- `{{INSTRUMENT_TYPE}}` — `stock` or `crypto`
- `{{EXCHANGE}}` — exchange name for crypto; empty string for stocks
- `{{DATE}}`, `{{WORKSPACE}}`
- `{{SCREENING_DATA}}` — short summary from screening (price, ATR, trend, phase) so the sub-agent can skip re-fetching

## Workflow

1. **Create directory:** `homework/YYYY-MM-DD/` (use today's date or user-specified).
2. **Screening (parallel):**
   - **Extract exchange first (crypto only):** Before dispatching any crypto subagent, identify the exchange from the user's message. If not stated, use `binance`. This value is `{{EXCHANGE}}` for every crypto subagent call — never omit it.
   - If user provided stock tickers: launch **one Task per ticker** — for each ticker read `prompts/subagents/screening-stocks.md`, substitute `{{TICKERS}}` = single ticker (e.g. `AAPL`), `{{DATE}}`, `{{OUTPUT_FILE}}` = `homework/YYYY-MM-DD/screening-stocks-{TICKER}.md`, `{{WORKSPACE}}`. Run all ticker Tasks **in parallel** simultaneously (no cap). After all complete: read every per-ticker staging file (`screening-stocks-*.md`), aggregate rows into a combined `homework/YYYY-MM-DD/screening-stocks.md`. Append `STATUS: success` (or `STATUS: error` if any ticker failed).
   - If user provided crypto pairs: launch **one Task per pair** — for each pair read `prompts/subagents/screening-crypto.md`, substitute `{{PAIRS}}` = single pair (e.g. `BTCUSDT`), `{{EXCHANGE}}`, `{{DATE}}`, `{{OUTPUT_FILE}}` = `homework/YYYY-MM-DD/screening-crypto-{exchange}-{PAIR}.md`, `{{WORKSPACE}}`. Run all pair Tasks **in parallel** simultaneously (no cap on number of parallel screening Tasks).
   - After all crypto pair Tasks complete: read every per-pair staging file (`screening-crypto-{exchange}-*.md`), aggregate the rows into a combined `homework/YYYY-MM-DD/screening-crypto-{exchange}.md` with the same table/candidates structure. Append `STATUS: success` (or `STATUS: error` if any pair failed).
3. **Collect screening results:** Read the screening markdown file(s). Check for `STATUS: success` or `STATUS: error` at the end. Present the "Кандидаты для домашки" (and table) to the user. Optionally ask to confirm or auto-proceed.
4. **Instrument analysis (batched):**
   - For each candidate from screening (or user-confirmed list): read `prompts/subagents/instrument-analysis.md`, substitute `{{SYMBOL}}`, `{{INSTRUMENT_TYPE}}`, `{{EXCHANGE}}`, `{{DATE}}`, `{{WORKSPACE}}`, `{{SCREENING_DATA}}` (short snippet from screening for that symbol). Launch **Task** with the filled prompt.
   - Run 2–4 analysis Tasks in parallel (batches). Wait for each batch before starting the next.
5. **Summary file:** After each analysis sub-agent completes, read the homework file or the sub-agent output for a line starting with `RECOMMENDATION:`. Parse: symbol, status (add-long / add-short / skip), comment. Append or update row in `homework/YYYY-MM-DD/Summary-YYYY-MM-DD.md` (table: Тикер (link to .md), Рекомендация, Комментарий).
6. **Present final summary:** Show the Summary table and list of homework files/charts.

## File Paths and Naming

- Screening (combined): `homework/YYYY-MM-DD/screening-stocks.md`, `homework/YYYY-MM-DD/screening-crypto-{exchange}.md` (written by orchestrator after aggregation).
- Per-ticker staging: `homework/YYYY-MM-DD/screening-stocks-{TICKER}.md` (one per ticker, written by subagent).
- Per-pair staging: `homework/YYYY-MM-DD/screening-crypto-{exchange}-{PAIR}.md` (one per pair, written by subagent).
- Homework: `homework/YYYY-MM-DD/SYMBOL.md` (stocks), `homework/YYYY-MM-DD/SYMBOL-EXCHANGE.md` (crypto).
- Chart: `homework/YYYY-MM-DD/SYMBOL_1D.png` or `homework/YYYY-MM-DD/SYMBOL-EXCHANGE_1D.png`.
- Summary: `homework/YYYY-MM-DD/Summary-YYYY-MM-DD.md`.

Use **absolute paths** for `{{OUTPUT_FILE}}` and `{{WORKSPACE}}` (e.g. workspace root from your environment).

## Error Handling

- If a screening file ends with `STATUS: error`, report and optionally retry or skip that group.
- If an analysis sub-agent fails or does not write the homework file, report the symbol and optionally retry once or run that instrument inline using the full homework skill (fallback).
- Do not block the whole run: continue with other instruments and note failures in the summary.

## Model Selection (if supported by Task)

- Screening: prefer faster model (data-heavy, criteria-based).
- Analysis: default model (levels and scenarios need deeper reasoning).

## What You Do Not Do

- Do **not** run screening logic (volume/ATR/trend/95% rule) yourself for the full list.
- Do **not** run level analysis or scenario formation yourself for each instrument.
- Do **not** keep large OHLCV or screening tables in your context; only read summary files and RECOMMENDATION lines.

## Fallback

If the user has only 1–3 instruments or explicitly asks for "разбор по одному без суб-агентов", use the usual homework skill and run the full workflow inline instead of dispatching sub-agents.
