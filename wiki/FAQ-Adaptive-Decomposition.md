# What is adaptive decomposition (vs prompt chaining)?

**Both are ways to break a big task into steps (Task 1.6). The difference is *when* you decide the steps.**

|  | Prompt chaining | Adaptive decomposition |
| --- | --- | --- |
| Steps decided | **Up front**, before starting | **As you go**, based on what each step finds |
| Plan | Fixed, the same every time | Changes as findings come in |
| Best for | Predictable tasks with known steps | Open-ended investigation |
| Analogy | A recipe | A detective following clues |

### Prompt chaining: the fixed plan

You know the steps before starting and run them in order every time. Reviewing every PR:

1. Check security.
2. Check correctness.
3. Check test coverage.
4. Write the summary.

Every PR goes through the same four steps: predictable, consistent, easy to debug.

### Adaptive decomposition: the plan that evolves

You *can't* know the steps in advance, because each finding decides the next move. “Checkout got twice as slow last week. Why?”

1. **Map the system:** what does checkout touch? Payments, inventory, pricing, tax.
2. **Find high-impact areas:** what changed last week? A pricing deploy and a new tax provider.
3. **Prioritize:** investigate the tax provider first (new, external, likely slow).
4. **Adapt:** tax calls look fine, but the pricing service now hits the database once per cart item. **New subtask:** check that query.
5. **Keep adapting** until the cause is found.

No fixed recipe could have predicted step 4; the plan was rewritten because of what was discovered.

**Pattern to memorize:** map → prioritize → investigate → adapt.

### A useful combination

Large code reviews mix both ideas: a fixed chain of **per-file passes**, then **one cross-file integration pass** ([Rule 9](FAQ-Independent-Review.md)). That's still prompt chaining, because the steps are known up front. Adaptive decomposition is only for when the next step genuinely depends on what you find.

### How it shows up on the exam

| Question says | Answer |
| --- | --- |
| “Review every PR for security, style and tests” | Prompt chaining |
| “Investigate why X broke / is slow” | Adaptive decomposition |
| “Add test coverage to a large legacy codebase” | Adaptive: map, find high-risk areas, plan, adapt |
| “Review a 14-file PR” | Chaining: per-file passes plus a cross-file pass |

**Traps:**

- **A fixed pipeline for an investigation** (“analyze all 40 services alphabetically”): wastes effort and ignores clues.
- **One giant prompt** (“here's the codebase and the latency graph, find the bug”): attention diluted, no structure.
- **Adaptive decomposition for a repeatable process:** loses the consistency you need.

### How it relates to coordinator decomposition

[Coordinator decomposition](FAQ-Coordinator-Decomposition.md) is *who* breaks the task down: the coordinator, splitting work across subagents. Chaining vs adaptive is *how* it's broken down: a fixed plan or an evolving one. A coordinator can use either.

**Remember:** Prompt chaining is a recipe: same steps every time. Adaptive decomposition is detective work: map, prioritize, and let each finding decide the next step.

Where this comes up: [Task 1.6 Task decomposition](Domain-1-Agentic-Architecture.md) · [Task 1.2 Coordinator and subagents](Domain-1-Agentic-Architecture.md) · [Task 4.6 Multi-pass review](Domain-4-Prompt-Engineering-and-Structured-Output.md) · [Task 5.4 Codebase exploration](Domain-5-Context-Management-and-Reliability.md)


---

← [FAQ Hooks](FAQ-Hooks.md) · [Home](README.md) · [FAQ Resume Fork Fresh Start](FAQ-Resume-Fork-Fresh-Start.md) →

Unofficial CCAF study guide · not affiliated with Anthropic · [Interactive site](https://ppotdar.github.io/ccaf-study-course/) · © 2026 Puja Potdar · [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)
