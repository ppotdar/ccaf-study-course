# CCAF Study Course Wiki

> Browse these pages right here on GitHub. Each page ends with Previous · Home · Next links.

A step-by-step study guide for the **Claude Certified Architect – Foundations** exam (CCAF, exam code CCAR-F): 60 scenario questions, 120 minutes, pass at 720/1000.

**Interactive version** with progress tracking and scored quizzes: [https://ppotdar.github.io/ccaf-study-course/](https://ppotdar.github.io/ccaf-study-course/)

## Study path

Work through the pages in this order. Heavier domains come first.

| # | Page | What you'll learn |
| --- | --- | --- |
| 1 | [Exam Overview](Exam-Overview.md) | Format, the 5 domains, the 6 scenarios, study plan |
| 2 | [The 10 Rules](The-10-Rules.md) | The principles that decide most questions, with worked examples |
| 3 | [Domain 1: Agentic Architecture](Domain-1-Agentic-Architecture.md) | Agent loops, coordinators and subagents, hooks, sessions (27%) |
| 4 | [Domain 3: Claude Code Configuration](Domain-3-Claude-Code-Configuration.md) | CLAUDE.md, commands and skills, permissions, hooks, CI (20%) |
| 5 | [Domain 4: Prompt Engineering & Structured Output](Domain-4-Prompt-Engineering-and-Structured-Output.md) | Criteria, few-shot, schemas, retries, Batch API (20%) |
| 6 | [Domain 2: Tool Design & MCP](Domain-2-Tool-Design-and-MCP.md) | Tool descriptions, MCP errors, tool_choice, built-in tools (18%) |
| 7 | [Domain 5: Context Management & Reliability](Domain-5-Context-Management-and-Reliability.md) | Context, escalation, error propagation, provenance (15%) |
| 8 | [Scenario Playbook](Scenario-Playbook.md) | Symptom → answer for all six scenarios |
| 9 | [Cheat Sheet](Cheat-Sheet.md) | Files, flags and signal words for exam morning |
| 10 | [FAQ](FAQ.md) | 15 deep dives on the tricky concepts |
| 11 | [Practice Questions](Practice-Questions.md) | 106 questions with answers and explanations |

## How each domain page is written

Every task statement follows the same pattern, so you can teach it to someone else:

- **Concept**: what it is
- **Exam answer**: what the right option usually looks like
- **Trap**: the plausible-but-wrong distractor

---

Unofficial study guide, not affiliated with or endorsed by Anthropic. © 2026 Puja Potdar · [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)

## All pages

**Study path**

1. [Exam Overview](Exam-Overview.md)
2. [The 10 Rules](The-10-Rules.md)
3. [D1 Agentic Architecture](Domain-1-Agentic-Architecture.md)
4. [D3 Claude Code Config](Domain-3-Claude-Code-Configuration.md)
5. [D4 Prompting & Output](Domain-4-Prompt-Engineering-and-Structured-Output.md)
6. [D2 Tools & MCP](Domain-2-Tool-Design-and-MCP.md)
7. [D5 Context & Reliability](Domain-5-Context-Management-and-Reliability.md)
8. [Scenario Playbook](Scenario-Playbook.md)
9. [Cheat Sheet](Cheat-Sheet.md)
10. [Practice Questions](Practice-Questions.md)

**FAQ**

- [What does “coordinator decomposition” mean?](FAQ-Coordinator-Decomposition.md)
- [What is a routing classifier?](FAQ-Routing-Classifier.md)
- [Can the synthesis agent read the coordinator's history?](FAQ-Subagent-Context.md)
- [How does stop_reason work?](FAQ-stop_reason.md)
- [How do you safeguard against runaway loops?](FAQ-Runaway-Loops.md)
- [Why do independent eyes review better? (Rule 9)](FAQ-Independent-Review.md)
- [How do you protect what matters in context? (Rule 10)](FAQ-Protecting-Context.md)
- [What does “spawning” a subagent mean?](FAQ-Spawning-Subagents.md)
- [Why use a programmatic prerequisite instead of a prompt?](FAQ-Programmatic-Prerequisites.md)
- [How do hooks work, and when should you use them?](FAQ-Hooks.md)
- [What is adaptive decomposition (vs prompt chaining)?](FAQ-Adaptive-Decomposition.md)
- [Resume, fork or fresh start: which one, and what are the tricky questions?](FAQ-Resume-Fork-Fresh-Start.md)
- [How do slash commands and skills work?](FAQ-Slash-Commands-and-Skills.md)
- [Which CLAUDE.md wins: project level or folder level?](FAQ-CLAUDE.md-Precedence.md)
- [What are Grep and Glob (and Read)?](FAQ-Grep-Glob-Read.md)

**[Interactive site](https://ppotdar.github.io/ccaf-study-course/)**
