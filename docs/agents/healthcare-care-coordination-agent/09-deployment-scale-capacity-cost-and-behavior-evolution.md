# Deployment, Scale, Capacity, Cost, and Behavior Evolution

Scale is the ability to preserve identity, authority, ownership, safety, closure, and recovery as volume and dependency failure increase. It is not the number of concurrent model calls.

## Deployment progression

| Topology | Use | Required properties | Do not add yet |
|---|---|---|---|
| Evaluation harness | Synthetic/de-identified offline tests | Deterministic mocks, full trajectory capture, no production effects | Production PHI or credentials |
| Shadow | Observe real workflow under approval | Read-only, PHI-approved, no patient-visible output, compare to humans | D3 authority |
| Staff pilot | One site/workflow/population | Manual approvals, low concurrency, immediate rollback | Multi-region or multi-agent |
| Reliable single region | Production v1 | Durable HA state, outbox, reconciliation, SLOs, tested backup safety route | Active-active writes |
| Partitioned scale | More tenants/workflows | Admission control, tenant cells, fair queues, workload isolation | Global shared PHI memory |
| Regional resilience | Jurisdiction/data-residency need | Explicit ownership, replicated durable data, fenced failover, recovery drills | Unproven automatic cross-region effects |

Advance only when evidence, not forecast volume, justifies complexity.

## Production topology

~~~mermaid
flowchart TB
    LB[Regional gateway and admission control]
    LB --> ID[Identity and policy services]
    ID --> API[Case command API]
    API --> DB[(Durable case, event, and effect stores)]
    API --> Q[Partitioned workflow queues]
    Q --> W[Workflow workers]
    W --> C[Context assemblers]
    C --> M[Model-provider gateway]
    W --> E[Effect dispatchers]
    E --> AD[Adapter cells]
    AD --> EXT[Healthcare domain systems]
    EXT --> RC[Reconciliation workers]
    RC --> DB
    W --> HQ[Human and clinical queues]
    SAFE[Independent safety routing] --> HQ
    W --> OBS[Redacted telemetry]
    DB --> AUD[Restricted audit access]
~~~

Partition state and queues by tenant and case. Isolate high-risk effect dispatch from model inference. Keep safety routing available when the model path is degraded.

## Capacity model

Estimate each resource separately.

~~~text
arrival_rate = new_cases_per_second + resumed_cases_per_second
model_demand = arrival_rate * model_calls_per_case
tool_demand_i = arrival_rate * calls_to_adapter_i_per_case
human_demand = arrival_rate * human_minutes_per_case
queue_concurrency = arrival_rate * target_step_latency_seconds
reconciliation_demand = D3_effect_rate * unknown_outcome_fraction
~~~

For burst and recovery:

~~~text
drain_time = backlog / (sustainable_completion_rate - incoming_rate)
~~~

The sustainable completion rate must preserve dependency quotas, reconciliation capacity, and staffed human/safety queues. A model tier that can process 1,000 cases per minute is irrelevant if the clinical queue can safely acknowledge 20.

### Capacity dimensions

| Dimension | Measure | Guardrail |
|---|---|---|
| Workflow workers | Runnable steps and lease contention | Autoscale without violating per-case serialization |
| Model gateway | Calls, tokens, latency, quotas | Budget, timeout, provider circuit breaker |
| EHR/HIE reads | Requests, pages, throttles | Cache only approved non-patient artifacts; backpressure |
| D3 effects | Dispatch and unknown rate | Reserve reconciliation capacity |
| Human queues | Arrival, service time, staffing, coverage | Admission and manual overflow |
| Safety queues | Trigger rate and acknowledgment | Independent priority and backup route |
| Communication | Send quotas and delivery lag | Channel rate limit and minimal retry |
| Database/event store | Write/read IOPS, history growth | Partition, archive, and retention |
| Audit/telemetry | Ingest, buffer, export | Audit durability must not depend on optional telemetry |

## Admission control and fairness

Apply admission control before work enters overloaded dependencies:

- per-tenant and global rate limits;
- separate safety, time-sensitive, ordinary, batch, and reconciliation queues;
- weighted fairness so one tenant/payer/outage cannot starve others;
- caps on concurrent cases per patient and workflow;
- backpressure from EHR, scheduler, payer, model, and communication quotas;
- budgets on model calls, tool calls, retries, and wall time;
- load shedding for nonessential summaries and analytics;
- a manual intake policy when capacity is unsafe.

Never shed safety escalation, audit, effect state persistence, or unknown-outcome reconciliation silently.

## Degradation modes

