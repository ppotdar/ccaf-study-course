# Domain 2: Tool Design & MCP Integration

How to make Claude pick the right tool, recover from tool failures, and wire MCP servers into a team setup. The five task statements come first, then [deeper cards](Domain-2-Tool-Design-and-MCP.md) on schema design, idempotency, MCP primitives and transports, least privilege, and production MCP.

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

## Going deeper: designing tools Claude uses well

Exam questions in this domain often show a broken or weak tool definition and ask you to spot the flaw, so precision on schema anatomy matters more than breadth. These six cards go deeper than the task statements above.

### Y1 Tool descriptions as routing

**Key idea: descriptions work like a router’s decision table.** Claude reads every tool description and picks the best match. If two tools have similar descriptions, expect Claude to pick the wrong one now and then. The fix is **sharper, more distinguishing language** in the descriptions, not a longer system prompt.

#### Before and after

#### Weak: Claude has to guess

```
{
  "name": "lookup_order",
  "description": "Looks up orders."
}
```

Which orders? From what input? Returns what? It also overlaps with `list_orders` and `get_customer`.

#### Strong: Claude can route

```
{
  "name": "lookup_order",
  "description": "Get the status, items and delivery date of ONE order. Use when the customer gives an order number (e.g. #48812) or asks where a specific order is. Not for account details (use get_customer) or all orders (use list_orders). Returns: status, items, amount, shipped_at, delivered_at."
}
```

Says what it does, when to use it and when not, and what comes back.

| Part | Answers the question | Example phrase |
| --- | --- | --- |
| What it does | What does this tool actually do? | “Get the status, items and delivery date of ONE order” |
| When to use it (and not) | Is this the right tool for this request? | “Use when the customer gives an order number… Not for account details (use get_customer)” |
| What it returns | Will the result answer the question? | “Returns: status, items, amount, shipped_at, delivered_at” |

**Concept.** The `description` is the main signal Claude uses to decide **which** tool to call and **when**. Vague or overlapping descriptions cause **wrong-tool selection**, not just untidy docs. A good description states three things: what the tool does, when to use it (and when not to), and what it returns.

**Exam answer.** Rewrite the description to cover all three parts, and name the neighbouring tool explicitly: “For account details, use `get_customer`.”

**Trap.** Treating the description as human documentation; fixing misrouting with few-shot examples or a router; reformatting a description without changing what it says.

### Y2 Tool input schema design

**Exam fact: nullable fields reduce hallucination.** When a field genuinely may not apply, allowing `null` is safer than forcing the model to invent a plausible-looking value. This matters most in structured-output schemas, such as a tool used for data extraction (Task 4.3).

```
"po_number": {
  "type": ["string", "null"],
  "description": "Purchase order number. Use null if the document has none."
}
```

#### Worked example: a well-designed schema

```
{
  "name": "process_refund",
  "description": "Refund all or part of ONE verified order.
    Use only after get_customer has verified the customer.
    Returns refund_id and status.",
  "input_schema": {
    "type": "object",
    "properties": {
      "order_id": {
        "type": "string",
        "pattern": "^ORD-[0-9]{6}$",
        "description": "Order ID, e.g. ORD-048812"
      },
      "amount": {
        "type": "number",
        "minimum": 0.01,
        "maximum": 500,
        "description": "Refund amount in USD"
      },
      "reason": {
        "type": "string",
        "enum": ["damaged", "late", "wrong_item",
                 "duplicate_charge", "other"]
      },
      "reason_detail": {
        "type": "string",
        "maxLength": 200,
        "description": "Required when reason is 'other'"
      },
      "notify_customer": {
        "type": "boolean",
        "default": true,
        "description": "Email the customer. Defaults to true."
      }
    },
    "required": ["order_id", "amount", "reason"]
  }
}
```

| Field | Design choice | Why |
| --- | --- | --- |
| `order_id` | `pattern` + example in the description | Rejects malformed IDs before the call; shows Claude the exact format |
| `amount` | `number` with `minimum` / `maximum` | No negative or oversized refunds slip through as text |
| `reason` | `enum` with `"other"` | A fixed set of values downstream systems can rely on, with room for new cases |
| `reason_detail` | Optional, `maxLength`, says when it’s needed | Captures the extra context only when it’s useful |
| `notify_customer` | Optional with `default: true` | Claude can leave it out and the behaviour is still predictable |
| `required` | Only the three fields the refund can’t work without | Nothing forces Claude to make up a value |

