# Analysis Ingest Rules（系分输入接入）

本文件定义 `ALIGN` 阶段如何把 `prd-to-design` 产物（上游系分）转为开发基线。

> 代码现状不在本阶段重新扫描，统一引用前置 `CONTEXT_SCAN` 阶段产出的 `context.<lang>.md`。

## Goal

在进入 PLAN/TASK/EXECUTION 前，先明确“做什么、改哪里、如何验收”。

## Required evidence

ALIGN 启动需同时满足：

1. 至少一份上游系分文件存在：
   - `review.<lang>.md`
   - 或 `system-analysis.<lang>.md`（或 `system-analysis.*.<lang>.md`）
2. 已确认的代码现状：`context.<lang>.md` 状态为 `confirmed`

任一缺失，必须向用户索要补充或回退到对应阶段，不得继续推进。

## Preferred source order

上游系分读取顺序：

1. `analysis.state.<lang>.json`
2. `review.<lang>.md`
3. `plan.<lang>.md`（上游 PLAN）
4. `impact.<lang>.md`（上游 IMPACT_SCAN 产物）
5. `spec.<lang>.md`
6. `system-analysis*.md`

代码现状来源（**不**在 ALIGN 重新扫描）：

- `context.<lang>.md`（本 skill 的 `CONTEXT_SCAN` 产物）

## ALIGN output checklist

`alignment.<lang>.md` 至少包含：

- 范围边界（in-scope / out-of-scope）
- 需求追踪矩阵（需求 -> 代码模块[引用 context.md 模块 ID] -> 数据/接口 -> 测试点）
- 上游系分 ↔ 代码现状的对照点（一致 / 冲突 / 待补）
- 风险与依赖
- 待确认项（`❓`）

补充约束：

- 写入 `alignment.<lang>.md` 后必须输出中间产物确认块。
- 用户未确认前不得进入 `PLAN`。
- 需求矩阵中的"模块"列必须可追溯到 `context.<lang>.md` 的模块 ID；映射不到的需求必须显式标 `❓` 并请用户决策（补扫 context / 在 PLAN 阶段新建 / 标 out-of-scope）。

## Conflict handling

若上游系分与 `context.<lang>.md` 中的代码现状冲突：

1. 先在 `alignment.<lang>.md` 中列冲突项和证据路径（上游引用 + context 模块 ID）。
2. 要求用户确认“以上游系分为准”还是“按代码现实回修系分”。
3. 未确认前不得进入 PLAN/EXECUTION。
4. 若需回修系分，提示用户回到 `prd-to-design` 跑回退流程，不得在本 skill 内自行修改上游产物。
