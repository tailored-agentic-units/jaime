---
layout: post
title: "Kernel Architecture — Overview"
date: 2026-02-07 14:00:00
tags: [kernel, architecture, go, connectrpc]
category: engineering
excerpt: "Architecture overview for the TAU kernel — a unified Go module composing nine subsystems into a reusable agent runtime."
---

The TAU kernel is a single Go module at [`github.com/tailored-agentic-units/kernel`](https://github.com/tailored-agentic-units/kernel) that composes nine subsystems into a reusable agent runtime. This overview covers the subsystem topology, dependency hierarchy, build order, and extension model that define how the kernel is structured and how it grows. For the motivation behind consolidating the libraries into this structure, see the [monorepo migration announcement]({% post_url 2026-02-07-consolidating-the-kernel-monorepo %}).

---

## Vision

The kernel is designed around a single architectural metaphor: a closed-loop processing system — analogous to the Linux kernel — where internal subsystems compose into a complete runtime and external capabilities connect through a well-defined interface boundary.

Each subsystem handles one responsibility. Subsystems compose through explicit Go interfaces. The kernel has zero awareness of what connects to it from the outside. This separation means the same kernel binary works whether extensions run locally on the host or as networked services in containers.

The design philosophy is deliberately simple: composable patterns over complex frameworks, following Anthropic's guidance on [building effective agents](https://docs.anthropic.com/en/docs/build-with-claude/agent-patterns).

---

## Subsystem Topology

Nine subsystems organized by domain, each implemented as a top-level Go package within the module:

| Subsystem | Domain | Status |
|-----------|--------|--------|
| **core** | Foundational types — protocols, messages, responses, configuration | Complete |
| **agent** | LLM client — agent interface, HTTP client, provider abstractions | Complete |
| **orchestrate** | Multi-agent coordination — hub messaging, state graphs, workflows | Complete |
| **memory** | Persistent memory — bootstrap loading, working memory, structured notes | Skeleton |
| **tools** | Tool execution — registry, permissions, built-in tools | Skeleton |
| **session** | Conversation management — history buffer, token tracking, compaction | Skeleton |
| **skills** | Progressive disclosure — SKILL.md discovery, trigger matching, 3-level loading | Skeleton |
| **mcp** | MCP client — transport abstraction, tool discovery, server management | Skeleton |
| **kernel** | Agent runtime — agentic loop, plan mode, hooks, runtime configuration | Skeleton |

Each subsystem is a Go package with explicit imports. The compiler enforces that dependencies only flow in one direction — there are no circular imports, no runtime discovery, and no interface-based indirection to work around dependency constraints.

---

## Dependency Hierarchy

The subsystems are organized into four layers by dependency depth:

- **Layer 0** — `core` has no internal dependencies. It provides the shared type vocabulary (protocols, messages, responses, configuration) that every other subsystem builds on.
- **Layer 1** — `agent`, `memory`, `tools`, and `session` each depend on core and nothing else. These are the foundational subsystems that can be built and tested independently.
- **Layer 2** — `skills` depends on memory for filesystem patterns. `mcp` depends on tools for the ToolExecutor interface. `orchestrate` depends on agent for the Agent interface. Each builds on exactly one Layer 1 subsystem.
- **Layer 3** — `kernel` composes all subsystems above into the agent runtime. It is the only package that depends on the full dependency graph.

The hierarchy is strict and acyclic with a maximum depth of three. Go's import rules enforce this at compile time — a package at a given layer may depend on packages below it but never on packages at the same layer or above. What this means in practice: any subsystem can be tested in isolation with only its declared dependencies, and breaking changes are caught immediately by `go test ./...` across the entire module.

---

## Build Order

The dependency hierarchy translates directly into a phased implementation plan:

| Phase | Subsystems | Description |
|-------|-----------|-------------|
| **0** | core, agent, orchestrate | Migrated from existing libraries with full test coverage. Complete. |
| **1** | memory, tools, session | Foundation subsystems — can be built in parallel with no mutual dependencies. |
| **2** | skills, mcp | Integration subsystems — build on Phase 1, can be built in parallel. |
| **3** | kernel | Composition — the agentic loop that composes all subsystems into a unified runtime. |

Each phase builds exclusively on the previous phase. Phase 1 subsystems depend only on Phase 0. Phase 2 subsystems depend on Phase 0 and Phase 1. This ordering is not a scheduling preference — it reflects actual dependency constraints that the compiler enforces.

---

## Extension Architecture

The kernel is a closed-loop system. It composes its nine subsystems into a complete agent runtime without external dependencies. Extensions — persistence backends, identity and access management, container sandboxes, MCP gateways, observability pipelines, user interfaces — connect through the kernel's interface rather than being compiled into it.

The interface is exposed via [ConnectRPC](https://connectrpc.com/), which provides gRPC-compatible service definitions over standard HTTP. ConnectRPC was chosen because it is Go-native (standard `net/http` handlers), supports multiple protocols simultaneously (gRPC, gRPC-Web, and Connect), and generates type-safe client and server code from Protobuf schemas.

What this means in practice: instead of forking the kernel to add a custom tool provider or a different memory backend, you implement the service interface and connect over the network. The kernel does not need to know — or care — what connects to it. A local development setup might use filesystem persistence and an Ollama instance on the host. A production deployment might use cloud-hosted persistence and Azure AI Foundry. The kernel binary is the same in both cases.

---

## Versioning

The monorepo uses a single version scheme for the entire module:

- **Phase target**: `v<major>.<minor>.<patch>` (e.g., `v0.1.0`)
- **Dev releases**: `v<target>-dev.<objective>.<issue>` (e.g., `v0.1.0-dev.3.7`)

All subsystems share one version tag and one release. Pre-1.0 signals that APIs are still stabilizing. When the kernel reaches 1.0, all subsystems share that stability guarantee — a single version for a single product.

---

## Key Principles

1. **One module, one version** — all subsystems share a single Go module and release tag
2. **Dependency hierarchy enforced by the compiler** — imports only flow downward through the subsystem layers
3. **Closed-loop kernel, open extension boundary** — the runtime is self-contained; external capabilities connect through ConnectRPC service interfaces
4. **Build order reflects dependency order** — implementation phases follow the natural dependency graph from core upward
5. **Packages that change together live together** — repository structure aligns with how the code actually evolves
