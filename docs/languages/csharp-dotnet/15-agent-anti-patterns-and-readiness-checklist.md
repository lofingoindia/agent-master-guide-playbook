# Agent Anti-Patterns and Readiness Checklist

> **Last researched:** 2026-08-31
> **Applies to:** .NET 10 LTS, C# 14

This guide turns the preceding design rules into review questions and failure signatures. It is intentionally skeptical: a demo that completes one run says little about behavior under cancellation, overload, crash, ambiguous effects, or upgrade.

## High-risk anti-patterns

| Anti-pattern | Why it fails | Replacement |
|---|---|---|
| Fire-and-forget run/task | Lost exceptions, scopes, and shutdown ownership | One run owner joins all children |
| Timeout only wraps the wait | Underlying call/process continues | Pass cancellation and retain cleanup ownership |
| Unbounded channel/list/transcript | Slow consumer becomes memory outage | Item and byte bounds with overload policy |
| New <code>HttpClient</code> per call | Connection churn and socket pressure | Long-lived client or factory handlers |
| Retries at every layer | Cost/traffic amplification and duplicate effects | One retry budget, lower layers configured |
| Schema equals trust | Valid malicious input still executes | Domain validation and authorization |
| Child process equals sandbox | Same identity can access host resources | OS/container/VM isolation and narrow capability |
| Broker equals exactly once | Redelivery and lost acknowledgments remain | Inbox, effect IDs, reconciliation |
| Conversation equals state | Missing leases, effects, versions, approvals | Explicit versioned state machine |
| One "memory" store | Context, state, transcript, and inferred facts gain the same trust/lifetime | Separate working, session, operational, and semantic contracts |
| Compaction is arbitrary summarization | Decisions, tool pairs, approvals, or unknown effects disappear | Versioned derived view with preservation invariants and continuation evals |
| Framework equals production runtime | Durability, admission, security remain | Assign responsibility per layer |
| Demo transcript equals evaluation | One nondeterministic sample hides task/safety regressions | Versioned representative dataset, invariant checks, and repeated evals |
| Metrics contain run/user IDs | Cardinality explosion | IDs in traces/logs; bounded metric dimensions |
| Native AOT by configuration only | Reflection/dynamic failures appear after publish | Publish-and-run compatibility gate |

## Failure signature matrix

| Symptom | Likely mechanism | Evidence to collect first |
|---|---|---|
| Latency rises, CPU low, threads rise | Thread-pool starvation | Runtime counters and blocking stacks |
| Memory grows with streaming clients | Broken backpressure/retained events | Channel bytes, heap graph, slow-client traces |
| Provider sees far more calls than runs | Layered retries | Logical-call versus attempt counters |
| Tool runs after user cancellation | Wait canceled, work not canceled | Task/process ownership trace |
| Duplicate email/payment/update | Ambiguous effect blindly retried | Effect journal and downstream receipt |
| Runs disappear during deploy | In-memory queue/fire-and-forget | Queue/state transition audit |
| Random tenant data appears | Shared singleton/session/handler scope | DI scope and correlation trace |
| Container never exits | Unjoined task/process or blocking exporter | Shutdown trace, process tree, task stacks |
| AOT build succeeds but endpoint fails | Uncovered reflection path | Publish warnings and artifact smoke test |
| Autoscaling increases 429s | Local-only rate limits | Provider partition concurrency across replicas |

## Architecture readiness

- [ ] The run owner, child-task tree, DI scope, and terminal commit are documented.
- [ ] Mutable state is run/attempt scoped; shared services are thread-safe.
- [ ] Parallel branches have limits, sibling-failure semantics, and resource reservations.
- [ ] In-memory and durable work classes are explicitly distinguished.
- [ ] State, message, attempt, and effect identifiers have separate meanings.

## Time and shutdown readiness

- [ ] Caller cancellation, total deadline, attempt timeout, stream idle timeout, and host shutdown are distinguishable.
- [ ] Tokens reach underlying SDK, storage, channel, and tool APIs.
- [ ] Canceling a wait does not orphan work.
- [ ] Readiness/admission stops before drain.
- [ ] Persistence, settlement, telemetry, and process termination fit inside the shutdown budget.
- [ ] Crash recovery does not depend on <code>StopAsync</code>.

## Resource readiness

- [ ] Every queue/buffer has item and byte bounds.
- [ ] Prompt, result, schema, frame, file, and durable-history sizes are limited.
- [ ] Admission covers tenants, providers, memory-heavy runs, and tool pools.
- [ ] Overload returns a controlled result or uses an external durable queue.
- [ ] RSS, heap/LOH, threads, sockets, processes, disk, and queue age plateau under load.

