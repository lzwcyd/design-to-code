# EXECUTION Stage Templates

## zh-CN

```markdown
# EXECUTION（执行记录）

## 执行摘要
- 状态：{running_or_done}
- 已完成任务：{done_count}
- 阻塞任务：{blocked_count}

## 任务执行明细
### {task_id} {task_name}
- auto：{Yes_or_No}
- 结果：{success_or_fail}
- 产物：{artifact_paths}
- 说明：{notes}

## 错误与重试
- 错误：{error_message}
- 重试次数：{retry_count}
- 处理结论：{resolution}

## 回退建议（如需）
- 建议回退阶段：PLAN / TASK
- 原因：{rollback_reason}
- 建议修订项：{revision_items}
```

## en

```markdown
# EXECUTION (Run Log)

## Execution Summary
- Status: {running_or_done}
- Completed tasks: {done_count}
- Blocked tasks: {blocked_count}

## Task Execution Details
### {task_id} {task_name}
- auto: {Yes_or_No}
- Result: {success_or_fail}
- Artifacts: {artifact_paths}
- Notes: {notes}

## Errors and Retries
- Error: {error_message}
- Retry count: {retry_count}
- Resolution: {resolution}

## Rollback Recommendation (if needed)
- Suggested rollback phase: PLAN / TASK
- Reason: {rollback_reason}
- Suggested revisions: {revision_items}
```
