## Streaming Tools Protocol

The kernel's tool calling wire format got a significant refactoring today as part of adding streaming tool call support ([PR #30](https://github.com/tailored-agentic-units/kernel/pull/30), issue #23). What started as "add `ToolsStream` to the Agent interface" turned into a deeper alignment decision: should the kernel maintain its own canonical ToolCall format, or align directly with the LLM API wire format?

The original `ToolCall` type used a flat structure (`{ID, Name, Arguments}`) with custom `MarshalJSON`/`UnmarshalJSON` methods to translate between the internal format and the nested format that providers actually send (`{id, type, function: {name, arguments}}`). This created friction — every serialization boundary required translation, and the custom unmarshal logic had a bug where streaming continuation chunks (carrying only partial `function.arguments` without a `function.name`) would be silently dropped.

The decision was to align with the external standard. The new `ToolCall` type mirrors the native LLM API format directly, with a `ToolFunction` named type for the nested structure and a `NewToolCall` constructor that handles the boilerplate. The custom JSON methods were deleted entirely — standard `encoding/json` handles everything. This touched 16 files across the codebase (core types, agent, mock, kernel, session, CLI), but the result is cleaner: no translation layer, no serialization bugs, and streaming tool call data flows through naturally.

The `ToolsStream` method itself follows the established `ChatStream`/`VisionStream` pattern — same `initMessages`, `mergeOptions`, `stream: true`, `ExecuteStream` pipeline. The `StreamingChunk` type was extended with a `ToolCalls` field in its Delta struct and a `ToolCalls()` accessor mirroring the existing `Content()` pattern. Tool call accumulation (reassembling partial streaming arguments into complete calls) is deferred to issue #26 where the streaming-first kernel loop is built.

Tagged as `v0.1.0-dev.2.23`.
