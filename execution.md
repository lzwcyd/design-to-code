# EXECUTION Rules（执行阶段）

## Goal

按已确认任务顺序执行，实现代码并沉淀可恢复执行记录。

## Rules

- 执行前先重新读取 `dev-task.<lang>.md`。
- `auto: Yes` 任务直接执行并产出代码/配置/文档。
- `auto: No` 任务暂停并向用户请求输入或决策。
- 若失败可有限重试；仍失败时必须给出原因并建议回退 `PLAN` 或 `TASK`。
- 若执行中发现上游系分变更，先进入回退流程，不得“带病继续”。

## Output

- 追加写入 `dev-execution.<lang>.md`
- 更新 `dev.state.<lang>.json`
