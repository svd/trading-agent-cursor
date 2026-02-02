---
name: homework
description: Complete homework workflow for trading session preparation. Use when user requests homework completion, preparation for trading session, or analysis of trading instruments. Includes screening, level analysis, prerequisites check, and trade scenario formation.
---

# Homework - Trading Session Preparation Workflow

Complete workflow for mandatory preparation before trading session. Without homework completion, trading is not allowed.

**CRITICALLY IMPORTANT:** All data retrieval and tool operations are performed **ONLY** through MCP tools. Use `call_mcp_tool` to call all tools from `user-markethub-mcp` server. **DO NOT create any Python scripts** for data retrieval or operations.

## When to Use

Apply this skill when:
- User requests homework completion
- User asks to prepare for trading session
- User wants to analyze trading instruments
- User mentions "домашка", "homework", "подготовка к сессии", "prepare for trading"

## Workflow Overview

The homework consists of 7 main stages:

1. **Screening** - Find instruments suitable for trading today
2. **Level Analysis** - Determine support/resistance levels (see @levels rule)
3. **Prerequisites Check** - Verify all conditions are met (see @trends rule)
4. **Trade Scenario Formation** - Create trading plan (see @trends, @entry-exit rules)
5. **Decision Making** - Add to watchlist or skip
6. **Summary File Maintenance** - Update summary after each instrument
7. **Chart Generation** - Generate chart with levels (use @chart-generation skill)

## Stage 1: Screening Instruments

**Goal:** Find instruments suitable for trading today.

**IMPORTANT:** When analyzing trends, market phases, and trend strength, use detailed rules from @trends.

### Screening Criteria

**For Stocks (US Market):**
- **Volume**: Average Volume > 300,000 (standard) or > 500,000 (high liquidity). Avoid < 300,000 (or < 1M for penny stocks)
- **Price**: Filter for stocks >$0.50 (so 100 shares > $50). For main list consider stocks from **$10**
- **Market**: US only (exclude ADR), exclude ETF (except SPY, IWM)
- **Volatility**: Average True Range > $1
- **Selection criteria**: Stock near level, very strong daily candle
- **95% Rule**: Strictly apply - **95%** of reviewed charts must be rejected. Trade only 5% with "super clear picture"
- **Filtering**: Select only "hard" situations. Exclude "infected zones", deep false breakouts, "saw" patterns

**For Cryptocurrencies:**
- **Liquidity**: Filter for liquidity > $1,000,000 (or >100 BTC for pairs)
- **Volume thresholds**:
  - Risk $100: Volume **> 60-70 million**
  - Risk $200+: Volume **> 100 million**
  - Minimum: **> 50 million** (manipulative instruments)
- **Spot check**: For coins with 50M volume on futures, spot volume should be **15-20 million+**

**For Forex:**
- Start with 10-15 pairs. Avoid trading same pair every day; rotate to find active markets
- **Liquidity formula**: `Coefficient = (0.1 * ATR_Daily) / Spread`. Threshold must be **> 4** (ideally **> 5**)

**Common Criteria** (see @trends):
- **Global trend (W1/MN)**: Determine overall market direction on higher timeframes
- **Local trend (D1/H1)**: Clear trend on D1 or H4, or instrument at range boundary
- **Trend strength**: Assess trend strength (price action + volumes + ATR)
- **Market phase**: Determine current phase (accumulation/impulse/correction/retest/flat/exhaustion) - avoid instruments in flat in middle of range

### MCP Tools for Screening

**All tools called via MCP using `call_mcp_tool` from `user-markethub-mcp` server:**

- **`get_market_stats`**: Get metrics (Average Volume, current volume) - stocks only
  - `server`: `"user-markethub-mcp"`
  - `toolName`: `"get_market_stats"`
  - `arguments`: `{"symbol": "AAPL"}`

- **For stocks**: Use `get_stock_history` for trend analysis on different timeframes (W1, D1, H1)
  - `server`: `"user-markethub-mcp"`
  - `toolName`: `"get_stock_history"`
  - `arguments`: `{"symbol": "AAPL", "interval": "1d", "period": "6mo"}` or with `start_date` and `end_date`

