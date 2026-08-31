# Reliability, Observability, Scaling, and Operations

> **Status:** Research-backed production blueprint  
> **Last researched:** 2026-08-31  
> **Scope:** Durable state, recovery, audit, telemetry, SLOs, capacity, cost, deployment, release, and incident operations.  
> **Section index:** [Security investigation and triage agent](README.md)

Security investigations outlive web requests and often outlive worker processes. Preserve the case and evidence state needed to resume, but avoid making every alert a heavyweight workflow. Begin with a queue-backed, idempotent read-only service; add a durable workflow runtime when human waits, long collection jobs, or mutating actions justify it.

## Reliability model

The system must distinguish:

- **delivery:** the alert or command reached the service;
- **processing:** a worker claimed and evaluated it;
- **proposal:** the model produced a valid structured result;
- **decision:** policy or an accountable human decided;
- **attempt:** an external action was submitted;
- **observed outcome:** the authoritative target reflects success or failure;
- **case completion:** required work and verification are done.

Acknowledging delivery is not completing an investigation. A model's final sentence is not an observed effect.

## Case, command, event, and effect contracts

Reuse the repository's canonical six-record boundary even when one database stores several records. The effect boundary is expanded below into intent/attempt and receipt because security actions require explicit reconciliation:

| Record | Meaning | May be retried? | Never treat it as |
|---|---|---:|---|
| Command or intent | A request to investigate, query, transition, or attempt one canonical action | Yes, under one stable command/intent ID | Proof it was accepted or completed |
| Durable state | Current materialized case/workflow view at a known version | Rebuilt, not replayed as a side effect | Complete history or immutable evidence |
| Domain event | An immutable statement that the application accepted a transition or observation | Redelivery yes; append logically once | An external target effect unless independently observed |
| Effect intent and attempt | A sealed action plus any submission to a source, case system, notification channel, or response actuator | Only under semantic idempotency and error policy | Success merely because transport returned `2xx` |
| Effect receipt | Native operation ID plus authoritative observed outcome, coverage, and reconciliation status | Observation may repeat safely | Model opinion, workflow checkpoint, or trace span |
| Delivery event or projection | What an analyst UI, case subscriber, or downstream workflow should render | Replay/coalescing only under an explicit cursor and gap contract | Authoritative case truth, evidence, or effect outcome |
| Telemetry | Trace, log, and metric used to operate or debug the platform | Sampling and retries are operational concerns | Durable audit, custody, case state, or authority |

Use expected case version on commands and append one event per accepted transition. Project events into current state; publish downstream notifications through an outbox or equivalent recoverable handoff. Model outputs first become validated proposals, not domain events that assert incident truth.

~~~yaml
effect_receipt:
  effect_id: eff_91
  intent_id: case_781:endpoint.isolate:device_44:plan_3
  case_id: case_781
  case_version_authorized: 22
  action: endpoint.isolate
  target: edr:tenant_acme:device:44
  canonical_arguments_digest: sha256:...
  native_operation_id: machine-action-8841
  submit_status: accepted
  outcome: observed_succeeded
  observed_at: 2026-08-31T04:26:18Z
  observed_state_ref: restricted://effects/eff_91/state
  approval_id: apr_52
  policy_version: soc-response/19
  expires_at: 2026-08-31T04:55:00Z
  rollback_effect_id: null
~~~

If a worker crashes after submit but before the receipt is persisted, recover the intent, locate the native operation or desired state, and append the reconciled receipt. Do not re-run model reasoning to invent a replacement action.

## Durable state boundaries

