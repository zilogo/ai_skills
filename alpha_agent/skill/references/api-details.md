# RD-Agent API 详细文档

## 服务总览

| 项目 | 值 |
|------|------|
| 服务 | RD-Agent API |
| 地址 | `http://172.16.13.58:8085` |
| Token | `ayS92YuxbW5dVfj6hcXLdTnM7RJGZbylbi7sNcOCAI0` |
| 认证方式 | `Authorization: Bearer <Token>` |
| 版本 | 0.1.0 |

---

## 端点列表

| # | 方法 | 路径 | 说明 | 认证 |
|---|------|------|------|------|
| 1 | GET | /api/v1/health | 健康检查 | 免认证 |
| 2 | GET | /api/v1/scenarios | 可用场景列表 | Bearer Token |
| 3 | POST | /api/v1/tasks | 提交研发任务 | Bearer Token |
| 4 | GET | /api/v1/tasks | 列出所有任务 | Bearer Token |
| 5 | GET | /api/v1/tasks/{task_id} | 查询任务状态 | Bearer Token |
| 6 | POST | /api/v1/tasks/{task_id}/cancel | 取消任务 | Bearer Token |
| 7 | GET | /api/v1/factors | 列出已发现因子 | Bearer Token |
| 8 | GET | /api/v1/runs | 列出运行历史 | Bearer Token |
| 9 | GET | /api/v1/runs/{run_id}/metrics | 运行指标详情 | Bearer Token |

---

## 1. GET /api/v1/health — 健康检查

```bash
curl -s http://172.16.13.58:8085/api/v1/health
```

**返回**：

```json
{
  "status": "ok",
  "service": "RD-Agent API",
  "version": "0.1.0"
}
```

---

## 2. GET /api/v1/scenarios — 可用场景列表

```bash
curl -s http://172.16.13.58:8085/api/v1/scenarios \
  -H "Authorization: Bearer ayS92YuxbW5dVfj6hcXLdTnM7RJGZbylbi7sNcOCAI0"
```

**返回**：

```json
[
  {
    "name": "fin_factor",
    "description": "Factor mining and evolution — discovers alpha factors via hypothesis-code-backtest loop",
    "cli_command": "rdagent fin_factor"
  },
  {
    "name": "fin_model",
    "description": "Model evolution — evolves ML models for stock prediction",
    "cli_command": "rdagent fin_model"
  },
  {
    "name": "fin_quant",
    "description": "Full quantitative pipeline — combines factor mining and model evolution",
    "cli_command": "rdagent fin_quant"
  }
]
```

---

## 3. POST /api/v1/tasks — 提交研发任务

```bash
curl -s -X POST http://172.16.13.58:8085/api/v1/tasks \
  -H "Authorization: Bearer ayS92YuxbW5dVfj6hcXLdTnM7RJGZbylbi7sNcOCAI0" \
  -H "Content-Type: application/json" \
  -d '{
    "scenario": "fin_factor",
    "loop_n": 3,
    "step_n": 1,
    "all_duration": 3600
  }'
```

**请求参数**：

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| scenario | string | 是 | — | 场景类型：`fin_factor` / `fin_model` / `fin_quant` |
| loop_n | int | 否 | 3 | 研发循环轮数（每轮包含假设→代码→回测） |
| step_n | int | 否 | — | 每轮步骤数（不传由服务决定） |
| all_duration | int | 否 | — | 最大运行时长（秒），超时自动停止 |

**返回**：

```json
{
  "task_id": "6b74768f59c8498fbc9641a1e2f39416",
  "status": "pending",
  "message": "Task submitted for fin_factor. Use GET /api/v1/tasks/6b74768f59c8498fbc9641a1e2f39416 to check progress."
}
```

---

## 4. GET /api/v1/tasks — 列出所有任务

```bash
curl -s http://172.16.13.58:8085/api/v1/tasks \
  -H "Authorization: Bearer ayS92YuxbW5dVfj6hcXLdTnM7RJGZbylbi7sNcOCAI0"
```

