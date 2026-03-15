# API 服务详细文档

## 服务总览

| 服务 | 地址 | Token | 用途 |
|------|------|-------|------|
| AI Hedge Fund | `http://172.16.13.58:8082` | `xG8XyygVqe9OtV5WcQH8phECIQ8wiIZrGaziiuOJtJQ` | 投资大师分析 + 自动化回测 |
| TradingAgents | `http://172.16.13.58:8080` | `ITBVOCmV4m2Wm1qHL-yRsxE4D3QVMQXPWLWxSIdBHKo` | 深度文本分析 |
| 回测引擎 | `http://172.16.13.58:8090` | `NoHKWxHvaFKdsNutAKGjJWD_RmDPipxurgT_U9H_Oh0` | 自定义策略回测 |
| Alpaca Trading | `http://172.16.13.58:8083` | `JISCaUdXQEyzIQmVwpe1HEh++6CFDVm+Aj1X9snsj4I=` | 模拟盘交易执行 |

所有 API 通过 HTTP 直接调用，地址为 `172.16.13.58`。

---

## AI Hedge Fund API（端口 8082）

### GET /hedge-fund/agents — 查看可用投资大师

```bash
curl -s http://172.16.13.58:8082/hedge-fund/agents \
  -H "Authorization: Bearer xG8XyygVqe9OtV5WcQH8phECIQ8wiIZrGaziiuOJtJQ"
```

返回 18 位可用 agent：

**投资大师**：aswath_damodaran, ben_graham, bill_ackman, cathie_wood, charlie_munger, michael_burry, mohnish_pabrai, peter_lynch, phil_fisher, rakesh_jhunjhunwala, stanley_druckenmiller, warren_buffett

**专业分析师**：technical_analyst, fundamentals_analyst, growth_analyst, news_sentiment_analyst, sentiment_analyst, valuation_analyst

### POST /api/v1/analyze — 简化分析（异步）

不需要构建 graph，接口简洁：

```bash
curl -s -X POST http://172.16.13.58:8082/api/v1/analyze \
  -H "Authorization: Bearer xG8XyygVqe9OtV5WcQH8phECIQ8wiIZrGaziiuOJtJQ" \
  -H "Content-Type: application/json" \
  -d '{
    "tickers": ["AAPL", "NVDA"],
    "analysts": ["warren_buffett", "michael_burry", "technical_analyst"],
    "show_reasoning": true,
    "start_date": "2025-11-28",
    "end_date": "2025-12-31"
  }'
```

**参数**：
- `tickers`（必填）：支持多只股票
- `analysts`：可选，省略则用全部 18 位
- `show_reasoning`：是否返回详细推理过程
- `start_date`/`end_date`：分析的时间范围
- `model_name`：默认 `gpt-4.1`
- `model_provider`：默认 `OpenAI`

**返回**：`{"task_id": "xxx", "status": "running"}`

### GET /api/v1/tasks/{task_id} — 轮询分析结果

```bash
curl -s http://172.16.13.58:8082/api/v1/tasks/<task_id> \
  -H "Authorization: Bearer xG8XyygVqe9OtV5WcQH8phECIQ8wiIZrGaziiuOJtJQ"
```

**返回数据结构**（completed 时）：

```json
{
  "task_id": "xxx",
  "status": "completed",
  "result": {
    "decisions": {
      "AAPL": {
        "action": "buy|sell|short|cover|hold",
        "quantity": 100,
        "confidence": 75,
        "reasoning": "..."
      }
    },
    "analyst_signals": {
      "warren_buffett_agent": {
        "AAPL": {
          "signal": "bullish|bearish|neutral",
          "confidence": 65,
          "reasoning": "..."
        }
      },
      "technical_analyst_agent": {
        "AAPL": {
          "signal": "neutral",
          "confidence": 14,
          "reasoning": {
            "trend_following": {"signal": "bearish", "confidence": 24, "metrics": {"adx": 24.39}},
            "mean_reversion": {"signal": "neutral", "confidence": 50, "metrics": {"rsi_14": 45.44, "z_score": -0.36}},
            "momentum": {"signal": "neutral", "confidence": 50, "metrics": {"momentum_1m": -0.06}},
            "volatility": {"signal": "neutral", "confidence": 50, "metrics": {"historical_volatility": 0.28}}
          }
        }
      }
    }
  }
}
```

### POST /hedge-fund/backtest — 自动化回测（SSE 流式）

需要构建 graph_nodes 和 graph_edges 来定义 agent 图：

