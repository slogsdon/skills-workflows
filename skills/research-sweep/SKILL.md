---
name: research-sweep
description: Research a list of topics in parallel, adversarially fact-check every load-bearing claim, and merge the survivors into one decision-grade brief. Use when Shane says "deep research on X", "research these topics", "run a research sweep", "I need a brief on X before deciding", or hands over a list of questions to answer with sources. Do NOT use for a single quick lookup (just search), for reading the vault (use /vault-retrieve), or for mining one account or corpus (use /source-mine).
---

# Skill: /research-sweep [topic list]

Produce a brief whose every number you would defend in public — by researching each topic independently, then paying a second agent to destroy the first one's claims.

**Don't:** dispatch a researcher without the preamble from [research-preamble.md](../_workflow-lib/research-preamble.md) — an agent researching from memory looks identical to one researching from sources until you check a citation. **Don't** let the same agent research a topic and verify it; self-verification confirms, it does not test. **Don't** merge unrefuted and refuted claims into one brief without marking which is which.

## Steps

| Step name | Stage | Runs |
|-----------|-------|------|
| `scope` | Topic list + shared context | orchestrator, with Shane |
| `research` | One agent per topic, parallel | researcher subagents |
| `verify` | One agent per load-bearing claim | fact-checker subagents |
| `merge` | Single decision-grade brief | orchestrator |

### Stage 1: Scope — step `scope`

Settle three things before dispatching anything:

1. **The topic list.** Each topic gets its own agent. A topic is one question with a stated angle, not a subject area — "which trigger signals empirically correlate with closed-won, and on what evidence" beats "buying signals".
2. **The shared context block.** Product, positioning, and audience, pasted identically into every researcher prompt so findings come back usable rather than generic.
3. **The counter-position requirement.** Every topic must also research the case against itself. Name it explicitly in the prompt.

Output: a topic list and a context block.
Checkpoint: show Shane the topic list before spending N agents on it.

### Stage 2: Research — step `research` — one subagent per topic

Each prompt is, in order: the preamble (with `<DATE>` substituted), then

```
You are a deep research analyst. Research one topic exhaustively and return structured findings.

# Topic: <title>
<what to cover — enumerate the specific sub-questions, named frameworks, and
originating sources to find. Be blunt about how much of this literature is
vendor content with no underlying study. Also research the counter-position.>

## <Brand> context (the brand this research serves)
<the shared context block>
```

Run topics in parallel. Each returns findings with sources and a per-claim confidence.

Output: N sets of findings.
Checkpoint: none — flow straight into verification per topic as each lands.

### Stage 3: Verify — step `verify` — one subagent per load-bearing claim

A claim is load-bearing if the brief's recommendation changes when it is false. Verify those; skip the rest.

Each prompt is the preamble, then:

```
You are an adversarial fact-checker. Your default posture is skepticism. Another
researcher, studying "<topic>", produced this claim:

CLAIM: <claim>
THEIR EVIDENCE: <evidence>
THEIR SOURCES: <urls>
THEIR CONFIDENCE: <confidence>

Try to REFUTE it. Run your own independent searches — do not just re-read their
sources. Check whether: the sources actually say what's claimed; the data is stale;
it's vendor marketing being treated as fact; it generalizes from one anecdote; a
plausible-sounding number was fabricated; the opposite is also documented somewhere
credible.

Set refuted=true if the claim is wrong, materially overstated, or unsupported by
anything you can find. When genuinely uncertain, lean toward refuted=true and
explain. Always fill correctedClaim with the version you would defend.
```

The uncertainty default is the point: uncertain resolves to refuted, and `correctedClaim` is always populated, so a survivable version exists even when the original dies.

Output: a verdict per claim.
Checkpoint: report the refutation count before merging — a high rate means the research stage was too credulous, not that the brief is thin.

### Stage 4: Merge — step `merge`

Write one brief. Refuted claims are dropped or replaced by their `correctedClaim`, never silently kept. State what could not be verified and what would change the recommendation.

Output: the brief.

## Resume protocol

`--resume-from <step>` skips earlier steps. Valid steps: `scope`, `research`, `verify`, `merge`.
Required state: `research` needs the topic list and context block; `verify` needs the findings; `merge` needs the verdicts.

## Failure handling

A subagent that dies leaves its topic unresearched — say so in the brief rather than merging a silently short sweep.
