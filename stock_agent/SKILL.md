---
description: '美股交易 Agent：支持 AI 自动分析回测、自定义策略回测、投资大师信号分析、模拟盘交易执行。当用户提到股票分析、交易策略、量化回测、股票代码（如 AAPL/NVDA/TSLA/AMD）、买卖决策、技术面/基本面分析、RSI/均线/止损、投资大师（巴菲特/芒格/Michael Burry）、买入、卖出、下单、持仓、平仓、账户、模拟盘、paper trading 等话题时触发。也适用于"帮我看看行情"、"回测一下"、"自动分析"、"对比几只股票"等日常表述。支持多股并行、双引擎分析、自动化与自定义两条回测路径、Alpaca 模拟盘交易。'
---

$ARGUMENTS

# 美股交易 Agent

串联四个远程 API 服务，支持六种使用流程。根据用户意图自动选择最合适的流程。

## 四个服务的定位

| 服务 | 路径 | 核心价值 |
|------|------|----------|
| **AI Hedge Fund** | /aihedgefund | 自动化全流程引擎：18 位投资大师量化信号 + LLM 每日动态决策回测 |
| **TradingAgents** | /tradingagents | 深度文本分析：技术/情绪/新闻/基本面四维报告 + 多空辩论，提供具体价格参数 |
| **回测引擎** | /ziplime | 自定义策略回测引擎：用户上传 Python 策略代码，固定规则执行 |
| **Alpaca Trading** | /alpacatrading | 模拟盘交易执行：下单/撤单/持仓/账户/行情 |

所有服务在 `https://finance.aifoundrys.com:7443`，Bearer Token 认证。完整 API 文档见 `stock-agent/references/api-details.md`。

---

## 子命令与流程路由

| 用法 | 流程 | 说明 |
|------|------|------|
| `/stock-agent AAPL` | 流程一 | 全自动：AI Hedge Fund 分析+回测 |
| `/stock-agent AAPL --custom` | 流程二 | 自定义策略：双引擎分析 → 设计策略 → 回测引擎验证 |
| `/stock-agent analyze AAPL` | 流程三 | 纯分析（默认双引擎） |
| `/stock-agent backtest AAPL` | 流程四 | 纯回测：用已有策略在回测引擎执行 |
| `/stock-agent compare AAPL` | 流程五 | 对比：自动化 vs 自定义策略表现 |
| `/stock-agent buy AAPL 1` | 流程六 | 市价买入 1 股 |
| `/stock-agent sell AAPL 1` | 流程六 | 市价卖出 1 股 |
| `/stock-agent limit buy AAPL 1 250` | 流程六 | 限价买入 |
| `/stock-agent portfolio` | — | 账户概览 + 全部持仓 |
| `/stock-agent positions` | — | 仅持仓列表 |
| `/stock-agent orders` | — | 订单列表 |
| `/stock-agent cancel <order_id>` | — | 取消订单 |
| `/stock-agent close AAPL` | — | 平仓指定标的 |
| `/stock-agent close all` | — | 全部平仓 |
| `/stock-agent quote AAPL` | — | 实时报价 |
| `/stock-agent status [task_id]` | — | 查询任务进度 |
| `/stock-agent bundles` | — | 查看回测引擎已导入数据 |
| `/stock-agent strategies` | — | 查看回测引擎已上传策略 |
| `/stock-agent agents` | — | 查看 AI Hedge Fund 可用投资大师 |

**路由规则**：
- 只有股票代码 → 流程一（全自动）
- 提到"自定义策略"、"我想设计"、"均线策略"等 → 流程二
- "分析"、"看看行情"、"什么信号" → 流程三
- "回测这个策略"、指定策略文件 → 流程四
- "对比"、"哪个好" → 流程五
- "买"、"卖"、"下单"、"交易"、"持仓"、"平仓"、"账户" → 交易命令（流程六）
- "行情"、"报价"、"现在多少钱" → 报价查询
- "按建议执行"、"执行分析结果"、"按建议买入" → 流程六（分析→交易联动）
- 多个代码 → 并行执行，每只股票独立走同一流程

