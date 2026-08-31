# Testing, Load, and Concurrency

> **Last researched:** 2026-08-31
> **Applies to:** .NET 10 LTS, C# 14

Agent tests must control time, model behavior, streams, tools, and failures. A happy-path mock that returns one string does not exercise the runtime's real contract.

## Test layers

| Layer | What it proves |
|---|---|
| Pure unit | State transitions, budgets, validation, retry classification |
| Component | Provider adapter, parser, tool dispatcher, durable codec |
| Host integration | DI scopes, middleware, auth, shutdown, health |
| Contract | Real SDK serialization and provider-compatible fixtures |
| Fault/recovery | Crashes, redelivery, lock loss, ambiguous effects |
| Load/soak | Bounds, fairness, saturation, leaks, recovery |
| Published-artifact | Container, trimming/AOT, OS/process behavior |

Use either Microsoft Testing Platform or VSTest deliberately. They are different test platforms; do not assume settings and extensions are interchangeable. MSTest, NUnit, xUnit, and TUnit can sit above supported platform integrations.

## Behavioral evaluation

Infrastructure tests prove that the loop behaves safely; evaluations prove that it performs the intended task. Maintain an application-owned dataset of typical, edge, adversarial, and production-incident cases. Version each case with the expected outcome and relevant prompt, model route/snapshot, tool/schema versions, policy, memory/compaction strategy, and knowledge snapshot.

Use the strongest evaluator available for each assertion:

1. deterministic checks for state transitions, authorization, budgets, tool name/arguments, citations/IDs, schema, effects, and prohibited actions;
2. reference- or rule-based scorers for grounded content and task-specific facts;
3. calibrated model judges only for subjective qualities such as coherence or style, with periodic human agreement checks.

Evaluate the full trace as well as the final answer. A plausible answer can hide an unauthorized tool call, a duplicated effect, an ignored refusal, or a compaction error. Because model output varies, run repeated samples for important cases and compare distributions/confidence intervals or stable pass-rate thresholds instead of exact prose or one lucky run.

The <code>Microsoft.Extensions.AI.Evaluation</code> libraries provide .NET evaluators and reporting/CI integration, including task-adherence, intent-resolution, and tool-call evaluators. They are building blocks, not a universal score. OpenAI's evaluation guidance likewise recommends task-specific, representative, continuous evaluation and human calibration. Its legacy Evals platform is scheduled to become read-only on 2026-10-31 and shut down on 2026-11-30, so do not make a new .NET release gate depend on that legacy service.

Cache deterministic fake/model responses for fast inner-loop tests, but run uncached canary and scheduled evaluations to detect provider/model drift. A critical invariant fails the release immediately; aggregate quality thresholds should also detect regressions by slice rather than hiding them in one average.

## Deterministic time

Inject <code>TimeProvider</code> into timeout, retry, lease, and idle-gap logic. <code>FakeTimeProvider</code> from <code>Microsoft.Extensions.TimeProvider.Testing</code> lets tests advance virtual time without sleeps.

Test:

- deadline just before and at expiry;
- cancellation while waiting for channel capacity;
- timer firing concurrently with successful completion;
- retry delay capped by remaining total budget;
- lease renewal and stale fencing token;
- shutdown drain deadline.

Virtual time does not control network stacks or arbitrary third-party SDK timers. Keep adapters behind interfaces and use a scripted server for end-to-end timing.

## Scripted model server

A useful fake provider is a real local HTTP server with scenario scripts:

- normal and fragmented SSE frames;
- tool calls split across arbitrary byte boundaries;
- malformed UTF-8/JSON and duplicate properties;
- response headers followed by a stalled body;
- disconnect before and after a complete event;
- 408, 409, 429, 5xx, and <code>Retry-After</code>;
- oversized headers, frames, and decompressed bodies;
- late response after caller timeout;
- partial output followed by provider error;
- request ID, usage, refusal, and incomplete terminal states.

Record observed network attempts so tests detect retry multiplication.

## Concurrency invariants

Test invariants instead of scheduling details:

- at most one current run owner/fencing token;
- every admitted run releases every resource permit;
- every completed effect has one normalized input;
- no terminal state transitions back to running;
- channel completion is observed exactly once;
- no scoped dependency survives its run;
- queue message is settled only after durable state;
- memory remains bounded with a slow consumer.

