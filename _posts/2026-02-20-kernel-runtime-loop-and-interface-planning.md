---
layout: post
title: "Kernel Runtime Loop and Interface Planning"
date: 2026-02-20 08:52:44
tags: [kernel, runtime, dev-workflow, architecture]
category: progress
excerpt: "The kernel's agentic loop landed, ran against a real LLM, and immediately prompted the next architectural question — how should external services connect?"
---

Last week closed with the kernel's foundation subsystems — session, tools, and memory — implemented but inert. This week brought them to life. The kernel runtime loop shipped, ran against a real LLM for the first time, and the resulting momentum carried straight into planning the kernel's external interface.

## Kernel Runtime Loop

The core agentic cycle landed in [PR #21](https://github.com/tailored-agentic-units/kernel/pull/21): observe, think, act, repeat. `kernel.New` composes all subsystems from configuration, and `kernel.Run` executes the loop — inject the user prompt into the session, build system content from memory, call the agent with full conversation history, execute any tool calls, and repeat until the agent produces a final response or the iteration budget runs out.

The implementation surfaced several cross-cutting changes. The Agent interface evolved so conversation methods accept full message history instead of a bare prompt string, enabling the kernel to pass session context on each iteration. A duplicate `ToolCall` type was consolidated — format concerns pushed to the deserialization boundary where they belong. Each subsystem gained its own config lifecycle with `DefaultConfig()`, `Merge()`, and config-driven constructors, so `kernel.Config` composes them declaratively.

The package shipped at 93.8% test coverage with 14 tests exercising multi-step tool call chains, iteration limits, context cancellation, memory injection, and error propagation.

## Runtime as the Integration Test

With the runtime loop merged, the next planned task was formal integration tests — mock-based tests exercising the composed system through deterministic agent sequences. Technically sound, but the wrong next step.

We'd established a principle in a previous project: when you have the runtime itself as a validation surface, formal integration tests are redundant at the early stage. In that project, the OpenAPI/Scalar UI served that role. Here, it's a CLI entry point against a real LLM. So [issue #15](https://github.com/tailored-agentic-units/kernel/pull/22) was refactored from integration tests to a runnable kernel CLI with three built-in tools — datetime, read_file, and list_directory.

The result: when asked to "list the files in cmd/kernel and read main.go," Qwen3 chains list_directory → read_file → summarize across three iterations, each tool call visible in the output. Every kernel subsystem is validated at runtime rather than through mocks alone. The CLI becomes both the validation mechanism and the first demonstrable artifact for stakeholders. This closed all sub-issues under Objective #1 (Kernel Core Loop).

## Dev-Workflow Transitions

After completing the kernel's first objective, an immediate workflow gap surfaced: the dev-workflow skill had no mechanism to transition between objectives. Finishing an objective meant manually closing GitHub issues, clearing planning documents, and figuring out what to do with incomplete sub-issues — all outside the structured workflow that handled everything else.

The fix split the monolithic planning command into dedicated `phase` and `objective` sub-commands, each with a transition closeout step. The closeout assesses completion status on GitHub, presents a summary, and asks the user to disposition any incomplete work — carry it forward or move it to the backlog. The core principle: no orphaned issues. Carry-forward items get atomically re-parented to the new objective after it's created, and only then is the old objective closed.

This shipped as [tau-marketplace v0.1.0](https://github.com/tailored-agentic-units/tau-marketplace/releases/tag/tau-v0.1.0) — the first minor release of the dev-workflow skill set.

## Objective #2: Kernel Interface

With the core loop complete, we turned to the kernel's next objective: its HTTP interface, the sole extensibility boundary through which external services connect. What started as "wire up ConnectRPC" became a deeper architecture session that surfaced foundational decisions:

- **Agent registry** — callers shouldn't pass full agent configs. The kernel needs named agent registration with capability awareness, similar to how Claude exposes Opus, Sonnet, and Haiku as named models.
- **Sessions as context boundary** — the kernel isn't one-per-session. It's a singleton managing multiple sessions, with every subsystem integration scoped to a session.
- **Streaming-first** — rather than building around blocking calls and retrofitting streaming later, the kernel loop should use `ToolsStream` from the start. The plumbing already exists in the agent subsystem.
- **Pure HTTP + SSE over ConnectRPC** — following the kernel's own principle of simple, composable patterns over complex frameworks.

The objective decomposed into 6 sub-issues with a clean dependency graph: three independent foundations (streaming tools, agent registry, observer), a central integration piece (multi-session kernel refactor), and the HTTP layer on top.

## First Interface Foundations

Two of those three foundations shipped this week. The streaming tools protocol ([PR #30](https://github.com/tailored-agentic-units/kernel/pull/30)) aligned the kernel's `ToolCall` type directly with the LLM API wire format, eliminating a custom serialization layer that was dropping streaming continuation chunks. The agent registry ([PR #31](https://github.com/tailored-agentic-units/kernel/pull/31)) introduced named agent registration with lazy instantiation — register a fleet of agents at startup without paying the initialization cost for ones that never get used.

## Looking Ahead

The observer subsystem is the remaining foundation piece. Once it's in place, the multi-session kernel refactor can begin — the central integration work that composes streaming, registry, and observer into a kernel that manages concurrent sessions with named agent selection. The HTTP layer sits on top of that. The kernel is starting to look like a service.
