# 策略生成指南

本指南仅适用于**流程二**（自定义策略 + 回测引擎 回测）。流程一的全自动回测由 AI Hedge Fund 的 LLM 动态决策，不需要手写策略。

## 核心原则

策略的目的是将 AI 分析转化为可回测的交易信号。关键在于：策略必须在回测区间内产生实际交易，否则无法验证 AI 判断的正确性。

入场条件需要合理、在 22 个交易日内大概率触发——不能用极端阈值（如 RSI<30）让策略全程空仓。

---

## 输入来源：双引擎分析

流程二使用双引擎并行分析，策略设计应综合两个来源：

| 来源 | 提供什么 | 用在策略哪里 |
|------|---------|-------------|
| **TradingAgents** | 具体价位（支撑/阻力）、均线周期、RSI 数值 | 入场/退出/止损的具体价格参数 |
| **AI Hedge Fund** | 大师信号共识（bullish/bearish/neutral）+ confidence | 决定仓位大小和策略激进程度 |

**综合示例**：
- TradingAgents: "200 SMA $189，RSI 37→回升，布林带下轨 $195"
- AI Hedge Fund: "巴菲特 neutral 50%，技术面 bearish 24%，Michael Burry bearish 70%"
- → 策略：$195 入场但仅 20%（大师共识偏空所以轻仓），$189 止损（200 SMA 下方）

**共识度→仓位映射**：
- 大师多数 bullish（≥60% 看多） → 按 BUY 策略仓位
- 大师分歧（无明确倾向） → 按 HOLD 策略仓位
- 大师多数 bearish（≥60% 看空） → 按 SELL 策略仓位

---

## 策略方向映射

AI 决策不是"买或不买"那么简单。每种信号都应转化为不同风格的交易策略：

| AI 决策 | 策略风格 | 典型逻辑 | 仓位范围 |
|---------|---------|---------|---------|
| BUY（强烈） | 激进做多 | 开盘即建仓 60-80%，均线金叉加仓至满仓，RSI>70 或跟踪止损退出 | 60-100% |
| BUY（谨慎） | 稳健做多 | 回调到支撑位入场 40-60%，反弹后逐步获利了结，均线死叉清仓 | 40-60% |
| HOLD | 双向波段 | 布林带下轨买入 30-50%，中轨卖出；支撑买入阻力卖出 | 30-50% |
| SELL（谨慎） | 防守做多 | 短期均线向上穿越时小仓位做多 20-30%，均线回落立即止损 | 20-30% |
| SELL（强烈） | 仅抄底反弹 | RSI<35 超卖时入场 15-20%，RSI>50 迅速获利了结，3% 严格止损 | 15-20% |

为什么 SELL 也要做多？因为 回测引擎 不支持做空，且即使 AI 看空，月度区间内也常有反弹波段。设计一个能捕捉反弹的低仓位策略，比全程空仓更有验证价值。

---

## 策略代码框架

```python
import datetime
import numpy as np
import structlog
from ziplime.finance.execution import MarketOrder

logger = structlog.get_logger(__name__)

# ── 双引擎分析摘要 ──
# TradingAgents Decision: {BUY/SELL/HOLD}
# AI Hedge Fund Consensus: {X/18 bullish, Y/18 bearish}
# Analysis date: {date}
# Key signals: {具体价格和技术指标数值}

async def initialize(context):
    context.asset = await context.symbol("{TICKER}")
    context.stop_loss_pct = {止损百分比}
    context.take_profit_pct = {止盈百分比}
    context.position_size = {仓位比例}       # 综合双引擎共识决定
    context.short_window = {短期均线, 5-10}
    context.long_window = {长期均线, 20-50}
    context.entry_price = None
    context.traded_today = False

async def handle_data(context, data):
    asset = context.asset
    price_df = data.history(assets=[asset], fields=["close"], bar_count=context.long_window)
    prices = price_df["close"].to_numpy()

    if len(prices) < context.long_window:
        return

    current_price = prices[-1]
    current_amount = getattr(context.portfolio.positions.get(asset, 0), 'amount', 0)
    has_position = current_amount > 0

    # ── 核心逻辑（根据双引擎分析动态生成）──
    # 计算技术指标（均线、RSI、布林带等）
    # 判断入场/退出/止损条件
    # 调用 context.order_target_percent(asset, target=X, style=MarketOrder())
```

---

## 回测引擎 API 速查

| API | 用法 | 说明 |
|-----|------|------|
| `await context.symbol("AAPL")` | 获取资产对象 | 在 initialize 中 |
| `data.history(assets=[asset], fields=["close"], bar_count=N)` | 历史数据 | 返回 DataFrame |
| `await context.order_target_percent(asset, target=0.5, style=MarketOrder())` | 按比例下单 | 0=清仓，0.5=50% |
| `context.portfolio.positions` | 当前持仓 | dict |
| `context.portfolio.cash` | 可用现金 | float |
| `context.portfolio.portfolio_value` | 总资产 | float |

---

## 生成原则

1. **忠实于双引擎分析**：TradingAgents 的具体价位 + AI Hedge Fund 的共识度，不要凭空编造
2. **均线交叉是基础信号**：短期（5-10 日）上穿长期（20-50 日）= 买入，下穿 = 卖出
3. **必须包含止损和止盈**：宽度匹配策略风格（激进 2-3%，稳健 5-8%）
4. **代码 100 行以内**，注释标注逻辑来源

---

## 生成后自检

- [ ] 入场条件在 22 个交易日内大概率触发？
- [ ] 仓位大小是否反映了大师共识度？
- [ ] 有止损和止盈？
- [ ] 代码语法正确？（async def, await, MarketOrder()）
- [ ] 注释中标注了双引擎分析的关键数据来源？
