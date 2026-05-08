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

## Stages

### Stage 1: Business context — invokes `biz-brief`

Generates a 1-page Business Context Brief and saves it to `docs/superpowers/briefs/YYYY-MM-DD-<feature>.md`.

Output: brief file exists.
Checkpoint: confirm brief is accurate before brainstorming.

### Stage 2: Technical spec — invokes `superpowers:brainstorming`

Brainstorms the technical spec with the brief loaded as context. Saves spec to `docs/superpowers/specs/YYYY-MM-DD-<feature>.md`.

Output: spec file exists.
Checkpoint: confirm spec coverage before writing the plan.

### Stage 3: Implementation plan — invokes `superpowers:writing-plans`

Reads the spec and produces a task-by-task plan at `docs/superpowers/plans/YYYY-MM-DD-<feature>.md`.

Output: plan file exists, ready to hand off to `subagent-driven-development` or `executing-plans`.

## Resume / skip

- `--resume-from <N>` to start from stage N.
- `--no-pause` to run end-to-end (rare — usually you want the checkpoints here).

## Failure handling

Brainstorming and writing-plans both have their own surface-assumptions / push-back behaviors built in. The workflow does not paper over those — when a sub-skill asks a clarifying question, the workflow surfaces it to the user.
