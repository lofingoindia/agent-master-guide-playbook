# Observability, Evaluation, Testing, and Debugging

Mastra supplies traces, logs, metrics, scores, datasets, experiments, and Studio.
Those tools become production evidence only when the application defines
stable IDs, redaction, test datasets, and release gates.

## Evidence layers

~~~mermaid
flowchart LR
    Request[Product request] --> Trace[Trace and spans]
    Trace --> Logs[Correlated logs]
    Trace --> Metrics[Derived metrics]
    Trace --> Live[Live scorers]
    Dataset[Versioned dataset] --> Experiment[Experiment]
    Experiment --> Scores[Case-level scores]
    Scores --> Gate[Release gate]
    Trace --> Debug[Studio or external backend]
    Gate --> Deploy[Versioned deployment]
~~~

Keep one product operation ID linked to trace ID, thread/resource ID, run ID,
workflow step, tool-call ID, and external idempotency key. This makes ambiguous
effects and client reconnects diagnosable.

## Observability configuration

The stable <code>@mastra/observability</code> package configures exporters,
processors, sampling, and logging. Mastra storage can export data to Studio;
other exporters bridge to external systems.

Production baseline:

- set service, environment, version, and region resource attributes;
- include a sensitive-data filter;
- disable raw input/output recording where policy requires;
- sample deliberately by route/risk, not accidentally;
- preserve errors and high-risk effects at a higher sampling rate;
- flush exporters in serverless lifecycles;
- set retention and access control in the backend;
- alert on exporter loss without taking down the product.

Tracing should not log secrets merely because a developer needs prompt
visibility. Store references or hashes for sensitive artifacts.

For hosted telemetry, current code should use
<code>MastraPlatformExporter</code>, which replaced the deprecated
<code>CloudExporter</code> name. On the pinned snapshot it buffers ended spans,
logs, metrics, scores, and feedback, sending at 1,000 events or after five
seconds; model chunk spans are not exported. Consequences:

- call the framework/exporter shutdown path in short-lived and serverless
  processes or the final partial batch can be lost;
- do not expect chunk-level token traces in Platform;
- keep <code>MastraStorageExporter</code> as well when local Studio access or
  trace rehydration for later <code>addFeedback()</code> is required;
- alert on export delay/loss without blocking the agent response.

## Storage choice

DuckDB is useful for local, bounded observability. Postgres can support moderate
shared workloads. ClickHouse is the more suitable current adapter for
high-volume telemetry and metric queries. A composite store can separate
transactional memory/workflows from telemetry.

Capacity-plan:

- spans per agent step and tool;
- stream/custom events;
- full input/output payload size;
- scorer writes;
- dataset/experiment retention;
- query concurrency for Studio and incident response.

## Metrics from traces

Useful service-level indicators:

| Signal | Measure |
|---|---|
| Availability | Product requests ending in a valid terminal state |
| Latency | End-to-end and time-to-first-token p50/p95/p99 |
| Agent control | Steps, repeated calls, stop reason, delegation depth |
| Tools | Calls, failures, timeout, approval wait, ambiguous outcomes |
| Workflows | active/waiting/suspended age, retries, resume conflicts |
| Models | provider/model latency, tokens, errors, fallbacks, cost |
| Memory | retrieval latency, result count, context tokens, observer calls |
| Streams | disconnect, replay success, cache miss, slow consumer |
| Operations | storage/worker/PubSub lag and shutdown drain timeout |

Business success should be measured outside framework traces too. A run that
finished successfully may still give the wrong answer or cause the wrong effect.

## Evals and scorers

The stable <code>@mastra/evals</code> package provides:

- deterministic quick checks for text and tool behavior;
- model-based and statistical scorers;
- live asynchronous scoring;
- step- and trace-level evaluation;
- datasets and experiments, including multi-turn cases;
- gates/verdicts for promotion decisions.

Scorers must be registered and storage configured for score persistence. Live
evaluation uses deterministic trace-ID sampling; if the trace itself is not
retained, the live scorer may not run. Coordinate trace sampling and eval
sampling rather than treating them as independent knobs.