- **For crypto**: Use `get_crypto_history` for trend analysis
  - `server`: `"user-markethub-mcp"`
  - `toolName`: `"get_crypto_history"`
  - `arguments`: `{"symbol": "BTCUSDT", "interval": "1d", "period": "6mo", "exchange": "binance"}` (exchange optional)

- **`get_real_time_quote`**: Check spread and current price (works for stocks and crypto)
  - `server`: `"user-markethub-mcp"`
  - `toolName`: `"get_real_time_quote"`
  - `arguments`: `{"symbol": "AAPL"}` or `{"symbol": "BTCUSDT"}`

**Result:** List of candidates (10-20 instruments for stocks, 26-30 for homework)

## Stage 2: Level Analysis

**IMPORTANT:** When determining levels, use detailed rules from @levels. Below is brief summary of key points.

### Get Historical Data

**IMPORTANT:** All data retrieval operations performed via MCP tools. Use `call_mcp_tool` to call tools from `user-markethub-mcp` server.

**IMPORTANT:** Use correct tool depending on instrument type:
- **For stocks**: use `get_stock_history` (causes error for crypto)
- **For crypto**: use `get_crypto_history` (with optional `exchange` parameter)

**Must use `start_date` and `end_date` parameters (format: "YYYY-MM-DD"):**
- **`start_date`**: date 6 months ago from current date (format: "YYYY-MM-DD") - for level analysis
- **`end_date`**: current date (format: "YYYY-MM-DD")
- **`interval`**: "1d" (daily timeframe) for D1 analysis
- For weekly (W1) and monthly (M1) levels use data aggregation or queries with corresponding intervals ("1wk", "1mo")
- Analyze data on timeframes D1 and higher (W1, M1)
- **IMPORTANT:** For chart generation use 5 months range (see Stage 7)

### Priority Level Types (see @levels for details)

**Strongest levels:**
- **Trend reversal levels** - form at points where previous trend changed direction (strongest)
- **False breakout levels** - levels around which several false breakouts occurred
- **Mirror levels** - resistance levels that became support, and vice versa

### Historical Levels

Based on data for last 6 months (obtained via MCP tools):
- **D1**: High/Low of last 5-10 days (especially last 2-3 days)
- **W1**: High/Low of last 4-8 weeks
- **M1**: High/Low of last 3-6 months (may remain relevant for years)

### Psychological Levels (Round Numbers)

- For stocks: 10, 50, 100, 150, 200, 250, 300...
- For futures: 5000, 5100, 5200...
- For currencies: 1.10, 1.20, 1.30...
- Especially significant when coinciding with historical levels

### Previous Day Levels

- **Yesterday High**: Previous day maximum
- **Yesterday Low**: Previous day minimum
- **Yesterday Close**: Previous day close

### Opening Levels

- **Week Open**: Week opening price
- **Month Open**: Month opening price

### Gap-Based Levels

- Boundaries of price gaps can serve as strong levels
- In 7-8 cases out of 10 such levels hold successfully

### Level Strength Assessment (see @levels for detailed criteria)

**IMPORTANT:** When counting level touches, consider 0.5% tolerance from level price. Touch is counted if price is within this tolerance from level, not only exact hit.

**Very strong level:**
- Trend reversal, mirror level, multiple false breakouts
- Tick-by-tick precision (to fifth decimal)
- Confirmation through breakout (even small)
- Level not "sawed" (not tested many times without breakout)
- Significant volume, higher timeframe (W1, M1)

**Strong level:**
- 3+ touches (with 0.5% tolerance), confirmation through breakout
- Low near-level volatility

**Medium level:**
- 2 touches (with 0.5% tolerance), recent level

**Weak level:**
- 1 touch (with 0.5% tolerance), no confirmation, "worn"

### Level Filtering

**After determining all levels:**
1. Assess strength of each level per @levels criteria
2. **Exclude weak levels**: If found 3+ levels with strength "Medium" or higher, exclude all weak levels from further analysis
3. If found more than 10 levels, keep only **5-10 strongest**
4. Prioritize in order:
   - Trend reversal levels
   - Mirror levels
   - False breakout levels
   - Consider timeframe (M1 > W1 > D1)
   - Consider touch count and confirmation
