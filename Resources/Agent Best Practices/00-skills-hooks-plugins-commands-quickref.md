---
name: skills-hooks-plugins-commands-quickref
description: Condensed, Claude-Code-specific quick reference for authoring skills, hooks, plugins, and commands - current frontmatter fields, manifest schemas, exit-code semantics, and a decision guide for which of the four to reach for. Pasteable, not a tutorial. Use immediately before creating or editing a SKILL.md, a hook, a plugin, or a slash command, when the current exact field names and defaults are needed rather than general philosophy.
---

# Skills, Hooks, Plugins & Commands - Quick Reference

Companion to `00-agent-best-practices-summary.md`, scoped to the four things you directly author. Full reasoning and checklists live in `03-skills.md`, `11-hooks.md`, and `19-plugins.md` - this file is the fast lookup, not the source.

## 0. Which one do I actually want?

| If... | Reach for | Because |
|:---|:---|:---|
| The model should decide on its own when this applies | A **skill**, default invocation | Description-matching is exactly what skills are for. |
| A human should always type it, never auto-trigger | A **skill** with `disable-model-invocation: true` (or a flat command file) | Removes false-trigger risk for costly/side-effecting workflows. |
| Something must run regardless of what the model decides | A **hook** | Hooks fire on a lifecycle event, not on model choice - see §2. |
| You're distributing several of the above to other people/projects | A **plugin** | One install grants all of it at once - see §3. |

## 1. Skills (and commands - now the same mechanism)

**Commands merged into skills.** `.claude/commands/deploy.md` and `.claude/skills/deploy/SKILL.md` both create `/deploy` and behave identically. A skill folder is the superset - a place for supporting files plus the invocation-control fields below. Default to a skill folder; use a flat command file only when there's nothing to bundle and no invocation control needed.

`name`/`description` are the only fields loaded for every installed skill at startup - get these right or nothing else matters:
- `name`: 1-64 chars, lowercase/numbers/hyphens, must match the folder name exactly.
- `description`: third person, states what + when, real trigger vocabulary. Combined with `when_to_use`, truncated at **1,536 chars** in the listing - lead with the trigger phrase.

Current frontmatter fields:

| Field | Does |
|:---|:---|
| `when_to_use` | Extra trigger phrases, appended to `description`. |
| `disable-model-invocation` | User-only - Claude never auto-triggers. |
| `user-invocable: false` | Claude-only - hidden from the `/` menu. |
| `allowed-tools` / `disallowed-tools` | Pre-approve or remove tools for the invoking turn; clears at next user message. |
| `argument-hint` | Autocomplete hint, e.g. `[issue-number]`. |
| `arguments` | Named positional args bound to `$name`, instead of parsing `$ARGUMENTS`. |
| `model` / `effort` | Override session model/effort while active. |
| `context: fork` + `agent` + `background` | Run in an isolated subagent; `background: false` waits inline. |
| `paths` | Glob patterns limiting auto-activation to matching edited files. |
| `shell` | `bash` (default) or `powershell` for `` !`command` `` execution. |
| `hooks` | Hooks scoped to this skill's own lifecycle - see §2. |
| `license` / `compatibility` / `metadata` | Open-standard fields; Claude Code accepts, doesn't act on them. |

Full checklist: `03-skills.md`.

## 2. Hooks

A hook fires automatically on a lifecycle event - it does not wait for the model to choose to call anything. That's the entire distinction from a skill/tool.

**Exit codes** (the single most common mistake: exit 1 does *not* block):

| Exit code | Effect |
|:---|:---|
| `0` | Success/no decision. Stdout parsed as JSON for structured control; on most events plain stdout goes only to the debug log. |
| `2` | **Blocking.** The only exit code that blocks, and it wins even over a JSON `permissionDecision: "allow"`. |
| anything else | Non-blocking error; the action proceeds. |

Blockable events: `PreToolUse`, `UserPromptSubmit`, `UserPromptExpansion`, `Stop`, `SubagentStop`, `PreModelSwitch`, `WorktreeCreate` (also blocks on *any* non-zero exit). Everything else ignores hook output for blocking purposes even on exit 2 - check the event supports blocking before relying on it.

Matcher: plain string = exact match or `|`/`,`-separated alternatives (`Edit|Write`); anything else is treated as an unanchored regex; empty/omitted matches everything.

Scope by location: `~/.claude/settings.json` (personal, every project) → `.claude/settings.json` (project, shareable) → `.claude/settings.local.json` (project, gitignored) → plugin `hooks/hooks.json` (active while plugin enabled) → skill/subagent frontmatter (active only while that component runs).

Full checklist: `11-hooks.md`.

## 3. Plugins

A plugin bundles any of the above (skills, commands, hooks, subagents, MCP/LSP servers) plus plugin-only extras (output styles, themes, background monitors) into one installable unit. Nothing behaves differently for being in a plugin - a bundled hook still fires exactly per §2. What changes is that **one install grants everything inside at once** - audit each bundled component against its own checklist, never the plugin as a single unit.

Manifest (`.claude-plugin/plugin.json`) - itself optional; only required field if present:

```json
{ "name": "plugin-name" }
```

`name` is kebab-case and namespaces every component (`plugin-name:skill-name`).

Component-path merge rules - easy to get backwards:
- **Adds to** default: `skills` (default `skills/` is always scanned too).
- **Replaces** default: `commands`, `agents`, `workflows`, `outputStyles`, `experimental.themes`, `experimental.monitors` - re-list the default path explicitly if you meant to add, not replace.
- **Merges** by its own logic: `hooks`, `mcpServers`, `lspServers`.

Environment variables: `${CLAUDE_PLUGIN_ROOT}` (install dir, for bundled scripts/binaries), `${CLAUDE_PLUGIN_DATA}` (`~/.claude/plugins/data/{id}/`, survives updates - use for anything generated at runtime), `${CLAUDE_PROJECT_DIR}` (current project root).

Full checklist: `19-plugins.md`.

---
*Companion to `00-agent-best-practices-summary.md`. Full reasoning in `03-skills.md`, `11-hooks.md`, `19-plugins.md`.*
