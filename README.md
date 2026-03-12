# FundFarm Skill (养基场)

> Connect your AI agent to FundFarm — A-share mutual fund data, portfolio tracking, and AI-powered analysis.

## Install

```bash
npx skills add cedar202510-dotcom/fundfarm-skill
```

## What is FundFarm?

[FundFarm (养基场)](https://myfundfarm.com) is a comprehensive Chinese A-share mutual fund management platform. It provides:

- 📊 **Fund Data**: Search, details, NAV history, top holdings, sector allocation
- 📈 **Market Data**: Major indices, sector rankings, market status
- 👤 **Portfolio Management**: Track holdings, watchlists, P&L
- 🤖 **AI Analysis**: AI-powered portfolio diagnosis and investment insights

## MCP Server

This skill connects your agent to FundFarm's MCP server, giving access to **22 tools** for fund data querying, portfolio management, and AI analysis.

```
MCP Server URL: https://funds-incubator.up.railway.app/mcp
```

## Authentication

Users need a FundFarm account. Two auth methods:

1. **OAuth2** — Agent handles the flow, user clicks "Authorize"
2. **API Key** — User generates key on FundFarm website, pastes to agent

See [SKILL.md](.agents/skills/mcp-integration/SKILL.md) for full integration details.

## License

MIT