Use barriers and task-completion sources to force races at specific boundaries. Repeat stress tests, but do not treat many passing iterations as a proof.

## Effect fault matrix

Inject failure at each edge:

| Point | Expected result |
|---|---|
| Before request leaves | Safe retry |
| After remote commit, before response | Unknown; reconcile |
| After receipt, before local commit | Reconcile and commit without repeating |
| After local commit, before queue settlement | Redelivery observes committed state |
| During terminal event delivery | Run remains terminal; client resumes or fetches |

Exceptions are not a complete crash simulation. For restart-safety tests, terminate the worker process after durable intent, after the remote fixture commits, after the local receipt, and before broker settlement. Restart with a fresh DI container and verify recovery only from persisted records. Also inject storage timeouts, stale reads, lease loss, clock skew, DNS/TLS failures, truncated blobs, telemetry backpressure, and unavailable secret/identity endpoints within their documented contracts.

## Host and process tests

Use <code>WebApplicationFactory</code>/<code>TestServer</code> for ASP.NET Core integration, then add tests with real sockets for behavior TestServer does not reproduce.

For process tools, run fixtures that:

- fill stdout and stderr concurrently;
- spawn descendants;
- ignore graceful termination;
- emit invalid/huge lines;
- close one pipe early;
- write a secret-shaped value for redaction tests.

Run these tests on every supported operating system and container base.

## Load method

Increase offered load through the knee of the system while mixing small, large, slow-stream, throttled, and tool-heavy runs. Hold the test long enough for DNS rotation, GC cycles, handler rotation, queue leases, and autoscaling.

Pass criteria should include:

- bounded RSS, heap, channel bytes, threads, sockets, processes, and disk;
- stable latency for admitted work;
- explicit controlled rejections;
- no retry storm after injected throttling;
- fair tenant progress;
- clean drain within the real orchestration grace period;
- no lost or duplicated business effects.

Collect counters and traces during the run, not only after failure.

## Failure patterns

- <code>Task.Delay</code> sleeps make timeout tests slow and flaky.
- Provider client is mocked so serialization and retry behavior never execute.
- Tests assert exception messages instead of domain categories.
- Only average latency is reported.
- Load generator stops when the service rejects, hiding overload behavior.
- AOT tests build but never execute the published binary.
- Concurrency tests share one <code>DbContext</code> fixture accidentally.

## Review checklist

- [ ] Time is injectable and cancellation races are tested.
- [ ] A scripted HTTP provider exercises real streaming and retry paths.
- [ ] State/effect invariants have targeted race tests.
- [ ] Crash and redelivery points cover ambiguous outcomes.
- [ ] Behavioral evals cover typical, edge, adversarial, and incident cases by workload slice.
- [ ] Exact invariants dominate model judges; subjective judges are human-calibrated.
- [ ] Compaction and long-term retrieval have multi-turn continuity, isolation, and deletion tests.
- [ ] Load includes slow consumers and mixed resource weights.
- [ ] Published containers run integration tests on supported platforms.
- [ ] Test artifacts include safe counters/traces for failed runs.

## Primary sources

- [.NET testing overview](https://learn.microsoft.com/en-us/dotnet/core/testing/)
- [Microsoft Testing Platform](https://learn.microsoft.com/en-us/dotnet/core/testing/microsoft-testing-platform-intro)
- [.NET test platform overview](https://learn.microsoft.com/en-us/dotnet/core/testing/test-platforms-overview)
- [TimeProvider overview](https://learn.microsoft.com/en-us/dotnet/standard/datetime/timeprovider-overview)
- [FakeTimeProvider testing](https://learn.microsoft.com/en-us/dotnet/core/extensions/timeprovider-testing)
- [ASP.NET Core integration tests](https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests?view=aspnetcore-10.0)
- [OpenAI .NET mocking support](https://github.com/openai/openai-dotnet#mock-a-client-for-testing)
- [.NET AI evaluation libraries](https://learn.microsoft.com/en-us/dotnet/ai/evaluation/libraries)
- [OpenAI evaluation best practices](https://developers.openai.com/api/docs/guides/evaluation-best-practices)
- [OpenAI agent trace evaluation](https://developers.openai.com/api/docs/guides/agent-evals)
