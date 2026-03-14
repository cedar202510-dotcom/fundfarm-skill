---
description: 如何将养基场 MCP Server 接入 AI 智能体（OpenClaw、Claude Desktop、Cursor 等）
---

# 养基场 MCP Server 接入指南

## 快速连接

### MCP Server 地址

```
https://funds-incubator.up.railway.app/mcp
```

协议: SSE (Server-Sent Events)，标准 MCP 协议。

---

## 认证方式（二选一）

### 方式一: OAuth2 授权（推荐）

Agent 自主发起 OAuth2 授权流程，用户只需在弹出的页面上点击"授权"。

**选择你的 client_id：**

| client_id | 适用 Agent |
|---|---|
| `claude-desktop` | Claude Desktop |
| `claude-code` | Claude Code (CLI) |
| `chatgpt-desktop` | ChatGPT Desktop |
| `cursor-ide` | Cursor |
| `windsurf-ide` | Windsurf |
| `gemini-cli` | Gemini CLI |
| `openclaw` | OpenClaw |
| `generic-mcp` | 其他未列出的 Agent |

> 所有 client_id 均为公开标识符（无 client_secret），不含任何密钥。

**1. 获取服务端元数据：**
```
GET https://funds-incubator.up.railway.app/api/v1/oauth/.well-known/oauth-authorization-server
```

返回的 JSON 包含所有端点地址（authorization_endpoint、token_endpoint 等）。

**2. 引导用户授权：**

将用户重定向到授权页面（将 `<YOUR_CLIENT_ID>` 替换为上表中对应的值）:
```
https://myfundfarm.com/#/oauth/authorize
  ?client_id=<YOUR_CLIENT_ID>
  &redirect_uri=你的回调地址
  &scope=fund:read portfolio:read portfolio:write ai:read ai:execute
  &response_type=code
  &code_challenge=<PKCE S256 challenge>
  &code_challenge_method=S256
```

用户会看到一个授权确认页面，显示应用名和请求的权限列表。

**3. 用授权码换取 Token：**
```http
POST /api/v1/oauth/token
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code
&code=<用户授权后返回的code>
&client_id=<YOUR_CLIENT_ID>
&redirect_uri=你的回调地址
&code_verifier=<PKCE verifier>
```

**4. Token 续期：**

Access Token 有效期 1 小时，过期后用 Refresh Token 续期（30天有效）：
```http
POST /api/v1/oauth/token
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token
&refresh_token=<refresh_token>
&client_id=<YOUR_CLIENT_ID>
```

### 方式二: API Key（快速接入）

用户在养基场网站生成一个专属 API Key，复制粘贴给 Agent 即可。

**用户操作：** 登录养基场 → 设置 → AI 智能体 → 点击「生成 API Key」→ 复制 Key → 粘贴给 Agent

**Agent 使用方式：** 在所有请求 Header 中携带：
```
Authorization: Bearer <用户提供的 API Key>
```

> API Key 有效期 90 天，过期后用户重新生成即可。每次生成新 Key 旧的自动失效。

---

## 可用工具（24个）

连接后通过 MCP 协议自动发现，以下是完整列表供参考：

### 📊 基金查询
| 工具 | 说明 | 关键参数 |
|------|------|---------|
| `search_funds` | 搜索基金 | `keyword`, `limit` |
| `get_fund_detail` | 基金详情 | `fund_code` |
| `get_fund_nav_history` | 历史净值 | `fund_code`, `days` |
| `get_fund_top_holdings` | 重仓股 | `fund_code` |
| `get_fund_sector_allocation` | 行业配置 | `fund_code` |
| `get_fund_estimate` | 盘中估值 | `fund_code` |
| `get_fund_ranking` | 排行榜 | `sort_by`, `order`, `limit` |

### 📈 市场数据
| 工具 | 说明 |
|------|------|
| `get_market_indices` | 大盘指数行情 |
| `get_sector_ranking` | 板块涨跌排名 |
| `get_market_status` | 交易状态 |

### 👤 用户持仓
| 工具 | 说明 |
|------|------|
| `get_my_holdings` | 持仓及收益 |
| `get_my_watchlist` | 自选基金 |
| `get_portfolio_summary` | 组合概览 |
| `get_my_sectors` | 板块分布 |
| `get_my_nests` | 鸡窝列表 |
| `get_transactions` | 交易流水（可按基金代码/鸡窝筛选） |

### 🤖 AI 分析
| 工具 | 说明 |
|------|------|
| `get_latest_ai_report` | 最新分析报告 |
| `run_ai_analysis` | 运行 AI 分析（消耗额度） |

### ✏️ 写操作
| 工具 | 限制 |
|------|------|
| `add_to_watchlist` | |
| `remove_from_watchlist` | |
| `add_holding` | 单笔买入 |
| `batch_add_holdings` | 单次最多50笔，仅算1次写操作 |
| `sell_holding` | 单次≤50%持仓 |
| `delete_transaction` | 只能删除 Agent 创建的记录，并自动回滚持仓 |

写操作限制：每日≤30次，每分钟≤10次。

---

## 可用 Scopes

| Scope | 说明 |
|-------|------|
| `fund:read` | 基金信息和市场数据（10个工具） |
| `portfolio:read` | 持仓、自选、交易流水（6个工具） |
| `portfolio:write` | 管理持仓/自选、撤销交易（6个工具） |
| `ai:read` | AI 分析报告（1个工具） |
| `ai:execute` | 运行 AI 分析（1个工具） |

---

## 典型对话

```
用户: 今天大盘怎么样 → get_market_indices
用户: 查一下 161725 → get_fund_detail("161725")
用户: 看看我的基金 → get_my_holdings
用户: 分析一下我的持仓 → run_ai_analysis 或 get_my_holdings + 自行分析
用户: 把 161725 加到自选 → add_to_watchlist("161725")
```

## 安全

- Token 绑定用户，只能访问自己的数据
- 写操作有白名单 + 量控
- 所有调用有审计日志
