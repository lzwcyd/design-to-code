# zh-CN 输出模板

## 每轮三件套

```markdown
**当前阶段：** ...
**下一步：** ...
**需要用户做什么：** ...
```

## 阶段确认块（ALIGN / PLAN / TASK / EXECUTION）

```markdown
-------------------
📌 当前阶段：[ALIGN | PLAN | TASK | EXECUTION]

请确认以下内容：
1. ...
2. ...

回复方式：
- 「确认」/「OK」/「继续」-> 进入下一阶段
- 直接提出修改意见 -> 当前阶段修订后再确认

约束：
- 未确认前不会进入下一节点，也不会执行下一批代码改动
-------------------
```

## 中间产物确认块（Intermediate Artifact Gate）

```markdown
已生成中间产物：`{artifact_path}`

关键内容摘要：
- {key_change_1}
- {key_change_2}

待确认项：
- {open_question_or_risk}

请回复：
- 「确认」/「继续」：进入下一节点
- 「修改：...」：在当前节点修订后再次提交确认
```

## 系分基线确认块（Analysis Baseline Gate）

```markdown
已读取以下系分产物作为开发基线：
- {loaded_artifacts}

识别到的作用域：
- in-scope: {in_scope}
- out-of-scope: {out_of_scope}
- 待确认: {open_questions}

请确认：
1. 是否按该基线推进？
2. 是否有需要排除/新增的范围？
```

## 阶段跳转入口校验提示块

```markdown
你指定从 `{start_phase}` 开始。

入口校验结果：
- 缺失上游产物：{missing_artifacts}
- 可直接继续：{can_continue}

请选择下一步：
1. 补齐缺失产物后继续
2. 回退到 `{fallback_phase}`
3. 使用你的摘要作为临时基线继续

说明：即使指定了起始阶段，也不能绕过未确认节点。
```

## 检测到人工修改提示块

```markdown
检测到你手工修改了 `{artifact_path}`。

请选择处理策略：
1. adopt：采用手改版本继续
2. merge：与当前草稿合并
3. regenerate：重生成本阶段
```

## 上游变更回退提示块

```markdown
检测到上游系分产物变更：`{upstream_artifact}`。
下游文档可能失效：{stale_artifacts}

请选择：
1. rebuild-from-align：回到 ALIGN 重新生成
2. continue-with-current-baseline：维持当前基线继续（需你确认风险）
```

## 执行步确认块（Execution Step Gate）

```markdown
已完成执行任务：`{task_id}` `{task_name}`
- 执行结果：{success_or_fail}
- 产物变更：{artifact_paths}
- 风险/备注：{notes}

请确认是否继续下一任务：
- 「继续」/「确认」：执行下一任务
- 「暂停」：停在当前状态，等待你进一步指令
- 「回退到 TASK」：回到任务拆解修订
```
