---
name: system-analysis-to-sdd-dev
description: Develops software with SDD using outputs from prd-to-system-analysis-skill. Ingests analysis artifacts (analysis.state, system-analysis, review, plan), then drives ALIGN -> PLAN -> TASK -> EXECUTION with resume, manual-edit sync, entry validation, and rollback on upstream analysis changes. Use when the user mentions 系分文档开发, 使用系分文档进行开发, 根据系分文档写代码, 按系统分析写代码, 系统分析文档开发, analysis.state, system-analysis, review.md, or asks to start development from analysis artifacts.
---

# System Analysis to SDD Dev（系分产物驱动开发）

**Version:** 1.1 · **Logical id:** `system_analysis_to_sdd_dev`

## Input

- `analysis_artifact_root` (recommended): `prd-to-system-analysis-skill` 产物目录（通常含 `analysis.state.<lang>.json`）
- `analysis_state_path` (optional): 显式指定分析状态文件路径
- `system_analysis_doc` (optional): 最终系分正文路径（支持单文档或多系统文档）
- `analysis_plan_doc` (optional): 上游 `plan.<lang>.md` 路径
- `analysis_review_doc` (optional): 上游 `review.<lang>.md` 路径
- `repo_root` (optional): 代码仓路径，默认当前工作目录
- `artifact_dir` (optional): 本 skill 开发产物基础目录（优先级最高）
- `session_id` (optional): 会话唯一 ID；未指定时优先使用运行时会话 ID
- `lang` (optional): `zh-CN` | `en`（默认按用户本轮语言）
- `start_phase` (optional): `AUTO` | `ALIGN` | `PLAN` | `TASK` | `EXECUTION`（默认 `AUTO`，仅用于恢复入口提示）
- `manual_edit_mode` (optional): `on` | `off`（默认 `on`）

## Strict SDD gate mode（强制节点闸门）

本 skill 默认并强制启用“节点闸门”：

1. 必须按 `ALIGN -> PLAN -> TASK -> EXECUTION` 顺序推进。
2. 每个阶段必须先产出并落盘中间文件，再请求用户确认。
3. 未收到确认前，不得进入下一阶段，不得执行下一批代码改动。
4. `start_phase` 只能用于恢复入口定位，不允许绕过未确认阶段。
5. `EXECUTION` 阶段按任务（或批次）确认后继续。

## Output

- 分阶段开发文档：`alignment/plan/task/execution`
- `auto: Yes` 任务对应的代码、配置、脚本与文档改动
- 可恢复状态：`dev.state.<lang>.json`

## Language mode

1. `lang=zh-CN`：全量中文输出。
2. `lang=en`：全量英文输出。
3. 未指定 `lang`：按用户本轮语言自动识别。
4. 每轮输出前读取：
   - `zh-CN` -> [templates/zh-CN.md](templates/zh-CN.md)
   - `en` -> [templates/en.md](templates/en.md)

## Upstream analysis ingestion（读取系分产物）

优先读取 `prd-to-system-analysis-skill` 产物作为开发基线，读取顺序如下：

1. `analysis_state_path`（若显式提供）
2. `<analysis_artifact_root>/analysis.state.<lang>.json`
3. `<analysis_artifact_root>/review.<lang>.md`
4. `<analysis_artifact_root>/plan.<lang>.md`
5. `<analysis_artifact_root>/current-state.<lang>.md`
6. `<analysis_artifact_root>/spec.<lang>.md`
7. `<analysis_artifact_root>/system-analysis.<lang>.md`
8. `<analysis_artifact_root>/system-analysis.*.<lang>.md`
9. 输入参数显式提供的 `system_analysis_doc` / `analysis_plan_doc` / `analysis_review_doc`

约束：

- 若关键输入缺失（至少应有 `review` 或 `system-analysis` 之一），不得臆造上下文，必须先提问补齐。
- 若存在多份 `system-analysis.*.<lang>.md`，先生成系统-模块映射并请求用户确认作用域。
- 若上游语言与当前 `lang` 不一致，先向用户确认是否跨语言继续。

