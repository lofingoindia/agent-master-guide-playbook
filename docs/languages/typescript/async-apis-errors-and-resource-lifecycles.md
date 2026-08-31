# Async APIs, Errors, and Resource Lifecycles

> **Last researched:** 2026-08-31  
> **Runtime boundary:** Type signatures express ownership and cancellation requirements. Event-loop behavior, socket/process cleanup, stream backpressure, worker isolation, signals, and shutdown sequencing belong in [TypeScript and Node.js agent runtimes](../typescript-node-agent-runtimes.md).

TypeScript can make asynchronous agent code easier to reason about, but it cannot add checked exceptions, guarantee cancellation, or close a resource. Model expected outcomes as data, catch unexpected failures as `unknown`, and make lifecycle ownership visible in API signatures.

## A promise types success, not rejection

`Promise<T>` says what fulfillment returns. It does not constrain rejection. JavaScript can throw or reject with any value, and TypeScript intentionally has no checked-exception surface. Therefore this is false confidence:

```ts
try {
  await provider.call(request);
} catch (error: ProviderRateLimitError) { // misleading assumption
  // Any value can arrive here.
}
```

With `useUnknownInCatchVariables`, narrow or normalize first:

```ts
function toError(value: unknown): Error {
  if (value instanceof Error) return value;
  return new Error("Non-Error value thrown", { cause: value });
}

try {
  return await executeCall();
} catch (caught: unknown) {
  throw classifyUnexpectedFailure(toError(caught));
}
```

An `instanceof` check works only when the value really shares that runtime constructor. Errors crossing workers, realms, JSON, or duplicated packages may not. Prefer stable discriminants for serialized failures and treat SDK error subclasses as adapter-local hints.

## Put expected failures in the return type

If a caller is expected to branch on an outcome—validation failure, policy denial, unsupported capability, quota exhaustion—use a discriminated union instead of a thrown exception:

```ts
type Result<T, E> =
  | { ok: true; value: T }
  | { ok: false; error: E };

type ToolPreparationError =
  | { code: "invalid_arguments"; issues: readonly ValidationIssue[] }
  | { code: "unknown_tool"; name: string }
  | { code: "policy_denied"; policyId: string }
  | { code: "unsupported_capability"; feature: string };

async function prepareToolCall(
  proposed: ProposedCall,
  context: PolicyContext,
): Promise<Result<AuthorizedCall, ToolPreparationError>>;
```

Throw for broken invariants, programmer defects, corrupted state, or failures that this layer cannot meaningfully handle. This is a design convention, not compiler enforcement; document it and test it.

Do not turn every function into `Result`. Internal pure helpers with no expected failure and APIs where interruption is exceptional remain clearer with ordinary return/throw behavior.

## Preserve error evidence without leaking it

Keep an internal failure and a serialized public form separate:

```ts
type InternalFailure = Readonly<{
  category: FailureCategory;
  retry: RetryAdvice;
  safeMessage: string;
  operation: string;
  providerCode?: string;
  requestId?: string;
  cause?: unknown;
}>;

type SerializedFailure = Readonly<{
  schema: "agent.failure";
  version: 1;
  category: FailureCategory;
  safeMessage: string;
  operation: string;
  providerCode?: string;
  requestId?: string;
}>;
```

`Error`, `cause`, stack traces, open handles, SDK instances, and arbitrary thrown values are not JSON wire contracts. Redact and serialize an allowlisted representation. Preserve detailed evidence in access-controlled logs or error systems according to policy, not in tool results sent back to a model.

Retry advice is descriptive. The caller still applies deadline, attempt count, idempotency, and operation semantics.

## Make cancellation and deadline inputs explicit

An async API that can block on I/O should accept execution context rather than reading hidden globals:

```ts
type OperationContext = Readonly<{
  signal: AbortSignal;
  deadline: number; // documented clock and unit
  correlationId: string;
}>;

async function callModel(
  request: ModelRequest,
  context: OperationContext,
): Promise<ModelResponse>;
```

The type forces propagation but does not guarantee the callee observes the signal. Contract tests must cancel during DNS/connection, response streaming, tool execution, and result persistence where those paths exist. Keep timeout creation, signal composition, and shutdown ownership in the runtime layer.

Avoid a loose options object with an optional signal on safety-critical paths. If cancellation is required, the property should be required. If a non-cancellable implementation exists, expose that limitation as a capability instead of accepting and ignoring a signal.

