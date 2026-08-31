# Mission, Boundary, and Reference Architecture

## The operational problem

Dashboards answer “what is visible now?” but rarely preserve the full chain from governed metric meaning to trustworthy observation, routed decision, accountable acknowledgement, external follow-through, and realized outcome. BI products can notify; incident systems can assign work; analytics teams can investigate. The missing capability is a durable control loop that keeps those responsibilities separate while preserving their evidence.

This agent owns that loop. It is not the metric authority, the data platform, the analyst, or the business decision-maker.

## When an agent is justified

Use a deterministic workflow without a model when the response is fully specified:

~~~mermaid
flowchart TD
    Q{Recurring governed watch?}
    Q -- No --> A[Bounded analytics investigation]
    Q -- Yes --> F{Fixed condition, route, and runbook sufficient?}
    F -- Yes --> N[Native BI or rule alert]
    F -- No --> V{Variable evidence triage but bounded actions?}
    V -- Yes --> B[BI monitoring agent]
    V -- No --> H[Human-led decision process]
~~~

The agent is justified only if variable evidence selection or explanation materially reduces time-to-understanding without weakening controls. Suitable examples include:

- correlate a metric break with approved deployment, campaign, calendar, data-health, and related-watch evidence;
- choose up to three drill-downs from the semantic contract's allowed dimensions;
- group a burst of related watch signals into one owner-facing case;
- draft a concise handoff that labels facts, hypotheses, missing evidence, and required decision;
- maintain a multi-day acknowledgement and outcome workflow across systems.

Unsuitable examples include choosing layoffs, prices, credit limits, medical actions, regulatory reports, or customer enforcement from a metric anomaly. An evidence packet may inform such a process; this agent must not decide it.

## Accountability model

| Role | Accountable for | Cannot delegate to the model |
|---|---|---|
| Metric owner | Business meaning, population, target, materiality, dimensions, semantic changes | Whether the metric means what the organization claims |
| Data product owner | Freshness, coverage, quality assertions, lineage, incident response | Whether data is fit for the watch |
| Watch owner | Schedule, detector, persistence, suppression, route, review date | Threshold and calibration approval |
| Accountable operator | Acknowledgement, disposition, action selection, follow-through | High-impact action judgment |
| Risk/privacy/security owner | Purpose, rights, destination, retention, geographic policy | Legal or policy interpretation |
| Platform owner | Runtime, state, effects, SLOs, release, incident recovery | System integrity |
| Model | Bounded evidence organization and explanation | Authority, facts, permissions, or final outcome |

An approval must bind to an immutable case version and exact proposed effect. A general “monitor this area” instruction is not reusable approval for future action.

## Reference architecture in depth

~~~mermaid
flowchart TB
    subgraph Control["Authoritative control plane"]
        WR[Watch registry]
        WC[Watch controller]
        CS[(Case and effect store)]
        PA[Policy and approval]
        RR[Reconciler]
    end
    subgraph Evidence["Evidence plane"]
        SL[Semantic API]
        DG[Data-health and lineage gate]
        OS[(Observation and artifact store)]
        DT[Detector runtime]
        CT[Context compiler]
        MT[Bounded triage model]
    end
    subgraph Delivery["Delivery and outcome plane"]
        NX[Notification adapter]
        TX[Work-item adapter]
        AX[Acknowledgement intake]
        OX[Outcome adapters]
    end
    WR --> WC
    WC --> SL
    SL --> DG
    DG --> OS
    OS --> DT
    DT --> WC
    WC --> CT
    CT --> MT
    MT --> WC
    WC --> PA
    PA --> CS
    CS --> NX
    CS --> TX
    NX --> RR
    TX --> RR
    AX --> WC
    OX --> WC
    RR --> WC
~~~

### Control-plane rules

- The watch registry stores immutable versions and one active pointer; promotion is transactional.
- The controller is the only component allowed to transition case state.
- Policy is executable, versioned code or a governed decision table. Model prose is input, never policy.
- The state store commits the effect intent before a worker attempts a remote write.
- Reconciliation reads the remote system using a stable key or returned identifier.
- Manual overrides create commands and audit events; operators do not edit rows by hand.

### Evidence-plane rules

- Semantic queries are templates or typed metric requests, not model-authored SQL.
- The data gate returns a typed verdict with evidence, not a boolean hidden in prompt text.
- Observation values and sensitive slices remain in governed stores; model context receives only the minimum permitted aggregates.
- Detectors run reproducibly from pinned code, configuration, baseline, and observation.
- The model sees a compiled case view and an allowlist of possible next reads.

### Delivery-plane rules

- Notification, ticket, and collaboration systems are effects, not workflow truth.
- An acknowledgement is accepted only from an authenticated authorized subject and is bound to the current case version.
- The agent may create or update a work item when policy permits; it cannot execute the work item's business action.
- Outcome adapters read authoritative operational and metric evidence. They do not infer success from a ticket closure alone.

## Data and control flow

| Step | Input | Deterministic output | Optional model output | Stop or escalation |
|---|---|---|---|---|
| Admit | Trigger and active watch version | Evaluation command and budget | None | Watch suspended, expired, or over capacity |
| Observe | Typed semantic request | Versioned observation or data-health verdict | None | Rights, freshness, quality, or lineage failure |
| Detect | Accepted observation and baseline | Signal score, rule results, and recovery state | None | Detector or baseline unreproducible |
| Correlate | Signals and open cases | Candidate grouping and dedup key | Related-evidence proposal | Ambiguous tenant/metric ownership |
| Triage | Approved evidence manifest | Case facts and missing evidence | Cited hypotheses and route proposal | Unsupported cause, unsafe slice, or budget exhausted |
| Decide | Case version and decision table | Required owner, approval, route, timers | Draft explanation | No authorized route or high-impact decision |
| Deliver | Authorized effect envelope | Intent and operation key | None | Stale approval or unknown prior effect |
| Follow | Receipt, acknowledgement, action, outcomes | State transitions and escalation | Summary draft | Deadline, conflict, or unverified outcome |

