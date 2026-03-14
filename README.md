# 🐔 FundFarm Skill (养基场)

> Connect your AI agent to **China's mutual fund market** — real-time data on 15,000+ domestic funds (A-share), portfolio tracking, and AI-powered investment analysis via MCP.

## Install

```bash
npx skills add cedar202510-dotcom/fundfarm-skill
```

## What Can Your Agent Do?

With this skill, your AI agent gets access to **24 real-time tools** via MCP:

### 📊 Fund Intelligence
- **Search** 15,000+ A-share mutual funds by name or code
- **Deep dive** into any fund: NAV history, top 10 holdings, sector allocation, intraday estimates
- **Rankings** by daily return, 1M/3M/6M/1Y performance

### 📈 Live Market Data
- **Major indices**: SSE Composite, CSI 300, ChiNext, CSI 500
- **Sector rankings**: Industry performance with leading stocks
- **Market status**: Real-time open/close/holiday detection

### 👤 Portfolio Management
- **View holdings** with real-time P&L calculation
- **Track watchlists** across multiple portfolio groups ("Nests")
- **Transaction history** with filtering by fund or nest
- **Buy & sell** with safety guardrails (rate limits, position limits)

### 🤖 AI Analysis
- **Get reports**: Retrieve the latest AI-generated portfolio diagnosis
- **Run analysis**: Trigger a fresh AI deep dive into user's holdings

## MCP Server

```
https://funds-incubator.up.railway.app/mcp
```

Protocol: **SSE** (Server-Sent Events) — standard MCP transport.

## Authentication

Users need a [FundFarm](https://myfundfarm.com) account. Two methods:

| Method | Best For | Setup |
|--------|----------|-------|
| **OAuth2 + PKCE** | Production agents | Agent handles the flow automatically |
| **API Key** | Quick testing | User generates key on website, pastes to agent |

## Write Operations

Write tools (buy, sell, watchlist) are rate-limited and designed for safe autonomous use. Agent-created transactions are reversible.

## Example Conversations

```
User: How's the market today?        → get_market_indices
User: Look up fund 161725            → get_fund_detail("161725")
User: Show my portfolio              → get_my_holdings
User: Analyze my holdings            → run_ai_analysis
User: Add 161725 to my watchlist     → add_to_watchlist("161725")
User: What are the top funds today?  → get_fund_ranking(sort_by="daily_return")
```

## Available Scopes

| Scope | Tools | Description |
|-------|:-----:|-------------|
| `fund:read` | 10 | Fund info and market data |
| `portfolio:read` | 6 | Holdings, watchlist, transactions |
| `portfolio:write` | 6 | Buy, sell, manage watchlist |
| `ai:read` | 1 | AI analysis reports |
| `ai:execute` | 1 | Run AI analysis |

## Learn More

- 🌐 **Website**: [myfundfarm.com](https://myfundfarm.com)
- 📖 **Full API Reference**: See [SKILL.md](.agents/skills/mcp-integration/SKILL.md)

## License

MIT
