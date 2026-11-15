# Which CLAUDE.md wins: project level or folder level?

**The folder-level (subdirectory) CLAUDE.md wins when the two conflict.** The more specific scope wins.

```
repo/
├── CLAUDE.md              ← project level: "Use 2-space indentation"
└── legacy/
    └── CLAUDE.md          ← folder level: "Use 4-space indentation"
```

| Claude is editing | Result | Why |
| --- | --- | --- |
| `legacy/billing.py` | **4 spaces** | The folder rule supersedes the project rule |
| `src/app.py` | **2 spaces** | Only the project rule applies there |
| `legacy/billing.py`, with project rule “write tests for every change” | **Tests still required** | No conflict, so the project rule still applies inside `legacy/` |

Both files load; only the **conflicting** instruction is overridden. They are never averaged: 2 and 4 never become 3. The folder file loads when Claude works on files in that folder, so it affects only that part of the codebase.

### The full order, general to specific

enterprise → user (`~/.claude/CLAUDE.md`) → project (`./CLAUDE.md`) → subdirectory (`legacy/CLAUDE.md`)

On the exam, apply “most specific wins.” The exception in spirit is the enterprise level: it’s the organization’s policy, set by IT, and meant to hold for everyone.

**Nuance.** CLAUDE.md files are instructions in Claude’s context, so “supersede” means Claude follows the more specific one; it isn’t enforced by code. If a rule must *never* break (never touch `migrations/`), a CLAUDE.md line isn’t enough: use a `deny` permission rule or a [hook](FAQ-Hooks.md) ([Rule 1](The-10-Rules.md)).

### Exam signals

| Question says | Answer |
| --- | --- |
| “Subfolder and root CLAUDE.md disagree; which applies to files in the subfolder?” | The subfolder’s |
| An option says the two are merged, averaged or combined into a compromise | Reject it |
| “Same convention for one file type across many folders” | Neither: `.claude/rules/` with `paths` |
| “This rule must never be broken” | A `deny` rule or hook, not CLAUDE.md |

**Remember:** All CLAUDE.md layers load. On a conflict the most specific wins outright (folder beats project beats user); non-conflicting rules from every layer still apply. Must-never rules need code, not CLAUDE.md.

Where this comes up: [Task 3.1 CLAUDE.md hierarchy](Domain-3-Claude-Code-Configuration.md) · [Card X2](Domain-3-Claude-Code-Configuration.md) · [Common exam traps](Domain-3-Claude-Code-Configuration.md)


---

← [FAQ Slash Commands and Skills](FAQ-Slash-Commands-and-Skills.md) · [Home](README.md) · [FAQ Grep Glob Read](FAQ-Grep-Glob-Read.md) →

Unofficial CCAF study guide · not affiliated with Anthropic · [Interactive site](https://ppotdar.github.io/ccaf-study-course/) · © 2026 Puja Potdar · [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)