```bash
curl -s -X POST http://172.16.13.58:8082/hedge-fund/backtest \
  -H "Authorization: Bearer xG8XyygVqe9OtV5WcQH8phECIQ8wiIZrGaziiuOJtJQ" \
  -H "Content-Type: application/json" \
  -d '{
    "tickers": ["AAPL"],
    "start_date": "2025-12-01",
    "end_date": "2025-12-31",
    "initial_capital": 100000,
    "graph_nodes": [
      {"id": "warren_buffett"},
      {"id": "technical_analyst"},
      {"id": "fundamentals_analyst"}
    ],
    "graph_edges": [
      {"id": "e1", "source": "warren_buffett", "target": "risk_manager"},
      {"id": "e2", "source": "technical_analyst", "target": "risk_manager"},
      {"id": "e3", "source": "fundamentals_analyst", "target": "risk_manager"}
    ]
  }'
```

**graph 构建规则**：
- `graph_nodes`：选择哪些 agent 参与分析，每个 node 只需 `id`（agent key）
- `graph_edges`：每个分析 agent 连接到 `risk_manager`（系统自动添加 risk_manager → portfolio_manager → END）
- 所有选中的 agent 并行分析，结果汇聚到风控，再由 portfolio_manager（LLM）做最终决策

**返回**：SSE 流式事件，每个交易日一条更新：

```
data: {"type": "progress", "current_date": "2025-12-01", "step": 1, "total_steps": 22}
data: {"type": "daily_result", "date": "2025-12-01", "portfolio_value": 100150.50, "decisions": {...}, "trades": [...]}
...
data: {"type": "final", "performance_metrics": {"sharpe_ratio": 1.2, "sortino_ratio": 1.5, "max_drawdown": -0.03, ...}, "final_portfolio": {...}}
```

**性能指标**（final 事件）：
- `sharpe_ratio`：年化 Sharpe（无风险利率 4.34%）
- `sortino_ratio`：只计下行风险
- `max_drawdown`：最大回撤
- `long_short_ratio`：多空比
- `total_exposure`：总敞口
- `net_exposure`：净敞口

### POST /hedge-fund/run — 单次决策（不回测）

与 backtest 相同的参数，但只运行一次 agent 图获取决策，不执行交易。

### GET /api/v1/analysts — 列出分析师

```bash
curl -s http://172.16.13.58:8082/api/v1/analysts \
  -H "Authorization: Bearer xG8XyygVqe9OtV5WcQH8phECIQ8wiIZrGaziiuOJtJQ"
```

同 `/hedge-fund/agents`，返回格式略有不同。

---

## TradingAgents API（端口 8080）

### POST /api/v1/analyze — 提交分析任务

```bash
curl -s -X POST http://172.16.13.58:8080/api/v1/analyze \
  -H "Authorization: Bearer ITBVOCmV4m2Wm1qHL-yRsxE4D3QVMQXPWLWxSIdBHKo" \
  -H "Content-Type: application/json" \
  -d '{
    "ticker": "AAPL",
    "date": "2025-11-28",
    "analysts": ["market", "social", "news", "fundamentals"],
    "max_debate_rounds": 1
  }'
```

**参数**：
- `ticker`（必填）：单只股票
- `date`（关键）：分析日期 YYYY-MM-DD，详见主文件「日期对齐规则」
- `analysts`：`full` = 全部 4 个，`quick` = `["market", "fundamentals"]`
- `max_debate_rounds`：辩论轮数，默认 1

**返回**：`{"task_id": "xxx"}`

### GET /api/v1/tasks/{task_id} — 轮询进度

```bash
curl -s http://172.16.13.58:8080/api/v1/tasks/<task_id> \
  -H "Authorization: Bearer ITBVOCmV4m2Wm1qHL-yRsxE4D3QVMQXPWLWxSIdBHKo"
```

进度阶段：market → social → news → fundamentals → debate → risk_assessment → complete。每 15 秒轮询，通常 3-10 分钟。

### 返回数据结构

```
result.reports.market_report       — 市场技术分析
result.reports.sentiment_report    — 社交媒体情绪
result.reports.news_report         — 新闻分析
result.reports.fundamentals_report — 基本面分析
result.investment_debate.bull_history   — 多头论点
result.investment_debate.bear_history   — 空头论点
result.investment_debate.judge_decision — 评审裁决
result.trader_investment_plan      — 交易计划
result.risk_debate                 — 风险评估
result.final_trade_decision        — 最终决策（BUY/SELL/HOLD）
```

### GET /api/v1/tasks — 列出所有任务

```bash
curl -s http://172.16.13.58:8080/api/v1/tasks \
  -H "Authorization: Bearer ITBVOCmV4m2Wm1qHL-yRsxE4D3QVMQXPWLWxSIdBHKo"
```

---

## 回测引擎 API（端口 8090）

