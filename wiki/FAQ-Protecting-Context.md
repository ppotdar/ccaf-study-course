# How do you protect what matters in context? (Rule 10)

**Claude only knows what's in its context window right now.** In long conversations and multi-agent systems, that window fills with noise, gets summarized, and gets passed between agents. Every step can quietly lose the one detail that matters. Rule 10 means deciding **what must survive** and protecting it on purpose.

| Technique | Protects against |
| --- | --- |
| **Case-facts block:** exact facts pinned to every prompt, outside summarized history | Summaries blurring “$1,247.50 by Friday” into “wants a refund” |
| **Trim tool output:** keep only the fields that matter, ideally in code | 45-field lookups crowding out the conversation |
| **Key summary first, clear headers** | “Lost in the middle”: models attend most to the start and end of long inputs |
| **Provenance:** claim-to-source mappings and dates through every hand-off | Citations disappearing; old data read as a contradiction |

### Simple scenarios

**1. The vanishing refund amount:** Turn 3: “I was charged $1,247.50 twice for order 88213, and the agent promised a refund by Friday.” By turn 40, older turns are summarized as “Customer had a billing issue,” and the agent asks the customer to repeat everything.

**Fix:** Extract the facts into a block attached to every prompt and never summarized:
```
CASE FACTS
order: 88213 | duplicate charge: $1,247.50 | refund promised: by Fri Nov 13
status: refund pending | customer ID: C-55102 (verified)
```

**Tempting distractor:** “Use a bigger context window” or “summarize less often.”

**2. The bloated lookup:** Every `lookup_order` call returns 45 fields (warehouse codes, internal SKUs, audit timestamps). After four lookups, most of the context is fields nobody asked about.

**Fix:** A `PostToolUse` hook keeps only order ID, status, amount, dates and items. Deterministic, and no wasted tokens.

**Tempting distractor:** “Tell Claude to ignore irrelevant fields.” They still take up the space.

**3. The forgotten middle:** Synthesis receives results from 12 research subagents in one long input. Findings from subagents 5 to 8 keep getting left out.

**Fix:** Put a short key-findings summary at the top, then each subagent's results under a clear header.

**Tempting distractor:** “Shuffle the order each run” or “cut to three subagents.”

### Complex scenarios

**4. Multi-issue support session ending in escalation:** Over 50 turns a customer raises a duplicate charge, a damaged item, and an address change. The agent resolves two; the customer then asks for a human about the third.

**What has to survive:** Track facts **per issue** (“charge: refunded”, “damage: replacement shipped”, “address: blocked, item in transit”) plus every promise with its date. The human gets a structured handoff built from those facts (customer ID, each issue and status, what was tried, recommended next step), not the raw transcript. **The hard part:** facts change, so the block must be **updated**, not only appended to, or the human sees stale information.

**5. Multi-agent research with “conflicting” numbers:** Subagent A finds “EV market share: 18%” in a 2023 report; subagent B finds 24% in a 2025 report. Synthesis writes “Sources disagree (18% vs 24%).” There's no disagreement: the share grew. The dates were dropped between agents.

**Fix:** Each subagent returns structured claims with `source`, `published` and `excerpt`; the coordinator passes them through intact, and synthesis reports a trend. If two 2025 figures truly disagreed, report both with sources rather than averaging or picking one.

**Tempting distractor:** Average the two numbers, or keep whichever looks more recent without checking dates.

**6. Three-hour codebase exploration:** Early on, Claude correctly identifies `LedgerReconciler` and `RefundGateway`. Two hours later it says things like “this typically uses a service layer.” It has lost the specifics.

**Fixes, in combination:** Combine: a **scratchpad file** (`findings.md`) re-read before each answer; **subagents** for verbose digging that return summaries; **phase summaries** injected into the next phase; `/compact` only after key facts are saved; and a **manifest** for crash recovery.

**Tempting distractor:** Keep going in the same session, or restart and re-read the whole codebase.

### Where it can go wrong

These failures happen even when you think you've followed Rule 10.

1. **A stale case-facts block.** “Refund pending” is never updated to “refund issued.” Protected facts that are confidently wrong are worse than missing ones.
2. **Storing interpretations instead of facts.** “Customer is angry and probably exaggerating” isn't a fact. Store what was said and verified; opinions in the block bias every later turn.
3. **Over-trimming.** The hook drops `delivery_date` because it usually doesn't matter, then a return-window question arrives. Trim to what tasks need, and revisit when new task types appear.
4. **Trimming in the prompt instead of code.** “Ignore irrelevant fields” still pays for the tokens. Trimming belongs in a hook or in the tool's response design.
5. **A wrong or lossy summary at the top.** First position gives it extra weight. If it drops a caveat (“only in a sample of 40 patients”), you've amplified the error.
6. **A case-facts block that bloats.** If every detail goes in, it becomes the new noise. Keep only facts that change decisions.
7. **Dropping provenance during “cleanup.”** A step turns structured claims into tidy prose; sources vanish until someone asks where 24% came from.
8. **Compacting before saving.** Running `/compact` before findings are in a scratchpad can lose them for good.
9. **A scratchpad nobody re-reads.** Writing findings down only helps if the agent is told to consult them.
10. **Sensitive data in protected context.** A block pinned to every prompt repeats card numbers or IDs in every call, log and handoff. Protect only what the task needs and mask the rest (last four digits). Not the exam's focus, but critical in regulated systems.

### Exam signals

| Question says | Answer |
| --- | --- |
| “Details from early in the conversation are lost after summarization” | Case-facts block outside summarized history |
| “Tool results return many fields; context fills quickly” | Trim to relevant fields (a hook) |
| “Findings from the middle of the input are omitted” | Key summary first, clear section headers |
| “Report cites nothing” / “attribution lost” | Claim-to-source mappings preserved through every step |
| “Figures flagged as contradictory but from different years” | Require publication or collection dates |
| “Answers become vague in a long exploration session” | Scratchpad, subagents, phase summaries, `/compact` |

Reject: **“use a bigger context window,” “summarize more aggressively,”** and **“tell Claude to remember.”** None of these protects anything.

**Remember:** Decide what must survive (facts, sources, dates) and keep it where every prompt can see it. Trim noise in code, put what matters first, and keep the protected facts current.

Where this comes up: [Rule 10](The-10-Rules.md) · [Task 5.1 Preserving information](Domain-5-Context-Management-and-Reliability.md) · [Task 5.4 Codebase exploration](Domain-5-Context-Management-and-Reliability.md) · [Task 5.6 Provenance](Domain-5-Context-Management-and-Reliability.md) · [Task 1.5 Hooks](Domain-1-Agentic-Architecture.md)


---

← [FAQ Independent Review](FAQ-Independent-Review.md) · [Home](README.md) · [FAQ Spawning Subagents](FAQ-Spawning-Subagents.md) →

Unofficial CCAF study guide · not affiliated with Anthropic · [Interactive site](https://ppotdar.github.io/ccaf-study-course/) · © 2026 Puja Potdar · [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)
