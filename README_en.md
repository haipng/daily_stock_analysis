<div align="center">

# 📈 Intelligent Stock Analysis System

[![GitHub stars](https://img.shields.io/github/stars/ZhuLinsen/daily_stock_analysis?style=social)](https://github.com/ZhuLinsen/daily_stock_analysis/stargazers)
[![CI](https://github.com/ZhuLinsen/daily_stock_analysis/actions/workflows/ci.yml/badge.svg)](https://github.com/ZhuLinsen/daily_stock_analysis/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-Ready-2088FF?logo=github-actions&logoColor=white)](https://github.com/features/actions)
[![Docker](https://img.shields.io/badge/Docker-Ready-2496ED?logo=docker&logoColor=white)](https://hub.docker.com/r/zhulinsen/daily_stock_analysis)

<p align="center">
  <a href="https://trendshift.io/repositories/18527" target="_blank"><img src="https://trendshift.io/api/badge/repositories/18527" alt="ZhuLinsen%2Fdaily_stock_analysis | Trendshift" width="230" /></a>&nbsp;<a href="https://hellogithub.com/repository/ZhuLinsen/daily_stock_analysis" target="_blank"><img src="https://api.hellogithub.com/v1/widgets/recommend.svg?rid=6daa16e405ce46ed97b4a57706aeb29f&claim_uid=pfiJMqhR9uvDGlT&theme=neutral" alt="Featured｜HelloGitHub" width="230" /></a>
</p>

> 🤖 AI-powered intelligent stock analysis system for A-shares/Hong Kong stocks/US stocks. Automatically analyzes and pushes "decision dashboard" daily to WeChat Work/Feishu/Telegram/Discord/Slack/Email

[**Product Preview**](#-product-preview) · [**Features**](#-features) · [**Quick Start**](#-quick-start) · [**Push Results**](#-push-results) · [**Documentation Center**](docs/INDEX_EN.md) · [**Complete Guide**](docs/full-guide_EN.md)

简体中文 | English | [繁體中文](docs/README_CHT.md)

</div>

## 💖 Sponsors
<div align="center">
  <p align="center">
    <a href="https://open.anspire.cn/?share_code=QFBC0FYC" target="_blank"><img src="./docs/assets/anspire.png" alt="Anspire Open - One-stop model and search service" width="300" height="141" style="width: 300px; height: 141px; object-fit: contain;"></a>
    <a href="https://serpapi.com/baidu-search-api?utm_source=github_daily_stock_analysis" target="_blank"><img src="./docs/assets/serpapi_banner_zh.png" alt="Easily fetch real-time financial news data from search engines - SerpApi" width="300" height="141" style="width: 300px; height: 141px; object-fit: contain;"></a>
  </p>
</div>

## 🖥️ Product Preview

<p align="center">
  <img src="docs/assets/readme_workspace_tour_20260510.gif" alt="DSA Web Workspace Demo" width="720">
</p>

## ✨ Features

| Capability | Coverage |
|----------|----------|
| AI Decision Report | Core conclusions, scoring, trends, buy/sell targets, risk alerts, catalysts, action checklist |
| Multi-Market Data Aggregation | A-shares, Hong Kong stocks, US stocks, ETFs; quotes, K-lines, technical indicators, fund flows, chip distribution, news, announcements, fundamentals |
| Web / Desktop Workbench | Manual analysis, task progress, historical reports, full Markdown, backtesting, portfolio management, light/dark themes |
| Agent Strategy Chat | Multi-turn Q&A with 15+ built-in strategies (Moving averages, Chan Theory, Wave Theory, Trends, Hot topics, Events, Growth, Expectations), covering Web/Bot/API |
| Intelligent Import & Auto-Complete | Image, CSV/Excel, clipboard import; stock code/name/pinyin/alias auto-completion |
| Automation & Push | GitHub Actions, Docker, local scheduled tasks, FastAPI service, and WeChat Work/Feishu/Telegram/Discord/Slack/Email push |

> For feature details, field contracts, fundamental P0 timeout semantics, trading discipline, data source priority, and Web/API behavior, see [Complete Configuration & Deployment Guide](docs/full-guide_EN.md).

### Technology Stack & Data Sources

| Type | Support |
|------|---------|
| AI Models | [Anspire](https://open.anspire.cn/?share_code=QFBC0FYC), [AIHubMix](https://aihubmix.com/?aff=CfMq), Gemini, OpenAI-compatible, DeepSeek, Qwen, Claude, Ollama local models, etc. |
| Market Data | [TickFlow](https://tickflow.org/auth/register?ref=WDSGSPS5XC), AkShare, Tushare, Pytdx, Baostock, YFinance, Longbridge |
| News Search | [Anspire](https://open.anspire.cn/?share_code=QFBC0FYC), [SerpAPI](https://serpapi.com/baidu-search-api?utm_source=github_daily_stock_analysis), [Tavily](https://tavily.com/), [Bocha](https://open.bocha.cn/), [Brave](https://brave.com/search/api/), [MiniMax](https://platform.minimaxi.com/), SearXNG |
| Social Sentiment | [Stock Sentiment API](https://api.adanos.org/docs) (Reddit / X / Polymarket, US stocks only, optional) |

> Complete rules at [Data Source Configuration](docs/full-guide_EN.md#data-source-configuration).

## 🚀 Quick Start

### Method 1: GitHub Actions (Recommended)

> Deploy in 5 minutes, zero cost, no server required.

#### 1. Fork This Repository

Click `Fork` button in top right (also give a Star ⭐ to support)

#### 2. Configure Secrets

`Settings` → `Secrets and variables` → `Actions` → `New repository secret`

**AI Model Configuration (Configure at least one)**

Select a model provider and fill in the API Key. For multi-model, image recognition, local models, or advanced routing, refer to [LLM Configuration Guide](docs/LLM_CONFIG_GUIDE_EN.md).

| Secret Name | Description | Required |
|------------|-------------|:--------:|
| `ANSPIRE_API_KEYS` | [Anspire](https://open.anspire.cn/?share_code=QFBC0FYC) API Key - one key enables popular global models and web search, no proxy needed, includes free credits | **Recommended** |
| `AIHUBMIX_KEY` | [AIHubMix](https://aihubmix.com/?aff=CfMq) API Key - switch between all models with one key, no proxy needed, 10% discount available | **Recommended** |
| `GEMINI_API_KEY` | Google Gemini API Key | Optional |
| `ANTHROPIC_API_KEY` | Anthropic Claude API Key | Optional |
| `OPENAI_API_KEY` | OpenAI-compatible API Key (supports DeepSeek, Qwen, etc.) | Optional |
| `OPENAI_BASE_URL` / `OPENAI_MODEL` | Fill when using OpenAI-compatible services | Optional |

> Ollama is better for local/Docker deployment. GitHub Actions recommends cloud APIs.

**Notification Channel Configuration (Configure at least one)**

| Secret Name | Description |
|------------|-------------|
| `WECHAT_WEBHOOK_URL` | WeChat Work bot |
| `FEISHU_WEBHOOK_URL` | Feishu bot |
| `TELEGRAM_BOT_TOKEN` + `TELEGRAM_CHAT_ID` | Telegram |
| `DISCORD_WEBHOOK_URL` | Discord Webhook |
| `SLACK_BOT_TOKEN` + `SLACK_CHANNEL_ID` | Slack Bot |
| `EMAIL_SENDER` + `EMAIL_PASSWORD` | Email push |

For more channels, signature verification, batch emails, Markdown to image, see [Notification Channel Detailed Configuration](docs/full-guide_EN.md#notification-channels-detailed-configuration).

**Watchlist Configuration (Required)**

| Secret Name | Description | Required |
|------------|-------------|:--------:|
| `STOCK_LIST` | Watchlist codes, e.g., `600519,hk00700,AAPL,TSLA` | ✅ |

**News Source Configuration (Recommended)**

News sources significantly impact sentiment quality, news, events, and catalyst quality. Recommend configuring at least one search service.

| Secret Name | Description | Required |
|------------|-------------|:--------:|
| `ANSPIRE_API_KEYS` | [Anspire AI Search](https://aisearch.anspire.cn/) - optimized for Chinese content, suitable for A-share news and sentiment; same key works as Anspire LLM | **Recommended** |
| `SERPAPI_API_KEYS` | [SerpAPI](https://serpapi.com/baidu-search-api?utm_source=github_daily_stock_analysis) - search engine results, suitable for real-time financial news | **Recommended** |
| `TAVILY_API_KEYS` | [Tavily](https://tavily.com/) - general news search API | Optional |
| `BOCHA_API_KEYS` | [Bocha Search](https://open.bocha.cn/) - Chinese search optimization, AI summary support | Optional |
| `BRAVE_API_KEYS` | [Brave Search](https://brave.com/search/api/) - privacy-first, US stock news enhancement | Optional |
| `MINIMAX_API_KEYS` | [MiniMax](https://platform.minimaxi.com/) - structured search results | Optional |
| `SEARXNG_BASE_URLS` | SearXNG self-hosted instance - no quota, suitable for private deployment | Optional |

For more search sources, social sentiment, and fallback rules, see [Search Service Configuration](docs/full-guide_EN.md#search-service-configuration).

#### 3. Enable Actions

`Actions` tab → `I understand my workflows, go ahead and enable them`

#### 4. Manual Test

`Actions` → `Daily Stock Analysis` → `Run workflow` → `Run workflow`

#### Finished

Runs automatically every **weekday at 18:00 Beijing time** by default, or manually trigger anytime. Non-trading days (A/H/US holidays) skip by default; see [Complete Guide](docs/full-guide_EN.md#scheduled-tasks-configuration) for force run, trading day check, resumable transfer rules.

### Method 2: Local Run / Docker Deployment

```bash
# Clone repository
git clone https://github.com/ZhuLinsen/daily_stock_analysis.git && cd daily_stock_analysis

# Install dependencies
pip install -r requirements.txt

# Configure environment variables
cp .env.example .env && vim .env

# Run analysis
python main.py
```

Common commands:

```bash
python main.py --debug
python main.py --dry-run
python main.py --stocks 600519,hk00700,AAPL
python main.py --market-review
python main.py --schedule
python main.py --serve-only
```

> For Docker deployment, scheduled tasks, cloud server access, see [Complete Guide](docs/full-guide_EN.md); for desktop app packaging, see [Desktop Packaging Guide](docs/desktop-package_EN.md).

## 📱 Push Results

### Decision Dashboard
```
🎯 2026-02-08 Decision Dashboard
Total analyzed: 3 stocks | 🟢Buy:0 🟡Hold:2 🔴Sell:1

📊 Analysis Summary
⚪ Tungsten High-Tech (000657): Hold | Score 65 | Bullish
⚪ Yongding Stock (600105): Hold | Score 48 | Consolidating
🟡 Xinlai Materials (300260): Sell | Score 35 | Bearish

⚪ Tungsten High-Tech (000657)
📰 Key Information Snapshot
💭 Market Sentiment: Market focuses on AI attributes and strong earnings growth, sentiment is positive but needs to digest short-term profit-taking and major capital outflows.
📊 Earnings Forecast: Based on sentiment data, company Q1-Q3 2025 earnings show strong YoY growth, solid fundamentals providing price support.

🚨 Risk Alerts:

Risk Point 1: Major capital large net selling of CNY 363M on Feb 5, be alert to short-term selling pressure.
Risk Point 2: Chip concentration at 35.15%, indicating dispersed chips, potential large lifting resistance.
Risk Point 3: Sentiment mentions company's historical violations and restructuring risk warnings, need continued monitoring.
✨ Positive Catalysts:

Catalyst 1: Company positioned as core AI server HDI supplier, benefiting from AI industry development.
Catalyst 2: Q1-Q3 2025 net profit (ex-extraordinary) surged 407.52% YoY, strong performance.
📢 Latest News: Sentiment shows company is AI PCB micro-drilling leader, deeply bound with global PCB/substrate leaders. Major capital net sell CNY 363M on Feb 5, watch fund flow.

---
Generated: 18:00
```

### Market Review
```
🎯 2026-01-10 Market Review

📊 Major Indices
- Shanghai Composite: 3250.12 (🟢+0.85%)
- Shenzhen Component: 10521.36 (🟢+1.02%)
- ChiNext: 2156.78 (🟢+1.35%)

📈 Market Overview
Up: 3920 | Down: 1349 | Limit Up: 155 | Limit Down: 3

🔥 Sector Performance
Leading: Internet Services, Cultural Media, Minor Metals
Lagging: Insurance, Aviation Airports, Photovoltaic Equipment
```

## ⚙️ Configuration

For complete environment variables, model channels, notification channels, data source priority, trading discipline, fundamental P0 semantics, and deployment docs, see [Complete Configuration Guide](docs/full-guide_EN.md).

## 🖥️ Web Interface

Web workbench provides configuration management, task monitoring, manual analysis, historical reports, full Markdown reports, Agent chat, backtesting, portfolio management, intelligent import, and light/dark themes. Start with:

```bash
python main.py --webui
python main.py --webui-only
```

Access `http://127.0.0.1:8000` to use. For auth, intelligent import, search completion, historical report copy, cloud server access details, see [Local WebUI Management Interface](docs/full-guide_EN.md#local-webui-management-interface).

## 🤖 Agent Strategy Chat

After configuring any available AI API Key, the Web `/chat` page enables strategy chat; set `AGENT_MODE=false` to disable explicitly.

- Supports built-in strategies: Moving average crossover, Chan Theory, Wave Theory, Bull trend, Hot topics, Event-driven, Growth quality, Expectation revision, etc.
- Supports real-time quotes, K-lines, technical indicators, news, and risk information access
- Supports multi-turn Q&A, session export, push to notification channels, and background execution
- Supports custom strategy files and multi-Agent orchestration (experimental)

> For Agent parameters, `skill` naming compatibility, multi-Agent mode, and budget guards, see [Complete Guide](docs/full-guide_EN.md#local-webui-management-interface) and [LLM Configuration Guide](docs/LLM_CONFIG_GUIDE_EN.md).

## 🧩 Related Projects

> DSA focuses on daily analysis reports. The two projects below cover stock selection, strategy validation, and strategy evolution, suitable for extended use as needed. They are currently independently maintained; future focus will explore integration with DSA for candidate stock import, backtest validation, and report linking.

| Project | Positioning |
|---------|-------------|
| [AlphaSift](https://github.com/ZhuLinsen/alphasift) | Multi-factor stock selection and full-market scanning to extract candidate stocks from the stock pool |
| [AlphaEvo](https://github.com/ZhuLinsen/alphaevo) | Strategy backtesting and self-evolution to validate strategy rules and iteratively explore strategy parameters and combinations |

## 📬 Contact & Cooperation

<table>
  <tr>
    <td width="92" valign="top"><strong>Cooperation Email</strong></td>
    <td valign="top">
      <a href="mailto:zhuls345@gmail.com">zhuls345@gmail.com</a><br>
      Project inquiries, deployment support, feature extensions
    </td>
    <td align="center" rowspan="3" valign="middle" width="148">
      <a href="http://xhslink.com/m/tU520DWCKT" target="_blank"><img src="./docs/assets/xiaohongshu_tick.jpg" width="112" alt="Xiaohongshu QR"></a><br>
      <sub>Scan to follow on Xiaohongshu</sub>
    </td>
  </tr>
  <tr>
    <td width="92" valign="top"><strong>Xiaohongshu</strong></td>
    <td valign="top"><a href="http://xhslink.com/m/tU520DWCKT">Welcome to follow on Xiaohongshu</a></td>
  </tr>
  <tr>
    <td width="92" valign="top"><strong>Issue Feedback</strong></td>
    <td valign="top"><a href="https://github.com/ZhuLinsen/daily_stock_analysis/issues">Submit Issue</a></td>
  </tr>
</table>

## 📄 License

[MIT License](LICENSE) © 2026 ZhuLinsen

Please acknowledge this repository when doing secondary development or referencing. Thank you for supporting continuous project maintenance.

## ⚠️ Disclaimer

This project is for learning and research purposes only and does not constitute any investment advice. Stock markets carry risk; invest cautiously. The author is not responsible for any losses from using this project.

---
