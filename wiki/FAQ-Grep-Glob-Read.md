# What are Grep and Glob (and Read)?

**Three of Claude Code’s built-in tools for *looking at* a codebase.** They’re read-only: none of them can change a file, which is why they run without asking for approval.

| Tool | Finds things by | Question it answers | Everyday analogy |
| --- | --- | --- | --- |
| **Glob** | File **name / path** | “Which files exist that match this pattern?” | Searching a filing cabinet by folder labels |
| **Grep** | File **contents** | “Which files contain this text?” | Ctrl+F across every document at once |
| **Read** | — opens one file | “What’s in this file?” | Opening the document and reading it |

### Glob: find files by name

Glob matches file **paths** with wildcard patterns. It never looks inside files. `*` means “anything in this folder”; `**` means “any number of folders deep.”

| Pattern | Matches |
| --- | --- |
| `*.py` | Python files in the current folder |
| `**/*.test.tsx` | Every `.test.tsx` file in any folder, at any depth |
| `src/**/config*` | Anything starting with “config” anywhere under `src/` |
| `**/Dockerfile` | Every Dockerfile in the repo |

These are the same globs used in `.claude/rules/` `paths:` frontmatter (Task 3.3).

### Grep: find text inside files

Grep searches file **contents** for a word or pattern and returns the matching lines and files.

| Search | Finds |
| --- | --- |
| `processRefund` | Every place the function is defined or called |
| `"Connection refused"` | Where an error message is produced |
| `import stripe` | Every file that uses the Stripe library |
| `TODO\|FIXME` | Every to-do comment |

### Read: open one file

Read returns a file’s full contents so Claude can follow the actual logic. Its partners for *changing* files, Edit and Write, need approval.

### How they work together

Exploring an unfamiliar codebase: “how do refunds work?”

1. **Glob** `**/*refund*` → finds `refund_service.py`, `refund_api.py`, `test_refund.py`.
2. **Grep** `processRefund` → it’s called from `checkout.py` and `admin.py`.
3. **Read** `refund_service.py` → follow the actual logic.
4. Repeat: Grep for what it imports, Read those files.

This is the Task 2.5 pattern: *Grep to find entry points, Read to follow imports and trace flows*. Build understanding step by step instead of reading every file up front.

### Exam signals

| Question says | Answer |
| --- | --- |
| “Find all files named `*.config.ts`” | **Glob** |
| “Find everywhere `applyDiscount` is called” | **Grep** |
| “Find where this error message comes from” | **Grep** |
| “Understand how this function works” | **Read** |
| Edit fails because its anchor text isn’t unique | **Read**, then **Write** the whole file |

**Remember:** Glob = file names. Grep = file contents. Read = open one file. The trap swaps the first two.

Where this comes up: [Task 2.5 Built-in tools](Domain-2-Tool-Design-and-MCP.md) · [Card X1 Permissions](Domain-3-Claude-Code-Configuration.md) · [Task 3.3 Path-specific rules](Domain-3-Claude-Code-Configuration.md) · [Task 5.4 Codebase exploration](Domain-5-Context-Management-and-Reliability.md)


---

← [FAQ CLAUDE.md Precedence](FAQ-CLAUDE.md-Precedence.md) · [Home](README.md)

Unofficial CCAF study guide · not affiliated with Anthropic · [Interactive site](https://ppotdar.github.io/ccaf-study-course/) · © 2026 Puja Potdar · [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)
