## Runnable Kernel CLI

The kernel can now run end-to-end against a real LLM. Issue #15 replaced the `cmd/kernel/main.go` stub with a functional CLI that exercises the full agentic loop — agent, session, memory, and tools all wired together and validated at runtime against Ollama/Qwen3.

Three built-in tools (datetime, read_file, list_directory) give the LLM enough capability to demonstrate multi-step reasoning. When asked to "list the files in cmd/kernel and read main.go," Qwen3 chains list_directory → read_file → summarize across three iterations, each tool call visible in the output. The verbose flag (`-verbose`) writes structured slog events to stderr showing the full loop sequence: memory loading, iteration starts, tool dispatches, and run completion — without cluttering stdout.

A seed memory directory at `cmd/kernel/memory/` with an identity file exercises the full memory → system prompt composition pipeline. Every kernel subsystem is now validated at runtime rather than through mocks alone, following the principle established early in the project: the runtime itself is the integration test.

Several design decisions came out of the session. The iteration model was extended so that `maxIterations=0` means "run until the agent produces a final response" — context cancellation via signal handling provides the safety net. The CLI flag uses -1 as a sentinel to distinguish "not provided" (use config default) from "unlimited" (0). For logging, the kernel accepts a `*slog.Logger` through a `WithLogger` functional option with a discard logger default. This seeds the pattern for per-subsystem structured logging, which is tracked as future work under Objective #4 (Local Development Mode).

This closes all sub-issues under Objective #1 (Kernel Core Loop). Tagged as `v0.1.0-dev.1.15`.

[PR #22](https://github.com/tailored-agentic-units/kernel/pull/22)