| State | Persist | Why |
|---|---:|---|
| Alert envelope and raw reference | Yes, before inference | Do not lose security input |
| Deduplication key and correlation result | Yes | Safe redelivery |
| Case version and workflow transition | Yes | Optimistic concurrency and audit |
| Query proposal and policy decision | Yes | Explain access and denials |
| Tool result reference and coverage | Yes | Resume without repeating expensive reads |
| Model result and version metadata | Yes, under retention policy | Reproduce case proposal |
| Hidden chain-of-thought | No | Not needed for correctness; sensitive and unstable |
| Approval and expiry | Yes, immutable | Effect authorization |
| Effect attempt and observed outcome | Yes | Reconcile ambiguous operations |
| In-memory caches | Rebuildable only | Never source of truth |

Store large evidence and tool results by reference. Workflow histories, queue messages, and traces have size and retention limits and are not evidence vaults.

## Workflow model

~~~mermaid
stateDiagram-v2
    [*] --> Queued
    Queued --> Preparing
    Preparing --> Investigating
    Investigating --> AwaitingAnalyst
    Investigating --> NeedsSpecialist
    Investigating --> FailedRetryable
    FailedRetryable --> Investigating
    AwaitingAnalyst --> Investigating: request more evidence
    AwaitingAnalyst --> ResponseProposed
    AwaitingAnalyst --> Disposed
    ResponseProposed --> AwaitingApproval
    AwaitingApproval --> Executing
    AwaitingApproval --> Expired
    Executing --> Reconciling
    Reconciling --> AwaitingAnalyst
    NeedsSpecialist --> AwaitingAnalyst
    Disposed --> [*]
~~~

Every transition should include expected prior version, actor, reason, timestamp, policy version, and causal run ID.

## Delivery and deduplication

Assume at-least-once delivery unless the infrastructure proves otherwise.

### Alert path

1. Authenticate and validate.
2. Begin transaction.
3. Insert delivery ID.
4. Insert or locate logical event ID.
5. Open/correlate case if not already done.
6. Commit durable state.
7. Acknowledge transport.
8. Enqueue investigation with stable case/run identity.

If the transport cannot participate in the database transaction, use an inbox/outbox pattern or equivalent reconciliation.

### Action path

Persist action intent and semantic idempotency key before submit. After timeout, reconcile with the target. Never infer from queue acknowledgement or workflow replay that an external effect did or did not occur.

## Retry policy

Classify errors:

| Error | Example | Retry |
|---|---|---|
| Transient before execution | Connection could not be established | Bounded retry with backoff/jitter |
| Ambiguous after submit | Connection dropped after request | Reconcile before retry |
| Rate limited | Provider 429 with retry guidance | Respect server delay; admission control |
| Permanent input | Invalid schema or canonical target | No retry; quarantine or analyst correction |
| Authorization | Denied scope, expired approval | No automatic retry; require new state/authority |
| Source coverage | Retention gap or sensor offline | Alternate source or explicit gap |
| Model output invalid | Schema/citation failure | One bounded repair or fallback, then abstain |
| Policy denial | Action not allowed | Final denial; alert if suspicious |
| Resource exhausted | Context, rows, bytes, queue | Narrow/defer; do not unbounded retry |

Cap retry attempts and total deadline. Use jitter to avoid synchronized bursts. Record each attempt but present one logical operation to analysts.

## Crash and replay safety

Test worker death:

- before and after alert acknowledgement;
- before and after case transition;
- during source pagination;
- after a model response but before case write;
- before and after action submit;
- during approval wait;
- during rollback or expiry.

For durable workflow runtimes:

- keep network, model, clock, randomness, and filesystem access in activities or equivalent effect boundaries;
- persist activity outcomes;
- make activity effects idempotent;
- run replay tests against historical workflow histories before deployment;
- version workflow definitions while old cases remain active;
- prevent model calls from repeating merely because deterministic workflow code replayed.

Temporal's event history and replay model are one implementation option; the principles apply to other durable runtimes.

## Backpressure and admission

Alert bursts are normal during incidents and sensor failures. Do not let the model layer become the admission controller.

Priority policy can include:

- incident/tenant emergency mode;
- asset or identity criticality;
- source confidence and detection family;
- correlated campaign/case;
- age and contractual SLA;
- expected investigation cost;
- starvation prevention.