**The schema guides Claude; the server still checks.** Constraints such as `maximum: 500` help Claude form valid calls, but the server (or a hook) must validate too. Never rely on the model alone for a hard limit ([card Y5](Domain-2-Tool-Design-and-MCP.md)).

#### Spot the flaw

| Flawed definition | Fix |
| --- | --- |
| `"reason": { "type": "string" }` | An `enum` of allowed reasons, plus `"other"` and a detail field |
| `"date": { "type": "string" }` | State the format: “ISO 8601, YYYY-MM-DD” (or `"format": "date"`) |
| All eight fields in `required` | Require only what the tool truly can’t work without |
| `"limit"` optional, no default stated | “Defaults to 20 if omitted” |
| Both `id` and `order_id` parameters | One clearly named parameter |
| `name: "handle"`, `description: "Handles requests"` | A specific verb_object name and a three-part description |

**Concept.** A tool definition has three parts: a **`name`** (a clear verb_object such as `process_refund`), a **`description`**, and an **`input_schema`** in JSON Schema. Use **enums and constraints** to narrow valid inputs up front instead of accepting free text and validating afterwards. Mark parameters **required or optional deliberately**, and give every optional field a sensible, stated default.

**Exam answer.** Free-text categories become an `enum` (plus `"other"` and a detail field); amounts become numbers with `minimum`/`maximum`; IDs get a `pattern`; dates say their format; optional fields say what happens when they’re left out.

**Trap.** Everything typed as a free-text string; every field required (Claude is pushed to invent values, the same lesson as extraction schemas in Task 4.3); an optional field with no stated default, so nobody knows what happens when it’s omitted.

### Y3 Tool error handling: categories, idempotency, partial success

**Decision rule: retry only what can succeed next time.** **Transient** errors (a timeout, a 503) trigger an **automatic retry with backoff**. **Business** errors (“item out of stock”) and **permission** errors (“not authorized”) are **not retried**: surface them to the user or escalate.

**Backoff** means waiting longer between each attempt, for example 1 s, then 2 s, then 4 s, with a maximum number of tries, so a struggling service isn’t hammered. Pair it with an idempotency key so the retries are safe.

| Category | Example | Retry? | What the agent does |
| --- | --- | --- | --- |
| Transient | Timeout, 503 | Yes | Retry, ideally with the same idempotency key |
| Validation | Bad date format | After fixing the input | Correct the argument and call again |
| Business | Refund over the $500 limit | No | Explain to the customer or escalate |
| Permission | Token lacks refund scope | No | Escalate or use another path |

#### Idempotency: why a retry can double-refund

1. Claude calls `process_refund` for $120.
2. The payment system **completes** the refund, but the response times out.
3. Claude sees a transient error and retries.
4. Without idempotency: a second $120 refund. With an `idempotency_key`: the server recognises the repeat and returns the original result.

```
{ "order_id": "ORD-048812", "amount": 120.00, "reason": "damaged",
  "idempotency_key": "refund-ORD-048812-2026-11-20-001" }
```

#### Partial success: report both halves

```
{
  "processed": [ { "order_id": "ORD-000101", "refund_id": "R-9001" },
                 { "order_id": "ORD-000102", "refund_id": "R-9002" } ],
  "failed":    [ { "order_id": "ORD-000103", "errorCategory": "business",
                   "isRetryable": false,
                   "message": "Refund exceeds the $500 limit; needs human approval" } ]
}
```

The agent can confirm the two refunds that worked and escalate only the third, rather than redoing everything or telling the customer nothing happened.

**Concept.** Structured errors tell the agent **what kind of failure** happened so it can choose the right recovery (Task 2.2). Two more ideas matter for production tools. **Idempotency:** calling the tool again with the same input must be safe, with no duplicate side effects. **Partial success:** when a call handles several items, return what succeeded alongside what failed, instead of all-or-nothing.

**Exam answer.** Error responses carry a category, a retryable flag and a clear message. Write tools that create or change things accept an **idempotency key** so a retry after a timeout can’t double-apply. Batch tools return `processed` and `failed` lists, each failure with its own category.

**Trap.** Retrying a non-idempotent write after a timeout (the first call may have succeeded, so the customer is refunded twice); failing a 10-item batch because 1 item failed; a generic “Operation failed.”

### Y4 MCP foundations: primitives and transports