5. Sort remaining levels by price (from maximum to minimum)

### Output Format

All levels consolidated in one table, sorted by price top to bottom (from maximum to minimum):

```markdown
## Уровни

| Цена | Тип уровня | Тип | Сила | Касания | Подтверждение | Комментарий |
|------|------------|-----|------|---------|---------------|-------------|
| 189.50 | Сопротивление | Излом тренда + W1 High | Очень сильный | 4 (с обеих сторон) | Пробой + точность тик-в-тик | Зеркальный уровень |
| 187.00 | Сопротивление | Круглое число | Средний | 2 | Нет | Психологический уровень |
| 185.50 | Сопротивление | D1 High (5 дней) | Сильный | 3 | Касание + пробой | ... |
| 180.00 | Поддержка | Круглое число + D1 Low | Сильный | 3 | Касание + пробой | Психологический уровень, совпадение |
| 178.50 | Поддержка | D1 Low (10 дней) | Сильный | 4 | Множественные касания | Отскок 4 раза, не "пилился" |
```

## Stage 3: Prerequisites Check (see @trends)

**IMPORTANT:** When checking trend and market phase, use detailed rules from @trends.

### Correlation Check

**Goal:** Ensure instrument has "own life" or check correlation with indices for entry decision.

**Main indices for check:**
- **S&P 500** (SPY) - main index for US stocks
- **NASDAQ** (QQQ) - technology sector
- **Dow Jones** (DIA) - industrial index
- **VIX** - volatility index

**Correlation check algorithm:**

1. **Determine main index** for instrument:
   - For US stocks: S&P 500 (SPY)
   - For tech stocks: NASDAQ (QQQ)
   - For crypto: Bitcoin (BTC) for altcoins

2. **Get correlating instrument data** (via MCP tools):
   - For stocks and indices: use `get_stock_history` via `call_mcp_tool` (server `user-markethub-mcp`)
   - For crypto: use `get_crypto_history` via `call_mcp_tool` (server `user-markethub-mcp`)
   - Use `get_real_time_quote` via `call_mcp_tool` (server `user-markethub-mcp`) for current quote

3. **Assess correlation**:
   - **For stocks**: Prefer instruments that **do not correlate** with index and have "own life"
   - **Exceptions**: For some instruments (e.g., Apple) correlation with index is critical; for InPlay (news-driven) correlation is secondary
   - **Index synchronization check**: For stocks check synchronization of Dow Jones, Nasdaq and S&P 500. Divergence between indices = risk signal

4. **Use correlation as filter**:
   - If entry signal on main instrument, check correlating instrument
   - **Direct correlation**: If signal on "Sell" for instrument A, and instrument B (with direct link) hit strong support and doesn't want to fall → **reject signal**
   - **Inverse correlation**: If signal on "Sell" for instrument A, and instrument B (with inverse link, e.g., DXY for EURUSD) broke level and going down (which should push instrument A up) → **reject signal**
   - **General rule**: Do not open trade if logic of related asset movement prohibits your asset movement in needed direction

5. **Market state check**:
   - If market falling (S&P 500 negative), avoid longs in weak stocks
   - If market rising but stock not updating highs and starts "closing" → possible short from key support levels

### Prerequisites Checklist

- [ ] **Global trend (W1/MN)**: Determined and synchronized with local (see @trends)
- [ ] **Local trend (D1/H1)**: Instrument in trend OR at range boundary (see @trends)
- [ ] **Trend strength**: Assessed (price action + volumes + ATR) (see @trends)
- [ ] **Market phase**: Determined (not flat in middle of range) (see @trends)
- [ ] **Level**: Clear level for work (not more than 0.5% from current price)
- [ ] **Volatility**: ATR(5) sufficient for movement to TP (minimum 2x from risk) (see @trends)
- [ ] **Volume**: Volume above average (minimum 1.2x from 20-day average, contrast volume 2-3x preferred) (see @trends)
- [ ] **Correlation**: Checked correlation with main index (S&P 500, NASDAQ, Dow Jones). Instrument has "own life" or correlation doesn't contradict trade direction
- [ ] **News**: No important news in next 2 hours
- [ ] **Session time**: Suitable time for trading (not before close without plan)