Use:

- per-tenant and global queue quotas;
- separate lanes for urgent, normal, bulk/backfill, and evaluation work;
- bounded worker concurrency per source and model provider;
- circuit breakers for unhealthy connectors;
- maximum in-flight queries and bytes;
- load shedding that preserves alerts and defers enrichment;
- deterministic fallback triage when inference is degraded.

Never auto-close the low-priority queue to protect model cost.

## Scaling unit

Estimate capacity from investigations, not alert count alone:

~~~text
daily work =
  alerts
  × fraction requiring model
  × average model turns
  × average tool calls per turn
  × retry/fallback multiplier
~~~

Stratify by alert family because a single identity lookup and a multi-source incident investigation have different cost and latency.

### Cost ledger

Record per run:

- input, cached-input, output, and reasoning-token accounting where provider exposes it;
- model and embedding cost;
- tool query cost and scanned bytes;
- storage and evidence egress;
- retry and fallback cost;
- analyst review and correction time;
- avoided or added downstream work.

Cost per alert is misleading if the system shifts false negatives or analyst burden. Track cost per correctly handled case and per material incident found.

### Cost controls

- deterministic filters for malformed and exact duplicate deliveries;
- prefetch only high-value authoritative context;
- stable prompt/tool prefixes when provider caching is measured;
- compact evidence excerpts with resolvable citations;
- route routine cases to a validated lower-cost model;
- stop after marginal query value falls below cost/risk;
- cache CTI by provider policy and freshness, not indefinitely;
- sample full content in traces only for approved diagnostic cases;
- batch offline backfills, not interactive incident decisions.

## Observability architecture

~~~mermaid
flowchart LR
    RUN["Run and workflow events"] --> AUDIT["Application audit store"]
    QUERY["Query decisions/results"] --> AUDIT
    EFFECT["Approval/effect ledger"] --> AUDIT
    AUDIT --> CASE["Case reconstruction"]
    RUN --> OTEL["OTel traces/metrics/logs"]
    QUERY --> OTEL
    EFFECT --> OTEL
    OTEL --> OPS["Operational dashboards and alerts"]
    AUDIT --> EVAL["Eval and incident review"]
~~~

OpenTelemetry is useful for transport and cross-service correlation. Its GenAI conventions remain Development as of the research date, and open design questions remain around causal tool links and workflow grouping. Maintain an application-owned event contract and map stable fields onto OTel.

### Audit event envelope

~~~yaml
audit_event:
  schema: security-investigation-audit/v1
  event_id: aud_91
  event_name: action.policy_decided
  occurred_at: 2026-08-31T04:24:09Z
  tenant_id: tenant_acme
  case_id: case_781
  run_id: run_71
  trace_id: 4bf92f3577b34da6a3ce929d0e0e4736
  actor:
    user_id: analyst_17
    workload_id: response-policy
  subject:
    type: endpoint
    id: edr:tenant_acme:device:44
  decision:
    outcome: require_approval
    policy_version: soc-response/19
    reason_codes: [critical_asset, isolation_requested]
  content_refs:
    - restricted://audit-payload/aud_91
  integrity:
    previous_event_digest: sha256:...
    event_digest: sha256:...
~~~

Trace IDs are correlation helpers, not security authority. W3C Trace Context warns about information exposure and denial-of-service risks; validate incoming headers and restart trace context at trust boundaries where appropriate.

## Minimum events

- alert.received, alert.deduplicated, alert.correlated;
- evidence.registered, evidence.integrity_checked, evidence.accessed;
- case.transitioned, claim.proposed, claim.confirmed, claim.superseded;
- run.started, context.compiled, model.called, model.result_validated, run.stopped;
- query.proposed, query.denied, query.started, query.completed;
- approval.requested, approval.granted, approval.rejected, approval.expired;
- action.proposed, action.authorized, action.submitted, action.reconciled, action.rolled_back;
- policy.changed, prompt.changed, tool.changed, model.changed;
- credential.issued, credential.revoked;
- retention.hold_applied, retention.deletion_started, retention.deletion_verified;
- kill_switch.changed.

