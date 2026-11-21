# Scenario playbook

The exam draws 4 of these 6 scenarios at random, and every question sits inside one of them. Start with what each scenario is and the kinds of questions it asks, then drill the recurring symptoms below. If you can answer the right-hand columns from memory, you are ready for that scenario.

## The six scenarios at a glance

| Scenario | The setup | Main domains | Types of questions |
| --- | --- | --- | --- |
| **1. Customer Support Resolution Agent** | An Agent SDK agent with MCP tools `get_customer`, `lookup_order`, `process_refund` and `escalate_to_human`. Target: 80%+ first-contact resolution. | 1, 2, 5 | Enforcing must-follow policies (identity check, refund limits) with hooks or gates rather than prompts · tool misrouting · when and how to escalate to a human · keeping case facts through long chats · structured tool errors |
| **2. Code Generation with Claude Code** | A team uses Claude Code day to day, with CLAUDE.md files, slash commands and plan mode. | 3, 5 | Which file and scope an instruction belongs in (CLAUDE.md levels, `.claude/rules/` with `paths`) · team commands and skills · plan mode vs direct execution · fixing output with concrete examples or tests · context in long sessions |
| **3. Multi-Agent Research System** | A coordinator delegates to search, document-analysis and synthesis subagents, then assembles a cited report. | 1, 2, 5 | Diagnosing gaps (coordinator decomposition vs a weak subagent) · what context to pass to subagents · which tools each agent gets (e.g. `verify_fact`) · passing errors up to the coordinator · keeping citations and handling conflicting sources |
| **4. Developer Productivity** | Engineers explore and change an unfamiliar legacy codebase with Read, Write, Edit, Bash, Grep, Glob and MCP tools. | 2, 3, 1 | Picking the right built-in tool (Grep vs Glob vs Read/Edit) · exploring incrementally · why the agent ignores a better MCP tool · answers drifting in long explorations (subagents, scratchpad, `/compact`) · resume vs fork vs a fresh session |
| **5. Claude Code for CI/CD** | Claude Code runs unattended in a pipeline: automated code review, test generation and PR comments. | 3, 4 | Non-interactive flags (`-p`, `--output-format json`, `--json-schema`, allowed tools) · cutting noisy or false-positive findings with explicit criteria · duplicate comments on re-runs · independent review vs self-review · Batch API vs real-time for cost |
| **6. Structured Data Extraction** | Unstructured documents (invoices, contracts, forms) are turned into JSON that matches a schema and is validated. | 4, 5 | Forcing the schema with tool use and `tool_choice` · nullable fields vs invented values · validation-and-retry loops and when to stop retrying · few-shot examples for varied formats · measuring accuracy per document type and routing to human review |

## Symptom → answer, by scenario

| Scenario | Symptom in the question | Answer pattern | Task |
| --- | --- | --- | --- |
| 1 Support | Agent skips identity check | [Programmatic prerequisite gate](FAQ-Programmatic-Prerequisites.md) | 1.4 |
| 1 Support | Refunds over the limit slip through | Pre-call hook blocks and redirects to escalation | 1.5 |
| 1 Support | Wrong tool for “check my order” | Expand and differentiate tool descriptions | 2.1 |
| 1 Support | 55% first-contact resolution | Explicit escalation criteria plus few-shot | 5.2 |
| 1 Support | “Get me a human” | Escalate now, with a structured handoff | 5.2, 1.4 |
| 1 Support | [Facts lost in long chats](FAQ-Protecting-Context.md) | Case-facts block, trimmed tool output | 5.1 |
| 2 Code Gen | Teammates don't get instructions | Move from user-level to project CLAUDE.md | 3.1 |
| 2 Code Gen | [Team `/review` command](FAQ-Slash-Commands-and-Skills.md) | `.claude/commands/` | 3.2 |
| 2 Code Gen | Test-file conventions across folders | `.claude/rules/` with `paths` | 3.3 |
| 2 Code Gen | Monolith split or big migration | Plan mode first | 3.4 |
| 2 Code Gen | Transformation keeps coming out wrong | 2–3 concrete I/O examples, or tests first | 3.5 |
| 3 Research | Report misses whole subtopics | [Coordinator decomposition](FAQ-Coordinator-Decomposition.md) too narrow | 1.2 |
| 3 Research | [Synthesis doesn't know what search found](FAQ-Subagent-Context.md) | Pass full findings explicitly, structured | 1.3 |
| 3 Research | Search timeout | Structured error context to the coordinator | 5.3, 2.2 |
| 3 Research | Fact checks add 40% latency | Scoped `verify_fact` tool | 2.3 |
| 3 Research | Citations lost; conflicting stats | Claim-to-source mapping; annotate conflicts with dates | 5.6 |
| 4 Dev Productivity | [Find files vs find code](FAQ-Grep-Glob-Read.md) | Glob vs Grep | 2.5 |
| 4 Dev Productivity | Agent prefers Grep over a better MCP tool | Improve the MCP tool's description | 2.4 |
| 4 Dev Productivity | Long exploration, answers drifting | Subagents, scratchpad, `/compact`, Explore | 5.4, 3.4 |
| 5 CI/CD | Pipeline hangs | `claude -p` | 3.6 |
| 5 CI/CD | Need parseable PR comments | `--output-format json --json-schema` | 3.6 |
| 5 CI/CD | Developers ignore noisy findings | Explicit criteria; disable the noisy category | 4.1 |
| 5 CI/CD | Duplicate comments on re-run | Include prior findings in context | 3.6 |
| 5 CI/CD | [Self-review misses bugs](FAQ-Independent-Review.md) | Independent instance; per-file plus cross-file | 4.6 |
| 5 CI/CD | Cut cost on pre-merge and nightly jobs | Batch only the nightly jobs | 4.5 |
| 6 Extraction | Malformed JSON | Tool use with a JSON schema | 4.3 |
| 6 Extraction | Invented values for missing fields | Nullable fields | 4.3 |
| 6 Extraction | Totals don't add up | Semantic validation, retry with specific errors | 4.4 |
| 6 Extraction | Retry keeps failing on absent data | Stop retrying; null plus human review | 4.4 |
| 6 Extraction | 97% overall, bad on one doc type | Segment accuracy; stratified sampling; calibration | 5.5 |

## Answering technique (2 minutes per question)

1. Read the **last sentence** first: is it asking for the root cause, the best fix, or the most efficient fix?
2. Name the task statement in your head (“this is 1.4 enforcement”).
3. Eliminate the probabilistic option if the rule must always hold, and the over-engineered option if a simpler fix works.
4. Between the last two, pick the one that fixes the **cause** rather than masking the symptom.
5. Flag and move on after 2.5 minutes. Never leave a blank.


---

← [Domain 5 Context Management and Reliability](Domain-5-Context-Management-and-Reliability.md) · [Home](README.md) · [Cheat Sheet](Cheat-Sheet.md) →

Unofficial CCAF study guide · not affiliated with Anthropic · [Interactive site](https://ppotdar.github.io/ccaf-study-course/) · © 2026 Puja Potdar · [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)
