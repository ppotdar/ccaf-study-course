# Exam overview and study plan

CCAF is Anthropic's Claude Certified Architect – Foundations: 60 scenario-based questions in 120 minutes, pass at 720 on a 100–1000 scaled score. Its current exam code is CCAR-F; older material says CCA-F.

| Item | Detail |
| --- | --- |
| Questions | 60 multiple choice: 1 correct, 3 plausible distractors |
| Time | 120 minutes, about 2 minutes per question |
| Passing score | 720 / 1000 scaled (not exactly 72% raw) |
| Guessing | No penalty; a blank is wrong, so answer everything |
| Scenarios | 4 of 6 drawn at random; every question sits inside a scenario |
| Fee | $125 per attempt (partner discounts at checkout) |
| Delivery | Pearson VUE, online-proctored or test center; no Claude open during the exam |
| Eligibility | Claude Partner Network organizations; register with a partner company email |

## How questions are built

Every question is a production problem: “the agent does X wrong 12% of the time, what is the best fix?” All four options sound reasonable. The right one targets the **root cause** with the **smallest, most reliable** mechanism. Wrong options are usually one of four types:

- **Probabilistic fix for a deterministic need.** “Add it to the system prompt” when a rule must hold every time.
- **Over-engineering.** A new classifier, [routing layer](FAQ-Routing-Classifier.md) or model when a better tool description would do.
- **Wrong layer.** Fixing subagents when the coordinator's decomposition is the bug.
- **Hiding the problem.** Empty results on error, generic “operation failed,” bigger iteration caps.

## The 5 domains

| # | Domain | Weight | ~Qs | Tasks |
| --- | --- | --- | --- | --- |
| 1 | Agentic Architecture & Orchestration | 27% | 16 | 1.1–1.7 |
| 2 | Tool Design & MCP Integration | 18% | 11 | 2.1–2.5 |
| 3 | Claude Code Configuration & Workflows | 20% | 12 | 3.1–3.6 |
| 4 | Prompt Engineering & Structured Output | 20% | 12 | 4.1–4.6 |
| 5 | Context Management & Reliability | 15% | 9 | 5.1–5.6 |

## The 6 scenarios

| # | Scenario | Primary domains |
| --- | --- | --- |
| 1 | Customer Support Resolution Agent: Agent SDK, MCP tools `get_customer`, `lookup_order`, `process_refund`, `escalate_to_human`; 80%+ first-contact resolution | 1, 2, 5 |
| 2 | Code Generation with Claude Code: CLAUDE.md, slash commands, plan mode | 3, 5 |
| 3 | Multi-Agent Research System: coordinator + search, document analysis, synthesis subagents | 1, 2, 5 |
| 4 | Developer Productivity: explore legacy codebases with Read, Write, Bash, Grep, Glob + MCP | 2, 3, 1 |
| 5 | Claude Code for CI/CD: automated review, test generation, PR feedback | 3, 4 |
| 6 | Structured Data Extraction: unstructured docs to JSON schema, validation | 4, 5 |

## Today's 4-hour session

The modules in the sidebar follow this order. Heavier domains come first so they get your freshest attention.

| Time | Module | Why this much |
| --- | --- | --- |
| 0:00–0:20 | Start here + The 10 rules | The rules answer about half the questions on their own |
| 0:20–1:20 | Domain 1 Agentic Architecture | 27%, about 16 questions |
| 1:20–1:30 | Break |  |
| 1:30–2:10 | Domain 3 Claude Code | 20%, most of the memorize-the-file-or-flag items |
| 2:10–2:50 | Domain 4 Prompting & Structured Output | 20%, about 12 questions |
| 2:50–3:20 | Domain 2 Tools & MCP | 18%, about 11 questions |
| 3:20–3:45 | Domain 5 Context & Reliability | 15%, but it shows up inside every scenario |
| 3:45–4:00 | Scenario playbook + Cheat sheet | Ties everything to the 6 scenarios |

## Days 2–5

1. **Wed Nov 11, practice and gaps (2–3 h).** Take the full practice exam. Log every miss by task number, then reread those task cards.
2. **Thu Nov 12, hands-on (2–3 h).** Build a tool-use loop on `stop_reason`, a `.mcp.json` with `${ENV_VAR}`, a `.claude/rules/` file with `paths:`, a `.claude/commands/` command, and run `claude -p "..." --output-format json`. Then teach Domain 1 out loud in 10 minutes.
3. **Fri Nov 13, timed mock (2.5 h).** 60 questions in 120 minutes. Mix this bank with a free external set. Aim for 80%+ before booking.
4. **Sat Nov 14, repair (1.5–2 h).** Redo missed questions only, then reread the cheat sheet.
5. **Exam day.** Morning: cheat sheet and the 10 rules only. No new material.


---

[Home](README.md) · [The 10 Rules](The-10-Rules.md) →

Unofficial CCAF study guide · not affiliated with Anthropic · [Interactive site](https://ppotdar.github.io/ccaf-study-course/) · © 2026 Puja Potdar · [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)
