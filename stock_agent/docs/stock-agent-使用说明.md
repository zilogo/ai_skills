# Stock Agent 使用说明

> 最后测试日期：2026-03-12 | 所有工作流均已验证通过

## 目录

- [快速开始](#快速开始)
- [流程一：全自动分析+回测](#流程一全自动分析回测)
- [流程二：双引擎分析+自定义策略+回测](#流程二双引擎分析自定义策略回测)
- [流程三：纯分析](#流程三纯分析)
- [流程四：纯回测](#流程四纯回测)
- [流程五：对比回测](#流程五对比回测)
- [流程六：模拟盘交易](#流程六模拟盘交易)
- [辅助命令](#辅助命令)
- [服务状态说明](#服务状态说明)

---

## 快速开始

### 触发方式

在 Claude Code 中直接用自然语言对话即可触发，无需记忆命令格式：

| 你说的话 | 触发的流程 |
|----------|-----------|
| "帮我看看 AAPL" | 流程一（全自动分析+回测） |
| "分析一下 NVDA 最近的走势" | 流程三（纯分析） |
| "我想设计一个均线策略来交易 TSLA" | 流程二（自定义策略） |
| "用已有策略回测 AAPL 12 月的表现" | 流程四（纯回测） |
| "对比一下自动策略和我的策略哪个好" | 流程五（对比回测） |
| "买入 1 股 AAPL" | 流程六（交易） |
| "AAPL 现在多少钱" | 报价查询 |
| "看看我的持仓" | 持仓查询 |

### 命令格式（可选）

> 需要额外安装：将 `skill/stock-agent.md` 复制到 `.claude/commands/stock-agent.md` 后才能使用斜杠命令。不安装也不影响使用，自然语言触发完全等效。

```
/stock-agent AAPL              → 流程一
/stock-agent analyze AAPL      → 流程三
/stock-agent AAPL --custom     → 流程二
/stock-agent backtest AAPL     → 流程四
/stock-agent compare AAPL      → 流程五
/stock-agent buy AAPL 1        → 流程六（买入）
/stock-agent quote AAPL        → 报价
/stock-agent portfolio         → 账户+持仓
```

---

## 流程一：全自动分析+回测

**适用场景**：快速了解一只股票值不值得买

**触发方式**：
- "帮我看看 AAPL"
- "AAPL 值得买吗"

### 执行过程

```
阶段 1：AI Hedge Fund 分析（~30 秒）
  └─ 18 位投资大师量化信号 → 综合决策
阶段 2：AI Hedge Fund 回测（SSE 流式，每交易日 ~15 秒）
  └─ 每日 LLM 动态决策 → 组合绩效
阶段 3：生成报告
```

### 阶段 1：投资大师分析

提交分析后返回 `task_id`，轮询直到 `status: completed`。以下是 AAPL 的实际测试结果：

**API 返回原始数据**：

```json
{
  "status": "completed",
  "result": {
    "decisions": {
      "AAPL": {
        "action": "short",
        "quantity": 91,
        "confidence": 85,
        "reasoning": "Burry's high confidence bearish signal (85%) drives short position..."
      }
    },
    "analyst_signals": {
      "technical_analyst_agent": {
        "AAPL": {
          "signal": "bearish",
          "confidence": 22,
          "reasoning": {
            "trend_following": { "signal": "bearish", "confidence": 44, "metrics": { "adx": 44.11 } },
            "mean_reversion": { "signal": "neutral", "confidence": 50, "metrics": { "rsi_14": 17.73 } },
            "momentum": { "signal": "neutral", "confidence": 50 },
            "volatility": { "signal": "neutral", "confidence": 50 }
          }
        }
      },
      "warren_buffett_agent": {
        "AAPL": {
          "signal": "neutral",
          "confidence": 55,
          "reasoning": "Wonderful business with durable moat and excellent ROE, but trading far above intrinsic value. Price matters."
        }
      },
      "michael_burry_agent": {
        "AAPL": {
          "signal": "bearish",
          "confidence": 85.0,
          "reasoning": "..."
        }
      }
    }
  }
}
```

**Agent 生成的报告（01_analysis_hedgefund.md）截取**：

以下是 TSLA 的实际大师信号报告（完整报告以此格式输出）：

```markdown
# TSLA 投资大师信号分析

> 分析时间：2025-11-28 | 模型：glm-5-fp8

## 最终决策

**Action: short** | Confidence: 90% | Quantity: 31

Reasoning: Overwhelming bearish consensus across valuation, fundamentals,
and sentiment supports short.

## 投资大师信号

| 大师 | 信号 | 置信度 | 核心理由 |
|------|------|--------|----------|
| Warren Buffett | **bearish** | **88%** | Weak ROE 4.8%, negative earnings growth, limited moat. Trading at 70x intrinsic value. |
| Michael Burry | **bearish** | **72%** | FCF yield 0.4%. Absurd valuation. Paying ~250x FCF. |
| Ben Graham | **bearish** | **90%** | Price $449.72 vs Graham Number $26.00 — 17x premium. |
| Bill Ackman | **bearish** | **88%** | Paying $1.5T for intrinsic value ~$105B is speculation. ROE 4.8% is pedestrian. |
| Cathie Wood | **bearish** | **68%** | Disruptive tech score 0.42/12. R&D at 6.8% of revenue lags 15-25% typical. |
| Peter Lynch | **bearish** | **72%** | P/E 394 with negative earnings growth — can't build a ten-bagger from here. |
| Valuation Analyst | **bearish** | **100%** | DCF value $43B vs market cap $1.5T (-97.1%). Massively overvalued. |
| Technical Analyst | **bullish** | **21%** | Trend following bullish (ADX 39.9), but weak bullish signal. |

## 大师共识

- 看多：1 位（Technical Analyst, conf=21%）
- 看空：16 位
- 中性：1 位（News Sentiment）
- **共识方向**：bearish（16/18，压倒性看空）

## 估值详细分析

| 方法 | 估值 | 市值 | 差距 | 权重 |
|------|------|------|------|------|
| DCF 分析 | $43.0B | $1,495.7B | -97.1% | 35% |
| Owner Earnings | $20.7B | $1,495.7B | -98.6% | 35% |
| EV/EBITDA | $806.9B | $1,495.7B | -46.0% | 20% |
```

**关键字段说明**：

| 字段 | 说明 |
|------|------|
| `decisions.AAPL.action` | 最终综合决策：buy/sell/short/cover/hold |
| `decisions.AAPL.confidence` | 决策置信度（0-100%），越高越确定 |
| `analyst_signals` | 每位大师的独立信号 |
| `signal` | 看多(bullish)/看空(bearish)/中性(neutral) |
| `reasoning` | 决策推理过程（技术分析师包含 RSI/ADX 等具体指标） |

### 阶段 2：SSE 流式回测

回测通过 SSE（Server-Sent Events）实时推送每个交易日的结果。以下是 AAPL 的实际 SSE 流数据截取：

```
event: start
data: {"type":"start","timestamp":null}

event: progress
data: {"type":"progress","agent":"backtest","status":"Processing 2025-12-22 (1/8)"}

event: progress
data: {"type":"progress","agent":"warren_buffett","ticker":"AAPL",
       "status":"Fetching financial metrics"}

event: progress
data: {"type":"progress","agent":"warren_buffett","ticker":"AAPL",
       "status":"Gathering financial line items"}

event: progress
data: {"type":"progress","agent":"warren_buffett","ticker":"AAPL",
       "status":"Analyzing fundamentals"}

event: progress
data: {"type":"progress","agent":"warren_buffett","ticker":"AAPL",
       "status":"Calculating intrinsic value"}

event: progress
data: {"type":"progress","agent":"warren_buffett","ticker":"AAPL",
       "status":"Done",
       "analysis":"Wonderful business with durable moat and excellent ROE,
                   but trading far above intrinsic value. Price matters."}

event: progress
data: {"type":"progress","agent":"technical_analyst","ticker":"AAPL",
       "status":"Analyzing price data"}

event: progress
data: {"type":"progress","agent":"technical_analyst","ticker":"AAPL",
       "status":"Calculating trend signals"}

event: progress
data: {"type":"progress","agent":"technical_analyst","ticker":"AAPL",
       "status":"Done",
       "analysis":"{\"AAPL\":{\"signal\":\"bearish\",\"confidence\":23,
                   \"reasoning\":{\"trend_following\":{\"signal\":\"bearish\",
                   \"confidence\":44,\"metrics\":{\"adx\":44.11}},
                   \"mean_reversion\":{\"signal\":\"neutral\",\"confidence\":50,
                   \"metrics\":{\"rsi_14\":17.73}}}}}"}

event: progress
data: {"type":"progress","agent":"backtest",
       "status":"Completed 2025-12-22 - Portfolio: $100,000.00",
       "analysis":"{
         \"date\": \"2025-12-22\",
         \"portfolio_value\": 100000.0,
         \"cash\": 100000.0,
         \"decisions\": {
           \"AAPL\": {
             \"signal\": \"neutral\",
             \"confidence\": 55,
             \"reasoning\": \"Wonderful business...but trading far above intrinsic value.\"
           }
         },
         \"executed_trades\": {\"AAPL\": 0},
         \"current_prices\": {\"AAPL\": 270.97},
         \"performance_metrics\": {
           \"sharpe_ratio\": 0.0,
           \"sortino_ratio\": 0.0,
           \"max_drawdown\": 0.0
         },
         \"ticker_details\": [{
           \"ticker\": \"AAPL\",
           \"action\": \"hold\",
           \"quantity\": 0,
           \"price\": 270.97,
           \"bullish_count\": 0,
           \"bearish_count\": 1,
           \"neutral_count\": 1
         }]
       }"}

event: progress
data: {"type":"progress","agent":"backtest","status":"Processing 2025-12-23 (2/8)"}
...（每个交易日重复以上流程）...

event: end
data: {"type":"final","performance_metrics":{
  "sharpe_ratio": 0.0,
  "sortino_ratio": 0.0,
  "max_drawdown": 0.0,
  "long_short_ratio": 0.0
}}
```

**SSE 事件流说明**：

| 事件类型 | 内容 |
|----------|------|
| `start` | 回测开始标志 |
| `progress` (agent=大师名) | 某位大师正在分析当日数据，包含状态流转（Fetching → Analyzing → Done） |
| `progress` (agent=backtest) | 当日分析完毕，包含组合价值、交易记录、各大师信号和绩效指标 |
| `end` / `final` | 回测完成，包含 Sharpe/Sortino/最大回撤等终期绩效指标 |

**Agent 生成的报告（03_backtest_auto.md）截取**：

以下是 TSLA 的实际自动化回测报告（展示报告格式）：

```markdown
# TSLA 自动化回测报告

> 回测区间：2025-12-01 ~ 2025-12-15 | 初始资金：$100,000
> 参与大师：10 位（warren_buffett, technical_analyst, fundamentals_analyst,
>   michael_burry, cathie_wood, bill_ackman, ben_graham, peter_lynch,
>   sentiment_analyst, valuation_analyst）

## 核心指标

| 指标 | 数值 |
|------|------|
| 最终净值 | $100,000.00 |
| 总收益率 | 0.00% |
| Sharpe 比率 | 0.00 |
| Sortino 比率 | 0.00 |
| 最大回撤 | 0.00% |
| 交易天数 | 11 |
| **实际交易次数** | **0** |

## 回测结果说明

本次 0 交易是**预期行为**：
1. **压倒性看空共识**：19 位大师中 17 位 bearish，仅 1 位 bullish
2. **LLM 正确决策**：面对强烈看空共识，portfolio_manager 不建仓是合理的风控决策
3. **最终分析决策为 short**（做空），回测默认不启用做空，保持空仓

### 关键看空信号
- Ben Graham: bearish 90% — 完全不满足安全边际要求
- Bill Ackman: bearish 88% — 估值极端
- Warren Buffett: bearish 88% — ROE 弱，P/E 过高
- Valuation Analyst: bearish 100% — DCF 估值远低于市价
```

### 阶段 3：生成综合报告

自动保存到 `./reports/AAPL/<日期>/`：
- `01_analysis_hedgefund.md` — 投资大师信号汇总
- `03_backtest_auto.md` — 自动化回测绩效
- `04_final_report.md` — 综合报告

---

## 流程二：双引擎分析+自定义策略+回测

**适用场景**：参考 AI 分析，自己设计交易策略

**触发方式**：
- "我想设计一个均线策略来交易 AAPL"
- "帮我做个 RSI 策略回测"

### 执行过程

```
阶段 1：双引擎并行分析（TradingAgents 3-10 分钟 + AI Hedge Fund ~30 秒）
阶段 2：策略代码生成（基于分析结果自动生成 Python 策略）
阶段 3：回测引擎验证（上传策略 → 导入数据 → 执行回测）
阶段 4：综合报告
```

### 阶段 1：双引擎并行分析

两个引擎并行分析，各有侧重。

#### TradingAgents 输出（深度文本报告）

TradingAgents 耗时 3-10 分钟，产出 4 份专业报告 + 多空辩论 + 最终决策。

进度阶段流转：
```
market(17%) → social(33%) → news(50%) → fundamentals(67%)
→ debate(83%) → risk_assessment(100%) → complete
```

**各子报告内容（AAPL 实际测试数据）**：

| 报告 | 字符量 | 内容概述 |
|------|--------|---------|
| `market_report` | 11,451 | 技术分析：均线/RSI/布林带/支撑阻力位 |
| `sentiment_report` | 16,006 | 社交媒体情绪分析 |
| `news_report` | 15,114 | 新闻分析 |
| `fundamentals_report` | 17,319 | 基本面分析：PE/PEG/ROE/现金流 |

**市场技术分析报告截取**（market_report 实际内容）：

```markdown
# Comprehensive Technical Analysis Report: AAPL (Apple Inc.)

**Analysis Date:** November 28, 2025

## Executive Summary

Apple (AAPL) is exhibiting a **strong bullish trend** across multiple
timeframes, with price action firmly supported by all major moving averages.
However, momentum indicators suggest the stock is approaching overbought
territory, and volatility has been gradually increasing...

## Detailed Technical Analysis

### 1. Moving Average Analysis: Bearish Divergence Pattern

**50-Day Simple Moving Average (SMA)**
- Current Value: 263.63 (March 10, 2026)
- 30-Day Change: -4.78 points, -1.78%
- Trend: Consistently downward sloping

**10-Day Exponential Moving Average (EMA)**
- Current Value: 262.51
- 10 EMA currently trading 1.12 points BELOW the 50 SMA → bearish signal

### 2. Bollinger Band Analysis

| Date    | Upper Band | Middle Band | Lower Band | Band Width |
|---------|------------|-------------|------------|------------|
| Feb 9   | 280.26     | 260.68      | 241.10     | 39.16      |
| Feb 26  | 281.76     | 268.31      | 254.86     | 26.90      |
| Mar 10  | 276.23     | 264.96      | 253.70     | 22.53      |
```

**多空辩论截取**（investment_debate 实际内容）：

```markdown
## 多头论点（Bull History）

Listen, I've heard the bear case loud and clear, and frankly, it relies
heavily on short-term noise and headline risks while ignoring the structural
shift happening right under our noses. When you look at the totality of the
evidence—technical, fundamental, and macro—the argument for staying long
and strong on Apple isn't just defensible; it's compelling.

### 1. The Growth Story is Reaccelerating, Not Stalling
The bear case loves to cling to the narrative of "maturing smartphone markets."
But let's look at the actual numbers...

## 空头论点（Bear History）

I appreciate the enthusiasm, but let's pump the brakes. You're confusing a
great company with a great stock, and in this market, that distinction is the
difference between locking in profits and catching a falling knife.

### 1. The Valuation Trap: Paying a Growth Price for Mature Growth
You're touting 15.7% quarterly revenue growth, but a PEG ratio of 2.34 for a
company growing at 6.4% annually represents a significant valuation disconnect...

## 裁判裁决（Judge Decision）

Alright, let's cut through the noise. We have a classic standoff here: the
Bull is betting on the unstoppable force of Apple's brand and buyback power,
while the Bear is warning that the valuation is a trap and the technicals
are flashing exhaustion...
```

**最终交易决策**（final_trade_decision 实际全文）：

```markdown
### Recommendation: SELL (Trim 30% of Position)

I have evaluated the debate and determined that the Trader's original plan
to sell 50% is emotionally reactive and technically flawed.

### Summary of Key Arguments
1. Aggressive Analyst: Argued for a full 50% liquidation based on a
   "Quality Trap" thesis. PEG ratio of 2.34 for a 6% grower is expensive.
2. Conservative Analyst: Advocated for a Hold, framing the valuation
   premium as the "price of certainty."
3. Neutral Analyst: Dismantled the "exhaustion" narrative by highlighting
   RSI reset from 71 to 51 without breaking price support—bullish consolidation.

### Refined Trader's Plan
1. Trim 30% Immediately: Reduce position to lock in profits.
2. Adjust Stop-Loss to $262 (50-day moving average).
3. Re-entry Strategy: Buy back in $240-$245 range.
   If breakout above $278 (upper Bollinger Band), re-enter on pullback.
```

**Agent 生成的报告（01_analysis_summary.md）截取**：

```markdown
# AAPL 分析摘要

**决策：SELL**

| 维度 | 核心结论 |
|------|----------|
| 技术面 | 均线空头排列+布林带压缩=大幅波动在即 |
| 情绪面 | 市场对 AI 延迟失望，但品牌忠诚度仍在 |
| 新闻面 | 宏观滞胀+防御性定价策略暗示需求疲软 |
| 基本面 | 毛利率仍高但估值过高+增长放缓 |

**最终交易决策原文：**

My final recommendation is to **SELL**.

While the Neutral analyst presents a compelling case for Apple's long-term
durability, the collective weight of the macro headwinds and the trader's
specific past failures with "reliable" mega-caps during downturns
necessitates an exit.

### Summary of Key Arguments
- **Aggressive Analyst:** Apple is a "valuation trap" trading at 33x earnings
  with contracting growth. The 18-month AI delay is an "unmitigated disaster."
- **Conservative Analyst:** Agrees with selling but opposes high-beta rotation.
  Advocates strict move to Treasuries.
- **Neutral Analyst:** Highlights expanding gross margins (48.2%) and massive
  buybacks ($24B last quarter). Advocates partial trim, not full exit.
```

#### AI Hedge Fund 输出（量化信号）

与流程一阶段 1 相同，提供 18 位大师的信号和置信度。约 30 秒完成。

**两个引擎的互补关系**：
- TradingAgents 告诉你**在哪个价位买卖**（如 "200 SMA $263.63，RSI 45.49，布林带下轨 $253.70"）
- AI Hedge Fund 告诉你**多少大师看多/看空**（如 "巴菲特 neutral 55%，Burry bearish 85%"）
- 交叉验证：两者方向一致 → 强信号；方向冲突 → 需谨慎

### 阶段 2：策略代码生成

Agent 综合两个引擎的分析自动生成 Python 策略代码。

**Agent 生成的报告（02_strategy_notes.md）截取**：

```markdown
# AAPL 策略设计说明

- **策略类型**：布林带均值回归（Bollinger Band Mean-Reversion）
- **基于分析**：TradingAgents AI 分析（2026-03-10）
- **AI 决策**：SELL

## 入场条件
- 价格触及布林带下轨（253-254 支撑区域）
- RSI < 35（来自"in a downtrend, 35-40 often acts as support"）

## 退出条件
- 价格回归布林带中轨（20 SMA）
- RSI > 55

## 止损逻辑
- 2.5% 固定止损（AAPL 波动率低于 TSLA，可以更紧凑）

## 仓位管理
- 最大仓位 40%（AAPL 相对稳定，但 AI 仍建议 SELL，不激进）

## 参数表
| 参数 | 值 | 来源 |
|------|-----|------|
| bb_period | 20 | 标准布林带 20 日周期 |
| bb_std | 2.0 | 标准 2 倍标准差 |
| rsi_period | 14 | 标准 RSI |
| rsi_oversold | 35 | "35-40 often acts as support in downtrend" |
| rsi_overbought | 55 | "weak demand above 55" |
| stop_loss_pct | 0.025 | AAPL 低波动，紧凑止损 |
| max_position | 0.4 | 中等仓位，防守型 |
```

### 阶段 3：回测引擎验证

回测引擎返回数据结构（API 原始数据）：

```json
{
  "status": "completed",
  "params": {
    "symbols": ["AAPL"],
    "start_date": "2025-12-01",
    "end_date": "2025-12-31",
    "algorithm_file": "uploads/strategy_aapl_202512.py",
    "capital_base": 100000.0,
    "trading_calendar": "NYSE",
    "bundle_name": "yahoo_finance_daily_data"
  },
  "result": {
    "symbols": ["AAPL"],
    "perf": {
      "period_open": ["2025-12-01T14:31:00+00:00", "2025-12-02T14:31:00+00:00", ...],
      "period_close": ["2025-12-01T16:00:00-05:00", "2025-12-02T16:00:00-05:00", ...],
      "returns": [0.0, 0.0, 0.0, ...],
      "portfolio_value": [100000.0, 100000.0, ...],
      "pnl": [0.0, 0.0, ...],
      "long_value": [0.0, 0.0, ...],
      "orders": [],
      "transactions": []
    },
    "errors": []
  }
}
```

**当策略有实际交易时**（TSLA 的回测报告截取，strategy_tsla.py 产生了真实交易）：

```markdown
# TSLA 回测结果报告

> 策略文件：strategy_tsla.py | 回测区间：2025-11-01 ~ 2025-12-31
> 初始资金：$100,000

## 核心指标

| 指标 | 数值 |
|------|------|
| 最终净值 | $101,083.82 |
| 总收益率 | +1.08% |
| Sharpe 比率 | 1.76 |
| 最大回撤 | -0.76% |
| 交易天数 | 41 |
| 持仓天数 | 26/41 (63%) |
| 交易轮次 | 约 6 轮 |

## 交易时间线

| 日期 | 操作 | 组合价值 | 说明 |
|------|------|---------|------|
| 11/14 | **入场** | $99,993 | Signal 触发，35% 仓位买入 |
| 11/20 | **退出** | $99,714 | 均线死叉 + 小幅亏损，止损退出 |
| 11/21 | **入场** | $99,708 | 信号再次触发 |
| 11/24 | **退出** | $100,529 | 布林带上轨附近获利了结 |
| 11/25 | **入场** | $100,523 | 新信号触发 |
| 12/03 | **退出** | $101,309 | 获利退出 |
| 12/04 | **入场** | $101,303 | 回调后再次入场 |
| 12/15 | **退出** | $101,857 | 获利退出（本轮最高点） |
| 12/16 | **入场** | $101,851 | 信号触发 |
| 12/29 | **退出** | $101,089 | 均线死叉退出 |
| 12/31 | **入场** | $101,083 | 期末持仓 |

## 策略特点
- 6 轮交易中 4 轮盈利、2 轮小亏，胜率约 67%
- 35% 轻仓有效控制了波动风险
- 最大单次亏损约 -0.76%
```

**注意**：如果 `returns` 全为 0 且 `errors` 不为空，通常是策略代码有 bug。常见错误：
```json
{
  "errors": [{
    "message": "Exception raised during simulation: KeyError: 22",
    "trace": "...data.history() → sid_indexes[asset_sid] → KeyError: 22...",
    "simulation_dt": "2025-12-01T16:00:00-05:00"
  }]
}
```
这表示策略的数据访问方式与当前数据包不兼容，需要调整。

### 阶段 4：综合报告

**Agent 生成的报告（04_final_report.md）截取**：

```markdown
# AAPL 综合交易分析报告

> 生成时间：2026-03-11 | 股票代码：AAPL | 分析周期：2025 年 12 月

## 一、AI 分析结论

**决策：SELL**

**核心逻辑**：
宏观滞胀环境（$119 油价 + 4.4% 失业率）、AI 战略延迟 18 个月、
33 倍市盈率在利润率压缩环境下不可持续。

**关键信号**：
- RSI 45.49，中性偏空
- 10 EMA (262.51) < 50 SMA (263.63)，短期下行趋势确认
- 布林带压缩 42%，大幅突破在即
- 支撑位 253-254，阻力位 276-277

**各维度评分**：

| 维度 | 方向 | 核心观点 |
|------|------|----------|
| 技术面 | 看空 | 均线空头排列+布林带压缩=大幅波动在即 |
| 情绪面 | 中性偏空 | 市场对 AI 延迟失望，但品牌忠诚度仍在 |
| 新闻面 | 看空 | 宏观滞胀+防御性定价策略暗示需求疲软 |
| 基本面 | 中性偏空 | 毛利率仍高但估值过高+增长放缓 |

## 二、策略设计

- **策略类型**：布林带均值回归
- **入场条件**：价格触及布林带下轨（±1%）且 RSI < 35
- **退出条件**：价格回归布林带中轨或 RSI > 55
- **止损**：2.5% | **仓位**：最大 40%

策略本质：**AI 说卖，策略只在极端位置做短线反弹**。

## 三、回测验证

| 指标 | 数值 |
|------|------|
| 总收益率 | 0.00% |
| 交易次数 | 0 |

## 四、AI 决策 vs 回测验证

AI 判定 **SELL**，回测结果为 **0% 收益（全程现金）**。
一致性：完全一致。策略忠实执行 AI 的 SELL 建议，
价格未触及入场条件，未入场。

## 六、建议操作

1. **减仓 AAPL**，但不必完全清仓（毛利率 48.2%）
2. **关注布林带突破方向**：突破上轨 276-277 可考虑加仓，跌破 253 坚决离场
3. **止损位设在 $253 下方**
4. **关注催化剂**：WWDC AI 产品发布、季度财报、宏观数据
```

---

## 流程三：纯分析

**适用场景**：只想了解某只股票当前状态，不需要回测

**触发方式**：
- "分析一下 AAPL"
- "帮我看看 NVDA 的基本面"
- "TSLA 现在什么信号"

### 三种分析模式

| 指令 | 引擎 | 耗时 | 适用场景 |
|------|------|------|----------|
| `analyze AAPL` | 双引擎（默认） | 3-10 分钟 | 最全面 |
| `analyze AAPL --deep` | 仅 TradingAgents | 3-10 分钟 | 需要具体价位参数 |
| `analyze AAPL --masters` | 仅 AI Hedge Fund | ~30 秒 | 快速查看大师共识 |

### 进度查看

分析过程中可以随时问 "进度如何"，Agent 会轮询 API 返回进度。

TradingAgents 进度阶段和对应百分比：
```
market(17%) → social(33%) → news(50%) → fundamentals(67%)
→ debate(83%) → risk_assessment(100%) → complete
```

实际测试耗时记录：
| 时间点 | 进度 | 说明 |
|--------|------|------|
| 提交 | 0% | 返回 task_id |
| +30s | 17% | market 阶段 |
| +2min | 67% | fundamentals 阶段 |
| +4min | 83% | debate 阶段 |
| +6min | 100% | risk_assessment |
| +7min | completed | 全部完成 |

### 交叉验证逻辑

| TradingAgents | AI Hedge Fund | 结论 |
|---------------|---------------|------|
| SELL | 多数大师 bearish | 强看空信号 |
| BUY | 多数大师 bullish | 强看多信号 |
| BUY | Michael Burry bearish | 存在分歧，需谨慎 |
| HOLD | 大师信号分散 | 观望为主 |

---

## 流程四：纯回测

**适用场景**：已有策略代码，想验证绩效

**触发方式**：
- "用已有策略回测 AAPL"
- "回测一下 12 月的 AAPL 策略"

### 执行过程

```
检查策略文件 → 上传到回测引擎 → 检查数据(自动 ingest) → 执行回测 → 绩效报告
```

### 策略来源优先级

1. 用户指定的策略文件
2. `./reports/{TICKER}/02_strategy.py`（流程二生成的策略）
3. 回测引擎已有策略（可通过 "有哪些策略" 查看）

### 已有策略列表（当前服务器）

| 策略名称 | 路径 |
|----------|------|
| strategy_aapl_202512 | uploads/strategy_aapl_202512.py |
| strategy_nvda_202512 | uploads/strategy_nvda_202512.py |
| strategy_tsla_202512 | uploads/strategy_tsla_202512.py |
| strategy_amd_202512 | uploads/strategy_amd_202512.py |
| strategy_tsla | uploads/strategy_tsla.py |
| test_algo | test_algo/test_algo.py |

### 已导入数据范围

| 时间范围 | 说明 |
|----------|------|
| 2025-01-01 ~ 2025-08-30 | 大范围数据 |
| 2025-11-01 ~ 2026-01-05 | 近期数据 |
| 2025-12-01 ~ 2025-12-31 | 12 月数据（多版本） |

如果回测日期不在已有数据范围内，Agent 会自动调用 ingest 导入缺失数据。

---

## 流程五：对比回测

**适用场景**：对比自动化策略和自定义策略哪个更好

**触发方式**：
- "对比一下 AAPL 的两种策略"
- "自动策略和我的策略哪个好"

### 实际对比报告截取

以下是 TSLA/AAPL/NVDA/AMD 四股对比的实际报告内容：

```markdown
# 四股对比分析报告：TSLA / AAPL / NVDA / AMD

> 分析日期：2026-03-10 | 回测周期：2025-12-01 ~ 2025-12-31

## 决策汇总

| 股票 | AI 决策 | 回测收益率 | Sharpe | 最大回撤 | 交易次数 | 综合评价 |
|------|---------|-----------|--------|---------|---------|---------|
| TSLA | **SELL** | 0.00% | N/A | 0.00% | 0 | 坚决回避 |
| AAPL | **SELL** | 0.00% | N/A | 0.00% | 0 | 谨慎回避 |
| NVDA | **SELL** | 0.00% | N/A | 0.00% | 0 | 短期回避 |
| AMD | **HOLD** | 0.00% | N/A | 0.00% | 0 | 等待入场 |

## AI 分析对比

| 维度 | TSLA | AAPL | NVDA | AMD |
|------|------|------|------|-----|
| 技术面 | 强烈看空：死亡交叉迫近 | 看空：均线空头+布林压缩 | 中性偏空 | 中性偏多：金叉完好 |
| 基本面 | 极度看空：P/E 369x | 中性偏空：P/E 33x | 缺失 | 分歧：PEG 0.60 |
| 情绪面 | 看空 | 中性偏空 | 看空（散户陷阱） | 中性 |
| 新闻面 | 看空：FSD 危机 | 看空：AI 延迟 | 强烈看空：台湾风险 | 中性偏多 |

## 排名（按综合风险调整评价）

1. **AMD** (HOLD) — 唯一非 SELL。PEG 0.60 暗示低估，金叉完好
2. **AAPL** (SELL) — 基本面最稳健（毛利率 48.2%），但估值偏高
3. **NVDA** (SELL) — 长期趋势完好，但地缘政治风险是黑天鹅
4. **TSLA** (SELL) — 基本面最差（P/E 369x + 利润率 4.6%），风险最高

## 组合建议

### 保守方案（推荐）
| 资产 | 比例 | 理由 |
|------|------|------|
| 现金/短期国债 | 60% | 滞胀环境下的避风港 |
| AMD | 20% | 唯一 HOLD 信号，等待 $190-200 入场 |
| AAPL | 10% | 基本面韧性，布林带突破上轨后加仓 |
| NVDA | 10% | 长期 AI 核心，200 SMA 附近小仓位 |
| TSLA | 0% | 完全回避 |
```

---

## 流程六：模拟盘交易

**适用场景**：基于分析执行模拟盘交易，或直接下单

### 直接交易命令

| 操作 | 自然语言触发 | 命令格式（需安装） |
|------|-------------|-------------------|
| 市价买入 | "买入 1 股 AAPL" | `/stock-agent buy AAPL 1` |
| 市价卖出 | "卖出 AAPL" | `/stock-agent sell AAPL 1` |
| 限价买入 | "在 250 限价买 AAPL" | `/stock-agent limit buy AAPL 1 250` |
| 查看报价 | "AAPL 现在多少钱" | `/stock-agent quote AAPL` |
| 查看持仓 | "看看我的持仓" | `/stock-agent positions` |
| 查看账户 | "账户余额多少" | `/stock-agent portfolio` |
| 取消订单 | "取消订单 xxx" | `/stock-agent cancel <order_id>` |
| 平仓 | "平掉 AAPL" | `/stock-agent close AAPL` |
| 全部平仓 | "全部平仓" | `/stock-agent close all` |

### 账户数据（实际测试）

```json
{
  "status": "ACTIVE",
  "buying_power": "199245.49",
  "equity": "100006.81",
  "cash": "99238.68",
  "portfolio_value": "100006.81",
  "long_market_value": "768.13",
  "pattern_day_trader": false,
  "shorting_enabled": true
}
```

### 持仓数据（实际测试）

```json
[
  {
    "symbol": "AAPL",
    "qty": "1",
    "side": "long",
    "avg_entry_price": "261.32",
    "current_price": "260.65",
    "market_value": "260.65",
    "unrealized_pl": "-0.67",
    "unrealized_plpc": "-0.00256",
    "change_today": "-0.00061"
  },
  {
    "symbol": "TSLA",
    "qty": "1.24331717",
    "side": "long",
    "avg_entry_price": "402.15",
    "current_price": "408.17",
    "market_value": "507.48",
    "unrealized_pl": "7.48",
    "unrealized_plpc": "0.01497",
    "change_today": "0.00086"
  }
]
```

### 报价数据（实际测试）

```json
{
  "AAPL": {
    "symbol": "AAPL",
    "bid_price": 247.96,
    "ask_price": 274.36,
    "bid_size": 100.0,
    "ask_size": 100.0,
    "timestamp": "2026-03-11T20:00:00.011033Z"
  }
}
```

### 安全规则

1. **所有下单必须二次确认**：Agent 先展示订单详情，等你说"确认"才执行
2. **分析方向冲突时强制警告**：双引擎意见不一致会明确提示风险
3. **全部平仓需额外确认**：会列出所有将被平仓的持仓让你确认
4. **不支持做空**：Alpaca 模拟盘限制，short/cover 操作会提示

### 仓位计算建议（基于 AI 分析）

| 置信度 | 建议投入比例 |
|--------|------------|
| >= 80% | 购买力的 20% |
| 60-79% | 购买力的 10% |
| 40-59% | 购买力的 5% |
| < 40% | 不建议交易 |

---

## 辅助查询

直接用自然语言即可，无需记忆命令：

| 自然语言 | 功能 |
|----------|------|
| "AAPL 多少钱" | 实时报价 |
| "看看我的账户" | 账户 + 持仓概览 |
| "有哪些持仓" | 持仓列表 |
| "看看我的订单" | 订单列表 |
| "有哪些大师" | 可用投资大师列表 |
| "有哪些数据" | 回测引擎已导入数据 |
| "有哪些策略" | 回测引擎已上传策略 |
| "分析到哪了" | 查询任务进度 |

### 18 位可用投资大师

#### 投资大师（12 位）

##### 1. Warren Buffett（沃伦·巴菲特）

- **API Key**：`warren_buffett`
- **流派**：价值投资
- **核心理念**：以合理价格买入具有持久竞争优势（"护城河"）的优质企业，长期持有。强调 ROE、自由现金流和管理层质量。
- **关注指标**：ROE、净利润率、内在价值 vs 市价、护城河宽度
- **典型信号**：
  > *"Wonderful business with durable moat and excellent ROE, but trading far above intrinsic value. Price matters."* — AAPL 分析
  >
  > *"Weak ROE 4.8%, negative earnings growth, limited moat. Trading at 70x intrinsic value — massive overvaluation. Avoid."* — TSLA 分析（bearish 88%）

##### 2. Charlie Munger（查理·芒格）

- **API Key**：`charlie_munger`
- **流派**：多学科思维模型 + 价值投资
- **核心理念**：运用多学科思维框架（心理学、经济学、数学等）评估企业质量。追求简单、可预测的商业模式，强调"反过来想"。
- **关注指标**：安全边际、商业可预测性、管理层诚信、估值溢价
- **典型信号**：
  > *"No margin of safety, weak moat, unpredictable. Expensive at 95% premium. This is speculation, not investment."* — TSLA 分析（bearish 26%）

##### 3. Michael Burry（迈克尔·伯里）

- **API Key**：`michael_burry`
- **流派**：逆向投资 + 深度价值
- **核心理念**：以《大空头》闻名，擅长发现被市场忽视的极端高估或低估标的。重点关注自由现金流收益率（FCF Yield），偏好做空高估值泡沫。
- **关注指标**：FCF Yield、P/FCF、市值 vs 内在价值、泡沫信号
- **典型信号**：
  > *"FCF yield 0.4%. Absurd valuation. Market cap $1.5T with minimal free cash flow generation. Paying ~250x FCF."* — TSLA 分析（bearish 72%）

##### 4. Ben Graham（本杰明·格雷厄姆）

- **API Key**：`ben_graham`
- **流派**：价值投资鼻祖
- **核心理念**：《聪明的投资者》作者，巴菲特的老师。核心是"安全边际"——只在股价显著低于内在价值时买入。使用 Graham Number（格雷厄姆数字）作为估值基准。
- **关注指标**：Graham Number、P/E、P/B、安全边际百分比
- **典型信号**：
  > *"Tesla fails Graham's most fundamental criterion: margin of safety. Price $449.72 vs Graham Number $26.00 — 17x premium."* — TSLA 分析（bearish 90%）

##### 5. Peter Lynch（彼得·林奇）

- **API Key**：`peter_lynch`
- **流派**：成长型价值投资
- **核心理念**：麦哲伦基金传奇经理，主张"买你懂的公司"。用 PEG 比率（P/E 除以盈利增长率）衡量成长性与估值的匹配度，寻找"十倍股"（ten-bagger）。
- **关注指标**：PEG 比率、P/E、盈利增长率、业务可理解性
- **典型信号**：
  > *"P/E 394 with negative earnings growth — can't build a ten-bagger from here. Great company doesn't always make great stock."* — TSLA 分析（bearish 72%）

##### 6. Cathie Wood（凯西·伍德）

- **API Key**：`cathie_wood`
- **流派**：颠覆式创新投资
- **核心理念**：ARK Invest 创始人，专注于颠覆式技术（AI、自动驾驶、基因编辑、区块链等）。评估企业的创新得分和研发投入强度，愿意为高成长付出溢价。
- **关注指标**：颠覆性技术得分（0-12）、R&D/收入比、创新管线、市场增长潜力
- **典型信号**：
  > *"Disruptive tech score 0.42/12. R&D at 6.8% of revenue lags 15-25% typical. -84.77% margin of safety."* — TSLA 分析（bearish 68%）

##### 7. Bill Ackman（比尔·阿克曼）

- **API Key**：`bill_ackman`
- **流派**：激进主义价值投资
- **核心理念**：Pershing Square 创始人，以激进投资者身份著称。评估内在价值与市价差距，关注 ROE 和管理层效率，必要时推动公司变革。
- **关注指标**：内在价值 vs 市值、ROE、管理层质量、激进主义改善空间
- **典型信号**：
  > *"Paying $1.5T for a business with intrinsic value ~$105B is speculation. ROE 4.8% is pedestrian. No activism angle."* — TSLA 分析（bearish 88%）

##### 8. Stanley Druckenmiller（斯坦利·德鲁肯米勒）

- **API Key**：`stanley_druckenmiller`
- **流派**：宏观投资 + 趋势跟踪
- **核心理念**：索罗斯基金首席投资官出身，擅长把握宏观经济周期和大类资产轮动。结合宏观判断与技术面确认，愿意在高确信度时下重注。
- **关注指标**：P/E vs 盈利增长趋势、YoY 变化、宏观环境、下行风险估算
- **典型信号**：
  > *"P/E 394x with -10.9% YoY earnings decline. Paying growth premium for shrinking margins. Downside risk 40-50%."* — TSLA 分析（bearish 68%）

##### 9. Phil Fisher（菲利普·费雪）

- **API Key**：`phil_fisher`
- **流派**：成长股长期投资
- **核心理念**：《怎样选择成长股》作者，主张通过深度调研（"闲聊法"）发现优质成长企业。关注毛利率趋势、运营效率和长期竞争力。
- **关注指标**：毛利率、营业利润率趋势、ROE、P/E、P/FCF
- **典型信号**：
  > *"Gross margins 18%, operating margins declining. ROE 4.6% inadequate. P/E 394 and P/FCF 240 unjustifiable."* — TSLA 分析（bearish 72%）

##### 10. Mohnish Pabrai（莫尼什·帕布莱）

- **API Key**：`mohnish_pabrai`
- **流派**：克隆投资 + 不对称风险
- **核心理念**：巴菲特忠实追随者，著有《The Dhandho Investor》。核心哲学是"正面我赢，反面我不会输太多"（heads I win, tails I don't lose much）。专注于高 FCF 收益率的深度价值标的。
- **关注指标**：FCF Yield、标准化 P/FCF、风险不对称性
- **典型信号**：
  > *"0.35% FCF yield, paying ~280x normalized FCF. Opposite of 'heads I win, tails I don't lose much.' Pass decisively."* — TSLA 分析（bearish 72%）

##### 11. Aswath Damodaran（阿斯沃斯·达摩达兰）

- **API Key**：`aswath_damodaran`
- **流派**：估值学术派
- **核心理念**：纽约大学教授，被誉为"估值院长"（Dean of Valuation）。用严谨的 DCF 模型和统计方法为企业定价，关注盈利质量与现金流可持续性。
- **关注指标**：P/E、盈利增长率、FCF Yield、估值合理性（是否 priced to perfection）
- **典型信号**：
  > *"P/E 394x with -25% earnings growth. FCF yield 0.4%. Classic 'priced to perfection' with deteriorating fundamentals."* — TSLA 分析（bearish 75%）

##### 12. Rakesh Jhunjhunwala（拉凯什·金君瓦拉）

- **API Key**：`rakesh_jhunjhunwala`
- **流派**：新兴市场成长投资
- **核心理念**：印度"股神"，擅长在新兴市场发现高增长机会。关注安全边际、收入增长趋势（CAGR）和 ROE，偏好在内在价值被严重低估时大胆建仓。
- **关注指标**：安全边际、内在价值 vs 市值、Revenue CAGR、ROE
- **典型信号**：
  > *"Margin of safety NEGATIVE 96.6%. Intrinsic value ~$50B vs market $1.5T. Revenue CAGR negative. ROE just 4.6%."* — TSLA 分析（bearish 88%）

---

#### 专业分析师（6 位）

##### 13. Technical Analyst（技术分析师）

- **API Key**：`technical_analyst`
- **专长**：纯技术图表分析，不考虑基本面
- **分析维度**：趋势跟踪（ADX）、均值回归（RSI/布林带）、动量（价量）、波动率（ATR/Historical Vol）、统计套利（Hurst 指数）
- **输出格式**：五维度独立信号 + 置信度 + 关键指标值
- **典型信号**：
  > | 类别 | 信号 | 置信度 | 关键指标 |
  > |------|------|--------|----------|
  > | Trend Following | bullish | 40% | ADX: 39.94 |
  > | Mean Reversion | neutral | 50% | RSI(14): 49.36, BB Position: 0.28 |
  > | Momentum | neutral | 50% | 1M Momentum: +5.0% |
  > | Volatility | neutral | 50% | Historical Vol: 38.5% |
  > | Statistical Arbitrage | neutral | 50% | Hurst: ~0 |

##### 14. Fundamentals Analyst（基本面分析师）

- **API Key**：`fundamentals_analyst`
- **专长**：财务报表深度分析
- **分析维度**：盈利能力（ROE/净利润率）、成长性（收入/盈利增长率）、估值水平（P/E/P/B）、财务健康（负债率）
- **典型信号**：
  > *"ROE 4.80%, Net Margin 4.00%, Revenue Growth -0.84%, Earnings Growth -25.34%. P/E 394.22, P/B 18.21."* — TSLA 分析（bearish 75%）

##### 15. Growth Analyst（成长性分析师）

- **API Key**：`growth_analyst`
- **专长**：企业成长潜力评估
- **分析维度**：收入增长趋势、EPS 增长率、内部人交易方向（insider flow ratio）、财务健康评分
- **典型信号**：
  > *"Revenue growth negative, EPS declining, insider net flow ratio -0.72. Only financial health scores well (D/E 0.67)."* — TSLA 分析（bearish 21%）

##### 16. Valuation Analyst（估值分析师）

- **API Key**：`valuation_analyst`
- **专长**：多方法估值交叉验证
- **分析方法**：DCF 折现现金流（权重 35%）、Owner Earnings 所有者盈利（权重 35%）、EV/EBITDA 相对估值（权重 20%）
- **输出格式**：每种方法的估值 vs 市值 + 差距百分比
- **典型信号**：
  > | 方法 | 估值 | 市值 | 差距 | 权重 |
  > |------|------|------|------|------|
  > | DCF 分析 | $43.0B | $1,495.7B | -97.1% | 35% |
  > | Owner Earnings | $20.7B | $1,495.7B | -98.6% | 35% |
  > | EV/EBITDA | $806.9B | $1,495.7B | -46.0% | 20% |
  >
  > *"DCF value $43B vs market cap $1.5T (-97.1%). Massively overvalued by all methods."* — TSLA（bearish 100%）

##### 17. Sentiment Analyst（情绪分析师）

- **API Key**：`sentiment_analyst`
- **专长**：市场情绪与内部人交易分析
- **数据源**：内部人买卖交易记录（insider trading）、机构持仓变动
- **关注指标**：看多/看空交易笔数、净加权信号强度
- **典型信号**：
  > *"Insider trading heavily bearish: 880/1000 bearish trades. Net weighted signal strongly bearish."* — TSLA 分析（bearish 88%）

##### 18. News Sentiment Analyst（新闻情绪分析师）

- **API Key**：`news_sentiment_analyst`
- **专长**：新闻舆情分析
- **数据源**：财经新闻、公告、媒体报道
- **分析方式**：对分析期间内的新闻进行情绪打分和趋势判断
- **典型信号**：
  > *"No news articles found in analysis period. Signal neutral by default."* — TSLA 分析（neutral 0%）
  >
  > 注意：当分析期间无相关新闻时，默认输出中性信号。

---

#### 大师信号汇总示例（TSLA 实测）

| # | 大师/分析师 | 信号 | 置信度 | 一句话理由 |
|---|------------|------|--------|-----------|
| 1 | Warren Buffett | bearish | 88% | ROE 4.8%，70 倍内在价值 |
| 2 | Charlie Munger | bearish | 26% | 无安全边际，不可预测 |
| 3 | Michael Burry | bearish | 72% | FCF Yield 0.4%，250 倍 FCF |
| 4 | Ben Graham | bearish | 90% | 价格 $450 vs Graham Number $26 |
| 5 | Peter Lynch | bearish | 72% | P/E 394，盈利负增长 |
| 6 | Cathie Wood | bearish | 68% | 颠覆性得分 0.42/12 |
| 7 | Bill Ackman | bearish | 88% | 市值 $1.5T vs 内在价值 $105B |
| 8 | Stanley Druckenmiller | bearish | 68% | 盈利 YoY -10.9%，下行风险 40-50% |
| 9 | Phil Fisher | bearish | 72% | 毛利率 18%，ROE 4.6% |
| 10 | Mohnish Pabrai | bearish | 72% | 280 倍标准化 FCF |
| 11 | Aswath Damodaran | bearish | 75% | "priced to perfection" + 基本面恶化 |
| 12 | Rakesh Jhunjhunwala | bearish | 88% | 安全边际 -96.6% |
| 13 | Technical Analyst | **bullish** | 21% | ADX 39.9 趋势看多（唯一看多） |
| 14 | Fundamentals Analyst | bearish | 75% | ROE/净利率/增长率全面疲弱 |
| 15 | Growth Analyst | bearish | 21% | 收入负增长，内部人净卖出 |
| 16 | Valuation Analyst | bearish | 100% | DCF 估值仅市值 2.9% |
| 17 | Sentiment Analyst | bearish | 88% | 内部人 880/1000 笔看空交易 |
| 18 | News Sentiment | neutral | 0% | 无新闻数据 |

---

## 服务状态说明

### 四个后端服务

| 服务 | 端口 | 响应时间 | 说明 |
|------|------|---------|------|
| AI Hedge Fund | 8082 | 分析 ~30s，回测按交易日数 | 速度快，量化信号 |
| TradingAgents | 8080 | 3-10 分钟 | 慢但深度，文本报告 |
| 回测引擎 | 8090 | 几秒到几分钟 | 取决于策略复杂度和时间跨度 |
| Alpaca Trading | 8083 | 实时 | 模拟盘交易执行 |

### 已知限制

| 限制 | 说明 |
|------|------|
| TradingAgents 单股票 | 一次只能分析一只股票，多股需串行提交 |
| AI Hedge Fund 支持多股 | 可一次传入多个 tickers |
| K 线接口 (bars) | 当前返回 500 错误，暂不可用 |
| 做空限制 | Alpaca 模拟盘不支持 short/cover |
| 策略兼容性 | 上传的策略需匹配回测引擎的数据格式 |
| 市场时间 | 美股交易时间 9:30-16:00 ET |

### 多股票并行

所有流程支持多股票，如 "分析 AAPL NVDA TSLA"，每只股票独立执行。

进度汇报格式：
```
| 股票 | 状态   | 进度 | 当前阶段   |
|------|--------|------|-----------|
| AAPL | 运行中 | 60%  | 基本面分析 |
| NVDA | 已完成 | 100% | BUY       |
| TSLA | 运行中 | 30%  | 技术分析   |
```

### 报告保存位置

所有报告保存到 `./reports/<TICKER>/<日期>/`：

```
reports/
├── AAPL/
│   └── 2025-12/
│       ├── 01_analysis_full.md          # TradingAgents 深度报告（~110KB）
│       ├── 01_analysis_summary.md       # 双引擎合并摘要
│       ├── 02_strategy.py               # 策略代码
│       ├── 02_strategy_notes.md         # 策略设计说明
│       ├── 03_backtest.md               # 自定义策略回测
│       └── 04_final_report.md           # 综合报告
├── TSLA/
│   └── 2025-12/
│       ├── 01_analysis_hedgefund.md     # AI Hedge Fund 大师信号
│       ├── 01_analysis_full.md          # TradingAgents 深度报告（~119KB）
│       ├── 01_analysis_summary.md       # 合并摘要
│       ├── 02_strategy.py / notes.md    # 策略
│       ├── 03_backtest_auto.md          # 自动化回测（流程一）
│       ├── 03_backtest.md               # 自定义回测（流程四）
│       └── 04_final_report.md           # 综合报告
├── NVDA/ AMD/                           # 同样结构
└── _comparison/
    └── 2025-12/
        └── comparison.md               # 多股票对比报告
```

---

## 常见问题

### Q: 分析太慢了怎么办？
TradingAgents 需要 3-10 分钟。如果只需要快速查看大师信号，可以用 `analyze AAPL --masters`（~30 秒）。

### Q: 回测收益全为 0？
两种可能：
1. **策略没有触发交易**（预期行为）：策略入场条件在回测期间未被满足。如 AI 判定 SELL，防守型策略全程持现金。
2. **策略代码有 bug**：检查回测结果中的 `errors` 字段。常见 `KeyError` 表示数据包与策略不兼容。

### Q: 下单被拒？
检查：
- 购买力是否充足（`portfolio` 查看）
- 美股市场是否开盘（东部时间 9:30-16:00）
- 标的是否可交易

### Q: 如何自定义分析的大师？
指定 analysts 参数：
- "用巴菲特和 Burry 分析 AAPL"

### Q: 回测时间范围怎么指定？
- "回测 AAPL 12 月表现" → 自动设为 2025-12-01 ~ 2025-12-31
- "回测 AAPL 最近 3 个月" → 自动计算
- 默认：最近 3 个月
