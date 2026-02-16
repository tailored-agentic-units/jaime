## Foundation review: runtime validation over integration tests

Ran the first formal project review on the TAU kernel after completing four of five sub-issues for Objective #1 (Kernel Core Loop). The foundation subsystems — memory, tools, session — are all implemented, and the kernel runtime loop is merged. But the only way to verify any of it was unit tests. No runnable entry point, nothing to demo.

The remaining task was #15: kernel integration tests. Mock-based tests exercising the composed system through deterministic agent sequences. Technically sound, but the wrong next step. We'd established a principle in a previous project (agent-lab): when you have the runtime itself as a validation surface, formal integration tests are redundant at the early stage. In agent-lab, the OpenAPI/Scalar UI served that role. Here, it's a CLI entry point against a real LLM.

Refactored #15 from integration tests to a runnable kernel CLI with built-in tools (datetime, read_file, list_directory). The kernel runtime loop already works — `kernel.New` + `kernel.Run` composes all subsystems. It just needed a ~100-line `main.go` and a config file pointing at Ollama. That CLI becomes both the validation mechanism and the first demonstrable artifact for stakeholders.

Also caught stale documentation: the standalone `skills/` package was removed back in #13 (consolidated into the memory namespace), but the project docs still referenced it everywhere. Cleaned that up across `_project/README.md`, `CLAUDE.md`, and the dependency hierarchy.

Confirmed Qwen3-8B (Q4_K_M) as the local development model — 5.38 GB on 8 GB VRAM (67.3%), ~69 tok/sec. Docker Compose now auto-loads the model on container startup.

Review report: [`.claude/context/reviews/2026-02-16-kernel.md`](https://github.com/tailored-agentic-units/kernel)
