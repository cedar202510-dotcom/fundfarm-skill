---
name: fundfarm-mcp
description: Connect to FundFarm (养基场) - A-share mutual fund data, portfolio tracking, and AI analysis via MCP
---

# FundFarm MCP Integration Skill

Connect your AI agent to [FundFarm (养基场)](https://myfundfarm.com) — a comprehensive A-share mutual fund platform with real-time data, portfolio management, and AI-powered analysis.

## Quick Connect

### MCP Server URL

```
https://funds-incubator.up.railway.app/mcp
```

Protocol: SSE (Server-Sent Events), standard MCP protocol.

---

## Authentication (choose one)

### Option 1: OAuth2 Authorization (Recommended)

Agent initiates the OAuth2 flow automatically. User just clicks "Authorize".

**1. Get server metadata:**
```
GET https://funds-incubator.up.railway.app/api/v1/oauth/.well-known/oauth-authorization-server
```

Returns JSON with all endpoint URLs (authorization_endpoint, token_endpoint, etc).

**2. Redirect user to authorize:**
```
https://myfundfarm.com/#/oauth/authorize
  ?client_id=YOUR_CLIENT_ID
  &redirect_uri=YOUR_REDIRECT_URI
  &scope=fund:read portfolio:read portfolio:write ai:read ai:execute
  &response_type=code
  &code_challenge=<PKCE S256 challenge>
  &code_challenge_method=S256
```

User sees an authorization page showing app name and requested permissions.

**3. Exchange code for token:**
```http
POST /api/v1/oauth/token
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code
&code=<authorization_code>
&client_id=YOUR_CLIENT_ID
&redirect_uri=YOUR_REDIRECT_URI
&code_verifier=<PKCE verifier>
```

**4. Token refresh:**

Access Token expires in 1 hour. Use Refresh Token (30-day validity) to renew:
```http
POST /api/v1/oauth/token
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token
&refresh_token=<refresh_token>
&client_id=YOUR_CLIENT_ID
```

### Option 2: API Key (Quick Setup)

User generates an API Key on FundFarm website and provides it to the agent.

**User steps:** Log in to FundFarm → Settings → AI Agent → Click "Generate API Key" → Copy → Paste to agent

**Agent usage:** Include in all request headers:
```
Authorization: Bearer <user's API Key>
```

> API Key is valid for 90 days. Generating a new key automatically revokes the old one.

---

## Available Tools (24)

Tools are auto-discovered via MCP protocol after connecting.

### 📊 Fund Query
| Tool | Description | Key Parameters |
|------|-------------|---------------|
| `search_funds` | Search funds by name/code | `keyword`, `limit` |
| `get_fund_detail` | Fund details | `fund_code` |
| `get_fund_nav_history` | Historical NAV | `fund_code`, `days` |
| `get_fund_top_holdings` | Top 10 holdings | `fund_code` |
| `get_fund_sector_allocation` | Sector allocation | `fund_code` |
| `get_fund_estimate` | Intraday estimate | `fund_code` |
| `get_fund_ranking` | Fund rankings | `sort_by`, `order`, `limit` |

### 📈 Market Data
| Tool | Description |
|------|-------------|
| `get_market_indices` | Major market indices (SSE, SZSE, ChiNext) |
| `get_sector_ranking` | Sector performance ranking |
| `get_market_status` | Market open/close status |

### 👤 User Portfolio
| Tool | Description |
|------|-------------|
| `get_my_holdings` | Holdings with P&L |
| `get_my_watchlist` | Watchlist |
| `get_portfolio_summary` | Portfolio overview (total invested/returns) |
| `get_my_sectors` | Holdings sector distribution |
| `get_my_nests` | Portfolio groups (nests) |
| `get_transactions` | Transaction history (filterable by fund code / nest) |

### 🤖 AI Analysis
| Tool | Description |
|------|-------------|
| `get_latest_ai_report` | Latest AI analysis report |
| `run_ai_analysis` | Run AI portfolio analysis (consumes quota) |

### ✏️ Write Operations
| Tool | Limits |
|------|--------|
| `add_to_watchlist` | |
| `remove_from_watchlist` | |
| `add_holding` | Single purchase |
| `batch_add_holdings` | Up to 50 items, counts as 1 write operation |
| `sell_holding` | Max 50% of position per request |
| `delete_transaction` | Agent-created records only; automatically reverses position changes |

Write operation limits: ≤30/day, ≤10/minute.

---

## Scopes

| Scope | Description |
|-------|-------------|
| `fund:read` | Fund info and market data (10 tools) |
| `portfolio:read` | Holdings, watchlist, and transaction history (6 tools) |
| `portfolio:write` | Manage holdings/watchlist, delete transactions (6 tools) |
| `ai:read` | AI analysis reports (1 tool) |
| `ai:execute` | Run AI analysis (1 tool) |

---

## Example Conversations

```
User: How's the market today? → get_market_indices
User: Look up fund 161725 → get_fund_detail("161725")
User: Show my portfolio → get_my_holdings
User: Analyze my holdings → run_ai_analysis or get_my_holdings + self-analyze
User: Add 161725 to watchlist → add_to_watchlist("161725")
```

## Important Notes

- **Registration required**: Users must register at https://myfundfarm.com before using any tools
- **Data isolation**: Token is bound to user identity, can only access own data
- **Membership limits apply**: Free users have limits on fund count, AI analysis quota, etc.
- **Write operations**: Whitelisted + rate-limited for safety
- **Audit logging**: All MCP calls are logged for security
