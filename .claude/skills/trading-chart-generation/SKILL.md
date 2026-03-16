---
name: trading-chart-generation
description: Use when user requests chart generation, needs to visualize price levels, or wants to create trading charts. Generates trading charts via MCP generate_chart_from_file tool with retry logic and error handling. Prevents infinite loops and retry attempts.
---

# Trading Chart Generation

## Overview

Generate trading charts using MCP `generate_chart_from_file` tool with retry logic and error handling. Creates candlestick charts with support/resistance levels for trading analysis.

## When to Use

Apply this skill when:
- User requests chart generation for a trading instrument
- User wants to visualize price levels on a chart
- User asks to create or update a chart
- User mentions "график", "chart", "generate chart", "create chart"
- Completing homework (Stage 7 of homework workflow)

## The Process

### Step 1: Determine Chart File Paths

**For stocks and other instruments:**
- Input JSON: `homework/YYYY-MM-DD/data/SYMBOL_PERIOD_chart_input.json`
  - Example: `homework/2026-01-25/data/AAPL_1D_chart_input.json`
- Output PNG: `homework/YYYY-MM-DD/charts/SYMBOL_PERIOD.png`
  - Example: `homework/2026-01-25/charts/AAPL_1D.png`

**For cryptocurrencies:**
- Input JSON: `homework/YYYY-MM-DD/data/SYMBOL-EXCHANGE_PERIOD_chart_input.json`
  - Example: `homework/2026-01-25/data/BTCUSDT-Binance_1D_chart_input.json`
- Output PNG: `homework/YYYY-MM-DD/charts/SYMBOL-EXCHANGE_PERIOD.png`
  - Example: `homework/2026-01-25/charts/BTCUSDT-Binance_1D.png`

**Period mapping:**
- From interval `1d` → use `1D` in filename
- From interval `1h` → use `1H` in filename (if intraday added later)

### Step 2: Check File Existence

Use `read_file` or `list_dir` to check if file exists:
- If file exists - inform user, but continue (will be overwritten)
- File will be automatically overwritten on generation

### Step 3: Generation with Decreasing Range on Failure

**MAXIMUM 3 attempts** for one chart with decreasing date range on failure.

**Attempt 1: 5 months range**
- Get OHLCV data via MCP `get_stock_history` (using `call_mcp_tool`)
  - `start_date`: date 5 months ago (format: "YYYY-MM-DD")
  - `end_date`: current date (format: "YYYY-MM-DD")
  - `interval`: "1d"
- Build ChartInput object:
  ```json
  {
    "ticker": "SYMBOL",
    "title": "SYMBOL - 5 месяцев",
    "ohlcv": [...],
    "levels": [...],
    "chart_type": "candle",
    "style": "light",
    "renderer": "mplfinance"
  }
  ```
- Write ChartInput to JSON file: `homework/YYYY-MM-DD/data/SYMBOL_1D_chart_input.json`
- Call MCP `generate_chart_from_file` via `call_mcp_tool`:
  ```json
  {
    "server": "user-market-charts",
    "toolName": "generate_chart_from_file",
    "arguments": {
      "input_path": "/absolute/path/to/homework/YYYY-MM-DD/data/SYMBOL_1D_chart_input.json",
      "output_path": "/absolute/path/to/homework/YYYY-MM-DD/charts/SYMBOL_1D.png"
    }
  }
  ```
- Verify success: tool returns `success: true` and/or file exists (using `read_file` or `list_dir`)
- If file exists and not empty → **STOP**, success
- Optionally delete input JSON file after success

**Attempt 2: 4 months range** (if attempt 1 failed)
- Get OHLCV data via MCP `get_stock_history` with `start_date` = date 4 months ago
- Build ChartInput with `title`: "SYMBOL - 4 месяца"
- Write to JSON file and call `generate_chart_from_file`
- Verify success
- If file exists and not empty → **STOP**, success

**Attempt 3: 3 months range** (if attempt 2 failed)
- Get OHLCV data via MCP `get_stock_history` with `start_date` = date 3 months ago
- Build ChartInput with `title`: "SYMBOL - 3 месяца"
- Write to JSON file and call `generate_chart_from_file`
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

## Critical Rules to Prevent Infinite Loops

### 1. File Overwrite on Regeneration

**On re-request, always generate chart and overwrite old file.**

**Agent actions:**
1. Check if chart file exists (using `read_file` or `list_dir`)
2. If file exists - inform user, but continue
3. **ALWAYS** call MCP `generate_chart_from_file` on request (old file will be overwritten automatically)
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

**ALWAYS** verify success of MCP `generate_chart_from_file` call.

