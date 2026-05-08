---
name: workflow-design
description: Run the full design pipeline for a brand or one-off artifact — design-plan (strategy) → design-system (tokens + showcase) → an artifact skill (linkedin / blog-hero / etc.) → screenshot-html (PNG export). Use when the user says "design a brand", "I need the full design system + assets", "set up a new brand from scratch", "design pipeline for X", or when starting visual work on a brand that has no DESIGN.md yet. Do NOT use when the brand already exists and you just need one artifact — call the artifact skill directly.
---

# Skill: /workflow-design

Orchestrates the four-stage design pipeline. Replaces the "which design skill comes first" question with a single invocation.

## When to use

- Brand has no `./design/<slug>/DESIGN.md` yet AND user wants one or more artifacts.
- User explicitly asks for "the full pipeline" / "set up the brand from scratch".

## When NOT to use

- Brand already has a design system; just need one artifact → call the artifact skill directly.
- User wants to iterate on existing tokens → call `design-system` directly.

## Steps

| Step name     | Stage              | Invokes                       |
|---------------|--------------------|-------------------------------|
| `plan`        | Strategic intent   | `design-plan`                 |
| `system`      | Token system       | `design-system`               |
| `artifacts`   | Artifact generation| `design-<artifact>` (varies)  |
| `screenshots` | PNG export         | `screenshot-html`             |

### Stage 1: Strategic intent — step `plan` — invokes `design-plan`

Asks the brand-strategy questions (audience, voice, hard NOs, mood references) and writes `./design/<slug>/DESIGN-PLAN.md`.

Output: `./design/<slug>/DESIGN-PLAN.md` exists.
Checkpoint: confirm DESIGN-PLAN.md before generating tokens.

### Stage 2: Token system — step `system` — invokes `design-system`

Reads DESIGN-PLAN.md and produces `DESIGN.md` (Google design.md spec format), `tokens.css`, and `showcase.html`.

Output: three files in `./design/<slug>/`.
Checkpoint: confirm `showcase.html` renders correctly in a browser before generating any artifacts.

### Stage 3: Artifact generation — step `artifacts` — invokes the artifact skill the user named

Asks: "Which artifact(s) do you want to generate? (linkedin-post / instagram-post / blog-hero / quote-card / business-card / podcast-cover / youtube-thumbnail / twitter-card / newsletter-header / talk-slide / speaker-bio-card / twitch-panels / obs-alert-overlay / obs-scene-pack / stream-overlay / link-in-bio / landing-page / ui-components / carousel-slide / youtube-channel-art / audit-report)"

For each chosen artifact, invokes the matching `design-<artifact>` skill with the brand slug. Produces an HTML file in `./design/<slug>/artifacts/`.

Output: one or more `.html` files in `artifacts/`.
Checkpoint: confirm artifacts before screenshotting.

### Stage 4: PNG export — step `screenshots` — invokes `screenshot-html`

Runs the screenshot-html script against `./design/<slug>/artifacts` with output to `./design/<slug>/screenshots`.

Output: PNG files matching the artifact set.

## Resume protocol

Default: run `plan` → `system` → `artifacts` → `screenshots` in order, with checkpoints between each.

`--resume-from <step>`: skip every step before `<step>` and begin from `<step>`. The named step itself runs.

If `<step>` is not in `{plan, system, artifacts, screenshots}`, print:

    Invalid resume-from step: <step>
    Valid steps: plan, system, artifacts, screenshots

…and stop.

### Required state when resuming

Before running the resume step, confirm the listed inputs exist. If anything is missing, prompt the user for it (paste, file path, or re-run from an earlier step).

- `--resume-from plan` — same as a fresh run; needs only the brand slug.
- `--resume-from system` — needs `./design/<slug>/DESIGN-PLAN.md`. If absent, ask the user to paste the plan or run from `plan`.
- `--resume-from artifacts` — needs `./design/<slug>/DESIGN.md`, `./design/<slug>/tokens.css`, and `./design/<slug>/showcase.html`. Ask the user which artifact(s) to generate before proceeding.
- `--resume-from screenshots` — needs at least one `.html` file in `./design/<slug>/artifacts/`. List the files found and confirm before screenshotting.

Examples:

    /workflow-design --resume-from system     # already have a DESIGN-PLAN.md, generate tokens
    /workflow-design --resume-from artifacts  # tokens look good, generate the LinkedIn post
    /workflow-design --resume-from screenshots --no-pause  # batch the PNG export

## Other flags

- `--skip <step>` to skip a step (e.g. `--skip screenshots` if user only wants HTML).
- `--no-pause` to run end-to-end without checkpoints.

## Failure handling

If any stage errors, stop and print: which step failed, the underlying skill's error, and the resume command (`/workflow-design --resume-from <step>`).