## Network and provider readiness

- [ ] Client lifetime and DNS rotation are explicit.
- [ ] SDK, handler, worker, and broker retries have been counted.
- [ ] Request/attempt/total/idle timeouts align with one deadline budget.
- [ ] Partial stream failure and incomplete/refusal terminal states are handled.
- [ ] Provider request IDs and rate-limit metadata are captured safely.
- [ ] Each model/deployment/schema combination has contract fixtures.

## Tool and effect readiness

- [ ] Tool discovery is separate from authorization.
- [ ] Tool inputs receive strict parse, domain validation, canonicalization, and policy.
- [ ] Filesystem, network, process, time, byte, and output limits are enforced outside the model.
- [ ] External writes use stable effect IDs and durable intent/receipt records.
- [ ] Ambiguous outcomes reconcile instead of replaying blindly.
- [ ] Approval state survives restarts and cannot be reused for changed input.

## State and durability readiness

- [ ] Durable state uses explicit states, versions, and migration/upcast strategy.
- [ ] Queue settlement follows durable commit.
- [ ] Leases use conditional writes or fencing tokens.
- [ ] Replay-based workflows keep nondeterministic work in activities.
- [ ] Large data is stored by bounded immutable reference.
- [ ] Dead-lettered work has an owner, safe reason, and replay procedure.

## Context and memory readiness

- [ ] Working context, session continuity, durable operational state, long-term memory, and knowledge/RAG are separate contracts.
- [ ] Context construction reserves output/tool-schema headroom and has a controlled too-large result.
- [ ] Compaction never splits tool call/results or discards policy, decisions, approvals, user corrections, identifiers, or effect status.
- [ ] Compacted views record source range/hash and version and can be rebuilt from authoritative records.
- [ ] Long-term memory has provenance, tenant scope, consent/retention, correction/deletion, conflict, and expiry policy.
- [ ] Similarity retrieval is followed by authoritative lookup and current authorization.

## Security and privacy readiness

- [ ] Model, retrieved, MCP, and tool content is untrusted.
- [ ] Tenant/caller identity is rechecked at every effect boundary.
- [ ] Secrets do not enter prompts, command lines, telemetry, or durable histories.
- [ ] Tool execution uses least privilege, egress policy, and real isolation where needed.
- [ ] Diagnostic channels, dumps, and content traces have restricted access and retention.
- [ ] Package feeds, identities, source mapping, audits, and artifacts are controlled.

## Verification and operations readiness

- [ ] Virtual-time tests cover cancellation and lease races.
- [ ] Scripted provider tests cover throttling, malformed/partial streams, stalls, and late completion.
- [ ] Effect tests fail before/after remote commit and local persistence.
- [ ] Load tests include slow consumers and mixed run weights.
- [ ] The exact published container passes startup, shutdown, and process tests.
- [ ] Dashboards separate logical operations from attempts and expose saturation.
- [ ] SLOs use explicit terminal semantics and expose admission, latency, recovery, unknown effects, and quality separately.
- [ ] Behavioral evals cover typical, edge, adversarial, and production-incident cases by important workload slice.
- [ ] Deterministic authorization/state/tool/effect assertions take precedence over model-judge scores.
- [ ] Failure injection includes real process termination between durable/network boundaries, not exceptions alone.
- [ ] Rollback preserves state/schema compatibility.

## Release decision

Do not release because every box is mechanically checked. Release when evidence demonstrates the invariants relevant to the workload. A read-only summarization agent may not need a durable effect journal; an agent that sends money or modifies infrastructure does. The rigor should follow the worst credible effect and failure, not the average request.

## Primary sources

- [.NET support policy](https://dotnet.microsoft.com/en-us/platform/support/policy)
- [.NET asynchronous programming](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/)
- [HttpClient guidelines](https://learn.microsoft.com/en-us/dotnet/fundamentals/networking/http/httpclient-guidelines)
- [.NET channels](https://learn.microsoft.com/en-us/dotnet/core/extensions/channels)
- [Dependency injection guidelines](https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/guidelines)
- [.NET diagnostics overview](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/)
- [.NET Native AOT](https://learn.microsoft.com/en-us/dotnet/core/deploying/native-aot/)
- [MCP authorization specification](https://modelcontextprotocol.io/specification/2025-06-18/basic/authorization)