**Agent actions:**
1. After calling `call_mcp_tool` with `generate_chart_from_file` check result (should return `success: true`)
2. Use `read_file` or `list_dir` to check file existence
3. If file exists and not empty - **STOP**, success
4. If file not created or empty - retry (max 2-3 times)

### 4. Data Check Before Generation

**Check** OHLCV record count **once** before each generation attempt with new range.

**Agent actions:**
1. Start with range **5 months** (`start_date`: date 5 months ago)
2. Get OHLCV data via MCP `get_stock_history` (using `call_mcp_tool`) for selected range
3. Check record count **once** before generation
4. Write ChartInput to JSON file, then call MCP `generate_chart_from_file` with file paths
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

### Generate Chart from File

- `server`: `"user-market-charts"`
- `toolName`: `"generate_chart_from_file"`
- `arguments`:
  - `input_path`: Absolute path to JSON file containing ChartInput data
  - `output_path`: Absolute path where PNG chart will be saved
- **Input JSON file** must contain ChartInput object with same schema:
  ```json
  {
    "ticker": "SYMBOL",
    "title": "SYMBOL - N месяцев",
    "ohlcv": [...],
    "levels": [...],
    "chart_type": "candle",
    "style": "light",
    "renderer": "mplfinance"
  }
  ```
- Returns: `{ "success": true, "output_path": "...", "format": "png", "metadata": {...} }`
- No base64 image in response; PNG is written directly to `output_path`

**Note:** For inline data (not recommended for large OHLCV series), `generate_chart` tool exists on the same server but should be avoided to prevent context overload.

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

MCP tool writes PNG directly to `output_path`:
1. Prepare ChartInput JSON with ticker, ohlcv, levels, chart_type, style, renderer, title
2. Write JSON to input file with period in name:
   - For stocks: `homework/YYYY-MM-DD/data/SYMBOL_PERIOD_chart_input.json` (e.g., `data/AAPL_1D_chart_input.json`)
   - For crypto: `homework/YYYY-MM-DD/data/SYMBOL-EXCHANGE_PERIOD_chart_input.json` (e.g., `data/BTCUSDT-Binance_1D_chart_input.json`)
3. Call `generate_chart_from_file` with absolute paths for `input_path` and `output_path`
4. Tool generates and saves PNG to output file:
   - For stocks: `homework/YYYY-MM-DD/charts/SYMBOL_PERIOD.png` (e.g., `charts/AAPL_1D.png`)
   - For crypto: `homework/YYYY-MM-DD/charts/SYMBOL-EXCHANGE_PERIOD.png` (e.g., `charts/BTCUSDT-Binance_1D.png`)
5. Verify file exists and is not empty
6. Optionally delete input JSON file after successful generation to avoid clutter
7. Add link to chart in homework file

### Add Chart to Homework File

**For stocks:**
```markdown
## График

![График {SYMBOL}](./charts/{SYMBOL}_1D.png)
```

**For cryptocurrencies:**
```markdown
## График

![График {SYMBOL} ({EXCHANGE})](./charts/{SYMBOL}-{EXCHANGE}_1D.png)
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

- **File-based generation** - use `generate_chart_from_file` with JSON input file to avoid sending large OHLCV data in context
- **Write ChartInput first** - always write ChartInput to JSON file before calling `generate_chart_from_file`
- **Period in filenames** - include period (e.g., `1D`) in both input JSON and output PNG filenames
- **Absolute paths** - use absolute paths for `input_path` and `output_path` arguments (workspace root + relative path)
- **Repeated generation** - always generate chart on request and overwrite old files
- **Decreasing range on failure** - start with 5 months, on failure try 4 months, then 3 months
- **Attempt limit** - maximum 3 attempts (5 → 4 → 3 months), then stop
- **Success check** - always check file existence and size after `generate_chart_from_file` call (tool returns `success: true`)
- **Stop after success** - after successful chart generation **STOP**, do not create additional versions or charts for other ranges
- **No additional versions** - after successful chart generation DO NOT create additional versions (OHLC, other types) and DO NOT create charts for other ranges
- **Update title** - update chart `title` according to actual data range (5/4/3 months)
- **Use MCP** - always use `call_mcp_tool` to call `generate_chart_from_file` and `get_stock_history`
- **File tools** - use `read_file` and `list_dir` to check file existence
- **Clean up** - optionally delete input JSON files after successful generation
- **Python examples** - these are logic illustrations, not executable code. Agent must apply logic through available tools

## References

- @trading-homework - Full homework workflow (chart generation is Stage 7)
- @trading-levels - Level identification and assessment (for level data on chart)