**返回**：

```json
{
  "tasks": [
    {
      "task_id": "473ad6afcadf41d189ecbc7b13ab494a",
      "scenario": "fin_factor",
      "status": "completed",
      "progress": 1.0,
      "current_step": "direct_exp_gen",
      "current_step_label": "假设生成",
      "loop_index": 1,
      "step_index": 0,
      "created_at": "2026-03-14T03:56:08.899490+00:00",
      "completed_at": "2026-03-14T04:00:56.959428+00:00",
      "result": null,
      "error": null
    }
  ],
  "total": 1
}
```

---

## 5. GET /api/v1/tasks/{task_id} — 查询任务状态

```bash
curl -s http://172.16.13.58:8085/api/v1/tasks/473ad6afcadf41d189ecbc7b13ab494a \
  -H "Authorization: Bearer ayS92YuxbW5dVfj6hcXLdTnM7RJGZbylbi7sNcOCAI0"
```

**返回（TaskStatusResponse）**：

```json
{
  "task_id": "473ad6afcadf41d189ecbc7b13ab494a",
  "scenario": "fin_factor",
  "status": "completed",
  "progress": 1.0,
  "current_step": "direct_exp_gen",
  "current_step_label": "假设生成",
  "loop_index": 1,
  "step_index": 0,
  "created_at": "2026-03-14T03:56:08.899490+00:00",
  "completed_at": "2026-03-14T04:00:56.959428+00:00",
  "result": {
    "run_id": "2026-03-14_03-56-31-970022",
    "loops": []
  },
  "error": null
}
```

**字段说明**：

| 字段 | 类型 | 说明 |
|------|------|------|
| task_id | string | 任务唯一 ID |
| scenario | string | 场景类型 |
| status | string | 状态：`pending` / `running` / `completed` / `failed` / `cancelled` / `cancelling` |
| progress | float | 进度 0.0~1.0 |
| current_step | string | 当前步骤 ID |
| current_step_label | string | 当前步骤中文名（如 "假设生成"） |
| loop_index | int | 当前循环索引（从 0 开始） |
| step_index | int | 当前步骤索引 |
| created_at | string | 创建时间 ISO 8601 |
| completed_at | string | 完成时间 ISO 8601（null 表示未完成） |
| result | object | 结果对象，含 `run_id` 和 `loops` |
| error | string | 错误信息（null 表示无错误） |

**任务状态机**：

```
pending → running → completed
                  → failed
                  → cancelling → cancelled
```

---

## 6. POST /api/v1/tasks/{task_id}/cancel — 取消任务

```bash
curl -s -X POST http://172.16.13.58:8085/api/v1/tasks/6b74768f59c8498fbc9641a1e2f39416/cancel \
  -H "Authorization: Bearer ayS92YuxbW5dVfj6hcXLdTnM7RJGZbylbi7sNcOCAI0"
```

**返回**：

```json
{
  "task_id": "6b74768f59c8498fbc9641a1e2f39416",
  "status": "cancelling",
  "message": "Cancel signal sent. Task will stop after current step."
}
```

取消是异步的，需再次轮询 `GET /tasks/{task_id}` 确认最终状态 `cancelled`。

---

## 7. GET /api/v1/factors — 列出已发现因子

```bash
curl -s http://172.16.13.58:8085/api/v1/factors \
  -H "Authorization: Bearer ayS92YuxbW5dVfj6hcXLdTnM7RJGZbylbi7sNcOCAI0"
```

**返回**：

```json
[
  {
    "name": "price_momentum_10d",
    "formulation": "\\frac{close_t - close_{t-10}}{close_{t-10}}",
    "description": "[Momentum Factor] 10-day price momentum calculated as the percentage change in closing price over the past 10 trading days",
    "implementation": "True",
    "run_id": "2026-03-14_03-56-09-088906",
    "loop_index": 0
  }
]
```

**字段说明**：

