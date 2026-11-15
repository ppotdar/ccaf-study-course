# What does “spawning” a subagent mean?

**Spawning means creating and starting a new subagent:** launching a fresh worker to handle one job. The coordinator is running, decides it needs help, and spins up a helper with its own instructions.

### “Subagents are spawned via the `Task` tool”

There's no special command for creating helpers. The coordinator uses a **tool**, the same way it would call `lookup_order` or `web_search`. That tool is named `Task`.

```
Coordinator's response:
  tool_use: Task
    subagent: "web_researcher"
    prompt:   "Find 2025 data on EV market share. Return claims with source URLs and dates."
```

The SDK starts a `web_researcher` subagent with that prompt, lets it work, and returns its final answer to the coordinator as the tool result. To the coordinator, it called a tool and got a result back; the “tool” just happened to be a whole agent.

### “The coordinator's `allowedTools` must include `"Task"`”

Every agent has a list of tools it may use. If `"Task"` isn't on the coordinator's list, it **can't** spawn anyone and ends up doing everything itself.

**Classic exam question.** “The coordinator never delegates to subagents. Why?” Answer: `"Task"` is missing from its `allowedTools`.

### “Each subagent type is an `AgentDefinition`”

Before anything runs, you define the *kinds* of helpers available, like job descriptions on file:

| AgentDefinition field | Example for `web_researcher` |
| --- | --- |
| Description | “Searches the web for current data and returns cited claims” |
| System prompt | “You are a research assistant. Always include source URLs and publication dates.” |
| Tools | `web_search`, `fetch_page` only |

The coordinator uses each definition's description to pick the right helper for a subtask, the same way tool descriptions help Claude pick tools.

### “Subagents do not inherit the parent's history or memory”

Each spawned subagent starts **blank**. It knows its `AgentDefinition` plus the prompt from the `Task` call, and nothing else, so that prompt must contain everything it needs. See [Can the synthesis agent read the coordinator's history?](FAQ-Subagent-Context.md)

### The analogy

| Concept | Everyday equivalent |
| --- | --- |
| `AgentDefinition` | A job description on file |
| Spawning with `Task` | Hiring a contractor for one assignment and emailing the brief |
| `allowedTools` includes `"Task"` | The manager has permission to hire |
| Doesn't inherit history | The contractor knows only what's in the brief |

### Bonus: parallel spawning

The coordinator can spawn **several subagents at once** by putting multiple `Task` calls in a single response, rather than one per turn. Exam distractors for “make them run in parallel” include the Batch API and `fork_session`; neither is right.

**Remember:** Spawning = calling the `Task` tool to start a fresh helper from its `AgentDefinition`. No `"Task"` in `allowedTools`, no helpers. Helpers know only their brief. Several `Task` calls in one response run in parallel.

Where this comes up: [Task 1.3 Spawning subagents and passing context](Domain-1-Agentic-Architecture.md) · [Task 1.2 Coordinator and subagents](Domain-1-Agentic-Architecture.md) · [Cheat sheet](Cheat-Sheet.md)


---

← [FAQ Protecting Context](FAQ-Protecting-Context.md) · [Home](README.md) · [FAQ Programmatic Prerequisites](FAQ-Programmatic-Prerequisites.md) →

Unofficial CCAF study guide · not affiliated with Anthropic · [Interactive site](https://ppotdar.github.io/ccaf-study-course/) · © 2026 Puja Potdar · [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)
