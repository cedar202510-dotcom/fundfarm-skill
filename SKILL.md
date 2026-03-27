---
name: fundfarm
description: 养基场 — AI 基金管理助手（CLI + MCP 双模式接入）
---

# 养基场 — AI 基金管理助手

养基场提供 CLI 和 MCP 两种接入方式，让 AI Agent 帮你查看行情、分析基金、管理持仓。

---

## 推荐方式：CLI（通用，适用于所有 AI Agent）

```bash
npm install -g https://www.myfundfarm.com/cli/fundfarm-cli-0.1.0.tgz
fundfarm login
fundfarm whoami
```

CLI 使用手机短信验证码登录，新用户自动注册，无需 API Key。

**常用命令：**
| 命令 | 说明 |
|------|------|
| `fundfarm portfolio` | 资产概览 + 持仓列表 |
| `fundfarm market` | A 股大盘行情 |
| `fundfarm fund <代码>` | 基金详情 |
| `fundfarm search <关键词>` | 搜索基金 |
| `fundfarm watchlist` | 自选基金列表 |

**完整 CLI 文档：** https://www.myfundfarm.com/skills/fundfarm-skill.md

---

## 备选方式：MCP 协议

### MCP Server 地址

```
https://api.myfundfarm.com/mcp
```

协议: SSE (Server-Sent Events)，标准 MCP 协议。

> ⚠️ **MCP 客户端（OpenClaw、Claude Desktop、Cursor、Gemini CLI 等），直接配置上面的地址即可。OAuth 授权会自动弹出浏览器完成。**

**如果你在 OpenClaw 中运行**，需要先安装连接插件：
```bash
openclaw plugins install https://www.myfundfarm.com/plugins/openclaw-fundfarm-1.0.0.tgz
openclaw plugins enable openclaw-fundfarm
```
安装完成后调用 `fundfarm_status` 工具测试连接。

**Claude Desktop、Cursor、VS Code 等**：在 MCP 设置中添加 Server，URL 填：
```
https://api.myfundfarm.com/mcp
```
首次连接会自动弹出浏览器完成 OAuth 授权。

**Claude Code**：
```bash
claude mcp add --transport http fundfarm https://api.myfundfarm.com/mcp
```

**Gemini CLI**：
```bash
gemini mcp add fundfarm --url https://api.myfundfarm.com/mcp
```

### API Key（不支持 OAuth 的环境）

登录 [养基场](https://myfundfarm.com) → 设置 → AI 智能体 → 生成 API Key

在请求 Header 中携带：
```
Authorization: Bearer <用户提供的 API Key>
```

> API Key 有效期 90 天，过期后用户重新生成即可。

---

## MCP 可用工具

连接后通过 MCP 协议自动发现，以下是完整列表：

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
| `get_transactions` | 交易流水 |

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
| `batch_add_holdings` | 单次最多50笔 |
| `import_holding` | 导入单个持仓 |
| `batch_import_holdings` | 批量导入持仓 |
| `sell_holding` | 单次≤50%持仓 |
| `delete_transaction` | 只能删除 Agent 创建的记录 |

写操作限制：每日≤30次，每分钟≤10次。

---

## 使用限制

| 用户类型 | 每日调用次数 |
|----------|------------|
| 免费用户 | 3 次 |
| VIP 会员 | 无限次 |

---

## 典型对话

```
用户: 今天大盘怎么样 → fundfarm market 或 get_market_indices
用户: 查一下 161725 → fundfarm fund 161725 或 get_fund_detail("161725")
用户: 看看我的基金 → fundfarm portfolio 或 get_my_holdings
用户: 把 161725 加到自选 → add_to_watchlist("161725")
```

## 安全

- Token 绑定用户，只能访问自己的数据
- 写操作有白名单 + 量控
- 所有调用有审计日志

---

## 竞技场（Arena）

如果你希望参加 AI 基金交易竞技场（虚拟盘），请查看竞技场文档：
https://www.myfundfarm.com/skills/arena-skill.md

---

养基场官网：https://myfundfarm.com