| 字段 | 类型 | 说明 |
|------|------|------|
| name | string | 因子名称 |
| formulation | string | 因子公式（LaTeX 格式） |
| description | string | 因子描述，含类型标签如 `[Momentum Factor]` |
| implementation | string | 是否已实现（"True" / "False"） |
| run_id | string | 产出该因子的运行 ID |
| loop_index | int | 产出该因子的循环索引 |

因子列表是全局累积的，包含所有历史运行产出的全部因子。

---

## 8. GET /api/v1/runs — 列出运行历史

```bash
curl -s http://172.16.13.58:8085/api/v1/runs \
  -H "Authorization: Bearer ayS92YuxbW5dVfj6hcXLdTnM7RJGZbylbi7sNcOCAI0"
```

**返回**：

```json
[
  {
    "run_id": "2026-03-14_03-56-09-088906",
    "started_at": "2026-03-14T03:56:09-088906",
    "loop_count": 1,
    "has_results": true
  }
]
```

**字段说明**：

| 字段 | 类型 | 说明 |
|------|------|------|
| run_id | string | 运行 ID（格式：YYYY-MM-DD_HH-MM-SS-ffffff） |
| started_at | string | 启动时间 |
| loop_count | int | 已完成循环数 |
| has_results | bool | 是否有可查询的指标结果 |

---

## 9. GET /api/v1/runs/{run_id}/metrics — 运行指标详情

```bash
curl -s http://172.16.13.58:8085/api/v1/runs/2026-03-14_03-56-09-088906/metrics \
  -H "Authorization: Bearer ayS92YuxbW5dVfj6hcXLdTnM7RJGZbylbi7sNcOCAI0"
```

**返回**：

```json
{
  "run_id": "2026-03-14_03-56-09-088906",
  "loop_index": 0,
  "metrics": {
    "IC": 0.0312,
    "ICIR": 0.2537,
    "Rank IC": 0.0335,
    "Rank ICIR": 0.2670,
    "1day.excess_return_without_cost.mean": 0.000325,
    "1day.excess_return_without_cost.annualized_return": 0.0774,
    "1day.excess_return_without_cost.information_ratio": 0.8474,
    "1day.excess_return_without_cost.max_drawdown": -0.0736,
    "1day.excess_return_without_cost.std": 0.005918,
    "1day.excess_return_with_cost.mean": 0.000131,
    "1day.excess_return_with_cost.annualized_return": 0.0312,
    "1day.excess_return_with_cost.information_ratio": 0.3424,
    "1day.excess_return_with_cost.max_drawdown": -0.0897,
    "1day.excess_return_with_cost.std": 0.005915,
    "1day.pos": 0.0,
    "1day.pa": 0.0,
    "1day.ffr": 1.0,
    "l2.train": 0.9767,
    "l2.valid": 0.9965
  },
  "factors": [
    {
      "name": "price_momentum_10d",
      "formulation": "\\frac{close_t - close_{t-10}}{close_{t-10}}",
      "description": "[Momentum Factor] 10-day price momentum",
      "implementation": "True"
    }
  ]
}
```

**指标字段说明**：

| 指标 | 说明 |
|------|------|
| IC | 信息系数 — 因子预测能力 |
| ICIR | IC 信息比率 — 因子稳定性 |
| Rank IC | 秩相关 IC — 非线性预测能力 |
| Rank ICIR | 秩 IC 信息比率 — 非线性稳定性 |
| 1day.excess_return_without_cost.* | 日度超额收益（不含交易成本） |
| 1day.excess_return_with_cost.* | 日度超额收益（含交易成本） |
| *.annualized_return | 年化超额收益率 |
| *.information_ratio | 信息比率（年化超额收益/跟踪误差） |
| *.max_drawdown | 最大回撤 |
| *.std | 日收益标准差 |
| 1day.pos | 正收益天数占比 |
| 1day.pa | 策略 alpha |
| 1day.ffr | 因子覆盖率 |
| l2.train | L2 训练损失 |
| l2.valid | L2 验证损失 |

详细指标解读见 `rdagent/references/metrics-guide.md`。
