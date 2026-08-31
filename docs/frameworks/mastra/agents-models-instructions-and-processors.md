# Agents, Models, Instructions, and Processors

A Mastra agent is a model-driven loop. Production quality comes from bounding
that loop, controlling what enters and leaves it, and moving policy and effects
into deterministic code.

## Turn lifecycle

~~~mermaid
sequenceDiagram
    participant App
    participant P as Input processors
    participant M as Memory
    participant L as Model loop
    participant T as Tools
    participant O as Output processors
    App->>M: Resolve thread/resource context
    M->>P: Loaded messages and memory context
    P->>L: Validated and transformed input
    loop Until stop condition
        L->>T: Proposed tool calls
        T-->>L: Validated results
    end
    L->>O: Stream or final response
    O-->>App: Allowed response
    O->>M: Persist accepted transcript
~~~

Memory's input processing occurs before user-configured input processors.
User-configured output processors run before the accepted response is persisted,
so an abort can prevent unsafe generated output from entering memory. Verify
this ordering after upgrades because it is a security-relevant contract.

## Define an agent contract

Every agent should document:

| Field | Required decision |
|---|---|
| Purpose | One bounded job and explicit non-goals |
| Model | Exact provider/model and fallback policy |
| Instructions | Stable behavioral guidance, not authorization |
| Tools | Minimum allowlist for the job |
| Memory | Thread/resource scope and retention |
| Bounds | Maximum steps, time, tool concurrency, and cost |
| Output | Schema or stream contract |
| Failure | User-visible error and retry/recovery policy |
| Evidence | Trace attributes and evaluation dataset |

Large universal agents weaken tool selection, authorization review, evaluation,
and cost control. Prefer a small toolset whose names and descriptions make
selection unambiguous.

## Generate versus stream

Use <code>generate()</code> when the caller needs one completed result and does
not benefit from incremental events. Use <code>stream()</code> when the product
needs text deltas, tool progress, approvals, or long-running feedback.

Streaming is a protocol, not only a latency optimization. The consumer must
handle tool events, errors, aborts, finish reasons, and the terminal result. Some
stream result promises settle only as the stream is consumed or closed; leaving
a stream unread can leave application state unresolved.

## Bound the loop

Set explicit:

- maximum steps or stop conditions;
- total turn deadline;
- per-model-step timeout;
- tool-call concurrency;
- input and output size limits;
- retry counts by error class;
- a propagated <code>AbortSignal</code>.

Current model settings distinguish a hard total deadline from a per-step model
deadline. A step timeout may trigger configured model fallback; a total timeout
terminates the run. Test the actual provider because cancellation and partial
billing behavior differ.

Do not use one budget for every request. A read-only lookup agent and an agent
allowed to send messages should have different bounds and approval policies.

The stable stream API exposes the two timeout budgets under
<code>modelSettings.timeout</code>:

~~~ts
const controller = new AbortController();

const stream = await agent.stream("Summarize the incident", {
  maxSteps: 6,
  toolCallConcurrency: 2,
  abortSignal: controller.signal,
  modelSettings: {
    timeout: {
      totalMs: 45_000,
      stepMs: 12_000,
    },
  },
});
~~~

A <code>totalMs</code> expiry terminates the whole run. A
<code>stepMs</code> expiry can advance to the next configured fallback model,
so it is also a routing decision. Reaching <code>maxSteps</code> while the model
is attempting another tool call can produce no useful final text. Test the last
step explicitly and, where the product requires a final summary, use a tested
processor/stop policy rather than increasing the limit indefinitely.

## Dynamic configuration

Instructions, model, tools, and memory may be resolved dynamically from request
context. This is useful for locale, tenant-specific model routing, entitlements,
and gradual rollout. It is also a place where cross-request leakage can be
introduced.

Safe pattern:

1. authenticate the request;
2. load authoritative tenant/user policy;
3. build a fresh typed request context;
4. resolve a minimal tool allowlist;
5. invoke the registered agent;
6. discard request-scoped objects when the turn ends.

Never mutate shared agent configuration for one request.

## Instructions are guidance, not enforcement

Instructions should cover role, task, sources of truth, tool-selection
guidance, uncertainty, and output requirements. They should not be the only
place that says:

