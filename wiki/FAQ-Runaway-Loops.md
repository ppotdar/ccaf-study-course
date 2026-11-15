# How do you safeguard against runaway loops?

**Layer a few cheap guards around the loop.** [`stop_reason`](FAQ-stop_reason.md) stays the normal way to stop; these guards only catch the cases where something has gone wrong.

### 1. An iteration cap (the classic safety net)

Count loop turns and stop at a sensible maximum, such as 25. The Agent SDK offers the same thing as a max-turns setting, so you don't have to hand-roll it.

```
for turn in range(MAX_TURNS):        # e.g. 25
    resp = client.messages.create(...)
    if resp.stop_reason != "tool_use":
        return resp                   # normal exit: end_turn
    run_tools_and_append(resp)
raise LoopLimitReached()              # safety exit: something went wrong
```

**Exam point.** Hitting the cap is a **failure signal**, not a finish line. Log it and escalate (for example to a human with a structured handoff) rather than quietly returning whatever you have.

### 2. Budgets on tokens, cost and time

Turn counts aren't the only thing that can run away. Whichever budget hits first stops the run:

- a total token or dollar budget per task,
- a wall-clock timeout (say, 5 minutes),
- a limit on calls to expensive tools.

### 3. Repeated-call detection

Real runaway loops usually look like the same tool called with the same arguments again and again. Track the last few `(tool, arguments)` pairs. If the same one repeats three times, stop and investigate instead of spending more turns.

### 4. Fix the causes, not just the symptom

| Cause | What happens | Fix |
| --- | --- | --- |
| `tool_result` not appended | Claude never sees the result, so it keeps asking | Always append results with the matching `tool_use_id` |
| Generic errors (“Operation failed”) | Claude can't tell retryable from permanent, so it retries forever | Structured errors with `errorCategory` and `isRetryable` |
| Vague tool descriptions | Claude cycles between tools looking for the right one | Clear descriptions with boundaries |
| Rule enforced only in the prompt | Claude keeps trying a forbidden action | A hook that blocks it and redirects |

### What the exam tests

Expect a question where an agent loops too long or burns tokens. The traps:

- **“Raise the iteration cap.”** Treats the symptom and lets the loop run even longer.
- **“Make the cap the main stopping rule.”** The anti-pattern from [Rule 8](The-10-Rules.md).
- **“Tell Claude in the prompt to stop after a few tries.”** Probabilistic and unreliable.

The right answer usually fixes the cause: append tool results, structure the errors, or add a hook. The cap stays only as the safety net.

**Remember:** `stop_reason` ends a healthy loop. A turn cap, budgets and repeat detection catch an unhealthy one. When a safeguard trips, treat it as a bug to fix, not a normal ending.

Where this comes up: [Rule 8](The-10-Rules.md) · [Task 1.1 Agentic loops](Domain-1-Agentic-Architecture.md) · [Task 2.2 Structured MCP errors](Domain-2-Tool-Design-and-MCP.md) · [Task 1.5 Hooks](Domain-1-Agentic-Architecture.md)


---

← [FAQ stop_reason](FAQ-stop_reason.md) · [Home](README.md) · [FAQ Independent Review](FAQ-Independent-Review.md) →

Unofficial CCAF study guide · not affiliated with Anthropic · [Interactive site](https://ppotdar.github.io/ccaf-study-course/) · © 2026 Puja Potdar · [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)
