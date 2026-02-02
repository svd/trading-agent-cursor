---
name: chart-generation
description: Generate trading charts via MCP generate_chart tool. Use when user requests chart generation, needs to visualize price levels, or wants to create trading charts. Prevents infinite loops and retry attempts.
---

# Chart Generation via MCP

Generate trading charts using MCP `generate_chart` tool with retry logic and error handling.

## When to Use

Apply this skill when:
- User requests chart generation for a trading instrument
- User wants to visualize price levels on a chart
- User asks to create or update a chart
- User mentions "график", "chart", "generate chart", "create chart"

## Critical Rules to Prevent Infinite Loops

### 1. File Overwrite on Regeneration

**On re-request, always generate chart and overwrite old file.**

**Agent actions:**
1. Check if chart file exists (using `read_file` or `list_dir`)
2. If file exists - inform user, but continue
3. **ALWAYS** call MCP `generate_chart` on request (old file will be overwritten automatically)
4. **DO NOT skip** generation even if file exists

### 2. Maximum 3 Attempts with Decreasing Date Range

**MAXIMUM 3 attempts** for one chart with decreasing date range on failure.

**Agent actions:**
1. **Attempt 1**: Range **5 months** (`start_date`: date 5 months ago)
2. If generation failed - **Attempt 2**: Range **4 months** (`start_date`: date 4 months ago)
3. If attempt 2 failed - **Attempt 3**: Range **3 months** (`start_date`: date 3 months ago)
4. After each attempt check success (read file or check existence via `read_file`/`list_dir`)
5. If successful - **STOP**
6. After 3 attempts - **STOP** and report error

### 3. Success Verification After Call

**ALWAYS** verify success of MCP `generate_chart` call.

**Agent actions:**
1. After calling `call_mcp_tool` with `generate_chart` check result
2. Use `read_file` or `list_dir` to check file existence
3. If file exists and not empty - **STOP**, success
4. If file not created or empty - retry (max 2-3 times)

### 4. Data Check Before Generation

**Check** OHLCV record count **once** before each generation attempt with new range.

**Agent actions:**
1. Start with range **5 months** (`start_date`: date 5 months ago)
2. Get OHLCV data via MCP `get_stock_history` (using `call_mcp_tool`) for selected range
3. Check record count **once** before generation
4. Call MCP `generate_chart` with data
5. If generation failed - decrease range to **4 months** and repeat steps 2-4
6. If second attempt failed - decrease range to **3 months** and repeat steps 2-4
7. **DO NOT check** data repeatedly after failed generation with same range (causes infinite loop!)

### 5. No Additional Charts After Success

**After successful chart generation** (file created and not empty) **DO NOT create** additional charts or versions.

**Agent actions:**
1. After successful generation (file exists and not empty) - **STOP**, complete process
2. **DO NOT create** additional charts for other ranges (3 and 4 months) after successful generation
3. **DO NOT create** additional chart versions (OHLC versions, other types) after successful generation
4. Ranges 4 and 3 months are used **only** for retry attempts when 5-month range fails

## Chart Generation Algorithm

### Step 1: Determine Chart File Path

**For stocks and other instruments:**
- Path: `homework/YYYY-MM-DD/SYMBOL.png`
- Example: `homework/2026-01-25/AAPL.png`

**For cryptocurrencies:**
- Path: `homework/YYYY-MM-DD/SYMBOL-EXCHANGE.png`
- Example: `homework/2026-01-25/BTCUSDT-Binance.png`

### Step 2: Check File Existence

Use `read_file` or `list_dir` to check if file exists:
- If file exists - inform user, but continue (will be overwritten)
- File will be automatically overwritten on generation

### Step 3: Generation with Decreasing Range on Failure

**Attempt 1: 5 months range**
- Get OHLCV data via MCP `get_stock_history` (using `call_mcp_tool`)
  - `start_date`: date 5 months ago (format: "YYYY-MM-DD")
  - `end_date`: current date (format: "YYYY-MM-DD")
  - `interval`: "1d"
- Call MCP `generate_chart` via `call_mcp_tool`:
  ```json
  {
    "server": "user-markethub-mcp",
    "toolName": "generate_chart",
    "arguments": {
      "input_data": {
        "ticker": "SYMBOL",
        "title": "SYMBOL - 5 месяцев",
        "ohlcv": [...],
        "levels": [...],
        "chart_type": "candle",
        "style": "light"
      }
    }
  }
  ```
- Verify success (using `read_file` or `list_dir`)
- If file exists and not empty → **STOP**, success

