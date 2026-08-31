# Business Intelligence Monitoring and Decision Operations Agent

**Status:** production blueprint  
**Research date:** 2026-08-31  
**Maturity:** architecture reference; product-specific adapters and jurisdictional controls require local validation  
**Refresh:** re-check volatile BI-platform, AI-governance, OpenTelemetry, and privacy sources within 90 days; stable statistical and provenance sources within 180 days  
**Scope:** persistent governed metric watches, trustworthy change detection, alert-to-decision workflows, acknowledgement, follow-through, and verified outcome feedback  
**Evidence packet:** [research packet](../../research/packets/business-intelligence-monitoring-agent-blueprint.md)

## Executive decision

Build this agent only when the organization has recurring business questions whose answers must be evaluated continuously and connected to accountable action. Most of the system should be deterministic: scheduling, semantic queries, freshness and quality gates, detector execution, state transitions, policy, approvals, delivery, and reconciliation. A model is justified only for bounded triage tasks such as selecting approved drill-downs, correlating cited evidence, summarizing uncertainty, and drafting a handoff.

If a fixed dashboard alert and an owned runbook solve the problem, use them. If the work is a one-off investigation, route it to the [Analytics Agent](../analytics-agent/README.md). If the data product itself is late or broken, route it to the [Data Pipeline Operations Agent](../data-pipeline-operations-agent/README.md). If the need is executive agenda construction, cross-domain prioritization, briefing, or commitment coordination, route the finalized evidence packet to the [Executive Operations Agent](../executive-operations-agent/README.md); that agent does not own the watch, metric truth, or detector decision. Do not add an agent merely to rename ordinary threshold automation.

> **Core principle:** an anomaly is an observation requiring interpretation, not a cause, a decision, or permission to act. Metric owners define meaning and thresholds. Accountable operators decide and execute high-impact action. The agent preserves evidence and follow-through between them.

## Ownership boundary

| Capability | Owner | This agent's role |
|---|---|---|
| Persistent watch definitions, evaluation cursors, alert cases, acknowledgement, escalation, and outcome state | BI monitoring agent | Owns the durable operational record |
| Metric definition, allowed dimensions, ownership, quality contract, and semantic change approval | Metric owner and semantic layer | Consumes only an approved, versioned contract |
| Bounded ad hoc analysis, causal investigation, experiments, and new metric discovery | [Analytics Agent](../analytics-agent/README.md) | Opens a cited investigation request and receives its result |
| Executive briefing, agenda, cross-domain priority, and commitment coordination | [Executive Operations Agent](../executive-operations-agent/README.md) | Publishes a finalized evidence/status packet; does not infer executive priorities or commitments |
| Pipeline freshness, schema, lineage, replay, and data-product recovery | [Data Pipeline Operations Agent](../data-pipeline-operations-agent/README.md) | Consumes health evidence and routes a data incident |
| Source-system mutation or operational remediation | Accountable business or service operator | May create an approved work item; never performs the source mutation |
| Threshold, escalation, suppression, and action authority | Named owners under policy | Enforces approved versions; cannot invent or silently tune them |
| Executive, regulated, employment, credit, pricing, safety, or other consequential decision | Authorized human or governed decision system | Presents evidence and records disposition; never decides unsupported action |

### Explicit non-goals

This agent does not:

- define metrics unilaterally or bypass the semantic layer with free-form production SQL;
- treat missing, stale, partial, or failed data as a business value;
- claim causality from correlation, ranking, anomaly scores, or model prose;
- replace bounded analyst investigations or experimentation;
- update warehouse or source rows, change prices, contact customers, block accounts, adjust staffing, or spend money;
- remember personal preferences outside governed ownership and routing directories;
- use notification delivery, acknowledgement, or task creation as proof of business outcome;
- optimize an alert threshold directly from observed outcomes without owner review and a versioned release.

## Primary workflows

