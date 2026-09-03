# MCP (Model Context Protocol)

Personal reference notes. Sources: [modelcontextprotocol.io - Server Features overview](https://modelcontextprotocol.io/specification/2025-06-18/server/) (spec 2025-06-18), [Authorization spec (draft)](https://modelcontextprotocol.io/specification/draft/basic/authorization), [Anthropic - Introducing MCP](https://www.anthropic.com/news/model-context-protocol), [Anthropic - Code execution with MCP](https://www.anthropic.com/engineering/code-execution-with-mcp).

## 1. What MCP is

**MCP** is an open standard, introduced by Anthropic in November 2024, for connecting AI applications to external tools and data sources through standardized server implementations - replacing the M×N problem (every app custom-integrating every tool) with a single protocol every app and server implements once.

Architecture: an MCP **host** (the AI application) runs one or more MCP **clients**, each with a 1:1 connection to an MCP **server**. Servers can run locally (stdio transport) or remotely (HTTP transport).

## 2. The three primitives - and their control model

This is the single most-missed distinction: MCP is not just "tools." The spec defines three primitives with three *different* control models:

| Primitive | Control | Description | Example |
|:---|:---|:---|:---|
| **Tools** | Model-controlled | Executable functions the LLM invokes automatically | API POST requests, file writes |
| **Resources** | Application-controlled | Structured data/content attached and managed by the client | File contents, git history |
| **Prompts** | User-controlled | Pre-defined templates invoked by explicit user choice | Slash commands, menu options |

**Resources are the natural home for retrieval** - they're literally described as pointers to content for grounding a model. In practice, adoption hasn't followed the spec's clean design: resources are the least mature primitive in terms of *client support*. The spec defines them comprehensively (annotations, subscriptions, URI templates), but most clients implement the underlying `resources/list` and `resources/read` operations with no standardized picker UI, and some require manual user selection from a list. **Practical consequence:** most real-world retrieval today is implemented as a **tool** (a `search_docs`-style function) rather than a resource, purely because tool support is universal and resource support isn't - even though resources are the architecturally cleaner fit. See `05-rag.md` for retrieval strategy generally.

**Prompts** solve a different problem: server-side deterministic orchestration. Instead of the model calling four tools and doing arithmetic in its own reasoning, a prompt lets the server execute exact steps (query, aggregate, compute) and hand the model one precomputed dataset with a single formatting instruction - the same "match prescriptiveness to fragility" principle from `03-skills.md`, expressed as a first-class protocol primitive instead of a skill-authoring convention.

## 3. Scaling tool-heavy MCP usage: code execution

At scale, tool-based access has a real cost problem - loading every tool definition upfront and routing every intermediate result through context is slow and token-expensive. **Code execution with MCP** addresses this directly: agents load tool definitions on demand, filter data *before* it reaches the model, and execute multi-step logic in a single code block instead of N sequential context round-trips. Full detail in `02-tools.md` §7 - this is the MCP-specific implementation of the same idea.

## 4. Security: the confused deputy problem

**Status flag:** MCP's authorization mechanism lives in the **draft** revision of the spec - treat implementation mechanics as more likely to shift than the stable tools/resources/prompts primitives above.

The named vulnerability class is the **confused deputy problem**: a legitimately-authorized MCP server gets tricked into using its own elevated privileges on behalf of an unauthorized caller - typically because it applies one static credential across every client it serves, rather than verifying which client is actually asking.

Structurally: the MCP client acts as an OAuth 2.1 client on the user's behalf; the authorization server (which may or may not be the same entity as the MCP server) issues access tokens; the MCP server acts as an OAuth 2.1 *resource server* and must require a valid token on every request.

Three concrete rules to check on any MCP server you build or connect to:

- **Verify token audience on every request.** A token issued for a *different* service must be rejected even if validly signed and unexpired - a confused deputy accepts tokens meant for someone else.
- **Never pass client tokens through to a backend service.** Validate directly against the authorization server, or use a token-exchange step. Passthrough creates a theft opportunity at every intermediary hop.
- **Match redirect URIs exactly.** A registered callback URI should reject any variation (extra query params, different subdomain, different path) rather than fuzzy-matching.

This composes with the skill-security content in `03-skills.md`: a skill that connects to an MCP server inherits that server's authorization posture. Auditing "what does this skill reach" now includes auditing whether the server it talks to correctly scopes who's asking.

## 5. Practical notes

- Adding a local server is typically one command (transport = stdio); remote servers use HTTP and require the OAuth flow above.
- MCP is currently first-class within the Anthropic ecosystem; other vendors (OpenAI's agent SDK, others) have added support, but cross-ecosystem adoption is still uneven - check current support before assuming portability.
- Treat all tool inputs as untrusted, since they originate from the LLM's interpretation of a request, not directly from the user - this is a restatement of the general "treat tool output as data, not instructions" rule from `03-skills.md`, applied to the input side of an MCP call.

---
*Part of a 12-file reference set: prompt engineering → tools → skills → context engineering → RAG → MCP → harness engineering → running LLMs locally → agent sandboxing → loop engineering → hooks → sandboxing.*
