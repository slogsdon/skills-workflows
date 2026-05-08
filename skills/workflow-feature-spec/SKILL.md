---
name: workflow-feature-spec
description: Run the full feature-spec pipeline for a new product / feature with commercial stakes — biz-brief (business context) → superpowers:brainstorming (technical spec) → superpowers:writing-plans (implementation plan). Use when the user says "let's build X", "I want to ship Y", "spec out this feature", "go from idea to plan for Z", or when starting any new feature with a user / market / business model. Do NOT use for purely internal tooling (skip biz-brief; go straight to brainstorming) or for bug fixes (use debugging-and-error-recovery instead).
---

# Skill: /workflow-feature-spec

Pipeline that takes a fuzzy feature idea and ends with an executable implementation plan grounded in business intent + technical spec.

## When to use

- New feature / product / initiative with commercial stakes.
- The user is at the "let's build X" stage; no spec yet.

## When NOT to use

- Internal tooling with no user/market layer → call `superpowers:brainstorming` directly.
- Bug fix → use the debug-and-fix flow instead.
- Already have a spec → call `superpowers:writing-plans` directly.

## Steps

| Step name | Stage              | Invokes                       |
|-----------|--------------------|-------------------------------|
| `brief`   | Business context   | `biz-brief`                   |
| `spec`    | Technical spec     | `superpowers:brainstorming`   |
| `plan`    | Implementation plan| `superpowers:writing-plans`   |

### Stage 1: Business context — step `brief` — invokes `biz-brief`

Generates a 1-page Business Context Brief and saves it to `docs/superpowers/briefs/YYYY-MM-DD-<feature>.md`.

Output: brief file exists.
Checkpoint: confirm brief is accurate before brainstorming.

### Stage 2: Technical spec — step `spec` — invokes `superpowers:brainstorming`

Brainstorms the technical spec with the brief loaded as context. Saves spec to `docs/superpowers/specs/YYYY-MM-DD-<feature>.md`.

Output: spec file exists.
Checkpoint: confirm spec coverage before writing the plan.

### Stage 3: Implementation plan — step `plan` — invokes `superpowers:writing-plans`

Reads the spec and produces a task-by-task plan at `docs/superpowers/plans/YYYY-MM-DD-<feature>.md`.

Output: plan file exists, ready to hand off to `subagent-driven-development` or `executing-plans`.

## Resume protocol

Default: run `brief` → `spec` → `plan` in order, with checkpoints between each.

`--resume-from <step>`: skip every step before `<step>` and begin from `<step>`. The named step itself runs.

If `<step>` is not in `{brief, spec, plan}`, print:

    Invalid resume-from step: <step>
    Valid steps: brief, spec, plan

…and stop.

### Required state when resuming

Before running the resume step, confirm the listed inputs exist. If anything is missing, prompt the user for it.

- `--resume-from brief` — same as a fresh run; needs only the feature name.
- `--resume-from spec` — needs the brief at `docs/superpowers/briefs/YYYY-MM-DD-<feature>.md`. If absent, ask the user to paste it or run from `brief`. Brainstorming loads the brief as context.
- `--resume-from plan` — needs the spec at `docs/superpowers/specs/YYYY-MM-DD-<feature>.md`. If absent, ask the user to paste it or run from `spec`. Writing-plans operates on the spec.

Examples:

    /workflow-feature-spec --resume-from spec   # brief already exists, brainstorm from it
    /workflow-feature-spec --resume-from plan   # spec is approved, just write the plan

## Other flags

- `--no-pause` to run end-to-end (rare — usually you want the checkpoints here).

## Failure handling

Brainstorming and writing-plans both have their own surface-assumptions / push-back behaviors built in. The workflow does not paper over those — when a sub-skill asks a clarifying question, the workflow surfaces it to the user. If a stage errors, stop and print the failed step plus the resume command (`/workflow-feature-spec --resume-from <step>`).
