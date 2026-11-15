# How do slash commands and skills work?

**Both package a reusable instruction in Claude Code so you don't retype the same long prompt (Task 3.2).** The exam tests two things: **where the file goes** (personal or team) and **which one to use** (command, skill or CLAUDE.md).

### Slash commands

A slash command is a **saved prompt** in a Markdown file. The file name becomes the command name: `.claude/commands/review.md` becomes `/review`.

```
---
description: Review the current changes against team standards
argument-hint: [file or folder to review]
allowed-tools: Read, Grep, Glob
---
Review $ARGUMENTS for:
1. Bugs that cause wrong results
2. Security issues
3. Missing tests
Skip style and formatting. Report each finding as: file, line, issue, suggested fix.
```

Typing `/review src/payments` sends that prompt with `$ARGUMENTS` replaced by `src/payments`. A command runs **only when someone types it**.

### Skills

A skill is a **folder** with a `SKILL.md` file, plus optional scripts, templates or reference files. It packages a **procedure** Claude can use when the task calls for it.

```
.claude/skills/release-notes/
├── SKILL.md
├── template.md
└── format_changelog.py
```
```
---
name: release-notes
description: Draft release notes from merged PRs since the last tag. Use when asked for release notes or a changelog.
context: fork
allowed-tools: Bash, Read
argument-hint: [version number]
---
1. Find the last git tag.
2. List PRs merged since then.
3. Group them into Features, Fixes and Breaking changes using template.md.
```

- **It can bundle files:** templates, scripts and reference docs live in the same folder.
- **Claude can pick it up on its own.** The `description` says when the skill applies, just as a tool description says when to use a tool. A vague description means the skill gets used at the wrong times, or never.

### The three frontmatter fields the exam names

| Field | What it does | When the exam wants it |
| --- | --- | --- |
| `context: fork` | Runs the skill in an **isolated sub-agent**; only its final result returns to the main conversation | The skill produces pages of exploratory output that clutter the chat |
| `allowed-tools` | Restricts which tools the skill may use | “This skill must only read files, never run shell commands” |
| `argument-hint` | Shows what argument is expected, prompting for it if missing | “Developers keep invoking `/triage` without the ticket ID” |

`context: fork` is the same idea as a [spawned subagent](FAQ-Spawning-Subagents.md): the noisy work happens in its own context and you get back a summary, which protects your main context ([Rule 10](FAQ-Protecting-Context.md)).

### Where the files go: team vs personal

|  | Team (shared through git) | Personal (only you) |
| --- | --- | --- |
| Slash commands | `.claude/commands/` in the repo | `~/.claude/commands/` |
| Skills | `.claude/skills/<name>/SKILL.md` in the repo | `~/.claude/skills/<name>/SKILL.md` |

**Same pattern as CLAUDE.md and MCP:** the repo's `.claude/` folder is the team's; `~/.claude/` in your home folder is yours.

### Commands vs skills vs CLAUDE.md

All three hold instructions. The difference is **when they load**:

|  | CLAUDE.md | Skill | Slash command |
| --- | --- | --- | --- |
| Loads | **Always**, every session | **On demand**, when relevant or invoked | **Only when typed** |
| Best for | Universal standards | Multi-step procedures used sometimes | A prompt you trigger deliberately |
| Example | “Use TypeScript strict mode” | “Draft release notes,” “run a security audit” | `/review`, `/deploy-check` |

**The test:** applies to *every* task? CLAUDE.md. A procedure for *some* tasks? A skill. Want to trigger it by name? A command. A 12-step procedure in CLAUDE.md eats context in every session.

In recent Claude Code versions commands and skills overlap more (a skill can also be invoked by name with a slash). For the exam, keep the distinction: CLAUDE.md is always loaded, skills are on demand.

### Tricky questions you might see

**1. “Every developer who clones the repo”:** You want `/review` available to the whole team.

**Right:** `.claude/commands/review.md`, committed to the repo.

**Distractors:** 