1. **Watch evaluation:** load an active watch version, obtain a semantic snapshot, apply rights, query the metric, and attach event-time, freshness, quality, and lineage evidence.
2. **Signal detection:** run a pinned deterministic detector, persistence and recovery rules, materiality constraints, and deduplication.
3. **Triage:** correlate related alerts and known events; optionally run approved read-only drill-downs; produce a cited evidence packet with uncertainty and no causal leap.
4. **Decision routing:** evaluate a versioned decision table, assign an accountable owner, request any required approval, and create a notification or work item with an idempotency key.
5. **Follow-through:** track acknowledgement, escalation timers, operator disposition, delegated action, and independent verification.
6. **Learning:** record realized outcomes as provenance-bearing episodes; use them for offline evaluation and owner-reviewed calibration, never automatic authority expansion.

## Reference architecture

~~~mermaid
flowchart LR
    T[Schedule, event, or manual trigger] --> C[Deterministic watch controller]
    C --> W[(Watch and case store)]
    C --> SG[Semantic-layer gateway]
    SG --> DQ[Freshness, quality, and lineage gate]
    DQ --> O[(Immutable observations)]
    O --> DR[Versioned detector registry]
    DR --> AR[Correlation and alert rules]
    AR --> MW[Bounded model triage worker]
    MW --> AR
    AR --> PE[Policy and approval engine]
    PE --> EX[Notification and work-item executors]
    EX --> EL[(Effect ledger and receipts)]
    EL --> RC[Reconciler]
    RC --> C
    C --> OV[Outcome verifier]
    OV --> EP[(Verified outcome episodes)]
    C --> OT[Traces, logs, metrics, and audit]
~~~

The controller owns workflow truth. The semantic layer owns metric meaning. Source systems own business facts. Notification and ticket systems own their remote objects. The effect ledger connects these systems without pretending they share a transaction.

### Component responsibilities

| Component | Must do | Must not do |
|---|---|---|
| Watch registry | Version owner-approved query, detector, schedule, rights, routes, and timers | Store an unreviewed model-generated metric |
| Semantic gateway | Resolve metric and dimensions under a pinned semantic snapshot | Generate arbitrary joins from a prompt |
| Quality gate | Distinguish accepted, stale, partial, invalid, and no-data observations | Convert data failure into zero or suppress it silently |
| Detector registry | Reproduce scores from versioned inputs and configuration | Choose a business action |
| Triage worker | Use an allowlisted evidence plan, cite facts, mark hypotheses | Fire alerts, grant approvals, or claim causality |
| Policy engine | Evaluate identity, tenant, severity, impact, data class, route, and approval | Delegate policy interpretation to the model |
| Effect executor | Deliver a typed, authorized effect and return a receipt | Retry an ambiguous request blindly |
| Outcome verifier | Compare the declared goal with authoritative evidence | Accept the executor's self-report as proof |

## Architecture variants

| Variant | Fit | Recommended shape | Reject when |
|---|---|---|---|
| Native BI alert plus runbook | Few fixed watches and no durable follow-through requirement | Existing BI alert, owner, and ticket integration | Cross-tool acknowledgement, revisions, outcome evidence, or strict reconciliation is required |
| Modular monolith | Default through reliable v1 | One deployable controller, workers, relational store, semantic API, queue, and external adapters | Independent scaling or blast-radius isolation is proven necessary |
| Durable workflow controller | Long waits, approvals, retries, and reconciliation span failures | Durable state machine with deterministic activities | Platform cannot expose effect identity or safe replay boundaries |
| Streaming evaluation | Low-latency event-time watches with late data | Stream processor for aggregation plus the same case/effect controller | Business action cannot tolerate provisional or revised results |
| Tenant/cell partitioned | High scale or regulated isolation | Tenant-scoped queues, stores, credentials, and regional cells | Operational capacity cannot support cell ownership and recovery |

Do not start with microservices, multiple agents, or a feature store. Start with a modular monolith and one relational source of workflow truth. Split only after measured contention, isolation, or release ownership justifies it.

## Runtime and authority summary

