# LangGraph Testing, Debugging, Observability, and Evaluation

**Research date:** 2026-08-31
**Status:** Research-backed verification guide

## Test four different systems

Do not collapse all verification into an LLM answer score.

| Layer | Question | Main technique |
|---|---|---|
| Node logic | Does one function transform valid/invalid state correctly? | Unit tests with fake context/clients |
| Graph semantics | Do routes, reducers, interrupts, and retries behave? | Compiled graph with fresh test checkpointer |
| Recovery/operations | Does the deployment survive crashes, queues, reconnects, migrations? | Integration and fault-injection tests |
| Agent quality | Does the system choose useful actions and produce acceptable outcomes? | Offline/online evaluation with datasets |

## Unit and graph tests

Compile a fresh graph and checkpointer per test to prevent thread-state leakage. Use deterministic fake models/tools for graph semantics. Assert:

- exact node updates;
- route destination;
- parallel merge under reordered completion;
- termination and remaining-step behavior;
- error/retry classification;
- interrupt payload and resume;
- state/output schema;
- state history and task writes where part of the contract.

Use real provider tests separately and label them for cost, nondeterminism, and rate limits.

## Golden checkpoint fixtures

Keep sanitized fixtures from the oldest supported schema/package versions:

```text
completed thread
interrupted root node
interrupt inside subgraph
parallel superstep with pending writes
state fork created by update_state
long message/evidence state
failed node with retry metadata
```

For every release candidate:

1. load the fixture using production serializer/saver code;
2. inspect current state;
3. resume or replay;
4. assert node/effect invocation counts;
5. compare final typed state;
6. verify no unauthorized field appears in output or trace.

## Failure injection

Inject failure at:

- before/after model request;
- before/after tool send and effect ledger commit;
- before/after task write and full checkpoint;
- worker lease acquisition and completion;
- interrupt save and resume request;
- SSE disconnect and Redis restart;
- database failover;
- rolling deployment across graph versions.

Track both graph result and external-world result. “Test passed because the graph completed” is insufficient if a payment, email, or ticket duplicated.

## Trace schema

Record consistent metadata:

- environment, deployment, graph release, package lock hash;
- assistant, thread, run, checkpoint, and logical work-item IDs;
- tenant-safe correlation ID;
- node/task/subgraph name and attempt;
- model/provider/prompt/schema version;
- tool/effect type and operation ID;
- interrupt/proposal ID;
- latency, usage, cost, input/output byte counts;
- outcome and typed error class.

Keep secrets, raw credentials, hidden state, and unnecessary personal data out of traces. Sampling must be deterministic for high-risk/error flows and probabilistic only for ordinary success traffic.

## LangSmith without conflation

LangSmith can automatically trace LangChain/LangGraph activity, accept metadata/tags, connect distributed traces, and run evaluations. It is optional for library execution. A graph must continue correctly if trace export is slow or unavailable.

For Agent Server distributed tracing, both client and graph need to opt into propagation. Verify cross-service parentage and ensure client-provided metadata cannot overwrite trusted tenant or policy fields.

## Evaluation design

LangSmith distinguishes offline experiments on datasets from online evaluation over production traces. For an agent graph, evaluate at least:

| Evaluation | Example |
|---|---|
| Final outcome | Correct resolution with required evidence |
| Single decision | Correct route/tool/approval requirement |
| Trajectory | Required tools used; forbidden/redundant steps absent |
| State invariant | No budget increase, proposal precedes effect |
| Recovery invariant | Same logical effect after crash/resume |
| Safety/security | Cross-tenant and prompt-injection probes denied |
| Efficiency | Tokens, calls, latency, cost under thresholds |

Exact trajectory matching is often too strict because several valid paths can exist. Prefer semantic constraints, required/forbidden actions, bounded steps, and state/effect invariants.

## Dataset construction

Include:

- curated normal and boundary cases;
- historical failures with sensitive data removed;
- adversarial tool arguments and prompt injection;
- stale/missing/contradictory memory;
- transient and permanent dependency failures;
- ambiguous effect outcomes;
- interrupted and migrated threads;
- long and multilingual inputs where supported;
- workload distributions, not only demos.

Version examples, evaluators, judges, reference policies, and dataset splits. Human calibration is required for LLM judges.

## Debugging workflow

1. Locate thread/run/checkpoint and graph release.
2. Inspect state values, next tasks, errors, interrupts, and namespace.
3. Correlate model/tool/effect IDs with traces and domain receipts.
4. Determine whether this is a bad decision, bad state, replay, transport, or external-service failure.
5. Reproduce from a sanitized checkpoint with effects disabled.
6. Add the case to a regression dataset and recovery fixture.
7. Repair through a versioned state/effect workflow, not an undocumented database edit.

Studio and time travel improve inspection, but replay can call models and APIs again. Use safe mocks or effect guards.

## Release gate

- [ ] Unit, graph, provider-contract, persistence, and deployment tests pass.
- [ ] Oldest supported checkpoints load and resume.
- [ ] Crash matrix has no unhandled duplicate or unknown effect.
- [ ] Offline quality and efficiency meet thresholds with confidence intervals/repetitions.
- [ ] Security/adversarial suite passes.
- [ ] Trace redaction and sampling are verified.
- [ ] Canary has rollback criteria based on outcome and recovery metrics.
- [ ] New production failures feed a dataset and owner backlog.

## Sources

- [LangGraph testing](https://docs.langchain.com/oss/python/langgraph/test)
- [Persistence/state history](https://docs.langchain.com/oss/python/langgraph/persistence)
- [LangSmith evaluation](https://docs.langchain.com/langsmith/evaluation)
- [Agent evaluation approaches](https://docs.langchain.com/langsmith/evaluation-approaches)
- [Trace LangChain applications](https://docs.langchain.com/langsmith/trace-with-langchain)
- [Distributed tracing with Agent Server](https://docs.langchain.com/langsmith/agent-server-distributed-tracing)
- [Trace sampling](https://docs.langchain.com/langsmith/sample-traces)

Next: [failure modes, migrations, and versioning](failure-modes-migrations-and-versioning.md).