- `~/.claude/commands/`: personal only; teammates don't get it.
- “A `commands` array in `.claude/config.json`”: **this doesn't exist**. A classic invented distractor.
- “Define it in CLAUDE.md”: CLAUDE.md holds instructions, not command definitions.

**2. A personal variant:** A developer wants their own version of the team's `/pr-summary` skill.

**Right:** Copy it into `~/.claude/skills/` **under a different name** and customize it there.

**Distractors:** 

- “Edit the project skill and don't commit”: fragile, and easy to commit by accident.
- “Same name in `~/.claude/skills/`”: a name clash makes it unclear which one runs.

**3. The output is flooding the conversation:** A codebase-survey skill dumps hundreds of lines into the chat.

**Right:** `context: fork`.

**Distractors:** 

- `allowed-tools`: controls tools, not output.
- “Run `/compact` afterwards”: the clutter already happened.

**4. “It must never run Bash”:** A skill should only read files.

**Right:** `allowed-tools: Read, Grep, Glob`.

**Distractors:** 

- “Add ‘do not use Bash’ to the skill's instructions”: probabilistic ([Rule 1](The-10-Rules.md)). `allowed-tools` enforces it.

**5. People forget the argument:** Developers keep invoking `/triage` without a ticket ID.

**Right:** `argument-hint`.

**6. A 12-step procedure in CLAUDE.md:** The team's release procedure lives in CLAUDE.md and runs once a sprint. Sessions feel bloated.

**Right:** Move it to a skill so it loads only when needed.

**Distractors:** 

- “Split CLAUDE.md with `@import`”: still always loaded.

**7. Conventions for test files across the repo:** Every `*.test.tsx` file, in any folder, follows the same conventions.

**Right:** Not a skill: `.claude/rules/` with `paths: ["**/*.test.tsx"]`, which loads automatically for matching files (Task 3.3).

**Distractors:** 

- “A skill for test conventions”: it would need to be invoked, so it would be forgotten.

**The trick:** Questions 6 and 7 both ask where instructions go. Occasional procedure: skill. Always-on for certain file types: path rules. Always-on for everything: CLAUDE.md.

**8. The skill never gets used:** A skill exists, but Claude never picks it up for relevant tasks.

**Right:** Improve the skill's `description`: what it does and when to use it.

**Distractors:** 

- “Add a line to CLAUDE.md telling Claude to use it”: works around the cause.

### Signal words

| Question says | Answer |
| --- | --- |
| “every developer who clones,” “whole team” | `.claude/commands/` or `.claude/skills/` in the repo |
| “just for me,” “experimental,” “without affecting teammates” | `~/.claude/...`, with a different name if it's a variant |
| “verbose output,” “clutters the conversation” | `context: fork` |
| “must only” / “must never use” a tool | `allowed-tools` |
| “forgets the parameter” | `argument-hint` |
| “applies to everything, always” | CLAUDE.md |
| “used occasionally,” “multi-step procedure” | Skill |
| “file types spread across folders” | `.claude/rules/` with `paths` |

**Remember:** Commands are saved prompts you type; skills are packaged procedures loaded when needed; CLAUDE.md is always on. Team files go in the repo's `.claude/`, personal ones in `~/.claude/`. Skill settings: `context: fork` (isolate output), `allowed-tools` (restrict tools), `argument-hint` (ask for input).

Where this comes up: [Task 3.2 Slash commands and skills](Domain-3-Claude-Code-Configuration.md) · [Task 3.1 CLAUDE.md hierarchy](Domain-3-Claude-Code-Configuration.md) · [Task 3.3 Path-specific rules](Domain-3-Claude-Code-Configuration.md) · [Cheat sheet](Cheat-Sheet.md)


---

← [FAQ Resume Fork Fresh Start](FAQ-Resume-Fork-Fresh-Start.md) · [Home](README.md) · [FAQ CLAUDE.md Precedence](FAQ-CLAUDE.md-Precedence.md) →

Unofficial CCAF study guide · not affiliated with Anthropic · [Interactive site](https://ppotdar.github.io/ccaf-study-course/) · © 2026 Puja Potdar · [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)