**If all prerequisites met** → can add to watchlist
**If at least one not met** → skip with explanation

## Stage 4: Trade Scenario Formation (see @trends)

For each instrument that passed check, create trade scenario:

**IMPORTANT:** When forming trade scenario, must consider information from @trends (global/local trend, trend strength, market phase, ATR(5) for stops, calculated stop = 20-30% from ATR).

**IMPORTANT:** When forming trade scenario, use level trading principles from @levels, especially proximity principle (trade as close to level as possible).

**CRITICALLY IMPORTANT:** NEVER use weak levels for trade scenario formation. For entry point use only levels with strength "Strong" or "Very strong". For stop-losses and take-profits can use levels with strength "Medium" or higher.

### Required Elements:

1. **Direction**: Long or Short (based on global and local trend, see @trends)
2. **Entry trigger**: Clear condition (e.g., "Breakout 185.50 with retest and volume confirmation") (see @trends)
3. **Entry point**: Specific price or condition
   - **Proximity principle:** Entry must be as close to level as possible (see @levels)
   - **Weak level ban:** Level for entry must be at least "Strong" by strength. Weak and medium levels not used for entry
   - Avoid entry on secondary movement (after breakout with return)
4. **Stop-loss** (see @trends): 
   - Not more than 1% from deposit per trade
   - **Technical stop priority:** Prefer technical stop (behind level), if it doesn't exceed calculated (20-30% from ATR(5)) (see @trends)
   - Calculate technical stop: behind support/resistance level (slightly behind level)
   - **Weak level ban:** Technical stop must be placed behind level at least "Medium" by strength. Weak levels not used for stop placement
   - Calculate calculated stop: 20-30% from ATR(5) (e.g., if ATR = $1.00, then calculated stop = $0.20 - $0.30) (see @trends)
   - Minimum 0.2% from instrument price
   - Minimum 0.5 ATR from entry (avoid market noise) (see @trends)
   - For round numbers: stop can be placed slightly below/above (e.g., 9.94 below 10.00)
5. **Take-profit**:
   - Minimum 2:1 to risk (R:R ≥ 2:1)
   - Preferably multiple levels (TP1, TP2)
   - **Weak level ban:** Take-profits must be placed on levels at least "Medium" by strength. Weak levels not used for take-profits
6. **Position size**: Calculate based on risk

### Output Format:

```markdown
## Торговый сценарий (см. @trends)

**Направление**: Long (на основе глобального тренда: восходящий W1)

**Триггер**: Пробой 185.50 с ретестом и подтверждением объёма (контрастный объем 2-3x среднего)

**Точка входа**: 185.52 (максимально близко к уровню 185.50 - принцип близости, см. @levels)

**Стоп-лосс**: 
  - Технический стоп: 184.80 (за уровнем 184.90)
  - Расчетный стоп (ATR): 185.40 (20-30% от ATR(5) = $0.70, используем 30% = $0.21)
  - **Выбранный стоп**: 184.80 (технический, риск: 0.39% = $0.72)

**Тейк-профит 1**: 187.00 (R:R 2:1)

**Тейк-профит 2**: 189.50 (R:R 5.5:1)
**Размер позиции**: X акций (риск: $Y = 1% депозита)
```

## Stage 5: Decision Making

After analyzing all elements:

**Add to watchlist**, if:
- All prerequisites met
- Clear trade scenario exists
- Risk management observed

**Skip**, if:
- At least one prerequisite not met
- No clear level for work
- Risk too high
- Insufficient data for analysis

### Recommendation Format:

```markdown
## Рекомендация

✅ **Добавить на лонг-лист** / ❌ **Пропустить**

**Причина**: 
- Все предпосылки выполнены
- Чёткий уровень 185.50
- R:R 2:1 минимум
```

## Stage 6: Summary File Maintenance

**IMPORTANT:** After analyzing each instrument (regardless of result - added to watchlist or skipped) must update Summary file.

**File location**: `homework/YYYY-MM-DD/Summary-YYYY-MM-DD.md`

