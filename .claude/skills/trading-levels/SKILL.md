---
name: trading-levels
description: Use when user requests level analysis, support/resistance identification, or working with price levels. Also invoke automatically during homework, trade scenario formation, and entry/exit planning — any time levels need to be found, assessed for strength, or used for entry/stop/TP placement. Applies Gerchik methodology: trend reversal > false breakout > mirror levels, 0.5% touch tolerance, 5-10 strongest levels per instrument.
---

# Trading Levels - Support and Resistance Analysis

## Overview

Comprehensive methodology for identifying, assessing, and trading support and resistance levels according to Gerchik methodology. Levels represent price zones where large players leave their "footprints" — where struggle occurs and volatility explodes after the battle.

## When to Use

Apply this skill when:
- User requests level analysis for a trading instrument
- User asks to identify support/resistance levels
- User mentions "уровни", "levels", "support/resistance"
- Forming trade scenarios that require entry points, stop-losses, or take-profits
- Completing homework that requires level analysis
- Assessing strength of price levels for trading decisions

## The Process

### Step 1: Get Historical Data

**IMPORTANT:** Use correct MCP tool depending on instrument type:
- **For stocks**: use `get_stock_history` (causes error for crypto)
- **For crypto**: use `get_crypto_history` (with optional `exchange` parameter)

**Must use `start_date` and `end_date` parameters (format: "YYYY-MM-DD"):**
- **`start_date`**: date 6 months ago from current date (format: "YYYY-MM-DD") - for level analysis
- **`end_date`**: current date (format: "YYYY-MM-DD")
- **`interval`**: "1d" (daily timeframe) for D1 analysis
- For weekly (W1) and monthly (M1) levels use data aggregation or queries with corresponding intervals ("1wk", "1mo")
- Analyze data on timeframes D1 and higher (W1, M1)

**CRITICAL RULE - Exclude Current Period:**

When determining historical levels **NEVER use** High/Low of the current incomplete period:
- **NEVER use** High/Low of current day for D1 levels
- **NEVER use** High/Low of current week for W1 levels
- **NEVER use** High/Low of current month for M1 levels

**Reason:** Current periods are incomplete and their High/Low can change, making them unreliable support/resistance levels.

### Step 2: Identify Priority Levels (Strongest First)

**Priority 1: Trend Reversal Levels**
- Formed at points where previous trend changed direction
- Strongest levels marking fundamental shift in balance of power
- Especially important for breakout formations

**Priority 2: False Breakout Levels**
- Levels around which multiple false breakouts occurred
- "Shaking out" clears market of weak players and strengthens level
- False breakouts usually drawn aggressively by large players to collect positions

**Priority 3: Mirror Levels**
- Resistance levels that became support (and vice versa)
- High informational value, especially if original level was trend reversal
- Role change indicates dominant force shift in market
- Often form after "first stop" or "first return" of price

### Step 3: Identify Historical Levels

**D1 (Daily):**
- High/Low of last 5-10 days (excluding current day)
- Especially important: levels of last 2-3 days (completed)
- Levels price bounced from multiple times
- Previous day levels (Yesterday High/Low/Close)

**W1 (Weekly):**
- High/Low of last 4-8 weeks (excluding current week)
- Stronger levels than daily
- Used for swing trading

**M1 (Monthly):**
- High/Low of last 3-6 months (excluding current month)
- Strongest historical levels
- Used for position trading
- Can remain relevant for years

### Step 4: Identify Additional Levels

**Psychological Levels (Round Numbers):**
- Stocks: 10, 50, 100, 150, 200, 250, 300...
- Futures: 5000, 5100, 5200...
- Currencies: 1.10, 1.20, 1.30...
- Especially significant when coinciding with historical levels

**Previous Day Levels:**
- Yesterday High: Previous day maximum
- Yesterday Low: Previous day minimum
- Yesterday Close: Previous day close

**Opening Levels:**
- Week Open: Week opening price
- Month Open: Month opening price

**Gap-Based Levels:**
- Boundaries of price gaps can serve as strong levels
- In 7-8 out of 10 cases such levels hold successfully
- Allow safer stop-loss placement

**Accumulation/Distribution Zones:**
- Zones where price consolidated for long time
- Wide accumulation zones (~25% of price) give strong exit
- Important to determine accumulation and distribution phases

### Step 5: Multi-Timeframe Analysis

- Check coincidence of levels on different timeframes
- Levels visible simultaneously on multiple timeframes are more significant
- M1 > W1 > D1 in terms of strength

### Step 6: Assess Level Strength

**Touch Tolerance:**

**IMPORTANT:** When counting level touches, price doesn't need to hit exact level. Touch is counted if price is within **0.5% tolerance** of level price.

**Formula:**
```
Tolerance = Level_Price × 0.005 (0.5%)
Touch = True if |Price - Level| ≤ Tolerance
```

**Examples:**
- Level 100.00: touch if price in range [99.50, 100.50]
- Level 50.00: touch if price in range [49.75, 50.25]
- Level 200.00: touch if price in range [199.00, 201.00]

**Very Strong Level:**
- Trend reversal, mirror level, multiple false breakouts
- Tick-by-tick precision (to fifth decimal place)
- Confirmation through breakout (even small)
- Level not "sawed" (not tested many times without breakout)
- Significant volume, higher timeframe (W1, M1)
- Historical significance (works for years)
- Coincides with round number
- "Clean zone" before level (without tails and touches)

**Strong Level:**
- 3+ touches (with 0.5% tolerance), confirmation through breakout
- Low near-level volatility (small bars)
- Level held for long time (several days/weeks)
- Coincides with round number
- Higher timeframe