~~~mermaid
sequenceDiagram
    participant C as Controller
    participant S as Semantic layer
    participant Q as Quality gate
    participant D as Detector
    participant M as Triage model
    participant H as Owner or operator
    participant E as Effect executor
    C->>S: Query metric under watch and semantic versions
    S-->>C: Observation plus provenance
    C->>Q: Validate freshness, completeness, and assertions
    Q-->>C: Accepted, quarantined, or indeterminate
    C->>D: Evaluate accepted observation
    D-->>C: Reproducible signal evidence
    opt Allowed triage
        C->>M: Bounded evidence and drill-down catalog
        M-->>C: Cited hypotheses and route proposal
    end
    C->>H: Evidence packet and scoped decision request
    H-->>C: Approved disposition bound to case version
    C->>E: Authorized effect with stable operation key
    E-->>C: Receipt or unknown outcome
    C->>C: Reconcile and verify realized outcome
~~~

The model receives no ambient effect credential. High-impact paths stop at an evidence packet and decision request. Low-risk notification or ticket creation may be pre-authorized, but the resulting business action remains outside this agent.

## Non-negotiable invariants

1. Every evaluation pins `watch_version`, `semantic_snapshot`, `detector_version`, event-time interval, query digest, tenant, and rights decision.
2. Freshness and quality gates run before business detection. A data-health alert is a separate watch class.
3. Observation, evidence, decision, effect, and telemetry are separate records.
4. A model statement is a hypothesis until supported by cited retrievable evidence.
5. Every remote effect has a stable semantic operation key and a durable intent record before execution.
6. `UNKNOWN_OUTCOME` triggers reconciliation, never an immediate retry.
7. An acknowledgement means “an accountable subject has accepted triage ownership,” not “fixed.”
8. Threshold, routing, and action changes are reviewed versioned releases.
9. Tenant and data rights are structural filters applied before query and retrieval.
10. The agent cannot mutate source data or expand its own permissions.
11. Revisions and backfills produce new observations and explicit supersession links; history is not rewritten.
12. Success requires a verified realized outcome or an explicit terminal disposition such as false positive, no action, expired, or indeterminate.

## Reader paths

| Need | Start here |
|---|---|
| Decide whether to build and establish accountability | [01 — Mission, boundary, and architecture](01-mission-boundary-and-architecture.md) |
| Define a safe watch and data gate | [02 — Watch, semantic, freshness, and quality contracts](02-watch-semantic-freshness-and-quality-contracts.md) |
| Choose detectors and alert-to-decision behavior | [03 — Detection, triage, routing, and decision workflows](03-detection-triage-routing-and-decision-workflows.md) |
| Implement state, memory, context, and compaction | [04 — State, events, effects, memory, and context](04-state-events-effects-memory-and-context.md) |
| Integrate systems and enforce privacy/security | [05 — Tools, integrations, security, and governance](05-tools-integrations-security-and-governance.md) |
| Survive retries, ambiguity, revisions, and partial failure | [06 — Reliability, idempotency, and reconciliation](06-reliability-idempotency-and-reconciliation.md) |
| Prove behavior and observe production | [07 — Evaluation, observability, and failure injection](07-evaluation-observability-and-failure-injection.md) |
| Deploy, scale, operate, and upgrade | [08 — Deployment, scale, incidents, cost, and evolution](08-deployment-scale-incidents-cost-and-evolution.md) |
| Deliver from zero to production | [09 — Zero-to-production roadmap](09-zero-to-production-roadmap.md) |
| Qualify semantic, warehouse, BI, quality, event, delivery, and workflow adapters | [10 — Adapter qualification and integration playbooks](10-adapter-qualification-and-integration-playbooks.md) |

## Stage roadmap

