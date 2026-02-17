## Agent Registry

The kernel now has a named agent registry — a key piece of the multi-agent architecture taking shape in Objective #2 (Kernel Interface). Instead of the kernel being hardwired to a single agent, callers can register agents by name (model-aligned names like `qwen3-8b`, `llava-13b`, `gpt-5`) and query their capabilities without instantiating them.

The registry lives in the `agent` package as an instance-owned type, not a global. The kernel creates and owns the instance, same pattern as `session.Session`. This was a deliberate divergence from the tools registry (which is global) — instance ownership gives test isolation and avoids shared state between kernel instances.

Lazy instantiation is the key design choice: `Register()` stores a config, and the actual `Agent` is only created via `agent.New()` on the first `Get()` call. This means you can register a fleet of agents at startup without paying the initialization cost for ones that never get used. Capabilities (which protocols does this agent support — chat, vision, tools, embeddings?) are derived directly from config keys, so querying them never triggers instantiation either.

An interesting discussion during implementation: the initial design used a read-lock fast path with double-checked locking for `Get()`, but we simplified to a single write lock. The TOCTOU race between releasing the read lock and acquiring the write lock would have required the double-check, but for agent access patterns the contention difference is negligible. Simpler code won.

This is the second of three foundational pieces (#23 streaming tools, #24 registry, #25 observer) that feed into #26 (multi-session kernel), where sessions will select agents from the registry by name or capability.

- PR: [#31](https://github.com/tailored-agentic-units/kernel/pull/31)
- Release: `v0.1.0-dev.2.24`
