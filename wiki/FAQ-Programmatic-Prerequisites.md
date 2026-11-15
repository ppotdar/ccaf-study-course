# Why use a programmatic prerequisite instead of a prompt?

**Prompts make the right order *likely*; code makes the wrong order *impossible*.** This is Task 1.4, and it's [Rule 1](The-10-Rules.md) applied to the order of steps.

### “Some step orders are business-critical”

In many workflows, step B is only safe if step A already happened. Get the order wrong and the damage is real.

| Must happen first | Before this |
| --- | --- |
| Verify the caller's identity (`get_customer`) | Issue a refund (`process_refund`) |
| Confirm the account owner | Change the email or password on file |
| Get manager approval | Pay an invoice over $10,000 |

### “Prompt guidance has a non-zero failure rate”

You can write: “Always call `get_customer` to verify identity before calling `process_refund`.” Claude follows it **most** of the time. But a prompt is guidance, not a guarantee. Some conversations will skip the step: unusual phrasing, a customer who sounds very confident (“I'm John Smith, just refund it”), or a long conversation where the instruction gets less attention.

Even a 1% failure rate is 100 wrongly refunded accounts per 10,000 chats. For money or security, that's unacceptable.

### “Programmatic prerequisites do not”

A programmatic prerequisite is a check in **your code** that makes the wrong order impossible, whatever Claude decides:

```
def process_refund(customer_id, amount, session):
    if not session.verified_customer_id:
        return {"isError": True, "errorCategory": "validation",
                "message": "Verify the customer with get_customer before refunding."}
    if customer_id != session.verified_customer_id:
        return {"isError": True, "errorCategory": "permission",
                "message": "Refund must match the verified customer."}
    # only now does the refund actually run
```

If Claude tries `process_refund` too early, the code blocks it with a clear error, and Claude goes back and calls `get_customer`. The skip rate is now **zero**. A `PreToolUse` hook that blocks `process_refund` until verification is recorded does the same thing in a different place.

### The analogy

A prompt is a **sign** on the door: “Staff must badge in before entering the vault.” Most people comply. A programmatic prerequisite is the **lock**: the door won't open without a badge. For a vault, you want the lock.

### How it shows up on the exam

*“The agent calls `process_refund` before `get_customer` in 12% of conversations, causing refunds to the wrong accounts. What's the best fix?”*

| Option | Verdict |
| --- | --- |
| Add a bold system-prompt rule to always verify first | Wrong: still probabilistic |
| Add few-shot examples showing verification first | Wrong: still probabilistic |
| Add a [routing classifier](FAQ-Routing-Classifier.md) | Wrong: controls which tools exist, not their order |
| **Programmatic prerequisite that blocks `process_refund` until `get_customer` returns a verified ID** | **Right: guarantees the order** |

**How to tell you need it:** the mistake would cost money, break security or violate compliance, and the question says it fails some percentage of the time. Then the answer is code (a gate or hook), not more prompting.

**Remember:** Prompts make the right order likely; code makes the wrong order impossible. When the order matters, use code.

Where this comes up: [Rule 1](The-10-Rules.md) · [Task 1.4 Enforcement and handoff](Domain-1-Agentic-Architecture.md) · [Task 1.5 Hooks](Domain-1-Agentic-Architecture.md) · [Scenario playbook](Scenario-Playbook.md)


---

← [FAQ Spawning Subagents](FAQ-Spawning-Subagents.md) · [Home](README.md) · [FAQ Hooks](FAQ-Hooks.md) →

Unofficial CCAF study guide · not affiliated with Anthropic · [Interactive site](https://ppotdar.github.io/ccaf-study-course/) · © 2026 Puja Potdar · [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)
