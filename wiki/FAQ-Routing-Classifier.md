# What is a routing classifier?

**A routing classifier is a separate component that sits in front of Claude and decides where a request should go before Claude ever sees it.** It reads the user's message, labels it (“billing question,” “order question”), and sends it to a particular tool, agent or workflow based on that label.

### A concrete picture

- **Without a router:** the customer types “Where's my order #48812?” Claude reads the message and the tool descriptions, then picks `lookup_order` itself.
- **With a router:** the message first hits a small model or rules engine trained on labeled examples. It outputs a label like `ORDER_INQUIRY`. Your code then calls `lookup_order` directly, or gives Claude only the order-related tools for that turn.

Think of a receptionist who hears why you called and transfers you to the right department, so the department never has to work out whether you're in the right place.

### Why it's usually the wrong answer on the exam

It appears as a distractor in two kinds of questions.

1. **Wrong tool picked.** The agent confuses `get_customer` and `lookup_order`, which both have one-line descriptions. A router would technically help, but it's a new component to train, label data for, deploy and maintain, and it works around the real problem. The root cause is vague tool descriptions; the fix is rewriting them. That's [Rule 3: proportionate beats elaborate](The-10-Rules.md).
2. **Steps done in the wrong order.** The agent sometimes calls `process_refund` before verifying the customer. A router decides *which* tools or agents handle a request. It doesn't control *what order* things happen in. The fix is a programmatic gate or hook that blocks the refund until identity is verified. That's [Rule 1: must-always rules go in code](The-10-Rules.md).

### A close cousin to watch for

“Train a separate classifier to decide escalations” is the same trap in Domain 5. The right answer there is explicit escalation criteria with few-shot examples in the system prompt.

**Remember:** A routing classifier is an extra gatekeeper model that picks the path for each request. On this exam it's almost always the over-engineered option; look for the simpler fix that addresses the cause.

Where this comes up: [Rules 1 and 3](The-10-Rules.md) · [Task 1.4 Enforcement](Domain-1-Agentic-Architecture.md) · [Task 2.1 Tool descriptions](Domain-2-Tool-Design-and-MCP.md) · [Task 5.2 Escalation](Domain-5-Context-Management-and-Reliability.md)


---

← [FAQ Coordinator Decomposition](FAQ-Coordinator-Decomposition.md) · [Home](README.md) · [FAQ Subagent Context](FAQ-Subagent-Context.md) →

Unofficial CCAF study guide · not affiliated with Anthropic · [Interactive site](https://ppotdar.github.io/ccaf-study-course/) · © 2026 Puja Potdar · [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)
