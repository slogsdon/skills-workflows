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

## Steps

| Step name  | Stage         | Invokes          |
|------------|---------------|------------------|
| `draft`    | First draft   | (in-skill draft) |
| `humanize` | Voice pass    | `humanize`       |
| `style`    | Style pass    | `ms-style-pass`  |

### Stage 1: First draft — step `draft`

Two paths depending on the input:

- **From an idea / outline** (no existing draft): write a first draft in Shane's voice, anchored on whatever vault evidence exists. ~600–1200 words. Save to `Inbox/Draft - <slug>.md`.
- **From an existing draft** (user pastes or names a vault note): read it; skip drafting; proceed to `humanize`.

Before writing prose, write the outline with a word budget per H2 — tables and
code blocks are their own line items, not free. Check the running count at each
section boundary, not at the end. Over budget means cut a section or merge two,
never trim adjectives: lexical trimming preserves the structure that caused the
overrun.

Output: draft file in vault Inbox.
Checkpoint: confirm draft direction before voice pass.

### Stage 2: Voice pass — step `humanize` — invokes `humanize`

Removes AI-typical patterns (hedging, list-iness, em-dash overuse, throat-clearing) and restores Shane's conversational rhythm. Preserves technical precision.

Output: humanized draft (in-place edit to the Inbox file).
Checkpoint: skim before style pass.

### Stage 3: Style pass — step `style` — invokes `ms-style-pass`

Applies Microsoft Writing Style Guide term preferences, bias-free language rules, and heading conventions. Doesn't touch voice.

Output: style-corrected draft (in-place edit).
Checkpoint: confirm draft is publish-ready before handoff.

### Handoff (NOT a step — explicit to user)

After `style`, print:

```
─── Draft staged ───
File: Inbox/Draft - <slug>.md
Word count: <N>

Ready for /publish-post when you are. That skill will:
  - generate brand artifacts (blog hero, OG card, LinkedIn companion)
  - run any final style checks
  - publish to shane.logsdon.io

Or, to iterate first: edit the draft directly and re-run /workflow-stage-draft --resume-from humanize.
```

## Resume protocol

Default: run `draft` → `humanize` → `style` in order, with checkpoints between each.

`--resume-from <step>`: skip every step before `<step>` and begin from `<step>`. The named step itself runs.

If `<step>` is not in `{draft, humanize, style}`, print:

    Invalid resume-from step: <step>
    Valid steps: draft, humanize, style

…and stop.

### Required state when resuming

Before running the resume step, confirm the listed inputs exist. If anything is missing, prompt the user for it.

- `--resume-from draft` — same as a fresh run; needs the topic / outline / source note.
- `--resume-from humanize` — needs an existing draft file (in `Inbox/` or pasted in conversation). If the user names a different file, accept it; otherwise default to `Inbox/Draft - <slug>.md`. Confirm the file path before running humanize.
- `--resume-from style` — needs a draft that has already been through humanize (or that the user explicitly says is voice-correct). Confirm the file path; warn if the file looks unedited (no recent changes since draft).

Examples:

    /workflow-stage-draft --resume-from humanize  # user edited the draft manually, re-run voice pass
    /workflow-stage-draft --resume-from style     # voice is fine, just want the style pass

## Other flags

- `--no-pause` to run end-to-end (use only when you've already iterated and just want a final pass).

## Session open

Check `Inbox/STATE.md` or the draft project's directory for a STATE.md. If present, read `General rules` and `Known failure modes` — these carry learned patterns (e.g. which humanize anti-patterns recur in Shane's technical writing) so they don't need to be re-discovered.

## Stage quality gate

After each stage: does the output meet the stage goal (draft = core argument present and in Shane's voice; humanize = AI-typical patterns removed without losing technical precision; style = terms and headings conform)? If it partially misses, retry once with an explicit note on the gap before presenting or continuing. On second failure, surface the issue rather than auto-continuing.

Humanize and style passes should be validated by re-reading with a critic framing (does this read like a human wrote it?) rather than the same pass that produced it.

## Session close

If the humanize or style pass hit recurring issues, append them to this SKILL.md under a `## Known failure modes` section. Patterns that compound here improve every future draft run.

## Failure handling

If `humanize` flags content it can't safely de-AI without losing technical accuracy, it stops and surfaces the conflict to the user. `style` then operates on whatever `humanize` produced. On any failure, print the failed step plus the resume command (`/workflow-stage-draft --resume-from <step>`).
