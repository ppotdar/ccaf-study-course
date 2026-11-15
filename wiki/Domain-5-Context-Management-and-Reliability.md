# Domain 5: Context Management & Reliability

The lightest domain by weight, but its ideas appear in every scenario: keeping the right information in context, handing off to humans, and staying honest about uncertainty.

### 5.1 Preserving information in long interactions

**Concept.** Three failure modes. Progressive summarization blurs exact facts (“$1,247.50 refund on order 88213 by Friday” becomes “wants a refund”). **Lost in the middle**: models attend best to the start and end of long input. Verbose tool results (40 fields when 5 matter) crowd out what matters.

**Exam answer.** Keep a persistent [**case-facts block**](FAQ-Protecting-Context.md) (amounts, dates, IDs, statuses, promises) in every prompt, outside summarized history. Trim tool output before it accumulates (a `PostToolUse` hook does this well). Put the key-findings summary at the **top** with clear section headers. Have upstream agents return structured data instead of prose. Send full history on each call; the API is stateless.

**Trap.** “Use a bigger context window”; summarizing more aggressively.

### 5.2 Escalation and ambiguity

| Situation | Right move |
| --- | --- |
| Customer explicitly asks for a human | Escalate **immediately**; do not investigate first |
| Frustrated, but the issue is simple and in scope | Acknowledge, offer to resolve; escalate if they ask again |
| Policy silent or ambiguous (competitor price match) | Escalate; it is a policy gap |
| No progress after reasonable attempts | Escalate with a structured handoff |
| Tool returns several matching customers | Ask for another identifier; never pick by heuristic |

**Concept.** Escalation triggers are an explicit request for a human, a policy gap, or no meaningful progress. Sentiment and self-reported confidence are not.

**Exam answer.** Put explicit escalation criteria with few-shot examples (escalate vs resolve) in the system prompt.

**Trap.** Sentiment-based escalation (anger is not complexity); self-reported confidence (poorly calibrated on exactly the hard cases); [a separate ML classifier](FAQ-Routing-Classifier.md).

### 5.3 Error propagation across agents

**Concept.** The coordinator can only recover well if it knows what failed and what is still usable.

**Exam answer.** The subagent returns structured context: failure type, the query attempted, partial results, possible alternatives. It retries transient failures locally first. Access failures are reported differently from valid empty results. Final reports carry **coverage annotations** showing which conclusions are well supported and where sources were missing.

**Trap.** A generic “search unavailable”; swallowing the error and returning empty results; letting one exception kill the job.

### 5.4 Context in large codebase exploration

**Concept.** In long sessions answers drift: Claude cites “typical patterns” instead of the specific classes it found earlier. That is context degradation.

**Exam answer.** Delegate specific questions to subagents (“find all test files,” “trace refund flow dependencies”). Keep a **scratchpad file** of key findings. Summarize each phase before spawning the next. For crash recovery, each agent writes state to a known location and the coordinator loads a **manifest** on resume. Use `/compact` when discovery output fills the context.

**Trap.** Restarting from scratch every time; one agent reading the whole repository.

### 5.5 Human review and confidence calibration

**Concept.** “97% accurate overall” can hide 70% accuracy on one document type or one field.

**Exam answer.** Measure accuracy **by document type and by field** before reducing review. Use **stratified random sampling** of high-confidence outputs to keep measuring errors and catch new patterns. Have the model emit field-level confidence, then **calibrate thresholds on a labeled validation set**. Route low-confidence or contradictory items to people first.

**Trap.** Trusting raw model confidence; auto-approving everything because the aggregate number looks good.

### 5.6 Provenance and uncertainty in synthesis

**Concept.** Attribution gets lost when findings are compressed without their sources.

**Exam answer.** Subagents output **claim-to-source mappings** (URL, document, excerpt) that synthesis preserves. When credible sources conflict, report both with attribution. Require publication or collection **dates** so a 2021 figure and a 2025 figure are not read as a contradiction. Separate settled from contested findings, and render each content type naturally (tables for financials, prose for news).

**Trap.** Averaging conflicting numbers; dropping citations while summarizing; forcing everything into one format.

## Check yourself

This domain's interactive quiz is on the [live site](https://ppotdar.github.io/ccaf-study-course/#d5). The same questions, with answers, are on [Practice Questions](Practice-Questions.md).


---

← [Domain 2 Tool Design and MCP](Domain-2-Tool-Design-and-MCP.md) · [Home](README.md) · [Scenario Playbook](Scenario-Playbook.md) →

Unofficial CCAF study guide · not affiliated with Anthropic · [Interactive site](https://ppotdar.github.io/ccaf-study-course/) · © 2026 Puja Potdar · [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)
