# design-to-code

基于 `prd-to-design` 产物（系分文档）推进 SDD 开发到代码的 Skill。

> 历史名：`system-analysis-to-sdd-dev`。新名 `design-to-code` 与并列两个 skill `prd-to-design`、`prd-to-code` 形成"输入→输出"对称命名。

## Typical input

- `analysis_artifact_root`: 系分产物目录（建议）
- `lang`: `zh-CN` / `en`
- `start_phase`: `ALIGN` / `PLAN` / `TASK` / `EXECUTION`
- `manual_edit_mode`: `on` / `off`

## Phase flow

`ALIGN -> PLAN -> TASK -> EXECUTION`

> 强制规则：每个阶段都要先产出中间文件并等待用户确认，未确认不得进入下一阶段。

## Main artifacts

- `alignment.<lang>.md`
- `dev-plan.<lang>.md`
- `dev-task.<lang>.md`
- `dev-execution.<lang>.md`
- `dev.state.<lang>.json`

## 默认产物目录

`artifact_root = <base_dir>/<session_id>/`，其中 `base_dir` 解析规则（避免与上游 `prd-to-design`、并列 `prd-to-code` 冲突）：

1. 显式指定 `artifact_dir`（或环境变量 `SDD_ARTIFACT_DIR`）— 最高，按用户指定路径整体替换
2. 缺省默认：`<parent_dir>/design-to-code/`
   - `parent_dir` 优先取 `analysis_artifact_root`（系分产物目录）；缺失时回退到 `repo_root` / 工作目录
   - 强制追加 `design-to-code/` skill 子目录，避免与上游系分产物或并列 skill 同名中间文件冲突

> 兼容提示：旧版默认 base 是 `<analysis_artifact_root>/dev-sdd/`，现已改为 `<analysis_artifact_root>/design-to-code/`。如需复用旧产物，可显式传 `artifact_dir=<analysis_artifact_root>/dev-sdd`。
