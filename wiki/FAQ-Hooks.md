# How do hooks work, and when should you use them?

**Hooks are your code that runs automatically at fixed points in Claude's agent loop.** Claude doesn't choose whether a hook runs; it runs every time. That's why hooks are how you **guarantee** a rule instead of asking for it.

### Where hooks sit

```
Claude requests a tool
        │
   ┌────▼─────┐   allow / block / modify / redirect
   │PreToolUse│ ◄── your code inspects the CALL before it runs
   └────┬─────┘
        │
   tool runs
        │
   ┌────▼──────┐  clean / trim / normalize / mask
   │PostToolUse│ ◄── your code inspects the RESULT before Claude sees it
   └────┬──────┘
        │
 Claude reads the result
```

Other hook points exist (when a session starts, when the user submits a prompt), but the exam focuses on these two.

### What each hook can do

| `PreToolUse`: sees the call before it runs | `PostToolUse`: sees the result before Claude does |
| --- | --- |
| **Allow** the call through | **Normalize** formats (Unix timestamps to ISO dates, codes to labels) |
| **Block** it, with a reason Claude can read | **Trim** 45 fields down to the 5 that matter |
| **Modify** the arguments | **Mask** sensitive data (card number to •••• 4417) |
| **Redirect**: block and say what to do instead | **Log** for auditing |

### Examples

**1. Refund limit (PreToolUse: block and redirect):** Policy: refunds over $500 need a human.

**How:** 
```
# simplified, illustrative
def pre_tool_use(tool_name, args):
    if tool_name == "process_refund" and args["amount"] > 500:
        return block(reason="Refunds over $500 need human approval. "
                            "Call escalate_to_human with the case summary.")
    return allow()
```
Claude reads the reason and calls `escalate_to_human`. No prompt wording can make a $600 refund go through.

**2. Messy dates from three systems (PostToolUse: normalize):** The orders, shipping and returns tools return dates in three formats.

**How:** A hook converts them all to ISO 8601 before Claude reads them, so return-window math is always right.

**3. Bloated lookups (PostToolUse: trim):** `lookup_order` returns 45 fields.

**How:** The hook keeps order ID, status, amount, dates and items: [Rule 10](FAQ-Protecting-Context.md) done in code.

**4. Dangerous commands in Claude Code (PreToolUse: block):** An agent with Bash access could run destructive commands.

**How:** A hook on the Bash tool blocks commands matching `rm -rf` or `git push --force` to the main branch, whatever the conversation says.

**5. PII masking (PostToolUse: mask):** Customer records include full card numbers.

**How:** A hook masks them before they enter context, so they never appear in logs, summaries or handoffs. Critical in regulated environments.

### Hooks vs a check inside the tool

The [prerequisite FAQ](FAQ-Programmatic-Prerequisites.md) put the verification check inside `process_refund`. Both approaches are deterministic and both satisfy Rule 1. The difference is where the check lives:

|  | Check inside the tool | Hook |
| --- | --- | --- |
| Where | In the tool's own code | Around every tool call, in one place |
| Best when | The rule belongs to that one tool | The rule spans many tools, or you don't own the tool (a third-party MCP server) |
| Example | `process_refund` checks verification | One hook masks PII from every tool's results |

On the exam, either counts as “programmatic.” What matters is choosing code over prompts.

### When not to use a hook

Hooks suit rules a program can check: an amount over $500, a missing field, a command matching a pattern. They're bad at **judgment calls**:

- “Escalate when policy is ambiguous” needs reasoning: explicit criteria and few-shot examples in the prompt.
- “Be empathetic with frustrated customers” is tone: the prompt.
- “Pick the right tool”: tool descriptions.

**Exam shortcut.** Can you write the rule as an `if` statement? Then it's a hook. Does it need judgment? Then it's a prompt.

### Where hooks go wrong

1. **Blocking without a reason.** A bare “denied” can make Claude retry the same call forever, a [runaway loop](FAQ-Runaway-Loops.md). Always say why and what to do instead.
2. **Over-broad rules.** A hook that blocks every `rm` also blocks `rm temp.txt`. Match precisely.
3. **Over-trimming.** A `PostToolUse` hook that drops `delivery_date` breaks return-window questions later.
4. **Checking too late.** Flagging a $600 refund in `PostToolUse` means the money already moved. Blocking belongs in `PreToolUse`.
5. **Using hooks for judgment.** Pattern-matching angry words to force escalation is sentiment-based escalation, which the exam calls unreliable.

### Exam signals

| Question says | Answer |
| --- | --- |
| “Must never exceed / must always / compliance / slips X% of the time” | `PreToolUse` hook (or prerequisite gate) |
| “Tools return data in different formats” | `PostToolUse` hook to normalize |
| “Tool results are verbose and fill context” | `PostToolUse` hook to trim |
| “A refund over the limit was flagged after processing” | Wrong hook point: move the check to `PreToolUse` |
| An option says “add it to the system prompt” for a hard rule | Reject: probabilistic |

**Remember:** Hooks are code that runs on every tool call: `PreToolUse` decides whether a call may happen; `PostToolUse` cleans what comes back. If a rule can be written as an `if` and must never be broken, it belongs in a hook.

Where this comes up: [Rule 1](The-10-Rules.md) · [Task 1.5 Agent SDK hooks](Domain-1-Agentic-Architecture.md) · [Task 1.4 Enforcement](Domain-1-Agentic-Architecture.md) · [Task 5.1 Trimming tool output](Domain-5-Context-Management-and-Reliability.md) · [Cheat sheet](Cheat-Sheet.md)


---

← [FAQ Programmatic Prerequisites](FAQ-Programmatic-Prerequisites.md) · [Home](README.md) · [FAQ Adaptive Decomposition](FAQ-Adaptive-Decomposition.md) →

Unofficial CCAF study guide · not affiliated with Anthropic · [Interactive site](https://ppotdar.github.io/ccaf-study-course/) · © 2026 Puja Potdar · [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)
