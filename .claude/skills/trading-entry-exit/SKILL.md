---
name: trading-entry-exit
description: Use when user requests entry/exit point analysis, stop-loss/take-profit calculation, or trade scenario formation. Applies Gerchik methodology for entry triggers, stop placement, and exit rules. Critical for homework trade scenarios and real-time trading decisions.
---

# Trading Entry and Exit Points

## Overview

Comprehensive methodology for determining entry and exit points, calculating stop-losses and take-profits according to Gerchik methodology. Covers entry triggers, stop-loss placement with ATR-based calculations, position sizing, and exit strategies.

## When to Use

Apply this skill when:
- User requests entry/exit point analysis
- User asks to calculate stop-loss or take-profit
- User mentions "вход", "выход", "entry", "exit", "stop-loss", "take-profit"
- Forming trade scenarios during homework
- Calculating position size based on risk
- Determining if entry trigger is met
- Planning exit strategy (planned or early exit)

## The Process

### Step 1: Pre-Entry Analysis

#### 1.1 Analyze Global and Local Trend

**Must check:**
- **Global trend (W1/MN):** determine overall market direction
- **Local trend (D1/H1):** determine short-term direction
- **Synchronization:** local trend must not contradict global
- **Priority:** prefer trades in global trend direction

**Rule:**
- If global trend is uptrend, look for long opportunities
- If local trend contradicts global, be cautious or skip trade

#### 1.2 Assess Trend Strength

**Check:**
- **Price action:** rewriting extremes, slope angle, pullback depth
- **Volumes:** contrast volume (2-3x average) confirms strength
- **ATR:** ATR consumption, expansion/compression
- **Complex assessment:** strong trend = price action + high volumes + ATR consumption

#### 1.3 Determine Market Phase

**Check current phase:**
- Accumulation
- Impulse
- Correction/Rollback
- Retest
- Flat/Range — **75-80% of time market in flat**
- Exhaustion

**Rule:**
- Avoid trading in middle of range (flat)
- Trade only at range boundaries or on clear breakout

### Step 2: Determine Entry Triggers

#### 2.1 Level Breakout

**Conditions:**
- Price broke support/resistance level
- **Volume above average (confirmation)** — contrast volume (2-3x average) preferred
- Candle close beyond level
- **Check global trend:** breakout should be in global trend direction

**Entry:** After breakout, on level retest (or immediately if strong breakout with high volume)

#### 2.2 Level Bounce

**Conditions:**
- Price approached support/resistance level
- Reversal candle formed (hammer, doji, engulfing)
- **Volume confirms reversal** — contrast volume preferred
- **Check global trend:** bounce should be in global trend direction

**Entry:** On reversal candle close or next candle

#### 2.3 Breakout with Retest

**Condition:** Price broke level, then returned to test it

**Entry:** On bounce from level (retest confirmed level)

### Step 2.4: ATR Calculation Reference

For accurate stop-loss and position sizing, use the ATR calculator implementation.

See: @trading-trends → Step 6.5 for full implementation

**Key Points:**
- The model reads the Python code and applies the logic (no execution needed)
- Report both technical_atr and calculated_atr when analyzing instruments
- Use calculated_atr for filtering anomalous bars in statistics
- Use technical_atr for real-time stop placement when volatility is normal

### Step 3: Calculate Stop-Loss

#### 3.1 Using ATR for Stop-Loss Calculation

**ATR Calculation:**
- **Period: 5 days** (5 bars on daily timeframe D1)
- Use the implementation from @trading-trends Step 6.5
- Exclude paranormal bars (>1.8-2x standard deviation) and small bars (<0.3-0.5x standard deviation)
- Report both technical_atr and calculated_atr values

**Example Applying the Calculator:**
```python
# Given OHLCV data from MCP tool get_stock_history or get_crypto_history
ohlcv_data = [
    {'high': 186.50, 'low': 184.00, 'close': 185.20, 'volume': 45000000},
    {'high': 187.00, 'low': 184.50, 'close': 186.80, 'volume': 52000000},
    {'high': 188.20, 'low': 185.00, 'close': 187.50, 'volume': 48000000},
    {'high': 189.00, 'low': 186.50, 'close': 188.80, 'volume': 61000000},  # Paranormal bar
    {'high': 189.50, 'low': 187.00, 'close': 188.00, 'volume': 43000000},
    {'high': 188.80, 'low': 186.20, 'close': 186.50, 'volume': 39000000},
]

result = calculate_atr_gerchik(ohlcv_data, period=5)
# Result: technical_atr = 2.35, calculated_atr = 1.92 (paranormal bar excluded)
```

**Distance from Entry based on ATR:**
- **Minimum:** 0.5-1.0 ATR from entry
- **Standard:** 1.0-1.5 ATR from entry
- **Wide (for volatile):** 1.5-2.0 ATR from entry

**Volatility Adaptation:**
- High volatility (current ATR > average): wider stops (1.5-2.0 ATR)
- Low volatility (current ATR < average): narrower stops (0.5-1.0 ATR)

#### 3.2 Technical Stop Priority

**IMPORTANT RULE:**
- **Prefer technical stop (beyond level), if it doesn't exceed calculated stop (based on ATR)**

**Selection Logic:**

1. **Calculate technical stop:**
   - For long: slightly below support level
   - For short: slightly above resistance level

2. **Calculate calculated stop:**
   - Based on ATR: **20-30% of ATR** (e.g., if ATR = $1.00, calculated stop = $0.20 - $0.30)
   - Consider instrument volatility

