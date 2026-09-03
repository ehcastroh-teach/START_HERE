---
name: agent-build-best-practices
description: Condensed best practices for building, combining, and reviewing agent tools, skills, hooks, harnesses, retrieval systems, sandboxes, and evals. Two parts - per-component rules and cross-stack synergies - meant to be pasted into a session or referenced directly, not read as a tutorial. Use whenever the task is to build or review any component of an agent system, or to check whether several components combined introduce a risk none of them has alone.
---

# Best Practices - Per Component and Across the Stack

 Part 1 covers each component alone. Part 2 covers what only becomes true once components combine - the harder, more frequently skipped half.

## Part 1 - Per-component best practices

Three highest-leverage rules per component, condensed from the full checklists in `01`–`18`. Read the source file for the reasoning; this is the distilled form for quick reference or pasting into a prompt.

**`01` Prompt engineering**
1. State the desired output format explicitly; never assume the model will infer it from context alone.
2. Be explicit about action vs. suggestion - an instruction-following model will describe a change instead of making it unless told directly to act.
3. Ground long-context tasks by asking the model to quote the relevant passage before using it - this anchors accuracy on 20k+ token inputs.

**`02` Tools**
1. Consolidate frequently-chained calls into one tool; don't mirror a REST endpoint one-to-one.
2. Return names, not raw IDs - this single change measurably reduces hallucination.
3. Keep the active tool set to roughly 10–20; accuracy degrades well before any platform ceiling.

**`03` Skills**
1. Write the description in third person, stating what it does *and* when to use it - first-person phrasing measurably hurts discovery.
2. Source content from doing the task by hand or mining real artifacts - never from asking a model to write the skill, which produces advice it already had.
3. Build evaluations *before* writing content: run with no skill, record the gap, write the minimum that closes it.

**`04` Context engineering**
1. Find the smallest set of high-signal tokens that maximizes the outcome - more context is not more signal.
2. Prefer just-in-time agentic search over pre-computed retrieval unless you specifically need the speed.
3. When compacting, maximize recall first, then iterate for precision - an over-eager first pass is safer than a lossy one.

**`05` RAG**
1. Use hybrid search (embeddings + BM25) - semantic search alone misses exact-match queries like error codes.
2. Prepend situating context to each chunk before embedding it - isolated chunks lose the reference that makes them useful.
3. Treat RAG as one retrieval strategy among three (long-context, RAG, just-in-time) - pick based on corpus size and volatility, not habit.

**`06` MCP**
1. Verify token audience on every request - a validly-signed token for a different service must still be rejected.
2. Never pass a client's token through to a backend service unvalidated - exchange or verify it directly.
3. Prefer resources for retrieval where client support allows; fall back to tools only because resource-picker UI support still lags.

**`07` Harness engineering**
1. Give every harness component a stated reason tied to a specific model limitation - and revisit it, since that limitation may not last.
2. Split long-running work into an initializer pass (durable state, feature list, init script) and a coding pass that reads that state before acting.
3. Equip the agent with real verification tools (browser automation, not just unit tests) for anything user-facing - some bugs are invisible from source code alone.

**`08` Running LLMs locally**
1. Run one shared inference server with multiple parallel slots - never one process per concurrent agent instance.
2. Budget KV cache explicitly against real VRAM headroom, verified with `nvidia-smi` under load, not assumed from weight size alone.
3. Pin Nix inputs to an explicit release; re-pin deliberately rather than floating on unstable.

**`09` Agent sandboxing**
1. Treat every allowlisted destination as a capability grant, not a filter - ask what's reachable through it, not just whether it's trusted.
2. Design containment at the environment layer first; a well-written prompt is a probabilistic defense and is never sufficient alone.
3. Match isolation strength to the user's capacity for oversight - too much friction gets disabled, too much trust exposes someone who can't evaluate what they approved.

**`10` Loop engineering**
1. Define what "done" looks like before writing the loop - an iteration cap is a backstop, not a termination design.
2. Rank verification methods by reliability: rules-based first, visual feedback second, LLM-as-judge only when a rule genuinely can't capture the criteria.
3. Detect stuck loops by action deduplication or no-progress tracking, well before the iteration cap is the thing that stops it.

**`11` Hooks**
1. Exit code 2 blocks; exit code 1 does not, on every event that supports blocking - this is the single most common mistake.
2. Scope every matcher as narrowly as the use case allows - a specific tool or server, never a bare `"*"` by default.
3. Review any hook from an external source exactly as you'd review a bundled script - a hook is code with real privileges, not a lightweight exception.

