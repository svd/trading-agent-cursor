---
name: trading-risk-management
description: Use when user requests position size calculation, risk management analysis, or stop-loss/take-profit sizing. Also invoke when user asks "сколько акций купить", "какой размер позиции", "могу ли я открыть сделку", "хватит ли депозита", "дневной лимит", or any question about how many shares/contracts/coins to trade. Applies Gerchik methodology: 1% max risk per trade, 3-5% daily limit, stop after 3 consecutive losses.
---

# Trading Risk Management

## Overview

Comprehensive risk management methodology according to Gerchik methodology. Covers position sizing, maximum risk limits, stop-loss placement rules, take-profit targets, and emotional control for capital preservation.

## When to Use

Apply this skill when:
- User requests position size calculation
- User asks about risk management rules
- User mentions "размер позиции", "position size", "риск", "risk", "stop-loss", "take-profit"
- Calculating position size during trade scenario formation
- Determining if trade risk is acceptable
- Planning daily risk limits
- Assessing emotional state and trading discipline

## The Process

### Step 1: Calculate Position Size

#### Formula

```
Position Size = (Risk in $) / (Entry Price - Stop-Loss)
```

Where:
- **Risk in $** = Deposit × 1% (or less)
- **Entry Price** = Price planned for entry
- **Stop-Loss** = Stop-loss price

#### Example

- Deposit: $10,000
- Risk: 1% = $100
- Entry: $185.60
- SL: $184.90
- Size: $100 / ($185.60 - $184.90) = $100 / $0.70 = 142 shares

#### For Futures

```
Position Size = (Risk in $) / ((Entry Price - Stop-Loss) × Contract Size)
```

Example (ES futures):
- Deposit: $10,000
- Risk: 1% = $100
- Entry: 5,200
- SL: 5,180
- Contract size: $50 per point
- Size: $100 / ((5,200 - 5,180) × $50) = $100 / $1,000 = 0.1 contract (round to 1)

### Step 2: Determine Maximum Risk per Trade

**Main Rule: Maximum 1% of deposit per trade**

This means if stop-loss triggers, loss will be no more than 1% of total deposit.

### Step 3: Calculate Maximum Daily Risk

**Recommendation**: No more than 3-5% of deposit per day

This means:
- If risk per trade is 1%, can open maximum 3-5 trades
- If one trade loses, can open 2-4 more
- If 2 trades lose, can open 1-3 more

**Important**: After 3 consecutive losing trades — **stop trading for the day**.

### Step 4: Position Size Based on Confidence

- **High confidence** (all prerequisites met): 1% risk
- **Medium confidence**: 0.5-0.7% risk
- **Low confidence**: Don't trade or 0.3% risk

### Step 5: Account for Volatility (ATR) in Position Sizing

**Important:**
- When calculating position size, consider instrument's ATR
- If ATR is high, stop will be wider, position size must be smaller (at fixed 1% risk)
- If ATR is low, stop can be narrower, position size can be larger (at fixed 1% risk)

**Formula remains the same:**
```
Position Size = (Risk in $) / (Entry Price - Stop-Loss)
```

But stop-loss is calculated considering ATR and technical stop priority.

## Stop-Loss Rules

### Using ATR for Stop-Loss Placement

**ATR Calculation:**
- **Period: 5 days** (5 bars on daily timeframe D1)
- Exclude paranormal bars (>1.8-2x standard) and small bars (<0.3-0.5x standard)

**Distance from Entry:**
- **Minimum:** 0.5-1.0 ATR from entry
- **Standard:** 1.0-1.5 ATR from entry
- **Wide (for volatile):** 1.5-2.0 ATR from entry

**Volatility Adaptation:**
- High volatility (current ATR > average): wider stops (1.5-2.0 ATR)
- Low volatility (current ATR < average): narrower stops (0.5-1.0 ATR)

**Market Phase Consideration:**
- In impulse: wider stops (1.5-2.0 ATR)
- In accumulation: standard stops (1.0-1.5 ATR)
- In flat: avoid trading or use narrow stops

### Technical Stop Priority

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

### Main Stop-Loss Rules

1. **Always use SL** — no exceptions
2. **SL beyond level** — minimum 0.2% from price, minimum 0.5 ATR from entry
3. **Technical stop priority** — use technical stop if it doesn't exceed calculated (ATR)
4. **Don't move SL against position** — only in profit direction (trailing stop)
5. **Don't increase risk** — if SL too wide, reduce position size
6. **Consider ATR** — adapt stops to instrument volatility

## Take-Profit Rules

1. **Minimum 2:1 R:R** — otherwise don't trade
2. **Partial close** — close part on TP1, rest on TP2
3. **Trailing stop** — after reaching TP1, move SL to breakeven

## Capital Management

### Capital Distribution

1. **Trading capital**: 80-90% of deposit
2. **Reserve**: 10-20% of deposit (for force-majeure)

### Daily Risk Limits

- **Maximum 3-5 trades per day**
- **Maximum 3-5% daily risk** (3-5 trades × 1% each)
- **Stop trading after 3 consecutive losses**

## Emotional Control

### Rules to Prevent Emotional Decisions

1. **Plan before session** — all scenarios written in advance
2. **Follow plan** — don't change SL/TP during trade
3. **Trade limit** — maximum 5 trades per day
4. **Break after loss** — minimum 30 minutes after stop-loss

### Emotional Trading Signs

- Increasing position size after loss (revenge trading)
- Deleting stop-loss
- Trading without plan
- Trading more than planned number of trades

**If you notice these signs — stop trading for the day.**

## Important Rules

1. **1% risk per trade** — maximum
2. **3-5% risk per day** — maximum
3. **Always use SL** — no exceptions
4. **Minimum 2:1 R:R** — otherwise don't trade
5. **Follow plan** — don't change during trade
6. **Stop after 3 consecutive losses**
7. **Technical stop priority** — use if doesn't exceed calculated
8. **Don't revenge trade** — accept losses and move on

## Position Size Calculation Examples

### Stock Example

```
Deposit: $10,000
Risk: 1% = $100
Entry: $185.60
SL: $184.90
Risk per share: $185.60 - $184.90 = $0.70
Position size: $100 / $0.70 = 142 shares
```

### Futures Example

```
Deposit: $10,000
Risk: 1% = $100
Entry: 5,200
SL: 5,180
Contract size: $50 per point
Risk per contract: (5,200 - 5,180) × $50 = 20 × $50 = $1,000
Position size: $100 / $1,000 = 0.1 contract (round to 1)
```

### Crypto Example

```
Deposit: $10,000
Risk: 1% = $100
Entry: $45,000
SL: $44,500
Risk per BTC: $45,000 - $44,500 = $500
Position size: $100 / $500 = 0.2 BTC
```

## Output Format for Trade Scenario

```markdown
## Размер позиции

**Депозит**: $10,000
**Риск на сделку**: 1% = $100

**Расчет**:
- Вход: $185.60
- Стоп-лосс: $184.90
- Риск на акцию: $0.70
- Размер позиции: $100 / $0.70 = 142 акции

**Результат**: Покупка 142 акций с риском $100 (1% депозита)
```

## References

- @trading-entry-exit - Entry/exit point rules and stop-loss calculation details
- @trading-trends - ATR calculation methodology and market phase considerations
- @trading-levels - Level-based stop-loss placement (technical stops)
- @trading-homework - Full homework workflow (includes position sizing in trade scenarios)
