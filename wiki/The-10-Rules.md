# The 10 rules that answer most questions

If you remember nothing else, these ten rules pick the right answer on most questions. Each comes with an exam-style example: the situation, the right fix, and the distractor the exam would tempt you with.

1. **[Must-always rules go in code, not prompts.](FAQ-Programmatic-Prerequisites.md)**Prompts and few-shot are probabilistic. Hooks, prerequisite gates and schemas are deterministic. Refund limits, identity checks before money moves, and compliance all mean hooks or gates.

   **Example:** Policy says refunds over $500 need a human. The system prompt states this in capitals, yet 3% of chats still issue a $600+ refund.

   **Right fix:** A `PreToolUse` hook checks every `process_refund` call, blocks any amount over $500, and routes the case to `escalate_to_human`.

   **Tempting distractor:** Repeat the limit at the start and end of the system prompt and add few-shot examples of declining large refunds.
2. **Fix the root cause at the right layer.**Incomplete research with healthy subagents means [coordinator decomposition](FAQ-Coordinator-Decomposition.md). Wrong tool picked means tool description. A teammate missing an instruction means a file in the wrong scope.

   **Example:** A report on “renewable energy adoption” covers solar and wind in depth but never mentions hydro, geothermal or storage. Logs show every subagent completed its assigned subtask.

   **Right fix:** Fix the [coordinator's decomposition](FAQ-Coordinator-Decomposition.md) so the subtasks cover the whole topic. The subagents did exactly what they were asked.

   **Tempting distractor:** Tune the search subagent's queries, or give the synthesis agent a bigger context window.
3. **Proportionate beats elaborate.**Prefer the smallest change that removes the cause. A new classifier, [routing model](FAQ-Routing-Classifier.md) or rewrite is usually the distractor.

   **Example:** The agent sometimes calls `get_customer` when a user asks “where is order #48812?” Both tools have one-line descriptions.

   **Right fix:** Rewrite the two tool descriptions so each says when to use it and when to use the other. It is a ten-minute change aimed at the cause.

   **Tempting distractor:** Train a [routing classifier](FAQ-Routing-Classifier.md) that picks the tool before Claude sees the message, or merge both tools into one.
4. **Tool descriptions drive tool selection.**State purpose, inputs, examples, edge cases, and when to use it instead of X. Overlapping descriptions cause misrouting.

   **Example:** `lookup_order` is described only as “Looks up orders.”

   **Right fix:** “Use when the customer gives an order number (e.g. #12345) or asks about the shipping, return or status of a specific order. Input: order_id string. For account details such as email or address, use `get_customer` instead.”

   **Tempting distractor:** Leave the description as is and add five few-shot examples of order questions to the system prompt.
5. **Subagents start blank.**They [inherit no conversation history or memory](FAQ-Subagent-Context.md). Everything they need goes in their prompt, ideally structured (content plus metadata like URL, page, date).

   **Example:** The search subagent found 12 sources. The synthesis subagent then writes a report that cites none of them.

   **Right fix:** The coordinator passes the search results into the synthesis prompt as structured items: claim, source URL, page, publication date.

   **Tempting distractor:** Assume the [synthesis agent can read the coordinator's history](FAQ-Subagent-Context.md), or give it the web-search tool so it can find the sources again.
6. **Fewer, scoped tools per agent.**4–5 role-specific tools beat 18. Agents misuse tools outside their specialty.

   **Example:** One agent holds 18 tools. During writing, the synthesis step keeps launching fresh web searches and drifting off-topic.

   **Right fix:** Split into specialists. Search gets the search tools; synthesis gets only writing tools plus a narrow `verify_fact` for quick checks.

   **Tempting distractor:** Keep all 18 tools on one agent and make every description longer.
7. **Errors must be structured and honest.**Category, retryable flag, what was attempted, partial results. Never “operation failed,” never empty-results-as-success, never crash the workflow on one failure. An empty result is not an error.

   **Example:** The order database times out, `lookup_order` returns `[]`, and the agent tells the customer they have no orders.

   **Right fix:** Return `{"isError": true, "errorCategory": "transient", "isRetryable": true, "message": "Order DB timed out after 5s"}`. The agent retries or tells the customer it is checking, instead of claiming there are no orders.

   **Tempting distractor:** Return `"Operation failed"` for every problem, or keep returning an empty list so the conversation does not break.
8. **[Loops stop on `stop_reason`, not text.](FAQ-stop_reason.md)**Continue on `"tool_use"`, stop on `"end_turn"`. [Iteration caps are a safety net](FAQ-Runaway-Loops.md), not the stopping mechanism.

   **Example:** The loop ends when the reply contains “done”. Claude writes “Done checking the order, now let me look up the refund policy” and the loop stops halfway.

   **Right fix:** Keep looping while `stop_reason == "tool_use"` and stop on `"end_turn"`. Keep a cap of around 25 iterations only as a [guard against runaway loops](FAQ-Runaway-Loops.md).

   **Tempting distractor:** Look for a stricter phrase like “TASK COMPLETE”, or stop after a fixed 10 iterations.
9. **[Independent eyes review better.](FAQ-Independent-Review.md)**A fresh instance without the generator's reasoning catches more than self-review. Big reviews use per-file passes plus one cross-file pass.

   **Example:** In CI, the session that wrote a feature also reviews it and approves a null-pointer bug. On a 20-file PR, the review is deep on the first files and shallow on the rest.

   **Right fix:** Run the review in a separate session with no generator context. Review each file on its own, then do one cross-file pass for data flow between files.

   **Tempting distractor:** Ask the same session to “review your work again carefully”, or review all 20 files in one pass with a larger context window.
10. **[Protect what matters in context.](FAQ-Protecting-Context.md)**Facts (amounts, dates, IDs) go in a persistent case-facts block. Trim tool output. Put the key summary first (lost in the middle). Keep claim-to-source mappings and dates through synthesis.

    **Example:** Forty turns into a support chat, the running summary has turned “$1,247.50 refund on order 88213, promised by Friday” into “customer wants a refund.”

    **Right fix:** Keep a case-facts block (order 88213, $1,247.50, due Friday, status pending) attached to every prompt, outside the summarized history. Trim each order lookup to the five fields that matter.

    **Tempting distractor:** Switch to a model with a larger context window, or summarize the history more often.

## Two more reflexes

Two bonus rules that answer many Domain 4 questions. You'll meet them again in that module.

1. **Explicit criteria beat vague instructions.**Telling Claude *how careful to be* doesn't change its behavior reliably. Telling it *exactly what counts* does. Name the categories to report and the ones to skip, and define severity with concrete examples.

   **Example:** A CI review bot flags so many false positives that developers start ignoring every comment, including real bugs. The prompt says “Be conservative and only report high-confidence issues.”

   **Right fix:** “Report only bugs that cause wrong results, security holes, and comments that contradict the code. Skip style, naming and formatting.” If one category is still noisy, turn it off temporarily to win back trust.

   **Tempting distractor:** “Only report issues you are 95% sure about,” or add “be conservative” at both the start and end of the prompt.

   Task 4.1
2. **Match the API to latency.**The Message Batches API costs about 50% less but can take up to 24 hours, with no speed guarantee and no multi-turn tool use inside a request. Use it only when nobody is waiting for the answer.

   | Use batch | Keep real-time |
   | --- | --- |
   | Overnight reports, weekly audits | Pre-merge PR checks (a developer is waiting) |
   | Nightly test generation | Live customer support (a customer is waiting) |
   | Backfilling 10,000 documents | Any agent that loops through tools |

   **Example:** To cut costs, a manager wants to move both the pre-merge PR checks and the nightly tech-debt report to the Batch API.

   **Right fix:** Batch only the nightly report. Pre-merge checks block a developer, so they stay real-time. If a batch job has a 30-hour SLA, submit at least every 6 hours (6 h wait + 24 h processing).

   **Tempting distractor:** “Batch both, since 50% savings outweighs latency,” or “batch the pre-merge checks because they run more often.”

   Task 4.5

**The one-line version.** Say exactly what you want, not how hard to try; and only batch the work nobody is waiting on.


---

← [Exam Overview](Exam-Overview.md) · [Home](README.md) · [Domain 1 Agentic Architecture](Domain-1-Agentic-Architecture.md) →

Unofficial CCAF study guide · not affiliated with Anthropic · [Interactive site](https://ppotdar.github.io/ccaf-study-course/) · © 2026 Puja Potdar · [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)
