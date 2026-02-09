---
layout: post
title: "Consolidating the Kernel Monorepo"
date: 2026-02-07 12:00:00
tags: [kernel, monorepo, go, architecture]
category: progress
excerpt: "Migrating nine independent TAU libraries into a single Go module at github.com/tailored-agentic-units/kernel."
---

The [TAU ecosystem's](https://github.com/tailored-agentic-units) nine independent Go libraries have been consolidated into a single module at [`github.com/tailored-agentic-units/kernel`](https://github.com/tailored-agentic-units/kernel). This is a foundational structural change — one that was straightforward to make now but would have been increasingly painful to defer.

## What Changed

The original TAU architecture distributed the agent runtime across nine separate repositories, each with its own Go module and version:

- **tau-core** — foundational types (protocols, messages, responses, configuration)
- **tau-agent** — LLM client (agent interface, providers, request pipeline)
- **tau-orchestrate** — multi-agent coordination (hub, state, workflows, observer)
- **tau-tools** — tool execution (registry, permissions, built-ins)
- **tau-memory** — persistent memory (bootstrap, working memory, notes)
- **tau-session** — conversation management (history, compaction)
- **tau-skills** — progressive disclosure (SKILL.md discovery, loading)
- **tau-mcp** — MCP client (transport, tool discovery)
- **tau-runtime** — agent runtime (agentic loop, plan mode, hooks)

All nine now live as packages within a single Go module. Import paths changed accordingly:

```
tau-core/pkg/protocol     →  kernel/core/protocol
tau-agent/pkg/agent       →  kernel/agent
tau-orchestrate/pkg/hub   →  kernel/orchestrate/hub
```

The module path is `github.com/tailored-agentic-units/kernel`, and the repository follows the kernel architectural metaphor — a single codebase with internal subsystems, each responsible for a distinct domain.

## Why a Monorepo

The multi-repo strategy introduced friction that was disproportionate to the project's scale. Three categories of overhead made the case for consolidation.

**Version cascade.** A change to tau-core required cascading commits and tags through tau-agent and tau-orchestrate — three or more commits and three or more tags for what was conceptually a single atomic change. Complex release procedures for identifying changed modules, determining version numbers, and tagging in the correct dependency order compounded the effort of every release.

**Diamond dependency risk.** Multiple libraries depending on different versions of tau-core during rapid iteration created version skew. The `go.work` workspace file coordinated the nine modules during development, but it was a workaround for a structural mismatch — the code was split across repository boundaries that did not reflect actual dependency boundaries.

**Organizational overhead.** Nine repositories meant nine CI configurations, nine CLAUDE.md files, nine settings.json files, nine release workflows, and nine label taxonomies to keep in sync. Each new repository required its own infrastructure bootstrapping, and each repository carried operational surface area that had to be maintained independently — even though the libraries changed together.

The Go team's own guidance reinforces the decision: "Packages that change together should live together." The Go standard library is a monorepo precisely because its packages have deep interdependencies and need atomic updates. The TAU kernel has the same property.

## Why Now

The timing was ideal. Only three of nine repositories had any code — tau-core, tau-agent, and tau-orchestrate. The remaining six were skeleton-only. The project is pre-release with zero external consumers, no published tags, and no downstream dependencies. The cost of migration was near zero; the cost of waiting would have compounded with every subsystem implementation and every release tag.

## What This Means in Practice

The monorepo gives the kernel five immediate structural advantages:

- **Atomic commits** — changes that span subsystem boundaries are a single commit, not a coordinated cascade across repositories
- **Single version** — one tag, one release, no dependency ordering to manage
- **One CI pipeline** — `go test ./...` validates the entire kernel in a single run
- **Compiler-enforced boundaries** — Go's import rules prevent inappropriate cross-dependencies between packages, providing the same isolation that separate repositories offered but without the operational overhead
- **Full context** — every contributor and every tool (including Claude Code) has the complete codebase available, with no context switching between repositories

The [Kernel Architecture — Overview]({% post_url 2026-02-07-kernel-architecture-overview %}) covers the subsystem topology, dependency hierarchy, and build order in detail.

The monorepo provides the structural foundation for the next phase of development — defining the subsystem interfaces and implementing the build order that will compose the kernel into a unified agent runtime.