**Key idea: choose the transport by who shares the server.** **stdio** is simplest for **local, single-user, trusted** integrations, such as tools on a developer’s own machine. A **remote** transport (SSE in study material, HTTP in current docs) fits when the server must be **shared across users or sessions**, or **deployed independently** of the client.

| Primitive | Controlled by | What it is | Example |
| --- | --- | --- | --- |
| **Tools** | The model | Actions Claude can call | `create_issue`, `process_refund` |
| **Resources** | The application | Readable data the app makes available | A database schema, a docs tree, issue summaries |
| **Prompts** | The user | Reusable prompt templates, often shown as slash commands | “Summarize this incident” template |

| Transport | Where the server runs | Typical use |
| --- | --- | --- |
| **stdio** | A local process on your machine; talks over standard input/output | Dev tools, local files, a local database |
| **HTTP** (Streamable HTTP) | A remote server reached over the network | Shared company services, SaaS integrations; the recommended remote option |
| **SSE** (Server-Sent Events) | A remote server, older style | Still common in study material and on the exam; now deprecated in favour of HTTP |

**Exam framing.** Study material often pairs “local = stdio, remote = SSE.” Current Claude Code docs mark SSE as deprecated and recommend HTTP for remote servers. If an option says SSE for a remote server and there’s no HTTP option, it’s the right idea.

**Concept.** The **Model Context Protocol (MCP)** is an open standard for connecting models to external tools and data: one protocol, many servers. The **host** (Claude Code, the Claude app, or your agent) runs an MCP client for each server it connects to. Servers expose three **primitives** and connect over a **transport**.

**Exam answer.** Choose the primitive by who starts it: the model (tool), the application (resource) or the user (prompt). Use stdio for a local tool on the developer’s machine and a remote transport (HTTP) for a shared service. For standard systems such as GitHub, use an existing community server rather than building your own.

**Trap.** Exposing a read-only catalog (a database schema, a docs tree) as dozens of tools instead of a resource; building a custom server for a system that already has one; assuming a remote server must use stdio.

### Y5 Tool authorization and least privilege

**Exam trap: the model as the only safeguard.** Relying on the model’s good judgment as the *only* protection against an over-privileged tool is a wrong-answer pattern. The exam consistently rewards enforcing least privilege at the **host and permission layer**, not through prompting alone.

| Integration | Credential | Scope |
| --- | --- | --- |
| Order lookup | `ORDERS_READ_TOKEN` | Read orders only |
| Refunds | `REFUNDS_TOKEN` | Create refunds; server rejects anything over $500 |
| GitHub (review agent) | `GITHUB_TOKEN` (fine-grained) | One repository, read-only |
| Ticketing | `JIRA_TOKEN` | Create and comment; no delete |

**Why the model can’t be the boundary.** Claude’s choices can be steered by what it reads: a web page or document can contain instructions (prompt injection). If the token itself can’t delete, a manipulated request to delete still fails. Least privilege limits the damage of any mistake.

**Concept.** Give each tool and each agent **only the permissions its job needs**. The **host and the server, not the model,** enforce the security boundary between tools and the systems behind them. Scope credentials **per integration** rather than sharing one broad credential across every tool.

**Exam answer.** Read-only tokens for lookup tools; a separate, narrowly scoped credential for writes; hard limits (such as the $500 refund cap) checked on the server; secrets passed as environment variables; each subagent’s tool list limited to its role.

**Trap.** A prompt line such as “never delete records” as the only protection; one admin token shared by every MCP server; trusting the model’s arguments without server-side validation.

### Y6 Advanced MCP: scopes, overlapping names, environment variables

**Exam fact: duplicate tool names are a real production failure.** With several MCP servers connected, ambiguous or duplicate tool names across servers lead to wrong calls. **Namespace or disambiguate tool names before deploying** a multi-server setup. Claude Code namespaces automatically (`mcp__github__search`); in your own agent, give tools unique names such as `github_search` and `jira_search`, plus distinct descriptions.

| Scope | Stored in | Loads in | Shared via git |
| --- | --- | --- | --- |
| **Local** (default) | `~/.claude.json`, under this project | This project, just you | No |
| **Project** | `.mcp.json` at the repo root | This project, whole team | Yes |
| **User** | `~/.claude.json` | All your projects | No |

Precedence when the same server is defined in several scopes: **local > project > user**. Claude Code connects once, using the whole entry from the highest one; fields are not merged. Study material often describes only two tiers (project vs user); that framing still answers most exam questions.

#### Overlapping tool names

