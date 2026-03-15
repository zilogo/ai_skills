# 报告模板

所有报告保存到 `./reports/<TICKER>/<日期>/`。

不同流程生成不同文件组合，见主文件「文件保存」章节。

---

## 01_analysis_hedgefund.md — AI Hedge Fund 信号报告

```markdown
# {TICKER} 投资大师信号分析

> 分析时间：{timestamp} | 模型：{model_name}

## 最终决策

**Action: {buy/sell/hold/short/cover}** | Confidence: {confidence}% | Quantity: {quantity}

Reasoning: {reasoning}

## 投资大师信号

| 大师 | 信号 | 置信度 | 核心理由 |
|------|------|--------|----------|
| Warren Buffett | bullish | 65% | {reasoning 摘要} |
| Michael Burry | bearish | 70% | {reasoning 摘要} |
| Technical Analyst | neutral | 14% | {reasoning 摘要} |
| ... | ... | ... | ... |

## 大师共识

- 看多：{N} 位 ({名单})
- 看空：{N} 位 ({名单})
- 中性：{N} 位 ({名单})
- **共识方向**：{bullish/bearish/neutral}（{看多数}/{总数}）

## 技术面详细指标（如有 technical_analyst）

| 维度 | 信号 | 置信度 | 关键指标 |
|------|------|--------|----------|
| 趋势跟踪 | bearish | 24% | ADX={adx} |
| 均值回归 | neutral | 50% | RSI_14={rsi}, Z-Score={z_score} |
| 动量 | neutral | 50% | 1M={momentum_1m} |
| 波动率 | neutral | 50% | 历史波动率={vol} |
```

---

## 01_analysis_full.md — TradingAgents 完整报告

```markdown
# {TICKER} AI 深度分析报告

> 分析日期：{date} | 决策：**{BUY/SELL/HOLD}** | 生成时间：{timestamp}

## 一、市场技术分析

{result.reports.market_report 原文}

## 二、社交媒体情绪分析

{result.reports.sentiment_report 原文}

## 三、新闻分析

{result.reports.news_report 原文}

## 四、基本面分析

{result.reports.fundamentals_report 原文}

## 五、投资辩论

### 多头论点
{result.investment_debate.bull_history 原文}

### 空头论点
{result.investment_debate.bear_history 原文}

### 评审裁决
{result.investment_debate.judge_decision 原文}

## 六、交易计划

{result.trader_investment_plan 原文}

## 七、风险评估与最终决策

{result.risk_debate 原文}

## 八、最终交易决策

{result.final_trade_decision 原文}
```

---

## 01_analysis_summary.md — 双引擎摘要

```markdown
# {TICKER} 分析摘要

## TradingAgents 结论

**决策：{BUY/SELL/HOLD}**

| 维度 | 核心结论 |
|------|----------|
| 技术面 | （2-3 句） |
| 情绪面 | （2-3 句） |
| 新闻面 | （2-3 句） |
| 基本面 | （2-3 句） |

**关键信号**：（3-5 个具体数值）

## AI Hedge Fund 大师共识

| 方向 | 大师 | 信号 | 置信度 |
|------|------|------|--------|
| 看多 | Warren Buffett | bullish | 65% |
| 看空 | Michael Burry | bearish | 70% |
| ... | ... | ... | ... |

**共识**：{N}/{total} 位大师看多，整体 {bullish/bearish/neutral}

## 综合判断

TradingAgents 说 {BUY/SELL/HOLD}，AI Hedge Fund {N}位大师看{多/空}。
（交叉验证分析：一致性评估和分歧原因）
```

流程一（仅 AI Hedge Fund）时，省略 TradingAgents 部分。

---

## 02_strategy.py — 策略代码

直接保存生成的 Python 策略代码。仅流程二生成。

---

## 02_strategy_notes.md — 策略说明

```markdown
# {TICKER} 策略设计说明

- **策略类型**：{均线交叉/趋势跟踪/均值回归/...}
- **基于分析**：TradingAgents + AI Hedge Fund 双引擎
- **TradingAgents 决策**：{BUY/SELL/HOLD}
- **大师共识**：{N}/{total} 看多

## 入场条件
- {条件1，标注来源}
- {条件2}

## 退出条件
- {条件1}
- {条件2}

## 止损逻辑
- {止损条件和具体数值}

## 仓位管理
- {仓位比例和调仓规则}
- 仓位决策依据：{大师共识度 → 仓位大小}

## 参数表
| 参数 | 值 | 来源 |
|------|-----|------|
| short_window | 10 | TradingAgents 技术分析 |
| long_window | 50 | TradingAgents 技术分析 |
| stop_loss_pct | 0.03 | 风险评估 |
| position_size | 0.3 | AI Hedge Fund 大师共识偏空 → 轻仓 |
```

仅流程二生成。

