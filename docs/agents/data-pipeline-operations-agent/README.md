# Data Pipeline Operations Agent

**Status:** production blueprint
**Research date:** 2026-08-31
**Scope:** governed operation of batch, streaming, change-data-capture, and data-product delivery pipelines
**Evidence packet:** [research packet](../../research/packets/data-pipeline-operations-agent-blueprint.md)

## Executive summary

A Data Pipeline Operations Agent is a safety controller around existing data infrastructure. It diagnoses pipeline failures, builds replay and backfill plans, evaluates contracts and lineage, proposes or executes narrowly authorized recovery actions, and proves the delivered data outcome.

It is not a new scheduler, stream processor, SQL engine, catalog, or universal autonomous operator. The model may explain uncertain evidence and propose a plan. Deterministic policy decides whether an action is permitted. Typed adapters perform the action with scoped credentials. Independent checks verify the resulting data product.

> **Core principle:** never equate a green task, acknowledged API call, or successful model turn with correct data. A run is complete only when its intended effect is reconciled and its output contract, quality gates, and delivery SLO are verified.

## Category boundary

This guide uses **DataOps** as shorthand for pipeline and data-product operations, not general database administration or analytics.

| Capability | Owning agent | Data Pipeline Operations Agent relationship |
|---|---|---|
| DAGs, ingestion, transformations, CDC, checkpoints, backfills, lineage, quarantine, delivery recovery | Data Pipeline Operations Agent | Owns |
| Engine configuration, locks, indexes, transaction tuning, schema DDL, database migrations, backups, failover | Database Operations Agent | Observes and routes; never silently assumes database authority |
| Bounded questions, statistical analysis, analytic code, charts, and narrative findings | Analytics Agent | Supplies governed data products; does not own the analysis |
| Persistent metric monitoring, business alerts, subscriptions, and decision follow-through | BI Agent | Publishes trusted inputs and receives outcome signals |
| Source application correctness and event emission semantics | Application/service owner | Verifies contract and coordinates remediation |
| Policy exceptions, legal holds, privacy decisions, and destructive publication | Human data steward or incident commander | Requires explicit approval |

Pipeline health can be monitored continuously because it is an operational SLO. Business metric interpretation and ongoing decision state remain BI concerns.

## What the agent is for

Use it for:

- diagnosing a failed or stale batch, stream, CDC connector, or data product;
- assessing the blast radius of an upstream or schema change;
- creating a bounded, version-pinned backfill or replay plan;
- reconciling source frontiers, orchestrator runs, sink commits, and downstream delivery;
- enforcing data contracts, schema compatibility, lineage, quality gates, and quarantine policy;
- supervising low-risk retries or pause/resume actions under deterministic policy;
- producing incident evidence and safe recovery recommendations.

Do not use it as:

- a free-form SQL or shell agent with production credentials;
- a replacement for Airflow, Dagster, Prefect, Flink, Spark, Kafka, Debezium, dbt, or a catalog;
- a database migration agent;
- an analytics copilot or BI alerting system;
- a reason to promise exactly-once delivery without proving every boundary;
- a store for raw customer rows, secrets, arbitrary logs, or unreviewed “memory”;
- an autonomous privacy, retention, or deletion decision-maker.

## Non-negotiable invariants

1. **Data and metadata are untrusted input.** Table names, comments, catalog descriptions, row samples, logs, tickets, and tool results can contain prompt injection.
2. **The model has no ambient mutation authority.** Read and effect credentials are separate; effect credentials are just-in-time, narrow, and environment-bound.
3. **Every effect has a stable key.** The controller records intent before execution and reconciles ambiguous results before retrying.
4. **Policy is executable code.** Environment, tenant, data classification, blast radius, action type, and approval requirements are evaluated outside the model.
5. **Outcome verification is independent.** The executor cannot declare its own action correct.
6. **Versions are pinned.** A replay records pipeline definition, contract, schema, transformation, image, dependency, and adapter versions.
7. **Parallelism is bounded twice.** A global controller limit and a source/sink-specific budget both apply.
8. **Tenant and environment boundaries are structural.** They are fields in identities, state keys, queues, credentials, logs, and policy—not prompt instructions.
9. **Durable state is minimal and attributable.** Store decisions, hashes, frontiers, receipts, and evidence references; keep raw data in governed systems.
10. **Deletion is verified end to end.** A workflow must cover descendants, caches, quarantine, replicas, snapshots subject to policy, and audit evidence.
11. **Identity is more than a name.** Every plan binds canonical pipeline, source, dataset/product, schema, contract, transform, adapter, and effect identities to immutable versions or frontiers.

## Reference architecture