| Dependency/problem | Safe degraded behavior |
|---|---|
| Model provider unavailable | Continue deterministic paths; queue ambiguity or route humans |
| EHR/FHIR unavailable | Stop decisions requiring fresh evidence; preserve visible wait |
| MPI unavailable | No new uncertain patient binding; existing bindings follow freshness policy |
| Policy service unavailable | Deny new reads/effects or use an explicitly approved short-lived fail-safe policy |
| Scheduler unavailable | No booking claims; retain requests and reconcile in-flight effects |
| Payer unavailable | Preserve draft/state; no fabricated status |
| Communication unavailable | Record pending/failed; use approved alternate channels |
| Clinical queue saturated | Activate backup route/admission limits; alert leadership |
| Audit ledger unavailable | Stop governed actions that cannot be durably reconstructed |
| Telemetry exporter unavailable | Buffer within limits; continue only if audit and safety monitoring remain adequate |
| Region unavailable | Fence old workers, restore durable state, reconcile all in-flight D3 effects |

Read-only mode is not automatically safe if stale data could drive a harmful delay. Declare which reads remain meaningful and when they expire.

## Resilience and disaster recovery

Set RTO/RPO by component and hazard. Do not copy one target across:

- case/event state;
- effect/outbox/reconciliation state;
- patient bindings and authority decisions;
- restricted audit;
- domain-policy artifacts;
- model prompts/cache;
- telemetry.

The effect ledger and safety handoff state generally need stronger recovery than recomputable context or metrics.

### Recovery exercise

1. Stop a region with model calls and D3 effects in flight.
2. Fence all old workers and credentials.
3. recover durable case/event/effect state to the approved point;
4. restore policy and behavior versions;
5. classify each effect as succeeded, failed, or unknown;
6. reconcile unknowns before resuming writes;
7. restore safety queues and confirm coverage;
8. resume controlled canary traffic;
9. prove no duplicate effects, lost handoffs, or wrong behavior version;
10. record actual RTO/RPO and corrective actions.

Backups without tested restore and reconciliation are not resilience.

## Cost model

Optimize cost per **valid closed case**, not per model call.

~~~text
case_cost =
  model_input_tokens * input_price
  + model_output_tokens * output_price
  + tool_and_vendor_fees
  + workflow_compute_and_storage
  + communication_cost
  + human_review_minutes * loaded_labor_rate
  + reconciliation_and_incident_cost
~~~

Track:

- deterministic-only versus model-assisted cases;
- tokens per accepted proposal and per closed case;
- invalid proposal/regeneration and abstention cost;
- adapter calls/pages and rate-limit overhead;
- human review and rework;
- unknown-effect and reconciliation labor;
- message/channel cost;
- storage and audit growth;
- safety and incident follow-up;
- cost by workflow, tenant, language/accessibility slice, behavior, and model.

A cheaper model that increases human rework, missed evidence, or safety escalation errors is not cheaper.

## Cost controls

- Route deterministic cases without a model.
- Retrieve only step-specific evidence.
- Use compact structured records, not full transcripts.
- Cache only safe versioned domain artifacts—not patient authority or clinical truth.
- Choose the smallest passing model per closed proposal type.
- Stop on authority, safety, evidence, or budget failure before inference.
- Batch only non-urgent, non-patient-specific background work that preserves ownership.
- Cap retries and model regenerations.
- Prefer one proposal per step.
- Archive/retain according to policy, not indefinitely.

Never reduce cost by weakening identity checks, dropping provenance, suppressing safety routes, delaying reconciliation, or hiding human labor.

## Behavior release manifest

Behavior is the combined system, not only a prompt.

~~~yaml
behavior_manifest:
  behavior_version: immutable-id
  intended_use_version: opaque
  workflow_definition_versions: {}
  model_routes:
    administrative-proposal: {provider: opaque, model: opaque, config: opaque}
  prompt_and_schema_versions: {}
  policy_versions: {}
  adapter_versions: {}
  fhir_and_ig_versions: {}
  terminology_versions: {}
  template_versions: {}
  safety_rule_versions: {}
  evaluation_suite_version: opaque
  approved_slices: []
  authority_ceiling: D3
  release_approvals: [opaque]
  created_at: timestamp
~~~

Store the manifest on every case, proposal, effect, handoff, and trace. Reconstructing only a model name is insufficient.

## Change classification

| Change | Typical review |
|---|---|
| Documentation typo with no runtime effect | Editorial |
| Message wording within approved meaning | Accessibility/privacy/operations regression |
| Prompt or output schema | Full proposal and clinical-boundary regression |
| Model/version/routing | Repeated trials, slice comparison, safety/authority gates |
| Workflow transition or timer | State/recovery/hazard and operational review |
| D-tier or preauthorization | Security/privacy/clinical/legal governance |
| Patient matching or proxy logic | Identity/privacy/safety review |
| Adapter/profile/terminology | Conformance, provenance, migration, reconciliation |
| Safety trigger/route/timing | Clinical-safety owner and operational capacity |
| New population, jurisdiction, channel, or intended use | Full scope, legal, safety, accessibility, regulatory review |

No component updates itself in production based on live patient conversations.

## Release pipeline

