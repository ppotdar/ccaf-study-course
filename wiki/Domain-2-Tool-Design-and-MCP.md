# Domain 2: Tool Design & MCP Integration

How to make Claude pick the right tool, recover from tool failures, and wire MCP servers into a team setup.

### 2.1 Tool interfaces and descriptions

**Concept.** The description is the main signal Claude uses to choose a tool. A good one states purpose, input format, an example query, edge cases, and when to use it instead of a similar tool.

**Exam answer.** Misrouting between `get_customer` and `lookup_order`: expand both descriptions with boundaries. Overlapping tools (`analyze_content` vs `analyze_document`): rename and differentiate, or split a generic tool into purpose-specific ones. Also check the system prompt: a keyword-heavy instruction can override good descriptions.

**Trap.** Few-shot examples to fix routing; [a routing classifier](FAQ-Routing-Classifier.md); merging every tool into one.

### 2.2 Structured MCP errors

| Category | Example | Right reaction |
| --- | --- | --- |
| Transient | Timeout, 503 | Retry, often locally in the subagent |
| Validation | Bad date format | Fix the input, retry |
| Business | Refund exceeds policy | Do not retry; explain or escalate |
| Permission | Not authorized | Do not retry; escalate or use another path |

**Concept.** MCP tools signal failure with `isError: true`. Each error category needs a different reaction.

**Exam answer.** Return `errorCategory`, `isRetryable` and a readable description; business-rule violations add `retriable: false` and a customer-friendly explanation. Subagents recover transient errors locally and pass up only what they cannot fix. Keep “the lookup failed” distinct from “the lookup worked and found nothing.”

**Trap.** A generic “Operation failed”; returning `[]` on a timeout, which looks like a real no-match.

### 2.3 Distributing tools and tool_choice

| tool_choice | Behavior | Use when |
| --- | --- | --- |
| `{"type":"auto"}` | May call a tool or answer in text | Normal agent turns |
| `{"type":"any"}` | Must call some tool | Structured output with several schemas |
| `{"type":"tool","name":"X"}` | Must call tool X | Force a first step or one schema |
| `{"type":"none"}` | No tools | Pure text turn |

**Concept.** More tools make selection harder, and agents given tools outside their role misuse them.

**Exam answer.** Give each subagent only its role's 4–5 tools. Replace generic tools with constrained ones (`fetch_url` becomes `load_document` that validates URLs). Give a narrow cross-role tool for a frequent need: synthesis gets `verify_fact` for simple lookups, while complex checks go through the coordinator.

**Trap.** Giving synthesis full web search to cut latency; one mega-agent with every tool.

### 2.4 MCP servers in Claude Code and agents

**Concept.** Scope decides who gets a server: project `.mcp.json` (committed, shared) or user `~/.claude.json` (personal). MCP has three primitives: **tools** (model-invoked), **resources** (read-only content the app exposes), **prompts** (user-invoked templates).

**Exam answer.** Shared servers go in `.mcp.json` with environment variable expansion (`Bearer ${GITHUB_TOKEN}`) so no secret is committed. All servers' tools are discovered at connect time and available together. Expose catalogs (schemas, doc trees) as **resources** to avoid exploratory calls. If the agent prefers Grep over a better MCP tool, improve that tool's description. Prefer a community server over a custom one for standard integrations.

**Trap.** Hard-coded tokens in `.mcp.json`; team servers in `~/.claude.json`; building a custom GitHub server.

### 2.5 Built-in tools

| Tool | Use it to | Example |
| --- | --- | --- |
| Grep | Search file **contents** | Callers of `processRefund`, an error string |
| Glob | Find files by **name pattern** | `**/*.test.tsx` |
| Read | Read a whole file | Follow imports from an entry point |
| Edit | Change a unique text match | One function |
| Write | Create or overwrite a file | Fallback when Edit's anchor isn't unique |
| Bash | Run commands | Tests, git, builds |

**Concept.** [Grep searches file contents. Glob finds files by name or path pattern.](FAQ-Grep-Glob-Read.md) Read reads a whole file, Edit changes a unique text match, Write creates or overwrites a file, Bash runs commands.

**Exam answer.** Explore incrementally: Grep for entry points, then Read to follow the flow. To trace usage through wrapper modules, find the exported name and Grep for it. When Edit's anchor text is not unique, Read the file and Write it back.

**Trap.** [Glob to search code text; Grep to find files by name](FAQ-Grep-Glob-Read.md); reading every file up front.

## Check yourself

This domain's interactive quiz is on the [live site](https://ppotdar.github.io/ccaf-study-course/#d2). The same questions, with answers, are on [Practice Questions](Practice-Questions.md).


---

← [Domain 4 Prompt Engineering and Structured Output](Domain-4-Prompt-Engineering-and-Structured-Output.md) · [Home](README.md) · [Domain 5 Context Management and Reliability](Domain-5-Context-Management-and-Reliability.md) →

Unofficial CCAF study guide · not affiliated with Anthropic · [Interactive site](https://ppotdar.github.io/ccaf-study-course/) · © 2026 Puja Potdar · [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)