## Minimal practical deployment

The default implementation is:

- one service with controller, scheduler consumers, detector workers, triage worker, API, and reconciler modules;
- one relational database for watch, case, approval, timer, and effect-ledger state;
- an object store for immutable evidence artifacts;
- an existing queue for due evaluations and reconciliation;
- adapters to the organization's semantic layer, identity provider, data-quality/lineage service, notification channel, and ticket system;
- an OpenTelemetry-compatible export path and an audit sink.

Separate processes may isolate model execution and effect credentials, but they can remain one codebase. A durable workflow engine becomes useful when waits and recovery span many days and replay semantics are understood. See [custom loop versus frameworks](../../comparisons/custom-loop-vs-framework-vs-workflow-engine.md); a framework does not remove the need for an effect ledger or explicit state.

## Bounded planning and orchestration

The known topology is deterministic:

`load watch -> authorize -> query -> gate -> detect -> correlate -> triage if allowed -> policy -> effect -> reconcile -> follow -> verify`.

Only triage contains semantic uncertainty. Its planner may:

1. choose an allowed evidence query;
2. state why it discriminates between named hypotheses;
3. accept the typed result;
4. stop when the evidence budget or confidence rule is reached.

It may not:

- add a new metric or dimension;
- query row-level data;
- recursively invent tools;
- repeat a failed read without a retry classification;
- turn a hypothesis into a causal statement;
- choose an external business action.

Use a small stateful graph only if the triage branch is conditional. Do not use a multi-agent topology by default. Multiple model personas increase cost and make evidence lineage and retry ownership harder without changing the underlying authority.

## Deployment boundaries

| Boundary | Enforcement |
|---|---|
| Tenant | Tenant included in identity, state primary keys, queue partition, artifact namespace, credentials, and policy |
| Environment | Production/non-production separated in endpoints, credentials, routes, and effect keys |
| Geography | Regional processing and storage policy checked before evidence retrieval |
| Purpose | Watch contract declares purpose; query and destination must be compatible |
| Data class | Aggregation floor, redaction, retention, prompt eligibility, and channel matrix |
| Authority | Separate read, notify, work-item, and administrative capabilities |
| Time | Approval expiry, observation deadline, baseline validity, and maximum case lifetime |

## Design alternatives and rejection criteria

### Native BI alerts

Prefer native alerts when they provide sufficient metric pinning, recipient governance, persistence, and operational ownership. Product mechanics differ: some tie alerts to dashboard tiles or individual subscriptions, some evaluate on refresh or daily cadence, and newer event-rule products add stateful actions. None should be assumed to provide an authoritative cross-system effect ledger, case-version approvals, or outcome verification without testing. The [research packet](../../research/packets/business-intelligence-monitoring-agent-blueprint.md) compares representative products.

### Observability alert managers

Alert managers are strong at grouping, deduplication, routing, silencing, and inhibition. Reuse them when the business metric behaves like an operational signal. Add a decision-operations controller only for metric semantics, human accountability, case versions, follow-through, and outcomes. Do not rebuild proven notification fan-out.

### General analytics agent

An analytics agent should investigate a bounded question using governed semantics and reproducible artifacts. It should not carry thousands of persistent watch timers or escalation states. This agent can open an analytics request with:

- exact metric and semantic snapshot;
- observation and detector evidence;
- allowed scope and question;
- deadline and data rights;
- expected artifact and accountable recipient.

The returned analysis is new evidence, not an automatic decision.

## Build-versus-buy questions

- Does the existing BI product expose the semantic version, query, freshness, and delivery receipt needed for audit?
- Can it distinguish no-data, stale data, and zero?
- Can alert identity survive edits, backfills, retries, and provider outages?
- Are acknowledgement, disposition, action, and outcome distinct?
- Can a remote write be queried by an idempotency key after a timeout?
- Can rights be enforced before the evidence enters a model?
- Can thresholds, routes, and suppressions be reviewed as code or immutable versions?
- Can historical observations be replayed against a candidate detector without notifying users?
- Can owner and channel alert budgets be enforced?
- Can the platform be paused safely during a semantic or data incident?

If “no” answers are material, add only the missing control plane; do not replace the BI, ITSM, or alerting product wholesale.

## Architecture review checklist

- [ ] The category boundary and named human accountabilities are documented.
- [ ] A non-agent baseline remains operable.
- [ ] Semantic and source systems remain authoritative for their facts.
- [ ] The controller has one authoritative state-transition path.
- [ ] The model lacks ambient credentials and source-mutation tools.
- [ ] Effects are intent-first, typed, idempotent, and reconcilable.
- [ ] Outcome verification is independent of notification and work-item creation.
- [ ] Tenant, geography, purpose, data class, and environment are structural.
- [ ] A modular monolith was considered before distributed services.
- [ ] Every optional autonomous capability has a measurable exit gate and rollback.

## Related guides

- [Watch and semantic contracts](02-watch-semantic-freshness-and-quality-contracts.md)
- [Decision workflows](03-detection-triage-routing-and-decision-workflows.md)
- [State and context](04-state-events-effects-memory-and-context.md)
- [Security and integrations](05-tools-integrations-security-and-governance.md)
- [Roadmap](09-zero-to-production-roadmap.md)
