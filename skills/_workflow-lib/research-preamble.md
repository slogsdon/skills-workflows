# Research Preamble

> Shared sourcing contract for `research-sweep` stages. Prepend verbatim to every
> researcher and fact-checker subagent prompt. Lifted from the prompts Shane
> hand-wrote 20+ times across the LeadSurface research sessions — do not soften it.

```
TOOLS: WebSearch and WebFetch are deferred in this session. Load them FIRST with a single call:
ToolSearch with query "select:WebSearch,WebFetch" and max_results 5.
Then research. Run at least 8-12 distinct searches. Fetch primary sources (vendor pricing pages,
docs, job posts, community threads, changelogs) rather than relying on search snippets.
Today's date is <DATE>. Prioritize <YEAR-1>-<YEAR> sources; explicitly flag anything older than
<YEAR-2> as potentially stale. NEVER invent a URL, price, or statistic. If you cannot verify
something, say so and mark confidence low. Do not write any files.
```

Substitute `<DATE>` with today's actual date before dispatch. The subagent has no
reliable clock — an unsubstituted placeholder produces confidently mis-dated research.

## Why each line is load-bearing

- **Deferred-tool load** — WebSearch/WebFetch are not in the default subagent toolset. Without this the agent silently researches from memory.
- **8–12 searches** — a floor, not a target. One search produces a summary of a summary.
- **Primary sources over snippets** — snippets are where fabricated statistics come from.
- **Stale flagging** — anything pre-`<YEAR-2>` in a fast-moving market is a different market.
- **Never invent** — the single most important line. A plausible fabricated number survives every downstream check that isn't explicitly hunting it.
- **Do not write any files** — researchers return text; only the orchestrator writes.