详情见 [analysis-ingest.md](analysis-ingest.md)。

## Artifact directory（开发产物落盘）

先解析 `base_dir`，再拼接会话子目录：

1. `artifact_dir`（用户显式指定）
2. `<analysis_artifact_root>/dev-sdd`（推荐默认）
3. 环境变量 `SDD_ARTIFACT_DIR`
4. skill 目录（`SKILL.md` 所在目录）

`session_id` 解析顺序：

1. 运行时会话唯一 ID
2. 输入参数 `session_id`
3. 若仍不可得，要求用户提供 `session_id` 后再继续（不得回退固定目录）

最终目录：

- `artifact_root = <base_dir>/<session_id>`

### Mandatory artifacts

在 `artifact_root` 下维护（按语言区分）：

- `alignment.<lang>.md`
- `dev-plan.<lang>.md`
- `dev-task.<lang>.md`
- `dev-execution.<lang>.md`
- `dev.state.<lang>.json`

## Resume behavior

每次进入 `INIT` 时先执行恢复检查：

1. 定位 `artifact_root` 与 `lang`。
2. 若存在 `dev.state.<lang>.json` 或阶段文档，汇总最近进度。
3. 询问用户选择：
   - `resume`（继续历史）
   - `restart`（从头开始）
4. `resume`：从最近未完成阶段继续，优先复用已落盘文档。
5. `restart`：保留旧文档（可时间戳备份）并开启新流程。

## Start-phase compatibility（起始阶段兼容）

起始阶段解析优先级：

1. `start_phase`（用户显式指定）
2. `dev.state.<lang>.json` 历史状态（resume）
3. 默认：`ALIGN`

强制约束：

- `AUTO` 必须解析为“最早未确认阶段”。
- 指定 `PLAN/TASK/EXECUTION` 时，必须校验前置阶段状态为 `confirmed`。
- 任一前置阶段未确认时，自动回退到最近未确认阶段并提示用户确认。
- 不允许通过 `start_phase` 绕过 `ALIGN_CONFIRMING/PLAN_CONFIRMING/TASK_CONFIRMING/EXECUTION_CONFIRMING`。

入口校验规则：

- `ALIGN`：需能读取至少一个有效系分输入。
- `PLAN`：必须已有 `alignment.<lang>.md` 且状态为 `confirmed`。
- `TASK`：必须已有 `dev-plan.<lang>.md` 且状态为 `confirmed`。
- `EXECUTION`：必须已有 `dev-task.<lang>.md` 且状态为 `confirmed`。

若上游产物缺失，不得假装已完成。必须让用户选择：

1. 补齐缺失产物后继续
2. 回退到上一阶段生成
3. 使用用户提供摘要作为临时基线继续（需显式确认风险）

## Manual edit synchronization

当 `manual_edit_mode=on`：

1. 每阶段开始前重新读取 `alignment/dev-plan/dev-task/dev-execution`。
2. 若校验和或时间戳变化，视为用户手改。
3. 提示用户选择：`adopt` | `merge` | `regenerate`。
4. 未确认前不覆盖用户手改内容。
5. 若用户只回复“ok/继续/确认”，默认按 `adopt` 并记录到状态文件。

## Upstream-change rollback（上游系分变更回退）

若检测到上游文件（如 `review`、`plan`、`system-analysis`）发生变化：

1. 进入 `ROLLBACK_SYNCING`。
2. 将下游产物标记为 `stale_due_to_analysis_change`。
3. 默认回退到 `ALIGN_CONFIRMING`。
4. 让用户选择：`rebuild-from-align` 或 `continue-with-current-baseline`。
5. 记录 `rollback.from_phase/trigger/reason/needs_regeneration`。

## Intent normalization（简短确认语义归一）

当用户仅回复 `ok/继续/确认`：

1. 若当前等待阶段确认 -> 视为本阶段确认通过。
2. 若当前等待手改策略 -> 默认 `adopt`。
3. 若当前等待方案选择（A/B）-> 不自动推断，必须二次确认。

