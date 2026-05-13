# CONTEXT_SCAN（工程画像 · 无 PRD / 上游依赖 · 宽而浅）

本文件只定义 CONTEXT_SCAN 阶段规则；固定输出骨架请使用 `templates/context-scan.md`，通用提示块请使用 `templates/zh-CN.md` 或 `templates/en.md`。

## 目标

在尚未读取 PRD / 上游系分的前提下，对当前代码仓做一次"宽而浅"的画像，给后续 `ALIGN` 提供：

- 现有命名 / 术语 / 业务 prefix
- 现有技术栈 / 构建启动方式
- 现有模块清单与入口位置
- 接口/数据层的"线索"（不展开签名/结构）
- 文档证据（README / CHANGELOG / 接口文档目录）

帮助 `ALIGN` 阶段以"消化上游系分 + 做映射"为唯一职责，不再重复扫代码。

## 阶段规则

- 必须基于代码证据，不得凭空假设。
- 必须"宽而浅"：覆盖仓库整体形状，但**不**展开实现细节。
- **不**读 PRD、**不**读上游系分文件（`review.<lang>.md` / `system-analysis*.md` / 上游 `plan.<lang>.md` 等），保持无依赖。
- 必须读取的对象（按存在与否取舍）：
  - `README` / `README.md` / `README.<lang>.md`
  - `CHANGELOG.md`
  - 包/依赖清单：`package.json` / `pyproject.toml` / `requirements*.txt` / `Cargo.toml` / `go.mod` / `pom.xml` / `build.gradle*`
  - 顶层目录与一级子目录（深度 ≤ 2）
  - 主要入口文件（`main.*` / `cli.*` / `index.*` / `app.*` / `server.*` 等）
  - 接口/服务目录线索（`controller/` / `api/` / `routes/` / `handler/` / `rpc/` 等仅列路径）
  - 数据层线索（`migrations/` / `entity/` / `model/` / `schema/` / `dao/` 等仅列路径）
- 仓库证据必须可追溯到具体路径。
- 若 `repo_root` 与 PRD 涉及的工程不一致，必须停下来请求用户确认。

## 与 ALIGN 的边界

| 阶段 | 职责 | 不做 |
|------|------|------|
| `CONTEXT_SCAN` | 摸代码现状（无 PRD/上游依赖） | 不做需求映射、不展开接口签名/表结构 |
| `ALIGN` | 消化上游系分 + 需求-模块映射 | **不**重复扫代码，引用 `context.<lang>.md` |

冲突处理：

- 若 ALIGN 在消化上游时发现"上游系分声称的能力 vs `context.<lang>.md` 实测现状"冲突，必须显式暴露并请求用户决策（不得自行抹平）。

## 输出与落盘

1. 按当前 `lang` 使用 `templates/context-scan.md` 对应语言骨架组织输出。
2. 写入 `artifact_root/context.<lang>.md`。
3. 更新 `artifact_root/dev.state.<lang>.json`（至少包含 `artifacts.context`、`current_state`、`last_updated`）。

## 最小校验清单

- 是否给出技术栈与构建启动方式
- 是否给出顶层目录树（深度 ≤ 2）
- 是否列出关键模块 / 入口
- 是否列出接口/数据层的目录线索（仅路径）
- 是否摘录可观察的命名 / 术语
- 证据是否可追溯到路径
- 是否完成阶段确认块并得到用户确认

## 与下游衔接

CONTEXT_SCAN 确认后进入 `ALIGN`：

- `ALIGN` 的"需求 -> 模块"映射必须能落回 `context.<lang>.md` 中已盘的模块条目。
- 若映射不到，`ALIGN` 必须用 `❓` 标出"该模块尚未在 context 中观察到"，并请用户决定：补扫 context / 在 PLAN 阶段新建 / 标 out-of-scope。
