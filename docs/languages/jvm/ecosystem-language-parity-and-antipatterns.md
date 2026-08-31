# Ecosystem, Language Parity, and Anti-Patterns

## Language decision

| Choose Java when | Choose Kotlin when |
|---|---|
| provider/enterprise libraries are Java-first | coroutine-first service and Flow composition are established |
| virtual-thread blocking style simplifies migration | suspending clients and structured scopes dominate |
| preview-free stable JDK APIs are required | Kotlin nullability/sealed/serialization models add value |
| team operations/debugging standardize on Java | team can audit cancellation and dispatcher behavior |

Both can share the same architecture. Do not rewrite a healthy Java service merely to get coroutines, or force callback/reactive Kotlin code where blocking Java on virtual threads is simpler.

## Capability parity is multidimensional

Score each candidate against:

- provider endpoints and streamed event fidelity;
- tool/MCP schema and content support;
- cancellation and deadline propagation;
- retry visibility and request IDs;
- serialization/BOM compatibility;
- telemetry context;
- durable runtime compatibility;
- release cadence, security response, and support policy;
- testing without live providers.

A framework can lead in demos and lag in exact provider features. Direct SDK adapters are often the best fallback for new endpoints.

## Anti-pattern catalog

### Unbounded cheap concurrency

**Symptom:** virtual thread/coroutine count grows while provider pools, heap, and latency collapse.

**Correction:** bound admission and each external resource; retain less per run.

### Hidden lifetime

**Symptom:** <code>GlobalScope</code>, fire-and-forget futures, or shared executors keep work after run cancellation.

**Correction:** owned scope, observed child result, staged shutdown.

### Reactive plus virtual threads everywhere

**Symptom:** callback/reactive pipeline runs on virtual threads without simpler stacks or actual backpressure.

**Correction:** choose one execution model per path; bridge only at boundaries.

### Swallowed cancellation

**Symptom:** broad exception handler retries InterruptedException/CancellationException.

**Correction:** propagate control signals before failure mapping.

### SDK as system architecture

**Symptom:** provider event classes are persistence schema; framework memory is source of truth.

**Correction:** internal state machine and ports; SDK/framework as adapters.

### Prompt history as state

**Symptom:** full chat transcript is the only record of decisions/effects.

**Correction:** structured versioned events and separate bounded prompt memory.

### Tool output is trusted

**Symptom:** retrieved/tool text overrides policy or becomes shell arguments.

**Correction:** treat output as untrusted evidence; schema/policy/approval boundary.

### Process equals sandbox

**Symptom:** <code>ProcessBuilder</code>, classloader, or obsolete Security Manager is the isolation claim.

**Correction:** OS/container/VM boundary with quotas and deny-by-default access.

### Whole-run retry

**Symptom:** transient provider error duplicates a completed tool side effect.

**Correction:** operation-level retry and durable idempotency record.

### Durable equals exactly once

**Symptom:** workflow/queue retry repeats an external effect after a crash window.

**Correction:** stable remote idempotency, reconciliation, outbox, or compensation.

### Unbounded streaming

**Symptom:** token queues and SharedFlow replay grow; “temporary” ofString holds entire output.

**Correction:** byte/event caps, flow control, explicit loss policy.

### Telemetry as data exfiltration

**Symptom:** prompts/tool args in spans and tenant/run IDs in metric labels.

**Correction:** redaction, opt-in content, secure references, low cardinality.

### Preview API without exit plan

**Symptom:** JDK structured-concurrency preview becomes public library contract.

**Correction:** hide it behind internal execution port; CI/runtime preview flags and migration plan.

## Decision records worth keeping

Record why the team selected:

- Java virtual threads or Kotlin coroutines for the main path;
- preview structured concurrency or stable executor ownership;
- provider SDK versus framework adapter;
- MCP version/capability floor;
- queue versus durable runtime;
- collector and container sizing;
- tool isolation tier;
- schema serializer and unknown-field policy.

Include evidence, rejected options, operational constraints, and a revisit trigger. This prevents benchmark folklore and provider marketing from replacing engineering context.

## Pre-production review

- [ ] Language boundary and interop cancellation are tested.
- [ ] Feature parity matrix uses exact pinned versions.
- [ ] No unowned tasks or unbounded buffers exist.
- [ ] State/effects survive crash and redelivery.
- [ ] Tool isolation matches consequence.
- [ ] Sensitive content is excluded from default telemetry.
- [ ] Load, cancellation, replay, and shutdown evidence exists.
- [ ] Preview/beta/experimental components are isolated and labeled.

## Sources

- [Oracle virtual threads guide](https://docs.oracle.com/en/java/javase/25/core/virtual-threads.html)
- [Kotlin coroutine exception handling](https://kotlinlang.org/docs/exception-handling.html)
- [MCP SDK tiers](https://modelcontextprotocol.io/community/sdk-tiers)
- [LangChain4j chat memory](https://docs.langchain4j.dev/tutorials/chat-memory/)
- [Micronaut LangChain4j status](https://micronaut-projects.github.io/micronaut-langchain4j/latest/guide/)
- [Quarkus LangChain4j tools](https://docs.quarkiverse.io/quarkus-langchain4j/dev/agent-and-tools.html)
