---
name: trading-trends
description: Use when user requests trend analysis, market phase determination, or pattern identification. Applies Gerchik methodology for global/local trend analysis, trend strength assessment, market phases, ATR calculation for stops, and volume analysis. Critical for screening, homework, and trade scenario formation.
---

# Trading Trends - Trend Analysis and Market Phases

## Overview

Comprehensive methodology for analyzing trends, assessing trend strength, determining market phases, and identifying price patterns according to Gerchik methodology. Essential for making informed trading decisions and determining trade direction.

## When to Use

Apply this skill when:
- User requests trend analysis for a trading instrument
- User asks to determine global/local trend direction
- User mentions "тренд", "trend", "market phase", "фаза рынка"
- Completing screening or homework that requires trend analysis
- Assessing trade direction (long vs short)
- Determining if instrument is suitable for trading (not in flat)
- Calculating ATR for stop-loss placement
- Analyzing volume confirmation for movements

## The Process

### Step 1: Get Historical Data for Multiple Timeframes

**IMPORTANT:** Use correct MCP tool depending on instrument type:
- **For stocks**: use `get_stock_history` (causes error for crypto)
- **For crypto**: use `get_crypto_history` (with optional `exchange` parameter)

**Required timeframes for trend analysis:** Use **only** `start_date` and `end_date` (YYYY-MM-DD). Do not use period.
- **Weekly (W1)**: `interval="1wk"`, `start_date` 1 year (or 2 years) ago, `end_date` current - for global trend
- **Daily (D1)**: `interval="1d"`, `start_date` 6 months (or 1 year) ago, `end_date` current - for local trend
- **Hourly (H1)**: `interval="1h"`, `start_date` 1 month ago, `end_date` current - for entry timing

**For ATR calculation:**
- Use `interval="1d"` with **start_date** and **end_date** (e.g. 3 months of data) to calculate ATR(5). Do not use period.

### Step 2: Determine Global Trend (W1/MN)

**Definition:**
- Determined on higher timeframes: **Weekly (W1)** and **Monthly (MN)**
- Shows overall market direction for long period
- Used for determining long-term goals and historical extremes
- Strong level on W1 can override D1 signal

**Trend Identification:**
- **Uptrend (Восходящий):** Higher Highs and Higher Lows (HH, HL), fixing above levels
- **Downtrend (Нисходящий):** Lower Highs and Lower Lows (LH, LL), updating lows
- **Sideways (Боковой):** Price moves in horizontal range

**Output:** "Восходящий" / "Нисходящий" / "Боковой"

### Step 3: Determine Local Trend (D1/H1)

**Definition:**
- Determined on lower timeframes: **Daily (D1)** and **Hourly (H1)**
- Shows short-term price movement direction
- Used for determining entry and exit points
- M15, M5 used for local trends and entry points

**Trend Identification:** Same as global (HH/HL for uptrend, LH/LL for downtrend)

**Output:** "Восходящий" / "Нисходящий" / "Боковой"

### Step 4: Check Trend Synchronization

**Rules:**
- Signals on lower timeframes MUST NOT contradict daily trend
- If Daily says Long, DO NOT open Short on M5 even if M5 looks bearish
- If signal on lower TF (15m/30m) contradicts higher TF (Daily) — trade forbidden

**Check:**
- Are global and local trends synchronized?
- Do they contradict each other?

**Output:** "Синхронизированы" / "Противоречат"

### Step 5: Assess Trend Strength

**Price Action Criteria:**

**Strong Trend:**
- Rewriting historical highs almost every day (e.g., AAPL)
- Vertical movement without deep pullbacks
- Parabolic movement (+40% in a day) indicates strong "Big Player" absorbing all liquidity
- "Train gathering speed": slow, constant movement without stops considered very strong
- No correction after strong movement indicates strong market control

**Weak Trend:**
- Small slope angle (~5 degrees)
- Sideways movement
- Even if technically a trend, lack of momentum makes it unprofitable for trading

**Volume Influence:**

**Strong Trend with Volume:**
- Price up + volume up = Strong uptrend
- Price down + volume up = Strong downtrend
- **Contrast volume** (2-3x average) = Movement strength confirmation
- High volume at key levels = Strong large player interest

**Weakness Signs:**
- Price up + volume down = Trend weakening, possible reversal
- Price down + volume down = Weak movement, possible bounce
- Volume drop = End of accumulation/distribution
- **Divergence:** price makes new highs but volume declines = Warning signal