3. **Compare and select:**
   - If **technical stop ≤ calculated stop** → use **technical stop**
   - If **technical stop > calculated stop** → reconsider entry point or use calculated stop (but don't exceed maximum 1% risk)

**Example:**
- Long entry: $185.60
- Support level: $184.90
- Technical stop: $184.80 (beyond level)
- ATR(5): $0.70
- Calculated stop: 20-30% of ATR = $0.14 - $0.21, so stop at $185.60 - $0.20 = $185.40 (using 30% = $0.21, rounded to $0.20)
- **Selection:** Technical stop $184.80 < calculated $185.40 → use technical $184.80

#### 3.3 Stop-Loss Placement Rules

1. **Beyond level:** SL must be beyond support/resistance level
2. **Minimum 0.2%:** SL must not be less than 0.2% from entry price
3. **Minimum 0.5 ATR:** SL must not be closer than 0.5 ATR from entry (avoid market noise)
4. **Maximum 1% risk:** Trade risk must not exceed 1% of deposit
5. **Consider volatility:** In volatile instruments SL can be wider (1.5-2.0 ATR)
6. **Technical stop priority:** Use technical stop if it doesn't exceed calculated

### Step 4: Calculate Position Size

**Formula:**
```
Position Size = (Risk in $) / (Entry Price - Stop-Loss)
```

**Example:**
- Deposit: $10,000
- Risk: 1% = $100
- Entry: $185.60
- SL: $184.90
- Size: $100 / ($185.60 - $184.90) = $100 / $0.70 = 142 shares

### Step 5: Calculate Take-Profit

#### 5.1 Minimum R:R Ratio

**Mandatory:** Minimum 2:1 (risk:reward)

**Example:**
- Entry: $185.60
- SL: $184.90 (risk $0.70)
- TP1: $187.00 (profit $1.40, R:R 2:1)
- TP2: $189.50 (profit $3.90, R:R 5.5:1)

#### 5.2 Multiple Take-Profits

Recommended to use multiple TPs:
- **TP1:** Conservative (2:1 or 3:1) — close part of position
- **TP2:** Moderate (4:1 or 5:1) — close another part
- **TP3:** Aggressive (7:1 or higher) — leave small part

#### 5.3 Take-Profit Placement

TP placement:
- At next support/resistance level
- At round numbers
- Based on ATR (e.g., 2x ATR from entry)

**IMPORTANT:** Take-profits must be placed on levels at least "Medium" by strength. Weak levels not used for take-profits.

### Step 6: Exit Strategy

#### 6.1 Planned Exit

1. **TP reached:** Close position on take-profit
2. **SL reached:** Close position on stop-loss
3. **Partial close:** Close part of position on TP1, rest on TP2

#### 6.2 Early Exit

Acceptable to exit early if:
- Conditions changed (important news)
- Level broken in opposite direction
- Reversal signal (reversal candle)

**Important:** Early exit must be justified, not emotional decision.

### Step 7: Volume Analysis for Entry Confirmation

**Volume Confirmation Rules:**

1. **Level breakout:**
   - High volume (contrast, 2-3x average) = Strong confirmation
   - Low volume = Weakness, possible false breakout

2. **Level bounce:**
   - Volume confirms reversal = Strong signal
   - No volume = Weak signal

3. **Trend continuation:**
   - Price up + volume up = Strong uptrend
   - Price down + volume up = Strong downtrend
   - Volume drop = Possible movement end

4. **Volume divergence:**
   - Price makes new highs but volume declines = Warning signal
   - Consider exit possibility or don't enter

## Key Principles

### Entry Must Be Close to Level

**Proximity Principle:**
- ALWAYS enter as close to level as possible
- Entry far from level (e.g., 6 points) = almost stop-loss size
- Entry far from level worsens risk/reward ratio

**Avoid Secondary Movement:**
- DO NOT enter after instrument already broke level and returns for retest
- Common mistake leading to large stop-losses
- Recommendation: skip such trades and wait for correct entry point

### Technical Stop Priority

**CRITICAL:** Prefer technical stop (beyond level) if it doesn't exceed calculated stop (ATR-based)

### Minimum R:R Ratio

**Mandatory:** Minimum 2:1 R:R, otherwise don't trade

### Volume Confirmation

**Required:** Movements without volume are suspicious

## Important Rules

1. **Check global and local trend** — prefer trades in global trend direction
2. **Assess trend strength** — price action + volumes + ATR
3. **Determine market phase** — avoid trading in middle of range
4. **ALWAYS use SL** — no exceptions
5. **Technical stop priority** — use technical stop if it doesn't exceed calculated (ATR)
6. **Minimum 2:1 R:R** — otherwise don't trade
7. **Volume confirmation** — movements without volume suspicious
8. **Calculate position in advance** — before entry
9. **Follow plan** — don't change SL/TP during trade (except justified cases)

## Output Format for Trade Scenario

```markdown
## Торговый сценарий

**Направление**: Long/Short (на основе глобального тренда: {global_trend})

**Триггер**: ... (что должно произойти для входа)

**Вход**: ... (максимально близко к уровню - принцип близости)

**Стоп-лосс**:
  - Технический стоп: ... (за уровнем {level_price})
  - Расчетный стоп (ATR): ... (20-30% от ATR(5) = {calculated_stop})
  - **Выбранный стоп**: ... (технический/расчетный, риск: ...%)

**Тейк-профит 1**: ... (R:R 2:1)

**Тейк-профит 2**: ... (R:R 5:1)

**Размер позиции**: ... (риск: $Y = 1% депозита)
```

## References

- @trading-trends - Trend analysis methodology (global/local trends, trend strength, market phases, ATR for stops)
- @trading-levels - Level identification and assessment (proximity principle, level strength for entry)
- @trading-risk-management - Risk management standards (position sizing, maximum risk)
- @trading-homework - Full homework workflow (includes trade scenario formation stage)
