# Scenario playbook

Each scenario has a handful of recurring problems. If you can answer the right-hand columns from memory, you are ready for that scenario.

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
