---
name: screening
description: Use when user requests screening of trading instruments (stocks, crypto, forex) for trading opportunities. Applies Gerchik methodology criteria for volume, volatility, trend analysis, and correlation checks.
---

# Screening Trading Instruments

## Overview

Systematic screening process to find trading instruments suitable for trading today using Gerchik methodology. Applies strict criteria for volume, volatility, trend analysis, and correlation to filter instruments, following the "95% rule" - only trade 5% with "super clear picture".

## When to Use

Apply this skill when:
- User requests screening of instruments (stocks, crypto, forex)
- User asks to find trading opportunities
- User wants to filter watchlist by criteria
- User mentions "скрининг", "screening", "find instruments", "trading candidates"

## Screening Criteria

### Stocks (US Market)

1. **Volume**: Average Volume > 300,000 (standard) or > 500,000 (high liquidity). Avoid instruments with volume < 300,000 (or < 1M for penny stocks)
2. **Price**: Filter for stocks >$0.50 (so 100 shares > $50). Separate lists for stocks >$10 and <$10
3. **Market**: US only (exclude ADR), exclude ETF (except SPY, IWM)
4. **Volatility**: Average True Range > $1
5. **95% Rule**: Strictly apply - **95%** of reviewed charts must be rejected. Trade only 5% with "super clear picture"
6. **Filtering**: Select only "hard" situations. Exclude "infected zones", deep false breakouts, "saw" patterns

### Cryptocurrencies

1. **Liquidity**: Filter for liquidity > $1,000,000 (or >100 BTC for pairs)
2. **Volume thresholds**:
   - Risk $100: Volume **> 60-70 million**
   - Risk $200+: Volume **> 100 million**
   - Minimum: **> 50 million** (manipulative instruments)
3. **Spot check**: For coins with 50M volume on futures, spot volume should be **15-20 million+**

### Forex

1. Start with 10-15 pairs. Avoid trading the same pair every day; rotate to find active markets
2. **Liquidity formula**: `Coefficient = (0.1 * ATR_Daily) / Spread`. Threshold must be **> 4** (ideally **> 5**)

### Common Criteria (see @trends rule)

1. **Global trend (W1/MN)**: Determine overall market direction on higher timeframes
2. **Local trend (D1/H1)**: Clear trend on D1 or H4, or instrument at range boundary
3. **Trend strength**: Assess trend strength (price action + volumes + ATR)
4. **Market phase**: Determine current phase (accumulation/impulse/correction/retest/flat/exhaustion) - avoid instruments in flat in middle of range

## Screening Process

### Step 1: Get Instrument List

Obtain list of instruments to screen (or use predefined list from user).

### Step 2: For Each Instrument

#### 2.1 Get Market Data

Use MCP tools to collect required data:

**For stocks:**
- `user-markethub-mcp/get_real_time_quote` - current price, spread, change
- `user-markethub-mcp/get_stock_history` - historical data for trend analysis:
  - W1 (weekly): `interval="1wk"`, `period="1y"` or `period="2y"`
  - D1 (daily): `interval="1d"`, `period="6mo"` or `period="1y"`
  - H1 (hourly): `interval="1h"`, `period="1mo"`
- `user-markethub-mcp/get_market_stats` - ATR, Average Volume, current volume

**For crypto:**
- `user-markethub-mcp/get_real_time_quote` - current price, spread
- `user-markethub-mcp/get_crypto_history` - historical data:
  - W1: `interval="1wk"`, `period="1y"`
  - D1: `interval="1d"`, `period="6mo"`
  - H1: `interval="1h"`, `period="1mo"`
- Note: `get_market_stats` is for stocks only, calculate ATR from historical data for crypto

**For forex:**
- `user-markethub-mcp/get_real_time_quote` - current price, spread
- `user-markethub-mcp/get_stock_history` or `get_crypto_history` (depending on broker) - D1 data for ATR calculation
- Calculate liquidity coefficient: `(0.1 * ATR_Daily) / Spread`

#### 2.2 Check Basic Criteria

**For stocks:**
- ✅ Average Volume > 300k (standard) or > 500k (high liquidity)
- ✅ ATR > $1
- ✅ Price >$0.50 (or >$10 for main list)
- ✅ US market only (exclude ADR, ETF except SPY, IWM)

**For crypto:**
- ✅ Liquidity > $1M, volume according to thresholds (50-100M depending on risk)

**For forex:**
- ✅ Liquidity coefficient = (0.1 * ATR_Daily) / Spread > 4 (ideally > 5)

If basic criteria fail → **REJECT** immediately.

#### 2.3 Analyze Trends

Use historical data from Step 2.1 and apply @trends rule:

1. **Global trend (W1/MN)**: Analyze weekly/monthly charts
   - Determine direction: Up / Down / Sideways
2. **Local trend (D1/H1)**: Analyze daily/hourly charts
   - Clear trend or at range boundary
3. **Trend strength**: Assess using price action + volumes + ATR
   - Strong / Medium / Weak
4. **Market phase**: Determine phase
   - Accumulation / Impulse / Correction / Retest / Flat / Exhaustion
   - **REJECT** if in flat in middle of range

#### 2.4 Apply 95% Rule

**Critical**: Strictly reject instrument if picture is not "super clear". Only 5% should pass.

Ask yourself:
- Is the setup obvious and clear?
- Are levels well-defined?
- Is trend unambiguous?
- Is entry point clear?

If not "super clear" → **REJECT**.

#### 2.5 Filter Problematic Patterns