---

## 参数

从 `$ARGUMENTS` 灵活提取：

| 参数 | 默认值 | 示例 |
|------|--------|------|
| 股票代码 | — (必须) | AAPL, NVDA |
| 回测起止日期 | 最近 3 个月 | 2025-12-01 ~ 2025-12-31 |
| 初始资金 | 100000 | 500000 |
| 投资大师 | 全部 | warren_buffett, michael_burry |
| 分析深度 | full | quick / full |

---

## 日期对齐规则

TradingAgents 的 `date` 参数决定 AI 基于哪天的数据分析。如果分析日期和回测日期不对齐，AI 看到的市场状态和回测行情完全是两回事，策略就会失效。

| 场景 | TradingAgents `date` | AI Hedge Fund `start/end_date` |
|------|---------------------|-------------------------------|
| 实时分析 | 不传（默认昨天） | 不传 |
| 回测 12 月 | `2025-11-28`（前一个交易日） | `start_date=2025-11-28` |
| 回测自定区间 | start_date 前一个交易日 | 与回测同区间或前一天 |

交易日计算：周一→上周五，周二~五→前一天，遇节假日再往前推。

---

## 流程一：全自动分析+回测

**场景**：用户只想知道"这只股票值不值得买"

```
AI Hedge Fund /api/v1/analyze → /hedge-fund/backtest → 报告
```

1. **分析**：调用 AI Hedge Fund `/api/v1/analyze`
   - 输入：tickers、analysts（可选，默认全部 18 位大师）、start_date、end_date
   - 输出：每只股票的 action + 各大师信号 + confidence
   - 轮询 `/api/v1/tasks/{task_id}`，向用户报告进度

2. **回测**：调用 AI Hedge Fund `/hedge-fund/backtest`（SSE 流式）
   - 需要构建 graph_nodes + graph_edges（选择哪些大师参与决策）
   - 每个交易日 LLM 动态决策，支持做多/做空/持有
   - 流式返回每日组合价值、交易记录
   - 输出：Sharpe、Sortino、最大回撤、多空比、最终组合

3. **报告**：保存到 `./reports/<TICKER>/<日期>/`
   - `01_analysis_hedgefund.md` — 投资大师信号汇总
   - `03_backtest_auto.md` — 自动化回测绩效
   - `04_final_report.md` — 综合报告

报告模板见 `stock-agent/references/report-templates.md`。

---

## 流程二：双引擎分析 + 自定义策略 + 回测

**场景**：用户想参考 AI 分析，自己设计/调整交易逻辑

```
TradingAgents + AI Hedge Fund 并行分析 → 用户设计策略 → 回测引擎验证
```

### 阶段 1：双引擎并行分析

**同时提交**两个分析请求（互相独立，并行执行）：

- **TradingAgents** `/api/v1/analyze`
  - 输出深度文本报告：支撑/阻力位、均线周期、RSI 数值、多空辩论
  - 为策略设计提供**具体价格参数**
  - 注意传入正确的 `date` 参数

- **AI Hedge Fund** `/api/v1/analyze`
  - 输出 18 位投资大师的量化信号：signal(bullish/bearish/neutral) + confidence
  - 提供**多空共识度**和风控信号

两个分析互相补充：TradingAgents 告诉你"在哪个价位买卖"，AI Hedge Fund 告诉你"多少大师看多/看空"。

保存文件：
- `01_analysis_full.md` — TradingAgents 完整报告
- `01_analysis_summary.md` — 双引擎摘要（含大师信号共识）

### 阶段 2：策略代码生成

Read `stock-agent/references/strategy-guide.md` 获取策略生成指南。

