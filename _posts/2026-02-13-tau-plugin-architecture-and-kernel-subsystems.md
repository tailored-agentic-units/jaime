---
layout: post
title: "TAU Plugin Architecture — Skills, Releases, and Kernel Foundations"
date: 2026-02-13 14:00:00
tags: [kernel, skills, architecture, ci-cd]
category: engineering
excerpt: "Technical overview of the TAU plugin skill architecture, CI/CD release pipeline, and the three kernel foundation subsystems — session, tools, and memory."
---

The TAU plugin ecosystem and kernel both advanced this week. The [tau-marketplace](https://github.com/tailored-agentic-units/tau-marketplace) plugin matured its skill architecture and release infrastructure, while the [kernel](https://github.com/tailored-agentic-units/kernel) completed the initial implementations of three foundation subsystems that will compose into the runtime loop. This post covers the technical details of both.

---

## TAU Plugin Ecosystem

The tau plugin is distributed through the [tau-marketplace](https://github.com/tailored-agentic-units/tau-marketplace) and provides seven skills organized by operational domain. Each skill follows a consistent directory structure:

```
skills/<skill-name>/
├── SKILL.md          # Main instructions and trigger definitions
├── commands/         # Invocable subcommands (actionable workflows)
├── references/       # Pure reference material (loaded contextually)
└── dev-types/        # Development type conventions (optional)
```

### Skills Inventory

| Skill | Sub-commands | Domain |
|-------|-------------|--------|
| **dev-workflow** | `concept`, `plan phase`, `plan objective <issue>`, `task <issue>`, `review`, `release <version>` | Structured development sessions — full lifecycle from concept through release |
| **github-cli** | — (reactive to gh operations) | GitHub CLI operations — issues, PRs, releases, labels, secrets, discussions, sub-issues, issue types |
| **go-patterns** | — (reactive to design questions) | Go design principles — interfaces, error handling, package structure, configuration lifecycle |
| **kernel** | — (reactive to kernel development) | TAU kernel usage — subsystem APIs, protocol execution, provider setup, mock testing patterns |
| **project-management** | — (reactive to project operations) | GitHub Projects v2 — boards, phases, objectives, cross-repo backlog, versioning conventions |
| **skill-creator** | — (reactive to skill authoring) | Skill development — SKILL.md format, frontmatter schema, directory conventions, string substitutions |
| **tau-overview** | — (non-invocable background) | Ecosystem reference — cross-skill integration map, context document structure, session continuity |

### Cross-Skill Integration

The dev-workflow skill acts as the orchestrator. It contextually loads other skills based on the session type:

- **Concept and planning sessions** load project-management and github-cli for project board and issue operations
- **Task execution sessions** load domain-specific skills (e.g., kernel for kernel development, go-patterns for Go code)
- **Release sessions** load github-cli for tag and release operations

This layered loading means a skill only enters context when the session actually needs it, keeping the working context focused.

### Context Documents

The dev-workflow skill produces structured context documents that persist across sessions:

| Path | Purpose |
|------|---------|
| `.claude/context/concepts/<slug>.md` | Architectural concepts and vision documents |
| `.claude/context/guides/<issue>-<slug>.md` | Implementation guides for specific tasks |
| `.claude/context/sessions/<issue>-<slug>.md` | Session summaries with outcomes and decisions |
| `.claude/context/reviews/<date>-<scope>.md` | Project review reports |
| `_project/README.md` | Project overview and phase tracking |
| `_project/phase.md` | Current phase scope and objectives |
| `_project/objective.md` | Current objective with sub-issue breakdown |

Each path has an `.archive/` subdirectory for completed work, maintaining a clean separation between active and historical context.

---

## CI/CD Release Pipeline

The marketplace uses a tag-triggered release workflow that automates GitHub Release creation.

### Tag Convention

```
{prefix}-v{version}
```

The prefix identifies the plugin (e.g., `tau`), and the version follows semver. Example: `tau-v0.0.7`.

### Workflow

The [release workflow](https://github.com/tailored-agentic-units/tau-marketplace/blob/main/.github/workflows/release.yml) triggers on tag push:

1. **Checkout** — full history (`fetch-depth: 0`) for changelog extraction
2. **Extract prefix** — parse the tag to identify the plugin being released
3. **Create GitHub Release** — extract release notes from `CHANGELOG.md` using the tag prefix as a filter, publish automatically

The workflow uses the `taiki-e/create-gh-release-action` for changelog parsing and release creation. The entire pipeline requires no manual steps beyond pushing the tag.

### Versioning

The kernel and marketplace share a versioning convention:

| Type | Format | Example | When |
|------|--------|---------|------|
| **Dev pre-release** | `v<target>-dev.<objective>.<issue>` | `v0.1.0-dev.1.12` | After each PR merge |
| **Phase release** | `v<major>.<minor>.<patch>` | `v0.1.0` | When all phase objectives are complete |

Dev releases track individual issue completions within an objective. The dev-workflow release command handles tagging and integrates with the CI pipeline.

---

## Kernel Foundation Subsystems

Three Level 0 subsystems received their initial implementations this week. All three depend exclusively on `core/protocol` — they are independent of each other and composable by design.

```
Level 0 (Foundation — depend only on core/protocol):
  ├── memory   (Store, FileStore, Cache, Entry)
  ├── tools    (Registry, Handler, Execute, List)
  └── session  (Session interface, in-memory implementation)
```

### Session Interface

**PR [#16](https://github.com/tailored-agentic-units/kernel/pull/16)** | Release `v0.1.0-dev.1.11`

The session subsystem manages conversation history. The core contract:

```go
type Session interface {
    ID() string
    AddMessage(msg protocol.Message)
    Messages() []protocol.Message
    Clear()
}
```

`Messages()` returns a defensive copy — callers cannot mutate session state through the returned slice. The in-memory implementation is concurrent-safe with `sync.RWMutex`.

This work also evolved `protocol.Message` to support multi-turn agentic conversations:

- **`Role`** — typed string enum (`RoleSystem`, `RoleUser`, `RoleAssistant`, `RoleTool`) replacing raw strings
- **`ToolCall`** — struct with `ID`, `Name`, and `Arguments` fields for tool invocations
- **`ToolCallID`** and **`ToolCalls`** fields on `Message` — linking tool results back to their invocations

The protocol evolution means session history natively captures the full agentic conversation flow — user messages, assistant responses, tool calls, and tool results — without lossy serialization.

### Tool Registry

**PR [#19](https://github.com/tailored-agentic-units/kernel/pull/19)** | Release `v0.1.0-dev.1.12`

The tool registry consolidates tool definitions into a single canonical type and provides global tool orchestration.

**Unified type.** `protocol.Tool` is the single source of truth for tool metadata, replacing the separate `agent.Tool` and `providers.ToolDefinition` types that existed previously.

**Global registry.** The registry follows the existing providers registry pattern:

```go
tools.Register(tool, handler)  // Add (rejects duplicates)
tools.Replace(tool, handler)   // Update existing
tools.Get(name)                // Retrieve by name
tools.List()                   // All registered tools
tools.Execute(ctx, name, args) // Run with JSON-encoded arguments
```

**Handler pattern.** Each tool registers a handler function:

```go
type Handler func(ctx context.Context, args json.RawMessage) (Result, error)

type Result struct {
    Content string
    IsError bool  // signals to the LLM that invocation failed
}
```

Built-in tools register via `init()` in sub-packages. External tools extend the catalog by calling `tools.Register()`. The registry is thread-safe with `sync.RWMutex`.

### Memory Store

**PR [#20](https://github.com/tailored-agentic-units/kernel/pull/20)** | Release `v0.1.0-dev.1.13`

The memory subsystem provides a pluggable persistence abstraction with a session-scoped caching layer.

**Store interface.** The persistence contract:

```go
type Store interface {
    List(ctx context.Context) ([]string, error)
    Load(ctx context.Context, keys ...string) ([]Entry, error)
    Save(ctx context.Context, entries ...Entry) error
    Delete(ctx context.Context, keys ...string) error
}
```

**FileStore.** The initial implementation uses the filesystem as a hierarchical key-value namespace. Keys map to file paths, entries carry their content as byte slices. Writes are atomic for durability. Hidden files (dotfiles) are filtered from listings.

**Cache.** The session-scoped cache wraps a Store to provide progressive on-demand loading:

1. **Bootstrap** — load the index (all available keys) and optionally warm specific prefixes
2. **Resolve** — lazy-load requested keys on demand as the session needs them
3. **Flush** — persist dirty entries back to the store

The cache tracks dirty and removed entries, so `Flush()` only writes what actually changed. It's concurrent-safe and separates concerns cleanly: the Store is stateless (every call hits persistence), while the Cache provides session-level performance optimization.

**Namespace convention.** Memory is organized into namespaces:

| Namespace | Content |
|-----------|---------|
| `memory/*` | Kernel-level context |
| `skills/*` | Shareable capability definitions |
| `agents/*` | Agent profile storage |

---

## Key Principles

1. **Protocol-first design** — `core/protocol` defines the canonical types. Subsystems consume them natively rather than defining their own representations.
2. **Foundation orthogonality** — session, tools, and memory are independent. None imports the other. This makes them composable without coupling.
3. **Defensive boundaries** — defensive copies in session, atomic writes in memory, duplicate rejection in the tool registry. Correctness at the interface level.
4. **Registry pattern** — both tools and providers follow the same global registry pattern: register, replace, get, list, execute. One pattern, applied consistently.
5. **Store/Cache separation** — stateless persistence (Store) composed with stateful session optimization (Cache). Each layer has one job.
