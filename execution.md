# EXECUTION Rules（执行阶段）

## Goal

按已确认任务顺序执行，实现代码并沉淀可恢复执行记录。

## Rules

- 执行前先重新读取 `dev-task.<lang>.md`。
- 执行前先生成“执行预备稿”（任务顺序、预计改动、风险、回滚点）写入 `dev-execution.<lang>.md`，并等待用户确认开工。
- `auto: Yes` 任务执行后必须先写入执行记录并请求用户确认，再继续下一任务。
- `auto: No` 任务暂停并向用户请求输入或决策。
- 用户回复“暂停/stop/pause”时，必须停在当前任务并保持可恢复状态，不得自动推进。
- 若失败可有限重试；仍失败时必须给出原因并建议回退 `PLAN` 或 `TASK`。
- 若执行中发现上游系分变更，先进入回退流程，不得“带病继续”。

## Output

- 追加写入 `dev-execution.<lang>.md`
- 更新 `dev.state.<lang>.json`
