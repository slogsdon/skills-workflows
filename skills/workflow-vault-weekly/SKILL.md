---
name: workflow-vault-weekly
description: Run the full weekly vault-health pipeline — vault-lint (consistency check) → backlinks (graph repair) → vault-index (rebuild overview) → weekly-signals (deferral / pattern surface) → weekly-learnings (publishable thread surface). Use when /weekly-review is invoked, on Friday afternoons, when the user says "weekly vault review", "audit my vault", "what should I publish this week", or after a week of heavy capture activity. Skips stages whose preconditions aren't met (e.g. weekly-learnings needs ≥3 daily notes from the week).
---

# Skill: /workflow-vault-weekly

Runs the five vault-health skills in dependency order so the later stages have current data from the earlier ones. Use weekly.

## When to use

- Friday-afternoon vault review.
- After a week of heavy note capture, before reviewing for patterns.
- The user explicitly says "weekly vault review" / "what should I publish this week".

## When NOT to use

- Daily; the daily rituals (`/morning`, `/eod`, `/log`) cover the within-day cadence.
- For specific known issues (one orphan note, one stale tag) — call the targeted skill directly.

## Stages

### Stage 1: Lint — invokes `vault-lint`

Scans Concepts/, finds orphans, broken links, missing pages, contradictions. Writes `vault-lint-report.md`.

Output: report file exists.
Checkpoint: surface any blocking issues before continuing.

### Stage 2: Backlinks — invokes `backlinks`

Audits and repairs the backlink graph; flags structurally important orphans.

Output: backlink-audit summary in conversation.
Checkpoint: confirm fixes applied before rebuilding the index.

### Stage 3: Vault index — invokes `vault-index`

Rebuilds the vault-index note from current state.

Output: `vault-index.md` updated.
Checkpoint: skip checkpoint by default — auto-continue.

### Stage 4: Weekly signals — invokes `weekly-signals`

Reads `patterns.md` + the week's daily notes; surfaces deferral patterns and flagged items.

Output: weekly-signals output (in conversation; can be saved if user asks).
Checkpoint: confirm signal review before surfacing publishable threads.

### Stage 5: Weekly learnings — invokes `weekly-learnings`

Looks for publishable threads from the week's signals + vault deltas.

Output: list of 1–3 candidate threads.

## Resume / skip

- `--resume-from <N>` to start from stage N.
- `--skip <N>` to skip stage N (e.g. `--skip 5` to stop after signals).
- `--no-pause` to run end-to-end (use this for the cron-driven weekly run).
