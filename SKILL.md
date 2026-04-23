---
name: design-to-code
description: >-
  Develops code from an EXISTING 系分文档 (produced by prd-to-design) via
  strict gated SDD flow ALIGN → PLAN → TASK → EXECUTION; bilingual (zh-CN/en),
  resumable, manual-edit takeover, upstream-analysis rollback, per-step
  EXECUTION confirmation. REQUIRES at least one upstream 系分 artifact. Use when
  the user mentions 系分文档开发 / 根据系分文档写代码 / 按系统分析写代码 /
  系统分析文档开发 / 从系分到代码 / 从系分进入 SDD / 需求已拆好开始开发 /
  按 SDD 节点逐步确认, OR references upstream analysis.state.<lang>.json /
  system-analysis.<lang>.md / review.<lang>.md, OR this skill's own
  alignment.<lang>.md / dev-plan.<lang>.md / dev-task.<lang>.md /
  dev-execution.<lang>.md / dev.state.<lang>.json / dev-sdd, OR ALIGN 阶段,
  analysis_artifact_root. Do NOT trigger when only a PRD is available without
  any 系分 artifact — defer to prd-to-design or prd-to-code. Do NOT
  trigger for design-doc / PlantUML — defer to prd-to-design.
  On ambiguity, ask A=系分 / B=直接代码 / C=已有系分写代码.
---

# System Analysis to SDD Dev（系分产物驱动开发）

**Version:** 1.1 · **Logical id:** `design_to_code`

## Input

- `analysis_artifact_root` (recommended): `prd-to-design` 产物目录（通常含 `analysis.state.<lang>.json`）
- `analysis_state_path` (optional): 显式指定分析状态文件路径
- `system_analysis_doc` (optional): 最终系分正文路径（支持单文档或多系统文档）
- `analysis_plan_doc` (optional): 上游 `plan.<lang>.md` 路径
- `analysis_review_doc` (optional): 上游 `review.<lang>.md` 路径
- `repo_root` (optional): 代码仓路径，默认当前工作目录
- `artifact_dir` (optional): 本 skill 开发产物基础目录（显式指定，优先级最高；缺省时按"系分产物目录 → 工作目录"回退，详见 Artifact directory 章节）
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

优先读取 `prd-to-design` 产物作为开发基线，读取顺序如下：

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

开发产物默认贴近上游系分产物或代码仓落盘，避免污染 skill 仓库自身。先解析 `base_dir`，再拼接会话子目录。

`base_dir` 解析优先级（高 → 低）：

1. **显式指定（最高）**
   - 输入参数 `artifact_dir`（用户在调用时显式指定）
   - 环境变量 `SDD_ARTIFACT_DIR`（全局显式配置）
   - 二者同时存在时，`artifact_dir` 优先于环境变量
2. **系分产物目录**：当 `analysis_artifact_root` 已提供（或可从 `analysis_state_path` 推导出）时，使用 `<analysis_artifact_root>/dev-sdd`
3. **工作目录**：当系分产物目录未提供、仅传了零散的 `system_analysis_doc/analysis_plan_doc/analysis_review_doc` 时，使用 `repo_root`（未提供则为当前工作目录 `cwd`）

禁止回退到 skill 自身目录（`SKILL.md` 所在目录），避免在 skill 仓库内堆积业务产物。

`session_id` 解析顺序：

1. 运行时会话唯一 ID
2. 输入参数 `session_id`
3. 若仍不可得，要求用户提供 `session_id` 后再继续（不得回退固定目录）

最终目录：

- `artifact_root = <base_dir>/<session_id>`

每轮进入 `INIT` 时必须把解析出的 `base_dir`、`base_dir_source`（来源：`artifact_dir` / `SDD_ARTIFACT_DIR` / `analysis_artifact_root` / `repo_root` / `cwd`）、`artifact_root` 显式回显给用户，便于人工核对。

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

## Auto-trigger keywords / 自动触发关键词

Cursor 通过 frontmatter 的 `description` 决定是否激活本 skill。下表是触发与排除的真源（source-of-truth），修改 description 时务必同步此处。本 skill 与上游 `prd-to-design` 和并列的 `prd-to-code` 共用同一份**互斥分发协议**，三处一致。

### 本 skill 定位

- **输入**：**已有的系分文档**（上游 `prd-to-design` 产物，至少含 `review.<lang>.md` 或 `system-analysis.<lang>.md` 之一）
- **输出**：代码 / 配置 / 脚本 / 文档改动
- **角色场景**：研发拿到正式系分后写代码 · 强节点闸门 · 严控未确认不推进

### Trigger（应触发）

