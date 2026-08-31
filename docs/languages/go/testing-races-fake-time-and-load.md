# Testing Go Agent Services: Races, Fake Time, and Load

> **Last researched:** 2026-08-31  
> **Baseline:** Go 1.27, `testing/synctest`, race detector, native fuzzing

Agent tests must prove lifecycle and effect invariants, not only compare final text. The highest-value failures occur during cancellation, timeout, duplicate delivery, slow consumption, lost responses, replay, shutdown, and resource exhaustion.

## Build a layered test portfolio

| Layer | What to prove |
|---|---|
| Pure unit/property | State transitions, budgets, validation, error/retry classification |
| `synctest` concurrency | Deadlines, timers, cancellation, quiescence, background cleanup |
| HTTP/protocol | Body limits, streaming, disconnects, negotiation, malformed events |
| Provider/tool contract | Exact SDK version, schema dialect, error/usage/stream behavior |
| Durable replay | Determinism, activity/step boundaries, version and payload migrations |
| Race/fuzz | Executed data races and hostile parser/validator inputs |
| Integration/load/soak | Pools, quotas, RSS, GC, scheduler, fairness, shutdown under realistic faults |

Do not make live model calls the foundation of deterministic tests. Use recorded/golden semantic fixtures and narrow fakes; keep a smaller live contract suite for current provider behavior.

## Use `testing/synctest` for time and quiescence

`testing/synctest` has been GA since Go 1.25. A test bubble gives goroutines a fake clock; time advances when every goroutine in the bubble is durably blocked. `synctest.Wait` waits for quiescence, and Go 1.27 adds `synctest.Sleep` as a combined helper.

```go
func TestRetryStopsOnDeadline(t *testing.T) {
	synctest.Test(t, func(t *testing.T) {
		ctx, cancel := context.WithTimeout(t.Context(), time.Minute)
		defer cancel()

		done := make(chan error, 1)
		go func() { done <- retry(ctx, alwaysTransient) }()

		synctest.Sleep(time.Minute)
		if err := <-done; !errors.Is(err, context.DeadlineExceeded) {
			t.Fatalf("got %v", err)
		}
	})
}
```

Use it for retry schedules, debounce/coalescing, cancellation cleanup, lease-heartbeat logic behind fakes, and proving something has not happened before a deadline.

Limitations matter:

- only goroutines associated with the bubble participate;
- real network/process behavior requires fakes or special test facilities;
- durable blocking/quiescence has defined rules; not every external wait is visible;
- a deterministic fake does not replace OS/proxy/provider integration tests.

Go 1.27 adds an in-memory `httptest.NewTestServer` designed for `synctest`, allowing HTTP behavior without a real network. Use the exact 1.27 API rather than older `httptest.NewServer` assumptions in bubble tests.

## Run the race detector on realistic paths

`go test -race ./...` finds races only on executed paths. Add race-enabled integration scenarios for concurrent tool completion, state updates, stream disconnects, cancellation, shutdown, and cache refresh.

Official documentation reports typical overhead of roughly 2–20× execution time and 5–10× memory, so do not use race runs for performance conclusions. Long-running goroutines with repeated `defer`/`recover` also have race-runtime memory behavior that is not visible in normal runtime profiles.

Schedule race suites separately if ordinary CI time/memory is insufficient. A race-free run does not prove absence of races; it proves none were observed on that execution.

## Fuzz hostile boundaries

Go's native fuzzing is coverage-guided. High-value targets include:

- JSON v1/v2 boundary decoders and version envelopes;
- SSE/MCP/provider event parsers;
- tool argument validators and canonical request digests;
- URL/path/redirect policy;
- archive/document metadata and artifact references;
- error-envelope and durable payload migrations;
- prompt/tool-result truncation and redaction.

Fuzz targets should be fast, deterministic, independent, and reset state for every call. Preserve minimized failing inputs in `testdata/fuzz` as regression corpus. Add resource assertions: no panic, bounded allocation/time, no path escape, stable classification, and no effect invocation before full validation.