Claude Code prefixes each MCP tool with its server name, so a `search` tool on two servers becomes `mcp__github__search` and `mcp__jira__search`. The names no longer collide, but Claude still needs descriptions that say which to use: “Search code and pull requests” versus “Search tickets and their comments.” Use the full prefixed names in permission rules and hook matchers.

#### Environment variable expansion

```
{
  "mcpServers": {
    "orders-api": {
      "type": "http",
      "url": "${ORDERS_API_URL:-https://api.example.com}/mcp",
      "headers": { "Authorization": "Bearer ${ORDERS_READ_TOKEN}" }
    }
  }
}
```

| Syntax | Meaning | Works in |
| --- | --- | --- |
| `${VAR}` | The value of `VAR` | `command`, `args`, `env`, `url`, `headers` |
| `${VAR:-default}` | `VAR` if set, otherwise `default` | Same fields; handy for dev vs production URLs |

**Concept.** In production you manage MCP servers across **scopes**, deal with **several servers exposing similar tools**, and keep configuration **flexible per environment** without committing secrets.

**Exam answer.** Team servers in the project’s `.mcp.json` with `${VAR}` for secrets and `${VAR:-default}` for environment-specific values; personal or experimental servers in user or local scope. When two servers both offer `search`, rely on Claude Code’s per-server names and give each tool a description that says how it differs; remove servers the task doesn’t need.

**Trap.** Hard-coding tokens in `.mcp.json`; putting personal experimental servers in the shared project file; assuming settings for the same server merge across scopes (the highest-precedence entry is used whole); connecting every available server “just in case”, which makes tool selection less reliable.

## Exam scenario lens: where this domain shows up

Tool design and MCP show up in all six scenarios, not just the agent ones.

| Scenario | How this domain appears | Typical question | Answer |
| --- | --- | --- | --- |
| **Structured Data Extraction** | Tool schema design with enums, constraints and nullable fields to reduce hallucinated output | “Extraction invents PO numbers for documents that don’t have one.” | Make the field nullable; use enums for categories (Y2) |
| **Customer Support Resolution Agent** | Structured errors (transient / business / permission) driving retry vs escalate | “The agent retries a refund the policy blocks, five times.” | Return `errorCategory: business`, `isRetryable: false`; explain or escalate (Y3) |
| **Multi-Agent Research System** | Multiple MCP servers with overlapping tools; disambiguation and least-privilege scoping per subagent | “Synthesis calls the wrong search tool, and has tools it shouldn’t.” | Distinct names and descriptions; scope each subagent’s tools to its role (Y5, Y6) |
| **Code Generation with Claude Code** | Local stdio MCP servers for dev-machine tools; project vs user MCP scope | “Every developer should get the team’s DB server; one dev wants to try another.” | Team server in `.mcp.json`; the experiment in user/local scope (Y4, Y6) |
| **Claude Code for CI/CD** | Idempotent tool calls so pipeline retries don’t cause duplicate side effects | “A rerun pipeline posted every PR comment twice.” | Make the posting tool idempotent (a key per finding) so reruns are safe (Y3) |
| **Developer Productivity** | Precise tool descriptions so Claude routes to the right internal tool | “The agent uses Grep instead of the semantic code_search tool.” | Rewrite code_search’s description: what, when, what it returns (Y1) |

## Common exam traps

The first four come from your slide deck; the rest come up repeatedly elsewhere in this domain.

**Trap 1:** **Retrying a business error (e.g. “insufficient inventory”) automatically, like a timeout.**

**Why it’s wrong:** Only transient errors should auto-retry, with backoff. Business and permission errors need surfacing or escalation, not repetition.

**Example:** Five retries of “out of stock” just produce five identical failures.

**Trap 2:** **Two tools with near-identical descriptions offered as an acceptable design.**

**Why it’s wrong:** Overlapping descriptions cause wrong-tool routing. Descriptions must clearly differentiate purpose and scope.

**Example:** `analyze_content` and `analyze_document`, both “Analyzes text.”

**Trap 3:** **Using a single broad credential shared across many tools “for simplicity.”**

**Why it’s wrong:** Least privilege means scoping credentials and permissions per tool or integration, not sharing one broad grant.

**Example:** A read-only review agent holding the same admin GitHub token as the deploy tool.

**Trap 4:** **Choosing SSE for a simple, single-user, local dev integration.**

**Why it’s wrong:** stdio is the simpler, correct choice for local single-user cases; keep remote transports (SSE, or HTTP in current docs) for shared or remote deployments.