Do not place raw evidence, secrets, full prompts, or full tool results in ordinary event attributes. Use restricted references and data-classification metadata.

Every event type needs a schema owner, compatibility policy, required actor/tenant/case/causation fields, retention class, sensitivity classification, and replay test. Consumers must tolerate additive fields and reject authority-sensitive unknown enum values rather than silently mapping them to a permissive default.

## Metrics and SLOs

### Service health

- alert intake success and age;
- queue depth/age by priority and tenant;
- investigation completion, timeout, and retry rates;
- model/provider latency and error distribution;
- tool latency, denial, timeout, coverage-gap, and truncation rates;
- worker crash/recovery and workflow replay failures;
- action unknown-outcome and reconciliation age;
- approval wait and expiry;
- evidence integrity failures;
- deletion and legal-hold reconciliation.

### Quality and safety

- true-positive recall and false-negative rate by alert family/severity;
- false-positive escalation and unnecessary-action rates;
- unsupported claim and invalid citation rates;
- confidence calibration and abstention quality;
- analyst confirm, correct, reopen, and override rates;
- queries per correct case and non-progress loops;
- cross-tenant denial and suspicious scope-escalation attempts;
- injection attack success and benign hard-negative block rate;
- time to acknowledged, investigated, decided, contained, and verified.

### Example SLO form

Use workload-specific values:

~~~yaml
slo:
  name: urgent-alert-investigation-start
  indicator: time_from_intake_commit_to_first_authorized_query
  objective: 99_percent_within_120_seconds
  scope:
    priority: urgent
  exclusions:
    - tenant_integration_disabled
  never_exclude:
    - model_provider_outage
    - internal_queue_overload
~~~

Do not exclude the failures the architecture is supposed to tolerate.

## Deployment topology

Separate:

- public/integration intake;
- tenant-aware case/evidence services;
- model gateway;
- read-only query workers;
- static artifact parsers;
- dynamic malware-analysis environment;
- policy/approval control plane;
- response executor;
- telemetry pipeline;
- offline eval.

Network rules should follow required flows, not flat internal trust. Keep evidence vault and response credentials unreachable from the model runtime. Use private provider connectivity where it materially improves policy, while still enforcing application authorization.

### Environment promotion

1. Offline fixtures and synthetic sources.
2. Non-production integration.
3. Production shadow reads with no analyst-visible output.
4. Analyst-visible advisory.
5. Bounded read-only queries.
6. Case writes.
7. Separately gated constrained action.

Keep production evidence out of lower environments unless explicitly de-identified and authorized.

## Release management

Version independently:

- model and provider endpoint;
- prompt/instructions;
- structured-output schema;
- tool catalog and adapter;
- query templates/compilers;
- context selection policy;
- case and audit schemas;
- authorization policy;
- playbooks and action schemas;
- OCSF/STIX/ATT&CK/Sigma content snapshots;
- parser and sandbox images;
- eval dataset, graders, and thresholds.

A change to any of these can change behavior. Generate a release manifest, run replay/compatibility/eval gates, deploy by tenant or alert-family canary, and retain immediate rollback.

Treat the complete release manifest as a **behavior bundle**. A case must identify the exact model/provider, prompt, structured-output schema, context and compaction policy, memory/retrieval configuration, tool catalog, adapter and query-template versions, case/event/effect schemas, policy/playbook content, parser images, and evaluator bundle. Rolling back only the model while leaving a changed adapter, policy, or compactor in place does not restore prior behavior.

### Supply chain

- pin dependencies and container images;
- verify signatures/provenance where available;
- generate and monitor an SBOM;
- scan parser/model gateway dependencies;
- isolate build from production credentials;
- review MCP/tool/server or connector metadata changes;
- block automatic installation of model-suggested packages;
- maintain emergency disable and rollback for each connector.