### POST /api/v1/strategies/upload — 上传策略

```bash
curl -s -X POST http://172.16.13.58:8090/api/v1/strategies/upload \
  -H "Authorization: Bearer NoHKWxHvaFKdsNutAKGjJWD_RmDPipxurgT_U9H_Oh0" \
  -F "file=@/tmp/strategy_aapl.py"
```

文件 ≤100KB，.py 后缀。

### POST /api/v1/ingest — 导入市场数据

```bash
curl -s -X POST http://172.16.13.58:8090/api/v1/ingest \
  -H "Authorization: Bearer NoHKWxHvaFKdsNutAKGjJWD_RmDPipxurgT_U9H_Oh0" \
  -H "Content-Type: application/json" \
  -d '{"symbols": ["AAPL"], "start_date": "2025-01-01", "end_date": "2025-02-01"}'
```

### POST /api/v1/backtest — 提交回测

```bash
curl -s -X POST http://172.16.13.58:8090/api/v1/backtest \
  -H "Authorization: Bearer NoHKWxHvaFKdsNutAKGjJWD_RmDPipxurgT_U9H_Oh0" \
  -H "Content-Type: application/json" \
  -d '{
    "symbols": ["AAPL"],
    "start_date": "2025-12-01",
    "end_date": "2025-12-31",
    "algorithm_file": "uploads/strategy_aapl.py",
    "capital_base": 100000
  }'
```

### GET /api/v1/bundles — 列出数据包

```bash
curl -s http://172.16.13.58:8090/api/v1/bundles \
  -H "Authorization: Bearer NoHKWxHvaFKdsNutAKGjJWD_RmDPipxurgT_U9H_Oh0"
```

### GET /api/v1/strategies — 列出已上传策略

```bash
curl -s http://172.16.13.58:8090/api/v1/strategies \
  -H "Authorization: Bearer NoHKWxHvaFKdsNutAKGjJWD_RmDPipxurgT_U9H_Oh0"
```

### GET /api/v1/tasks/{task_id} — 查询回测结果

```bash
curl -s http://172.16.13.58:8090/api/v1/tasks/<task_id> \
  -H "Authorization: Bearer NoHKWxHvaFKdsNutAKGjJWD_RmDPipxurgT_U9H_Oh0"
```

### 健康检查

```bash
curl -s http://172.16.13.58:8082/api/v1/health  # AI Hedge Fund
curl -s http://172.16.13.58:8080/api/v1/health  # TradingAgents
curl -s http://172.16.13.58:8090/api/v1/health  # 回测引擎
curl -s http://172.16.13.58:8083/health          # Alpaca Trading
```

---

## Alpaca Trading API（端口 8083）

模拟盘交易执行服务，支持下单、撤单、持仓管理、账户查询和行情获取。

### GET /health — 健康检查

```bash
curl -s http://172.16.13.58:8083/health \
  -H "Authorization: Bearer JISCaUdXQEyzIQmVwpe1HEh++6CFDVm+Aj1X9snsj4I="
```

### GET /account — 账户信息

```bash
curl -s http://172.16.13.58:8083/account \
  -H "Authorization: Bearer JISCaUdXQEyzIQmVwpe1HEh++6CFDVm+Aj1X9snsj4I="
```

**返回关键字段**：
- `buying_power`：可用购买力
- `equity`：总权益
- `cash`：现金余额
- `portfolio_value`：投资组合价值
- `status`：账户状态

### POST /orders — 下单

```bash
# 市价买入
curl -s -X POST http://172.16.13.58:8083/orders \
  -H "Authorization: Bearer JISCaUdXQEyzIQmVwpe1HEh++6CFDVm+Aj1X9snsj4I=" \
  -H "Content-Type: application/json" \
  -d '{
    "symbol": "AAPL",
    "qty": 1,
    "side": "buy",
    "type": "market",
    "time_in_force": "day"
  }'

# 限价买入
curl -s -X POST http://172.16.13.58:8083/orders \
  -H "Authorization: Bearer JISCaUdXQEyzIQmVwpe1HEh++6CFDVm+Aj1X9snsj4I=" \
  -H "Content-Type: application/json" \
  -d '{
    "symbol": "AAPL",
    "qty": 1,
    "side": "buy",
    "type": "limit",
    "limit_price": 250,
    "time_in_force": "day"
  }'
```

**请求参数**：
- `symbol`（必填）：股票代码
- `qty`（必填）：数量（支持小数，如 0.01）
- `side`（必填）：`buy` 或 `sell`
- `type`（必填）：`market`（市价）或 `limit`（限价）
- `limit_price`：限价单价格（type=limit 时必填）
- `time_in_force`：有效期，默认 `day`（当日有效），可选 `gtc`（撤单前有效）

