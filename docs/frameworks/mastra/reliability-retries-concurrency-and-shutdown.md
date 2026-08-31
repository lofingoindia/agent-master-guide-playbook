# Reliability, Retries, Concurrency, and Shutdown

Mastra provides retry, timeout, snapshot, PubSub, worker, and shutdown
mechanisms. Reliability still depends on an application contract for ambiguous
effects, bounded concurrency, recovery ownership, and termination order.

## Failure classification

| Class | Example | Default action |
|---|---|---|
| Invalid | Schema or business rule failure | Do not retry; return actionable error |
| Unauthorized | Actor cannot access tool/record | Do not retry; audit denial |
| Transient | Provider 429/503 before effect | Bounded backoff retry |
| Ambiguous | Timeout after sending a charge | Reconcile; never blind retry |
| Permanent dependency | Unsupported model/tool capability | Fail or route to validated fallback |
| Cancellation | Client/product abort | Propagate and stop new child work |
| Process loss | Runtime dies mid-step | Recover from checkpoint and receipts |
| Poison event | Repeatedly fails worker handler | Quarantine/operator action; workers lacked built-in DLQ |

Retry budgets should include total elapsed time and downstream limits. Avoid
nested retry multiplication between model SDK, Mastra step, HTTP client, worker,
and external engine.

## Agent bounds

Set maximum steps, stop conditions, total timeout, per-model-step timeout, and
tool-call concurrency. Pass the abort signal into every tool and client that can
accept it. On cancellation, record whether any effect is still ambiguous.

If an approval- or suspend-capable tool is registered, tool calls can be forced
to execute sequentially. Account for this in latency testing rather than raising
concurrency globally.

## Workflow retries

Mastra workflows have workflow-level retry configuration and per-step
overrides. The safest rule is:

- retry pure reads and model calls for explicit transient codes;
- use exponential backoff with jitter and a maximum delay;
- use idempotency and receipts for effects;
- make retry count visible in step output/traces;
- stop when the workflow's total deadline is exhausted.

An effect step should return a canonical receipt, not only a boolean.

## Concurrency controls

Concurrency exists at several layers:

~~~mermaid
flowchart TD
    A[Admission per user/tenant] --> B[Concurrent agent/workflow runs]
    B --> C[Agent tool calls]
    B --> D[Workflow parallel/foreach]
    C --> E[Provider/API limiters]
    D --> E
    E --> F[Database and connection pools]
~~~

Bound all of them. A <code>foreach</code> concurrency of five in each of 100
simultaneous runs is 500 outbound calls. Use global/per-tenant limiters or a
queue at the application boundary.

Mastra <code>parallel</code> begins all fixed branches without a global cap.
Loops need an application maximum. Backpressure should reject or queue before
large prompts and model calls consume capacity.

## Concurrent resume

Two clients, retries, or worker redelivery can resume the same suspended run.
Stable <code>@mastra/core@1.63.2</code> includes an atomic claim: only one caller
advances a suspension, a losing SDK caller receives
<code>WORKFLOW_RESUME_ALREADY_CLAIMED</code>, and the generated HTTP route
surfaces <code>409</code>. Preserve this as an adapter conformance test; it is a
workflow-state guarantee, not an exactly-once external-effect guarantee.

Also protect the product operation:

- unique approval consumption key;
- idempotent effect key;
- state-transition compare-and-set;
- duplicate event handling;
- regression test with simultaneous resume requests.

## Generated-server shutdown

The current generated server handles SIGINT/SIGTERM by stopping new admission,
draining active requests/streams for a configured interval, and then invoking
<code>mastra.shutdown()</code>. The documented default drain is five seconds; a
second signal can force immediate termination.

~~~ts
const mastra = new Mastra({
  // Keep this below the orchestrator's hard termination grace.
  server: { drainTimeout: 60_000 },
});
~~~

Lifecycle ownership depends on how Mastra is hosted:

| Surface | Who drains HTTP? | Who shuts down Mastra and extra clients? |
|---|---|---|
| Generated <code>mastra start</code>, signal handling enabled | Generated entry stops connections and drains active requests/streams | Generated entry invokes <code>mastra.shutdown()</code>; register application cleanup where documented |
| Generated entry with <code>handleShutdownSignals: false</code> | Nobody in the generated entry; Node's default signal behavior can terminate immediately | Application must provide a complete host lifecycle; prefer an adapter when HTTP server handles are needed |
| Embedded Hono/Express/Fastify/Koa/framework adapter | Host application | Host application, in an ordered shutdown hook |

Disabling generated signal handling is not merely replacing one callback. It
also removes the generated entry's access-driven drain sequence unless the
application owns an equivalent server handle.

