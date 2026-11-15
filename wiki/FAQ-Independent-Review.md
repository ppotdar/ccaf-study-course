# Why do independent eyes review better? (Rule 9)

**Rule 9 has two parts.** Don't let the author grade its own work: a fresh Claude instance reviews better than the one that produced the output. And don't review everything in one gulp: big reviews work better as focused passes plus one pass that looks across them.

### Part 1: Why self-review fails

When a session writes code, its context holds all the reasoning behind it: “I'll skip the null check because this field is always set.” Ask that same session to review the code and it re-reads its own reasoning and agrees with itself. It's why you can't proofread your own essay right after writing it: you see what you meant, not what's on the page.

A **separate instance** starts with only the code and the review criteria. It doesn't know why each choice was made, so it has to check whether each choice is actually right.

**Example 1: CI code review:** A Claude Code session builds a checkout feature. The pipeline asks the same session to review the PR, and it approves a bug where a coupon can push the total below zero.

**Right fix:** Run the review in a new, independent session that gets the diff, CLAUDE.md standards and review criteria, but not the generating conversation.

**Tempting distractor:** “Ask the same session to review again, more carefully,” or “add ‘be very critical’ to the prompt.” Same biased context, louder instructions.

**Example 2: Data extraction:** One instance extracts invoice fields. You want to catch its errors.

**Right fix:** A second instance checks the extraction against the source document, paired with code-level checks such as `calculated_total` vs `stated_total`.

**Tempting distractor:** Ask the extracting instance “are you sure?”

### Part 2: Why one big review fails

Give Claude a 20-file PR in one pass and its attention spreads thin: deep comments on the first few files, shallow or missed issues in later ones, and feedback that contradicts itself. This is **attention dilution**. A bigger context window doesn't fix it; the model can *fit* all 20 files, it just doesn't attend to them evenly.

1. **Per-file passes:** review each file on its own, with full attention, for local bugs.
2. **One cross-file integration pass:** look only at how files interact: data flowing between them, interface mismatches, a signature changed in one file but not in its callers.

**Example 3: The 20-file PR:** Developers complain the review bot misses bugs in later files and contradicts itself.

**Right fix:** 20 per-file passes, then one cross-file integration pass.

**Tempting distractor:** “Increase `max_tokens`,” “use a larger context window,” or “run the full-PR review twice and keep findings both report.”

**Example 4: The bug only integration catches:** `orders.py` changes `get_total()` to return cents instead of dollars. Each file looks fine on its own.

**Why both passes:** Only the cross-file pass notices that `invoice.py` still treats the value as dollars: a 100x billing error.

### How it shows up on the exam

| Question says | Answer |
| --- | --- |
| “The session that wrote the code reviewed it and missed bugs” | Independent instance without the generator's context |
| “Review of a large PR is inconsistent, shallow on later files” | Per-file passes plus a cross-file integration pass |
| “Each file looks correct but the system breaks” | The cross-file integration pass |
| “Self-review isn't catching errors; one option is ‘review twice’” | Reject it: same context, same blind spots |

**Remember:** The author can't see its own blind spots, so use a fresh reviewer. Big reviews get one focused pass per file, then one pass to see how the pieces fit.

Where this comes up: [Rule 9](The-10-Rules.md) · [Task 4.6 Independent and multi-pass review](Domain-4-Prompt-Engineering-and-Structured-Output.md) · [Task 3.6 Claude Code in CI/CD](Domain-3-Claude-Code-Configuration.md) · [Task 1.6 Task decomposition](Domain-1-Agentic-Architecture.md)


---

← [FAQ Runaway Loops](FAQ-Runaway-Loops.md) · [Home](README.md) · [FAQ Protecting Context](FAQ-Protecting-Context.md) →

Unofficial CCAF study guide · not affiliated with Anthropic · [Interactive site](https://ppotdar.github.io/ccaf-study-course/) · © 2026 Puja Potdar · [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)