## End-to-end flow

```text
[START: AUTO|ALIGN|PLAN|TASK|EXECUTION]
-> 读取上游系分产物
-> 入口校验
-> 手改同步/上游变更检测
-> ALIGN_DRAFT -> ALIGN_CONFIRMING
-> PLAN_DRAFT -> PLAN_CONFIRMING
-> TASK_DRAFT -> TASK_CONFIRMING
-> EXECUTION_DRAFT -> EXECUTION_CONFIRMING
-> EXECUTION_RUNNING（逐任务/逐批确认）
-> EXECUTION_DONE_CONFIRMING
-> DONE
```

## Non-negotiable rules

1. 强制按 `ALIGN -> PLAN -> TASK -> EXECUTION` 顺序执行。
2. 每阶段必须先写入中间文件，再请求确认。
3. 未确认前不得进入下一阶段，不得继续下一批代码改动。
4. Unknowns 必须标记 `❓`，禁止静默假设。
5. `ALIGN` 必须产出需求-模块-测试可追踪矩阵并等待确认。
6. `PLAN` 必须输出至少 A/B 两案，并等待用户选择与确认。
7. `TASK` 每项必须含输入、输出、依赖、`auto`、验收标准，并等待确认。
8. `EXECUTION` 前必须再次同步最新 `dev-task.<lang>.md`，并先完成开工确认。
9. `EXECUTION` 过程中默认逐任务（或逐批）确认后再继续。
10. 每阶段完成必须落盘并更新 `dev.state.<lang>.json`。
11. 检测到用户手改或上游分析变更时，不得直接覆盖或忽略。
12. 不得编造现状能力、接口、表结构；不确定项必须回到用户确认。
13. 若上游系分与代码现实冲突，优先显式暴露冲突并请求决策。

## State machine

| State | Meaning |
|------|---------|
| `INIT` | 读取输入并检查可恢复状态 |
| `ANALYSIS_INGESTING` | 读取并解析上游系分产物 |
| `ENTRY_VALIDATING` | 校验阶段跳转前置条件 |
| `ARTIFACT_SYNCING` | 同步用户手改开发文档 |
| `ROLLBACK_SYNCING` | 上游系分变更后的回退处理 |
| `ALIGN_DRAFT` / `ALIGN_CONFIRMING` | 需求到实现映射与确认 |
| `PLAN_DRAFT` / `PLAN_CONFIRMING` | 技术方案 A/B 与确认 |
| `TASK_DRAFT` / `TASK_CONFIRMING` | 任务拆解与确认 |
| `EXECUTION_DRAFT` / `EXECUTION_CONFIRMING` | 执行前准备与开工确认 |
| `EXECUTION_RUNNING` | 执行任务并实现代码（逐任务/逐批确认） |
| `EXECUTION_STEP_CONFIRMING` | 单任务或批次执行后的继续确认 |
| `EXECUTION_DONE_CONFIRMING` | 执行完成后的收口确认 |
| `DONE` | 结束 |

Forbidden：未通过入口校验直接跳转；未确认直接进入下一阶段；检测到手改/上游变更后直接覆盖写盘。

## Mandatory output blocks（每轮必带）

按 `lang` 读取模板并使用：

- Per-turn Footer / 每轮三件套
- Phase Confirmation Block / 阶段确认块
- Intermediate Artifact Gate Block / 中间产物确认块
- Analysis Baseline Gate / 系分基线确认块
- Phase-jump Entry Validation Block / 阶段跳转入口校验提示块
- Manual Edit Detected Block / 检测到人工修改提示块
- Rollback Prompt Block / 上游变更回退提示块
- Execution Step Gate Block / 执行步确认块

## Phase routing

### Phase: ALIGN

1. 读取 [analysis-ingest.md](analysis-ingest.md)。
2. 读取 [templates/alignment.md](templates/alignment.md) 对应语言骨架。
3. 输出“需求 -> 模块 -> 数据/接口 -> 测试 -> 风险”映射矩阵并写入 `alignment.<lang>.md`。
4. 输出中间产物确认块，展示关键变化与 `❓` 缺口，等待用户确认后进入 `PLAN`。