综合两个分析结果设计策略：
- TradingAgents 提供具体价位："200 SMA $189，RSI 37，布林带下轨 $195"
- AI Hedge Fund 提供共识："巴菲特 neutral，技术面 bearish confidence 24"
- → 策略：$195 入场 20%（大师共识偏中性所以轻仓），$189 止损

**输入来源（按优先级）**：
1. 阶段 1 的双引擎分析结果
2. 本地已有报告 `./reports/{TICKER}/`
3. 用户自己描述交易思路

保存 `02_strategy.py` + `02_strategy_notes.md`。用户可反复修改直到满意。

### 阶段 3：回测引擎验证

1. 策略代码写入 `/tmp/strategy_<ticker>.py`，上传到回测引擎
2. 检查数据（GET /bundles），不足时自动 ingest
3. 提交回测，轮询结果
4. 保存 `03_backtest.md`

### 阶段 4：综合报告

汇总双引擎分析 + 策略设计 + 回测绩效，保存 `04_final_report.md`。

---

## 流程三：纯分析（不回测）

**场景**：用户只想了解某只股票当前状态

默认使用双引擎（并行），用户也可指定单引擎：

| 指令 | 分析引擎 |
|------|----------|
| `analyze AAPL` | 双引擎（默认） |
| `analyze AAPL --deep` | 仅 TradingAgents（深度文本） |
| `analyze AAPL --masters` | 仅 AI Hedge Fund（大师信号） |

双引擎结果交叉验证：
- TradingAgents 说 SELL + 巴菲特说 bearish → 强看空
- TradingAgents 说 BUY + Michael Burry 说 bearish → 分歧，需谨慎

---

## 流程四：纯回测（已有策略）

**场景**：用户有自己的策略代码

```
用户策略 .py → 回测引擎上传 + ingest + 回测 → 绩效报告
```

- 用户指定策略文件 → 直接用
- 未指定 → 检查 `./reports/{TICKER}/02_strategy.py` 或回测引擎已有策略

---

## 流程五：对比回测

**场景**：对比自动化 vs 自定义策略

```
同一只股票/同一区间：
  路径 A：AI Hedge Fund /backtest → 自动化绩效
  路径 B：回测引擎 /backtest → 自定义策略绩效
  → 对比报告
```

输出对比表：

```
| 维度 | AI 自动 | 自定义策略 |
|------|---------|-----------|
| 收益率 | +5.2% | +3.1% |
| Sharpe | 1.8 | 1.2 |
| 最大回撤 | -3.1% | -5.5% |
| 交易次数 | 15 | 8 |
```

---

## 流程六：分析 → 模拟盘交易

**场景**：用户想基于 AI 分析结果执行模拟盘交易，或直接下单交易

```
阶段 1: AI 分析（复用流程一/三逻辑）
  ├─ AI Hedge Fund /api/v1/analyze → decisions {action, quantity, confidence}
  └─ TradingAgents /api/v1/analyze → final_trade_decision (可选)

阶段 2: 交易前检查（并行调用 Alpaca API）
  ├─ GET /account → buying_power, equity
  ├─ GET /positions → 当前持仓
  └─ GET /quotes/{symbol} → 最新报价

阶段 3: 生成交易建议并展示给用户
  - 标的、方向、数量、预估金额
  - 账户购买力
  - 双引擎一致性评估

阶段 4: 用户确认后下单
  - POST /orders → 返回订单状态
  - 保存 reports/{TICKER}/{日期}/05_trade_execution.md
```

直接下单命令（如 `/stock-agent buy AAPL 1`）跳过阶段 1，直接从阶段 2 开始。

### 交易安全规则

- **所有下单必须二次确认**：展示订单详情（标的、方向、数量、预估金额），等用户说"确认"/"是"/"执行"
- **双引擎方向冲突时强制手动确认**：展示分歧详情，明确告知风险
- **批量操作（close all）额外警告**：列出所有将被平仓的持仓，需用户明确确认
- AI 决策→订单方向映射：

