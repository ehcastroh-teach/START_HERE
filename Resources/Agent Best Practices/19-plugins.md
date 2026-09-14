# Plugins

Personal reference notes. Sources: [Claude Code - Plugins](https://code.claude.com/docs/en/plugins.md), [Plugins reference](https://code.claude.com/docs/en/plugins-reference.md), and [Plugin marketplaces](https://code.claude.com/docs/en/plugin-marketplaces.md).

## 1. What a plugin is

A **plugin** is a distribution unit, not a new capability - it's a folder that bundles any combination of the things `01`–`18` already cover (skills, commands, hooks, subagents, MCP servers) plus a few plugin-only extras (LSP servers, output styles, themes, background monitors), installable as one action instead of copying each piece into a project by hand. Nothing a plugin contains behaves differently *because* it's in a plugin - a bundled hook still fires exactly as `11-hooks.md` describes, a bundled skill still loads exactly as `03-skills.md` describes. What changes is the trust boundary: one install grants everything inside at once (§6).

## 2. The manifest is almost entirely optional

`.claude-plugin/plugin.json` is itself optional - Claude Code auto-discovers components in the default directories and derives the plugin's name from the folder name if no manifest exists. If you do write one, only `name` is required:

```json
{ "name": "plugin-name" }
```

`name` must be kebab-case (no spaces, no control or bidirectional-formatting characters) - it's what namespaces every component inside (`plugin-name:skill-name`, `plugin-name:command-name`), which is how two plugins can each ship a skill called `deploy` without colliding.

Everything else is optional metadata (`displayName`, `version`, `description`, `author`, `homepage`, `repository`, `license`, `keywords`, `metadata`, `defaultEnabled`) or a component-path override (§3). Don't add fields you don't need just because the schema allows them - an empty `{ "name": "..." }` is a completely valid, working plugin.

## 3. Component paths: two different merge rules, easy to get backwards

Optional manifest fields can point at non-default locations for each component type, but they don't all merge the same way:

| Behavior | Fields | Practical effect |
|:---|:---|:---|
| **Adds to** the default | `skills` | Default `skills/` is *always* scanned in addition to whatever you point at - you can't accidentally hide your own default skills this way. |
| **Replaces** the default | `commands`, `agents`, `workflows`, `outputStyles`, `experimental.themes`, `experimental.monitors` | Pointing at a custom path stops the default directory from being scanned at all. To keep the default *and* add more, list both explicitly: `"commands": ["./commands/", "./extras/"]`. |
| **Merges** by its own rules | `hooks`, `mcpServers`, `lspServers` | Combines manifest-declared and default-file config per that component's own merge logic, not a simple replace/add. |

The "replaces" column is the one that bites people - adding a `commands` override with the intent of *adding* a second command directory silently drops every command in the default `commands/` folder unless you explicitly re-list it.

All paths must be relative, start with `./` (or `.` for `skills`), and use forward slashes - backslash paths only resolve on Windows.

## 4. Directory layout

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json          # optional manifest
├── skills/<name>/SKILL.md   # skills, one folder each
├── commands/*.md            # flat command files (skills/ is preferred for new plugins)
├── agents/*.md              # subagent definitions
├── hooks/hooks.json         # hook config
├── .mcp.json                # MCP server config
├── .lsp.json                # LSP server config
├── workflows/*.js           # workflow scripts
├── output-styles/*.md       # output style definitions
├── themes/*.json            # color themes (experimental)
├── monitors/monitors.json   # background monitors (experimental)
├── bin/                     # executables added to the Bash tool's PATH
├── scripts/                 # scripts referenced by hooks/commands - see `15` digest in `00`
└── settings.json            # default settings (only `agent` and `subagentStatusLine` keys)
```

`commands/` is the older, flat-file convention (see `03-skills.md` §1 on commands merging into skills) - prefer `skills/<name>/SKILL.md` for anything new so the component gets a home for supporting files and the invocation-control frontmatter.

## 5. Environment variables: pick the one that matches the lifetime you need

| Variable | Resolves to | Use for |
|:---|:---|:---|
| `${CLAUDE_PLUGIN_ROOT}` | The plugin's install directory | Bundled scripts, binaries, static config shipped *with* the plugin |
| `${CLAUDE_PLUGIN_DATA}` | `~/.claude/plugins/data/{id}/`, persists across updates | Anything generated at runtime that must survive the next `version` bump - installed deps, caches, generated code |
| `${CLAUDE_PROJECT_DIR}` | The current project root | Project-local scripts/config, same as outside a plugin context |

Hardcoding an absolute install path instead of `${CLAUDE_PLUGIN_ROOT}` is the plugin-specific version of `15`'s general scripting discipline: it works once, on your machine, and breaks the moment the plugin is reinstalled somewhere else or updated to a new path.

## 6. Security: one install action is the whole `06`/`09`/`11` chain at once

`00`'s synergy S6 already states the general rule: reviewing a skill that talks to an MCP server that's guarded by a hook inside a sandbox requires reading all four checklists, because real incidents are chain failures, not single missed items. A plugin is that exact chain, pre-assembled, activated with one `/plugin install`:

- A bundled **hook** runs with real privileges the instant the plugin is enabled - `11-hooks.md` §7's rule ("anything that can define a hook is exactly as sensitive as a script you'd run directly") applies to every hook a plugin ships, and there is no separate per-hook confirmation step at install time.
- A bundled **MCP server** gets whatever `06-mcp.md` would require you to vet if you added it by hand - token scope, what it's actually connected to - except now it arrived bundled with everything else in the same trust decision.
- A bundled **skill** gets `03-skills.md` §8's full audit bar (read every file, inventory what it reaches, watch for private-data-access + untrusted-content + outbound-channel together) - "it's just one skill inside a plugin" is not a lighter bar than a skill installed on its own.

Practical consequence: audit a plugin by unbundling it mentally into its components first, then apply each component's own checklist - never audit "the plugin" as one unit, because the manifest tells you what's inside but not whether any one piece is safe.

Marketplace provenance is a weak signal, not a substitute for this: `anthropics/claude-plugins-official` is curated, `anthropics/claude-plugins-community` accepts public submissions under review, and a team/private marketplace is whatever the team pointed it at - none of these tiers change what the audit above needs to check, only how likely a problem is to have already been caught by someone else first.

## 7. Precedence and lifecycle

- Project-scoped components (`.claude/agents/`, `.claude/skills/`) override a plugin's same-named component; a plugin's own skills don't override project skills either way - both stay available, namespaced (`plugin-name:skill-name`) to avoid ambiguity.
- `/reload-plugins` picks up changes without restarting the session - useful while developing a plugin locally.
- `claude plugin validate ./plugin-dir` checks the manifest schema before you distribute anything - run it before publishing, not after someone else's install fails.
- `dependencies` in the manifest can pin another plugin by name or `{name, version}` with a semver range - a missing or unsatisfied dependency is a plugin author's problem to declare explicitly rather than discover from a user's bug report.

## 8. Checklist

- [ ] Manifest has only the fields actually needed - `name` is the only required one
- [ ] Any `commands`/`agents`/`workflows`/`outputStyles`/theme/monitor path override explicitly re-lists the default directory if it was meant to be additive, not a replacement
- [ ] Bundled scripts/binaries reference `${CLAUDE_PLUGIN_ROOT}`, not a hardcoded path; anything that must survive an update writes to `${CLAUDE_PLUGIN_DATA}`
- [ ] Every bundled hook, MCP server, and skill has been audited against its own checklist (`11` §7-8, `06`, `03` §8) - not waved through because it's "just part of the plugin"
- [ ] `claude plugin validate` run before distributing
- [ ] Marketplace tier (official / community / private) noted, but not relied on in place of the audit above

---
*Part of a reference set: prompt engineering → tools → skills → context engineering → RAG → MCP → harness engineering → running LLMs locally → agent sandboxing → loop engineering → hooks → sandboxing → evals → agent orchestration → plugins. See also `00-skills-hooks-plugins-commands-quickref.md` for the condensed, field-by-field version of this file.*