**Example:** A server that reads files on your own laptop → stdio.

**Trap 5:** **Fixing wrong-tool selection with few-shot examples or a router.**

**Why it’s wrong:** Descriptions are the routing signal. Rewrite them: what, when (and when not), what it returns.

**Example:** “Agent calls get_customer for order questions” → improve both descriptions.

**Trap 6:** **Accepting free text and validating afterwards.**

**Why it’s wrong:** Constrain inputs in the schema: enums, patterns, min/max, formats.

**Example:** `"reason": {"type": "string"}` → an enum with `"other"`.

**Trap 7:** **Making every parameter required.**

**Why it’s wrong:** Required fields push Claude to invent values. Require only what the tool can’t work without, and state defaults for the rest.

**Example:** Optional `limit` → “defaults to 20.”

**Trap 8:** **Retrying a write that isn’t idempotent.**

**Why it’s wrong:** A timeout doesn’t mean the action failed. Use an idempotency key so repeats are safe.

**Example:** A retried refund pays the customer twice.

**Trap 9:** **Returning all-or-nothing results for batch tools.**

**Why it’s wrong:** Report what succeeded and what failed, each failure with its category.

**Example:** 9 of 10 refunds worked; escalate just the tenth.

**Trap 10:** **Trusting the model to respect a security boundary.**

**Why it’s wrong:** Scope credentials and validate on the server; the model’s choices can be manipulated.

**Example:** “Never delete” in the prompt, but the token can delete.

**Trap 11:** **Exposing a read-only catalog as many tools.**

**Why it’s wrong:** Use an MCP resource so the agent can see what exists without exploratory calls.

**Example:** A database schema browsed via 30 `describe_table` calls.

## Key terminology

Short definitions to recognize instantly on the exam. The last column shows where each term is covered in depth.

| Term | Meaning | Covered in |
| --- | --- | --- |
| **MCP (Model Context Protocol)** | An open standard for connecting AI models to external tools, resources and prompt templates in a consistent way. One protocol, many servers. | Y4, Task 2.4 |
| **Tool schema** | The JSON definition of a tool: its `name`, `description` and `input_schema`. Claude uses it to decide whether and how to call the tool. | Y2 |
| **Enum constraint** | Restricting a parameter to a fixed set of valid values, e.g. `"enum": ["damaged", "late", "other"]`. Reduces malformed or unpredictable tool calls. | Y2 |
| **Nullable field** | A schema field allowed to be `null` when it doesn’t apply, e.g. `"type": ["string", "null"]`. Reduces pressure on the model to invent a value. | Y2, Task 4.3 |
| **Idempotency** | Calling a tool again with the same input produces the same result with no extra side effects, so it’s safe to retry. Usually done with an idempotency key. | Y3 |
| **Partial success** | A response that reports which parts of a multi-part action succeeded and which failed, instead of one pass/fail result. | Y3 |
| **Transient error** | A temporary failure (timeout, rate limit, 503) that is generally safe to **retry with backoff**. | Y3, Task 2.2 |
| **Business error** | A failure caused by domain rules (e.g. out of stock, refund over the limit). Not a bug, and a retry won’t fix it: explain or escalate. | Y3, Task 2.2 |
| **Permission error** | A failure due to insufficient authorization. Surface or escalate it; never retry it silently. | Y3, Task 2.2 |
| **stdio transport** | A local MCP transport: the server runs as a subprocess on the same machine as the client. Simple and single-user. | Y4 |
| **SSE transport** | A remote MCP transport that lets one server be shared across users and sessions, deployed independently of the client. Current Claude Code docs mark SSE as deprecated in favour of HTTP; both mean “remote” on the exam. | Y4 |
| **Principle of least privilege** | Granting a tool or agent only the minimum access its specific task needs, **enforced at the host layer**, not by prompting. | Y5 |

## Check yourself

This domain's interactive quiz is on the [live site](https://ppotdar.github.io/ccaf-study-course/#d2). The same questions, with answers, are on [Practice Questions](Practice-Questions.md).


---

← [Domain 4 Prompt Engineering and Structured Output](Domain-4-Prompt-Engineering-and-Structured-Output.md) · [Home](README.md) · [Domain 5 Context Management and Reliability](Domain-5-Context-Management-and-Reliability.md) →

Unofficial CCAF study guide · not affiliated with Anthropic · [Interactive site](https://ppotdar.github.io/ccaf-study-course/) · © 2026 Puja Potdar · [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)
