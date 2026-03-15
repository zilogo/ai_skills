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

| 场景 | 核心能力 |
|------|----------|
| **fin_factor** | 因子挖掘与演化 — 假设→代码→回测循环，自动发现 alpha 因子 |
| **fin_model** | 模型进化 — 演化 ML 模型用于股票预测 |
| **fin_quant** | 全量化流水线 — 因子挖掘 + 模型进化端到端 |

---

## 意图识别与流程路由

从 `$ARGUMENTS`（用户自然语言）自动识别意图，路由到对应流程：

| 用户说法示例 | 流程 | 说明 |
|-------------|------|------|
| "挖因子"、"跑一轮因子"、"factor"、"alpha" | 流程一（提交） | 因子挖掘（fin_factor） |
| "训练模型"、"模型进化"、"model"、"ML" | 流程二（提交） | 模型进化（fin_model） |
| "量化研究"、"端到端"、"全流水线"、"quant" | 流程三（提交） | 端到端量化（fin_quant） |
| "查看状态"、"任务进度"、"status xxx" | 流程四 | 任务管理 |
| "取消任务 xxx"、"cancel xxx" | 流程四 | 取消任务 |
| "看结果"、"report xxx"、"生成报告" | 流程一~三（收割） | 获取结果 + 生成报告 |
| "看看因子"、"因子指标"、"历史记录"、"IC" | 流程五 | 数据浏览 |
| "健康检查"、"服务状态" | — | 健康检查 |
| 无明确意图 | 流程一（提交） | 默认因子挖掘 |

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

## 异步执行模型

流程一~三均为**异步提交**：提交任务后立即返回 task_id，不阻塞对话。用户随时可通过流程四查询进度、通过流程五查看结果。

```
提交阶段：health → POST /tasks → 返回 task_id → 结束（不等待）
查询阶段：用户随时 /alpha-agent status <task_id> 查看进度
收割阶段：任务完成后 /alpha-agent report <task_id> 生成报告
```

**用户交互流程**：
1. 用户："跑 3 轮因子" → 提交任务，返回 task_id，提示查询命令
2. 用户继续做其他事情（不阻塞）
3. 用户随时："查看状态" → 展示任务进度
4. 任务完成后："看看结果" → 获取因子 + 指标 + 生成报告

---

## 流程一：因子挖掘（fin_factor）

**场景**：用户想自动发现有效的 alpha 因子

### 提交阶段

1. **健康检查**：`GET /api/v1/health`
   - 确认服务可用，不可达则报告并终止

2. **提交任务**：`POST /api/v1/tasks`
   ```json
   {
     "scenario": "fin_factor",
     "loop_n": 3
   }
   ```

3. **立即返回**，向用户报告：
   ```
   任务已提交
   ━━━━━━━━━━━━━━━━━━━━
   任务 ID：{task_id}
   场景：fin_factor（因子挖掘）
   循环轮数：{loop_n}
   预估耗时：{loop_n × 3~5} 分钟

   查看进度：/alpha-agent status {task_id}
   查看结果：/alpha-agent report {task_id}
   取消任务：/alpha-agent cancel {task_id}
   ```

### 收割阶段（用户请求查看结果时）

当用户通过 `/alpha-agent report <task_id>` 或 "看看结果" 触发：

1. **检查状态**：`GET /api/v1/tasks/{task_id}`
   - 未完成 → 报告当前进度，提示稍后再查
   - 已完成 → 继续

2. **获取因子**：`GET /api/v1/factors`
   - 展示新发现的因子列表（名称、公式、描述）

3. **获取指标**：从任务结果中提取 `run_id`，调用 `GET /api/v1/runs/{run_id}/metrics`
   - 展示 IC、ICIR、Rank IC、Rank ICIR 等核心指标
   - 指标解读参考 `alpha-agent/references/metrics-guide.md`

4. **保存报告**：写入 `reports/alpha-agent/{task_id}/`
   - `01_task_summary.md` — 任务概览（场景、参数、耗时）
   - `02_factors.md` — 因子列表（名称、公式、描述、实现状态）
   - `03_metrics.md` — 指标详情 + 解读
   - `04_report.md` — 综合报告

---

## 流程二：模型进化（fin_model）

**场景**：用户想演化 ML 模型用于股票预测

### 提交阶段

同流程一，scenario 改为 `fin_model`。提示预估耗时更长。

### 收割阶段

1. 检查状态 → 2. 获取指标（`GET /api/v1/runs/{run_id}/metrics`，重点 l2.train / l2.valid） → 3. 保存报告

报告文件：
- `01_task_summary.md` — 任务概览
- `02_model_metrics.md` — 模型指标 + 解读
- `03_report.md` — 综合报告

---

## 流程三：端到端量化（fin_quant）

**场景**：用户想执行完整的量化研究流水线（因子挖掘 + 模型进化）

### 提交阶段

同流程一，scenario 改为 `fin_quant`。提示这是最全面也最耗时的场景。

### 收割阶段

1. 检查状态 → 2. 获取因子 → 3. 获取指标（因子 + 模型） → 4. 保存报告

报告文件：
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

## 状态查询格式

当用户查询任务状态时，按以下格式展示：

```
Alpha Agent 任务状态 [{task_id}]
━━━━━━━━━━━━━━━━━━━━━━
场景：{scenario}
状态：{status}
进度：{progress}%  ▓▓▓▓▓▓▓░░░
阶段：{current_step_label}
循环：{loop_index}/{loop_n} | 步骤：{step_index}
已运行：{elapsed}（创建于 {created_at}）
```

**状态终态**：`completed` / `failed` / `cancelled`

任务完成时追加提示：
```
任务已完成！查看结果：/alpha-agent report {task_id}
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
