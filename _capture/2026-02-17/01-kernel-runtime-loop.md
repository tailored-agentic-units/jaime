## Kernel runtime loop: the agentic cycle lands

Implemented the core runtime loop for the TAU kernel — the observe/think/act/repeat cycle that composes agent, tools, session, and memory into a working agentic system. `kernel.New(*Config, ...Option)` follows a cold start pattern: configuration drives all subsystem creation, and functional options (`WithAgent`, `WithSession`, `WithToolExecutor`, `WithMemoryStore`) exist purely for test overrides. `kernel.Run()` executes the loop: inject the user prompt into the session, build system content from memory, call `agent.Tools()` with the full conversation history, execute any tool calls, and repeat until the agent produces a final response or the iteration budget is exhausted.

The implementation surfaced several cross-cutting changes that rippled through the codebase. The Agent interface evolved — conversation methods now accept `[]protocol.Message` instead of a bare `prompt string`, enabling the kernel to pass full session history on each iteration. `response.ToolCall` (a nested struct mirroring the raw LLM wire format) was consolidated into `protocol.ToolCall` (a flat struct) with a custom `UnmarshalJSON` that transparently handles both nested and flat JSON representations. This eliminated a duplicate type and pushed format concerns to the deserialization boundary where they belong.

Each subsystem gained its own config lifecycle: `session.Config` and `memory.Config` with `DefaultConfig()`, `Merge()`, and config-driven constructors, so `kernel.Config` composes them declaratively. A `protocol.InitMessages(role, content)` convenience function replaced 30+ verbose `[]protocol.Message{protocol.NewMessage(...)}` call sites across the codebase.

The kernel package shipped at 93.8% test coverage with 14 tests exercising multi-step tool call chains, iteration limits, context cancellation, memory injection, and error propagation — all driven by a `sequentialAgent` test helper that returns different responses on successive calls, simulating multi-turn agentic conversations without touching a real LLM.

Tagged as `v0.1.0-dev.1.14`. [PR #21](https://github.com/tailored-agentic-units/kernel/pull/21)
