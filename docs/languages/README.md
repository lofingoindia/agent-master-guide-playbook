# Language and Runtime Guides

> **Status:** Research-backed selection and production guides are available; Python, TypeScript, Node.js, JVM, and .NET deep areas have completed dedicated Pass-2 usefulness/production refinement.
>
> **Research baseline:** 2026-08-31
>
> **Evidence:** [Runtime language selection packet](../research/packets/runtime-language-selection.md), [Python and TypeScript/Node.js runtime packet](../research/packets/python-and-typescript-agent-runtimes.md), [Python agent-engineering deep-dive packet](../research/packets/python-agent-engineering-deep-dive.md), [TypeScript agent-engineering deep-dive packet](../research/packets/typescript-agent-engineering-deep-dive.md), [Node.js agent-runtime deep-dive packet](../research/packets/nodejs-agent-runtime-deep-dive.md), [Go runtime packet](../research/packets/go-agent-runtimes.md), [Rust agent-engineering packet](../research/packets/rust-agent-engineering-deep-dive.md), [JVM agent-engineering packet](../research/packets/jvm-agent-engineering-deep-dive.md), and [C#/.NET agent-engineering packet](../research/packets/csharp-dotnet-agent-engineering-deep-dive.md)

Begin with [Choosing an agent runtime language](choosing-an-agent-runtime-language.md). If Go is in scope, continue with the [three-runtime comparison](../comparisons/go-vs-python-vs-typescript-node-agent-runtimes.md). If the choice is only Python versus TypeScript/Node.js, use their [focused comparison](../comparisons/python-vs-typescript-node-agent-runtimes.md).

The [deep knowledge-area expansion program](../research/deep-expansion-program.md) defines how these overview guides will become language-specific production playbooks without duplicating generic agent architecture.

## Available guides

| Guide | Use it for |
|---|---|
| [Choosing an agent runtime language](choosing-an-agent-runtime-language.md) | Component-level language selection, hard capability gates, runtime profiles, and justified polyglot boundaries |
| [Go agent runtimes in production](go-agent-runtimes.md) | Goroutine ownership, context cancellation, bounded concurrency, HTTP/process safety, schemas, durability, profiling, testing, and shutdown |
| [Go agent engineering](go/README.md) | Fourteen-guide production playbook for run ownership, deadlines, byte-aware backpressure, streams, tools/processes, schemas, effects, durable workers, resources, diagnostics, tests, supply chain, deployment, and ecosystem choices |
| [Python agent runtimes in production](python-agent-runtimes.md) | `asyncio`, cancellation, blocking/CPU isolation, schemas, workers, packaging, telemetry, and shutdown |
| [Python agent engineering](python/README.md) | Fourteen-guide production playbook for task ownership, deadlines, isolation, streams, schemas, effects, durability, state, workers, diagnostics, testing, supply chain, deployment, and ecosystem choices |
| [TypeScript and Node.js agent runtimes in production](typescript-node-agent-runtimes.md) | Event-loop health, abort propagation, workers, streams, runtime schemas, deployment targets, supply chain, and shutdown |
| [TypeScript agent engineering](typescript/README.md) | Eleven-guide language and contract playbook for compiler/module contracts, runtime schemas, state machines, tools/events, provider adapters, packages, artifacts, testing, supply chain, and migrations |
| [Node.js agent runtime engineering](nodejs/README.md) | Fifteen-guide runtime playbook for event-loop ownership, abort/deadline trees, streams, Undici, workers/processes, sandboxes, queues, memory, effects, tracing, diagnostics, tests, security, and deployment |
| [JVM agent engineering](jvm/README.md) | Sixteen-guide Java/Kotlin production playbook for virtual threads, coroutines, cancellation, streams, schemas, effects, durable workers, tools, memory, observability, tests, build integrity, deployment, and ecosystem parity |
| [C# and .NET agent engineering](csharp-dotnet/README.md) | Fifteen-guide .NET 10/C# 14 playbook for task ownership, cancellation, channels, HTTP resilience, tools, schemas, effects, durable workers, resources, telemetry, testing, NuGet, deployment, and SDK maturity |
| [Rust agent engineering](rust/README.md) | Twelve-guide playbook for ownership, Tokio cancellation, streaming, schemas, effects, durability, ecosystem maturity, resource control, sandboxing, testing, supply chain, and failure injection |
| [Go vs Python vs TypeScript/Node.js](../comparisons/go-vs-python-vs-typescript-node-agent-runtimes.md) | Component-first selection, framework maturity, failure-injection bake-off, and polyglot boundary rules |
| [Python vs TypeScript/Node.js](../comparisons/python-vs-typescript-node-agent-runtimes.md) | Direct selection criteria, failure-injection bake-off, operational costs, and polyglot decision rules |

## Shared research rubric

Each language guide covers only what changes materially in that ecosystem:

- concurrency and structured concurrency;
- cancellation propagation and uncooperative work;
- async I/O, streaming, HTTP connection reuse, and backpressure;
- thread/process/worker models and queue integration;
- memory management and resource lifetime;
- state serialization and durable-runtime support;
- error taxonomies, retries, and idempotency support;
- schema/structured-output tooling;
- testing, fakes, deterministic clocks, and chaos testing;
- telemetry, context propagation, and profiling;
- command/process execution safety;
- packaging, dependency risk, deployment, and cold starts.

## Python

[Python agent runtimes in production](python-agent-runtimes.md) is research-backed against CPython 3.14.7. It covers `asyncio` task ownership, thread/process/subinterpreter escape hatches, cancellation limits for synchronous tools, runtime validation, worker deployment, reproducible packaging, and runtime-health telemetry.

Continue into the [14-guide Python agent-engineering playbook](python/README.md) when implementing or reviewing a Python production runtime in depth.

## TypeScript and Node.js

[TypeScript and Node.js agent runtimes in production](typescript-node-agent-runtimes.md) is research-backed against Node.js 24.20.0 LTS and 26.8.1 Current. It covers event-loop saturation, `AbortSignal` propagation, streams and backpressure, workers and processes, runtime schemas, Node/serverless/edge boundaries, supply chain, and lifecycle operations.

Continue into the [TypeScript agent-engineering playbook](typescript/README.md) for compiler, erased-type, schema, state/event, provider-adapter, package, artifact, and migration engineering. Use the [Node.js agent-runtime playbook](nodejs/README.md) for event-loop, abort, transport, streaming, worker/process, memory, diagnostics, security, and deployment mechanics. These areas are complementary rather than duplicate alternatives.

## Go

[Go agent runtimes in production](go-agent-runtimes.md) is research-backed against Go 1.27.0. It covers goroutine and channel ownership, `context.Context`, bounded concurrency, HTTP transport reuse, subprocess containment, JSON validation, framework and durable-runtime maturity, memory/CPU limits, diagnostics, testing, supply chain, and shutdown.

Continue into the [14-guide Go agent-engineering playbook](go/README.md) when implementing or reviewing a Go production runtime in depth.

## Rust

[Rust agent engineering](rust/README.md) is a research-backed 12-guide production playbook against Rust 1.98.0 and Tokio 1.53.1. It covers owned task trees, cancellation safety, byte-aware backpressure, process/Wasm isolation, Serde boundary validation, typed failure and effect semantics, durable-runtime maturity, provider/MCP choices, observability, resource admission, Cargo supply chain, and failure injection.

## Java and Kotlin on the JVM

[JVM agent engineering](jvm/README.md) is a research-backed 16-guide production playbook against JDK 25 LTS, Kotlin 2.3.20, and kotlinx.coroutines 1.11.0. It keeps Java virtual-thread/interruption semantics and Kotlin coroutine/cancellation semantics explicit while sharing application-owned state, schema, effect, durability, observability, build, and deployment contracts. Preview, beta, and provider/agent-SDK maturity remain labeled separately.

## C# and .NET

[C# and .NET agent engineering](csharp-dotnet/README.md) is a research-backed 15-guide production playbook against .NET 10 LTS and C# 14. It covers `Task` ownership, `CancellationToken`, bounded channels and streams, `HttpClient`/resilience policy, tool processes, strict JSON and structured output, retry-safe effects, durable workers, memory/admission, OpenTelemetry and diagnostics, concurrency tests, NuGet supply chain, deployment, Azure, MCP, Agent Framework, and durable-runtime maturity.

Other languages will be added when serious SDK, runtime, and production evidence justifies dedicated coverage.

See the [language index](../indexes/language-index.md).
