# Workflow Skill Conventions

> Internal reference for skills under `skills/workflow-*`. Underscore-prefixed dir so it isn't registered as a skill.

## Anatomy of a workflow skill

A workflow skill is a standard SKILL.md whose body is a numbered orchestration of existing skills. It does not introduce new behavior — it sequences and checkpoints the skills it calls.

```yaml
---
name: workflow-<noun>
description: <Use when …, situational triggers, slash command, what it pipelines>
---

# Skill: /workflow-<noun>

<One paragraph: what this pipelines, why this exists vs running the steps manually>

## Stages

### Stage 1: <Stage name> — invokes `<skill>`
- What it does
- What it produces (file path, vault note, in-conversation output)
- Checkpoint: pause for user confirmation before Stage 2

### Stage 2: …

## Resume / skip

To re-run from a specific stage: pass `--resume-from <N>` (e.g. `/workflow-design --resume-from 3`).
To skip a stage: pass `--skip <N>`.

## Failure handling

If a stage errors, stop. Print the failed stage + the underlying skill's error. Suggest: "Re-run this stage manually with `<skill>`, fix the issue, then resume with `--resume-from <N>`."
```

## Checkpoint format

Between stages, output exactly:

```
─── Stage <N> complete ───
<one-line summary of what was produced>

Continue to Stage <N+1>: <next stage name>? (y/n/skip):
```

If the user passes `--no-pause`, skip checkpoints entirely.

## Calling sub-skills

Use the `Skill` tool. Pass arguments via the skill's documented `args` shape — do not invent new conventions. If a sub-skill takes free-text args (most do), the workflow constructs them from prior-stage outputs.

## What workflow skills must NOT do

- Re-implement logic that lives in the underlying skill. If `design-system` already creates `tokens.css`, the workflow just calls it; it does not write tokens.css itself.
- Add side effects that aren't documented in the underlying skill's "Output" section.
- Skip sub-skill verification steps. If a stage's underlying skill has an output checklist, the workflow waits for it to be satisfied.
