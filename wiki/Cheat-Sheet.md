# Cheat sheet and teach-back

Reread this the morning of the exam.

## Names to recognize instantly

| Term | Meaning |
| --- | --- |
| [`stop_reason`](FAQ-stop_reason.md) `"tool_use"` / `"end_turn"` | Keep looping / done |
| `tool_result` + `tool_use_id` | How you return a tool's output to Claude |
| `tool_choice` auto / any / tool / none | May / must use some / must use this one / no tools |
| [`Task` tool, `allowedTools: ["Task"]`](FAQ-Spawning-Subagents.md) | How a coordinator spawns subagents |
| `AgentDefinition` | A subagent's description, prompt, tools |
| [`PreToolUse` / `PostToolUse`](FAQ-Hooks.md) | Intercept a call before it runs / a result before Claude sees it |
| [`--resume <name>` / `fork_session`](FAQ-Resume-Fork-Fresh-Start.md) | Continue a session / branch from a baseline |
| `isError`, `errorCategory`, `isRetryable` | Structured MCP error |
| `.mcp.json` / `~/.claude.json` | Team MCP servers / personal MCP servers |
| `${ENV_VAR}` in `.mcp.json` | Secrets without committing them |
| MCP resources | Read-only catalogs the agent can browse |
| `~/.claude/CLAUDE.md` / root `CLAUDE.md` / subfolder | User / project / directory instructions |
| `@import` | Pull another file into CLAUDE.md |
| `.claude/rules/*.md` with `paths:` | Conventions loaded only for matching files |
| [`.claude/commands/` / `~/.claude/commands/`](FAQ-Slash-Commands-and-Skills.md) | Team / personal slash commands |
| [`.claude/skills/X/SKILL.md`](FAQ-Slash-Commands-and-Skills.md) | On-demand skill: `context: fork`, `allowed-tools`, `argument-hint` |
| `/memory` | See which memory files are loaded |
| `/compact` | Shrink a long session's context |
| Plan mode / Explore subagent | Design before editing / isolate noisy discovery |
| `claude -p`, `--output-format json`, `--json-schema` | CI: non-interactive, machine-readable, schema-bound |
| Message Batches API, `custom_id` | 50% cost, up to 24 h, no SLA, no in-request multi-turn tools |
| [Case-facts block](FAQ-Protecting-Context.md) | Exact facts kept outside summarized history |
| Stratified sampling | Keep measuring errors in high-confidence outputs |
| `.claude/settings.json` / `settings.local.json` | Team permissions and hooks / your personal overrides for this project |
| `permissions` `allow` / `ask` / `deny` | Auto-run / prompt first / block, e.g. `deny: Read(./.env)` |
| Hook exit codes | 0 proceed, **2 block**, anything else = non-blocking warning |
| `.claude/agents/<name>.md` | Team subagent with its own prompt, tools and context window |
| `disable-model-invocation: true` | Skill runs only when a person types it |
| Tool description: 3 parts | What it does · when to use it (and not) · what it returns |
| Schema constraints | `enum`, `pattern`, `minimum`/`maximum`, `maxLength`, stated defaults |
| Idempotency key | Makes a retried write safe: no double refunds |
| MCP primitives | Tools (model) · resources (app) · prompts (user) |
| MCP transports | stdio = local · HTTP = remote (SSE = older, deprecated) |
| MCP scopes | Local > project (`.mcp.json`) > user; highest entry wins whole |
| `${VAR:-default}` | Env var with a fallback value in `.mcp.json` |

## Words that signal the answer

| When the question says | Look for |
| --- | --- |
| “must always,” “guarantee,” “skips it X% of the time” | Hook or programmatic gate |
| “picks the wrong tool” | Tool description |
| “every developer who clones” | Project-scoped file in the repo |
| “test files across the codebase” | `.claude/rules/` with a glob |
| “hangs in CI” | `-p` |
| “misses whole topics, subagents worked fine” | [Coordinator decomposition](FAQ-Coordinator-Decomposition.md) |
| “timeout” | Structured error context, not empty results |
| “invents values” / “values don't add up” | Nullable fields / semantic validation |
| “overnight, weekly” / “before merge, customer waiting” | Batch / synchronous |
| “asks for a human” | Escalate now |

## Teach it in 30 minutes

1. **5 min, frame.** Five domains, six scenarios, and one idea: the exam rewards the proportionate, deterministic fix at the root cause.
2. **10 min, draw the agent.** The loop on `stop_reason`, tools wrapped in hooks, a coordinator spawning subagents that start blank, errors flowing back structured. That one picture covers Domains 1, 2 and half of 5.
3. **8 min, Claude Code files.** Whiteboard `~/.claude/` (personal) next to the repo's `.claude/` (team): CLAUDE.md, rules, commands, skills, `.mcp.json`. Then `claude -p` for CI.
4. **5 min, structured output.** Tool use plus a schema fixes syntax, not semantics; nullable fields; retry with specific errors; batch only non-blocking work.
5. **2 min, quiz.** Read three symptoms from the playbook and have them name the fix.

## Sources

- [CCA-F Official Exam Guide, task-by-task breakdown](https://jc-aiwiki.pages.dev/claude-ai/official-exam-guide)
- [CCAR-F Exam Guide 2026, OpenExamPrep](https://open-exam-prep.com/blog/cca-f-claude-certified-architect-foundations-guide-2026)
- [Anthropic Partner Academy prep courses](https://anthropic-partners.skilljar.com/page/claude-certified-architect-foundations-prep-courses)
- [Claude API docs](https://platform.claude.com/docs/en/home) · [Claude Code docs](https://code.claude.com/docs/en/overview) · [MCP](https://modelcontextprotocol.io/)


---

← [Scenario Playbook](Scenario-Playbook.md) · [Home](README.md) · [FAQ](FAQ.md) →

Unofficial CCAF study guide · not affiliated with Anthropic · [Interactive site](https://ppotdar.github.io/ccaf-study-course/) · © 2026 Puja Potdar · [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)
