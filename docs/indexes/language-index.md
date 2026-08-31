# Language Index

**Last updated:** 2026-08-31

Language guides will not duplicate generic agent concepts. Each must research runtime behavior that changes production design.

Start with the research-backed [language/runtime decision guide](../languages/choosing-an-agent-runtime-language.md). If Go is in scope, use the [Go/Python/TypeScript comparison](../comparisons/go-vs-python-vs-typescript-node-agent-runtimes.md); for a Python-versus-TypeScript-only decision, use the [focused comparison](../comparisons/python-vs-typescript-node-agent-runtimes.md). Rust, the JVM, and .NET now have dedicated production playbooks.

| Ecosystem | Required deep-dive concerns | Current state |
|---|---|---|
| [Python](../languages/python/README.md) | `asyncio`, task groups, thread cancellation limits, type/validation ecosystem, workers, packaging, GIL/process/subinterpreter choices | Deep 14-guide production playbook plus [overview](../languages/python-agent-runtimes.md) |
| [TypeScript](../languages/typescript/README.md) | Compiler/module contracts, erased types, runtime schemas, state machines, provider adapters, packages, artifacts, compatibility, and migration | Deep 11-guide language playbook plus [Node runtime overview](../languages/typescript-node-agent-runtimes.md) |
| [Node.js/JavaScript](../languages/nodejs/README.md) | Event-loop and libuv ownership, abort trees, HTTP/Undici, process failure, streams, workers, permissions, queues, memory, diagnostics, and deployment targets | Deep 15-guide production playbook plus [overview](../languages/typescript-node-agent-runtimes.md) |
| [Go](../languages/go/README.md) | goroutines, context/deadline ownership, byte-aware backpressure, HTTP/process safety, schemas/effects, durable workers, resources, diagnostics, testing, supply chain, deployment, and framework maturity | Deep 14-guide production playbook plus [overview](../languages/go-agent-runtimes.md) |
| [Rust](../languages/rust/README.md) | Tokio cancellation and task ownership, streaming/backpressure, schemas, effects, durability, provider/MCP maturity, sandbox/process boundaries, observability, resources, and supply chain | Deep research-backed production playbook |
| [Java](../languages/jvm/README.md) | Virtual threads, preview structured concurrency, interruption, HTTP/streams, tools/processes, schemas/effects, durable workers, memory, JFR/JMX/OTel, build, and deployment | Deep 16-guide JVM production playbook |
| [C#/.NET](../languages/csharp-dotnet/README.md) | `Task`/scope ownership, `CancellationToken`, channels/streams, HTTP resilience, tools/processes, schemas/effects, durable workers, resources, OTel/diagnostics, NuGet, deployment, Azure, and SDK maturity | Deep 15-guide .NET 10/C# 14 production playbook |
| [Kotlin](../languages/jvm/README.md) | Coroutine scopes, cancellation, Flow/backpressure, Java interoperability, serialization, durable workers, framework parity, observability, and JVM deployment | Deep 16-guide JVM production playbook with separate Kotlin maturity labels |

Do not infer a winner from framework counts. Selection must include team skills, concurrency model, deployment target, provider SDK quality, observability, durable-runtime support, and the workload's dominant bottleneck.
