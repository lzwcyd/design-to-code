# Analysis Ingest Rules（系分输入接入）

本文件定义如何把 `prd-to-system-analysis-skill` 产物转为开发基线。

## Goal

在进入 PLAN/TASK/EXECUTION 前，先明确“做什么、改哪里、如何验收”。

## Required evidence

至少满足其一：

1. `review.<lang>.md`
2. `system-analysis.<lang>.md`（或 `system-analysis.*.<lang>.md`）

若两者都缺失，必须向用户索要补充，不得继续推进。

## Preferred source order

1. `analysis.state.<lang>.json`
2. `review.<lang>.md`
3. `plan.<lang>.md`
4. `current-state.<lang>.md`
5. `spec.<lang>.md`
6. `system-analysis*.md`

## ALIGN output checklist

`alignment.<lang>.md` 至少包含：

- 范围边界（in-scope / out-of-scope）
- 需求追踪矩阵（需求 -> 代码模块 -> 数据/接口 -> 测试点）
- 风险与依赖
- 待确认项（`❓`）

补充约束：

- 写入 `alignment.<lang>.md` 后必须输出中间产物确认块。
- 用户未确认前不得进入 `PLAN`。

## Conflict handling

若上游文档与代码现状冲突：

1. 先列冲突项和证据路径。
2. 要求用户确认“以上游系分为准”还是“按代码现实回修系分”。
3. 未确认前不得进入 EXECUTION。
