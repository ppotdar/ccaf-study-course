# Domain 4: Prompt Engineering & Structured Output

How to get precise, parseable, trustworthy output at production scale. This is prompting for pipelines, not chat.

### 4.1 Explicit criteria

**Concept.** Vague instructions (“be conservative,” “only high-confidence issues”) do not change behavior reliably. Specific categorical criteria do.

**Exam answer.** State exactly what to report (bugs, security, comments that contradict the code) and what to skip (style nits, local conventions). Define each severity level with a concrete code example. If one category produces many false positives, **temporarily disable it** to restore trust while you fix it.

**Trap.** “Tell it to be more careful”; “only report findings above 90% confidence.”

### 4.2 Few-shot examples

**Concept.** Few-shot is the most effective way to get consistent format and to show how ambiguous cases should be handled. Good examples generalize to cases you did not list.

**Exam answer.** 2–4 targeted examples of the **ambiguous** cases, each showing the reasoning for the chosen action and the exact output format. For extraction, show varied document layouts and informal values; this reduces hallucination and empty fields.

**Trap.** Dozens of easy examples; happy-path-only examples.

### 4.3 Structured output with tool use and JSON schema

**Concept.** Define a tool whose `input_schema` is your output schema; the `tool_use` block's `input` is your JSON. A schema removes **syntax** errors (malformed JSON, missing keys). It does **not** prevent **semantic** errors: totals that do not add up, a value in the wrong field, a plausible invented value.

**Exam answer.** Make fields that may be absent **nullable**, since required fields push the model to invent. Give enums an `"other"` value plus a detail string, and `"unclear"` for ambiguous sources. Put normalization rules (ISO dates, numeric currency) in the prompt. Use `tool_choice: "any"` when several schemas exist; force a named tool to run it first.

**Trap.** “Ask for JSON in the prompt and parse it”; making every field required “for completeness.”

### 4.4 Validation, retry and feedback

**Concept.** Retry-with-error-feedback: send the original document, the failed extraction and the **specific** errors (“line items sum to 410, total says 450”).

**Exam answer.** Add self-check fields such as `calculated_total` beside `stated_total` and `conflict_detected`. Add `detected_pattern` to findings so you can analyze which patterns get dismissed. Retries help with format and structure; they do **not** help when the information is missing from the source. Return null and route to review.

**Trap.** Blind retries with the same prompt; retrying until a value appears.

### 4.5 Message Batches API

**Concept.** About 50% cheaper, up to 24 hours to complete, no latency SLA, no multi-turn tool calling inside a request, and `custom_id` to match requests to results.

**Exam answer.** Batch overnight reports, weekly audits, nightly test generation and backfills. Keep anything blocking (pre-merge checks, live support) synchronous. SLA math: with a 30-hour SLA and up to 24 hours of processing, submit at least every 6 hours. On partial failure, resubmit only the failed `custom_id`s with fixes. Tune the prompt on a sample first.

**Trap.** Moving pre-merge checks to batch to save 50%; resubmitting the whole batch.

### 4.6 Independent and multi-pass review

**Concept.** The instance that wrote something [keeps its reasoning in context and tends to agree with itself](FAQ-Independent-Review.md).

**Exam answer.** Use a second, **independent** instance without the generator's context. Split big PR reviews into per-file passes plus one cross-file integration pass. Ask for a confidence value per finding in a verification pass.

**Trap.** “Ask the same session to double-check its work”; one pass over 14 files with a bigger context window.

## Check yourself

This domain's interactive quiz is on the [live site](https://ppotdar.github.io/ccaf-study-course/#d4). The same questions, with answers, are on [Practice Questions](Practice-Questions.md).


---

← [Domain 3 Claude Code Configuration](Domain-3-Claude-Code-Configuration.md) · [Home](README.md) · [Domain 2 Tool Design and MCP](Domain-2-Tool-Design-and-MCP.md) →

Unofficial CCAF study guide · not affiliated with Anthropic · [Interactive site](https://ppotdar.github.io/ccaf-study-course/) · © 2026 Puja Potdar · [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)
