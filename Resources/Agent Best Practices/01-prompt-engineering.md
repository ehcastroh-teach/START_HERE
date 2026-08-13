# Prompt Engineering

Personal reference notes. Sources: [Anthropic's prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) (the living reference, updated per model generation), [Anthropic's prompt engineering overview](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/overview), and [Google's Gemini prompt design strategies](https://ai.google.dev/gemini-api/docs/prompting-strategies).

## 1. Scope: what prompt engineering covers, and what it doesn't

**Prompt engineering** is the practice of structuring the words in a single request — instructions, context, examples, format — to reliably get a wanted output. It is the innermost of three concentric layers of agent design:

| Layer | Question it answers | File |
|:---|:---|:---|
| Prompt engineering | What do I say to the model this turn? | this file |
| Context engineering | What surrounds the prompt — tools, history, retrieved data — and how do I curate it? | `04-context-engineering.md` |
| Harness engineering | What software scaffolding runs the loop, calls tools, and persists state across turns? | `07-harness-engineering.md` |

Anthropic frames context engineering explicitly as prompt engineering's natural extension — the same curation instinct applied to the full token budget, not just the system prompt. Keep that framing in mind: techniques below (XML structuring, examples, roles) are prompt-level; they still matter inside a context-engineered or harness-driven system, just as one input among many.

**Before prompt engineering at all**, have three things ready: a clear definition of success for the task, a way to test against that definition, and a first draft prompt to improve. Optimizing without those is optimizing blind. Prompt engineering also isn't always the right lever — sometimes cost or latency problems are better solved by picking a different model than by tuning the prompt further.

## 2. Anthropic's general principles

**Be clear and direct.** Treat Claude like a brilliant new hire with zero context on your norms. State the desired output format and constraints explicitly; use numbered steps when order matters. The golden rule: show the prompt to a colleague with minimal context and ask them to follow it — if they'd be confused, so will the model. If you want "above and beyond" behavior, ask for it explicitly rather than hoping the model infers it.

**Add context, not just instructions.** Explaining *why* a behavior matters lets the model generalize correctly to cases you didn't enumerate, rather than pattern-matching narrowly to your literal wording.

**Use examples (multishot prompting).** The single most reliable lever for output format, tone, and structure. Good examples are:

- **Relevant** — mirror the actual use case
- **Diverse** — cover edge cases, vary enough that the model doesn't lock onto an unintended pattern
- **Structured** — wrapped in `<example>` tags (`<examples>` for a set), separated cleanly from instructions

3–5 examples is the sweet spot for most tasks.

**Structure prompts with XML tags.** When a prompt mixes instructions, context, examples, and variable input, wrap each in its own tag (`<instructions>`, `<context>`, `<input>`) so the model doesn't have to infer boundaries. Nest tags when content has real hierarchy — documents inside `<documents>`, each individually in `<document index="n">`.

**Give the model a role.** A one-sentence system-prompt role (`"You are a helpful coding assistant specializing in Python."`) measurably focuses tone and behavior.

## 3. Long-context prompting (20k+ tokens)

- **Put long documents at the top**, above the query and instructions. Queries placed at the end can improve response quality by as much as 30% in testing, especially for complex multi-document inputs.
- **Wrap each document in `<document>` tags** with `<document_content>` and `<source>` subtags for multi-document inputs.
- **Ground responses in quotes.** For long-document tasks, ask the model to quote the relevant passage before doing the task — this anchors it to the right section instead of skimming the whole document.

## 4. Controlling output format

The most effective lever is stating the positive form, not the negative:

| Instead of | Try |
|:---|:---|
| "Do not use markdown" | "Write in smoothly flowing prose paragraphs." |
| Vague formatting ask | XML format indicators: "Write prose in `<prose>` tags." |
| Hoping the model infers style | Match your *prompt's* formatting to your *desired output's* formatting — models mirror input style |

For persistent control across a whole conversation, a detailed system-prompt block (e.g., an explicit `<avoid_excessive_markdown_and_bullet_points>` instruction) outperforms a one-line ask.

## 5. Thinking and reasoning

Current Claude models default to **adaptive thinking**: the model decides when and how much to think based on an `effort` parameter and query complexity, rather than a manually set token budget. Guidance:

- Prefer general instructions ("think thoroughly") over a hand-written step-by-step plan — the model's own reasoning frequently exceeds what a human would prescribe.
- Multishot examples work with thinking: show `<thinking>` tags inside few-shot examples to teach the reasoning *style*, not just the output.
- Ask for self-checks explicitly: "Before you finish, verify your answer against [criteria]." This reliably catches coding and math errors — except on models tuned to already self-verify well, where an explicit check can cause redundant over-verification. Check current per-model guidance before adding this.
- If thinking triggers more than wanted (common with large system prompts), state a threshold explicitly: reserve it for genuinely multistep problems, respond directly otherwise.

## 6. Agentic and tool-use prompting

- **Be explicit about action vs. suggestion.** Models trained for precise instruction-following will sometimes *describe* a change rather than make it unless told directly to act. A `<default_to_action>` system-prompt block fixes under-triggering; a `<do_not_act_before_instructions>` block fixes over-triggering. Tune in the direction your failures point — not both at once.
- **Parallel tool calls are steerable.** Models run independent tool calls in parallel by default at a high success rate; an explicit `<use_parallel_tool_calls>` instruction can push this toward 100%, or an explicit sequential instruction can suppress it when stability matters more than speed.
- **State reversibility expectations.** Without guidance, an agentic model may take hard-to-reverse actions (force-push, delete, post externally). A system-prompt block naming examples of what warrants confirmation (destructive operations, operations visible to others) meaningfully changes behavior.
- **Subagent orchestration happens natively** in current models when subagent tools are defined and described — no explicit instruction required. The failure mode runs the other way: watch for *overuse*, spawning a subagent for a task a direct tool call would have handled faster. Add explicit guidance on when subagents are and aren't warranted if this happens.
- **Avoid over-engineering and over-testing prompts as a category.** If a model tends to add unrequested abstractions, over-document, or over-verify, name the scope boundary directly ("don't add configurability that wasn't asked for") rather than trying to under-specify your way to minimalism.

These four points are really harness concerns wearing prompt-engineering clothes — the actual mechanism (approval gates, sandboxing) belongs in `07-harness-engineering.md`; what belongs here is only the instruction text that steers behavior within whatever harness exists.

## 7. Gemini comparison

Google's own framing is close to Anthropic's at the principle level, with some Gemini-specific mechanics:

| Practice | Gemini guidance |
|:---|:---|
| Clarity | Be precise and direct; avoid unnecessary or persuasive language — state the goal, don't sell it |
| Structure | Use XML tags or Markdown headers consistently — pick one delimiter style per prompt and don't mix |
| Ambiguity | Explicitly define any ambiguous term or parameter rather than assuming shared meaning |
| System instructions | Set persona and behavioral constraints at the system level, before the conversation starts |
| Temperature | Keep at the default (1.0) unless the task specifically needs deviation — changing it for reasoning tasks risks inconsistent or looping output |
| Debugging a bad prompt | Checklist: typos in task-defining keywords, grammar (run-ons, mismatched subject/verb), punctuation, and undefined domain jargon or acronyms used without definition |
| Iteration | Prompt design is explicitly test-driven and iterative — treat any guideline as a starting point to refine against observed output, not a fixed rule |

The throughline across both vendors: **structure delimiters, explicit success criteria, and iteration against real test cases** matter more than any specific phrasing trick.

## 8. Anti-patterns worth naming

- Vague, high-level guidance that assumes shared context the model doesn't have.
- The opposite failure: hardcoding brittle if-else logic into a prompt to force one exact behavior — fragile, and expensive to maintain as the task drifts.
- Stuffing a laundry list of edge cases into the prompt instead of curating a small, diverse example set — more instructions is not more signal.
- Negative-only formatting instructions ("don't do X") instead of positive format specification.
- Treating "aggressive" language ("CRITICAL: you MUST...") as a universal fix for under-triggering — on more responsive models this causes the opposite failure, over-triggering. Match instruction intensity to the actual failure observed, and dial back over time.

---
*Part of a 12-file reference set: prompt engineering → tools → skills → context engineering → RAG → MCP → harness engineering → running LLMs locally → agent sandboxing → loop engineering → hooks → sandboxing.*
