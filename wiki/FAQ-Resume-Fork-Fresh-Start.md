# Resume, fork or fresh start: which one, and what are the tricky questions?

**Three ways to continue earlier work (Task 1.7). The exam tests whether you pick the right one.**

| Option | What it does | Analogy |
| --- | --- | --- |
| **Resume** (`--resume <session-name>`) | Continues a named session with its full history | Reopening a saved document |
| **Fork** (`fork_session`) | Copies a session's state into an independent branch | “Save as” a copy, or a git branch |
| **Fresh start + summary** | New session with a structured summary injected at the start | A new document plus a briefing memo |

### Resume

```
claude --resume payments-investigation
```

Same conversation, same tool results, same decisions. Naming sessions matters: with several investigations open, the name gets the right one instead of “the most recent.”

- **Use it when** the earlier context is **still mostly true**.
- **The catch:** the session remembers *old* tool results. If files changed, **tell it what changed**: “Since yesterday, `refund.py`, `ledger.py` and `config.yaml` were modified. Re-read them before continuing.”

### Fork

```
          analysis of legacy payments module
                       │
          ┌────────────┴────────────┐
       fork A                     fork B
  "refactor with              "refactor with
   strategy pattern"           event sourcing"
          │                         │
     try it, evaluate          try it, evaluate
```

- **Use it when** comparing alternatives from the same starting point, without redoing the analysis.
- **Why not try both in one session?** Attempt B would remember attempt A: biased thinking, cluttered context. Forks keep each attempt clean.

### Fresh start + summary

A new session given key findings, decisions, open questions and which files matter.

- **Use it when** earlier tool results are **stale**: many files changed, or the old session has drifted.
- **Why not resume?** Outdated tool output gets treated as current. A clean summary carries only what's still true.

### The decision guide

```
Is the old context still mostly accurate?
├── Yes → RESUME (and tell it exactly what changed)
└── No  → FRESH START with a structured summary

Want to compare two approaches from the same baseline?
└── Yes → FORK
```

### Tricky questions you might see

**1. A few files changed:** You resume a named investigation. Overnight a teammate edited 3 of the 40 files you analyzed; answers now describe old code.

**Right:** Resume and tell the session which 3 files changed so it re-reads them.

**Distractors:** 

- “Start fresh and redo the analysis”: throws away 37 files of valid work.
- “Fork the session”: the fork copies the same stale context.
- “Run `/compact`”: shrinks context but doesn't refresh stale facts.

**2. Almost everything changed:** Same setup, but a large refactor touched about 80% of the files.

**Right:** Fresh start with a structured summary of findings that are still valid.

**Distractors:** 

- “Resume and list every changed file”: most old tool results are now wrong.

**The trick:** Questions 1 and 2 look nearly identical. The deciding detail is **how much changed**.

**3. Compare two approaches:** After a long analysis, you want to try two refactoring strategies and compare.

**Right:** Fork from the shared analysis.

**Distractors:** 

- “Two new sessions that each redo the analysis”: wasteful.
- “Resume and try one, then the other”: the second attempt is biased by the first.
- “Plan mode”: designs one approach before editing; doesn't run two in isolation.

**4. Fork vs parallel subagents:** Two things sound like “run in parallel.”

**Right:** **Fork** = *alternatives* to the same problem (“which approach is better?”). **Parallel `Task` calls** = *different parts* of one task (“search news, papers and patents”). Comparing options: fork. Splitting work: [parallel subagents](FAQ-Spawning-Subagents.md).

**Distractors:** 

- Using fork to split one task into parts.
- Using parallel subagents to compare two strategies.

**5. “Forks share updates”:** A distractor claims a fix in branch A automatically appears in branch B.

**Right:** False. Forks are independent; that's the point of forking.

**Distractors:** 

- Any option that relies on branches staying in sync.

**6. The session has gone vague:** Three hours in, Claude refers to “typical patterns” instead of the classes it found.

**Right:** Fresh start with a summary, or a scratchpad, subagents and `/compact` (Task 5.4).

**Distractors:** 

- “Resume it tomorrow”: the degradation comes with it.

**7. Decisions valid, tool output stale:** Yesterday's architecture decisions stand, but the code they were based on was reworked.

**Right:** Fresh start, injecting a summary of the **decisions** without the stale tool output.

**Distractors:** 

- “Resume, since the decisions are still valid.”

**The trick:** Judge “mostly valid” on the **tool results**, not just the decisions. Old file reads are what mislead Claude.

**8. Crash recovery in a multi-agent system:** A coordinator with five subagents crashes mid-job.

**Right:** Each agent exports state to a known location; the coordinator loads a manifest on restart (Task 5.4).

**Distractors:** 

- “`--resume` the coordinator session”: restores only the coordinator. Subagents start blank, so their progress is lost.

**9. Which session?:** Several investigations are open; continue the payments one.

**Right:** `--resume payments-investigation`, a **named** session.

**Distractors:** 

- “Continue the most recent session”: might be a different investigation.

### Signal words

| Question says | Answer |
| --- | --- |
| “pick up where we left off,” “a few files changed” | Resume, and tell it what changed |
| “most files changed,” “tool results outdated,” “session degraded” | Fresh start with a structured summary |
| “compare,” “try two approaches,” “same starting analysis” | Fork |
| “split the work,” “different sources at the same time” | Parallel subagents, not fork |
| “crashed,” “restore all agents” | State exports plus a manifest |

**Remember:** Resume when the old context is still true (and say what changed). Start fresh with a summary when it's stale. Fork to compare alternatives from the same starting point.

Where this comes up: [Task 1.7 Session state](Domain-1-Agentic-Architecture.md) · [Task 1.3 Spawning subagents](Domain-1-Agentic-Architecture.md) · [Task 5.4 Codebase exploration](Domain-5-Context-Management-and-Reliability.md) · [Task 3.4 Plan mode](Domain-3-Claude-Code-Configuration.md) · [Cheat sheet](Cheat-Sheet.md)


---

← [FAQ Adaptive Decomposition](FAQ-Adaptive-Decomposition.md) · [Home](README.md) · [FAQ Slash Commands and Skills](FAQ-Slash-Commands-and-Skills.md) →

Unofficial CCAF study guide · not affiliated with Anthropic · [Interactive site](https://ppotdar.github.io/ccaf-study-course/) · © 2026 Puja Potdar · [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)
