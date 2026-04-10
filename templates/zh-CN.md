# zh-CN 输出模板

## 每轮三件套

```markdown
**当前阶段：** ...
**下一步：** ...
**需要用户做什么：** ...
```

## 阶段确认块（ALIGN / PLAN / TASK）

```markdown
-------------------
📌 当前阶段：[ALIGN | PLAN | TASK]

请确认以下内容：
1. ...
2. ...

回复方式：
- 「确认」/「OK」/「继续」-> 进入下一阶段
- 直接提出修改意见 -> 当前阶段修订后再确认
-------------------
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