**REJECT** instruments with:
- "Infected zones" (areas with many false breakouts)
- Deep false breakouts
- "Saw" patterns (choppy, unclear price action)
- Multiple conflicting signals

#### 2.6 Check Correlation (Stocks Only)

For stocks, check correlation with main index:

1. Get index data:
   - S&P 500 (^GSPC) for US stocks
   - NASDAQ (^IXIC) for tech stocks
   - Use `get_stock_history` for both instrument and index

2. Compare price action:
   - Prefer instruments with **low correlation** - "own life"
   - For some instruments (e.g., Apple) correlation is critical
   - For InPlay (news-driven) correlation is secondary

3. Check index synchronization:
   - Compare Dow Jones, Nasdaq, S&P 500
   - Divergence between indices = risk signal

4. Market context:
   - If market falling (S&P 500 negative), avoid longs in weak stocks

### Step 3: Filter Results

Keep only instruments that passed all criteria.

### Step 4: Return Candidates

Return list of 10-20 candidates (or fewer if strict filtering).

## Output Format

Use this format for screening results:

```markdown
# Скрининг инструментов - {DATE}

## Критерии
- **Акции**: Average Volume > 300k (стандарт) / > 500k (высокая ликвидность), ATR > $1, цена >$0.50, только США
- **Криптовалюты**: Ликвидность > $1M, объём > 50-100M (в зависимости от риска)
- **Forex**: Коэффициент ликвидности > 4 (идеально > 5)
- **Правило 95%**: Торговать только 5% с "супер понятной картинкой"
- Глобальный тренд (W1/MN): Восходящий/нисходящий/боковой (см. @trends)
- Локальный тренд (D1/H1): Чёткий тренд или граница диапазона (см. @trends)
- Сила тренда: Сильный/средний/слабый (см. @trends)
- Фаза рынка: Накопление/импульс/коррекция/ретест/флэт/истощение (см. @trends)
- Корреляция: Инструмент имеет "собственную жизнь" или низкая корреляция с индексом (для акций)

## Результаты

| Символ | Цена | Avg Volume | ATR | Изменение | Глоб. тренд | Лок. тренд | Сила | Фаза | Корреляция с индексом | Статус |
|--------|------|------------|-----|-----------|-------------|------------|------|------|----------------------|--------|
| AAPL   | $185.50 | 65M (>500k ✅) | $2.10 (>$1 ✅) | +1.2% | Up (W1) | Up (D1) | Сильный | Импульс | Низкая (собственная жизнь) | ✅     |
| TSLA   | $245.30 | 120M (>500k ✅) | $0.85 (<$1 ❌) | -0.5% | Down (W1) | Flat (D1) | Слабый | Флэт | Высокая (следует за S&P) | ❌     |

**Примечания к таблице**:
- **Avg Volume**: Средний объём (должен быть > 300k для стандарта или > 500k для высокой ликвидности)
- **ATR**: Average True Range (должен быть > $1 для акций)
- **Цена**: Должна быть >$0.50 (или >$10 для основного списка)
- **Статус**: ✅ — проходит все критерии, ❌ — не проходит критерии

## Кандидаты для анализа
1. AAPL
2. MSFT
...
```

## MCP Tools Reference

### user-markethub-mcp

**get_real_time_quote**
- Purpose: Get current price, spread, change
- Args: `symbol` (string) - ticker symbol
- Works for: stocks, crypto, forex

**get_stock_history**
- Purpose: Historical OHLCV data for stocks
- Args:
  - `symbol` (string) - ticker symbol
  - `interval` (string) - "1m", "5m", "15m", "1h", "1d", "1wk", "1mo"
  - `period` (string) - "1d", "5d", "1mo", "3mo", "6mo", "1y", "2y", "5y", "max"
  - `start_date`, `end_date` (optional) - YYYY-MM-DD format
- Use for: stocks only (causes error for crypto)

**get_crypto_history**
- Purpose: Historical OHLCV data for crypto
- Args:
  - `symbol` (string) - crypto symbol (e.g., "BTCUSDT")
  - `interval` (string) - same as get_stock_history
  - `period` (string) - same as get_stock_history
  - `exchange` (optional) - exchange name (e.g., "binance")
- Use for: crypto only

**get_market_stats**
- Purpose: Market statistics including ATR, Average Volume
- Args: `symbol` (string) - ticker symbol
- Works for: stocks only

### user-gerchik-kb

Use for:
- Clarifying Gerchik methodology rules
- Checking setup compliance with methodology
- Answering theory questions

## Common Mistakes

1. **Not applying 95% rule strictly** - Remember: reject 95%, only 5% pass
2. **Trading unclear setups** - Only trade "super clear picture"
3. **Ignoring correlation** - Check correlation with indices for stocks
4. **Not checking all timeframes** - Must analyze W1, D1, H1
5. **Accepting flat markets** - Reject instruments in flat in middle of range
6. **Missing basic criteria** - Always check volume, ATR, price first

## Quick Reference

| Criterion | Stocks | Crypto | Forex |
|-----------|--------|--------|-------|
| Volume | >300k (std) / >500k (high) | >50-100M | N/A |
| ATR | >$1 | N/A | N/A |
| Price | >$0.50 | N/A | N/A |
| Liquidity | N/A | >$1M | Coef >4 (>5 ideal) |
| Trend | Required (W1/D1/H1) | Required (W1/D1/H1) | Required (D1/H1) |
| Phase | Avoid flat | Avoid flat | Avoid flat |
| Correlation | Check with indices | N/A | N/A |