**Volume Confirmation Rules:**
- Trend continuation requires volume confirmation
- Movements without volume are suspicious
- High volume at level breakout = Strong interest
- Low volume at breakout = Weakness, possible false breakout

**ATR Influence:**

**ATR as Strength Indicator:**
- **Movement >80-85% of daily ATR** = Fuel exhausted, avoid chasing price
- **Movement 10-15% of ATR** = Significant potential remains
- **Constant ATR consumption** = Strong impulse, trend continues
- **ATR expansion during trend** = Movement strengthening
- **ATR compression** = Possible consolidation or reversal

**ATR in Trend vs Flat:**
- In trend (channel): ATR counted from level to level in movement direction
- In flat (range): ATR resets to zero

**Weekly ATR:**
- Usually passes in 3-9 days on daily chart
- Used for evaluating long-term potential

**Complex Assessment:**

For determining trend strength must consider:
1. **Price action:** rewriting extremes, slope angle, pullback depth
2. **Volumes:** contrast volume, movement confirmation, divergences
3. **ATR:** ATR consumption, expansion/compression, movement potential

**Strong Trend =** Price Action + High Volumes + Constant ATR Consumption

**Output:** "Сильный" / "Средний" / "Слабый"

**Additional Analysis:**
- Price action description
- Volume analysis
- ATR analysis

### Step 6: Determine Market Phase

**Market Phases:**

**1. Accumulation (Накопление)**
- Sideways movement (channel/range)
- Price accumulates energy for next movement
- Often precedes strong directional movement
- Volumes usually decrease during accumulation
- Small bars (low volatility)
- Duration: several days to weeks
- **Trading rule:** Wait for range breakout, don't trade in middle of range, look for entry points at range boundaries

**2. Impulse (Импульс)**
- Main movement after breakout or accumulation
- Strong movement consisting of large bars
- High volumes
- Fast ATR consumption
- Breakout with high volume
- Large bars (above average ATR)
- Minimal pullbacks
- Fast price movement
- **Trading rule:** Enter on breakout with volume confirmation, use wider stops (1.5-2.0 ATR), fast movement can be short

**3. Correction/Rollback (Коррекция)**
- Counter-trend movement
- Pullback against main trend
- Usually 30-50% of previous movement
- Smaller volumes than impulse
- Smaller bars
- **Trading rule:** No correction after strong movement = Strength sign; Deep correction (>50%) may indicate trend weakness; Can trade bounce from correction in trend direction

**4. Retest (Ретест)**
- Price return to broken level
- Test level "from other side"
- Breakout confirmation
- Price broke level, then returned to it
- Bounced from level (retest confirmed level)
- **Trading rule:** If level holds (not broken back) = Trend continues; Enter on bounce from level after retest; Near retest more reliable than far

**5. Flat/Range (Флэт/Рейндж)**
- Low activity period
- **75-80% of time market is in flat**
- Price moves in horizontal range
- Low volatility
- Price doesn't make new highs/lows
- Movement in channel
- Low volumes
- Small bars
- **Trading rule:** AVOID trading if no clear setup; DON'T trade in middle of range (high stop-hunt risk); Trade only at range boundaries; Wait for clear breakout or bounce

**6. Exhaustion (Истощение)**
- Inability to update highs (in uptrend)
- Inability to update lows (in downtrend)
- Movement can't be infinite
- Long movements without correction require accumulation for "refueling"
- Price can't break previous extreme
- Volumes decrease on movement attempts
- Large reversal bars appear
- Movement >100% ATR and return to open (ATR=0)
- **Trading rule:** Possible reversal signal; Possible opposite direction trade; Wait for reversal confirmation (key level breakout)

**Output:** "Накопление" / "Импульс" / "Коррекция" / "Ретест" / "Флэт" / "Истощение"

### Step 7: Calculate ATR for Stop-Loss

**ATR Calculation:**

**Period:**
- **Standard: 5 days** (5 bars on daily timeframe D1)
- For cryptocurrencies: 3 days (3 bars) due to extreme volatility
- Current forming candle NOT taken into account, only completed days

**Filtering Anomalies:**
- Exclude **Paranormal bars** (>1.8-2x standard)
- Exclude **Small bars** (<0.3-0.5x standard)

**Technical vs Calculated ATR:**
- **Technical ATR:** Includes paranormal bars (real movement)
- **Calculated ATR:** Excludes paranormal bars (statistical average)

**Output:** ATR value with note if calculated (excludes paranormal bars)