## Test stream lifecycle

For each HTTP/SSE/WebSocket/MCP stream:

- disconnect before headers, mid-event, and after effect commit;
- stop reading and fill pending buffers;
- send oversized/fragmented/malformed events;
- pause long enough to hit idle timeout;
- cancel while a handler is blocked on provider read or client write;
- race process shutdown with active streams;
- verify terminal events/state and reconnect semantics;
- check all bodies, pipes, connections, and goroutines close.

Track baseline goroutines and use profiles/Go 1.27 `goroutineleak` after quiescence, while allowing documented runtime/test harness goroutines. A raw goroutine-count equality assertion is often flaky and uninformative; assert ownership-specific completion first.

## Test durable and idempotent behavior

Use deterministic fault points around every correctness boundary:

1. before effect reservation;
2. after reservation, before call;
3. after remote commit, before response;
4. after response, before receipt persistence;
5. after receipt, before state transition;
6. after transition, before queue acknowledgement.

Restart/redeliver at each point. The outcome should be one committed effect or a visible reconciliation state—not silent duplication.

Replay retained workflow histories/payloads against new code before rollout. Test engine-specific nondeterminism detection and versioning, including map iteration, goroutine/select usage, time/random values, and changed activity order.

## Load tests must include faults

A prompt-throughput benchmark is not a capacity test. Exercise:

- maximum prompt/retrieval/tool-result bytes at target concurrency;
- long streams and slow consumers;
- provider rate limits and retry storms;
- one tenant attempting to monopolize permits;
- DB/pool saturation independently from provider saturation;
- maximum subprocess/sandbox load;
- cancellation storms and rolling termination;
- GC/memory limit and cgroup CPU throttling;
- queue backlog, lease extension, redelivery, and version rollout.

Measure p50/p95/p99 for admission wait, model first event, tool wait/execution, stream delivery, and total run. Record throughput, rejected/deferred work, active permits, pending bytes, RSS, GC CPU, scheduler latency, goroutines, connections, and dependency attempt counts.

## Avoid weak tests

| Weak test | Why it fails | Better proof |
|---|---|---|
| `time.Sleep` then inspect | Slow and flaky | `synctest` quiescence or explicit join |
| Assert exact provider prose | Model nondeterminism | Schema/invariant/effect assertions |
| Only mock SDK methods | Misses wire/stream compatibility | Local protocol server + live contract suite |
| Goroutine count equals baseline | Runtime noise; no ownership proof | Wait for owned group/connection registry and profile leaks |
| Retry test checks final success | May hide attempt explosion | Assert attempt count, delays, effect IDs, deadline |
| Performance test with `-race` | Distorted CPU/memory | Separate race and benchmark/load runs |
| Happy-path workflow replay | Misses upgrade nondeterminism | Replay retained production-like histories across versions |

## Release gate

- [ ] Unit/property tests cover run-state and error/retry matrices.
- [ ] `synctest` covers deadlines, cancellation, retry timing, and cleanup.
- [ ] Race-enabled suites exercise concurrent integration paths.
- [ ] Fuzzers cover untrusted JSON, stream, path, URL, and migration boundaries.
- [ ] Provider/framework/MCP contract tests pin exact versions and negotiated behavior.
- [ ] Lost-response-after-commit and duplicate delivery are proven safe.
- [ ] Shutdown tests include active HTTP, SSE, WebSocket, queue, durable, and subprocess work.
- [ ] Faulted load tests stay within memory/CPU/byte/concurrency budgets and preserve fairness.
- [ ] Profiles and traces are captured for regressions, not only pass/fail numbers.

## Selected primary sources

- [Testing time with `testing/synctest`](https://go.dev/blog/testing-time)
- [`testing/synctest` Go 1.27](https://pkg.go.dev/testing/synctest@go1.27.0)
- [Go race detector](https://go.dev/doc/articles/race_detector)
- [Go fuzzing](https://go.dev/doc/security/fuzz/)
- [Go 1.27 release notes](https://go.dev/doc/go1.27)