### Summary File Rules:

1. **File creation**: On first instrument analysis for day, create file `Summary-YYYY-MM-DD.md` in folder `homework/YYYY-MM-DD/`
2. **Update entry**: If ticker already in table - update its entry (replace row)
3. **Add entry**: If ticker not in table - add new row
4. **Table format**: Use table with columns: Ticker (link), Recommendation, Comment

### Summary File Format:

```markdown
# Summary - YYYY-MM-DD

| Тикер | Рекомендация | Комментарий |
|-------|--------------|-------------|
| [MU](./MU.md) | ✅ Добавить на лонг-лист | Сильнейший тренд, обновление ATH. Вход на пробой 412.50 или откат к 390. |
| [STX](./STX.md) | ✅ Добавить на лонг-лист | Поджатие к 350.00. Отличный кандидат на пробой. |
| [AAPL](./AAPL.md) | ✅ Добавить на лонг-лист | Техничная коррекция лидера. Вход на возврате к 250. |
| [BTCUSDT (Binance)](./BTCUSDT-Binance.md) | ✅ Добавить на лонг-лист | Отскок от уровня излома. Вход на пробой 90,363. |
| [LLY](./LLY.md) | ❌ Пропустить | Боковик, "пила". Ждать выхода из диапазона 1015-1095. |
```

**Column rules:**
- **Ticker**: 
  - **For stocks**: Link to homework file in format `[SYMBOL](./SYMBOL.md)`. Example: `[AAPL](./AAPL.md)`
  - **For crypto**: Link in format `[SYMBOL (EXCHANGE)](./SYMBOL-EXCHANGE.md)`. Example: `[BTCUSDT (Binance)](./BTCUSDT-Binance.md)`
- **Recommendation**: 
  - `✅ Добавить на лонг-лист` - instruments that passed all checks
  - `⚠️ Добавить (С осторожностью)` - instruments with increased risk
  - `⚠️ Добавить (High Risk)` - high risk instruments
  - `❌ Пропустить` - instruments that didn't pass check
- **Comment**: Brief summary of key analysis points - trend, levels, entry, reason for decision. Should be informative but concise (1-2 sentences)

**Important**: 
- Summary file must be updated after each processed instrument
- If ticker analyzed again same day - update its entry in table
- Table should be sorted logically (e.g., added to watchlist first, then skipped, or alphabetically)

## Stage 7: Chart Generation

**CRITICALLY IMPORTANT:** Charts generated **ONLY** via MCP tool `generate_chart` directly. **DO NOT create any additional scripts** (Python, bash, etc.) for chart generation.

**Tool**: Use `call_mcp_tool` to call `generate_chart` from MCP server `user-markethub-mcp`

**IMPORTANT:** Use @chart-generation skill for detailed chart generation workflow.

### Chart Generation Process:

1. **Prepare data**: Get historical OHLCV data for chart generation:
   - **Attempt 1**: Range **5 months** (`start_date`: date 5 months ago)
   - **Attempt 2** (if attempt 1 failed): Range **4 months** (`start_date`: date 4 months ago)
   - **Attempt 3** (if attempt 2 failed): Range **3 months** (`start_date`: date 3 months ago)
   - Use same levels from analysis (regardless of data range for chart)

2. **Call MCP tool**: Use `call_mcp_tool` with parameters (see @chart-generation skill for details)

3. **Verify success**: After each `generate_chart` call, check file existence via `read_file` or `list_dir`
   - If file exists and not empty → **STOP**, success
   - If file not created or empty → try smaller range (if attempts not exhausted)

4. **Save chart**: 
   - MCP tool returns base64-encoded image
   - Decode base64 and save image to file:
     - For stocks: `homework/YYYY-MM-DD/SYMBOL.png`
     - For crypto: `homework/YYYY-MM-DD/SYMBOL-EXCHANGE.png`
   - Add link to chart in homework file

5. **If all 3 attempts failed**:
   - Stop and inform user about error
   - **DO NOT continue** infinite attempts

### Add Chart to Homework File

**For stocks:**
```markdown
## График

![График {SYMBOL}](./{SYMBOL}.png)
```

