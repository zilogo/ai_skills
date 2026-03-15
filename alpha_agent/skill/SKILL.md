---
description: 'Alpha Agent：AI Alpha 因子挖掘与模型进化工具。当用户提到因子挖掘、因子研究、alpha 因子、IC/ICIR、模型进化、模型训练、量化研究、端到端量化、fin_factor、fin_model、fin_quant、alpha-agent、挖因子、训练模型、因子表现、假设验证、因子演化等话题时触发。也适用于"跑一轮因子"、"看看因子指标"、"启动量化研究"、"查看任务状态"等日常表述。'
---

$ARGUMENTS

# Alpha Agent — AI Alpha 因子挖掘与模型进化

串联 RD-Agent API 服务，支持五种使用流程。根据用户意图自动选择最合适的流程。

## 服务配置

| 项目 | 值 |
|------|------|
| 地址 | `http://172.16.13.58:8085` |
| Token | `ayS92YuxbW5dVfj6hcXLdTnM7RJGZbylbi7sNcOCAI0` |
| 认证 | `Authorization: Bearer <Token>` |

完整 API 文档见 `alpha-agent/references/api-details.md`。

---

## 三大场景

| 场景 | CLI 命令 | 核心能力 |
|------|----------|----------|
| **fin_factor** | `alpha-agent fin_factor` | 因子挖掘与演化 — 假设→代码→回测循环，自动发现 alpha 因子 |
| **fin_model** | `alpha-agent fin_model` | 模型进化 — 演化 ML 模型用于股票预测 |
| **fin_quant** | `alpha-agent fin_quant` | 全量化流水线 — 因子挖掘 + 模型进化端到端 |

---

## 子命令与流程路由

| 用法 | 流程 | 说明 |
|------|------|------|
| `/alpha-agent factor` | 流程一 | 因子挖掘（fin_factor） |
| `/alpha-agent model` | 流程二 | 模型进化（fin_model） |
| `/alpha-agent quant` | 流程三 | 端到端量化（fin_quant） |
| `/alpha-agent status [task_id]` | 流程四 | 查看任务状态 |
| `/alpha-agent cancel <task_id>` | 流程四 | 取消任务 |
| `/alpha-agent tasks` | 流程四 | 列出所有任务 |
| `/alpha-agent factors` | 流程五 | 查看已发现因子 |
| `/alpha-agent runs` | 流程五 | 查看运行历史 |
| `/alpha-agent metrics <run_id>` | 流程五 | 查看运行指标 |
| `/alpha-agent scenarios` | — | 查看可用场景 |
| `/alpha-agent health` | — | 服务健康检查 |

**路由规则**：
- "挖因子"、"factor"、"因子研究"、"alpha" → 流程一
- "训练模型"、"model"、"模型进化"、"ML" → 流程二
- "量化研究"、"quant"、"端到端"、"全流水线" → 流程三
- "状态"、"进度"、"取消"、"任务列表" → 流程四
- "看因子"、"因子指标"、"历史"、"runs"、"IC" → 流程五
- 无明确意图 → 默认流程一（因子挖掘）

---

## 参数

从 `$ARGUMENTS` 灵活提取：

| 参数 | 说明 | 默认值 | 示例 |
|------|------|--------|------|
| scenario | 场景类型 | fin_factor | fin_model, fin_quant |
| loop_n | 研发循环轮数 | 3 | 1, 5, 10 |
| step_n | 每轮步骤数（可选） | 不传（由服务决定） | 1, 2 |
| all_duration | 最大运行时长秒数（可选） | 不传（无限制） | 3600 |

**参数提取规则**：
- "跑 5 轮因子" → loop_n=5, scenario=fin_factor
- "快速测试" → loop_n=1
- "限时 1 小时" → all_duration=3600

---

## 流程一：因子挖掘（fin_factor）

**场景**：用户想自动发现有效的 alpha 因子

```
health check → POST /tasks(fin_factor) → 轮询进度 → GET /factors → 报告
```

### 步骤

1. **健康检查**：`GET /api/v1/health`
   - 确认服务可用，不可达则报告并终止