- which tenant can be read;
- how much money may be spent;
- whether deletion is allowed;
- whether a user is an administrator;
- whether an approval is still valid.

Those checks belong in middleware and tool/workflow code. Models can be induced
to ignore text, and tool results can contain prompt injection.

## Structured output

Use a schema when another program consumes the result. Validate again at the
application boundary; model structured output is not a business invariant.

Provider support varies when structured output and tool calls occur in the same
turn. A separate structuring model can turn a free-form/tool-using result into a
schema when the primary provider cannot reliably do both. That adds latency,
cost, and another failure mode, so evaluate it explicitly.

Production tests should include:

- missing, extra, nullable, and enum fields;
- tool calls before the final structured response;
- truncated streams and provider refusal;
- schema evolution across stored results;
- model fallback to a provider with different capabilities.

## Processor pipeline

Processors are the correct framework seam for cross-cutting model I/O controls:

### Input processors

- normalize and size-limit messages;
- remove unsupported content;
- detect prompt injection signals;
- redact data that must not reach the model;
- select or suppress tools for the current step;
- enforce policy based on trusted request context.

### Output processors

- redact sensitive output;
- validate format and citations;
- abort or retry unacceptable responses;
- transform streaming content;
- attach safe metadata.

### Error processors

- normalize provider errors;
- record classified failure data;
- select a user-safe response;
- avoid leaking raw provider payloads or secrets.

Order processors deliberately. A sanitizer after a logger is too late for the
log. A policy processor after memory loading may still allow forbidden records
to be retrieved even if they are later removed from the prompt.

## Response caching is beta

Mastra's processor response cache is beta. It can cache model responses based on
the resolved prompt, model, step, and scope. Treat it as unsafe for a turn that
invokes side-effecting tools: a cache hit can replay recorded tool-call content
without re-executing the tool, making the transcript look like an effect
occurred when it did not.

If adopted:

- limit it to pure, read-only generation;
- include tenant/resource scope in the key;
- use a shared production cache;
- set short, explicit TTLs;
- record hit/miss and model/profile version;
- exclude secrets and unstable context;
- run a cache-isolation test with concurrent tenants.

## Model fallback policy

Fallback improves availability only if alternate models satisfy the same
contract. Grade each candidate on tool support, schema adherence, latency,
price, safety, data residency, and context length. A cheaper model that omits a
required tool argument is not a successful fallback.

Recommended fallback categories:

- retry the same provider only for a classified transient error;
- move to a compatible provider when the operation is still safe to repeat;
- do not silently fallback when policy, residency, or approval semantics differ;
- surface degraded mode in traces and product state.

## Common failures

| Failure | Likely cause | Production response |
|---|---|---|
| Endless tool loop | No step/stop bound; ambiguous tool results | Stop, trace loop signature, improve tool contract |
| Cross-tenant behavior | Shared mutable configuration or unscoped cache | Disable path, inspect context construction, add isolation test |
| Valid JSON, wrong action | Schema validates syntax but not business rule | Enforce invariant in tool/workflow |
| Stream never completes | Consumer stopped reading or terminal event lost | Abort/close and reconcile run state |
| Different behavior after model swap | Provider capability mismatch | Run provider conformance suite before rollout |
| Unsafe output persisted | Processor order/configuration regression | Block release with persistence-order test |

## Release checklist

- [ ] Agent purpose and non-goals are documented.
- [ ] Tools are minimal and authorized in code.
- [ ] Step, time, concurrency, and cost bounds are explicit.
- [ ] Provider and fallback profiles pass the same tests.
- [ ] Structured output is business-validated.
- [ ] Processor ordering has a regression test.
- [ ] Response caching is disabled for side effects.
- [ ] Stream consumers handle error, abort, and terminal events.

## Primary sources

- [Agents documentation source](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/agents)
- [Processors documentation source](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/agents/processors.mdx)
- [Structured output documentation source](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/agents/structured-output.mdx)
- [Agent source](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/packages/core/src/agent)
- [Core changelog](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/packages/core/CHANGELOG.md)
- [Canonical run-controls guide](../../runtime/run-controls.md)
