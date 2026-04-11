# PLAN Rules（技术方案阶段）

## Goal

把 `ALIGN` 的需求映射转为可落地技术方案，并驱动用户完成方案决策。

## Rules

- 必须给出至少 A/B 两个方案并明确取舍。
- 每个方案都要覆盖：改动模块、数据/接口影响、风险、回滚策略。
- 需要明确与上游系分的一致性：哪些点“直接继承”，哪些点“实现层补充”。
- 若用户已手改 `dev-plan.<lang>.md`，先走 `adopt/merge/regenerate`，再继续。
- 写入 `dev-plan.<lang>.md` 后必须展示中间产物确认块，未确认不得进入 `TASK`。

## Output

- 写入 `dev-plan.<lang>.md`
- 更新 `dev.state.<lang>.json`