**Medium Level:**
- 2 touches (with 0.5% tolerance), recent level
- Current timeframe
- Partial coincidence with other types

**Weak Level:**
- 1 touch (with 0.5% tolerance)
- Recent (yesterday's)
- No coincidence with other types
- Level "worn" (multiple tests without breakout)
- No confirmation

**Signs of Level Weakness:**
- Level "saws" (multiple tests without breakout)
- Deep pullbacks (>30% of daily ATR)
- High near-level volatility (big bars)
- Price easily breaks level when closing near it
- **No reaction at strong level** = breakout signal

### Step 7: Filter Levels

After determining all levels:
1. Assess strength of each level
2. **Exclude weak levels**: If found 3+ levels with strength "Medium" or higher, exclude all weak levels
3. If found more than 10 levels, keep only **5-10 strongest**
4. Prioritize in order:
   - Trend reversal levels
   - Mirror levels
   - False breakout levels
   - Consider timeframe (M1 > W1 > D1)
   - Consider touch count and confirmation
5. Sort remaining levels by price (from maximum to minimum)

### Step 8: Output Format

All levels consolidated in one table, sorted by price top to bottom:

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

**Level Type must indicate:**
- Priority type (trend reversal, false breakouts, mirror)
- Timeframe (D1, W1, M1)
- Additional characteristics (round number, gap, accumulation)

**Level Strength:** Very strong / Strong / Medium / Weak

**Touches:**
- Number of level touches (counted with 0.5% tolerance)
- Specify: from both sides or from one side
- **Note:** Touch counted if price within 0.5% of level, not only exact hit

**Confirmation:**
- Multiple touches
- Breakout (even small)
- Precision of working
- False breakouts
- Volume
- Multi-timeframe coincidence

**Comment must include:**
- Level features (mirror, reversal, false breakouts)
- Precision of working
- Signs of large player activity
- Trading recommendations (close entry, wait for reaction)

## Key Principles

### 1. Principle of Proximity

**ALWAYS trade as close to level as possible**
- Entry far from level (e.g., 6 points) = almost size of stop-loss
- Entry far from level worsens risk/reward ratio
- Optimal entry: as close to level as possible

### 2. Avoid Secondary Movement

- DO NOT enter after instrument already broke level and returns for retest
- Common mistake leading to large stop-losses
- Recommendation: skip such trades and wait for correct entry point in direction of main impulse

### 3. Trade Only From Very Strong Levels

- Especially important for far retests
- If instrument closes near level and level easily breaks = important information about level weakness

### 4. Second Touch Strategy

- First touch can be false
- Second touch confirms level significance
- Entry after second touch more reliable

### 5. Key Signal: No Reaction

**Most important signal:** When instrument approaches strong key level but **doesn't bounce** from it = highest probability breakout signal

## Retest Types

**Near Retest:**
- Always happens at recently broken trend reversal level
- Entry conditions:
  - Close approach to level
  - Daily candle close near high (for long) or low (for short)
  - Close in corresponding trading zone

**Far Retest:**
- Happens at older, historical levels
- Overall market picture important, not touch fact itself
- Key condition — "clean" zone before level (without tails and touches)
- Large players shouldn't be in level zone if trading breakout

## Working With Breakouts

**Breakout Confirmation:**
- Breakout confirmed if price closes above (resistance) or below (support) level
- For short positions especially important to be at short extreme points

**False Breakout (LPL):**
- Powerful reversal signal
- Usually drawn very aggressively by large players to collect positions
- Paranormal, volatile movement with return back beyond level
- Often precedes strong movement in opposite direction
- Multiple tests (three or more) of one point more likely indicate false breakout

**Primary Breakout:**
- Level never broken in nearest left space
- Multiple tests without breakout indicate instrument readiness for movement

**Secondary Breakout:**
- Repeat attempt to break level
- Success conditions:
  1. Level shouldn't "saw"
  2. No deep pullbacks (<30% of daily ATR)
  3. Very low near-level volatility (small bars)
  4. Picture doesn't break (movement after breakout continues same direction)

## Stop-Loss Placement

- Stop-losses placed slightly beyond entry level
- Dangerous when price returns back through level after counter-trend movement (liquidity collection)
- For round numbers: stop can be placed slightly below/above (e.g., 9.94 below 10.00)
- Gap boundaries can serve as level for safer stop-loss

## Trading in Range

- Entry can be based on price behavior analysis inside channel
- Multiple (5+ candles) failed attempts to break range boundary without aggressive sales indicate likely bounce
- From lower boundaries of accumulation zones can open long positions

## Large Player Role

**Indicators of Large Player Activity:**
- If price can't fall despite negative news and "gets bought every time" = large player support signal
- Significant trading volume in level zone
- Analysis of who and why supports instrument at level
- Stop-loss clusters slightly below/above level (breakout can be initiated to collect them)

## Important Rules

1. **NEVER use current period High/Low** — exclude current day for D1, current week for W1, current month for M1
2. **Don't trade in middle of range** without clear levels
3. **Always wait for level confirmation** (bounce or breakout)
4. **Consider timeframe:** Higher TF levels more important
5. **Check volume:** Breakout/bounce should be accompanied by volume
6. **Trade as close to level as possible** (proximity principle)
7. **No reaction at strong level** = important breakout signal

## References

- @trading-trends - Trend analysis methodology (global/local trends, trend strength, market phases)
- @trading-entry-exit - Entry/exit point rules and stop-loss/take-profit calculation
- @trading-risk-management - Risk management standards
- @trading-homework - Full homework workflow (includes level analysis stage)
