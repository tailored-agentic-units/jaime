---
layout: post
title: "Skills Onboarding and Kernel Foundations"
date: 2026-02-13 12:00:00
tags: [skills, marketplace, kernel, onboarding]
category: progress
excerpt: "Balancing team enablement through the TAU skills marketplace with initial kernel foundation subsystems — session, tools, and memory."
---

This week split between two priorities: cleaning up and beginning the onboarding process for the [TAU skills marketplace](https://github.com/tailored-agentic-units/tau-marketplace) and putting the initial kernel foundation subsystems in place. The week was heavier on meetings and team engagements than on raw development output, but the infrastructure getting put into place will pay dividends moving forward. The pace of implementation will increase as we settle into our battle rhythm.

## TAU Skills Marketplace

The TAU skills marketplace is a centralized distribution point for Claude Code skills — reusable, structured workflows that standardize how the team interacts with Claude Code across all TAU repositories. This week I focused on cleaning up the marketplace plugin and onboarding team members and leadership to its usage.

> For details on Claude Skills, see: [Extend Claude with skills](https://code.claude.com/docs/en/skills).

### What the Plugin Provides

The [tau plugin](https://github.com/tailored-agentic-units/tau-marketplace/tree/main/plugins/tau) ships seven skills, each covering a distinct operational domain:

| Skill | Purpose |
|-------|---------|
| **dev-workflow** | Structured development sessions — concept development, phase/objective planning, task execution, project review, and releases |
| **github-cli** | GitHub operations via `gh` — issues, PRs, releases, labels, secrets, discussions, sub-issues, and issue types |
| **go-patterns** | Go design principles — interface design, error handling, package structure, configuration lifecycle |
| **kernel** | TAU kernel usage — subsystem APIs, protocol execution, provider setup, mock testing |
| **project-management** | GitHub Projects v2 — boards, phases, objectives, cross-repo backlog management |
| **skill-creator** | Skill authoring — SKILL.md format, frontmatter, directory conventions |
| **tau-overview** | Ecosystem reference — cross-skill integration, context documents, session continuity |

The dev-workflow skill is the orchestrator. It drives the full development lifecycle — from concept through release — and loads project-management and github-cli contextually when a session needs them. What this means in practice: a developer starts a session with `/dev-workflow task 42`, and the skill loads the relevant issue context, creates an implementation guide, and structures the work into a branch-per-issue, PR-per-task workflow.

### Marketplace Changes This Week

The plugin went through several rounds of refinement:

- **Restructured dev-workflow** — moved actionable subcommands from `references/` to `commands/`, separating invocable workflows from pure reference material
- **Introduced issue type conventions** — standardized Bug, Task, and Objective as organization-level issue types across github-cli, project-management, and dev-workflow
- **Added `_project/` convention** — established `_project/README.md`, `_project/phase.md`, and `_project/objective.md` as the standard project documentation structure
- **Refined the development pipeline** — split validation into separate testing and validation phases, strengthened implementation guide conventions

### CI/CD Release Cycle

The marketplace uses a tag-triggered release pipeline. When a tag matching the pattern `{prefix}-v{version}` is pushed (e.g., `tau-v0.0.7`), the [release workflow](https://github.com/tailored-agentic-units/tau-marketplace/blob/main/.github/workflows/release.yml) automatically creates a GitHub Release with notes extracted from the CHANGELOG. The process is fully automated — tag and push, and the release appears on GitHub within seconds.

### Why This Matters

Standardizing how the team uses Claude Code is a force multiplier. Instead of each developer learning their own patterns for issue management, project planning, and development workflow, the plugin provides a shared vocabulary and shared processes. Every project that installs the tau plugin gets the same structured workflows, the same issue conventions, and the same release process. The investment in cleaning up and onboarding pays dividends every time someone starts a new development session.

## Kernel Foundation Subsystems

On the kernel side, three foundation subsystems received their initial implementations this week — each addressing a core capability that the kernel runtime loop will compose.

### Session Interface

The [session subsystem](https://github.com/tailored-agentic-units/kernel/pull/16) establishes conversation history management as a first-class kernel primitive. It defines a `Session` interface for adding messages, retrieving history, and clearing state — with a concurrent-safe in-memory implementation that uses defensive copies to prevent external mutation. This work also evolved `protocol.Message` with a `Role` enum and `ToolCall` struct, laying the groundwork for multi-turn agentic conversations.

### Tool Registry

The [tool registry](https://github.com/tailored-agentic-units/kernel/pull/19) unifies tool definitions into a single canonical `protocol.Tool` type and provides a global registry for tool registration and execution. Tools register with a handler function, and the registry provides lookup, listing, and execution by name. The pattern follows the existing providers registry — thread-safe, duplicate-aware, and extensible through `init()` functions in sub-packages.

### Memory Store

The [memory store](https://github.com/tailored-agentic-units/kernel/pull/20) implements a pluggable persistence abstraction with a `Store` interface for listing, loading, saving, and deleting entries. The filesystem-backed `FileStore` provides the initial implementation with atomic writes and hierarchical key-value namespaces. On top of the store sits a session-scoped `Cache` that provides progressive on-demand loading — it knows what keys are available without loading their content, resolving entries lazily as the session needs them.

### What These Enable

All three subsystems sit at Level 0 in the kernel's dependency hierarchy — they depend only on `core/protocol` and nothing else. They are independent and composable, which means the [kernel runtime loop](https://github.com/tailored-agentic-units/kernel/issues/14) can compose them together without any of them needing to know about the others. That's the next step.

## Looking Ahead

The foundation is in place. The next objective tasks are the kernel runtime loop and integration tests — composing session, tools, and memory into the agentic loop that will drive the kernel. Once we get this infrastructure in place, we'll be able to see the kernel start to come to life.