| 类别 | 关键词 |
|------|--------|
| 流程 | `系分文档开发`、`使用系分文档进行开发`、`根据系分文档写代码`、`按系统分析写代码`、`系统分析文档开发`、`从系分到代码`、`从系分进入 SDD`、`需求已拆好开始开发`、`按 SDD 节点逐步确认` |
| 阶段 | `ALIGN`、`PLAN`、`TASK`、`EXECUTION`、`A B 方案`、`A/B 方案`、`auto: Yes`、`auto: No`、`节点闸门`、`逐任务确认`、`逐批确认` |
| 上游输入（来自 #1） | `analysis.state.<lang>.json`、`system-analysis.<lang>.md`、`review.<lang>.md`、`plan.<lang>.md`（上游 PLAN）、`current-state.<lang>.md`、`split.<lang>.md` |
| **本 skill 独占产物** | `alignment.<lang>.md`、`dev-plan.<lang>.md`、`dev-task.<lang>.md`、`dev-execution.<lang>.md`、`dev.state.<lang>.json`、`dev-sdd` 目录 |
| 参数 | `analysis_artifact_root`、`analysis_state_path`、`system_analysis_doc`、`analysis_plan_doc`、`analysis_review_doc`、`repo_root`、`start_phase`、`manual_edit_mode`、`session_id`、`artifact_dir` |
| 操作 | `继续历史` / `resume`、`restart`、`adopt` / `merge` / `regenerate`、`上游系分变更`、`rebuild-from-align`、`continue-with-current-baseline` |

### Do NOT trigger（应避让）

- 用户只有 PRD、**没有任何系分文档** → 让 [`prd-to-design`](../prd-to-design/SKILL.md)（先出系分文档）或 [`prd-to-code`](../prd-to-code/SKILL.md)（跳过系分直出代码）接管
- 用户要 **生成系分 / 设计文档 / PlantUML / 架构图** → 让 `prd-to-design` 接管
- 用户的"系分文档"其实只是一份 PRD（无 SPLIT/CURRENT_STATE/PLAN/REVIEW 等系分阶段产物）→ 先反问要不要走 `prd-to-design` 补齐

### Disambiguation（互斥分发协议 · 三 skill 一致）

| 用户输入特征 | 路由 |
|--------------|------|
| 已有系分文档（`analysis.state` / `system-analysis` / `review.md` / `dev-plan` / `alignment`）+ `开发` / `写代码` / `实现` / `落地` | **本 skill（#2）** |
| 仅 PRD + `系分` / `系统分析` / `PlantUML` / `方案设计` / `架构设计` / `概要设计` / `详细设计` / `接口设计` / `A B 方案` / `多系统` / `走流程` / `评审` | `prd-to-design`（#1） |
| 仅 PRD + `写代码` / `开发` / `实现` + `个人需求` / `小需求` / `小工具` / `内部工具` / `POC` / `原型` / `快速实现` / `不要系分` / `跳过系分` | `prd-to-code`（#3） |
| 仅 PRD + 无场景信号（只说"做一下"/"开始"） | **三方都反问，统一话术见下** |

### Default ask on ambiguity（仅 PRD + 无下游意图时三方一致反问）

> 这个 PRD 你想要：
> - **A. 先生成系分文档**（适合多系统、对齐、正式流程、要架构图）→ `prd-to-design`
> - **B. 直接生成代码**（适合个人需求、小工具、原型、POC、快速实现）→ `prd-to-code`
> - **C. 已有系分文档要写代码** → `design-to-code`

### Trigger examples / 触发示例（正反例）

LLM 路由判断锚点。**新增触发场景前，先在此处加一条正/反例**。

#### ✅ 应触发本 skill（#2）

1. "`@analysis.state.zh-CN.json` 系分文档定稿了，按 SDD 开始开发"
2. "`@system-analysis.lcs.zh-CN.md` 这份系分对应的代码改动开始拆任务"
3. "`@review.zh-CN.md` 评审通过，进入 ALIGN，按节点逐步确认"
4. "需求已拆好，从系分进入 SDD 开发"
5. "`@dev-task.zh-CN.md` 我手改了几个任务的 `auto` 字段，继续 EXECUTION"
6. "上游系分 `plan.zh-CN.md` 改了，下游 `dev-plan.zh-CN.md` 怎么处理"
7. "`analysis_artifact_root=./runs/abc123` 从这个目录恢复开发会话"
8. "按系统分析写代码，按 `ALIGN -> PLAN -> TASK -> EXECUTION` 强闸门推进"

#### ❌ 不应触发本 skill（应让位）

| Prompt | 该路由到 |
|--------|----------|
| "`@PRD.md` 帮我开发个小工具" | `prd-to-code`（无系分 + 个人小工具） |
| "PRD 在这里，帮我直接写代码" | `prd-to-code`（无系分 + 直接写代码） |
| "PRD 来了，先出方案设计 + PlantUML" | `prd-to-design`（要的是系分文档） |
| "`@PRD-微粒贷.md` 这是多系统改动，请按 lcs/instloancore 分别出系分" | `prd-to-design` |
| "把这份 PRD 解读一下" | 不触发（无开发意图） |
| "我有一份所谓系分文档其实就是 PRD" | 反问 / 先去 `prd-to-design` 补齐系分 |
| "`@spec.zh-CN.md` 帮我快速生成代码"（无 ALIGN/dev-* 等系分锚点） | `prd-to-code`（轻量直出代码场景） |

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
