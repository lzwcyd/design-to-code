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