## Streaming APIs need an ownership contract

`AsyncIterable<T>` is an appropriate domain surface for normalized incremental events, but it hides lifecycle decisions unless documented:

```ts
interface ModelGateway {
  stream(
    request: ModelRequest,
    context: OperationContext,
  ): AsyncIterable<ModelEvent>;
}
```

Specify:

- who creates and owns the underlying connection;
- what `return()`/early loop exit does;
- whether the stream can yield after cancellation;
- whether it emits or throws terminal failures;
- whether exactly one terminal domain event is yielded;
- maximum buffering and fragment sizes;
- whether the iterable can be consumed more than once.

For an app-owned event stream, choose one failure convention. A practical split is to yield expected provider terminal outcomes as `failed` events and throw only for consumer misuse or invariant violations. Mixing “sometimes terminal event, sometimes thrown provider error” creates duplicate cleanup and missed telemetry.

```ts
for await (const event of gateway.stream(request, context)) {
  await consume(event);
  if (event.type === "completed" || event.type === "failed") break;
}
```

Early exit must trigger cleanup in the implementation's `finally` block. The type alone cannot prove that.

A minimal adapter-owned generator makes the ownership point concrete:

```ts
async function* normalizeStream(
  source: AsyncIterable<unknown>,
  context: OperationContext,
): AsyncGenerator<ModelEvent, void, void> {
  const iterator = source[Symbol.asyncIterator]();
  let terminal = false;

  try {
    while (true) {
      if (context.signal.aborted) throw context.signal.reason;
      const next = await iterator.next();
      if (next.done) break;

      for (const event of normalizeProviderEvent(next.value)) {
        if (terminal) throw new ContractViolation("provider event after terminal");
        terminal = event.type === "completed" || event.type === "failed";
        yield event;
      }
    }

    if (!terminal) {
      yield { type: "failed", error: prematureEndFailure() };
    }
  } finally {
    try {
      await iterator.return?.();
    } catch (cleanupFailure: unknown) {
      recordCleanupFailure(cleanupFailure, context.correlationId);
    }
  }
}
```

The implementation still has to pass the signal into the SDK/transport; checking it before `next()` cannot interrupt an already pending `next()`. Cleanup failure is recorded separately so it does not rewrite an already committed domain outcome. If cleanup failure compromises process health, the runtime supervisor—not the iterator type—decides whether to drain/restart the worker.

Test lifecycle races at the public TypeScript boundary:

| Race/edge | Required observable result |
|---|---|
| Signal already aborted | No provider request starts; one normalized cancellation result |
| Abort while `next()` is pending | Underlying operation is asked to abort; iterator settles within the cleanup budget |
| Consumer breaks after first delta | `return()`/`finally` runs and no later domain event is delivered |
| Provider ends without terminal | Exactly one normalized premature-end failure |
| Provider emits two terminals or data after terminal | Contract violation is detected; the second terminal is never accepted |
| Terminal emitted, then close fails | Domain outcome remains stable; cleanup failure is separately observable |
| Timeout after an effect may have reached an external system | Outcome is `unknown`/reconcile, not an automatic “failed then retry” |

Use fake iterators that expose whether `return()` was called and allow `next()`/close to settle in controlled orders. Wall-clock-only tests are slower and often miss the exact race.

## Resource ownership should be lexical where possible

TypeScript supports ECMAScript explicit resource management syntax. A resource implementing `Disposable` or `AsyncDisposable` can be bound with `using` or `await using`, and cleanup runs when the scope exits:

```ts
await using session = await provider.openSession(context);
return await runConversation(session, input);
```

This is valuable only when the emitted/runtime target and dependency ecosystem support the required symbols and semantics. If cross-runtime deployment is required, verify the built artifact on every target. A plain `try/finally` remains the clearest portable baseline:

```ts
const session = await provider.openSession(context);
try {
  return await runConversation(session, input);
} finally {
  await session.close();
}
```

Do not return a client, stream, or iterator after disposing the resource it depends on. Do not hide long-lived clients inside request-scoped factories. State ownership—application, run, request, or iterator—in the interface documentation.

## Supervise concurrency

Every started promise needs an owner that awaits it, returns it, or deliberately supervises it. Enable `@typescript-eslint/no-floating-promises` with type-aware linting. Prefixing with `void` suppresses some checks; use it only at an explicit background-task boundary that records failures and participates in shutdown.

