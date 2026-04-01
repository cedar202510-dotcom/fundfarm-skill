---
name: fundfarm
description: 养基场 — AI 基金管理助手（CLI + MCP 双模式接入）
---

# 养基场 — AI 基金管理助手

养基场提供 CLI 和 MCP 两种接入方式，让 AI Agent 帮你查看行情、分析基金、管理持仓。

---

## 推荐方式：CLI（通用，适用于所有 AI Agent）

```bash
npm install -g https://www.myfundfarm.com/cli/fundfarm-cli-0.1.1.tgz
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
| `fundfarm watchlist add <代码> --yes` | 添加自选 |
| `fundfarm watchlist remove <代码> --yes` | 移除自选 |
| `fundfarm ranking` | 基金涨跌幅排行榜 |
| `fundfarm trade buy <代码> --amount <金额> --yes` | 买入基金 |
| `fundfarm trade sell <代码> --shares <份额> --yes` | 卖出基金 |
| `fundfarm trade cancel <交易ID> --yes` | 撤销交易（回滚持仓） |
| `fundfarm ai report` | 获取最新 AI 报告 |
| `fundfarm ai analyze --yes` | 运行 AI 深度分析 |

**完整 CLI 文档：** https://www.myfundfarm.com/skills/fundfarm-skill.md

---

## 写操作安全机制

所有写操作（trade、watchlist add/remove、ai analyze）具有**多层安全保护**：

### 后端安全（服务端强制执行，无法绕过）
- **JWT 认证 + 用户隔离**：所有查询限定 `auth.user.username`
- **操作白名单**：仅允许 8 个特定写操作
- **频率限制**：每日 ≤30 次，每分钟 ≤10 次
- **卖出熔断**：单次卖出 ≤50% 持仓
- **买入上限**：单笔 ≤100 万元
- **NAV 范围校验**：0 < nav < 1000
- **审计日志**：每次调用记录用户、操作、客户端、IP

### CLI 安全（客户端纵深防御）
- **交互式确认**：默认弹出 `⚠️ ... 确认执行此操作？(y/N)`
- **`--yes` 标志**：AI Agent 可跳过确认，后端安全网仍然生效
- **`--dry-run` 标志**：仅预览操作，不发送任何请求
- **输入校验**：基金代码格式、金额范围、UUID 格式在请求前校验

### 撤销安全
- `trade cancel` **只能**删除 Agent 创建的交易（通过 note 中包含"通过 MCP"来判断）
- 用户手动录入的交易记录**绝不可**通过 CLI 删除
- 撤销时自动回滚持仓份额和成本

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
| 免费用户 | 5 次 |
| VIP 会员 | 无限次 |

---

## 典型对话

```
用户: 今天大盘怎么样     → fundfarm market 或 get_market_indices
用户: 查一下 161725      → fundfarm fund 161725 或 get_fund_detail("161725")
用户: 看看我的基金        → fundfarm portfolio 或 get_my_holdings
用户: 把 161725 加到自选  → fundfarm watchlist add 161725 --yes 或 add_to_watchlist("161725")
用户: 帮我买入 2000 元    → fundfarm trade buy 161725 --amount 2000 --yes 或 add_holding("161725", 2000)
用户: 卖出 500 份         → fundfarm trade sell 161725 --shares 500 --yes 或 sell_holding("161725", 500)
用户: 撤销刚才的交易      → fundfarm trade cancel <tx_id> --yes 或 delete_transaction("<tx_id>")
用户: 分析一下我的持仓    → fundfarm ai analyze --yes 或 run_ai_analysis
```

## 安全

- Token 绑定用户，只能访问自己的数据
- 写操作有白名单 + 量控 + 卖出熔断
- 撤销操作只能删除 Agent 创建的记录
- 所有调用有审计日志

---

## 竞技场（Arena）

如果你希望参加 AI 基金交易竞技场（虚拟盘），请查看竞技场文档：
https://www.myfundfarm.com/skills/arena-skill.md

---

养基场官网：https://myfundfarm.com
