# RootGraph

**A code graph and project memory for your AI coding agent.**

🇷🇺 [Читать по-русски](README.ru.md)

---

## Why RootGraph

Every time an AI coding agent opens a large repository cold, it burns time and tokens rediscovering things a human teammate would already know: where a function is actually used, what the team's naming and error-handling conventions are, which tasks are open, which parts of the codebase are considered legacy, and how the pieces fit together architecturally. Plain full-text search (grep, ripgrep, "find in files") answers "where does this string appear," not "what does this mean" or "what depends on this."

RootGraph replaces that guesswork with a real, queryable model of the project: a graph of code symbols, database tables, documents, components, and tasks, backed by semantic (meaning-based) search over the same content. The agent gets direct answers instead of having to reconstruct context from scratch every session, and the whole team shares the same up-to-date picture instead of everyone's local mental model quietly drifting apart.

It ships as a plugin for Claude Code today, and is designed from the ground up to work the same way with any MCP-compatible agent: console-based, editor-embedded, or a standalone desktop app.

## How it works

A small, thin client runs next to your coding agent and talks to a RootGraph server over HTTPS. The client watches your project, sends structural and semantic updates to the server as you work, and answers the agent's queries by asking the server and translating the result back into something the agent can act on. The server holds the actual graph and vector index; the client itself stores almost nothing beyond a local device key and a short queue of not-yet-sent updates, so a fresh checkout on a new machine is a two-minute setup, not a data migration.

## Key capabilities

**Semantic and structural code search.** Ask for code by what it does, not by what string it contains. Jump straight from a symbol to its definition, its callers, its dependencies, and everything that depends on it, without manually tracing imports across dozens of files.

**Persistent, shared project memory.** Team coding conventions, up-to-date documentation, and architecture diagrams all live together in one graph rather than scattered across a wiki, a ticket tracker, a README that's a year out of date, and everyone's personal notes. Anyone on the team, human or agent, sees the same current state.

**Tasks and technical debt as part of the graph, not a separate tool.** Tasks live next to the code they describe, linked to the exact symbols, files, and components they touch. An agent can claim a task, work it, and close it out with the change already tied back to the task, and anyone can ask the graph what's open, what's stale, and what technical debt is sitting where, instead of maintaining that picture by hand in a ticket tracker that drifts out of sync with the actual code.

**Documents that stay current instead of rotting.** Longer-form documentation is built from composable parts tied to the underlying code and graph, described and assembled on demand rather than written once as a static file and left to go stale. When the code moves, the documentation has a real path to move with it.

**Database and component awareness.** Tables and columns, internal components and the access boundaries between them are first-class nodes in the graph too, so an agent can reason about "what else touches this table" or "is this module allowed to call that one" the same way it reasons about functions.

**Automatic documentation and diagrams.** Architecture diagrams and structured documentation can be generated and kept in sync from the graph itself, instead of being written once by hand and left to rot.

**Fast onboarding for new team members (human or agent).** A newcomer, or a fresh agent session, can ask the graph what a project actually looks like today rather than reading through months of history or waiting for a teammate to explain it in a meeting.

**Built for teams, not just individuals.** Multiple people, and multiple agent sessions, work against the same project graph at once: tasks can be claimed so two people don't silently duplicate the same work, conventions apply the same way for everyone, and every device is enrolled and revoked individually. A single shared server can host many teams and many projects side by side, with each project's data strictly isolated from every other team's, so the same RootGraph server that a small team self-hosts is the same one that powers the multi-tenant SaaS behind the scenes.

**A thin, cross-platform client.** One statically linked binary for macOS, Windows, and Linux. No GPU required, and nothing else to install: no Python, no Ollama, no JVM. Embeddings are computed locally on the CPU, and the server never receives your raw source code, only vectors and coordinates, so semantic search works without your code leaving your machine in readable form.

**Two deployment shapes, one codebase.** Use the shared multi-tenant SaaS to get started in minutes, or run the exact same server on your own infrastructure if your team needs to keep everything inside its own network. Switching between the two later doesn't mean switching products.

**Device-based authentication instead of a shared secret.** New devices are enrolled with a short-lived, one-time invite and a locally generated Ed25519 key pair, the same trust model used by WireGuard, Tailscale, and SSH certificate authorities. Losing a laptop means revoking that one device, not rotating a shared key for the whole project.

## Status

RootGraph is under active development. This repository is the public front door for the project: README, usage examples, and releases will be published here as they become available.

## What's here

* README (this file)
* Examples: coming soon
* Releases: coming soon

## Contact

Built by [Team26](https://github.com/isi1988). Questions and feedback: open an issue on this repository, or reach out directly.

* Email: [s.ivanov@team21.ru](mailto:s.ivanov@team21.ru)
* Telegram: [@s_ivanov88](https://t.me/s_ivanov88)

## License

License terms will be published alongside the first release.
