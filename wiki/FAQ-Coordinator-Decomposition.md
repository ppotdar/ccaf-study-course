# What does “coordinator decomposition” mean?

**Coordinator decomposition is how the lead agent splits a big task into smaller subtasks and hands each one to a helper agent.**

A multi-agent system has two roles:

- **The coordinator is the manager.** It receives the request, decides which pieces of work are needed, and assigns each piece to a subagent.
- **The subagents are the workers.** Each does only the piece it was given. They don't see the original request or each other's work.

“Decomposition” just means “breaking down.” So coordinator decomposition is the coordinator's plan: which subtasks exist and what each one covers.

### An everyday analogy

You ask a manager for a report on “how remote work affects companies.” The manager assigns three people:

1. Research productivity studies.
2. Research employee satisfaction surveys.
3. Research collaboration tools.

All three do excellent work. But the report says nothing about office real estate, hiring across cities, or tax and legal issues. Nobody did their job badly. The **manager's breakdown** left those areas out. That is a coordinator decomposition failure.

### How it shows up on the exam

*“The research report on creative industries covers only visual arts and misses music, film and writing. Logs show every subagent completed its task correctly. What is the root cause?”*

The wrong options blame a worker: the search agent's queries were too narrow, the synthesis agent dropped findings, or the context window was too small. The phrase **“every subagent completed its task correctly”** is your signal. If the workers did their jobs and coverage is still incomplete, the gaps were never assigned. The answer is that **the coordinator's decomposition was too narrow.**

### What good decomposition looks like

- **Covers the whole topic.** Together, the subtasks span everything the question asks.
- **Doesn't overlap.** Two subagents don't research the same thing twice.
- **Fits the question.** A simple factual question may need one subagent, not all five.
- **Checks its own coverage.** When results come back, the coordinator looks for gaps and sends out more subtasks if something is missing.

**Remember:** Workers did their job but something is missing? Blame the manager's plan.

Where this comes up: [Rule 2](The-10-Rules.md) · [Task 1.2 Coordinator and subagents](Domain-1-Agentic-Architecture.md) · [Task 1.6 Task decomposition](Domain-1-Agentic-Architecture.md) · [Scenario playbook](Scenario-Playbook.md)


---

← [FAQ](FAQ.md) · [Home](README.md) · [FAQ Routing Classifier](FAQ-Routing-Classifier.md) →

Unofficial CCAF study guide · not affiliated with Anthropic · [Interactive site](https://ppotdar.github.io/ccaf-study-course/) · © 2026 Puja Potdar · [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)
