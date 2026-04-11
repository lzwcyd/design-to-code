# TASK Rules（任务拆解阶段）

## Goal

把方案拆成可排程、可执行、可追踪的任务清单。

## Rules

- 任务按优先级分层（P0/P1/P2 或等价分层）。
- 每条任务必须包含：输入、输出、依赖、`auto: Yes/No`、验收标准。
- 任务顺序必须可直接用于 EXECUTION。
- 若用户已手改 `dev-task.<lang>.md`，先走 `adopt/merge/regenerate`。
- 写入 `dev-task.<lang>.md` 后必须等待用户确认，未确认不得进入 `EXECUTION`。

## Output

- 写入 `dev-task.<lang>.md`
- 更新 `dev.state.<lang>.json`
