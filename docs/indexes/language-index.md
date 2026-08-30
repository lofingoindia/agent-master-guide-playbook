# Language Index

**Ecosystem discovery date:** 2026-08-30

Language guides will not duplicate generic agent concepts. Each must research runtime behavior that changes production design.

Start with the research-backed [language/runtime decision guide](../languages/choosing-an-agent-runtime-language.md). The individual ecosystem entries below remain discovery queues until their dedicated packets are complete.

| Ecosystem | Required deep-dive concerns | Current state |
|---|---|---|
| [Python](../languages/README.md#python) | `asyncio`, task groups, thread cancellation limits, type/validation ecosystem, workers, packaging, GIL/process choices | Discovery |
| [TypeScript](../languages/README.md#typescript-and-nodejs) | event loop, `AbortSignal`, streams, worker threads/processes, schema libraries, serverless lifetimes | Discovery |
| [Node.js/JavaScript](../languages/README.md#typescript-and-nodejs) | JavaScript-specific runtime/error/type pitfalls beyond TypeScript | Discovery |
| [Go](../languages/README.md#go) | goroutines, `context.Context`, channel ownership, bounded concurrency, HTTP connection reuse | Discovery |
| [Rust](../languages/README.md#rust) | Tokio cancellation, ownership across tasks, error types, async traits, sandbox/process boundaries | Discovery |
| [Java](../languages/README.md#java) | virtual threads/reactive choices, structured concurrency, interruption, executor lifecycle, JVM observability | Discovery |
| [C#/.NET](../languages/README.md#c-and-net) | `Task`, `CancellationToken`, async streams, hosted services, resilience handlers, Azure ecosystem | Discovery |
| Kotlin | coroutines, structured concurrency, cancellation, JVM interoperability, ADK support | Discovery |

Do not infer a winner from framework counts. Selection must include team skills, concurrency model, deployment target, provider SDK quality, observability, durable-runtime support, and the workload's dominant bottleneck.
