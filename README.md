# design-to-code

基于 `prd-to-design` 产物（系分文档）推进 SDD 开发到代码的 Skill。

> 历史名：`system-analysis-to-sdd-dev`。新名 `design-to-code` 与并列两个 skill `prd-to-design`、`prd-to-code` 形成"输入→输出"对称命名。

## Typical input

- `analysis_artifact_root`: 系分产物目录（建议）
- `lang`: `zh-CN` / `en`
- `start_phase`: `CONTEXT_SCAN` / `ALIGN` / `PLAN` / `TASK` / `EXECUTION`
- `manual_edit_mode`: `on` / `off`

## Main artifacts

- `context.<lang>.md`（**新增**：CONTEXT_SCAN 工程画像，无 PRD/上游依赖）
- `alignment.<lang>.md`
- `dev-plan.<lang>.md`
- `dev-task.<lang>.md`
- `dev-execution.<lang>.md`
- `dev.state.<lang>.json`

## 状态机（State Machine）

### 阶段流程图

```mermaid
flowchart LR
    contextScan[CONTEXT_SCAN] --> align[ALIGN]
    align --> plan[PLAN]
    plan --> task[TASK]
    task --> execution[EXECUTION]
    execution --> doneNode[DONE]
```

### 阶段说明

| 阶段 | 输入 | 产物 | 是否需用户确认 | 入口前置 |
|------|------|------|----------------|----------|
| `CONTEXT_SCAN` | 代码仓（**不**读 PRD/上游系分） | `context.<lang>.md` | 是 | 无 |
| `ALIGN` | `context.<lang>.md` + 上游系分（`review` / `system-analysis` / 上游 `plan` 等） | `alignment.<lang>.md`（需求→模块映射 + 上游 vs 现状对照） | 是 | CONTEXT_SCAN 已确认；存在至少一份上游系分 |
| `PLAN` | `alignment.<lang>.md` | `dev-plan.<lang>.md`（含 A/B 方案） | 是 | ALIGN 已确认 |
| `TASK` | `dev-plan.<lang>.md` | `dev-task.<lang>.md`（含 `auto` 字段） | 是 | PLAN 已确认 |
| `EXECUTION` | `dev-task.<lang>.md` | `dev-execution.<lang>.md` + 代码改动 | 开工 + 逐任务/逐批确认 | TASK 已确认 |

> **强制规则**：每个阶段都要先产出中间文件并等待用户确认，未确认不得进入下一阶段；EXECUTION 内部默认逐任务/逐批确认。
>
> **ALIGN 减负**：ALIGN 只负责消化上游系分 + 做需求-代码模块映射；代码现状一律引用 `context.<lang>.md`，**不**在 ALIGN 内重新扫代码。

### 状态枚举

| State | 含义 |
|-------|------|
| `INIT` | 读取输入并检查可恢复 |
| `ANALYSIS_INGESTING` | 读取并解析上游系分产物（供 ALIGN 使用） |
| `ENTRY_VALIDATING` | 校验阶段跳转前置 |
| `ARTIFACT_SYNCING` | 同步用户手改开发文档 |
| `ROLLBACK_SYNCING` | 上游系分变更后的回退处理 |
| `CONTEXT_SCAN_DRAFT` / `CONTEXT_SCAN_CONFIRMING` | 代码现状画像与确认 |
| `ALIGN_DRAFT` / `ALIGN_CONFIRMING` | 需求-模块映射与确认（基于 context + 上游） |
| `PLAN_DRAFT` / `PLAN_CONFIRMING` | A/B 方案与确认 |
| `TASK_DRAFT` / `TASK_CONFIRMING` | 任务拆解与确认 |
| `EXECUTION_DRAFT` / `EXECUTION_CONFIRMING` | 执行预备稿与开工确认 |
| `EXECUTION_RUNNING` | 执行任务（逐任务/逐批） |
| `EXECUTION_STEP_CONFIRMING` | 单任务/批次执行后的继续确认 |
| `EXECUTION_DONE_CONFIRMING` | 执行完成收口确认 |
| `DONE` | 终态 |

### 特殊行为

- **手改同步**：`manual_edit_mode=on` 时每阶段前重读 `context/alignment/dev-plan/dev-task/dev-execution`，检测到变化提示 `adopt` / `merge` / `regenerate`。
- **上游变更回退**：若上游系分（`review` / `plan` / `system-analysis`）发生变化，进入 `ROLLBACK_SYNCING`，下游产物标记 `stale_due_to_analysis_change`，默认回退到 `ALIGN_CONFIRMING`。
- **阶段跳转校验**：`start_phase` 仅用于恢复入口定位，不允许绕过任一 `*_CONFIRMING` 闸门。
- **上游 ↔ 代码现状冲突**：ALIGN 必须显式列冲突项并请求用户决策（以上游为准 / 回修系分），未决策前不得进入 PLAN/EXECUTION。
- **首次会话默认**：`start_phase=AUTO` 解析为 `CONTEXT_SCAN`；存在已确认 `context.<lang>.md` 时回落到 `ALIGN`。

## 默认产物目录

`artifact_root = <base_dir>/<session_id>/`，其中 `base_dir` 解析规则（避免与上游 `prd-to-design`、并列 `prd-to-code` 冲突）：

1. 显式指定 `artifact_dir`（或环境变量 `SDD_ARTIFACT_DIR`）— 最高，按用户指定路径整体替换
2. 缺省默认：`<parent_dir>/design-to-code/`
   - `parent_dir` 优先取 `analysis_artifact_root`（系分产物目录）；缺失时回退到 `repo_root` / 工作目录
   - 强制追加 `design-to-code/` skill 子目录，避免与上游系分产物或并列 skill 同名中间文件冲突

> 兼容提示：旧版默认 base 是 `<analysis_artifact_root>/dev-sdd/`，现已改为 `<analysis_artifact_root>/design-to-code/`。如需复用旧产物，可显式传 `artifact_dir=<analysis_artifact_root>/dev-sdd`。