2. **提交任务**：`POST /api/v1/tasks`
   ```json
   {
     "scenario": "fin_factor",
     "loop_n": 3
   }
   ```
   - 返回 `task_id` 和 `status: "pending"`
   - 向用户确认：任务已提交，ID 为 xxx

3. **轮询进度**：`GET /api/v1/tasks/{task_id}`，每 30 秒一次
   - 向用户报告：进度百分比 + 当前阶段（current_step_label）+ 当前循环/步骤
   - 进度报告格式：
     ```
     | 任务 | 进度 | 当前阶段 | 循环 |
     |------|------|----------|------|
     | {task_id} | 45% | 假设生成 | 2/3 |
     ```
   - 状态终态：`completed`、`failed`、`cancelled`

4. **获取因子**：任务完成后，`GET /api/v1/factors`
   - 展示新发现的因子列表（名称、公式、描述）

5. **获取指标**：从任务结果中提取 `run_id`，调用 `GET /api/v1/runs/{run_id}/metrics`
   - 展示 IC、ICIR、Rank IC、Rank ICIR 等核心指标
   - 指标解读参考 `alpha-agent/references/metrics-guide.md`

6. **保存报告**：写入 `reports/alpha-agent/{task_id}/`
   - `01_task_summary.md` — 任务概览（场景、参数、耗时）
   - `02_factors.md` — 因子列表（名称、公式、描述、实现状态）
   - `03_metrics.md` — 指标详情 + 解读
   - `04_report.md` — 综合报告

---

## 流程二：模型进化（fin_model）

**场景**：用户想演化 ML 模型用于股票预测

```
health check → POST /tasks(fin_model) → 轮询进度 → 报告
```

### 步骤

1. **健康检查**：同流程一

2. **提交任务**：`POST /api/v1/tasks`
   ```json
   {
     "scenario": "fin_model",
     "loop_n": 3
   }
   ```

3. **轮询进度**：同流程一，每 30 秒轮询
   - fin_model 通常耗时较长，提醒用户耐心等待

4. **获取结果**：从任务结果中提取 `run_id`，查看指标
   - `GET /api/v1/runs/{run_id}/metrics`
   - 重点关注 l2.train、l2.valid 等模型训练指标

5. **保存报告**：写入 `reports/alpha-agent/{task_id}/`
   - `01_task_summary.md` — 任务概览
   - `02_model_metrics.md` — 模型指标 + 解读
   - `03_report.md` — 综合报告

---

## 流程三：端到端量化（fin_quant）

**场景**：用户想执行完整的量化研究流水线（因子挖掘 + 模型进化）

```
health check → POST /tasks(fin_quant) → 轮询进度 → GET /factors → metrics → 报告
```

### 步骤

1. **健康检查**：同流程一

2. **提交任务**：`POST /api/v1/tasks`
   ```json
   {
     "scenario": "fin_quant",
     "loop_n": 3
   }
   ```
   - fin_quant 是最全面的场景，耗时最长

3. **轮询进度**：同流程一，每 30 秒轮询
   - fin_quant 包含因子挖掘和模型进化两个阶段

4. **获取因子**：`GET /api/v1/factors`

5. **获取指标**：`GET /api/v1/runs/{run_id}/metrics`
   - 同时展示因子指标和模型指标

6. **保存报告**：写入 `reports/alpha-agent/{task_id}/`
   - `01_task_summary.md` — 任务概览
   - `02_factors.md` — 因子列表
   - `03_metrics.md` — 全部指标（因子 + 模型）
   - `04_report.md` — 综合报告

---

## 流程四：任务管理

**场景**：用户想查看、取消或管理已有任务

### 查看所有任务

`GET /api/v1/tasks`

输出格式：
```
| 任务 ID | 场景 | 状态 | 进度 | 创建时间 |
|---------|------|------|------|----------|
| abc123  | fin_factor | completed | 100% | 2026-03-14 |
| def456  | fin_model  | running   | 45%  | 2026-03-14 |
```

### 查看单个任务

`GET /api/v1/tasks/{task_id}`