**返回关键字段**：
- `id`：订单 ID
- `status`：`accepted`/`new`/`filled`/`partially_filled`/`canceled`
- `filled_avg_price`：成交均价
- `filled_qty`：已成交数量

### GET /orders — 订单列表

```bash
# 获取所有订单
curl -s http://172.16.13.58:8083/orders \
  -H "Authorization: Bearer JISCaUdXQEyzIQmVwpe1HEh++6CFDVm+Aj1X9snsj4I="

# 带过滤参数
curl -s "http://172.16.13.58:8083/orders?status=open&limit=10" \
  -H "Authorization: Bearer JISCaUdXQEyzIQmVwpe1HEh++6CFDVm+Aj1X9snsj4I="
```

**查询参数**：
- `status`：`open`/`closed`/`all`（默认 `open`）
- `limit`：返回数量限制
- `direction`：`asc`/`desc`

### DELETE /orders/{order_id} — 取消订单

```bash
curl -s -X DELETE http://172.16.13.58:8083/orders/<order_id> \
  -H "Authorization: Bearer JISCaUdXQEyzIQmVwpe1HEh++6CFDVm+Aj1X9snsj4I="
```

### GET /positions — 持仓列表

```bash
curl -s http://172.16.13.58:8083/positions \
  -H "Authorization: Bearer JISCaUdXQEyzIQmVwpe1HEh++6CFDVm+Aj1X9snsj4I="
```

**返回每个持仓**：
- `symbol`：标的
- `qty`：持仓数量
- `avg_entry_price`：平均成本
- `current_price`：当前价格
- `market_value`：市值
- `unrealized_pl`：未实现盈亏
- `unrealized_plpc`：未实现盈亏百分比
- `side`：`long`

### DELETE /positions/{symbol} — 平仓指定标的

```bash
curl -s -X DELETE http://172.16.13.58:8083/positions/AAPL \
  -H "Authorization: Bearer JISCaUdXQEyzIQmVwpe1HEh++6CFDVm+Aj1X9snsj4I="
```

### DELETE /positions — 全部平仓

```bash
curl -s -X DELETE http://172.16.13.58:8083/positions \
  -H "Authorization: Bearer JISCaUdXQEyzIQmVwpe1HEh++6CFDVm+Aj1X9snsj4I="
```

### GET /quotes/{symbol} — 实时报价

```bash
curl -s http://172.16.13.58:8083/quotes/AAPL \
  -H "Authorization: Bearer JISCaUdXQEyzIQmVwpe1HEh++6CFDVm+Aj1X9snsj4I="
```

**返回关键字段**：
- `symbol`：标的
- `last_price`：最新价格
- `bid_price`/`ask_price`：买一/卖一价
- `bid_size`/`ask_size`：买一/卖一量
- `volume`：成交量
- `timestamp`：时间戳

### GET /bars/{symbol} — K线数据

```bash
curl -s "http://172.16.13.58:8083/bars/AAPL?timeframe=1Day&limit=5" \
  -H "Authorization: Bearer JISCaUdXQEyzIQmVwpe1HEh++6CFDVm+Aj1X9snsj4I="
```

**查询参数**：
- `timeframe`：`1Min`/`5Min`/`15Min`/`1Hour`/`1Day`（默认 `1Day`）
- `limit`：K线数量
- `start`/`end`：时间范围（ISO 8601 格式）

**返回每根K线**：
- `timestamp`、`open`、`high`、`low`、`close`、`volume`

---

## 并行请求模式

### 双引擎并行分析（流程二/三）

TradingAgents 只支持单股票，需要循环；AI Hedge Fund 支持多股票：

```bash
# TradingAgents（每只股票独立提交）
for TICKER in AAPL NVDA; do
  curl -s -X POST http://172.16.13.58:8080/api/v1/analyze \
    -H "Authorization: Bearer ITBVOCmV4m2Wm1qHL-yRsxE4D3QVMQXPWLWxSIdBHKo" \
    -H "Content-Type: application/json" \
    -d "{\"ticker\":\"$TICKER\",\"date\":\"2025-11-28\",\"analysts\":[\"market\",\"social\",\"news\",\"fundamentals\"],\"max_debate_rounds\":1}" &
done

# AI Hedge Fund（一次传多只）
curl -s -X POST http://172.16.13.58:8082/api/v1/analyze \
  -H "Authorization: Bearer xG8XyygVqe9OtV5WcQH8phECIQ8wiIZrGaziiuOJtJQ" \
  -H "Content-Type: application/json" \
  -d '{"tickers":["AAPL","NVDA"],"show_reasoning":true}' &

wait
```
