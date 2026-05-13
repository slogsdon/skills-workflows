---
name: agent-harness
description: Orchestrate external agent CLI harnesses (pi, hermes) in turn-by-turn sessions from within Claude Code. Use when the user wants to delegate tasks to pi or hermes, chain multiple agent turns, compare outputs across harnesses, or build multi-agent workflows where Claude acts as the coordinator.
---

# Agent Harness Orchestration

You are orchestrating external agent harnesses (pi, hermes) via the `agent-turn` wrapper script. You are the coordinator — you decide what prompt to send each turn, read the response, and determine next steps.

## Prerequisites

Locate the wrapper script relative to the repo root:
```bash
AGENT_TURN="$(git rev-parse --show-toplevel)/scripts/agent-turn"
```

Use `$AGENT_TURN` in place of `agent-turn` throughout all Bash calls in this skill.

## Session ID Convention

Generate a stable session ID at the start of a task and reuse it across all turns:
```bash
SESSION="$(date +%Y%m%d)-$(echo "$TASK_DESCRIPTION" | tr ' ' '-' | tr '[:upper:]' '[:lower:]' | cut -c1-30)"
echo "$SESSION"  # e.g. 20260512-summarize-quarterly-report
```

Store it in context — you'll need it for every subsequent turn.

## Invoking a Turn

```bash
"$AGENT_TURN" <tool> <session-id> "<prompt>"
# tool: pi | hermes
# session-id: stable string, reused across turns
# prompt: the message for this turn
```

Output is printed to stdout and logged to `/tmp/agent-sessions/<session-id>.log`.

## Turn-by-Turn Loop

Follow this loop until the task is complete or you hit a stop condition:

1. **Send turn** — call `agent-turn` with the current prompt
2. **Read response** — the full output is in stdout
3. **Evaluate** — decide: done / needs follow-up / needs correction / switch tools
4. **Act**:
   - Done → summarize and present to user
   - Follow-up → compose next prompt, go to step 1
   - Correction → compose corrective prompt with specific feedback, go to step 1
   - Switch tools → use the other harness with a new session ID, carry relevant context forward in the prompt

## Switching Between Harnesses

pi and hermes use separate session IDs. To hand off, synthesize the relevant context from the current session log and pass it as part of the first prompt in the new harness:

```bash
# Read current session log for context
cat /tmp/agent-sessions/${SESSION}.log

# Start hermes turn with context summary
HERMES_SESSION="${SESSION}-hermes"
"$AGENT_TURN" hermes "$HERMES_SESSION" "Context from prior session: [summary]. Now: [new task]"
```

## Reviewing Session History

```bash
# Full log for a session
cat /tmp/agent-sessions/<session-id>.log

# Just agent responses
grep -A999 '\[agent\]' /tmp/agent-sessions/<session-id>.log
```

## Stop Conditions

Stop the loop and present results when:
- The agent's response fully satisfies the original task
- The agent indicates it cannot proceed (escalate to user)
- You've made 3+ corrective turns on the same issue without progress (escalate to user)
- The user has specified a max number of turns

## Multi-Agent Pattern (pi + hermes in parallel)

For tasks where you want both harnesses to attempt independently:

```bash
# Run both concurrently, capture outputs
PI_OUT=$("$AGENT_TURN" pi "$SESSION-pi" "$PROMPT")
HERMES_OUT=$("$AGENT_TURN" hermes "$SESSION-hermes" "$PROMPT")

# Compare and synthesize
echo "=== pi ===" && echo "$PI_OUT"
echo "=== hermes ===" && echo "$HERMES_OUT"
```

Then synthesize or pick the better response before continuing.

## Example Orchestration

```
Task: Research and summarize recent advances in vector databases

Turn 1 (pi): "List the 5 most significant advances in vector databases in 2025. Be specific — names, benchmarks, capabilities."
→ Response: [list]

Turn 2 (pi): "For each item you listed, identify one concrete use case where it changes what's now possible."
→ Response: [use cases]

Turn 3 (pi): "Synthesize this into a 3-paragraph summary suitable for a technical blog post introduction."
→ Response: [draft]

Evaluate: Draft is good. Present to user.
```
