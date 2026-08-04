# Changelog

## 0.4.0 — 2026-08-04

**Added three skills**, each replacing a prompt Shane had been hand-writing
repeatedly. Counts are from a prompt-and-skill audit over 539 sessions
(2026-04-03 → 2026-08-04), sampling 271 opening prompts.

- **`research-sweep`** — fan out one researcher per topic, adversarially
  fact-check every load-bearing claim, merge the survivors into one brief.
  Replaces the largest single pattern in the corpus: 33 deep-research
  openings + 25 fact-check openings + 20 tool-preamble headers, **78 of 271**.
  The pipeline had been retyped end to end twice over the same 12 topics
  (2026-07-30 and 2026-07-31).
- **`adversarial-diff-verify`** — check a builder agent's structured claim
  against the file and its actual git diff, one verdict per assertion.
  3 openings, all during the design-system v3 fan-out.
- **`plugin-manifest-reference`** — packaging ground truth: the plugin.json
  and marketplace.json field sets in use, settings keys, cache resolution,
  and why a plugin-bundled CLAUDE.md does not load. The same question was
  asked in 5 sessions across 3 repos.

**Added `_workflow-lib/research-preamble.md`** — the deferred-tool and
sourcing contract shared by both `research-sweep` stages, lifted verbatim from
the hand-written prompts rather than paraphrased.

Note: `plugin-manifest-reference` separates verified facts from one genuinely
open question (cross-plugin dependency declaration) rather than guessing at it.
Its natural home would have been `skills-engineering-reference`, retired the
same day; `skills-workflows` already carries `agents-md-generator`, its closest
sibling.

## 0.3.0

Pipeline orchestrators, publish-post, agents-md-generator, work-tracking.