展示完整状态：task_id、scenario、status、progress、current_step_label、loop_index、step_index、created_at、completed_at、error。

### 取消任务

`POST /api/v1/tasks/{task_id}/cancel`

- 仅 `running` / `pending` 状态可取消
- 取消后任务进入 `cancelling` → `cancelled`

---

## 流程五：数据浏览

**场景**：用户想查看已发现的因子、历史运行和指标

### 查看因子列表

`GET /api/v1/factors`

输出格式：
```
| 因子名 | 公式 | 类型 | 运行 ID | 循环 |
|--------|------|------|---------|------|
| price_momentum_10d | (close_t - close_{t-10}) / close_{t-10} | 动量 | xxx | 0 |
```

### 查看运行历史

`GET /api/v1/runs`

输出格式：
```
| 运行 ID | 开始时间 | 循环数 | 有结果 |
|---------|----------|--------|--------|
| 2026-03-14_03-56-09 | 2026-03-14 | 1 | 是 |
```

### 查看运行指标

`GET /api/v1/runs/{run_id}/metrics`

展示关键指标 + 解读（参考 `alpha-agent/references/metrics-guide.md`）：
- IC / ICIR / Rank IC / Rank ICIR
- 年化超额收益（含/不含成本）
- 信息比率、最大回撤
- 模型损失（l2.train / l2.valid）

---

## 轮询策略

| 参数 | 值 |
|------|------|
| 轮询间隔 | 30 秒 |
| 超时限制 | 无硬性超时（fin_quant 可能运行数小时） |
| 进度报告 | 每次轮询向用户展示进度百分比 + 当前阶段 |
| 状态终态 | completed / failed / cancelled |

**进度报告模板**：
```
Alpha Agent 任务进度 [{task_id}]
━━━━━━━━━━━━━━━━━━━━━━
场景：{scenario}
进度：{progress}%  ▓▓▓▓▓▓▓░░░
阶段：{current_step_label}
循环：{loop_index}/{loop_n} | 步骤：{step_index}
耗时：{elapsed}
```

---

## 错误处理

| 情况 | 处理方式 |
|------|----------|
| API 不可达 | curl health 端点检测，报告服务不可用 |
| 认证失败（401） | 检查 Token 是否正确 |
| 任务提交失败 | 检查 scenario 是否有效（仅 fin_factor/fin_model/fin_quant） |
| 任务执行失败 | 从 task.error 提取错误信息，报告给用户 |
| 取消不存在的任务 | 提示任务 ID 无效 |
| 轮询中连接断开 | 等待 30 秒后重试，连续 3 次失败则终止 |
| 无因子产出 | loop_n 可能过小或任务中途取消，建议增加轮数重试 |

---

## 报告保存

所有报告保存到 `./reports/alpha-agent/<task_id>/`：

| 文件 | 流程一 | 流程二 | 流程三 | 说明 |
|------|:---:|:---:|:---:|------|
| `01_task_summary.md` | Y | Y | Y | 任务概览 |
| `02_factors.md` | Y | — | Y | 因子列表 |
| `02_model_metrics.md` | — | Y | — | 模型指标 |
| `03_metrics.md` | Y | Y | Y | 指标详情 + 解读 |
| `04_report.md` | Y | — | Y | 综合报告 |
| `03_report.md` | — | Y | — | 综合报告 |

---

## 注意事项

- fin_factor 通常 3-5 分钟/轮，fin_model 更长，fin_quant 最长
- loop_n 越大发现因子越多，但耗时线性增长
- 因子列表是全局累积的，包含所有历史运行产出的因子
- 运行指标仅在 has_results=true 的 run 中可用
- 任务取消是异步的，发送取消信号后需再次轮询确认状态

---

## Reference 文件索引

| 文件 | 内容 | 何时读取 |
|------|------|----------|
| `alpha-agent/references/api-details.md` | 9 个端点完整文档、Token、curl 示例 | 调用任何 API 前 |
| `alpha-agent/references/metrics-guide.md` | IC/ICIR 等指标定义与解读规则 | 展示指标时 |
