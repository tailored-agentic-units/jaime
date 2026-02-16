## Objective #2 Planning: Kernel Interface

With the kernel's core loop complete (Objective #1 — all 5 sub-issues closed), we turned to the next objective: establishing the kernel's HTTP interface, the sole extensibility boundary through which external services connect.

What started as "wire up ConnectRPC" became a deep architecture session that surfaced foundational decisions the kernel needs to get right before the interface crystallizes. The key realizations:

- **Agent registry**: Callers shouldn't pass full agent configs. The kernel needs named agent registration with capability awareness — model-aligned names like `qwen3-8b` or `gpt-5`, similar to how Claude exposes Opus, Sonnet, and Haiku. This is kernel infrastructure, distinct from the memory `agents/` namespace which holds subagent profile content.

- **Sessions as context boundary**: The kernel isn't one-per-session — it's a singleton managing multiple sessions. Every subsystem integration (memory, tools, agent selection) is scoped to a session. This sets the stage for child sessions when subagent orchestration arrives.

- **Streaming-first**: Rather than building around blocking calls and retrofitting streaming later, the kernel loop should use `ToolsStream` from the start. All the plumbing already exists in the agent subsystem — the streaming chunk parser, provider SSE processing, client `ExecuteStream` — only the `ToolsStream` method on the Agent interface was missing.

- **Observer replaces logger**: The kernel's ad-hoc `slog.Logger` and the orchestrate package's Observer pattern share the same integration points. Unifying them under Observer gives structured event emission that can drive logging, streaming, and metrics from a single source.

- **Pure HTTP + SSE over ConnectRPC**: Following the kernel's own principle — simple, composable patterns over complex frameworks — we dropped ConnectRPC in favor of standard `net/http` with JSON and Server-Sent Events. The initial proto schema was speculative scaffolding; the actual interface should be designed from the kernel architecture, not the other way around.

The objective decomposed into 6 sub-issues with a clean dependency graph: three independent foundations (streaming tools, agent registry, observer), a central integration piece (multi-session kernel refactor), and the HTTP layer on top (API handlers + server entry point). PR merged, all issues linked and tracked on the project board.
