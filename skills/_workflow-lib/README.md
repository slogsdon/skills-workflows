# Workflow Skill Conventions

> Internal reference for skills under `skills/workflow-*` and `skills/publish-post`. Underscore-prefixed dir so it isn't registered as a skill.

## Anatomy of a workflow skill

A workflow skill is a standard SKILL.md whose body is a numbered orchestration of existing skills. It does not introduce new behavior — it sequences and checkpoints the skills it calls.

```yaml
---
name: workflow-<noun>
description: <Use when …, situational triggers, slash command, what it pipelines>
---

# Skill: /workflow-<noun>

<One paragraph: what this pipelines, why this exists vs running the steps manually>

## Steps

| Step name | Stage | Invokes |
|-----------|-------|---------|
| `<step-1>` | <Stage 1 name> | `<sub-skill>` |
| `<step-2>` | <Stage 2 name> | `<sub-skill>` |
| …         | …             | …             |

### Stage 1: <Stage name> — step `<step-1>` — invokes `<skill>`
- What it does
- What it produces (file path, vault note, in-conversation output)
- Checkpoint: pause for user confirmation before next step

### Stage 2: …

## Resume protocol

Default: run all steps in order, with checkpoints between each.

`--resume-from <step>`: skip every step before `<step>` and begin from `<step>`. The named step itself runs.

When `<step>` is not in the canonical list above, print:

    Invalid resume-from step: <step>
    Valid steps: <step-1>, <step-2>, …

…and stop. Do not guess, do not run a fuzzy match.

When resuming, the workflow needs the outputs that the skipped steps would have produced. List the required state per resume point in a "Required state when resuming" subsection — file paths that must exist, conversation context the user must paste, or values the user must provide. If required state is missing, prompt the user for it before running the resume step.

## Failure handling

If a stage errors, stop. Print the failed stage name + the underlying skill's error. Suggest: "Re-run this stage manually with `<skill>`, fix the issue, then resume with `--resume-from <step>`."
```

## Checkpoint format

Between stages, output exactly:

```
─── Step <step> complete ───
<one-line summary of what was produced>

Continue to <next-step>: <stage name>? (y/n):
```

If the user passes `--no-pause`, skip checkpoints entirely.

## Calling sub-skills

Use the `Skill` tool. Pass arguments via the skill's documented `args` shape — do not invent new conventions. If a sub-skill takes free-text args (most do), the workflow constructs them from prior-stage outputs.

## Step naming rules

- Lowercase, kebab-case, single concept (`plan`, `system`, `hero-shot`).
- Unique within a workflow.
- Stable — once a workflow ships, do not rename steps without bumping behavior, since users depend on the names for `--resume-from`.
- Stages and steps are 1:1. A stage gets exactly one step name.

## What workflow skills must NOT do

- Re-implement logic that lives in the underlying skill. If `design-system` already creates `tokens.css`, the workflow just calls it; it does not write tokens.css itself.
- Add side effects that aren't documented in the underlying skill's "Output" section.
- Skip sub-skill verification steps. If a stage's underlying skill has an output checklist, the workflow waits for it to be satisfied.
- Accept a numeric `--resume-from <N>`. Step names only — they survive renumbering and self-document.
