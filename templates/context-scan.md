# CONTEXT_SCAN Stage Templates

固定文案单一来源：CONTEXT_SCAN 阶段输出骨架请仅使用本文件对应语言模板。

## zh-CN

```markdown
# CONTEXT_SCAN（代码仓画像 · 无 PRD / 上游依赖）

## 仓库形状
- 仓库类型：{repo_kind}（mono-repo / single / library / app / ...）
- 主语言：{primary_language}
- 技术栈：{tech_stack}
- 构建/启动入口：{build_entry}

## 顶层目录树（≤ 2 层）
```
{directory_tree}
```

## 关键模块 / 入口
| 模块ID | 名称 | 路径 | 类型（service/api/dao/...） | 备注 |
|--------|------|------|------------------------------|------|
| M-001  | {name} | {path} | {kind} | {note} |

## 接口层线索（仅列路径，不展开签名）
- {api_dir_1}
- {api_dir_2}

## 数据层线索（仅列路径，不展开表结构）
- {data_dir_1}
- {data_dir_2}

## 命名 / 术语片段
- 业务 prefix：{prefix_list}
- 领域词：{domain_terms}
- 文件命名习惯：{naming_convention}

## 文档证据
- README 摘要：{readme_summary}
- CHANGELOG 摘要：{changelog_summary}
- 接口文档目录：{api_doc_path}

## 与上游系分潜在冲突点（仅提示，留给 ALIGN 决策）
- ❓{open_point_1}
- ❓{open_point_2}

## 证据路径
- {evidence_path_1}
- {evidence_path_2}
```

## en

```markdown
# CONTEXT_SCAN (Repo Snapshot · PRD/Upstream-independent)

## Repo Shape
- Repo kind: {repo_kind} (mono-repo / single / library / app / ...)
- Primary language: {primary_language}
- Tech stack: {tech_stack}
- Build / run entry: {build_entry}

## Top-level Directory Tree (depth ≤ 2)
```
{directory_tree}
```

## Key Modules / Entry Points
| Module ID | Name | Path | Kind (service/api/dao/...) | Note |
|-----------|------|------|----------------------------|------|
| M-001     | {name} | {path} | {kind} | {note} |

## API Layer Hints (paths only, no signature)
- {api_dir_1}
- {api_dir_2}

## Data Layer Hints (paths only, no schema)
- {data_dir_1}
- {data_dir_2}

## Naming / Terms
- Business prefix: {prefix_list}
- Domain terms: {domain_terms}
- Naming convention: {naming_convention}

## Doc Evidence
- README summary: {readme_summary}
- CHANGELOG summary: {changelog_summary}
- API doc directory: {api_doc_path}

## Potential Conflicts with Upstream Analysis (hints only, defer to ALIGN)
- ❓{open_point_1}
- ❓{open_point_2}

## Evidence Paths
- {evidence_path_1}
- {evidence_path_2}
```