### Phase: PLAN

1. 读取 [plan.md](plan.md)。
2. 读取 [templates/plan.md](templates/plan.md) 对应语言骨架。
3. 强制输出 A/B 方案并要求用户选择。
4. 写入 `dev-plan.<lang>.md`，输出中间产物确认块，等待用户确认后进入 `TASK`。

### Phase: TASK

1. 读取 [task.md](task.md)。
2. 读取 [templates/task.md](templates/task.md) 对应语言骨架。
3. 产出可执行任务（含输入/输出/依赖/`auto`）并写入 `dev-task.<lang>.md`。
4. 输出中间产物确认块，等待用户确认后进入 `EXECUTION`。

### Phase: EXECUTION

1. 读取 [execution.md](execution.md)。
2. 读取 [templates/execution.md](templates/execution.md) 对应语言骨架。
3. 先生成执行预备稿（任务顺序、预计改动、风险、回滚点）并写入 `dev-execution.<lang>.md`。
4. 输出阶段确认块，未确认不得开始执行。
5. 按任务顺序执行：`auto: Yes` 执行当前任务后写盘并等待“继续”；`auto: No` 暂停等待用户输入。
6. 每次任务完成后必须输出执行步确认块，确认后才继续下一任务。
7. 收口后进入 `EXECUTION_DONE_CONFIRMING`，确认后写入 `DONE`。

## State file contract（minimum）

`dev.state.<lang>.json` 最小字段建议：

```json
{
  "lang": "zh-CN",
  "analysis_artifact_root": "/path/to/analysis/artifacts",
  "artifact_root": "/path/to/dev/artifacts/session-id",
  "current_state": "TASK_CONFIRMING",
  "last_updated": "2026-04-10T00:00:00Z",
  "manual_edit_mode": "on",
  "start_phase": "AUTO",
  "effective_start_phase": "ALIGN",
  "confirmation": {
    "required": true,
    "last_confirmed_state": "PLAN_CONFIRMING",
    "pending_gate": "TASK_CONFIRMING",
    "last_user_reply": "确认"
  },
  "entry_validation": {
    "status": "passed",
    "missing_upstream": []
  },
  "rollback": {
    "from_phase": "PLAN_CONFIRMING",
    "trigger": "analysis_plan_changed",
    "reason": "upstream analysis updated",
    "needs_regeneration": ["dev-task.zh-CN.md"]
  },
  "artifacts": {
    "alignment": { "path": "alignment.zh-CN.md", "status": "confirmed" },
    "plan": { "path": "dev-plan.zh-CN.md", "status": "confirmed" },
    "task": { "path": "dev-task.zh-CN.md", "status": "draft" },
    "execution": { "path": "dev-execution.zh-CN.md", "status": "pending" }
  },
  "execution_control": {
    "mode": "step-gated",
    "next_task_id": "P0-1",
    "last_confirmed_task_id": "P0-0"
  }
}
```

## Trigger keywords

- `根据系分文档开发`
- `按系统分析写代码`
- `analysis.state`
- `system-analysis`
- `review.md`
- `从系分进入 SDD`
- `需求已拆好，开始开发`
- `按SDD节点逐步确认`

## Additional resources

- [analysis-ingest.md](analysis-ingest.md) — 上游系分产物读取与基线确认规则
- [plan.md](plan.md) — PLAN 阶段规则
- [task.md](task.md) — TASK 阶段规则
- [execution.md](execution.md) — EXECUTION 阶段规则
- [templates/zh-CN.md](templates/zh-CN.md) — 中文模板块
- [templates/en.md](templates/en.md) — English template blocks
- [templates/alignment.md](templates/alignment.md) — ALIGN 固定骨架
- [templates/plan.md](templates/plan.md) — PLAN 固定骨架
- [templates/task.md](templates/task.md) — TASK 固定骨架
- [templates/execution.md](templates/execution.md) — EXECUTION 固定骨架
