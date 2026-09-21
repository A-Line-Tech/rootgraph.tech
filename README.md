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

## Install

RootGraph ships as one client binary plus a Claude Code plugin. Nothing else has to be installed on a developer machine.

**1. Install the client** (macOS and Linux):

```sh
curl -fsSL https://rootgraph.tech/install.sh | sh
```

On Windows (PowerShell):

```powershell
irm https://rootgraph.tech/install.ps1 | iex
```

The installer picks the right build for your OS and CPU (macOS arm64 and amd64, Linux arm64 and amd64, Windows amd64), verifies its SHA-256 checksum, and puts `rootgraph` into `~/.local/bin` (or `%LOCALAPPDATA%\Programs\rootgraph` on Windows). You can also download a build by hand from the [latest release](https://github.com/A-Line-Tech/rootgraph.tech/releases/latest) and verify it against `checksums.txt`.

**2. Add the plugin to Claude Code:**

```sh
claude plugin marketplace add A-Line-Tech/rootgraph.tech
claude plugin install rootgraph@rootgraph
```

Inside Claude Code the same commands are `/plugin marketplace add A-Line-Tech/rootgraph.tech` and `/plugin install rootgraph@rootgraph`.

**3. Connect a repository.** Create a project in the [web dashboard](https://rootgraph.tech), create an invite on its Devices page, then in the root of your git repository run:

```sh
rootgraph init
```

`rootgraph init` asks for the invite code with hidden input, so the code stays out of your shell history and out of any chat. For scripts pass it on stdin (`printf '%s' "$INVITE_CODE" | rootgraph init --invite -`) or in `ROOTGRAPH_INVITE`. On the first device of a project `init` also builds the index and the base documents (merge everything you need into one branch and update it first: the index is built from the code in the current folder); if the index is already on the server, it only catches up your local changes. The invite is single-use. The device key is generated locally and never leaves your machine.

**4. Check the setup.** `rootgraph doctor` lists what is not ready yet and how to fix it. If the plugin is installed in the middle of a Claude Code session, the hooks work at once and the `rootgraph_*` tools appear with a delay (restart the session if they do not).

### Secrets need your consent

Indexing finds secrets in project files. Their values never enter the index (a placeholder stands in their place), and they go to the encrypted vault only after you agree: in a terminal `rootgraph index` shows a summary without values and asks `[y/N]`. Decide later with `rootgraph vault consent upload` or `decline`, see what was found with `rootgraph vault list`, and mark test values as "not a secret" with `rootgraph secrets allow <path>`. The agent cannot make these decisions for you.

### Git worktrees

`.claude/rootgraph/config.json` belongs to your device and is not committed (`rootgraph init` adds a `.gitignore` for it; commit that file). Agent sessions in linked worktrees take the settings from the main checkout and only read the shared index: changes made in a worktree reach the index after they are merged into the main checkout.

## Status

RootGraph is under active development. This repository is the public front door for the project: the README, the Claude Code plugin marketplace, and the client releases live here, and usage examples will be added as they become available.

## What's here

* README (this file)
* `.claude-plugin/marketplace.json` and `plugins/rootgraph`: the Claude Code plugin marketplace
* [Releases](https://github.com/A-Line-Tech/rootgraph.tech/releases): client builds for macOS, Linux and Windows
* Examples: coming soon

## Contact

Built by [Team26](https://github.com/isi1988). Questions and feedback: open an issue on this repository, or reach out directly.

* Email: [s.ivanov@team21.ru](mailto:s.ivanov@team21.ru)
* Telegram: [@s_ivanov88](https://t.me/s_ivanov88)

## License

License terms will be published alongside the first release.