**For cryptocurrencies:**
```markdown
## График

![График {SYMBOL} ({EXCHANGE})](./{SYMBOL}-{EXCHANGE}.png)
```

## Homework File Structure

**IMPORTANT:** Use format specified in these rules, not examples from old homework files.

### File Naming

**For stocks and other instruments:**
- File saved in `homework/YYYY-MM-DD/SYMBOL.md`
- Example: `homework/2026-01-24/AAPL.md`

**For cryptocurrencies:**
- **IMPORTANT:** File must include exchange name in filename to distinguish analysis of same pair on different exchanges
- Format: `homework/YYYY-MM-DD/SYMBOL-EXCHANGE.md`
- Examples: 
  - `homework/2026-01-24/BTCUSDT-Binance.md`
  - `homework/2026-01-24/BTCUSDT-Bybit.md`
- Exchange name with capital first letter (Binance, Bybit, Coinbase, etc.)

### File Content Structure

**For stocks:**
```markdown
# {SYMBOL} - {DATE}
```

**For cryptocurrencies:**
```markdown
# {SYMBOL} ({EXCHANGE}) - {DATE}
```

**Full file structure:**

```markdown
# {SYMBOL} ({EXCHANGE}) - {DATE}

## Текущая ситуация
- Цена: {price}
- Изменение: {change}%
- Объём: {volume} (vs avg: {volume_vs_avg})
- ATR(5): {atr} (расчетный, без паранормальных баров, см. @trends)

## График

![График {SYMBOL}](./{SYMBOL}.png)

## Тренды и фазы рынка (см. @trends)
- **Глобальный тренд (W1/MN):** {global_trend} (восходящий/нисходящий/боковой)
- **Локальный тренд (D1/H1):** {local_trend} (восходящий/нисходящий/боковой)
- **Синхронизация:** {sync_status} (синхронизированы/противоречат)
- **Сила тренда:** {trend_strength} (сильный/средний/слабый)
  - Ценовое действие: {price_action}
  - Объемы: {volume_analysis}
  - ATR: {atr_analysis}
- **Фаза рынка:** {market_phase} (накопление/импульс/коррекция/ретест/флэт/истощение)

## Уровни

| Цена | Тип уровня | Тип | Сила | Касания | Подтверждение | Комментарий |
|------|------------|-----|------|---------|---------------|-------------|
| ... | Сопротивление/Поддержка | ... | ... | ... | ... | ... |

*Таблица отсортирована по цене сверху вниз (от максимальной к минимальной). Включены только 5-10 самых сильных уровней.*

## Предпосылки
[чеклист, включая проверку корреляции]

## Корреляция с индексами
- **Основной индекс**: {main_index} (S&P 500 / NASDAQ / Dow Jones)
- **Корреляция**: {correlation_status} (низкая / средняя / высокая)
- **Анализ**: {correlation_analysis}
  - Инструмент имеет "собственную жизнь": {yes/no}
  - Синхронность индексов: {synchronized/divergence}
  - Состояние рынка: {market_state}
- **Решение**: {correlation_decision} (соответствует направлению сделки / противоречит)

## Торговый сценарий (см. @trends)

**Направление**: Long/Short (на основе глобального тренда: {global_trend})

**Триггер**: ... (что должно произойти для входа)

**Вход**: ... (максимально близко к уровню - принцип близости)

**Стоп-лосс**: 
  - Технический стоп: ... (за уровнем {level_price})
  - Расчетный стоп (ATR): ... (20-30% от ATR(5) = {calculated_stop})
  - **Выбранный стоп**: ... (технический/расчетный, риск: ...%)

**Тейк-профит**: ... (R:R минимум 2:1)

## Рекомендация
[решение и причина]
```

## Important Rules

1. **Never trade without homework**
2. **Always write scenario before session start** (emotions should not influence)
3. **Stop-loss mandatory** for each trade
4. **Minimum 2:1 R:R** - otherwise don't trade
5. **Maximum 1% risk** per trade

## References

- @trends - Trend analysis methodology
- @levels - Level identification and assessment
- @entry-exit - Entry/exit point rules
- @risk-management - Risk management standards
- @chart-generation - Chart generation workflow
