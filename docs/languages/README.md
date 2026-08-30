# Language and Runtime Guides

> **Status:** Research-backed selection guide available; per-language deep dives remain active research.  
> **Research baseline:** 2026-08-30  
> **Evidence:** [Runtime language selection packet](../research/packets/runtime-language-selection.md)

Begin with [Choosing an agent runtime language](choosing-an-agent-runtime-language.md). It evaluates components, ecosystem parity, cancellation, schemas, operations, and justified polyglot boundaries without declaring a universal winner.

## Shared research rubric

Each language guide will cover only what changes materially in that ecosystem:

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

Research `asyncio` task semantics, task groups, thread/process escape hatches, cancellation limits for synchronous tools, typed validation, worker deployment, and the unusually broad agent-framework ecosystem.

## TypeScript and Node.js

Research event-loop saturation, `AbortSignal` propagation, web/Node streams, worker threads and child processes, schema inference, serverless execution lifetimes, and provider/framework portability.

## Go

Research goroutine lifecycle, `context.Context`, channel ownership, bounded parallelism, HTTP transport reuse, explicit error handling, low-overhead workers, and current framework/durable-runtime maturity.

## Rust

Research Tokio task cancellation, ownership of shared run state, async traits, typed errors, process isolation, resource bounds, and the smaller but performance-oriented SDK ecosystem.

## Java

Research virtual threads vs reactive stacks, interruption and structured concurrency, executor ownership, JVM memory/telemetry, Spring integration, and ADK/Microsoft/provider SDK maturity.

## C# and .NET

Research `Task`, `CancellationToken`, async streams, hosted services, HTTP resilience, dependency injection scopes, Azure Durable Functions/Agent Framework integration, and OpenTelemetry support.

## Additional ecosystems

Kotlin is already relevant through coroutines, JVM deployment, and Google ADK support. Other languages will be added when serious SDK, runtime, and production evidence justifies dedicated coverage.

See the [language index](../indexes/language-index.md).