## Deterministic before probabilistic

Prefer deterministic assertions for:

- tool called or forbidden;
- exact schema and business invariants;
- tenant/resource filtering;
- approval before effect;
- no secret in output;
- maximum step/delegation count;
- correct terminal status;
- idempotent retry;
- stream protocol sequence.

Use model judges for qualities that resist deterministic checks, such as
helpfulness, groundedness, or tone. Calibrate judges against human-labeled cases,
pin the judge profile, and inspect disagreement by slice.

## Dataset design

A production dataset should include:

- normal representative traffic;
- high-value and high-risk cases;
- long context and memory conflicts;
- malformed tool/provider responses;
- prompt injection in user and tool content;
- multilingual and accessibility cases where relevant;
- approval/decline/expiry;
- timeout, retry, disconnect, resume, and process loss;
- tenant isolation;
- known historical regressions.

Version the input, expected contract, evaluator, model, prompt, tools, and
workflow. Never overwrite the only evidence for a previous release.

## Experiment gates

Do not gate only on an average score. Require:

- zero critical authorization or secret-leak failures;
- every must-pass case succeeds;
- no material regression in important slices;
- confidence intervals or repeated runs for stochastic metrics;
- latency and cost within budget;
- tool/effect behavior unchanged or explicitly approved.

Mastra's <code>runEvals</code> and experiment tools can execute the suite, but
the product owns thresholds and risk acceptance.

## Safe tool testing

Dataset experiments can use tool mocks and report mock usage. Fail closed if an
effecting tool lacks a mock. Unit tests should fake model, storage, tool, and
clock boundaries; integration tests should use real selected adapters; a small
canary suite should exercise real providers with restricted credentials.

Never let a CI evaluation send production email, spend credits, mutate customer
records, or call a production MCP server.

## Debugging workflow

When a run fails:

1. start from product operation and version profile;
2. find the trace and canonical run state;
3. identify whether failure was transport, model, tool, workflow, storage, or
   policy;
4. inspect redacted input/context and processor order;
5. check effect receipts before retrying;
6. reproduce with a deterministic fake;
7. add the minimal case to the regression dataset;
8. verify the fix across stream, resume, and deployed topology.

Studio is useful for visual inspection and local debugging. It is not a
substitute for a product control plane, durable run index, or restricted
operator workflow.

## Common observability failures

| Failure | Consequence | Control |
|---|---|---|
| Record all prompts by default | Data exposure and retention cost | Field allowlists and sensitive-data filter |
| Sample before live eval selection | Critical cases never scored | Coordinate sampling policy |
| Only aggregate scores | Critical slice hidden by average | Case/slice gates |
| No version attributes | Regression cannot be bisected | Full profile on every trace/experiment |
| Synchronous exporter on hot path | Tail latency/outage coupling | Async bounded export and flush |
| Studio as run database | Unstable product recovery | Application-owned operation state |

## Checklist

- [ ] Product, trace, run, tool, and effect IDs correlate.
- [ ] Sensitive data is filtered before export.
- [ ] Sampling preserves required live-eval cases.
- [ ] Storage and retention match telemetry volume.
- [ ] Deterministic invariants precede model judges.
- [ ] Dataset covers failures, isolation, and historical regressions.
- [ ] Gates operate per critical case and slice.
- [ ] CI tools are mocked and fail closed.
- [ ] Every release records the complete version profile.

## Primary sources

- [Observability documentation source](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/observability)
- [Evaluation documentation source](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/evals)
- [Dataset and experiment documentation source](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/datasets)
- [Observability packages](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/observability)
- [Evaluation package](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/packages/evals)
- [Mastra Platform exporter documentation source](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/mastra-platform/observability.mdx)
- [Canonical observability and tracing guide](../../evaluation/observability-and-tracing.md)
- [Canonical evaluation-driven development guide](../../evaluation/evaluation-driven-development.md)
