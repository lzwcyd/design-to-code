# ALIGN Stage Templates

## zh-CN

```markdown
# ALIGN（系分到开发映射）

> 代码现状一律引用前置 `context.<lang>.md`，本阶段不重复扫代码。

## 输入基线
- 代码现状：{context_path}（CONTEXT_SCAN 产物）
- 上游状态：{analysis_state_path}
- 系分正文：{system_analysis_docs}
- 评审结论：{review_doc}

## 范围定义
- in-scope：{in_scope}
- out-of-scope：{out_of_scope}

## 需求追踪矩阵
| Req ID | 来源文档 | 业务规则 | 影响模块（context 模块ID） | 接口/数据变更 | 测试点 | 风险 |
|--------|----------|----------|----------------------------|---------------|--------|------|
| R-001  | {source} | {rule}   | M-xxx（{module_name}）     | {api_or_db}   | {test} | {risk} |

## 上游系分 ↔ 代码现状对照
| 对照点 | 上游声明（引用） | 代码现状（context 引用） | 结论（一致/冲突/待补） | 处理 |
|--------|------------------|---------------------------|------------------------|------|
| {topic} | {upstream_ref}   | {context_ref}             | {verdict}              | {action} |

## 风险与依赖
- {risk_or_dep_1}

## 待确认
- ❓ {open_question_1}
- ❓ {open_question_2}（无法在 context 中找到的模块映射）
```

## en

```markdown
# ALIGN (Analysis-to-Dev Mapping)

> Code reality is referenced from the preceding `context.<lang>.md`; this phase does NOT rescan code.

## Input Baseline
- Code context: {context_path} (CONTEXT_SCAN artifact)
- Upstream state: {analysis_state_path}
- System analysis docs: {system_analysis_docs}
- Review result: {review_doc}

## Scope
- in-scope: {in_scope}
- out-of-scope: {out_of_scope}

## Requirement Traceability Matrix
| Req ID | Source Doc | Business Rule | Impacted Module (context module ID) | API/Data Change | Test Point | Risk |
|--------|------------|---------------|--------------------------------------|-----------------|------------|------|
| R-001  | {source}   | {rule}        | M-xxx ({module_name})                | {api_or_db}     | {test}     | {risk} |

## Upstream Analysis ↔ Code Reality Cross-Check
| Topic | Upstream Claim (ref) | Code Reality (context ref) | Verdict (match/conflict/missing) | Action |
|-------|----------------------|----------------------------|----------------------------------|--------|
| {topic} | {upstream_ref}     | {context_ref}              | {verdict}                        | {action} |

## Risks & Dependencies
- {risk_or_dep_1}

## Open Questions
- ❓ {open_question_1}
- ❓ {open_question_2} (modules referenced by upstream but not found in context)
```