~~~mermaid
flowchart LR
    U[Operator or incident system] --> API[Identity-aware API]
    API --> C[Deterministic controller]
    C --> S[(Durable run and effect ledger)]
    C --> R[Read-only evidence adapters]
    R --> O[Orchestrators and compute]
    R --> M[Catalog, contracts, lineage]
    R --> Q[Quality and warehouse metadata]
    C --> G[Model gateway]
    G --> C
    C --> P[Policy and approval engine]
    P -->|authorized effect envelope| E[Isolated effect executor]
    E --> O
    E --> Q
    E --> X[Receipt store]
    X --> V[Independent verifier]
    M --> V
    Q --> V
    V --> C
    C --> N[Incident, audit, and SLO signals]
~~~

The controller is the system of record for the agent workflow. The orchestrator remains the system of record for task execution. The catalog remains authoritative for governed metadata. The warehouse or table format remains authoritative for committed output. Receipts connect these systems without pretending any one API spans an atomic transaction across them.

## Control loop

~~~mermaid
stateDiagram-v2
    [*] --> Intake
    Intake --> Evidence: identity and scope accepted
    Evidence --> Diagnose: evidence snapshot recorded
    Diagnose --> Plan: hypothesis has support
    Diagnose --> NeedsHuman: evidence conflicts or scope expands
    Plan --> Policy: typed action plan
    Policy --> NeedsApproval: policy requires approval
    Policy --> Execute: pre-authorized low-risk action
    NeedsApproval --> Execute: approval bound to plan hash
    Execute --> Ambiguous: timeout or lost acknowledgement
    Ambiguous --> Reconcile
    Reconcile --> Execute: no prior effect and retry permitted
    Reconcile --> Verify: effect already exists
    Execute --> Verify: durable receipt captured
    Verify --> Quarantine: output unsafe to publish
    Verify --> Complete: effect and outcome proven
    Verify --> Recover: bounded forward or rollback plan
    Recover --> Policy
    NeedsHuman --> [*]
    Quarantine --> [*]
    Complete --> [*]
~~~

The machine must reject an impossible transition. For example, a backfill cannot move from `PLAN` directly to `COMPLETE` because an orchestrator accepted it.

## Durable records

### Pipeline operation

~~~yaml
operation_id: op_01J...
tenant_id: retail-eu
environment: production
pipeline_ref: orders_hourly
requested_by:
  subject: oncall@example.com
  auth_context: incident-4821
objective:
  type: backfill
  interval: ["2026-08-28T00:00:00Z", "2026-08-28T06:00:00Z"]
versions:
  definition: git:4f6c...
  contract: odcs:orders-output:3.1.0
  transform: dbt-manifest:sha256:...
  runtime_image: sha256:...
frontiers:
  source: kafka:orders:partition-offset-map-hash
  target_before: iceberg:snapshot:88931
policy:
  decision_id: pd_...
  plan_hash: sha256:...
  approvals: []
effect_key: retail-eu/prod/orders_hourly/backfill/20260828T00-06/v4f6c
receipt_ref: null
verification_ref: null
state: PLANNED
~~~

### Effect receipt

A receipt is not merely an HTTP 200. It records enough to reconcile:

- effect key and request hash;
- adapter and API version;
- target tenant, environment, pipeline, and interval;
- remote run, job, transaction, snapshot, or deployment identifier;
- acknowledgement and terminal status timestamps;
- sink commit or table snapshot when available;
- checkpoint or source frontier before and after;
- error classification and retry disposition;
- immutable evidence references and content hashes.

### Independent verification

Verification should compare the intended outcome with authoritative systems:

- exact partition or key range delivered;
- input and output frontier movement;
- row/key counts and duplicate/delete handling;
- contract and schema checks;
- data-quality assertions and quarantine counts;
- table snapshot, warehouse job, stream checkpoint, or downstream acknowledgement;
- lineage completeness for the changed interval;
- freshness and downstream SLO recovery;
- absence of unauthorized cross-tenant or cross-environment effects.

## Autonomy ladder

| Stage | Agent authority | Required controls | Exit evidence |
|---|---|---|---|
| 0. Platform baseline | None | Contracts, lineage, deterministic runbooks, retention, test fixtures | Operators can recover without an LLM |
| 1. Read-only diagnosis | Read metadata and sanitized evidence | Typed read tools, provenance, injection defenses | Better diagnosis without unsafe suggestions |
| 2. Proposal MVP | Generate plans and patches only | Policy simulation, human review, plan hashes | Plans outperform baseline and are reviewable |
| 3. Supervised operations | Retry, pause, resume, or quarantine approved low-risk scopes | Durable state, effect keys, receipts, reconciliation, JIT credentials | No duplicate or untracked effects in fault tests |
| 4. Production readiness | Limited pre-authorized effects | HA, SLOs, DR, incident response, red-team gates | Meets safety and reliability release thresholds |
| 5. Scale and resilience | Bounded multi-pipeline concurrency | Tenant isolation, source budgets, admission control, cost caps | Load, degradation, and regional failure tests pass |
| 6. Continuous evolution | No automatic authority expansion | Offline evals, shadow runs, canary adapters, reviewed memory | Improvements are measured and reversible |

