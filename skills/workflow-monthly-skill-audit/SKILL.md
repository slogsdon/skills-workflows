---
name: workflow-monthly-skill-audit
description: Run the monthly Claude Code skill hygiene pipeline — skill-audit (rank skills by usage + quality) → consolidate-memory (prune stale memories) → vault-lint (catch orphans created by deprecated workflows). Use when the 1st of the month rolls around, when the user says "monthly skill audit", "is my context layer healthy", "skill hygiene check", or after deprecating a workflow / removing a skill. Do NOT use for single-skill review (read the SKILL.md directly) or in the same week as the last audit. Output goes to ~/Downloads + Inbox/.
---

# Skill: /workflow-monthly-skill-audit

Three-stage hygiene pipeline. Mirrors the message of the LLM-context-files post: context decays without maintenance.

## When to use

- 1st of every month (cron candidate — pair with `/loop` or `/schedule`).
- After deprecating a workflow or removing skills, to catch downstream effects.
- When the user explicitly asks "is my skill layer healthy".

## When NOT to use

- Single-skill review → read the SKILL.md directly.
- Inside the same week as the last audit — usage data won't have shifted.

## Steps

| Step name    | Stage                | Invokes                              |
|--------------|----------------------|--------------------------------------|
| `audit`      | Skill audit          | `skill-audit`                        |
| `memory`     | Memory consolidation | `anthropic-skills:consolidate-memory`|
| `vault-lint` | Vault lint           | `vault-lint`                         |

### Stage 1: Skill audit — step `audit` — invokes `skill-audit`

Produces `~/Downloads/skill-audit-report.md`.

Output: report file exists.
Checkpoint: review top/bottom 5 before pruning memory.

### Stage 2: Memory consolidation — step `memory` — invokes `anthropic-skills:consolidate-memory`

Reflective pass over MEMORY.md; merges duplicates, prunes stale facts, fixes the index.

Output: MEMORY.md updated; commit summary in conversation.
Checkpoint: confirm memory edits look correct.

### Stage 3: Vault lint — step `vault-lint` — invokes `vault-lint`

Catches orphan notes / broken links that may have been left by deprecated workflows.

Output: lint report in conversation.

## Resume protocol

Default: run `audit` → `memory` → `vault-lint` in order, with checkpoints between each.

`--resume-from <step>`: skip every step before `<step>` and begin from `<step>`. The named step itself runs.

If `<step>` is not in `{audit, memory, vault-lint}`, print:

    Invalid resume-from step: <step>
    Valid steps: audit, memory, vault-lint

…and stop.

### Required state when resuming

- `--resume-from audit` — same as a fresh run.
- `--resume-from memory` — `consolidate-memory` operates on MEMORY.md directly; it does not need the audit report. If the audit report is absent, warn the user that the typical flow uses audit findings to inform memory pruning, and ask whether to continue.
- `--resume-from vault-lint` — independent of prior steps; runs against the live vault.

Examples:

    /workflow-monthly-skill-audit --resume-from memory     # audit report from a prior session
    /workflow-monthly-skill-audit --resume-from vault-lint # just want the orphan check today

## Other flags

- `--no-pause` for cron-driven runs.

## Output destination

Stage 1 report goes to `~/Downloads/skill-audit-report.md` and is auto-copied to vault `Inbox/Skill Audit YYYY-MM-DD.md` if the user opts in (asked at Stage 1 checkpoint).

## Failure handling

If a stage errors, stop and print: which step failed, the underlying skill's error, and the resume command (`/workflow-monthly-skill-audit --resume-from <step>`).