### Step 8: Identify Price Patterns

**Breakout Patterns:**

**False Breakout (Ложный пробой):**
- Price breaks level (often with 1-2 bars) and returns
- Strong signal for counter-trend entry
- Works on any timeframe (D1, H1, M5)
- Most effective false breakouts occur with 1-2 bars at extremes
- Usually drawn aggressively by large players to collect positions

**Breakout (Пробой):**
- Price crosses level
- Confirmed by consolidation or close beyond level
- Requires volume confirmation

**Squeeze (Поджатие):**
- Narrow consolidation (small bars) near level
- Indicates accumulation or energy preservation
- Look on H1 when approaching daily level
- In uptrend important to watch squeeze: if not broken, long entry not recommended

**Candlestick Patterns:**

**Paranormal Bar (Паранормальный бар):**
- Bar size 2-3x ATR
- Indicates high volume/energy
- Can lead to bounce or continuation
- **DO NOT enter immediately on these bars** — trading inside paranormal bar dangerous

**Pin Bar / Doji:**
- Candlestick patterns have 50/50 probability in isolation
- Trade only if located at very strong level
- If tail larger than half bar body, signals possible reversal

**Gaps:**

**Common Gap:** Occurs in range (channel), usually insignificant

**Breakaway Gap:** Occurs with high volume at trend beginning (breakout from accumulation)

**Runaway Gap:** Occurs in middle of trend

**Exhaustion Gap:** Occurs at trend end ("last bullets")

**Friday-Monday Gap:** "Acceleration gap" confirms current trend; closing such gap signals trend end

## Key Principles

### Top-Down Approach
- Always analyze from global to local
- Ignoring higher timeframe leads to losses on lower timeframe noise
- First determine global trend (W1/MN), then local (D1/H1)

### Global Trend Priority
- Prefer trades in direction of global trend
- Example: global EUR/USD rise with local deep correction — look for long opportunities

### Trend Synchronization
- Lower timeframe signals MUST NOT contradict daily trend
- If Daily says Long, DON'T open Short on M5

### In Uptrend Trade Only Longs
- Counter-trend shorts prohibited

### Don't Trade in Middle of Range
- Only at boundaries or on clear breakout

### 75-80% Rule
- Market in flat 75-80% of time
- Avoid trading if no clear setup

## Output Format

```markdown
## Тренды и фазы рынка

- **Глобальный тренд (W1/MN):** {global_trend} (восходящий/нисходящий/боковой)
- **Локальный тренд (D1/H1):** {local_trend} (восходящий/нисходящий/боковой)
- **Синхронизация:** {sync_status} (синхронизированы/противоречат)
- **Сила тренда:** {trend_strength} (сильный/средний/слабый)
  - Ценовое действие: {price_action}
  - Объемы: {volume_analysis}
  - ATR: {atr_analysis}
- **Фаза рынка:** {market_phase} (накопление/импульс/коррекция/ретест/флэт/истощение)
```

## ATR for Stop-Loss

**Distance from Entry:**
- **Minimum:** 0.5-1.0 ATR from entry
- **Standard:** 1.0-1.5 ATR from entry
- **Wide (for volatile):** 1.5-2.0 ATR from entry

**Placement Rules:**
- For long: stop below support level minus 0.5-1.0 ATR buffer
- For short: stop above resistance level plus 0.5-1.0 ATR buffer
- **NEVER place stop closer than 0.5 ATR from entry** (avoid market noise)

### Technical Stop Priority

See **@trading-entry-exit** for the full stop selection logic (technical vs calculated ATR-based stop, examples, and volatility adaptation rules).

## Important Rules

1. **Always distinguish global and local trend** — prefer trades in global direction
2. **Assess trend strength comprehensively** — price action + volumes + ATR
3. **Determine current market phase** — 75-80% of time market in flat
4. **Use ATR(5) for stop calculation** — prefer technical stop if it doesn't exceed calculated
5. **Consider volumes in strength assessment** — contrast volume confirms movement
6. **Don't trade in middle of range** — only at boundaries or on clear breakout
7. **Wait for pattern confirmation** — patterns only work at strong levels

## References

- @trading-levels - Level identification and assessment (trend reversal levels are strongest)
- @trading-entry-exit - Entry/exit point rules and stop-loss/take-profit calculation
- @trading-risk-management - Risk management standards
- @trading-homework - Full homework workflow (includes trend analysis stage)
- @trading-screening - Instrument screening workflow (includes trend filtering)