**`12` Sandboxing**
1. Never rely on namespaces alone - pair them with dropped capabilities and seccomp; a root process in a namespace is still root for anything the namespace doesn't cover.
2. Match isolation tier to actual adversarial pressure: a process sandbox for trusted-but-imperfect code, a real VM for genuinely untrusted or multi-tenant workloads.
3. Prefer an existing, battle-tested primitive (Nix's own build sandbox, `systemd-nspawn`) over building custom isolation.

**`13` Evals**
1. Diagnose bottom-up - a failure visible at the top layer usually has its root cause one or two layers down; check there first.
2. Never trust a single run - classify consistency (consistent pass / flaky / consistent failure) across repeated runs before trusting a score.
3. Run both offline (pre-release) and online (continuous) evaluation - offline alone misses model drift entirely.

**`14` Agent orchestration**
1. Give every delegated subtask all four of objective, output format, tool/source guidance, and explicit boundaries - skipping any one causes duplicated work or coverage gaps.
2. Weigh the ~15x token cost against the task's actual shape - reserve decomposition for genuinely breadth-first, independent work, not sequential tasks wearing a parallel disguise.
3. Choose a result-aggregation strategy deliberately (concatenate, rank, reduce, or a dedicated synthesis pass) - don't default to concatenation for outputs that actually need reconciling.

**`15` Scripts**
1. Run the three-question mapping test (same steps every time, rule-checkable branches, single correct output) before scripting - a "no" on any one means the step needs agent judgment, not code.
2. Keep script output a short, structured, single success signal - a verbose dump reintroduces the interpretation cost the script was meant to remove.
3. Don't script a workflow before watching it run correctly and consistently across several real agent turns first - a wrong-but-fast script fails silently, which is worse than a slow agent that can self-correct.

**`16` References**
1. Move content to a reference only if it's needed occasionally mid-task - content needed to decide whether the skill applies at all stays in the `SKILL.md` body.
2. Keep every reference one level deep - an agent's partial-read preview won't reliably follow a pointer to a second reference nested inside the first.
3. Partition by domain into separate, independently-useful files rather than one long reference - a task in one domain should load nothing from another.

**`17` Assets**
1. Test with "would reading this file's content help the model decide what to do, or does it just need to use the file?" - the second case is an asset, always.
2. Never describe an asset's content redundantly in prose - point at its path in one line instead of transcribing what's inside it.
3. Keep any real logic touching an asset in a script, not in the asset itself and not spelled out as `SKILL.md` instructions - the asset stays static.

**`18` Skill pruning and ablation**
1. Settle no-op questions empirically, by running the skill with and without the line - never by reading the sentence and deciding it sounds vague.
2. Run without-tests in genuinely clean context: a session that merely *discussed* the skill is contaminated even if the skill was never invoked.
3. Prune the description harder than the body - it costs context on every turn, not just on the turns the skill fires.

## Part 2 - Synergies: what's only true once components combine

These don't belong to any single file's checklist because no single component produces them. Each one is a rule that only becomes visible when two or more parts of the stack interact.

**S1. An allowlist's safety depends on every component behind it, not just the component that checks it.** `06`'s token-audience validation, `09`'s capability-grant reframing, and `11`'s "a hook is a script with real privileges" are three separate rules until you notice they're describing the same failure shape from three angles: a trusted-looking gate (a hostname, a hook, a server) that doesn't verify the *specific* thing trying to pass through it. Auditing one of these without the other two is auditing a third of the actual boundary.

**S2. The determinism gradient (`03`, `10`) and the reliability tiers (`Relationships` §2) are the same rule at two different scopes.** "Prose for flexible steps, scripts for fragile ones" is a per-skill authoring choice. Applied to the whole stack, it's the reason `11` (hooks) and `12` (sandboxing) exist as separate concepts at all - a hook is the *scripted* version of a must-always rule; a sandbox is the *structural* version of the same instinct, one tier stronger. When a non-negotiable rule keeps getting violated despite being stated clearly in a prompt or skill, that's the signal to move it down a tier - hook first, sandbox boundary if the hook itself can't be trusted to run.

**S3. Retrieval mechanism choice (`05`, `06` resources, `03` skill references) should be made once, explicitly, not per-file by habit.** All three implement "load the smallest high-signal token set, only when needed" - they differ only in where data lives and how volatile it is. Building a RAG pipeline for data that changes hourly, when a skill reference or a live MCP resource would do, is optimizing the wrong axis. Decide by data volatility and corpus size first; let that decision pick the mechanism, not the other way around.

**S4. A harness's containment (`09`) and its coordination (`07`) are independently necessary and neither substitutes for the other.** Git worktrees or separate sandboxes stop concurrent agent instances from colliding on files or privileges. Neither one makes two instances *agree* on approach - a shared task list, lock files, or explicit human assignment is a separate requirement layered on top. Build both; verify both; assuming isolation implies coordination (or vice versa) is a documented failure shape in `07` §5 and `09` §7.

**S5. Evals (`13`) validate every other component, but only as thoroughly as the layer you chose to check.** A tool that passes its Layer 0 schema test and a harness that passes its Layer 2 trajectory test can still fail together at Layer 3 if the failure only emerges from their interaction - a correctly-selected tool called with a subtly wrong argument constructed by an otherwise-sound reasoning trace. Component-level passes are necessary, never sufficient, for system-level confidence. Budget for at least one eval layer above whichever one the immediate task seems to require.

**S6. The security chain (`Relationships` §4) means a single-file review is a partial review.** Vetting a skill (`03`) that connects to an MCP server (`06`) which is itself protected by a hook (`11`) inside a sandbox (`09`/`12`) requires reading all four checklists, because the actual incidents on record are chain failures - a gap in one layer that the others didn't happen to cover, not a single checklist item anyone skipped. When a task touches more than one of these four files, treat that as the signal to read all of them, not just the one that named the task.

**S7. "Prefer the smallest destination" (`00`'s placement rubric) and "diagnose at the lowest layer" (`13`'s bottom-up rule) are the same discipline applied at build time versus debug time.** Building: don't reach for a harness when a skill would do. Debugging: don't assume a system-level fix when the root cause is one prompt. Both mistakes come from looking at the top of the stack first because that's where the symptom or the ambition is visible - the fix in both directions is checking the smallest, lowest unit before escalating.

**S8. Orchestration (`14`) doesn't introduce new failure modes so much as multiply the existing ones.** Every non-negotiable that applies to a single loop, a single sandbox, a single eval, now applies N times over once that loop is running as N parallel subagents. A loop-termination bug (`10`) that would strand one agent strands N agents when parallelized. A sandboxing gap (`09`/`12`) that exposes one instance's blast radius exposes N instances' worth when they run concurrently. An eval that only checks Layer 2 (`13`) for a single agent's trajectory says nothing about whether N agents' trajectories interact safely - that's specifically what `13`'s Layer 3 and `14`'s five-part rubric exist to check, and skipping it is the most common way a system that passed every single-agent test still fails once it's parallelized. Practical consequence: before scaling a working single-agent pattern out to several concurrent instances, re-run its checklist with "times N" appended to every line, not just the lines that mention concurrency by name.

**S9. Scripts (`15`) aren't a ninth destination alongside tools, skills, and hooks - they're what's inside three of the other files whenever the task inside them is deterministic.** A tool's implementation (`02`) is almost always a script wearing an LLM-callable interface. A skill's `scripts/` folder (`03`) is the same technique, invoked by explicit instruction instead of a function call. A hook (`11`) is the same technique again, invoked automatically by a lifecycle event instead of either. This means `15`'s mapping test and authoring rules apply retroactively to content already covered in three other files - a tool's internals, a skill's bundled script, a hook's command all deserve the same "does this pass the three-question mapping test, and does its output stay high-signal" scrutiny, not just newly-written standalone scripts. When any of those three files' checklists mentions a script, that line is a pointer into `15`, not independent guidance.

**S10. Scripts, references, and assets (`15`, `16`, `17`) are one three-way test, not three unrelated folders.** Every piece of bundled content in a skill answers exactly one of three questions: does the model need to *run* it (`scripts/`), *read* it conditionally (`references/`), or *use* it without reading it at all (`assets/`)? Misfiling across this split has a specific, avoidable cost each time: logic written as prose in a reference instead of a script reintroduces the reliability problem `15` exists to solve; a template's content transcribed into `SKILL.md` instead of left as an asset wastes exactly the tokens `17` exists to save; documentation duplicated in the body instead of moved to a reference pays `16`'s discovery-cost tax on every single trigger instead of only when needed. One correct question, asked once per file, prevents all three.

**S11. Ablation (`18`) and evals (`13`) answer opposite questions with the same machinery, and confusing them wastes both.** An eval asks *does this skill work* - does the hook block, does the script flag what it should, does the output meet the rubric. An ablation asks *does this skill's content matter* - would the model have done this anyway. A skill can pass every eval it has while being made largely of no-ops, because evals test the mechanism and never question whether the content driving it was necessary. The practical consequence: a mature skill needs both suites, and they belong in the same `evals/` folder but in **separate subdirectories with separate protocols** - mechanics tests re-run after every script edit, ablation tests re-run after a model upgrade. Running only mechanics gives you a skill that works and slowly fills with filler nobody questions; running only ablation gives you lean content that may not actually execute correctly.

**S12. Every rule in this document is itself subject to `18`'s no-op test, including this one.** The uncomfortable implication of taking pruning seriously: guidance that was load-bearing against one model generation can become a no-op against the next, and this manifesto is guidance. `07`'s "ask what you can stop doing" applies to harness scaffolding; `18` applies it to skill content; the same logic applies to the reference set itself. Concretely - if a future model reliably resolves opaque IDs to names unprompted (`02`), or defaults to private-then-verify when publishing (`09`), those rules become documentation of history rather than instruction. Re-test the highest-leverage rules periodically against the model you're actually running, and treat a rule's continued survival as evidence rather than assumption.

---
*Companion to `00-agent-build-guidance.md` and the 18-file reference set. Part 1 is a condensed digest - read the source file for the full checklist and reasoning. Part 2 exists only here; it isn't duplicated in any single topic file, by construction.*