Start at Stage 0. A model cannot compensate for missing contracts, unverifiable sinks, or a scheduler that cannot identify historical work precisely.

## Guide map

| Guide | Question answered |
|---|---|
| [01 — Workload fit, boundaries, and autonomy](01-workload-fit-boundaries-and-autonomy.md) | What should this agent own, and how much may it do? |
| [02 — Reference architecture, runtime, and state](02-reference-architecture-runtime-and-state.md) | What components and durable records make the loop safe? |
| [03 — Batch, streaming, CDC, and adapters](03-batch-stream-cdc-and-tool-adapters.md) | How do semantics differ across runtimes and connectors? |
| [04 — Contracts, schema evolution, and data products](04-contracts-schema-evolution-and-data-product-delivery.md) | How are changes admitted and published safely? |
| [05 — Lineage, quality, quarantine, and governance](05-lineage-quality-quarantine-and-governance.md) | How is trust, blast radius, and privacy enforced? |
| [06 — Backfills, replay, idempotency, and reconciliation](06-backfills-replay-idempotency-and-reconciliation.md) | How are historical effects made bounded and provable? |
| [07 — Security, identity, tenancy, memory, and context](07-security-identity-tenancy-memory-and-context.md) | How is agent-specific risk contained? |
| [08 — Observability, SLOs, scale, deployment, and incidents](08-observability-slos-scaling-deployment-and-incidents.md) | What keeps it reliable in production? |
| [09 — Evaluation, failure injection, and delivery roadmap](09-evaluation-failure-injection-and-delivery-roadmap.md) | How is readiness measured from zero to production? |
| [10 — Worked production flows and operator runbooks](10-worked-production-flows-and-operator-runbooks.md) | How do the contracts and gates behave in realistic incidents? |

For implementation, read 01–02 first, then the relevant domain guides (03–06), apply security and operations controls (07–09), and use guide 10 as the integration exercise and exit-evidence checklist.

## First implementation slice

The smallest useful implementation is a read-only incident assistant for one batch pipeline family:

1. Normalize orchestrator run, contract, lineage, warehouse job, and quality evidence into typed records.
2. Record a signed evidence snapshot and its freshness.
3. Produce a diagnosis with cited facts, uncertainty, and a deterministic next-action plan.
4. Simulate policy and show which approval would be required.
5. Compare its answer with the existing runbook and real incident outcome.

Do not add mutation until ambiguous acknowledgements, duplicate requests, stale metadata, partial completion, and verifier failures are covered by tests.

## Decision checklist

Before authorizing any action, answer:

- Is the owning category and responsible human clear?
- Are tenant, environment, pipeline, interval, and versions explicit?
- Is the evidence current, attributable, and independently retrievable?
- Could any evidence contain executable or adversarial instructions?
- Is the action reversible or forward-recoverable?
- What is the effect key, and how will an ambiguous result be reconciled?
- What source, sink, compute, and concurrency budgets apply?
- What data contract and compatibility mode apply?
- What constitutes successful publication?
- What will be quarantined, and who can release it?
- Which downstream consumers are affected?
- What happens if the controller, worker, orchestrator, source, sink, or verifier fails?
- Is the expected data and compute cost within budget?
- Which evidence will prove success, rollback, or escalation?

## Selected primary sources

- [Airflow DAG runs, data intervals, catchup, and backfill](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/dag-run.html)
- [Apache Flink fault tolerance and exactly-once boundaries](https://nightlies.apache.org/flink/flink-docs-stable/docs/learn-flink/fault_tolerance/)
- [Debezium exactly-once delivery](https://debezium.io/documentation/reference/3.5/configuration/eos.html)
- [OpenLineage specification](https://github.com/OpenLineage/OpenLineage/blob/main/spec/OpenLineage.md)
- [Open Data Contract Standard](https://github.com/bitol-io/open-data-contract-standard/blob/main/docs/README.md)
- [Google SRE Workbook: Data Processing Pipelines](https://sre.google/workbook/data-processing/)
- [OWASP AI Agent Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/AI_Agent_Security_Cheat_Sheet.html)

The [evidence packet](../../research/packets/data-pipeline-operations-agent-blueprint.md) records the wider source set, claims, disagreements, and refresh triggers.
