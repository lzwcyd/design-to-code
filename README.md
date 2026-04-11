# system-analysis-to-sdd-dev

基于 `prd-to-system-analysis-skill` 产物推进 SDD 开发的 Skill。

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
