# Can the synthesis agent read the coordinator's history?

**No. That idea is a trap, not a fact.** A subagent sees only what the coordinator puts in the prompt it sends. Nothing else carries over.

### The setup

- The **coordinator** talks to the user, plans the work, and spawns helpers.
- The **search subagent** goes out and finds sources.
- The **synthesis subagent** writes the final report.

It's natural to picture them as one team sharing a whiteboard: the coordinator saw everything, so the writer can surely look back at it. That's the wrong mental model.

### What actually happens

When the coordinator spawns a subagent with the `Task` tool, the subagent starts with a **brand-new, empty conversation**. It has its own system prompt (from its `AgentDefinition`), its own tools, and **only the text the coordinator puts in the prompt**.

It does **not** see the user's original message, the coordinator's planning, what the search subagent found, or any shared memory. Each subagent is like a contractor you email a brief to: they know what's in the email and nothing else. That's [Rule 5: subagents start blank](The-10-Rules.md).

### How the bug shows up

The search subagent finds 12 good sources. The coordinator spawns the synthesis subagent with “Write the final report on renewable energy adoption.” The report comes back generic, with no citations, as if the research never happened. From the synthesis agent's point of view, it didn't: nobody handed it the findings.

### The right fix

The coordinator passes the findings into the synthesis prompt explicitly, ideally as structured data. Keeping claim, source, page and date together keeps citations attached as content moves between agents.

```
Write the report using ONLY these findings:
[
  {"claim": "Solar capacity grew 24% in 2025",
   "source": "https://...", "page": 12, "published": "2026-02"},
  ...
]
```

### Distractors you'll see

| Distractor | Why it's wrong |
| --- | --- |
| “Enable shared memory so subagents read the coordinator's history” | No such setting; subagents are isolated by design |
| “Give the synthesis agent the web-search tool” | Duplicates the search agent's job and invites misuse of tools outside its role ([Rule 6](The-10-Rules.md)) |
| “Increase the synthesis agent's max_tokens” | Output length isn't the problem; the input never arrived |
| “Tell the synthesis agent to use all available findings” | It has no findings available to use |

**Remember:** Subagents only know what's in the prompt you send them. If the report is missing what another agent found, the coordinator never passed it along.

Where this comes up: [Rule 5](The-10-Rules.md) · [Task 1.3 Spawning subagents and passing context](Domain-1-Agentic-Architecture.md) · [Task 5.6 Provenance](Domain-5-Context-Management-and-Reliability.md) · [Scenario playbook](Scenario-Playbook.md)


---

← [FAQ Routing Classifier](FAQ-Routing-Classifier.md) · [Home](README.md) · [FAQ stop_reason](FAQ-stop_reason.md) →

Unofficial CCAF study guide · not affiliated with Anthropic · [Interactive site](https://ppotdar.github.io/ccaf-study-course/) · © 2026 Puja Potdar · [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)