| Stage | Capability | Authority ceiling | Exit gate |
|---|---|---|---|
| 0 — Deterministic baseline | Governed metric, query, data gate, fixed alert, owner, and runbook | No model; humans operate | Historical replay proves the watch and runbook are useful |
| 1 — First bounded loop | One read-only triage path and one pre-approved notification route | Draft and notify only | Evidence is faithful; no quality-gate bypass or unsupported cause claim |
| 2 — MVP | Multiple versioned watches, correlation, approvals, case state, and tickets | Low-risk communication effects only | Offline and shadow evaluation meet per-watch safety and load gates |
| 3 — Reliable v1 | Durable waits, idempotent effects, reconciliation, revisions, acknowledgement, outcomes | Same authority; stronger guarantees | Crash, duplicate, timeout, backfill, and stale-approval tests pass |
| 4 — Production | Identity, tenancy, SLOs, on-call, DR, canaries, privacy, and audit | Explicit policy matrix | Security, operational, and business-owner release review passes |
| 5 — Scale | Partitioned queues/cells, fairness, backpressure, degradation, regional recovery | No authority expansion | Peak-load and cell/tenant failure tests preserve safety and priority |
| 6 — Continuous evolution | Failure mining, drift monitoring, shadow upgrades, reviewed calibration | No automatic authority expansion | Every change is measured, reversible, and owner approved |

Full input, output, state, approval, failure, evaluation, and exit contracts are in the [roadmap](09-zero-to-production-roadmap.md).

## Stop conditions

Stop the current evaluation or case and surface a typed reason when:

- the metric, watch, owner, decision table, or semantic version is missing, expired, or incompatible;
- the freshness deadline, quality assertion, lineage expectation, or rights check fails;
- the query result is partial, revised beyond the allowed window, or cannot distinguish no-data from zero;
- a detector version or baseline cannot be reproduced;
- the case crosses a tenant, geography, purpose, or sensitivity boundary;
- the requested drill-down is outside approved dimensions or minimum cohort policy;
- evidence conflicts materially or a causal conclusion would be required;
- an approval is stale relative to the case, threshold, route, target, or action;
- an effect outcome is unknown, a duplicate cannot be reconciled, or cancellation races an effect;
- alert volume exceeds the watch, owner, tenant, or channel budget;
- the outcome cannot be verified before the configured terminal deadline.

## Top production risks

| Risk | Early signal | Required control |
|---|---|---|
| Semantic drift masquerades as business change | Break aligns with metric/model deployment | Pin semantic snapshots; shadow and compare before promotion |
| Stale or partial data produces false urgency | Freshness or coverage deviates from contract | Gate before detection; emit a separate data-health case |
| Alert fatigue | Low acknowledgement, repeated suppression, owner overload | Materiality, persistence, grouping, budgets, and quarterly pruning |
| Baseline contamination | Model learns an incident, promotion, backfill, or regime change | Exclusion windows, versioned baselines, owner-approved re-baselining |
| Multiple-testing explosion | False positives rise with slice count | Limit approved slices; control family-wise/FDR risk; evaluate event-level load |
| Automation bias | Operators accept model prose without evidence | Cited facts, explicit uncertainty, challengeable approvals, no model authority |
| Duplicate or lost work item | Timeout occurs around remote creation | Intent-first effect ledger, idempotency key, lookup reconciliation |
| Privacy leakage in drill-down | Small cohort or sensitive dimension appears in prompt/notification | Rights before retrieval, aggregation floors, redaction, destination policy |
| “Acknowledged” mistaken for “resolved” | Cases close after a click | Separate acknowledgement, disposition, action, and verified outcome states |
| Outcome feedback changes rules silently | Threshold behavior drifts without a release | Offline calibration proposal and owner-reviewed version promotion |

## Definition of done

The blueprint is implemented for a workload only when:

- each watch has a named metric owner, accountable operator, semantic contract, decision table, approved detector, data-quality gate, route, timers, and retirement condition;
- deterministic replay can reproduce observations and detector decisions from pinned versions;
- the runtime preserves authoritative state and effect identity across crash, retry, cancellation, backfill, and duplicate delivery;
- the model can access only allowed aggregated evidence and cannot write source data or approve action;
- cross-tenant, prompt-injection, stale-approval, unknown-effect, and notification-overload tests pass;
- business owners accept measured false-alert load, missed-event risk, acknowledgement SLO, and escalation policy;
- on-call staff can diagnose, pause, reconcile, re-drive, and roll back a release from documented runbooks;
- outcome verification distinguishes delivered notification, accepted work, executed action, and realized business result;
- behavior-bundle manifests pin metric/semantic inputs, detector/baseline, model/prompt/context, adapter dossiers, policy/decision tables, tools, runtime, capacity profile, and eval corpus;
- a simpler native alerting solution has been reconsidered and rejected for documented reasons.

## Canonical contracts reused

| Concern | Canonical guide | Workload-specific extension here |
|---|---|---|
| Commands, domain events, effects, delivery, telemetry | [Agent state and event contracts](../../runtime/agent-state-and-event-contracts.md) | Watch, observation, alert-case, acknowledgement, and outcome event types |
| Durable replay and external effects | [Durable execution](../../runtime/durable-execution.md) | Evaluation revisions, long acknowledgement waits, and remote work-item reconciliation |
| Execution authority | [Execution boundaries](../../runtime/execution-boundaries.md) | Read-only semantic access and source-mutation prohibition |
| Typed adapters | [Tool contracts](../../tools/tool-contracts.md) | Semantic query, quality, route, ticket, acknowledgement, and outcome contracts |
| Evidence and provenance | [Tool results, artifacts, and provenance](../../tools/tool-results-artifacts-and-provenance.md) | Semantic snapshots, detector evidence, and decision packets |
| Context assembly | [Context engineering](../../context-memory/context-engineering.md) | Per-case evidence compiler and drill-down budget |
| Memory | [Memory architecture](../../context-memory/memory-architecture.md) | Verified outcome episodes; personal and conversational memory disabled |
| Compaction | [Compaction and continuity](../../context-memory/compaction-and-continuity.md) | Case checkpoints with unresolved decisions and effect frontier |
| Threats and untrusted data | [Agent threat model](../../security/agent-threat-model.md) and [prompt injection](../../security/prompt-injection-and-untrusted-data.md) | Metric labels, ticket content, dashboard text, and model-generated explanations treated as data |
| Side-effect reliability | [Idempotency and side effects](../../reliability/idempotency-and-side-effects.md) | Alert, notification, ticket, escalation, and acknowledgement operation keys |
| Evaluation | [Evaluation-driven development](../../evaluation/evaluation-driven-development.md) | Event-level detection, alert load, decision latency, and outcome effectiveness |
| Observability | [Observability and tracing](../../evaluation/observability-and-tracing.md) | Watch-to-outcome correlation while minimizing sensitive values |
| Queues and scale | [Queues, scheduling, and backpressure](../../operations/queues-scheduling-and-backpressure.md) | Watch criticality, tenant fairness, owner/channel budgets, and freshness deadlines |
| Deployment and incidents | [Deployment, release, and incident response](../../operations/deployment-release-and-incident-response.md) | Shadow semantic/detector releases, alert freezes, and effect reconciliation |
| Planning | [Planning and replanning](../../orchestration/planning-and-replanning.md) | Deterministic evaluation graph plus bounded semantic triage planning |

## Research basis and limitations

The design synthesizes semantic-layer and data-contract mechanics, provenance and data-quality specifications, statistical process monitoring, alert-management practice, decision-model standards, privacy and AI-governance guidance, BI-platform alerting behavior, and agent runtime contracts. The [evidence packet](../../research/packets/business-intelligence-monitoring-agent-blueprint.md) records the claims, disagreements, benchmark limitations, licensing, jurisdictional caveats, and refresh triggers.

No public benchmark establishes end-to-end business value for this exact workload. Statistical detection results do not transfer automatically across metrics, aggregation levels, organizations, or intervention costs. Product limits and legal obligations change. Production promotion therefore depends on local historical replay, shadow operation, owner review, and jurisdiction-specific counsel—not on this blueprint alone.