---

## 03_backtest_auto.md — AI Hedge Fund 自动化回测

```markdown
# {TICKER} 自动化回测报告

> 回测区间：{start} ~ {end} | 初始资金：${capital} | 参与大师：{agent_list}

## 核心指标

| 指标 | 数值 |
|------|------|
| 最终净值 | ${ending_value} |
| 总收益率 | {return}% |
| Sharpe 比率 | {sharpe} |
| Sortino 比率 | {sortino} |
| 最大回撤 | {max_drawdown}% |
| 多空比 | {long_short_ratio} |
| 交易天数 | {days} |

## 每日交易记录（关键日期）

| 日期 | 操作 | 数量 | 价格 | 组合价值 | 理由 |
|------|------|------|------|---------|------|
| {date} | buy | 100 | $150.2 | $100,150 | {LLM reasoning} |
| ... | ... | ... | ... | ... | ... |

## 最终持仓

| 股票 | 多头 | 空头 | 成本基础 | 已实现盈亏 |
|------|------|------|---------|-----------|
| {TICKER} | {long} | {short} | ${cost_basis} | ${realized} |
```

仅流程一和流程五生成。

---

## 03_backtest.md — 回测引擎 自定义策略回测

```markdown
# {TICKER} 回测结果报告

> 策略文件：{file} | 回测区间：{start} ~ {end} | 初始资金：${capital}

## 核心指标

| 指标 | 数值 |
|------|------|
| 最终净值 | ${ending_value} |
| 总收益率 | {return}% |
| Sharpe 比率 | {sharpe} |
| 最大回撤 | {max_drawdown}% |
| 交易天数 | {days} |
| 交易次数 | {trades} |

## 策略执行说明

{入场/退出/止损触发情况描述}
```

仅流程二、四、五生成。

---

## 04_final_report.md — 综合报告

```markdown
# {TICKER} 综合交易分析报告

> 生成时间：{timestamp} | 股票代码：{TICKER} | 分析周期：{period}

---

## 一、AI 分析结论

### TradingAgents（如有）
**决策：{BUY/SELL/HOLD}**
**关键信号**：{3-5 个具体数值}

### AI Hedge Fund
**Action: {action}** | 大师共识：{N}/{total} 看{多/空}

| 维度 | 方向 | 核心观点 |
|------|------|----------|
| 技术面 | ... | ... |
| 大师共识 | ... | ... |

---

## 二、回测验证

### 自动化回测（AI Hedge Fund，如有）
| 指标 | 数值 |
|------|------|
| 收益率 | {return}% |
| Sharpe | {sharpe} |
| 最大回撤 | {drawdown}% |

### 自定义策略回测（回测引擎，如有）
| 指标 | 数值 |
|------|------|
| 收益率 | {return}% |
| Sharpe | {sharpe} |
| 最大回撤 | {drawdown}% |

---

## 三、策略设计（仅流程二）

- **类型**：{类型}
- **入场**：{条件}
- **退出**：{条件}
- **止损**：{规则}
（完整代码见 `02_strategy.py`）

---

## 四、风险提示

- {风险1}
- {风险2}

---

## 五、建议操作

{综合分析 + 回测的最终建议}

---

## 附件

| 文件 | 说明 |
|------|------|
| （列出本次流程实际生成的所有文件） |
```

---

## 05_trade_execution.md — 交易执行记录

```markdown
# {TICKER} 交易执行记录

> 执行时间：{timestamp} | 关联分析：{report_date}

## 交易详情

| 项目 | 值 |
|------|-----|
| 标的 | {symbol} |
| 方向 | {buy/sell} |
| 数量 | {qty} |
| 订单类型 | {market/limit} |
| 订单状态 | {filled/accepted} |
| 成交价格 | ${price} |
| 成交金额 | ${amount} |

## AI 分析依据

- AI Hedge Fund: {action} (confidence {N}%)
- TradingAgents: {BUY/SELL/HOLD}（如有）
- 大师共识: {N}/{total} 看{多/空}

## 执行前后账户

| 项目 | 执行前 | 执行后 |
|------|--------|--------|
| 购买力 | ${before} | ${after} |
| 总权益 | ${before} | ${after} |
```

仅流程六生成。

---

## 多股票对比报告

保存至 `./reports/_comparison/<日期>/comparison.md`：

```markdown
# 多股票对比分析

> 分析日期：{date} | 股票：{TICKER1}, {TICKER2}, ...

## 决策汇总

| 股票 | TradingAgents | AI Hedge Fund | 大师共识 | 自动回测收益 | 自定义回测收益 |
|------|:---:|:---:|:---:|:---:|:---:|
| AAPL | BUY | buy 75% | 12/18 看多 | +5.2% | +3.1% |

## 排名
1. {最优} — 理由

## 组合建议
{仓位分配建议}
```
