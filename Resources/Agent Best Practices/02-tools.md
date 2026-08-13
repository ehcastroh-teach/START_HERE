# Tools

Personal reference notes. Sources: [Anthropic — Writing effective tools for agents, with agents](https://www.anthropic.com/engineering/writing-tools-for-agents), [Anthropic — Code execution with MCP](https://www.anthropic.com/engineering/code-execution-with-mcp), [Google — Function calling with the Gemini API](https://ai.google.dev/gemini-api/docs/generate-content/function-calling).

## 1. What a tool is, and when to build one

A **tool** extends what the model can do — I/O, network calls, state mutation, exact arithmetic — as opposed to a **skill**, which teaches the model a procedure for something it can already do (see `03-skills.md`). Tools are a contract between a deterministic system and a non-deterministic caller, and that changes how they should be designed compared to a normal API:

- Build a few tools for **high-impact workflows**, not one tool per endpoint.
- Consolidate frequently-chained calls into a single tool that returns the combined result — this moves orchestration out of the agent's context and into your code. Prefer `schedule_event` over `list_users` + `list_events` + `create_event`.
- Prefer a targeted operation over a dump: `search_contacts` beats `list_contacts`, `search_logs` beats `read_logs`.
- Overlapping or near-duplicate tools distract the agent and degrade selection accuracy. Removing a tool is a legitimate improvement, not a regression.

A common failure Anthropic calls out directly: tools that simply mirror an existing REST endpoint 1:1, regardless of whether that shape actually suits an agent calling it mid-reasoning.

## 2. Naming and namespacing

- Descriptive names, underscores or camelCase, no spaces/periods/dashes.
- Namespace by **service and resource**: `asana_projects_search`, not `search`.
- Prefix vs. suffix namespacing has a measurable, *model-dependent* effect on selection accuracy — don't assume one convention is universally better; check it against your own evaluation.
- Keep the **active tool set small**. Google's guidance: ideally 10–20 contextually relevant tools, with dynamic selection if your total inventory is larger; the hard protocol ceiling is 128 function declarations per request. Accuracy degrades well before you hit that ceiling.

## 3. Input schema design

Write each tool description as if onboarding a new hire — the specialized query formats, niche terminology, and resource relationships you'd otherwise explain verbally.

- Every parameter gets **its own description** with format and an example (an ISO date parameter should show `YYYY-MM-DD` explicitly).
- Name parameters unambiguously: `user_id`, never `user`.
- Use **enums** for any closed set of values — free text invites hallucinated values.
- Minimize required fields; every additional required parameter is another chance for the call to fail before it starts.
- Keep schemas shallow — very large or deeply nested schemas can be rejected outright under constrained/JSON-mode calling.
- Tool descriptions are prompt engineering: small wording refinements produce disproportionate accuracy gains. Iterate them against an evaluation, not once and done.

## 4. Return payload design

- Return **high-signal fields only** — drop low-level technical noise (raw UUIDs, MIME types, thumbnail URLs) unless something downstream actually needs it.
- **Resolve opaque identifiers into names.** This is one of the highest-yield, lowest-effort changes available: converting cryptic IDs into semantically meaningful names measurably improves retrieval precision and reduces hallucination.
- When both a human-readable view *and* a machine ID are genuinely needed (e.g., a search result that feeds a downstream call), expose a `response_format` enum (`concise` / `detailed`) and let the agent choose per-call.
- Paginate, filter, and truncate anything that could return a large payload, with sensible defaults. State the truncation in the response itself and tell the agent what to do next (narrow the query, request a page).
- Response *shape* — JSON vs. XML vs. Markdown — has no universal winner; each performs differently by task. Treat it as an empirical question, not a style preference.

## 5. Error design

Errors are instructions to the agent's next action, not logs for a human.

- State what was wrong **and** what valid input looks like: an invalid-date error should show the expected format and the value actually received.
- List valid options when an unknown enum value or field name is supplied.
- Never return a bare traceback or opaque error code.
- Distinguish retryable failures from permanent ones so the agent doesn't loop on something that can't succeed.

## 6. Gemini-specific mechanics

| Concern | Guidance |
|:---|:---|
| Determinism | Use a low temperature for reliable, repeatable function calls |
| Call mapping | Always return the exact function-call ID in the result so parallel calls map back correctly — results need not arrive in request order |
| Failure detection | Always check `finishReason` to catch turns where a valid call was never actually produced |
| Reasoning continuity | Some models attach opaque "thought signatures" to response parts that must be passed back verbatim on the next call — official SDKs handle this; hand-rolled request construction usually doesn't |
| Mixed responses | A single turn can interleave custom-tool and built-in-tool parts — iterate the parts array, don't assume position |

## 7. Scaling past a handful of tools: code execution

When an agent connects to hundreds or thousands of tools across many servers, loading every definition upfront and routing every intermediate result through context becomes slow and expensive. The fix Anthropic describes: expose tool servers as **code APIs** the agent calls from within a code-execution environment, so tools load on demand, results get filtered *before* they reach the model, and multi-step chains execute as one piece of code instead of N sequential round-trips through context. This is the same principle as tool consolidation in Section 1, generalized to the orchestration layer — see `06-mcp.md` for how MCP specifically implements this.

## 8. Review checklist

- [ ] Each tool has a distinct purpose; no near-duplicates in the active set
- [ ] Names are namespaced consistently (service + resource)
- [ ] Every parameter is unambiguously named, typed, and exemplified
- [ ] Closed sets use enums; required fields are minimal
- [ ] Returns favor names over raw identifiers; noisy fields are dropped
- [ ] Large responses are paginated/filtered/truncated with explicit guidance on what to do next
- [ ] Errors state the fix, not just the failure
- [ ] Descriptions have been iterated against a real evaluation, not written once and shipped

---
*Part of a 12-file reference set: prompt engineering → tools → skills → context engineering → RAG → MCP → harness engineering → running LLMs locally → agent sandboxing → loop engineering → hooks → sandboxing.*
