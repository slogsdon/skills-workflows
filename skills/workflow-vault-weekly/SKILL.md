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

## Steps

| Step name    | Stage             | Invokes             |
|--------------|-------------------|---------------------|
| `lint`       | Lint              | `vault-lint`        |
| `backlinks`  | Backlinks         | `backlinks`         |
| `index`      | Vault index       | `vault-index`       |
| `signals`    | Weekly signals    | `weekly-signals`    |
| `learnings`  | Weekly learnings  | `weekly-learnings`  |

### Stage 1: Lint — step `lint` — invokes `vault-lint`

Scans Concepts/, finds orphans, broken links, missing pages, contradictions. Writes `vault-lint-report.md`.

Output: report file exists.
Checkpoint: surface any blocking issues before continuing.

### Stage 2: Backlinks — step `backlinks` — invokes `backlinks`

Audits and repairs the backlink graph; flags structurally important orphans.

Output: backlink-audit summary in conversation.
Checkpoint: confirm fixes applied before rebuilding the index.

### Stage 3: Vault index — step `index` — invokes `vault-index`

Rebuilds the vault-index note from current state.

Output: `vault-index.md` updated.
Checkpoint: skip checkpoint by default — auto-continue.

### Stage 4: Weekly signals — step `signals` — invokes `weekly-signals`

Reads `patterns.md` + the week's daily notes; surfaces deferral patterns and flagged items.

Output: weekly-signals output (in conversation; can be saved if user asks).
Checkpoint: confirm signal review before surfacing publishable threads.

### Stage 5: Weekly learnings — step `learnings` — invokes `weekly-learnings`

Looks for publishable threads from the week's signals + vault deltas.

Output: list of 1–3 candidate threads.

## Resume protocol

Default: run `lint` → `backlinks` → `index` → `signals` → `learnings` in order, with checkpoints between each (except `index`, which auto-continues).

`--resume-from <step>`: skip every step before `<step>` and begin from `<step>`. The named step itself runs.

If `<step>` is not in `{lint, backlinks, index, signals, learnings}`, print:

    Invalid resume-from step: <step>
    Valid steps: lint, backlinks, index, signals, learnings

…and stop.

### Required state when resuming

The vault stages mostly read from the live vault, so missing prior outputs are usually fine — but a few steps depend on prior reports.

- `--resume-from lint` — same as a fresh run.
- `--resume-from backlinks` — runs against the live vault. If `vault-lint-report.md` is absent, warn the user that `lint` would normally have surfaced issues first; ask whether to continue.
- `--resume-from index` — runs against the live vault. No required prior state.
- `--resume-from signals` — needs `patterns.md` and at least the current week's daily notes. If `patterns.md` is missing, surface the gap and stop.
- `--resume-from learnings` — works best with the previous step's signals output. If signals weren't run this week, ask the user to paste the signals output or confirm we should derive learnings without them.

Examples:

    /workflow-vault-weekly --resume-from signals --no-pause  # cron-style mid-week signal pull
    /workflow-vault-weekly --resume-from learnings           # signals already reviewed manually

## Other flags

- `--skip <step>` to skip a step (e.g. `--skip learnings` to stop after signals).
- `--no-pause` to run end-to-end (use this for the cron-driven weekly run).

## Failure handling

If a stage errors, stop and print: which step failed, the underlying skill's error, and the resume command (`/workflow-vault-weekly --resume-from <step>`).
