# Trading Strategies Directory / Trading Strategies

This directory stores **natural language trading strategy files** (YAML format). The system automatically loads all `.yaml` files in this directory at startup.

For users and documentation, we continue calling these abilities "strategies"; in code, configuration, and API fields, they are uniformly named `skill`, which you can understand as "reusable strategy capability packages".

## How to Write Custom Strategies (Strategy Skill)

Simply create a `.yaml` file describing your trading strategy in natural language (English or any language), **no code needed**.

### Minimal Template

```yaml
name: my_strategy          # Unique identifier (English, underscore-separated)
display_name: My Strategy  # Display name
description: Brief description of strategy purpose

instructions: |
  Your strategy description...
  Write in natural language your judgment criteria, entry conditions, exit conditions, etc.
  Can reference tool names (like get_daily_history, analyze_trend) to guide AI which data to use.
```

### Complete Template

```yaml
name: my_strategy
display_name: My Strategy
description: Brief description of strategy suitable market scenario

# Strategy category: trend, pattern, reversal, framework
category: trend

# Associated core trading rule numbers (1-7), optional
core_rules: [1, 2]

# List of tools this strategy needs, optional
# Available tools: get_daily_history, analyze_trend, get_realtime_quote,
#                 get_sector_rankings, search_stock_news, get_stock_info
required_tools:
  - get_daily_history
  - analyze_trend

# Optional aliases (for use in /ask and natural language skill selection)
aliases: [my_method, my_model]

# Following metadata drives default behavior (optional)
# default_active: whether belongs to default activated skill set
# default_router: whether belongs to routing fallback skill set
# default_priority: default display/sorting priority, lower values rank higher
# market_regimes: market state tags this skill preferentially adapts to
default_active: true
default_router: false
default_priority: 100
market_regimes: [trending_up]

# Detailed strategy explanation (natural language, supports Markdown)
instructions: |
  **My Strategy Name**

  Judgment Criteria:

  1. **Condition One**:
     - Use `analyze_trend` to check moving average arrangement.
     - Describe the trend characteristics you expect to see...

  2. **Condition Two**:
     - Describe volume requirements...

  Score Adjustment:
  - Suggested sentiment_score adjustment when conditions are met
  - Note strategy name in `buy_reason`
```

### Core Trading Rules Reference

| Number | Rule |
|--------|------|
| 1 | Strict Entry: Divergence rate < 5% to consider entry |
| 2 | Trend Trading: MA5 > MA10 > MA20 bullish arrangement |
| 3 | Efficiency First: Volume confirms trend validity |
| 4 | Buy Point Preference: Prioritize pullback to MA support |
| 5 | Risk Screening: Negative news vetoes entry |
| 6 | Volume-Price Coordination: Trading volume verifies price movement |
| 7 | Strong Trend Stocks Relaxed: Leading stocks can relax standards appropriately |

## Custom Strategy Directory

Besides this directory (built-in strategies), you can specify additional custom strategy directories via environment variable:

```env
AGENT_SKILL_DIR=./my_skills
```

The system will load both built-in and custom strategies. If names conflict, custom strategies override built-in ones.

The environment variable name is still `AGENT_SKILL_DIR`, this is the internal unified naming convention; in product semantics, it still represents "custom strategy directory".
