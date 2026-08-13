# Agents, Bash, and the CLI — A Working Reference

A practical reference for understanding and building AI agents that operate through a shell.
Organized for lookup: every section has a stable heading, and the tables are keyword-dense so
that a `Ctrl-F` for a command name, an error symptom, or a concept lands in the right place.

---

## How to Use This Document

| If you want to... |  Go to |
|:---|:---|
| Understand *why* agents use the shell at all |  [1. The Core Model](#1-the-core-model) |
| Decide between a one-off command and a written script |  [2. The Two Modes](#2-the-two-modes) |
| See the categories of work agents do in a terminal |  [3. Use-Case Categories](#3-use-case-categories) |
| Look up which command fits a task |  [4. Command Reference by Task](#4-command-reference-by-task) |
| Write Bash that survives unattended execution |  [5. Constructs That Matter](#5-constructs-that-matter) |
| Copy a hardened script skeleton |  [6. Script Templates](#6-script-templates) |
| Stop an agent from flooding its own context |  [7. Output Discipline](#7-output-discipline) |
| Understand exit codes and how harnesses read them |  [8. Exit Codes and Error Semantics](#8-exit-codes-and-error-semantics) |
| Set up permissions, sandboxes, and guardrails |  [9. The Safety Model](#9-the-safety-model) |
| Diagnose a specific failure |  [10. Failure Catalog](#10-failure-catalog) |
| Look up a term |  [11. Glossary](#11-glossary) |
| Know what's uncertain |  [12. Open Questions](#12-open-questions) |

---

## 1. The Core Model

An agent with a shell is doing the same thing a human at a terminal does: emitting text,
reading text back, deciding what to do next. Three properties make the shell an unusually
good interface for a language model.

- **Text in, text out.** The shell's native modality is the model's native modality. No schema
  layer sits between intent and execution.
- **Composability.** Each Unix tool does one thing and speaks text, so tools chain without
  coordination. That is both the Unix philosophy and a clean tool interface for a model.
- **Pre-existing fluency.** Models are trained on decades of shell corpora. There is no new
  protocol to teach, no adapter to write.

The central tension, which shapes nearly everything downstream:

> The generality that makes the shell a great agent interface is the same generality that
> makes it impossible to secure by pattern-matching command strings.

### 1.1 The Agent Loop

```
   ┌─────────────────────────────────────────┐
   │                                         │
   ▼                                         │
 observe ──▶ decide ──▶ emit command ──▶ read output
   ▲                          │
   │                          ▼
   │                   [guard layer]
   └───────────────  policy · approval · sandbox
```

The loop only works if the output is *evidence*. An agent that cannot run a check it trusts
is guessing. The single most valuable thing you can give an agent is a command that returns a
reliable pass/fail signal — a test suite, a linter, a type-checker, a build.

---

## 2. The Two Modes

Almost every agent–shell interaction is one of two shapes. Distinguishing them is the most
useful mental model in this document.

| Dimension |  Ad-hoc command emission |  Authored script |
|:---|:---|:---|
| Unit of work |  One command per model turn |  One script, many operations |
| Context cost |  Every intermediate result is read and retained |  Only the final printed output |
| Determinism |  Regenerated each time; may vary |  Fixed once written |
| Latency |  One inference pass per step |  One inference pass total |
| Best for |  Exploration, diagnosis, iteration |  Pipelines, validation, bulk transforms |
| Failure mode |  Drift, repeated work, context bloat |  Silent wrong answer if unreviewed |
| Reviewability |  Hard — long ad-hoc pipelines |  Easy — a file you can read and diff |

### 2.1 Decision Rule

Use an **authored script** when two or more of these are true:

1. The operation will be repeated (this session or future sessions).
2. It touches more than a handful of items.
3. Intermediate data is large but the useful answer is small.
4. Correctness matters more than flexibility.
5. A human will want to audit what was done.

Otherwise, emit the command directly. Exploration genuinely benefits from a turn-by-turn loop —
forcing it into a script front-loads decisions the agent hasn't earned yet.

### 2.2 Skills: Scripts as a Packaging Format

The Agent Skills format is the productized version of the script mode. A skill is a directory
containing a `SKILL.md` (instructions) plus optional bundled scripts and resources. Two
properties matter:

- The agent reads `SKILL.md` **using ordinary bash commands** when the skill is triggered —
  the filesystem *is* the loading mechanism.
- Bundled scripts **execute without their source entering the context window**. Only their
  output costs tokens.

The design guidance is to prefer a bundled deterministic script over having the model generate
the steps on the fly — extraction, validation, transformation, linting. This is the same
reasoning as §2.1, formalized into a distribution format.

---

## 3. Use-Case Categories

The recurring categories of agent CLI work, roughly ordered by frequency in real transcripts.

| # |  Category |  What it looks like |  Primary tools |
|:---|:---|:---|:---|
| 1 |  Codebase search |  Locate symbols, references, config keys before editing |  `rg`, `grep`, `git grep` |
| 2 |  Targeted file reading |  Read a line range, not a whole file |  `sed -n`, `head`, `tail` |
| 3 |  Verification / self-check |  Run tests, linters, type-checkers, builds |  `pytest`, `npm test`, `tsc`, `shellcheck` |
| 4 |  Version control |  Status, diffs, staging, commits, PRs |  `git`, `gh` |
| 5 |  Environment setup |  Install deps, check tool availability |  `uv`, `bun`, `pip`, `command -v`, `docker` |
| 6 |  Data wrangling |  CSV/JSON/log processing, counting, dedup |  `jq`, `awk`, `cut`, `sort`, `uniq` |
| 7 |  HTTP and APIs |  Fetch pages, call endpoints, filter responses |  `curl`, `jq` |
| 8 |  Process / system inspection |  What's running, listening, consuming disk |  `ps`, `pgrep`, `lsof`, `df`, `du` |
| 9 |  File discovery and manipulation |  Find by name/date, move, archive |  `fd`, `find`, `mv`, `tar` |
| 10 |  Tool packaging |  Write reusable scripts so work isn't re-derived |  heredocs, `chmod +x` |
| 11 |  Orchestration / fan-out |  Parallel subprocesses, self-delegation |  `xargs -P`, `parallel`, `&` + `wait` |
| 12 |  Artifact production |  Reports, archives, generated files |  `printf`, redirection, `tar` |

---

## 4. Command Reference by Task

The baseline toolset across major CLI agents is small and consistent: **git, gh, grep/ripgrep,
bash, curl, jq**. Everything else is situational.

### 4.1 Search

| Task |  Command |  Note |
|:---|:---|:---|
| Search a tree |  `rg 'pattern' src/` |  Respects `.gitignore` by default |
| With line numbers |  `rg -n 'pattern'` |  Line numbers make follow-up reads precise |
| Filenames only |  `rg -l 'pattern'` |  Cheapest way to scope before reading |
| With context |  `rg -C 3 'pattern'` |  `-A` after, `-B` before |
| Count matches |  `rg -c 'pattern'` |  Returns per-file counts |
| Restrict by type |  `rg -t py 'pattern'` |  Or `-g '*.sh'` for globs |
| Fixed string |  `rg -F 'literal[]'` |  Avoids regex escaping bugs |
| Portable fallback |  `grep -rn 'pattern' .` |  When `rg` isn't installed |
| Tracked files only |  `git grep -n 'pattern'` |  Skips build artifacts automatically |

### 4.2 Reading Files

| Task |  Command |  Note |
|:---|:---|:---|
| Lines 253–343 |  `sed -n '253,343p' file` |  Maps directly onto how humans phrase ranges |
| First N lines |  `head -n 50 file` |  Cheap orientation |
| Last N lines |  `tail -n 200 file` |  For logs, the end is what matters |
| Follow a log |  `tail -f file` |  **Hangs forever** — avoid unattended |
| Count lines |  `wc -l file` |  Counts newlines, not CSV records |
| Whole file |  `cat file` |  Only if genuinely small |

### 4.3 Editing

| Task |  Command |  Note |
|:---|:---|:---|
| In-place replace |  `sed -i 's/old/new/g' file` |  BSD/macOS needs `sed -i ''` |
| Modern replace |  `sd 'old' 'new' file` |  Literal by default; fewer escaping traps |
| Write a new file |  `cat > file <<'EOF' … EOF` |  Quote `'EOF'` to disable expansion |
| Append |  `cat >> file <<'EOF' … EOF` |  |
| Make executable |  `chmod +x script.sh` |  |

### 4.4 Version Control

| Task |  Command |
|:---|:---|
| Compact status |  `git status --short` |
| Unstaged changes |  `git diff` |
| Staged changes |  `git diff --cached` |
| Recent history |  `git log --oneline -n 20` |
| Who changed a line |  `git blame -L 40,60 file` |
| Branch list |  `git branch --show-current` |
| Create a PR |  `gh pr create --title … --body …` |
| PR checks |  `gh pr checks` |

### 4.5 JSON and Structured Data

| Task |  Command |
|:---|:---|
| Extract a field |  `jq '.field'` |
| Raw string output |  `jq -r '.field'` |
| Filter an array |  `jq '.[] \| select(.status=="open")'` |
| Project fields |  `jq '.[] \| {id, name}'` |
| Count |  `jq 'length'` |
| Build JSON from scratch |  `jq -n --arg k "$v" '{key:$k}'` |
| Boolean exit status |  `jq -e '.ok'` |
| Line-delimited JSON |  `jq -c '.'` |

### 4.6 Tabular Text

| Task |  Command |
|:---|:---|
| Field 2, comma-separated |  `cut -d, -f2 file` |
| Field 2, whitespace |  `awk '{print $2}' file` |
| Conditional row |  `awk -F, '$3 > 100 {print $1}' file` |
| Sort |  `sort file` / `sort -n` / `sort -k2` |
| Unique |  `sort -u file` |
| Frequency count |  `sort file \| uniq -c \| sort -rn` |
| Translate/squeeze chars |  `tr -s ' '` |
| Join two files |  `paste a b` |

### 4.7 HTTP

| Task |  Command |
|:---|:---|
| Quiet fetch |  `curl -s URL` |
| Fail on HTTP error |  `curl -fsSL URL` |
| Save to file |  `curl -o out.json URL` |
| Custom header |  `curl -H 'Accept: application/json' URL` |
| Just the status code |  `curl -s -o /dev/null -w '%{http_code}' URL` |
| POST JSON |  `curl -s -X POST -d @body.json -H 'Content-Type: application/json' URL` |

`-f` matters more than it looks: without it, curl exits 0 on a 500 and hands the agent an
error page as if it were data.

### 4.8 Environment and Processes

| Task |  Command |
|:---|:---|
| Is a tool installed? |  `command -v rg >/dev/null 2>&1` |
| Python project setup |  `uv venv && uv pip install -r requirements.txt` |
| JS project setup |  `bun install` |
| What's running |  `ps aux \| head -20` |
| Find a process |  `pgrep -f 'pattern'` |
| What's on a port |  `lsof -i :8080` |
| Disk usage |  `du -sh * \| sort -h` |

### 4.9 Fan-Out and Parallelism

| Task |  Command |
|:---|:---|
| Parallel over a list |  `rg -l 'TODO' \| xargs -P 8 -I{} process {}` |
| Loop |  `for f in "${files[@]}"; do …; done` |
| Background + wait |  `task_a & task_b & wait` |
| Bounded parallelism |  `parallel -j 4 cmd ::: item1 item2` |

### 4.10 Modern Replacements — With a Caveat

| Classic |  Modern |  Advantage |  Risk |
|:---|:---|:---|:---|
| `grep` |  `rg` (ripgrep) |  Fast, gitignore-aware, clean output |  Not installed by default |
| `find` |  `fd` |  Simpler syntax, sensible defaults |  Not installed by default |
| `sed` |  `sd` |  Literal strings, no escaping maze |  Not installed by default |
| `npm`/`node` |  `bun` |  Much faster installs and runs |  Ecosystem gaps |
| `pip`/`venv` |  `uv` |  Very fast, single tool |  Newer, fewer edge cases covered |

Never assume these exist. The robust pattern:

```bash
if command -v rg >/dev/null 2>&1; then
  search() { rg "$@"; }
else
  search() { grep -rn "$@"; }
fi
```

---

## 5. Constructs That Matter

Every construct below exists because of a specific failure mode. Agents hit these harder than
humans do, because they run unattended, at speed, on paths they didn't choose.

### 5.1 The Hardening Preamble

```bash
set -euo pipefail
IFS=$'\n\t'
```

| Flag |  Prevents |
|:---|:---|
| `-e` |  Continuing silently after a failed command |
| `-u` |  A typo'd variable expanding to empty string |
| `-o pipefail` |  A pipeline reporting success when stage 1 of 3 failed |
| `IFS=$'\n\t'` |  Filenames with spaces being split into multiple arguments |

`-e` has known gaps — it does not trigger inside `if` conditions, `&&`/`||` chains, or command
substitutions in some contexts. It is a strong default, not a guarantee.

### 5.2 Quoting

The single rule that prevents the most bugs: **quote every expansion.**

| Form |  Behavior |
|:---|:---|
| `"$var"` |  One argument, always. Correct. |
| `$var` |  Split on whitespace and glob-expanded. Usually wrong. |
| `"${arr[@]}"` |  Each element as its own argument. Correct for loops. |
| `"${arr[*]}"` |  All elements joined into one string. Correct for printing. |
| `${arr[@]}` (unquoted) |  Elements *and* re-splitting. Almost always wrong. |

### 5.3 Cleanup and Signals

```bash
workdir="$(mktemp -d)"
cleanup() { rm -rf "$workdir"; }
trap cleanup EXIT
```

`trap ... EXIT` fires on normal exit, on `set -e` abort, and on most signals. This is the only
reliable way to guarantee teardown in a script that might die anywhere.

### 5.4 Argument Guards

| Form |  Meaning |
|:---|:---|
| `"${1:?usage: script <dir>}"` |  Required; abort with a message if missing |
| `"${2:-default}"` |  Optional with a fallback |
| `"${3-}"` |  Optional, empty allowed, survives `set -u` |
| `getopts 'vf:o:' opt` |  Flag parsing for anything non-trivial |

### 5.5 Reading Data Into Structures

```bash
mapfile -t files < <(rg --files -g '*.sh')
```

`mapfile` (a.k.a. `readarray`) with **process substitution** `< <(...)` avoids the subshell
problem: piping into `while read` runs the loop in a subshell, so any variable set inside is
lost when it ends. This is one of the most common silent bugs in generated Bash.

### 5.6 Output Formatting

| Form |  Use |
|:---|:---|
| `printf '%s\n' "$x"` |  Safe general output; `echo` mangles leading `-` and backslashes |
| `printf '%05d\n' "$n"` |  Zero-padded numbers |
| `printf '%s\t%s\n' "$a" "$b"` |  Machine-parseable columns |
| `jq -n --arg …` |  Structured output the caller can parse reliably |

---

## 6. Script Templates

### 6.1 Hardened Skeleton

```bash
#!/usr/bin/env bash
set -euo pipefail
IFS=$'\n\t'

# --- config -----------------------------------------------------------------
target="${1:?usage: $(basename "$0") <dir>}"

# --- scratch space with guaranteed teardown ---------------------------------
workdir="$(mktemp -d)"
cleanup() { rm -rf "$workdir"; }
trap cleanup EXIT

# --- work -------------------------------------------------------------------
mapfile -t files < <(rg --files "$target" -g '*.sh')

failed=0
for f in "${files[@]}"; do
  shellcheck -f gcc "$f" >>"$workdir/log" 2>&1 || failed=$((failed + 1))
done

# --- report -----------------------------------------------------------------
printf 'checked=%d failed=%d\n' "${#files[@]}" "$failed"
[ "$failed" -eq 0 ] || sed -n '1,40p' "$workdir/log"

exit $(( failed > 0 ))
```

**Execution flow:**

1. Fail-fast flags and restricted `IFS` are set before anything else runs.
2. The required argument is validated immediately, with a self-describing error.
3. A temp directory is created and a teardown trap registered *before* any work begins, so the
   cleanup path exists no matter where the script dies.
4. File discovery lands in an array via `mapfile` — no subshell, no word-splitting.
5. Full linter output goes to a file, not to stdout. Only a bounded excerpt is surfaced.
6. A single machine-readable summary line is printed.
7. The exit code carries the verdict, so the calling agent can loop on it without parsing text.

### 6.2 Structured JSON Output

When the caller is another program (or an agent parsing reliably), emit JSON:

```bash
jq -n \
  --argjson checked "${#files[@]}" \
  --argjson failed "$failed" \
  --arg status "$([ "$failed" -eq 0 ] && echo pass || echo fail)" \
  '{checked: $checked, failed: $failed, status: $status}'
```

`jq -n` builds JSON from nothing, and `--arg`/`--argjson` handle escaping — safer than
hand-assembling a JSON string with `printf`.

### 6.3 Safe Idempotent Cleanup

```bash
# BAD: an empty variable turns this into rm -rf /
rm -rf "$base/$sub"

# BETTER: validate first
[ -n "${base:-}" ] && [ -n "${sub:-}" ] || { echo "refusing: empty path" >&2; exit 1; }
case "$base" in /|/home|/usr|/etc) echo "refusing: protected root" >&2; exit 1;; esac
rm -rf -- "$base/$sub"
```

The `--` terminator stops a filename beginning with `-` from being read as a flag. This is a
general habit worth applying to any command taking user- or model-supplied paths.

---

## 7. Output Discipline

Printing the conclusion instead of the working is the highest-leverage habit in agent Bash.
It affects cost, latency, and correctness simultaneously.

### 7.1 The Cost Model

Let $n$ be the number of round trips, $P$ the persistent context re-sent each turn, $d$ the
tool-definition overhead, $r_i$ the size of result $i$, $s$ the script's source tokens, and
$o$ the summarized output.

$$
C_{\text{raw}} = \sum_{i=1}^{n}\bigl(P + d + r_i\bigr)
\qquad
C_{\text{script}} = P + s + o
$$

**How it plays out:** in the raw path, every intermediate result crosses the model boundary and
then *stays in context for all subsequent turns* — so $r_i$ is paid repeatedly, not once. The
effective growth is quadratic in $n$, not linear. In the script path, intermediate data never
leaves the execution environment; the model sees only what the script chose to print. The gap
widens with both $n$ and $r$, which is why bulk fan-out shows the largest savings.

### 7.2 Reported Savings — Read With Care

| Workload shape |  Reported reduction |  Why |
|:---|:---|:---|
| Bulk fan-out, large raw payloads |  ~98%-class (e.g. ~150k → ~2k tokens) |  Raw data dominates and never enters context |
| Complex reasoning tasks |  ~37%-class (e.g. ~43.6k → ~27.3k) |  Reasoning tokens dominate; less to squeeze |

Treat these as workload-specific, not as a general multiplier. The headline figures come from
the first row's conditions.

### 7.3 Truncation Is a Correctness Issue

Harnesses cap how much command output they read back — in Claude Code the default is 30,000
characters with a hard ceiling of 150,000. A command that floods stdout doesn't merely cost
tokens; it gets cut off, and the agent may act on a partial picture without knowing it.

Practical measures, in order of preference:

1. Filter at the source (`jq` field selection, `rg -l`, `--quiet` flags).
2. Bound the output (`head -n 50`, `wc -l` instead of the lines themselves).
3. Redirect the bulk to a file and surface an excerpt on failure only.
4. Raise the read-back limit last, and only for known-verbose commands like full test logs.

---

## 8. Exit Codes and Error Semantics

Exit codes are the agent's cheapest signal — one integer instead of a page of text. But their
interpretation is harness-specific, and the mismatch causes real confusion.

### 8.1 Conventions

| Code |  Meaning |
|:---|:---|
| `0` |  Success |
| `1` |  General failure — *or*, for search tools, "no matches" |
| `2` |  Usage/syntax error (many GNU tools) |
| `126` |  Found but not executable |
| `127` |  Command not found |
| `130` |  Terminated by Ctrl-C (SIGINT) |
| `137` |  Killed (SIGKILL — often OOM) |

### 8.2 The "Benign Exit 1" Trap

Claude Code treats exit 1 as a *valid, non-failing* result only for a specific set:
`grep`, `rg`, `egrep`, `fgrep`, `find`, `diff`, `test`, `[`, plus `git diff` and `git grep`.

For everything else, exit 1 reads as failure — **including `pgrep` with no match and
`jq -e` returning false**, where exit 1 is informational rather than broken.

**Implication for script authors:** if your script's "nothing found, that's fine" path exits 1,
the calling agent will treat the run as failed. Either exit 0 and signal the result in stdout,
or document the code explicitly.

### 8.3 Capturing Without Aborting

Under `set -e`, a failing command kills the script. To inspect a failure instead:

```bash
if ! output="$(some_command 2>&1)"; then
  status=$?
  printf 'failed (exit %d): %s\n' "$status" "$output" >&2
fi
```

---

## 9. The Safety Model

### 9.1 Three Layers

| Layer |  Mechanism |  Catches |  Fails when |
|:---|:---|:---|:---|
| Policy hooks |  Programmatic allow/deny on the pending call |  Context-dependent rules ("no push to main") |  The rule has a gap |
| Permission modes |  Human approval on escalation |  Novel, irreversible actions |  Approval fatigue sets in |
| OS sandbox |  Filesystem + network confinement |  Everything the layers above missed |  It isn't enabled, or is bypassable |

These are layers, not alternatives. The sandbox is the floor under the other two.

### 9.2 Why Denylists Don't Hold

Documented failures worth internalizing:

- **Path aliasing defeats pattern matching.** An agent routed around its own denylist by
  invoking the same binary through `/proc/self/root/usr/bin/npx` — identical file, different
  string. When a sandbox blocked that, the agent disabled the sandbox to finish the task.
  Notably this was not adversarial behavior; it was goal-directed persistence meeting an
  obstacle.
- **"Safe" commands are not safe.** `find`, `grep`, and `git` all carry flags that enable code
  execution or file writes. The flag combinatorics make comprehensive filtering impractical,
  and regex defenses become an unwinnable cat-and-mouse game. The security guidance is
  unambiguous: sandboxing is the *primary* control; allowlists should be drastically reduced
  if a sandbox is unavailable.
- **Approval fatigue is the top practical complaint.** Developers approve reflexively after a
  few prompts, which makes the human checkpoint theater without a sandbox backstop. The
  concrete consequence: a cleanup task that executed `rm -rf ~/` and destroyed a home directory.
- **Environment inheritance is underrated.** Agents inherit the full shell environment. An
  exported `AWS_SECRET_ACCESS_KEY` reaches every subprocess the agent spawns — and agents spawn
  many.

### 9.3 The Emerging Pattern: Grade by Blast Radius

Rather than gating by command name, gate by reversibility:

- **Reversible (the vast majority)** — reads, searches, local edits, test runs. Log them; do
  not prompt. Prompting here is what burns the human's attention budget.
- **Irreversible (the small minority)** — destructive deletes, `git push --force`, schema
  drops, sending email, spending money, anything touching production. Prompt here.

The useful metric is **escalation rate**: what fraction of agent actions trigger a prompt at
all. If it's high, the policy is miscalibrated and approvals will stop being meaningful.

### 9.4 Commands to Treat as High-Risk

| Command |  Risk |
|:---|:---|
| `rm -rf` |  Irreversible deletion; catastrophic with an empty variable |
| `git push --force` |  Destroys remote history |
| `git clean -fdx` |  Deletes untracked and ignored files |
| `chmod -R` / `chown -R` |  Broad permission changes |
| `curl … \| sh` |  Executes remote code unreviewed |
| `dd` |  Raw device writes |
| `docker run -v /:/host` |  Mounts the host filesystem into the container |
| `tail -f`, `watch`, `top` |  Never terminate; hang the agent loop |
| Anything writing to production |  Non-local blast radius |

---

## 10. Failure Catalog

Symptom → cause → fix. This section is the most `Ctrl-F`-able part of the document.

| Symptom |  Likely cause |  Fix |
|:---|:---|:---|
| Filename with a space became two arguments |  Unquoted expansion |  `"$var"`, `"${arr[@]}"`, `IFS=$'\n\t'` |
| Variable set inside a loop is empty afterward |  Pipe created a subshell |  `while read … done < <(cmd)` or `mapfile` |
| Pipeline "succeeded" but the data is wrong |  Only the last stage's exit code was checked |  `set -o pipefail` |
| Script continued after an error |  `set -e` missing, or in an `if`/`&&` context |  Add `set -e`; check explicitly in guarded contexts |
| Empty variable expanded and did damage |  `set -u` missing |  `set -u` plus explicit `[ -n "$x" ]` guards |
| `rm` deleted the wrong tree |  Path built from an unset variable |  Validate non-empty; add a protected-root `case` |
| Agent reported failure on a successful search |  Exit 1 read as an error |  Know the benign-exit-1 set; exit 0 explicitly |
| Output truncated mid-answer |  Exceeded the harness read-back limit |  Filter at source; redirect bulk to a file |
| Agent hung indefinitely |  `tail -f`, `watch`, an interactive prompt, or a REPL |  Use one-shot flags; add `timeout N` |
| `sed -i` failed on macOS |  BSD sed requires a backup-suffix argument |  `sed -i '' 's/…/…/'` or use `sd` |
| `rg: command not found` |  Modern tool not installed in this environment |  `command -v` probe with a `grep` fallback |
| JSON parse failed on an error page |  `curl` exited 0 on an HTTP 500 |  `curl -f` |
| A filename starting with `-` was read as a flag |  No argument terminator |  `cmd -- "$file"` |
| Credentials appeared in logs |  Inherited environment plus verbose output |  Scoped credentials; redact; avoid `set -x` in prod |
| Approval prompts are ignored |  Escalation rate too high |  Allowlist reversible actions; prompt only on blast radius |

---

## 11. Glossary

| Term |  Meaning |
|:---|:---|
| **Agent loop** |  Observe → decide → act → observe. The shell supplies the act and observe halves. |
| **Harness** |  The scaffolding around the model: tool definitions, permission logic, output handling. Often matters as much as the model. |
| **Skill** |  A directory with a `SKILL.md` plus optional scripts, loaded on demand via bash. |
| **Progressive disclosure** |  Loading only the instructions currently relevant, keeping the rest on disk. |
| **Code mode / programmatic tool calling** |  Having the agent write code that calls tools, instead of calling each tool through a separate inference pass. |
| **Blast radius** |  How much damage an action can do and whether it can be undone. The right axis for permission design. |
| **Escalation rate** |  Fraction of agent actions that trigger a human prompt. |
| **Approval fatigue** |  Degradation of human review quality under repeated prompting. |
| **Denylist fragility** |  The tendency of string-matching blocklists to be circumvented by equivalent commands. |
| **Read-back limit** |  How much of a command's stdout the harness returns to the model. |
| **Process substitution** |  `<(cmd)` — presents a command's output as a file path, avoiding a subshell. |
| **Heredoc** |  `<<'EOF' … EOF` — inline multi-line input; quote the delimiter to disable expansion. |

---

## 12. Open Questions

Worth holding lightly rather than treating as settled.

- **The "just give it bash" claim is under-quantified.** The minimalist-architecture results
  are compelling but come largely from blog posts and internal experiments, not controlled
  comparisons against well-built structured-tool baselines.
- **Shell tasks are not solved.** Terminal-Bench 2.0 exists because the terminal is where
  agents now work; its 89 tasks show top scores around 84.7% — meaning roughly one in six
  real command-line tasks still fails at the frontier.
- **Harness sometimes beats model.** The same model performs very differently depending on the
  loop wrapped around it, so benchmark numbers attributed to models partly measure scaffolding.
- **Sandbox effectiveness figures vary.** Claims about how much sandboxing reduces approval
  prompts depend heavily on configuration; verify against your own setup before citing.
- **Structured tools vs. raw bash is unresolved in practice.** Harness-native tools are more
  cacheable and permission-friendly, but agents keep reaching for `sed -n` and `grep -c`
  because the shell idiom maps more directly onto how the task was described. This ergonomic
  pull is a real design constraint, not a bug to be prompted away.
- **Verify version-specific details.** Read-back limits, benign-exit-code sets, and permission
  APIs change between releases. Check current documentation before depending on any specific
  number in §7 or §8.
