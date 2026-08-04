---
name: adversarial-diff-verify
description: Check a builder agent's structured claim against the file and its actual git diff, confirming or refuting each assertion independently. Use when a subagent reports "done" with a JSON or bulleted claim about what it changed, when Shane says "verify that conversion", "check what the builder actually did", "adversarially verify this diff", or after any fan-out where agents edited files in parallel. Do NOT use for reviewing code quality (use a code-review skill), for verifying a factual claim about the world (use /research-sweep's verify stage), or when there is no diff to read.
---

# Skill: /adversarial-diff-verify [file] [builder claim]

Read what the builder actually did, not what it said it did — a claim and a diff disagreeing is the single most common failure in parallel agent work, and it is invisible unless someone reads both.

**Don't:** accept a claim because it is detailed and confident; the fabricated ones are the most detailed. **Don't** read only the diff — a diff shows what moved, not what the file now says, and claims about preservation are only checkable against the whole file. **Don't** return a verdict per file; return one per assertion, because builders are usually right about most of them and wrong about one.

## Inputs

- `file` — the path the builder edited.
- `builder claim` — its structured report. Typically JSON with fields like `changes[]`, `contentPreserved`, `buildPassed`, plus task-specific assertions.

If either is missing, ask. Do not infer the claim from the diff — that defeats the check.

## Steps

1. **Read the file IN FULL.** Not a grep, not a range. Preservation claims are unfalsifiable without the whole text.
2. **Read the whole diff:** `cd <repo> && git diff -- <file>`. If the builder already committed, use `git show <sha> -- <file>`.
3. **Decompose the claim into assertions.** Each one gets its own verdict. A typical builder report yields 5–15.
4. **Test each assertion against the evidence**, in this order of suspicion:
   - **Preservation claims** ("kept verbatim: …") — check each named item is present *and unchanged*. This is where claims fail most often.
   - **Negative claims** ("no new fact, no invented number", "zero figures shipped") — these need a search to confirm, not a reading. Run the grep. A negative asserted without a grep is unverified, not true.
   - **Build/test claims** (`buildPassed: true`) — re-run it. Do not take the flag.
   - **Change claims** ("wrapped in the shell, added X") — confirm the diff actually contains them.
   - **Reasoning claims** ("every figure type fails its entry requirement") — check the reasoning against the stated rule, not against plausibility.
5. **Return a verdict per assertion:** `confirmed`, `refuted`, or `unverifiable` — with the evidence line or diff hunk that decided it. When uncertain, `refuted`, matching the fact-check default.

## Output

```
<file>
  confirmed:    <n>
  refuted:      <n>   ← list each with the contradicting evidence
  unverifiable: <n>   ← list each with what would settle it
```

Lead with the refuted ones. A clean run reports zero refuted and still lists what was unverifiable — silence on unverifiable assertions reads as confirmation.

## Fan-out

One verifier per file, dispatched in parallel, each blind to the others' results. Do not let one agent verify several files — it starts pattern-matching the second file against the first and stops reading.
