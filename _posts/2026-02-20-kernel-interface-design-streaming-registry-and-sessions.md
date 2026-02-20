---
layout: post
title: "Kernel Interface Design — Streaming, Registry, and Sessions"
date: 2026-02-20 08:53:44
tags: [kernel, architecture, streaming, agent-registry]
category: engineering
excerpt: "Technical design of the kernel's interface layer — streaming tool calls, named agent registry with lazy instantiation, and the multi-session architecture."
---

The kernel's core loop is complete. The next objective is the kernel's HTTP interface — the sole extensibility boundary through which external services connect. This post covers the architectural decisions driving that interface and the two foundation subsystems already implemented: streaming tools and the agent registry.

---

## From ConnectRPC to Pure HTTP

The initial plan specified ConnectRPC for the kernel's external interface. The proto schema was already scaffolded. But as Objective #2 planning progressed, the mismatch became clear: ConnectRPC is a framework that imposes its own conventions for service definition, code generation, and transport. The kernel's design principle — simple, composable patterns over complex frameworks — argues against adopting a framework at the integration boundary.

The decision: standard `net/http` with JSON request/response for synchronous operations and Server-Sent Events for streaming. The interface should be designed from the kernel architecture, not from a proto schema.

| Concern | ConnectRPC | Pure HTTP + SSE |
|---------|-----------|-----------------|
| Service definition | Proto files + code generation | Go handler functions |
| Streaming | Connect streaming protocol | Standard SSE |
| Client requirements | Generated client stubs | Any HTTP client |
| Dependencies | `connectrpc`, `protobuf`, `buf` toolchain | `net/http` (stdlib) |
| Schema evolution | Proto versioning conventions | JSON field addition |

---

## Streaming Tools Protocol

The kernel's tool calling wire format went through a significant alignment as part of adding `ToolsStream` to the Agent interface ([PR #30](https://github.com/tailored-agentic-units/kernel/pull/30)).

### The Problem

The original `ToolCall` type used a flat internal structure:

```go
type ToolCall struct {
    ID        string
    Name      string
    Arguments string
}
```

Custom `MarshalJSON`/`UnmarshalJSON` methods translated between this format and the nested format that LLM providers actually send:

```json
{
    "id": "call_abc123",
    "type": "function",
    "function": {
        "name": "read_file",
        "arguments": "{\"path\": \"main.go\"}"
    }
}
```

This created friction at every serialization boundary. Worse, the custom unmarshal logic had a bug: streaming continuation chunks carry only partial `function.arguments` without a `function.name`, and these were silently dropped.

### The Decision

Align with the external standard. The new `ToolCall` mirrors the native LLM API format directly:

```go
type ToolFunction struct {
    Name      string `json:"name,omitempty"`
    Arguments string `json:"arguments,omitempty"`
}

type ToolCall struct {
    ID       string       `json:"id,omitempty"`
    Type     string       `json:"type"`
    Function ToolFunction `json:"function"`
}
```

A `NewToolCall(id, name, arguments)` constructor handles the boilerplate. The custom JSON methods were deleted entirely — standard `encoding/json` handles everything. This touched 16 files across the codebase, but the result is cleaner: no translation layer, no serialization bugs, and streaming tool call data flows through naturally.

### ToolsStream

The `ToolsStream` method follows the established `ChatStream`/`VisionStream` pattern:

1. `initMessages` — convert messages to provider format
2. `mergeOptions` — apply model-level defaults
3. Set `stream: true`
4. `ExecuteStream` — return a channel of `StreamingChunk`

The `StreamingChunk` type was extended with a `ToolCalls` field in its `Delta` struct and a `ToolCalls()` accessor mirroring the existing `Content()` pattern. Tool call accumulation — reassembling partial streaming arguments into complete calls — is deferred to the multi-session kernel refactor where the streaming-first loop is built.

---

## Agent Registry

The kernel's agent registry ([PR #31](https://github.com/tailored-agentic-units/kernel/pull/31)) introduces named agent registration with capability-aware lookup.

### Design

Instead of the kernel being hardwired to a single agent, callers register agents by name — model-aligned names like `qwen3-8b`, `llava-13b`, `gpt-5` — and query their capabilities without instantiating them.

```go
registry := agent.NewRegistry()
registry.Register("qwen3-8b", qwenConfig)
registry.Register("llava-13b", llavaConfig)

// Query capabilities without instantiation
caps := registry.Capabilities("llava-13b")
// → [Chat, Vision, Tools]

// First Get() triggers agent.New()
agent, err := registry.Get("qwen3-8b")
```

### Instance Ownership

The registry is an instance-owned type, not a global. The kernel creates and owns the instance, same pattern as `session.Session`. This was a deliberate divergence from the tools registry (which is global) — instance ownership gives test isolation and avoids shared state between kernel instances.

### Lazy Instantiation

`Register()` stores a config. The actual `Agent` is only created via `agent.New()` on the first `Get()` call. This means a fleet of agents can be registered at startup without paying initialization cost for ones that never get used. Capabilities are derived directly from config keys, so querying them never triggers instantiation.

### Concurrency

The initial design used a read-lock fast path with double-checked locking for `Get()`, but we simplified to a single write lock. The TOCTOU race between releasing the read lock and acquiring the write lock would have required the double-check, but for agent access patterns the contention difference is negligible. Simpler code won.

---

## Multi-Session Architecture

The kernel is not one-per-session — it's a singleton managing multiple concurrent sessions. This is the central architectural insight driving Objective #2.

| Concept | Scope | Responsibility |
|---------|-------|---------------|
| **Kernel** | Singleton | Agent registry, observer, tool registry, HTTP interface |
| **Session** | Per-conversation | Message history, memory cache, agent selection, iteration state |

Every subsystem integration is scoped to a session. When a session starts, it selects an agent from the registry by name or capability. Memory is loaded per-session. Tool execution is scoped to session context. This sets the stage for child sessions when subagent orchestration arrives — a parent session spawns a child with a different agent selection, its own message history, and its own memory scope.

---

## Dependency Graph

The objective decomposes into 6 sub-issues with explicit dependencies:

| Issue | Title | Dependencies |
|-------|-------|-------------|
| #23 | Streaming tools protocol | None (foundation) |
| #24 | Agent registry | None (foundation) |
| #25 | Observer | None (foundation) |
| #26 | Multi-session kernel refactor | #23, #24, #25 |
| #27 | HTTP API handlers | #26 |
| #28 | Server entry point | #27 |

Issues #23 and #24 are complete. The observer (#25) is the remaining foundation piece before the multi-session refactor can begin.

---

## Key Principles

- **Align with external standards** — the kernel's internal types should mirror the formats they exchange with, not invent their own. Translation layers accumulate bugs at serialization boundaries.
- **Lazy over eager** — register capabilities at startup, instantiate on demand. The cost of deferred initialization is negligible; the cost of eager initialization scales with the number of registered agents.
- **Instance over global** — registries that the kernel owns should be instance-scoped for test isolation. Global registries are appropriate only for truly static registrations (like tool definitions).
- **Streaming-first** — retrofitting streaming onto a blocking interface is always harder than building streaming from the start. The kernel loop should consume `ToolsStream` natively.
- **Stdlib over frameworks** — at the integration boundary, the simplest correct abstraction is `net/http`. Frameworks earn their complexity budget in application code, not infrastructure code.