Set drain timeout below the platform's termination grace but above measured
high-percentile persistence/flush time. Five seconds is rarely enough for an
unbounded model turn, so normal operation must already have step deadlines and
checkpoint strategy.

Recommended order:

1. fail readiness and stop accepting new work;
2. stop schedule claims and worker admission;
3. abort or drain active HTTP streams;
4. let bounded steps persist terminal/suspended state;
5. stop PubSub subscriptions and background workers;
6. flush observability;
7. call <code>mastra.shutdown()</code> and close extra clients;
8. exit before the platform sends a hard kill.

## Process-loss reality

A plain agent stream cannot resume from where model generation stopped after
process exit. A workflow snapshot can recover from the last durable state, not
guarantee that an in-flight external effect did not happen. Persistent stream
history needs a distributed PubSub/cache, separate from snapshots.

GitHub issue
[#21193](https://github.com/mastra-ai/mastra/issues/21193) reported a
version-specific Platform/core 1.56 case where shutdown closed the Postgres pool
before a durable step persisted. The issue was closed by the research date and
generated-server lifecycle has continued to evolve. Treat the report as a
kill-point test requirement, not proof of current behavior on 1.63.2.

## Workers are beta

Mastra workers can split orchestration, scheduling, and background tasks from
the API process over shared storage and distributed PubSub. Current documented
limitations materially affect reliability:

- no built-in dead-letter queue;
- exactly one scheduler process is required to avoid duplicate schedules;
- API process loss mid-step can leave a run in “running” state;
- durable-agent recovery can address some paths, but requires idempotent work;
- schedules missed during downtime are not automatically replayed.

Worker message handling is at-least-once in practical distributed operation.
Make handlers idempotent and expose lag, retry, and stuck-work metrics. Protect
worker endpoints with authentication and network controls.

## Recovery ownership

Stable durable-agent recovery acquires a per-agent/run lease and renews it while
the recovered segment runs. Whether that lease fences replicas depends on the
configured PubSub: Redis/Valkey Streams implement the lease-provider contract
on the pinned snapshot; a backend without that contract falls back to an
always-win no-op provider plus a process-local claim. In the latter topology,
two replicas can still recover the same run.

Run a two-process lease conformance test against the exact adapter. If it cannot
prove exclusion and lease-loss cancellation, designate one recovery leader or
use a database lease, queue partition, or external engine with expiry and
fencing. Lease ownership prevents competing recovery drivers; it does not make
replayed tool effects exactly once.

Product recovery table:

| Framework state | Product action |
|---|---|
| Terminal | Load canonical result and close |
| Suspended | Show verified current action; await authorized resume |
| Waiting | Show next wake time; scheduler/engine owns wake |
| Active with live owner | Observe; do not duplicate |
| Active without owner | Claim recovery, reconcile effects, restart safely |
| Unknown/ambiguous | Block blind retry; operator or downstream reconciliation |

## Chaos tests

Kill at:

- before and after tool send;
- before and after receipt commit;
- during snapshot persistence;
- during streaming after run ID;
- during approval resume;
- while observability flushes;
- while scheduler publishes;
- immediately before storage close.

Repeat with two replicas and duplicate delivery. Verify no cross-run context
pollution, no lost suspend payload, no duplicate effect, and a stable product
state.

## Checklist

- [ ] Failures are classified before retry.
- [ ] Retry multiplication and total elapsed time are bounded.
- [ ] Every effect has idempotency, receipt, and ambiguity handling.
- [ ] Concurrency is bounded globally and per tenant.
- [ ] Concurrent resume is tested on the selected adapter.
- [ ] Shutdown order and drain timeout fit the deployment platform.
- [ ] Exactly one layer owns signals, HTTP drain, Mastra shutdown, and extra clients.
- [ ] Worker beta limitations have explicit operational controls.
- [ ] Recovery has a single owner with fencing.
- [ ] Kill-point tests cover every persistence/effect boundary.

## Primary sources

- [Server lifecycle documentation source](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/server)
- [Worker documentation source](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/deployment/workers.mdx)
- [Workflow retry implementation](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/packages/core/src/workflows)
- [Durable shutdown failure report](https://github.com/mastra-ai/mastra/issues/21193)
- [Core changelog](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/packages/core/CHANGELOG.md)
- [Atomic resume claim implementation](https://github.com/mastra-ai/mastra/pull/21725)
- [Canonical durable-execution guide](../../runtime/durable-execution.md)
- [Canonical idempotency and side-effect guide](../../reliability/idempotency-and-side-effects.md)
