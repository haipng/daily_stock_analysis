---
name: "stock_analyzer"
description: "Analyze stocks and markets. Call when user wants to analyze individual or multiple stocks, or perform market review."
---

# Stock Analyzer

This skill is based on the logic of `src/services/analyzer_service.py`, providing functions to analyze stocks and overall market.

## Output Structure (`AnalysisResult`)

The analysis function returns an `AnalysisResult` object (or list), which has a rich structure. Below is a brief overview of its key components with real output examples:

The `dashboard` property contains core analysis divided into four main parts:
1.  **`core_conclusion`**: One-sentence summary, signal type, and position suggestion.
2.  **`data_perspective`**: Technical data including trend status, price location, volume analysis, and chip structure.
3.  **`intelligence`**: Qualitative information such as news, risk alerts, and positive catalysts.
4.  **`battle_plan`**: Actionable strategies including snipe points (buy/sell targets), position strategy, and risk control checklist.

## Configuration (`Config`)

All analysis functions can accept an optional `config` object. This object contains all application configuration such as API keys, notification settings, and analysis parameters.

If no `config` object is provided, the function automatically uses the global singleton instance loaded from `.env` file.

**Reference:** [`Config`](src/config.py)

## Functions

### 1. Analyze Single Stock

**Description:** Analyze a single stock and return analysis result.

**When to use:** When user requests analysis of a specific stock.

**Input:**
- `stock_code` (str): Stock code to analyze.
- `config` (Config, optional): Configuration object. Default is `None`.
- `full_report` (bool, optional): Whether to generate full report. Default is `False`.
- `notifier` (NotificationService, optional): Notification service object. Default is `None`.

**Output:** `Optional[AnalysisResult]`
An `AnalysisResult` object containing analysis result, or `None` if analysis fails.

**Example:**

```python
from src.services.analyzer_service import analyze_stock

# Analyze single stock
result = analyze_stock("600989")
if result:
    print(f"Stock: {result.name} ({result.code})")
    print(f"Sentiment Score: {result.sentiment_score}")
    print(f"Operation Advice: {result.operation_advice}")
```

**Reference:** [`analyze_stock`](src/services/analyzer_service.py)

### 2. Analyze Multiple Stocks

**Description:** Analyze a list of stocks and return analysis result list.

**When to use:** When user wants to analyze multiple stocks at once.

**Input:**
- `stock_codes` (List[str]): List of stock codes to analyze.
- `config` (Config, optional): Configuration object. Default is `None`.
- `full_report` (bool, optional): Whether to generate full report for each stock. Default is `False`.
- `notifier` (NotificationService, optional): Notification service object. Default is `None`.

**Output:** `List[AnalysisResult]`
A list of `AnalysisResult` objects.

**Example:**

```python
from src.services.analyzer_service import analyze_stocks

# Analyze multiple stocks
results = analyze_stocks(["600989", "000001"])
for result in results:
    print(f"Stock: {result.name}, Operation Advice: {result.operation_advice}")
```

**Reference:** [`analyze_stocks`](src/services/analyzer_service.py)

### 3. Perform Market Review

**Description:** Perform market review of overall market and return a report.

**When to use:** When user requests market overview, summary, or review.

**Input:**
- `config` (Config, optional): Configuration object. Default is `None`.
- `notifier` (NotificationService, optional): Notification service object. Default is `None`.

**Output:** `Optional[str]`
A string containing market review report, or `None` if fails.

**Example:**

```python
from src.services.analyzer_service import perform_market_review

# Perform market review
report = perform_market_review()
if report:
    print(report)
```

**Reference:** [`perform_market_review`](src/services/analyzer_service.py)
