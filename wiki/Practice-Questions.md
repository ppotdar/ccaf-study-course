# Practice Questions

104 scenario questions grouped by domain. Try each one before opening the answer. For a scored, shuffled run, use the [practice exam on the live site](https://ppotdar.github.io/ccaf-study-course/#practice). Download: [CSV](https://github.com/ppotdar/ccaf-study-course/blob/main/ccaf_mcq_bank.csv) · [JSON](https://github.com/ppotdar/ccaf-study-course/blob/main/ccaf_mcq_bank.json)

**Jump to:** [Domain 1](#domain-1-agentic-architecture--orchestration) · [Domain 2](#domain-2-tool-design--mcp-integration) · [Domain 3](#domain-3-claude-code-configuration--workflows) · [Domain 4](#domain-4-prompt-engineering--structured-output) · [Domain 5](#domain-5-context-management--reliability)

## Domain 1: Agentic Architecture & Orchestration

**Q01** · Task 1.1 · Customer Support

Your agent loop ends when the assistant's text contains the phrase 'task complete'. In production, some runs stop early and others never stop. What is the correct termination logic?

- **A.** Raise the iteration cap to 50 and treat reaching it as the normal stop
- **B.** Instruct Claude in the system prompt to always end with 'TASK COMPLETE'
- **C.** Keep looping while stop_reason is "tool_use" and stop when it is "end_turn"
- **D.** Stop as soon as a response contains any text block alongside tool calls

<details><summary>Show answer</summary>

**C.** Keep looping while stop_reason is "tool_use" and stop when it is "end_turn"

The API tells you whether Claude wants a tool (tool_use) or is finished (end_turn). Parsing natural language and iteration caps are anti-patterns; caps are only a safety net.

</details>

**Q02** · Task 1.1 · Multi-Agent Research

After running a tool, your agent calls the exact same tool with the same input again and again. The tool itself works correctly. What is the most likely cause?

- **A.** Tool results aren't appended before the next request
- **B.** max_tokens is too low for the model to read the tool output
- **C.** The temperature is too high, which makes the model's tool selection repetitive
- **D.** tool_choice is set to "auto" instead of "any"

<details><summary>Show answer</summary>

**A.** Tool results aren't appended before the next request

Claude only knows a tool ran if you send back a tool_result (matching tool_use_id) in the next user turn. Without it, Claude still sees an unanswered request and asks again.

</details>

**Q03** · Task 1.1 · Structured Data Extraction

A long extraction response returns stop_reason "max_tokens", and your loop treats it as a finished answer. What should the loop do instead?

- **A.** Treat it exactly like end_turn, because no tool was requested and the response already contains text
- **B.** Switch tool_choice to "any" so the model is forced to finish
- **C.** Treat it as truncated: raise max_tokens or continue
- **D.** Resend the identical request until stop_reason becomes end_turn

<details><summary>Show answer</summary>

**C.** Treat it as truncated: raise max_tokens or continue

max_tokens means the output was cut off. It is neither a tool request nor a natural finish. Only end_turn signals completion.

</details>

**Q04** · Task 1.2 · Multi-Agent Research

A research system was asked about 'the impact of remote work'. The report covers productivity in depth but says nothing about commercial real estate, cities, or mental health. Logs show every subagent completed its assigned subtask correctly. What is the root cause?

- **A.** The document-analysis subagent filtered out non-academic sources
- **B.** The coordinator decomposed the topic too narrowly when assigning subtasks
- **C.** The synthesis subagent dropped findings because its context was full
- **D.** The web-search subagent used queries that were too generic

<details><summary>Show answer</summary>

**B.** The coordinator decomposed the topic too narrowly when assigning subtasks

If every subagent did its job and whole areas are missing, the gaps were never assigned. Coverage is the coordinator's decomposition responsibility.

</details>

**Q05** · Task 1.2 · Multi-Agent Research

Your coordinator always invokes all five subagents, even for simple factual questions that one web search would answer. Latency and cost are high. What is the best fix?

- **A.** Cache every subagent's output so repeated questions are cheaper
- **B.** Merge the five subagents into one agent that holds all their tools
- **C.** Run all five subagents through the Message Batches API to halve cost
- **D.** Let the coordinator pick subagents by query complexity

<details><summary>Show answer</summary>

**D.** Let the coordinator pick subagents by query complexity

A coordinator should route dynamically rather than push every query through the full pipeline. Batching adds latency; merging agents hurts tool selection.

</details>

**Q06** · Task 1.3 · Multi-Agent Research

The synthesis subagent writes a report that ignores everything the document-analysis subagent found. Both ran successfully under the same coordinator. What is the fix?

- **A.** Pass the analysis findings explicitly in synthesis's prompt
- **B.** Enable shared memory so subagents read the coordinator's conversation
- **C.** Give the synthesis subagent the document-analysis tools as well
- **D.** Increase the synthesis subagent's max_tokens so it can read more history

<details><summary>Show answer</summary>

**A.** Pass the analysis findings explicitly in synthesis's prompt

Subagents start with isolated context and inherit nothing automatically. Anything they need must be passed in their prompt, ideally in a structured format.

</details>

**Q07** · Task 1.3 · Multi-Agent Research

Your coordinator spawns four independent research subagents one per turn, waiting for each to finish before starting the next. How do you run them in parallel?

- **A.** Have the coordinator emit multiple Task tool calls in a single response
- **B.** Use fork_session four times from the coordinator's current state
- **C.** Submit the four subagent prompts through the Message Batches API
- **D.** Set tool_choice to "any" so the coordinator calls more tools per turn

<details><summary>Show answer</summary>

**A.** Have the coordinator emit multiple Task tool calls in a single response

Parallel subagents are spawned by issuing several Task calls in one coordinator response. Batch API is asynchronous with no latency guarantee; fork_session is for divergent exploration.

</details>

**Q08** · Task 1.3 · Multi-Agent Research

You defined three AgentDefinitions, but the coordinator never spawns any subagent and answers everything itself. What configuration is most likely missing?

- **A.** "Task" is not included in the coordinator's allowedTools
- **B.** fork_session has not been enabled for the coordinator session
- **C.** The subagents are not listed in the project's CLAUDE.md file
- **D.** The subagents' system prompts do not mention the coordinator

<details><summary>Show answer</summary>

**A.** "Task" is not included in the coordinator's allowedTools

In the Agent SDK, subagents are spawned via the Task tool, so the coordinator must be allowed to use it.

</details>

**Q09** · Task 1.3 · Multi-Agent Research

The final report cites the right sources but often with the wrong page numbers. Findings are passed to synthesis as long paragraphs of prose. What change best fixes attribution?

- **A.** Pass findings as structured content plus source metadata
- **B.** Drop page numbers from the report and cite only the source URLs
- **C.** Add an instruction telling the synthesis agent to double-check every citation before it writes
- **D.** Let the synthesis agent re-fetch every source to confirm the page numbers

<details><summary>Show answer</summary>

**A.** Pass findings as structured content plus source metadata

Structured findings keep claim-to-source metadata intact through handoffs. A prompt reminder is probabilistic; re-fetching duplicates the search agent's job.

</details>

**Q10** · Task 1.4 · Customer Support

In about 8% of conversations the agent calls process_refund before get_customer has verified the caller's identity. Which fix guarantees the order?

- **A.** Few-shot examples that always show get_customer being called first
- **B.** A bold system-prompt rule stating that identity must always be verified before any refund is processed
- **C.** A code-level gate: no process_refund until the ID is verified
- **D.** Lowering the temperature so the agent follows the instructions more reliably

<details><summary>Show answer</summary>

**C.** A code-level gate: no process_refund until the ID is verified

When a business rule must hold every time, enforce it in code. Prompts and examples reduce but never eliminate the failure rate.

</details>

**Q11** · Task 1.4 · Customer Support

A customer writes in one message about a late delivery, a duplicate charge, and an address change. What is the best handling pattern?

- **A.** Escalate automatically, because multi-issue messages exceed agent scope
- **B.** Answer all three from the first tool lookup to keep the reply fast
- **C.** Split into three items, investigate each, reply once
- **D.** Resolve the first issue now and ask the customer to open separate tickets for the other two so each is tracked

<details><summary>Show answer</summary>

**C.** Split into three items, investigate each, reply once

Decompose multi-concern requests into distinct items, investigate each, and synthesize a single unified resolution.

</details>

**Q12** · Task 1.4 · Customer Support

Human agents complain that escalated cases arrive with no context, so they must re-ask the customer everything. What should the AI agent do when escalating?

- **A.** Send a structured handoff summary with the escalation
- **B.** Forward the entire raw conversation transcript so the human sees every message exactly as the customer wrote it
- **C.** Tell the customer to repeat their details to the human agent
- **D.** Escalate earlier in the conversation so there is less to repeat

<details><summary>Show answer</summary>

**A.** Send a structured handoff summary with the escalation

A structured handoff protocol lets the human continue instead of restarting. Raw transcripts push the summarization work onto the human.

</details>

**Q13** · Task 1.5 · Customer Support

Three MCP tools return dates as Unix timestamps, ISO 8601 strings, and 'MM/DD' text. The agent sometimes miscalculates return-window eligibility. What is the most reliable fix?

- **A.** A PostToolUse hook that normalizes every date format
- **B.** Few-shot examples showing how to compare dates in different formats
- **C.** A system-prompt instruction telling Claude to convert all dates to ISO 8601
- **D.** Rewriting all three backend services so they return ISO 8601

<details><summary>Show answer</summary>

**A.** A PostToolUse hook that normalizes every date format

PostToolUse hooks transform tool results deterministically before the model reasons on them. Rewriting backends works but is far more effort than needed.

</details>

**Q14** · Task 1.5 · Customer Support

Policy says any refund above $500 must be approved by a human. How should you enforce this?

- **A.** A system-prompt rule listing the $500 limit in capital letters
- **B.** A PostToolUse check that flags refunds above $500 after they process so a human can review them
- **C.** A sentiment classifier that escalates customers asking for large refunds
- **D.** A hook that blocks refunds over $500 and routes to escalation

<details><summary>Show answer</summary>

**D.** A hook that blocks refunds over $500 and routes to escalation

Interception before the call runs gives a deterministic guarantee. Checking after the refund executes is too late.

</details>

**Q15** · Task 1.6 · Developer Productivity

An engineer asks the agent: 'Checkout latency doubled last week - find out why.' Which decomposition strategy fits best?

- **A.** A single prompt containing the entire codebase and the latency graph
- **B.** A fixed prompt chain that analyzes every service in the same order, one after another, on every run
- **C.** Adaptive decomposition that re-plans as findings arrive
- **D.** Parallel subagents for all 40 services at once, each with identical instructions

<details><summary>Show answer</summary>

**C.** Adaptive decomposition that re-plans as findings arrive

Open-ended investigations need subtasks generated from what is discovered. Fixed chains suit predictable, known steps such as structured reviews.

</details>

**Q16** · Task 1.6 · Claude Code for CI/CD

You need a repeatable review of every PR with the same steps: security, correctness, then test coverage. Which decomposition fits best?

- **A.** One open-ended prompt asking for 'a thorough review'
- **B.** Prompt chaining: the same fixed sequence of steps each time
- **C.** A coordinator that spawns a new research subagent per changed line
- **D.** Adaptive decomposition that invents new subtasks for every PR

<details><summary>Show answer</summary>

**B.** Prompt chaining: the same fixed sequence of steps each time

When the steps are predictable, prompt chaining gives consistency. Adaptive decomposition is for open-ended investigation.

</details>

**Q17** · Task 1.7 · Developer Productivity

You resume yesterday's named investigation session with --resume. Overnight, teammates changed three files the agent had analyzed, and its answers now describe the old code. What is the best fix?

- **A.** Run /compact so the old analysis takes up less context
- **B.** Tell the resumed session which files changed
- **C.** Re-run the entire analysis from scratch every morning
- **D.** Use fork_session so the new branch picks up the latest file versions

<details><summary>Show answer</summary>

**B.** Tell the resumed session which files changed

Resume when most prior context is valid, and inform the agent of specific changes for targeted re-analysis. If most results are stale, start fresh with an injected summary instead.

</details>

**Q18** · Task 1.7 · Code Generation

After a long analysis of a legacy module, you want to compare two refactoring approaches without redoing the analysis or letting the two attempts interfere. What should you use?

- **A.** fork_session from the shared analysis baseline
- **B.** Plan mode, so both approaches are drafted in one plan
- **C.** --resume twice on the same session, trying one approach after the other
- **D.** Two brand-new sessions that each repeat the analysis

<details><summary>Show answer</summary>

**A.** fork_session from the shared analysis baseline

Forking creates independent branches from the same baseline, ideal for comparing alternatives.

</details>

## Domain 2: Tool Design & MCP Integration

**Q19** · Task 2.1 · Customer Support

Customers who say 'check order #48812' often trigger search_customers instead of search_orders. Both tools have one-line descriptions. What is the most effective first fix?

- **A.** Expand both descriptions with formats, examples, boundaries
- **B.** Add a routing classifier that picks the tool before Claude runs
- **C.** Merge both tools into one search tool that accepts any query and decides internally what to look up
- **D.** Add few-shot examples of order questions to the system prompt

<details><summary>Show answer</summary>

**A.** Expand both descriptions with formats, examples, boundaries

Tool descriptions are the main signal for tool selection. Fixing them addresses the root cause with the smallest change.

</details>

**Q20** · Task 2.1 · Customer Support

Tool descriptions are clear, yet the agent calls get_account_history whenever a customer uses the word 'history'. The system prompt says: 'Always review history before answering.' What should you change?

- **A.** Add more tools so the agent has better alternatives
- **B.** Force tool_choice to the correct tool on every turn
- **C.** Reword the keyword-sensitive system-prompt instruction
- **D.** Rename get_account_history to a name that doesn't contain the word 'history'

<details><summary>Show answer</summary>

**C.** Reword the keyword-sensitive system-prompt instruction

System prompt wording can create unintended keyword associations that override well-written tool descriptions.

</details>

**Q21** · Task 2.2 · Customer Support

An MCP tool returns 'Operation failed' for both network timeouts and policy denials. The agent retries policy denials five times before giving up. What is the best fix?

- **A.** Tell the agent in the prompt not to retry too often
- **B.** Cap the agent at two retries for every tool error
- **C.** Return errors with a category and an isRetryable flag
- **D.** Return an empty result instead of an error for policy denials

<details><summary>Show answer</summary>

**C.** Return errors with a category and an isRetryable flag

Structured metadata lets the agent retry transient errors and stop on business or permission errors.

</details>

**Q22** · Task 2.2 · Customer Support

When the order database times out, lookup_order returns an empty list, and the agent tells customers they have no orders. What should change?

- **A.** Instruct the agent to treat every empty result as suspicious
- **B.** Have lookup_order retry automatically until it returns a non-empty list, then pass that list to the agent as normal
- **C.** Escalate to a human whenever any lookup returns zero results
- **D.** Return it as a transient error, not as an empty result

<details><summary>Show answer</summary>

**D.** Return it as a transient error, not as an empty result

Access failures and valid empty results need different handling. Returning empty-as-success hides the failure.

</details>

**Q23** · Task 2.3 · Developer Productivity

A single agent has 18 tools and frequently picks the wrong one. What is the best architectural change?

- **A.** Rewrite all 18 tool descriptions to be longer and more detailed so the agent can tell them apart
- **B.** Add a few-shot example for each of the 18 tools
- **C.** Move to a model with a larger context window
- **D.** Split the work across specialized subagents, each with only the 4-5 tools its role needs

<details><summary>Show answer</summary>

**D.** Split the work across specialized subagents, each with only the 4-5 tools its role needs

Too many tools degrades selection. Scoped, role-specific tool sets improve reliability and prevent cross-role misuse.

</details>

**Q24** · Task 2.3 · Structured Data Extraction

Your pipeline must always run extract_metadata first, before any other tool. How do you guarantee it?

- **A.** Set tool_choice to "any" on the first request
- **B.** List extract_metadata first in the tools array
- **C.** Force tool_choice {"type":"tool","name":"extract_metadata"}
- **D.** Set tool_choice to "auto" and describe the order in the system prompt

<details><summary>Show answer</summary>

**C.** Force tool_choice {"type":"tool","name":"extract_metadata"}

Forced tool selection guarantees a specific tool runs. "any" only guarantees some tool; "auto" may return text.

</details>

**Q25** · Task 2.3 · Structured Data Extraction

Documents are invoices, receipts, or contracts, each with its own extraction tool and schema. You must always get structured output, never free text. Which tool_choice fits?

- **A.** "none", then parse JSON out of the text response
- **B.** "auto", so Claude can decide for itself whether a tool is needed
- **C.** Forced selection of the invoice tool, with a fallback in code
- **D.** "any", which forces a call to one of the extraction tools

<details><summary>Show answer</summary>

**D.** "any", which forces a call to one of the extraction tools

"any" guarantees a tool call while letting Claude choose among several schemas.

</details>

**Q26** · Task 2.3 · Multi-Agent Research

The synthesis agent needs to verify simple facts during writing. Today each check round-trips through the coordinator to the search agent, adding 40% latency. Most checks are simple lookups. What is the best change?

- **A.** Have the search agent pre-fetch every fact it predicts synthesis will need
- **B.** Give the synthesis agent the full web-search tool set so it can verify any fact itself without waiting
- **C.** Collect all verification requests and run them after the draft is finished
- **D.** A scoped verify_fact tool; complex checks via the coordinator

<details><summary>Show answer</summary>

**D.** A scoped verify_fact tool; complex checks via the coordinator

A scoped cross-role tool covers the common case without handing synthesis tools outside its specialty.

</details>

**Q27** · Task 2.4 · Developer Productivity

Your team wants every developer to get the same GitHub MCP server in Claude Code without committing the access token. What should you do?

- **A.** Put the token directly in .mcp.json since the repo is private
- **B.** Add the server to the project's .mcp.json and reference the token as ${GITHUB_TOKEN}
- **C.** Describe the server setup in CLAUDE.md so Claude configures it
- **D.** Have each developer add the server, with their own token, to their personal ~/.claude.json

<details><summary>Show answer</summary>

**B.** Add the server to the project's .mcp.json and reference the token as ${GITHUB_TOKEN}

Project-scoped .mcp.json is shared via version control; environment variable expansion keeps secrets out of git.

</details>

**Q28** · Task 2.4 · Developer Productivity

A developer wants to try an experimental MCP server without affecting teammates. Where should it be configured?

- **A.** In the project's root CLAUDE.md
- **B.** In their user-level ~/.claude.json
- **C.** In the project's .mcp.json on a personal branch
- **D.** In .claude/commands/ as a slash command

<details><summary>Show answer</summary>

**B.** In their user-level ~/.claude.json

User-scoped configuration is personal and not shared through the repository.

</details>

**Q29** · Task 2.4 · Developer Productivity

The agent makes dozens of exploratory tool calls just to discover which database tables and columns exist. What is the best improvement?

- **A.** Raise the agent's tool-call limit so exploration finishes
- **B.** Paste the full schema into the system prompt and update it by hand
- **C.** Build a separate custom tool for each table
- **D.** Expose the database schema as an MCP resource the agent can read directly

<details><summary>Show answer</summary>

**D.** Expose the database schema as an MCP resource the agent can read directly

MCP resources act as content catalogs, giving visibility without exploratory calls.

</details>

**Q30** · Task 2.4 · Developer Productivity

You added an MCP code_search tool backed by a semantic index, but the agent keeps using the built-in Grep instead. What should you do first?

- **A.** Force tool_choice to code_search on every request
- **B.** Remove Grep from the agent's available tools
- **C.** Improve code_search's description versus Grep
- **D.** Add 'never use built-in tools such as Grep when an MCP tool exists' to the system prompt

<details><summary>Show answer</summary>

**C.** Improve code_search's description versus Grep

Selection follows descriptions. Make the more capable tool's advantages explicit before removing options.

</details>

**Q31** · Task 2.5 · Developer Productivity

You need a list of every file matching *.config.ts anywhere in the repository. Which built-in tool fits?

- **A.** Edit
- **B.** Read
- **C.** Glob
- **D.** Grep

<details><summary>Show answer</summary>

**C.** Glob

Glob matches file paths and names. Grep searches file contents.

</details>

**Q32** · Task 2.5 · Developer Productivity

You need every place where the function applyDiscount is called. Which built-in tool fits?

- **A.** Write
- **B.** Read
- **C.** Grep
- **D.** Glob

<details><summary>Show answer</summary>

**C.** Grep

Grep searches inside files for content such as function names, error messages, and imports.

</details>

**Q33** · Task 2.5 · Code Generation

An Edit call fails because its anchor text appears several times in the file. What is the recommended fallback?

- **A.** Run a global sed replacement through Bash to change every occurrence
- **B.** Read the whole file, then Write the full updated content
- **C.** Retry the same Edit until it succeeds
- **D.** Use Glob to find a different file to edit

<details><summary>Show answer</summary>

**B.** Read the whole file, then Write the full updated content

Edit needs a unique match. When none exists, Read plus Write is the documented fallback; a global replace may change the wrong occurrences.

</details>

**Q99** · Task 2.x Schema design · Customer Support

A refund tool's schema defines "reason": {"type": "string"}. Downstream reports break because Claude sends values like "item broke", "Damaged!" and "arrived damaged". What is the best fix?

- **A.** Change reason to an enum of allowed values, plus "other" and a reason_detail field
- **B.** Add a sentence to the system prompt asking Claude to use consistent reason wording
- **C.** Normalize the reason text in the reporting job after each refund is stored
- **D.** Remove the reason field from the schema so reports no longer depend on it

<details><summary>Show answer</summary>

**A.** Change reason to an enum of allowed values, plus "other" and a reason_detail field

Constrain inputs in the schema instead of accepting free text and fixing it afterwards. An enum gives downstream systems a fixed set of values; "other" plus a detail field keeps it extensible.

</details>

**Q100** · Task 2.x Schema design · Developer Productivity

A search tool has an optional "limit" parameter with no description. Sometimes Claude omits it and gets 10,000 results; sometimes it sets 5 and misses matches. What is the most effective change?

- **A.** Make limit required so Claude always has to choose a value for it
- **B.** State the default and range in the schema: "Defaults to 20 if omitted, maximum 100"
- **C.** Remove limit and always return every matching result to Claude
- **D.** Tell Claude in the system prompt to always pass limit=50 when searching

<details><summary>Show answer</summary>

**B.** State the default and range in the schema: "Defaults to 20 if omitted, maximum 100"

Every optional field needs a sensible, stated default. Making it required pushes Claude to guess; returning everything floods context; a prompt rule is weaker than documenting the behaviour in the schema itself.

</details>

**Q101** · Task 2.x Error handling · Customer Support

process_refund times out. The agent retries, and the customer is refunded twice because the first call had actually completed. What design change prevents this?

- **A.** Stop retrying any tool call that returns a timeout error
- **B.** Increase the tool's timeout so the first call always returns a response
- **C.** Accept an idempotency key so a repeated call with the same key returns the original result
- **D.** Ask Claude to confirm with the customer before every retry it attempts

<details><summary>Show answer</summary>

**C.** Accept an idempotency key so a repeated call with the same key returns the original result

A timeout doesn't mean the action failed. Idempotent write tools make retries safe: the server recognises the repeated key and returns the first result instead of refunding again.

</details>

**Q102** · Task 2.x Error handling · Structured Data Extraction

A bulk_update tool processes 10 records. If any one record fails validation, it returns only "Operation failed" and the agent re-submits all 10. What should the tool return instead?

- **A.** A generic failure, but with a longer timeout so fewer calls fail
- **B.** Success for the whole batch, logging the failed record separately
- **C.** An exception that stops the agent until a human reviews the batch
- **D.** The processed records plus a failed list with each record's error category and message

<details><summary>Show answer</summary>

**D.** The processed records plus a failed list with each record's error category and message

Report partial success: what worked and what failed, each failure categorised. The agent then fixes or escalates only the failed record instead of redoing everything or hiding the failure.

</details>

**Q103** · Task 2.x MCP primitives · Developer Productivity

Your team wants a reusable "summarize this incident" template that engineers pick from a menu when they choose to, not something Claude calls on its own. Which MCP primitive fits?

- **A.** An MCP prompt, since prompts are user-controlled reusable templates
- **B.** An MCP tool, since tools are what Claude can call to take action
- **C.** An MCP resource, since resources hold readable data for the application
- **D.** A new MCP server dedicated to incident summaries and nothing else

<details><summary>Show answer</summary>

**A.** An MCP prompt, since prompts are user-controlled reusable templates

Pick the primitive by who starts it: tools are model-controlled, resources are application-controlled, prompts are user-controlled templates (often shown as slash commands).

</details>

**Q104** · Task 2.x MCP transports · Developer Productivity

A developer wants an MCP server that reads files from a folder on their own laptop. Which transport fits best?

- **A.** A remote HTTP server deployed in the company cloud
- **B.** stdio, running the server as a local process on the laptop
- **C.** Server-Sent Events from a shared server that mounts the laptop's folder
- **D.** No MCP server; paste the files into the system prompt instead

<details><summary>Show answer</summary>

**B.** stdio, running the server as a local process on the laptop

stdio runs the server as a local process and suits local tools and files. Remote transports (HTTP, or SSE in older material) are for shared network services.

</details>

**Q105** · Task 2.x Least privilege · Customer Support

The system prompt says "never delete customer records." After a crafted support message, the agent calls delete_customer and it succeeds. What is the right fix?

- **A.** Repeat the rule at the start and end of the system prompt in capitals
- **B.** Add few-shot examples showing the agent refusing to delete records
- **C.** Remove delete permission from the agent's tools and credential, so the host blocks it
- **D.** Add a classifier that scores each message for deletion intent first

<details><summary>Show answer</summary>

**C.** Remove delete permission from the agent's tools and credential, so the host blocks it

The host and server enforce the security boundary, not the model. Least privilege means the agent's tools and token simply can't delete, so even a manipulated request fails.

</details>

**Q106** · Task 2.x Advanced MCP · Developer Productivity

Both a GitHub server and a Jira server expose a tool named search, and Claude often searches the wrong system. Claude Code shows them as mcp__github__search and mcp__jira__search. What is the best fix?

- **A.** Rename both servers so their tool names are shorter and easier to read
- **B.** Disconnect one of the two servers so that only one search tool remains
- **C.** Add a system-prompt rule that GitHub must always be searched first
- **D.** Give each search tool a description that says what it searches and when to use it

<details><summary>Show answer</summary>

**D.** Give each search tool a description that says what it searches and when to use it

The server prefix already prevents a name clash; wrong selection is a description problem. Say what each searches (code and PRs vs tickets and comments) and when to use it.

</details>

## Domain 3: Claude Code Configuration & Workflows

**Q34** · Task 3.1 · Code Generation

A senior developer's Claude Code follows the team's API conventions perfectly; a new hire's does not. Both use the same repo. What is the most likely cause?

- **A.** The conventions should be in .mcp.json rather than CLAUDE.md
- **B.** The new hire has not run /compact since cloning
- **C.** The conventions sit in the senior dev's user-level CLAUDE.md
- **D.** The new hire is running an older Claude Code version that doesn't read the project's CLAUDE.md file

<details><summary>Show answer</summary>

**C.** The conventions sit in the senior dev's user-level CLAUDE.md

User-level CLAUDE.md applies only to that user and is not shared via version control. Team standards belong at project level.

</details>

**Q35** · Task 3.1 · Code Generation

In a monorepo, each package needs a different subset of shared standards files (security, logging, API style). How do you keep each package's CLAUDE.md small and in sync?

- **A.** Copy the relevant standards text into each package's own CLAUDE.md file
- **B.** @import only the needed shared files in each package
- **C.** Combine all standards into one root CLAUDE.md for every package
- **D.** Put the standards in each developer's ~/.claude/CLAUDE.md

<details><summary>Show answer</summary>

**B.** @import only the needed shared files in each package

@import keeps CLAUDE.md modular and lets each package include exactly the standards it needs from one source.

</details>

**Q36** · Task 3.1 · Code Generation

Claude Code behaves inconsistently and you suspect it is not loading the instruction file you expect. What do you run to check?

- **A.** --output-format json
- **B.** /memory
- **C.** --resume
- **D.** /compact

<details><summary>Show answer</summary>

**B.** /memory

/memory shows which memory (CLAUDE.md) files are loaded.

</details>

**Q37** · Task 3.2 · Code Generation

You want a /deploy-check command available to every developer who clones the repository. Where do you put it?

- **A.** .claude/commands/deploy-check.md in the repository
- **B.** ~/.claude/commands/deploy-check.md on each developer's machine
- **C.** A section of the project's CLAUDE.md
- **D.** A commands array in .claude/config.json

<details><summary>Show answer</summary>

**A.** .claude/commands/deploy-check.md in the repository

Project-scoped commands in .claude/commands/ are shared through version control. ~/.claude/commands/ is personal.

</details>

**Q38** · Task 3.2 · Developer Productivity

A codebase-survey skill produces pages of exploratory output that clutter the main conversation. Which SKILL.md frontmatter setting fixes this?

- **A.** allowed-tools
- **B.** argument-hint
- **C.** context: fork
- **D.** paths

<details><summary>Show answer</summary>

**C.** context: fork

context: fork runs the skill in an isolated sub-agent, so only its result returns to the main conversation.

</details>

**Q39** · Task 3.2 · Developer Productivity

Developers often invoke a /triage skill without the ticket ID it needs. Which frontmatter field helps?

- **A.** description
- **B.** context: fork
- **C.** allowed-tools
- **D.** argument-hint

<details><summary>Show answer</summary>

**D.** argument-hint

argument-hint prompts for required parameters when the skill is invoked without them.

</details>

**Q40** · Task 3.2 · Developer Productivity

A skill should only read files and must never run shell commands. Which frontmatter field enforces this?

- **A.** allowed-tools
- **B.** argument-hint
- **C.** paths
- **D.** context: fork

<details><summary>Show answer</summary>

**A.** allowed-tools

allowed-tools restricts which tools the skill may use during execution.

</details>

**Q41** · Task 3.2 · Code Generation

A developer wants a personal variation of the team's /pr-summary skill without affecting teammates. What should they do?

- **A.** Add their variation as extra instructions in the project CLAUDE.md
- **B.** Edit the project skill locally and avoid committing the change
- **C.** Copy it into ~/.claude/skills/ under a different name and customize it there
- **D.** Ask the team to add a personal-mode flag to the shared skill

<details><summary>Show answer</summary>

**C.** Copy it into ~/.claude/skills/ under a different name and customize it there

Personal customization lives in ~/.claude/skills/ with a distinct name so the shared skill is untouched.

</details>

**Q42** · Task 3.2 · Code Generation

Which of these belongs in a skill rather than in CLAUDE.md?

- **A.** 'Name React components in PascalCase'
- **B.** 'Run the test suite before every commit'
- **C.** 'Use TypeScript strict mode in every file in the repository'
- **D.** A 12-step release-notes procedure run once per sprint

<details><summary>Show answer</summary>

**D.** A 12-step release-notes procedure run once per sprint

CLAUDE.md is always loaded, so it holds universal standards. Occasional, task-specific procedures belong in on-demand skills.

</details>

**Q43** · Task 3.3 · Code Generation

Terraform files live in many folders across the repo and need special conventions. What is the most efficient configuration?

- **A.** A /terraform skill developers invoke before editing
- **B.** A 'when editing Terraform' section in the root CLAUDE.md
- **C.** A CLAUDE.md file inside every folder that contains Terraform
- **D.** .claude/rules/terraform.md with paths: ["**/*.tf"]

<details><summary>Show answer</summary>

**D.** .claude/rules/terraform.md with paths: ["**/*.tf"]

Path-scoped rules load automatically only for matching files, wherever they are. Directory CLAUDE.md files follow folders, not file types.

</details>

**Q44** · Task 3.3 · Code Generation

Why are path-scoped .claude/rules/ files preferred over putting file-type conventions in the root CLAUDE.md?

- **A.** They override user-level CLAUDE.md files for every user
- **B.** They let Claude run tools that CLAUDE.md cannot authorize
- **C.** They load only when matching files are being edited, saving context and avoiding reliance on inference
- **D.** They are the only instruction files Claude Code reads when it runs non-interactively in CI pipelines with -p

<details><summary>Show answer</summary>

**C.** They load only when matching files are being edited, saving context and avoiding reliance on inference

Conditional loading reduces irrelevant context and applies conventions deterministically by file pattern.

</details>

**Q45** · Task 3.4 · Code Generation

You must migrate 60 files from v2 to v3 of a library with breaking API changes and several possible migration orders. How should you start?

- **A.** Run the migration in CI with -p and review the result
- **B.** Plan mode, then implement the agreed plan
- **C.** Direct execution, starting with the file that has the most errors
- **D.** Direct execution on all 60 files in one request

<details><summary>Show answer</summary>

**B.** Plan mode, then implement the agreed plan

Large multi-file changes with design choices call for plan mode first, then direct execution of the agreed plan.

</details>

**Q46** · Task 3.4 · Code Generation

You need to add a missing null check in one function in one file. Which approach fits?

- **A.** Plan mode with a written migration plan
- **B.** An Explore subagent to map the codebase first
- **C.** fork_session to compare two null-check styles
- **D.** Direct execution

<details><summary>Show answer</summary>

**D.** Direct execution

Simple, well-scoped changes do not need planning overhead.

</details>

**Q47** · Task 3.5 · Code Generation

Despite detailed prose instructions, Claude keeps transforming address records inconsistently. What is the most effective next step?

- **A.** Rewrite the instructions in capital letters with more detail
- **B.** Lower max_tokens so the output stays shorter
- **C.** Switch to plan mode before each transformation
- **D.** Give 2-3 concrete input/output examples

<details><summary>Show answer</summary>

**D.** Give 2-3 concrete input/output examples

Concrete I/O examples communicate a transformation more reliably than prose that is being read inconsistently.

</details>

**Q48** · Task 3.5 · Code Generation

You found two bugs in the same checkout flow: a cache invalidation bug and a race condition that triggers it. How should you ask Claude to fix them?

- **A.** Describe both in one message, because the fixes interact
- **B.** Open two separate sessions, one per bug
- **C.** Fix the race condition and ignore the cache bug, which is a symptom
- **D.** Fix the cache bug first, then raise the race condition in a later message

<details><summary>Show answer</summary>

**A.** Describe both in one message, because the fixes interact

Interacting problems should be addressed together. Independent problems are better handled one at a time.

</details>

**Q49** · Task 3.6 · Claude Code for CI/CD

Your CI job runs Claude Code to review PRs, but the job hangs until it times out. What is the fix?

- **A.** Run Claude Code with the -p (--print) flag for non-interactive mode
- **B.** Pipe 'yes' into Claude Code's standard input to answer its prompts
- **C.** Raise the CI job's timeout to two hours
- **D.** Add --resume so it reuses the previous review session

<details><summary>Show answer</summary>

**A.** Run Claude Code with the -p (--print) flag for non-interactive mode

-p processes the prompt, prints the result, and exits, so it never waits for interactive input.

</details>

**Q50** · Task 3.6 · Claude Code for CI/CD

Your pipeline needs to post review findings as PR comments automatically. How do you get reliably parseable output?

- **A.** Write findings to CLAUDE.md and read the file afterwards
- **B.** Use --output-format json with --json-schema defining the findings structure
- **C.** Ask for JSON in the prompt and extract it from the text output with a regex
- **D.** Ask for a markdown table and split it on pipe characters

<details><summary>Show answer</summary>

**B.** Use --output-format json with --json-schema defining the findings structure

Structured CLI output with a schema gives machine-parseable findings.

</details>

**Q51** · Task 3.6 · Claude Code for CI/CD

After new commits, the review re-runs and re-posts comments that were already made. What is the best fix?

- **A.** Delete all previous review comments on the PR before each new review run starts
- **B.** Use the Batch API so reviews run once per day
- **C.** Review only the newest commit and ignore the rest of the PR
- **D.** Include prior findings; ask only for new or unresolved issues

<details><summary>Show answer</summary>

**D.** Include prior findings; ask only for new or unresolved issues

Supplying prior findings prevents duplicate comments while still catching new problems.

</details>

**Q52** · Task 3.6 · Claude Code for CI/CD

The same Claude Code session that generated a feature is then asked to review it, and it misses obvious bugs. What is the best change?

- **A.** Ask the generating session to review its work a second time using a stricter checklist
- **B.** Run the review in a separate, independent session without the generator's context
- **C.** Raise the temperature during the review step
- **D.** Add 'be very critical' to the review prompt

<details><summary>Show answer</summary>

**B.** Run the review in a separate, independent session without the generator's context

A session that wrote the code carries its own reasoning and is less likely to question it. An independent instance catches more.

</details>

**Q88** · Task 3.x Hooks · Code Generation

A PreToolUse hook script detects a forbidden command, writes a reason to stderr and exits with code 1. The command still runs. Why?

- **A.** Exit code 1 is a non-blocking error; only exit code 2 blocks the call
- **B.** PreToolUse hooks can only log calls, never block them
- **C.** The reason must be written to stdout, not stderr, for the block to apply
- **D.** Hooks only run in interactive sessions, not when Claude Code is run with -p

<details><summary>Show answer</summary>

**A.** Exit code 1 is a non-blocking error; only exit code 2 blocks the call

In Claude Code hooks, exit 0 means proceed, exit 2 means block (stderr is sent to Claude), and any other non-zero code is a non-blocking error: the action still proceeds.

</details>

**Q89** · Task 3.x Hooks · Code Generation

The team wants every file Claude edits to be formatted with Prettier, every time, with no exceptions. What is the best approach?

- **A.** Add 'always run Prettier after editing a file' to the project CLAUDE.md
- **B.** A PostToolUse hook with matcher Edit|Write that runs Prettier on the changed file
- **C.** A PreToolUse hook on Bash that runs Prettier before each shell command
- **D.** A /format slash command that developers run before committing

<details><summary>Show answer</summary>

**B.** A PostToolUse hook with matcher Edit|Write that runs Prettier on the changed file

'Every time' means deterministic: a hook. PostToolUse fires after the edit, and the Edit|Write matcher limits it to file changes. CLAUDE.md is probabilistic; a slash command depends on people remembering.

</details>

**Q90** · Task 3.x Permissions · Developer Productivity

Claude Code must never read the .env file in this repository, for every developer on the team. Where should you configure this?

- **A.** A line in the project CLAUDE.md telling Claude never to open .env
- **B.** A deny rule in ~/.claude/settings.json on your own machine
- **C.** A deny rule such as Read(./.env) in the project's .claude/settings.json
- **D.** An entry for .env in the repository's .gitignore file

<details><summary>Show answer</summary>

**C.** A deny rule such as Read(./.env) in the project's .claude/settings.json

A deny permission rule is enforced, not just suggested, and .claude/settings.json is committed so it applies to the whole team. User settings cover only you; .gitignore controls git, not Claude's file access.

</details>

**Q91** · Task 3.x Permissions · Developer Productivity

A developer wants Claude Code to run `npm run storybook` without asking, but only for themselves in this project and without changing teammates' settings. Where does the allow rule go?

- **A.** .claude/settings.json, committed to the repository
- **B.** The project CLAUDE.md, under a 'permissions' heading
- **C.** ~/.claude/settings.json, which applies to all their projects
- **D.** .claude/settings.local.json in this project

<details><summary>Show answer</summary>

**D.** .claude/settings.local.json in this project

.claude/settings.local.json is project-specific and personal (git-ignored). The shared settings.json would affect teammates; the user file would apply to every project.

</details>

**Q92** · Task 3.x Subagents · Code Generation

You want a reusable 'security-reviewer' subagent with read-only tools that everyone on the team can use. What do you create?

- **A.** A Markdown file in .claude/agents/ with a name, a description, and tools limited to Read, Grep and Glob
- **B.** A 'Security reviews' section in the project CLAUDE.md explaining the review steps
- **C.** A file in ~/.claude/agents/ on your own machine with the same settings
- **D.** An MCP server that exposes a security_review tool to Claude Code

<details><summary>Show answer</summary>

**A.** A Markdown file in .claude/agents/ with a name, a description, and tools limited to Read, Grep and Glob

Team subagents are Markdown files in the repo's .claude/agents/ folder, with their own prompt, tools and context window. ~/.claude/agents/ is personal; CLAUDE.md is always-loaded instructions, not a specialist.

</details>

**Q93** · Task 3.x Subagents · Claude Code for CI/CD

A test-writing subagent cannot find a fixture it needs, so it invents a plausible one and the tests pass against fake data. What is the best fix?

- **A.** Give the subagent write access to the shared fixtures folder
- **B.** Require a structured result with a blockers field, and instruct it to report missing inputs instead of guessing
- **C.** Raise the subagent's maximum turns so it searches for longer
- **D.** Stop using a subagent and write the tests in the main conversation

<details><summary>Show answer</summary>

**B.** Require a structured result with a blockers field, and instruct it to report missing inputs instead of guessing

Structured output with obstacle reporting lets the subagent hand a blocker back to the main agent cleanly. The root problem is guessing when stuck, not search time or access.

</details>

**Q94** · Task 3.x Skills · Code Generation

A /deploy skill pushes to production. It should run only when a person types /deploy, never because Claude decides it is relevant. Which frontmatter setting does this?

- **A.** context: fork
- **B.** argument-hint: [environment]
- **C.** disable-model-invocation: true
- **D.** user-invocable: false

<details><summary>Show answer</summary>

**C.** disable-model-invocation: true

disable-model-invocation: true makes a skill manual-only. user-invocable: false is the opposite: it hides the skill from the / menu while Claude can still load it.

</details>

**Q95** · Task 3.x Architecture · Code Generation

A developer explained the team's branch-naming convention to Claude yesterday. Today, in a new session, Claude ignores it. What is the right fix?

- **A.** Switch to a model with a larger context window
- **B.** Ask Claude at the start of every session to remember the convention
- **C.** Run /compact before closing each session so the convention is kept
- **D.** Add the convention to the project CLAUDE.md, which loads in every new session

<details><summary>Show answer</summary>

**D.** Add the convention to the project CLAUDE.md, which loads in every new session

Context lasts only within a session. A new session is re-established from CLAUDE.md, so team conventions that must persist belong there.

</details>

**Q96** · Task 3.x CI/CD · Claude Code for CI/CD

In a CI job, Claude Code should only read files and run the test suite, without any permission prompts. What is the best configuration?

- **A.** Run it with --dangerously-skip-permissions so nothing is blocked
- **B.** Run with -p and --allowedTools limited to Read, Grep, Glob and the test command
- **C.** Run it interactively and have an engineer approve each step
- **D.** Add 'only read files and run tests' to the project CLAUDE.md

<details><summary>Show answer</summary>

**B.** Run with -p and --allowedTools limited to Read, Grep, Glob and the test command

-p makes the run non-interactive, and --allowedTools grants exactly what the job needs. Skipping permissions removes every guard; a CLAUDE.md line is only guidance.

</details>

## Domain 4: Prompt Engineering & Structured Output

**Q53** · Task 4.1 · Claude Code for CI/CD

You added 'only report high-confidence issues' to the review prompt, but false positives barely dropped. What will work better?

- **A.** Add 'be conservative' at both the start and end of the prompt
- **B.** Change it to 'only report issues you are 95% confident about'
- **C.** Ask for a confidence score on every finding and automatically hide any finding scored below 0.9
- **D.** List exactly which issue types to report and which to skip

<details><summary>Show answer</summary>

**D.** List exactly which issue types to report and which to skip

Explicit categorical criteria change behavior; vague confidence instructions do not.

</details>

**Q54** · Task 4.1 · Claude Code for CI/CD

The 'naming style' category produces mostly false positives, and developers have started ignoring all findings, including real bugs. What is the best immediate step?

- **A.** Raise the review's temperature to diversify findings
- **B.** Add more naming examples so the category posts more findings
- **C.** Temporarily disable the naming-style category to restore trust while you refine it
- **D.** Keep every category but relabel naming-style findings as 'low priority' in the output

<details><summary>Show answer</summary>

**C.** Temporarily disable the naming-style category to restore trust while you refine it

High false-positive rates in one category undermine confidence in every category. Removing the noisy one restores trust.

</details>

**Q55** · Task 4.1 · Claude Code for CI/CD

Severity labels on findings are inconsistent between runs. What is the best fix?

- **A.** Ask Claude to keep severity labels consistent across every review run
- **B.** Use a different model for severity assignment
- **C.** Remove severity and list findings in file order
- **D.** Define each severity level with a concrete code example

<details><summary>Show answer</summary>

**D.** Define each severity level with a concrete code example

Explicit criteria with concrete examples make severity calibration consistent.

</details>

**Q56** · Task 4.2 · Structured Data Extraction

Field notes contain informal measurements such as 'about two and a half inches' and '3ft-ish'. Extraction often returns null or invented numbers. What is the most effective fix?

- **A.** Add few-shot examples showing correct extraction from varied informal formats
- **B.** Tell Claude to never return null for measurements
- **C.** Increase max_tokens so Claude can reason longer
- **D.** Make every measurement field required in the schema so the model can't leave values empty

<details><summary>Show answer</summary>

**A.** Add few-shot examples showing correct extraction from varied informal formats

Few-shot examples of varied formats reduce both empty fields and hallucination. Required fields push the model to fabricate.

</details>

**Q57** · Task 4.2 · Claude Code for CI/CD

You want few-shot examples to improve how the reviewer handles ambiguous code patterns. What kind of examples work best?

- **A.** 2-4 ambiguous-case examples that show the reasoning
- **B.** Examples that show only correctly flagged issues
- **C.** 20 or more examples of obvious bugs, so every common pattern appears at least once
- **D.** One long example that reviews an entire repository

<details><summary>Show answer</summary>

**A.** 2-4 ambiguous-case examples that show the reasoning

A few targeted examples of the hard cases, with reasoning, generalize best.

</details>

**Q58** · Task 4.3 · Structured Data Extraction

About 3% of extraction responses fail JSON parsing (missing braces, trailing commas). What is the most reliable fix?

- **A.** Add 'return valid JSON only, with no extra text' to the system prompt and validate the response afterwards
- **B.** Define an extraction tool whose input_schema is your JSON schema and read the tool_use input
- **C.** Retry any response that fails to parse until it succeeds
- **D.** Post-process responses with a JSON repair library

<details><summary>Show answer</summary>

**B.** Define an extraction tool whose input_schema is your JSON schema and read the tool_use input

Tool use with a JSON schema is the most reliable way to get schema-compliant structured output.

</details>

**Q59** · Task 4.3 · Structured Data Extraction

Extraction via tool use with a strict schema now always parses, but some invoices have line items that do not add up to the stated total. Why?

- **A.** Schemas stop syntax errors, not semantic ones; validate values
- **B.** Tool use only validates types for the first level of the JSON
- **C.** The schema isn't strict enough; adding more required fields and tighter types will catch these mismatches
- **D.** tool_choice should be "any" instead of a forced tool

<details><summary>Show answer</summary>

**A.** Schemas stop syntax errors, not semantic ones; validate values

Structure is guaranteed; correctness of values is not. Add semantic validation and retry with the specific error.

</details>

**Q60** · Task 4.3 · Structured Data Extraction

Many source documents have no PO number, but the extraction returns plausible-looking PO numbers anyway. What schema change helps most?

- **A.** Lower the temperature to zero
- **B.** Make po_number optional/nullable and instruct Claude to return null when it is absent
- **C.** Add a second tool that double-checks PO numbers
- **D.** Make po_number required with a strict regex pattern so only correctly formatted numbers are accepted

<details><summary>Show answer</summary>

**B.** Make po_number optional/nullable and instruct Claude to return null when it is absent

Required fields pressure the model to fabricate. Nullable fields let it report absence honestly.

</details>

**Q61** · Task 4.3 · Structured Data Extraction

Your document_type enum keeps failing on new document kinds the business has started receiving. What schema design fixes this?

- **A.** Add every document type the business can think of to the enum, and update the list whenever a new kind appears
- **B.** Reject documents whose type is not in the enum
- **C.** Add an "other" enum value plus a free-text detail field (and "unclear" for ambiguous cases)
- **D.** Remove the enum and accept any string

<details><summary>Show answer</summary>

**C.** Add an "other" enum value plus a free-text detail field (and "unclear" for ambiguous cases)

"other" + detail keeps categories extensible without losing structure; "unclear" handles ambiguity.

</details>

**Q62** · Task 4.4 · Structured Data Extraction

Validation flags two problems: (1) a date written as '04.03.25' that should be ISO 8601, and (2) a missing signature date that is not in the document. Which will a retry with error feedback most likely fix?

- **A.** Neither, because retries never improve extraction
- **B.** The date format problem only
- **C.** Both, if the retry includes the original document
- **D.** The missing signature date, because Claude can infer it from context

<details><summary>Show answer</summary>

**B.** The date format problem only

Retries fix format and structural errors. When information is absent, retrying invites fabrication; return null and route to review.

</details>

**Q63** · Task 4.4 · Structured Data Extraction

What should a retry request contain after a semantic validation failure?

- **A.** The failed extraction plus a general instruction to 'please review and fix any errors' before returning it
- **B.** The original document, the failed extraction, and the specific validation errors found
- **C.** The original prompt with temperature raised
- **D.** Only the original document, so Claude starts fresh

<details><summary>Show answer</summary>

**B.** The original document, the failed extraction, and the specific validation errors found

Retry-with-error-feedback works because Claude sees exactly what was wrong and why.

</details>

**Q64** · Task 4.5 · Claude Code for CI/CD

To cut costs, a manager proposes moving both pre-merge PR checks and the nightly tech-debt report to the Message Batches API. What do you recommend?

- **A.** Batch neither, because Batch results are less accurate
- **B.** Batch both, since 50% savings outweighs latency
- **C.** Batch the nightly report; keep pre-merge synchronous
- **D.** Batch the pre-merge checks only, since they run more often

<details><summary>Show answer</summary>

**C.** Batch the nightly report; keep pre-merge synchronous

Batch processing can take up to 24 hours with no latency SLA, so it fits only non-blocking work.

</details>

**Q65** · Task 4.5 · Structured Data Extraction

A batch of 10,000 documents finishes with 120 failures. What is the best next step?

- **A.** Resubmit the whole batch so all 10,000 results come from one consistent run
- **B.** Resubmit the 120 failures unchanged until they succeed
- **C.** Process the 120 failures by hand
- **D.** Resubmit only the failed custom_ids after fixing the cause

<details><summary>Show answer</summary>

**D.** Resubmit only the failed custom_ids after fixing the cause

custom_id lets you correlate results and resubmit only what failed, with modifications.

</details>

**Q66** · Task 4.5 · Structured Data Extraction

Results must be delivered within 30 hours of a document arriving. Batches can take up to 24 hours. How often should you submit batches?

- **A.** At least every 6 hours
- **B.** Every 12 hours, morning and evening
- **C.** Once a day, after business hours
- **D.** Once a week, to maximize batch size

<details><summary>Show answer</summary>

**A.** At least every 6 hours

Worst case = waiting for the next submission + 24 hours of processing. Submitting every 6 hours keeps that at or under 30 hours.

</details>

**Q67** · Task 4.5 · Customer Support

Why is the Message Batches API a poor fit for running the customer-support agent's conversations?

- **A.** Batch requests can't do multi-turn tool calling within a request, and there is no latency guarantee
- **B.** Batch requests are limited to 100 messages per day
- **C.** Batch requests do not support JSON schemas
- **D.** Batch requests cannot include a system prompt, so the support agent's policies and escalation rules would be lost

<details><summary>Show answer</summary>

**A.** Batch requests can't do multi-turn tool calling within a request, and there is no latency guarantee

Agentic tool loops need synchronous turns, and a waiting customer needs low latency.

</details>

**Q68** · Task 4.6 · Claude Code for CI/CD

A single-pass review of a 20-file PR gives shallow feedback on some files and contradicts itself across others. What is the best redesign?

- **A.** One pass that reviews the files in alphabetical order
- **B.** One pass with a much larger max_tokens
- **C.** Two identical full-PR passes, keeping findings both report
- **D.** Per-file passes plus one cross-file integration pass

<details><summary>Show answer</summary>

**D.** Per-file passes plus one cross-file integration pass

Splitting avoids attention dilution; the integration pass catches cross-file data-flow issues.

</details>

**Q69** · Task 4.6 · Code Generation

Why does an independent Claude instance catch more bugs than asking the generating instance to review its own code?

- **A.** Independent instances are routed to a more capable model than the one that generated the code
- **B.** The generator keeps its own reasoning in context and is less likely to question its decisions
- **C.** Self-review is blocked by the API for safety reasons
- **D.** The generator's context window is too full to read the code again

<details><summary>Show answer</summary>

**B.** The generator keeps its own reasoning in context and is less likely to question its decisions

Reviewing without the generator's reasoning context removes confirmation bias.

</details>

## Domain 5: Context Management & Reliability

**Q70** · Task 5.1 · Customer Support

In long support chats, earlier summaries reduce 'refund of $1,247.50 for order 88213, promised by Friday' to 'customer wants a refund'. What is the best fix?

- **A.** Summarize the history more often so it stays short and the most recent facts always sit near the end of the prompt
- **B.** Keep a persistent case-facts block (amounts, IDs, dates, promises) in every prompt, outside the summarized history
- **C.** Switch to a model with a larger context window
- **D.** Ask the customer to restate key details every few turns

<details><summary>Show answer</summary>

**B.** Keep a persistent case-facts block (amounts, IDs, dates, promises) in every prompt, outside the summarized history

Exact transactional facts must be preserved verbatim outside progressive summarization.

</details>

**Q71** · Task 5.1 · Customer Support

Each order lookup returns 45 fields, but the agent needs about 5. Context fills quickly in multi-order conversations. What is the best fix?

- **A.** Use /compact after every lookup
- **B.** Trim tool results to the relevant fields before they enter context (e.g., with a PostToolUse hook)
- **C.** Instruct Claude in the system prompt to ignore the fields it doesn't need when it reads each order lookup result
- **D.** Limit customers to one order per conversation

<details><summary>Show answer</summary>

**B.** Trim tool results to the relevant fields before they enter context (e.g., with a PostToolUse hook)

Verbose tool output consumes tokens out of proportion to its value. Trim it deterministically at the source.

</details>

**Q72** · Task 5.1 · Multi-Agent Research

Synthesis receives results from 12 subagents in one long input and keeps omitting findings from the middle subagents. What is the best mitigation?

- **A.** Randomize the order of subagent results on each run
- **B.** Concatenate results with no structure to save tokens
- **C.** Put a key-findings summary at the top and organize detailed results under clear section headers
- **D.** Reduce the number of research subagents from twelve to three so there is less material to combine

<details><summary>Show answer</summary>

**C.** Put a key-findings summary at the top and organize detailed results under clear section headers

Models attend most reliably to the start and end of long input (lost in the middle). Lead with the summary and structure the rest.

</details>

**Q73** · Task 5.2 · Customer Support

A customer with a simple password-reset issue writes: 'I want to talk to a real person.' What should the agent do?

- **A.** Explain it can handle password resets and continue
- **B.** Ask why they prefer a human before deciding
- **C.** Resolve the reset first, since it is quick, then offer escalation
- **D.** Escalate to a human immediately

<details><summary>Show answer</summary>

**D.** Escalate to a human immediately

Explicit requests for a human are honored immediately, without investigating first.

</details>

**Q74** · Task 5.2 · Customer Support

A customer is clearly frustrated about a late delivery the agent can fix, but has not asked for a human. What is the best response pattern?

- **A.** Escalate immediately, because sentiment analysis detected negative emotion in the message
- **B.** Acknowledge it, offer to resolve; escalate if asked again
- **C.** Offer a discount before investigating the issue
- **D.** Ignore the emotion and proceed directly to the fix

<details><summary>Show answer</summary>

**B.** Acknowledge it, offer to resolve; escalate if asked again

Frustration alone is not an escalation trigger when the issue is in scope; a reiterated request for a human is.

</details>

**Q75** · Task 5.2 · Customer Support

A customer asks the agent to match a competitor's lower price. Policy covers price adjustments for your own site only and says nothing about competitors. What should the agent do?

- **A.** Escalate, because policy is silent on this request
- **B.** Approve it, since price adjustments are already allowed
- **C.** Deny it, since competitors are not mentioned in policy
- **D.** Ask the customer for proof, then decide on its own

<details><summary>Show answer</summary>

**A.** Escalate, because policy is silent on this request

Policy gaps or ambiguity are escalation triggers. The agent should not invent policy in either direction.

</details>

**Q76** · Task 5.2 · Customer Support

get_customer returns three customers named 'Maria Lopez'. What should the agent do?

- **A.** Pick the account in the customer's stated city
- **B.** Ask the customer for an additional identifier, such as email or order number
- **C.** Proceed with all three accounts and filter later
- **D.** Pick the account with the most recent order, since that is most likely the person contacting support

<details><summary>Show answer</summary>

**B.** Ask the customer for an additional identifier, such as email or order number

Multiple matches require clarification, not heuristic selection.

</details>

**Q77** · Task 5.2 · Customer Support

The agent resolves only 55% of contacts at first touch (target 80%): it escalates easy cases and tries to handle policy exceptions alone. What is the best fix?

- **A.** Escalate whenever sentiment analysis detects frustration
- **B.** Explicit escalation criteria with few-shot examples
- **C.** Train a separate classifier to decide every escalation
- **D.** Escalate whenever the model's self-reported confidence is below 0.7

<details><summary>Show answer</summary>

**B.** Explicit escalation criteria with few-shot examples

Self-reported confidence and sentiment are poor proxies for complexity; explicit criteria with examples fix the decision itself.

</details>

**Q78** · Task 5.3 · Multi-Agent Research

The web-search subagent times out mid-research. What should it return to the coordinator?

- **A.** An empty result marked successful so the workflow continues
- **B.** Structured context: error type, query, partial results, alternatives
- **C.** An unhandled exception that stops the whole research job
- **D.** A short 'search unavailable' status so the coordinator knows the search step did not complete

<details><summary>Show answer</summary>

**B.** Structured context: error type, query, partial results, alternatives

Structured error context lets the coordinator decide whether to retry, rephrase, or proceed with partial data.

</details>

**Q79** · Task 5.3 · Multi-Agent Research

Two of eight sources were unreachable during research. How should the final report reflect this?

- **A.** Fail the entire report until all eight sources are reachable
- **B.** Fill the gaps with the model's general knowledge so the report reads as complete for stakeholders
- **C.** Leave the report as is, since six sources are enough
- **D.** Include coverage annotations showing which findings are well-supported and which areas have gaps

<details><summary>Show answer</summary>

**D.** Include coverage annotations showing which findings are well-supported and which areas have gaps

Coverage annotations keep the report honest about what is supported and what is missing.

</details>

**Q80** · Task 5.4 · Developer Productivity

Two hours into exploring a legacy codebase, Claude starts referring to 'typical patterns' instead of the specific classes it found earlier. What is the best approach?

- **A.** Raise the temperature so answers become more specific
- **B.** Keep going in the same session; the context window is large enough
- **C.** Restart and re-read the entire codebase from scratch in a fresh session
- **D.** A scratchpad file of findings plus focused subagents

<details><summary>Show answer</summary>

**D.** A scratchpad file of findings plus focused subagents

This is context degradation. Scratchpads persist findings and subagents keep verbose exploration out of the main context.

</details>

**Q81** · Task 5.4 · Multi-Agent Research

Long multi-agent jobs occasionally crash and must restart from zero. What design enables recovery?

- **A.** Per-agent state exports plus a manifest loaded on resume
- **B.** Use --resume on the coordinator's session only
- **C.** Rerun crashed jobs automatically from the beginning
- **D.** Keep every agent's full transcript in the coordinator's conversation history so a resumed coordinator can replay all of it

<details><summary>Show answer</summary>

**A.** Per-agent state exports plus a manifest loaded on resume

Structured state exports plus a manifest let the coordinator restore every agent's progress.

</details>

**Q82** · Task 5.5 · Structured Data Extraction

Extraction is 97% accurate overall, and the team wants to stop human review. What should you do first?

- **A.** Stop human review now, since 97% overall accuracy is above the 95% target
- **B.** Keep reviewing 100% of documents indefinitely
- **C.** Re-run the overall accuracy test on a bigger sample
- **D.** Check accuracy by document type and by field first

<details><summary>Show answer</summary>

**D.** Check accuracy by document type and by field first

Aggregate accuracy can hide weak segments, such as one document type or one field.

</details>

**Q83** · Task 5.5 · Structured Data Extraction

After automating high-confidence extractions, how do you keep measuring their error rate and catch new error patterns?

- **A.** Stratified random sampling of high-confidence outputs for human review
- **B.** Review only the low-confidence outputs, since high-confidence ones are already reliable
- **C.** Have Claude re-score its own confidence weekly
- **D.** Re-check outputs only when a customer complains

<details><summary>Show answer</summary>

**A.** Stratified random sampling of high-confidence outputs for human review

Stratified sampling keeps error measurement honest across segments and surfaces novel failure modes.

</details>

**Q84** · Task 5.5 · Structured Data Extraction

The model outputs a confidence score for each field. How should you use it to route documents to human review?

- **A.** Calibrate review thresholds against a labeled validation set, then route low-confidence fields to reviewers
- **B.** Ask Claude to set its own review threshold
- **C.** Trust the raw scores and auto-approve every field above 0.8, sending only the fields below that number to reviewers
- **D.** Ignore the scores and review a fixed 10% at random

<details><summary>Show answer</summary>

**A.** Calibrate review thresholds against a labeled validation set, then route low-confidence fields to reviewers

Raw model confidence is not calibrated; a labeled validation set turns it into a usable routing signal.

</details>

**Q85** · Task 5.6 · Multi-Agent Research

Two credible sources report 34% and 41% for the same statistic. What should the synthesis do?

- **A.** Report the average, 37.5%
- **B.** Omit the statistic to avoid confusion
- **C.** Report both values, each with its source
- **D.** Report the value from the more recent-looking source

<details><summary>Show answer</summary>

**C.** Report both values, each with its source

Annotate conflicts with attribution rather than silently choosing or blending values.

</details>

**Q86** · Task 5.6 · Multi-Agent Research

Synthesis flags a 'contradiction' between a 2019 and a 2025 figure for market size, which simply grew. What upstream change prevents this?

- **A.** Have synthesis pick the larger figure
- **B.** Tell the synthesis agent to ignore any figure that is more than two years old
- **C.** Require publication or collection dates in subagent output
- **D.** Remove numeric figures from subagent outputs

<details><summary>Show answer</summary>

**C.** Require publication or collection dates in subagent output

Dates let synthesis interpret differences correctly instead of mislabeling change over time as conflict.

</details>

**Q87** · Task 5.6 · Multi-Agent Research

Final reports often state findings with no citation, even though subagents found sources. Where is attribution usually lost, and what is the fix?

- **A.** In the search API; switch to a different search provider
- **B.** In report formatting; add a references section template
- **C.** During summarization; require claim-to-source mappings that every downstream step must preserve
- **D.** In the coordinator; give it the search tools directly so it can re-attach sources itself while writing

<details><summary>Show answer</summary>

**C.** During summarization; require claim-to-source mappings that every downstream step must preserve

Compression drops claim-source links unless they travel as structured data through every step.

</details>


---

← [FAQ](FAQ.md) · [Home](README.md)

Unofficial CCAF study guide · not affiliated with Anthropic · [Interactive site](https://ppotdar.github.io/ccaf-study-course/) · © 2026 Puja Potdar · [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)