| AI action | 操作 |
|-----------|------|
| buy | 买入 |
| sell | 查持仓→有则平仓，无则提示 |
| short | 不支持（Alpaca Paper 做空限制），提示用户 |
| cover | 不支持，提示 |
| hold | 不下单，仅展示分析 |

### 仓位计算建议

当基于 AI 分析结果自动计算下单数量时，参考以下规则：

| confidence | 建议投入比例（占购买力） |
|------------|------------------------|
| >= 80% | 20% |
| 60-79% | 10% |
| 40-59% | 5% |
| < 40% | 不建议交易 |

---

## 多股票并行

所有流程都支持多股票，每只独立执行、容错隔离。

进度汇报格式：
```
| 股票 | 状态 | 进度 | 当前阶段 |
|------|------|------|----------|
| AAPL | 运行中 | 60% | 基本面分析 |
| NVDA | 已完成 | 100% | BUY |
```

多股票完成后额外生成 `./reports/_comparison/<日期>/comparison.md`（对比排名+组合建议）。

---

## 文件保存

所有报告保存到 `./reports/<TICKER>/<日期>/`：

| 文件 | 流程一 | 流程二 | 流程六 | 说明 |
|------|:---:|:---:|:---:|------|
| `01_analysis_full.md` | — | TradingAgents | TradingAgents(可选) | 深度文本报告 |
| `01_analysis_hedgefund.md` | AI Hedge Fund | AI Hedge Fund | AI Hedge Fund(可选) | 大师信号汇总 |
| `01_analysis_summary.md` | 信号摘要 | 双引擎摘要 | 双引擎摘要(可选) | 合并摘要 |
| `02_strategy.py` | — | 策略代码 | — | 仅流程二 |
| `02_strategy_notes.md` | — | 策略说明 | — | 仅流程二 |
| `03_backtest_auto.md` | AI Hedge Fund | — | — | 自动化回测 |
| `03_backtest.md` | — | 回测引擎 | — | 自定义策略回测 |
| `04_final_report.md` | 综合报告 | 综合报告 | — | 最终汇总 |
| `05_trade_execution.md` | — | — | 交易记录 | 交易执行记录 |

---

## 错误处理

| 情况 | 处理方式 |
|------|----------|
| API 不可达 | curl health 端点检测，报告哪个服务不可用 |
| 分析失败/超时 | 重试一次；仍失败则跳过，输出已完成部分 |
| 分析被限速 | 等待 30 秒后重新提交 |
| 双引擎中一个失败 | 另一个结果仍可用，降级为单引擎 |
| 策略上传失败 | 检查 .py 后缀和 ≤100KB 大小 |
| 回测 0% 收益 | 入场条件过严——参考策略指南调整 |
| SSE 流式中断 | AI Hedge Fund backtest 用 SSE，网络断开需重试 |
| 下单被拒 | 检查购买力、市场是否开盘、标的是否可交易 |
| 订单未成交 | 限价单可能挂起，提示用户查看 orders 状态 |
| 购买力不足 | 展示当前购买力和订单所需金额，建议调整数量 |

---

## 注意事项

- 策略文件先写到 `/tmp/` 再上传
- 回测日期必须在 ingest 数据范围内
- TradingAgents 分析 3-10 分钟，AI Hedge Fund 分析 ~30 秒
- AI Hedge Fund backtest 每个交易日调用 LLM，耗时随区间长度增长
- 用户随时可查看已保存的详细报告

---

## Reference 文件索引

| 文件 | 内容 | 何时读取 |
|------|------|----------|
| `stock-agent/references/api-details.md` | 四个服务的完整 API 文档、Token、curl 示例 | 调用任何 API 前 |
| `stock-agent/references/strategy-guide.md` | 策略方向映射、代码框架、生成原则 | 流程二阶段 2 生成策略时 |
| `stock-agent/references/report-templates.md` | 所有报告的 Markdown 模板 | 保存报告时 |