**Attempt 2: 4 months range** (if attempt 1 failed)
- Get OHLCV data via MCP `get_stock_history` with `start_date` = date 4 months ago
- Call MCP `generate_chart` with `title`: "SYMBOL - 4 месяца"
- Verify success
- If file exists and not empty → **STOP**, success

**Attempt 3: 3 months range** (if attempt 2 failed)
- Get OHLCV data via MCP `get_stock_history` with `start_date` = date 3 months ago
- Call MCP `generate_chart` with `title`: "SYMBOL - 3 месяца"
- Verify success
- If file exists and not empty → **STOP**, success

### Step 4: After Successful Generation

- Verify success (file exists and not empty)
- **STOP**, complete process
- **DO NOT create** additional chart versions (OHLC, other types)
- **DO NOT create** charts for other ranges

### Step 5: After All Attempts Failed

- Stop and inform user about error
- **DO NOT continue** infinite attempts
- Maximum 3 attempts (5 → 4 → 3 months)

## MCP Tool Usage

### Get Historical Data

**For stocks:**
- `server`: `"user-markethub-mcp"`
- `toolName`: `"get_stock_history"`
- `arguments`: 
  - `symbol`: stock ticker (e.g., "AAPL")
  - `start_date`: date N months ago (format: "YYYY-MM-DD")
  - `end_date`: current date (format: "YYYY-MM-DD")
  - `interval`: "1d"

**For cryptocurrencies:**
- `server`: `"user-markethub-mcp"`
- `toolName`: `"get_crypto_history"`
- `arguments`: 
  - `symbol`: crypto symbol (e.g., "BTCUSDT")
  - `start_date`: date N months ago (format: "YYYY-MM-DD")
  - `end_date`: current date (format: "YYYY-MM-DD")
  - `interval`: "1d"
  - `exchange`: optional, exchange name (e.g., "binance")

### Generate Chart

- `server`: `"user-markethub-mcp"`
- `toolName`: `"generate_chart"`
- `arguments`: 
  ```json
  {
    "input_data": {
      "ticker": "SYMBOL",
      "title": "SYMBOL - N месяцев",
      "ohlcv": [...],
      "levels": [...],
      "chart_type": "candle",
      "style": "light",
      "renderer": "mplfinance"
    }
  }
  ```

### Level Format Conversion

Convert levels from analysis to `ChartLevel` format:
- **Level type**: 
  - "Поддержка" → `support`
  - "Сопротивление" → `resistance`
- **Level strength**:
  - "Слабый" → `weak`
  - "Средний" → `medium`
  - "Сильный" → `strong`
  - "Очень сильный" → `very_strong`
- **Price**: Use level price from analysis
- **Label**: Optional, brief description (e.g., "D1 High", "Круглое число", "Излом тренда")

## Chart File Handling

### Save Chart Image

MCP tool returns base64-encoded image:
1. Decode base64
2. Save image to file:
   - For stocks: `homework/YYYY-MM-DD/SYMBOL.png`
   - For crypto: `homework/YYYY-MM-DD/SYMBOL-EXCHANGE.png`
3. Add link to chart in homework file

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

## Prohibited Actions

❌ **DO NOT:**
- Skip generation if file already exists (always generate on request)
- Repeat generation more than 3 times (5 → 4 → 3 months) without success check
- Use range more than 5 months on first attempt
- Continue attempts after successful generation
- Create charts for other ranges (3 and 4 months) after successful chart generation
- Create additional chart versions (OHLC versions, other types) after successful main chart generation
- Check OHLCV record count repeatedly in loop after failed generation with same range
- Use Python examples as executable code (they are only logic illustrations)

## Important Reminders

- **Repeated generation** - always generate chart on request and overwrite old file
- **Decreasing range on failure** - start with 5 months, on failure try 4 months, then 3 months
- **Attempt limit** - maximum 3 attempts (5 → 4 → 3 months), then stop
- **Success check** - always check file existence and size after `generate_chart` call
- **Stop after success** - after successful chart generation **STOP**, do not create additional versions or charts for other ranges
- **No additional versions** - after successful chart generation DO NOT create additional versions (OHLC, other types) and DO NOT create charts for other ranges
- **Update title** - update chart `title` according to actual data range (5/4/3 months)
- **Use MCP** - always use `call_mcp_tool` to call `generate_chart` and `get_stock_history`
- **File tools** - use `read_file` and `list_dir` to check file existence
- **Python examples** - these are logic illustrations, not executable code. Agent must apply logic through available tools
