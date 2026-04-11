# English Output Templates

## Per-turn Footer

```markdown
**Current Phase:** ...
**Next Step:** ...
**What I Need From You:** ...
```

## Phase Confirmation Block (ALIGN / PLAN / TASK / EXECUTION)

```markdown
-------------------
📌 Current Stage: [ALIGN | PLAN | TASK | EXECUTION]

Please confirm:
1. ...
2. ...

Reply with:
- "Confirm" / "OK" / "Continue" -> move to next phase
- Or provide revisions -> update this phase and reconfirm

Constraint:
- No next node or next code batch is allowed before confirmation
-------------------
```

## Intermediate Artifact Gate Block

```markdown
Generated intermediate artifact: `{artifact_path}`

Key updates:
- {key_change_1}
- {key_change_2}

Open item(s):
- {open_question_or_risk}

Reply with:
- "Confirm" / "Continue" to move to next node
- "Revise: ..." to keep editing in current node and reconfirm
```

## Analysis Baseline Gate

```markdown
I loaded the following analysis artifacts as development baseline:
- {loaded_artifacts}

Identified scope:
- in-scope: {in_scope}
- out-of-scope: {out_of_scope}
- open questions: {open_questions}

Please confirm:
1. Should I proceed with this baseline?
2. Any scope to add or remove?
```

## Phase-jump Entry Validation Block

```markdown
You asked to start from `{start_phase}`.

Entry validation result:
- Missing upstream artifacts: {missing_artifacts}
- Can continue directly: {can_continue}

Choose next step:
1. Provide missing artifacts and continue
2. Fall back to `{fallback_phase}`
3. Continue with your summary as temporary baseline

Note: start phase cannot bypass unconfirmed gates.
```

## Manual Edit Detected Block

```markdown
I detected manual edits in `{artifact_path}`.

Choose how to proceed:
1. adopt: continue with your edited version
2. merge: merge it with the current draft
3. regenerate: regenerate this phase
```

## Upstream-change Rollback Prompt

```markdown
I detected an upstream analysis update: `{upstream_artifact}`.
Potentially stale downstream artifacts: {stale_artifacts}

Choose:
1. rebuild-from-align: regenerate from ALIGN
2. continue-with-current-baseline: continue as-is (with acknowledged risk)
```

## Execution Step Gate Block

```markdown
Completed execution task: `{task_id}` `{task_name}`
- Result: {success_or_fail}
- Artifacts changed: {artifact_paths}
- Notes/Risks: {notes}

Please confirm next action:
- "Continue" / "Confirm": execute next task
- "Pause": stay at current state
- "Back to TASK": return to task planning refinement
```