## Operational incidents

Maintain runbooks for:

- model provider outage or degradation;
- connector compromise or schema drift;
- prompt/tool configuration compromise;
- cross-tenant access attempt or leak;
- evidence integrity failure;
- response executor compromise;
- queue storm and cost runaway;
- widespread wrong disposition;
- prompt-injection campaign;
- stale approvals or duplicate effects;
- retention/deletion failure;
- compromised signing key or workload identity.

During agent compromise, preserve its audit/configuration state, revoke credentials, stop mutating actions, isolate affected connectors, and continue alert intake through a deterministic fallback.

## Failure-injection matrix

| Injection | Expected invariant |
|---|---|
| Kill worker after model returns | One case result or visible retry; no lost alert |
| Duplicate alert 100 times | One logical event/case path with redelivery count |
| Drop connection after action submit | Outcome unknown then reconcile; no blind duplicate |
| Delay approval past target change | Approval expires or revalidation denies |
| Return truncated source page | Coverage incomplete; verdict does not treat absence as benign |
| Provider 429 burst | Backpressure and fallback; bounded retry/cost |
| Corrupt evidence copy | Integrity alert, quarantine, no continued analysis |
| Send forged traceparent | Validated/restarted context; no authority inherited |
| Break deletion in one derived store | Deletion not marked complete; reconciliation alert |
| Change workflow code with active cases | Replay/compatibility gate fails before rollout |

## Production checklist

- [ ] Alert acknowledgement follows durable intake, not inference success.
- [ ] Every retryable effect has an idempotency and reconciliation strategy.
- [ ] Queue admission protects priority and tenant fairness without dropping evidence.
- [ ] Application audit can reconstruct a case without full sensitive trace content.
- [ ] Unknown actions, integrity failures, and cross-tenant denials are paging conditions.
- [ ] Capacity and cost models include tool calls, retries, analyst time, and alert-family mix.
- [ ] Release manifests pin every behavior-changing component.
- [ ] Shadow/canary rollout and rollback are tested.
- [ ] Agent-platform incident runbooks have been exercised.

## Related guides

- [Security integrations and adapter qualification](security-integrations-and-adapter-qualification.md)
- [Authority, approvals, and constrained response](authority-approvals-and-constrained-response.md)
- [Evaluation, rollout, and build roadmap](evaluation-rollout-and-build-roadmap.md)
- [Durable execution](../../runtime/durable-execution.md)
- [Observability and tracing](../../evaluation/observability-and-tracing.md)
- [Deployment, release, and incident response](../../operations/deployment-release-and-incident-response.md)

## Selected sources

- [NIST SP 800-61 Rev. 3](https://csrc.nist.gov/pubs/sp/800/61/r3/final)
- [NIST SP 800-92 Rev. 1 initial public draft](https://csrc.nist.gov/pubs/sp/800/92/r1/ipd)
- [OCSF 1.8.0 release](https://github.com/ocsf/ocsf-schema/releases/tag/1.8.0)
- [OpenTelemetry GenAI agent conventions, Development](https://github.com/open-telemetry/semantic-conventions-genai/blob/main/docs/gen-ai/gen-ai-agent-spans.md)
- [W3C Trace Context](https://www.w3.org/TR/trace-context/)
- [Temporal event history](https://github.com/temporalio/documentation/blob/main/docs/encyclopedia/event-history/event-history.mdx)
- [Temporal retry policies](https://github.com/temporalio/documentation/blob/main/docs/encyclopedia/retry-policies.mdx)
- [Google Cloud Eventarc idempotent event-handler guidance](https://cloud.google.com/eventarc/docs/retry-events)
- [NIST SP 800-53 Rev. 5 controls](https://csrc.nist.gov/pubs/sp/800/53/r5/final)