```ts
function startSupervised(
  task: Promise<void>,
  supervisor: TaskSupervisor,
): void {
  supervisor.track(task); // owns rejection reporting and shutdown wait
}
```

Choose aggregation semantics intentionally:

| Construct | Semantics | Appropriate use |
|---|---|---|
| `Promise.all` | Rejects on first observed rejection; other work keeps running unless separately cancelled | All results required and failures trigger coordinated cancellation |
| `Promise.allSettled` | Waits for every result and exposes per-item outcome | Independent operations where partial results matter |
| Sequential loop | Bounded, ordered execution | Dependencies, strict rate limits, side effects requiring order |
| Bounded concurrency | Limited parallelism | Provider/tool fan-out without resource explosion |

TypeScript preserves tuple information well when inputs are a `const` tuple, but no type controls concurrency. Avoid `array.forEach(async ...)`; the returned promises are ignored. Use `for...of`, `Promise.all(array.map(...))`, or a runtime concurrency limiter.

## Callback rejections are also unknown

`useUnknownInCatchVariables` covers `catch`, but promise callback APIs can still type rejection reasons as `any`. The typed rule `@typescript-eslint/use-unknown-in-catch-callback-variable` closes that gap for `.catch()` and rejection callbacks.

Prefer `try/catch` around awaited code when it clarifies ownership. When a `.catch()` is appropriate:

```ts
await promise.catch((reason: unknown) => {
  throw classifyUnexpectedFailure(toError(reason));
});
```

## Failure patterns

| Pattern | Failure | Safer design |
|---|---|---|
| `Promise<T, E>` invented in a type alias | JavaScript promises have no typed rejection channel | `Promise<Result<T, E>>` for expected failures |
| Catch cast (`error as SdkError`) | Masks arbitrary thrown values | Catch `unknown`, narrow and classify |
| Returning raw `Error` in JSON | Drops fields or leaks stack/cause | Explicit versioned serialized failure |
| Optional cancellation everywhere | Implementations silently ignore it | Required context on blocking operations |
| Floating “fire-and-forget” promise | Lost rejection and shutdown race | Supervisor owns every background task |
| Async `forEach` | Work is not awaited | Explicit sequential or aggregate construct |
| SDK class used across packages/workers | `instanceof` may fail | App-owned discriminant and normalization |
| Stream throws expected provider outcome | Cleanup/terminal handling splits | One documented terminal convention |
| Abort checked only before `next()` | Pending I/O continues indefinitely | Propagate signal into transport and assert iterator settlement |
| Cleanup throw overwrites completed result | Operational defect changes domain history | Record cleanup separately; supervisor owns health action |

## Review checklist

- [ ] Expected failures are visible in return unions; unexpected failures enter as `unknown`.
- [ ] Catch clauses and rejection callbacks do not assume a type.
- [ ] Serialized errors are explicit, versioned, redacted JSON contracts.
- [ ] Blocking APIs require cancellation/deadline context where appropriate.
- [ ] Stream ownership, terminal semantics, early exit, and cleanup are documented and tested.
- [ ] Pre-abort, pending-read abort, early return, premature end, duplicate terminal, and cleanup-failure races have deterministic tests.
- [ ] Every promise is awaited, returned, or owned by a supervisor.
- [ ] Aggregation semantics and concurrency limits are intentional.
- [ ] Resource scope is clear and portable to all deployment targets.
- [ ] Retry classification does not bypass runtime attempt/idempotency policy.

## Primary sources

- [TypeScript TSConfig: `useUnknownInCatchVariables`](https://www.typescriptlang.org/tsconfig/useUnknownInCatchVariables.html)
- [TypeScript FAQ: checked exceptions](https://github.com/microsoft/TypeScript/wiki/faq)
- [TypeScript 5.2 explicit resource management](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-2.html)
- [TypeScript `AsyncIterable` library option](https://www.typescriptlang.org/tsconfig/lib.html)
- [typescript-eslint typed linting](https://typescript-eslint.io/getting-started/typed-linting/)
- [`no-floating-promises` rule](https://typescript-eslint.io/rules/no-floating-promises/)
- [`use-unknown-in-catch-callback-variable` rule](https://typescript-eslint.io/rules/use-unknown-in-catch-callback-variable/)
