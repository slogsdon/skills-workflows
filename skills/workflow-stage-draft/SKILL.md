---
name: workflow-stage-draft
description: Draft a blog post or article and prepare it for publishing — idea → first draft → humanize (remove AI-typical patterns) → ms-style-pass (term + bias + heading conventions). Use when the user says "draft a post about X", "write up this idea as a blog", "prepare a draft on Y", "stage a draft for publishing", or after /learned surfaces a publishable thread. Hands off to publish-post for the heavyweight pipeline (artifact generation + publishing); does NOT publish itself.
---

# Skill: /workflow-stage-draft

Three-stage drafting pipeline. Outputs a clean, voice-corrected, style-checked draft that's ready for `/publish-post` to take over.

## When to use

- User says "draft a post on X" / "write this up as a blog" / "prepare a draft on Y".
- A `/learned` or `/weekly-learnings` run surfaced a publishable thread and the user wants to develop it.
- An existing draft needs a voice + style pass before publishing.

## When NOT to use

- The piece is already drafted AND voice-corrected → call `/publish-post` directly.
- Capturing notes, not drafting → use `/log` or the obsidian skill.
- Ghost-writing in a different voice → use `/ghost` first, then this workflow.

## Stages

### Stage 1: First draft

Two paths depending on the input:

- **From an idea / outline** (no existing draft): write a first draft in Shane's voice, anchored on whatever vault evidence exists. ~600–1200 words. Save to `Inbox/Draft - <slug>.md`.
- **From an existing draft** (user pastes or names a vault note): read it; skip drafting; proceed to Stage 2.

Output: draft file in vault Inbox.
Checkpoint: confirm draft direction before voice pass.

### Stage 2: Voice pass — invokes `humanize`

Removes AI-typical patterns (hedging, list-iness, em-dash overuse, throat-clearing) and restores Shane's conversational rhythm. Preserves technical precision.

Output: humanized draft (in-place edit to the Inbox file).
Checkpoint: skim before style pass.

### Stage 3: Style pass — invokes `ms-style-pass`

Applies Microsoft Writing Style Guide term preferences, bias-free language rules, and heading conventions. Doesn't touch voice.

Output: style-corrected draft (in-place edit).
Checkpoint: confirm draft is publish-ready before handoff.

### Handoff (NOT a stage — explicit to user)

After Stage 3, print:

```
─── Draft staged ───
File: Inbox/Draft - <slug>.md
Word count: <N>

Ready for /publish-post when you are. That skill will:
  - generate brand artifacts (blog hero, OG card, LinkedIn companion)
  - run any final style checks
  - publish to shane.logsdon.io

Or, to iterate first: edit the draft directly and re-run /workflow-stage-draft --resume-from 2.
```

## Resume / skip

- `--resume-from <N>` to start from stage N (e.g. user edits the draft manually, then `--resume-from 2` re-runs humanize).
- `--skip <N>` to bypass a stage (rarely useful here — the stages are tight).
- `--no-pause` to run end-to-end (use only when you've already iterated and just want a final pass).

## Failure handling

If Stage 2 (humanize) flags content it can't safely de-AI without losing technical accuracy, it stops and surfaces the conflict to the user. Stage 3 then operates on whatever Stage 2 produced.
