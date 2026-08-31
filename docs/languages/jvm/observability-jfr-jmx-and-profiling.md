# Observability, JFR, JMX, and Profiling

## Correlate the agent, not just HTTP

Use stable identifiers:

- trace ID for causal request flow;
- run ID across retries/resume;
- attempt ID for one worker execution;
- step ID for model/decision/tool stage;
- effect ID across duplicate tool attempts;
- provider request ID for support/reconciliation.

Do not put raw prompts, tool arguments, outputs, memory text, secrets, or user identifiers into metric labels or default span attributes.

## Span model

~~~mermaid
flowchart TD
    R[agent.run] --> A[agent.attempt]
    A --> M[model.call]
    A --> T[tool.effect]
    A --> P[state.persist]
    T --> S[sandbox/process]
    M --> H[HTTP]
~~~

Create manual spans for run semantics; auto-instrumentation supplies HTTP, database, messaging, and framework edges. Record model/provider, operation, bounded token/usage counts, retry number, result class, and duration. Treat GenAI semantic conventions as development-status and isolate attribute mapping behind one telemetry adapter.

Keep the internal contract stable even if OpenTelemetry naming changes:

| Internal field | Span/log placement | Cardinality/content rule |
|---|---|---|
| run/attempt/effect IDs | span attribute and structured log | searchable IDs; never metric labels |
| model/provider/operation | span and metric dimension | allowlisted, bounded values |
| tool class/version | span and metric dimension | registered name, not model-supplied text |
| input/output tokens and cost estimate | numeric span/metric | mark absent/estimated/provider-reported |
| argument/result | secure artifact reference or digest | content capture off by default |
| outcome | span status plus domain result attribute | distinguish rejected, failed, cancelled, unknown |

Do not map domain rejection to transport error blindly: a policy denial or insufficient funds can be a successful tool invocation with a rejected domain outcome. Conversely, a span marked `OK` does not prove the effect was durably recorded.

## Metrics

Useful low-cardinality metrics:

- admitted, active, queued, completed, cancelled, failed runs;
- end-to-end and queue latency histograms;
- model call latency, tokens/usage, throttles, retries;
- tool latency and outcome by registered tool class;
- provider/tool semaphore wait time and utilization;
- stream buffered bytes/events and overflow cancellations;
- unknown effects awaiting reconciliation;
- worker lease conflicts and replay/history size;
- JVM heap/native memory, GC, CPU, threads, descriptors.

Tenant/run IDs belong in logs/traces, not metric labels.

## Logs and audit

Structured operational logs answer what failed. Audit events answer who authorized and what effect occurred. They have different access and retention policies. Log hashes, sizes, classifications, and secure references instead of sensitive content. Sampling must never remove required audit records.

## OpenTelemetry posture

OpenTelemetry Java traces, metrics, and logs are stable on the research date. Kotlin-specific OTel status remains development, but Kotlin/JVM services can use the stable Java API/SDK and Java agent. Pin a patched instrumentation-agent release: Java instrumentation has had security fixes, and attaching an agent changes application bytecode and supply-chain exposure.

Propagate context deliberately through executors, virtual threads, coroutines, queue messages, and workflow activities. Test it; ThreadLocal-based propagation can be lost at custom bridges.

At an asynchronous boundary, capture and restore only correlation context—not credentials, mutable request objects, or a giant `ThreadLocal` graph. Queue messages should carry a validated W3C trace context plus application run/attempt IDs; consumers start a new processing span linked/parented according to the messaging contract. Durable replay must not recreate old external spans as if work happened twice: record replay state and create telemetry for the current attempt.

## JFR and JMX

JFR is the primary low-overhead runtime evidence source. Start with predefined <code>default</code> or <code>profile</code> settings, then enable targeted events. Enabling every event can produce enormous data. Keep a rolling recording with controlled size/age so pre-incident evidence survives.

Use:

- allocation/object statistics for retention;
- socket/file events for blocking;
- GC and CPU samples;
- monitor/park events for contention;
- virtual-thread pin/submit events;
- custom low-volume events for queue saturation or run-state anomalies.

Start with a bounded rolling recording, for example:

~~~text
-XX:StartFlightRecording=name=agent,settings=default,maxage=30m,maxsize=512m,dumponexit=true,filename=/var/diagnostics/agent.jfr
~~~

The writable path must have a quota and restricted access. Use `jcmd <pid> JFR.check` and `JFR.dump` in the incident runbook, and parse large recordings as a stream instead of loading all events into heap. Custom agent JFR events should carry IDs and bounded numeric state, never raw prompts or tool output.

JMX platform MXBeans expose memory, GC, threads, class loading, and OS metrics. Remote JMX is a privileged management plane; Oracle warns insecure remote configurations expose the JVM. Prefer local access or authenticated TLS behind a restricted network. Never publish it directly.

## Incident workflow

1. preserve run/effect IDs and provider request IDs;
2. capture bounded thread dump and JFR window;
3. inspect queue/bulkhead saturation before blaming GC;
4. compare heap, native memory, descriptors, and child processes;
5. identify unknown effects and pause unsafe retries;
6. retain sanitized provider/tool envelopes;
7. turn the incident into a deterministic or fault-injection test.

## Checklist

- [ ] Run, attempt, step, effect, and provider IDs correlate.
- [ ] Business spans exist in addition to auto-instrumentation.
- [ ] Sensitive GenAI content is disabled by default.
- [ ] Metrics have bounded cardinality.
- [ ] Audit is durable and independent of log sampling.
- [ ] Rolling JFR is sized and tested.
- [ ] JMX is local or authenticated/TLS-restricted.
- [ ] Telemetry agent versions are pinned and patched.

## Sources

- [OpenTelemetry Java](https://opentelemetry.io/docs/languages/java/)
- [OpenTelemetry Java agent](https://opentelemetry.io/docs/zero-code/java/agent/)
- [OpenTelemetry GenAI attributes](https://opentelemetry.io/docs/specs/semconv/registry/attributes/gen-ai/)
- [JFR programmer guide](https://docs.oracle.com/en/java/javase/25/jfapi/index.html)
- [JFR configuration](https://docs.oracle.com/en/java/javase/25/jfapi/configuration.html)
- [Java management package](https://docs.oracle.com/en/java/javase/25/docs/api/java.management/java/lang/management/package-summary.html)
- [JFR recording-file parsing](https://docs.oracle.com/en/java/javase/25/jfapi/parsing-recording-file.html)
- [JMX remote connector security](https://docs.oracle.com/en/java/javase/25/jmx/using-jmx-connectors-manage-resources-remotely.html)