~~~mermaid
flowchart LR
    C[Proposed behavior change] --> I[Impact and hazard analysis]
    I --> O[Offline contract and repeated-trial evaluation]
    O --> R[Clinical, privacy, security, accessibility, operations review]
    R --> S[Shadow]
    S --> P[Staff pilot]
    P --> K[Restricted canary]
    K --> M[Monitored expansion]
    M --> F[Full approved scope]
    K -->|hard gate or drift| B[Rollback and reconcile]
    M -->|hard gate or drift| B
    B --> C
~~~

Promotion requires an immutable evidence bundle. Rollback must be possible per action, adapter, tenant, workflow, and behavior version.

The canary unit is the **whole behavior bundle**, not a model name. It includes workflow/timers, model route and configuration, prompts/schemas, policy, adapter manifests, FHIR/HL7/X12 profiles and mappings, terminology, templates, safety rules, parser versions and UI/channel behavior. Route a case consistently to one bundle; do not independently percentage-split components that can create untested combinations.

Before promotion, prove:

- shadow comparison and repeated trials on the exact bundle and supported slices;
- hard invariants after every trajectory transition, not only final output;
- D3 dispatch, reconciliation and kill switches can be disabled independently from read-only service;
- canary admission is bounded by tenant/workflow/population/channel and human/safety/reconciliation capacity;
- rollback restores the previous immutable bundle while preserving newly created records and evidence;
- every in-flight effect is pinned to its originating contract and classified before replay/migration;
- schema/event/database compatibility supports rollback or a forward recovery is explicitly rehearsed;
- monitoring distinguishes bundle regression from dependency, population-mix and queue-capacity change.

A rollback is complete only after traffic is pinned to the accepted bundle, old workers/credentials are fenced, unknown effects are reconciled, migrated cases pass resume invariants, safety/human queues are assessed, and affected cases receive accountable follow-up. Reverting a prompt or container image alone is not recovery.

### In-flight case policy

For every release declare whether existing cases:

- remain pinned to the old behavior;
- migrate at an explicit safe checkpoint;
- stop for human review;
- continue with old workflow but new security-only control.

Never switch behavior midway through an approval/effect without invalidating and rechecking the bound decision.

## Continuous governed evolution

### Feedback sources

- validator rejection and abstention patterns;
- human corrections with structured reason codes;
- overdue tasks and handoffs;
- unknown/diverged effects;
- accessibility and language failures;
- safety events and near misses;
- privacy/security incidents;
- adapter conformance changes;
- cost/latency/rework drift;
- patient and staff complaints through governed channels.

### Learning workflow

1. de-identify or synthesize a minimal case under governance;
2. retain provenance and incident/hazard mapping;
3. add it to the evaluation corpus;
4. propose a deterministic rule, workflow, adapter, policy, prompt, or model change;
5. prefer a deterministic fix when the failure has a stable rule;
6. run the full affected regression and hazard suite;
7. obtain accountable approval;
8. canary and monitor by slice;
9. update the safety case, runbook, and refresh date.

Do not use a vector database of raw patient conversations as “continuous learning.” Do not fine-tune or send feedback to a provider without approved data governance and a separate release process.

## Scale anti-patterns

- A shared global queue with no tenant or safety isolation.
- One retry policy for reads, writes, and unknown effects.
- Replicating model context rather than durable state.
- Active-active D3 dispatch without a single semantic effect authority and fencing.
- Autoscaling model calls while human or reconciliation queues saturate.
- Caching consent/proxy decisions without expiration and invalidation.
- Sampling away rare hard-gate events.
- Cost dashboards that omit human review and incident recovery.
- Silent model or prompt updates.
- One “FHIR version” claim across vendors and tenants.

## Stage 4–6 operations checklist

- [ ] Topology and complexity match measured load and jurisdiction needs.
- [ ] Queues isolate safety, ordinary, batch, and reconciliation work.
- [ ] Admission control includes human and dependency capacity.
- [ ] Degraded modes and manual fallback are explicit and tested.
- [ ] RTO/RPO are hazard-informed; failover fences workers and reconciles effects.
- [ ] Cost is reported per valid closed case and relevant slice.
- [ ] Behavior manifest pins model, prompt, schema, policy, adapter, profile, terminology, and safety rules.
- [ ] Release pipeline includes impact analysis, offline gates, shadow, canary, rollback, and in-flight policy.
- [ ] Continuous learning uses governed de-identified/synthetic failure evidence.
- [ ] Refresh triggers cover law, source standards, intended use, incidents, and provider behavior.

## Related guides

- Previous: [Observability, Evaluation, Failure Injection, and Incidents](08-observability-evaluation-failure-injection-and-incidents.md)
- Next: [Stage 0–6 Delivery Gates, Schemas, and Checklists](10-stage-0-6-delivery-gates-schemas-and-checklists.md)
- Home: [Healthcare Care Coordination Agent](README.md)
- Canonical: [Scaling, Capacity, and SLOs](../../operations/scaling-capacity-and-slos.md)
