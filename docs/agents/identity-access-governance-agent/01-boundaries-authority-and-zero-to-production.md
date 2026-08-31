# Boundaries, Authority, and Zero-to-Production Stages

> **Purpose:** Decide whether an agent belongs, define accountable ownership, and progress through stages 0–6 using measurable gates.

## Stage 0 begins with a non-agent baseline

An identity-governance workload is not automatically agentic. Prefer ordinary software when structured events and explicit policy can decide the next step.

| Workload | Simplest suitable controller |
| --- | --- |
| Disable a known directory account after a signed termination event | Deterministic lifecycle workflow |
| Expire a time-bounded assignment at its recorded end time | Scheduler plus IAM/IGA API |
| Synchronize users and groups between compatible systems | Supported provisioning connector, often SCIM-profiled |
| Evaluate a fully specified incompatible-role matrix | Deterministic graph/policy query |
| Route a request from known attributes and thresholds | Rules engine or workflow table |
| Explain a six-hop effective-access path to a reviewer | Bounded model over a verified graph projection |
| Triage contradictory HR, sponsor, directory, and application records | Durable workflow plus bounded model analysis |
| Prepare a decision packet from heterogeneous evidence and exceptions | Workflow plus bounded model worker |

The model is justified only when it improves a measured semantic bottleneck—such as explaining complex inherited access, classifying evidence gaps, or preparing exception packets—without becoming the source of identity or policy.

## Operating contract

Define one workload before choosing technology:

```yaml
workload: mover_access_reconciliation
tenant: tenant-7
business_owner: identity_governance
control_owners:
  sod_policy: finance_controls
  privileged_access: security_operations
authoritative_sources:
  employment: hris-workforce
  directory_account: workforce-directory
  application_entitlement: erp-production
allowed_subject_types: [workforce_person]
terminal_outcomes:
  - reconciled_no_change
  - approved_change_verified
  - denied
  - exception_assigned
prohibited:
  - infer_subject_from_name_or_chat
  - model_decides_policy
  - self_approval
  - privileged_grant
  - claim_compliance
```

Every deployment must name the identity authority, resource owner, control owner, approver, IAM platform owner, connector owner, incident commander, privacy owner, and service owner. “IAM team” is not precise enough for a production responsibility matrix.

## Stages 0–6 at a glance

| Stage | Authority ceiling | Principal deliverable | Expansion rule |
| --- | --- | --- | --- |
| 0 — qualify | No production read beyond approved exports; no effect | Deterministic baseline and non-goals | Continue only if semantic ambiguity is measured and material |
| 1 — bounded loop | I0, one tenant and read-only sources | Source-backed discrepancy/review explanation | No durable waits or mutation |
| 2 — useful MVP | I1; optional I2 draft ticket | Entitlement graph, review packet, JML exception queue | Real connectors, still no access-changing write |
| 3 — reliable v1 | One I3 low/non-privileged effect class | Durable approvals, idempotent dispatch, target verification | One connector and one policy cell at a time |
| 4 — production readiness | Risk-approved I0–I3 cells | Tenant isolation, SLOs, releases, incidents, privacy | Canary authority separately from model quality |
| 5 — scale and resilience | I4 only for proven narrow repairs; I5 remains proposal-only | Incremental graph, cells, backpressure, DR | No global batch authority |
| 6 — continuous evolution | No automatic authority expansion | Failure-mined evals and versioned upgrade program | Upgrades shadow first; policy owners approve semantics |

## Stage 0 — qualify the problem

| Concern | Required design |
| --- | --- |
| Authority | Offline analysis of approved exports; no connector credential in the reasoning process and no production effect |
| Architecture | SQL/graph query or existing IGA report plus human workflow; optionally a model in an isolated evaluation harness |
| Inputs and outputs | Versioned sample snapshots in; discrepancy list, deterministic baseline, labeled decisions, and cost/time measures out |
| State, events, effects | `snapshot_id`, source cutoff, policy version, and evaluation case; effects are prohibited |
| Approvals | Data owner approves evaluation use; IAM/control owners approve workload boundaries |
| Recovery | Re-run from immutable snapshot; detect non-reproducible queries and missing pages |
| Evaluation | Compare rules-only, existing product, and model-assisted approaches on the same labeled cases |
| Exit gates | 100% of sources, owners, subject types, effect classes, and non-goals named; deterministic baseline measured; no identity inferred from free text; a documented semantic gap remains; privacy/security review allows Stage 1 |

Stage 0 may end with “do not build an agent.” That is a successful result.

## Stage 1 — first bounded loop

Build one loop: compile a read-only evidence packet, let the model choose from an allowlisted analysis action set, validate the result, and stop.

```text
inspect_case -> request_allowed_evidence? -> analyze -> validate -> complete | abstain
```

| Concern | Required design |
| --- | --- |
| Authority | I0; one tenant, one workflow, one or two read-only connectors; hard call, token, row, and wall-clock budgets |
| Architecture | Stateless request handler, deterministic context compiler, typed read tools, bounded model, schema/evidence validator |
| Inputs and outputs | Canonical case ID and source references in; `finding`, `evidence_refs`, `missing_evidence`, `abstain_reason` out |
| State, events, effects | Ephemeral run state only; events include `run_started`, `evidence_read`, `analysis_proposed`, `run_completed`; no effects |
| Approvals | Analyst launches each run and owns disposition; tool admission is pre-approved, not chosen from discovery at runtime |
| Recovery | Failed run restarts from the same frozen source cut; no resume claim and no hidden conversational continuity |
| Evaluation | Schema validity, citation precision, unsupported-claim rate, ambiguity detection, forbidden-tool attempts, resource budgets |
| Exit gates | 100% schema-valid outputs; zero cross-tenant reads and zero effect attempts; every substantive claim maps to an evidence reference; all ambiguous identity matches abstain; model beats or complements baseline on a ratified task metric |

Do not add “memory” to improve a weak Stage 1 result. First fix source selection, graph semantics, tool contracts, or task definition.

## Stage 2 — useful MVP

| Concern | Required design |
| --- | --- |
| Authority | I1; I2 only for a non-authoritative draft ticket/request after exact-target validation |
| Architecture | Real read connectors, normalized evidence store, versioned entitlement graph, durable case state, policy analyzer, human queue, model analyst |
| Inputs and outputs | HR/partner events, accounts, groups, roles, permissions, resource ownership, usage signals, and policy versions in; review packet, JML discrepancy, SoD finding, orphan hypothesis, and draft proposal out |
| State, events, effects | Durable case/event records; evidence, observation, decision, and effect types separated; the only optional effect is `create_draft_case` |
| Approvals | Correlation ambiguities and all access dispositions go to named people; reviewer sees source freshness and access path, not only a recommendation |
| Recovery | Cursor checkpoints, snapshot manifests, replay-safe case creation, dead-letter queue, periodic full reconciliation |
| Evaluation | Connector contract tests; graph-path oracle; stale/missing/conflicting-source cases; reviewer time, correction rate, and evidence sufficiency |
| Exit gates | Full snapshot and incremental sync converge; zero silent pagination loss in fault suite; graph oracle matches target samples; 100% review items show direct/inherited path and cutoff; unresolved correlation can never reach an effect; operators clear simulated backlog within the recovery objective |

MVP usefulness is measured by safer/faster decisions and earlier lifecycle discrepancies, not by message quality.

## Stage 3 — reliable v1

Enable one bounded I3 effect, such as submitting an approved non-privileged group removal through the existing IGA system.

| Concern | Required design |
| --- | --- |
| Authority | One exact effect cell: connector × tenant × subject type × resource class × operation; privileged and policy-changing operations remain I5 proposal-only |
| Architecture | Durable workflow, approval service, commit-time policy check, credential broker, effect ledger, target verifier, reconciler |
| Inputs and outputs | Canonical proposal and approval digest in; provider receipt, target-state evidence, reconciliation result, and exception assignment out |
| State, events, effects | `approval_granted`, `approval_denied`, `approval_expired`, `effect_reserved`, `effect_dispatched`, `effect_unknown`, `effect_observed`, `postcondition_verified`; immutable operation ID |
| Approvals | Independent approver with current authority; no proposer/self approval; high-risk or SoD-sensitive cells require the configured dual-control route |
| Recovery | Crash-before/after-dispatch tests, idempotency reuse, status/read-after-write reconciliation, correction rather than history deletion |
| Evaluation | Stale approval, changed manager/owner, termination race, duplicate delivery, rate limit, timeout, partial group propagation, rollback/forward-recovery cases |
| Exit gates | Zero unauthorized or duplicate commits in the release suite; every timeout resolves to observed/no-effect/assigned exception; target verification covers every committed test; approval digest changes on every material field; manual fallback and pause switch drilled |

“Zero unauthorized effects” is a hard invariant. Accuracy or speed cannot average it away.

## Stage 4 — production readiness

| Concern | Required design |
| --- | --- |
| Authority | Explicit matrix of I0–I3 cells; default deny; independent I5 administration; emergency disable outside the model |
| Architecture | Separate control/effect/evidence/telemetry planes; tenant cells; workload identity; secrets broker; immutable release manifest; canary routing |
| Inputs and outputs | Production data under purpose/field projections; redacted operational signals; durable audit evidence and incident-impact queries |
| State, events, effects | Version-pinned runs, migration/quarantine rules, deletion/retention state, incident and release correlation |
| Approvals | Approval-service SLO, fallback/reassignment rules, anti-fatigue sampling, protected-access review by PAM/control owner |
| Recovery | Rollback model/prompt/policy/connector independently; revoke credentials; quarantine source; drain/pause effects; restore state and reconcile targets |
| Evaluation | Representative, adversarial, privacy, tenancy, load, repeated-reliability, human-factors, disaster, and incident exercises |
| Exit gates | Threat model signed; zero tenant escapes and secret exposures in tests; approved SLOs and dashboards live; canary/rollback and credential-revocation drills pass; evidence retention/deletion verified; on-call and accountable business owner accept launch |

## Stage 5 — scale and resilience

| Concern | Required design |
| --- | --- |
| Authority | No breadth increase from batching; selectors are canonicalized to a bounded manifest; I4 only for a proven reversible drift class |
| Architecture | Per-tenant/cell partitioning, incremental graph updates, sharded reconciliation, workload queues, priority lanes, admission control, regional strategy |
| Inputs and outputs | Change feeds plus scheduled full snapshots; chunked review campaigns and manifests; tenant-isolated artifacts |
| State, events, effects | Partition key, fairness budget, graph epoch, watermark, batch manifest, per-item effect state; never one opaque batch success flag |
| Approvals | Approval capacity is modeled; campaigns split to remain reviewable; batch approval binds the enumerated set or stable predicate plus cutoff |
| Recovery | Cell-level failover, replay from persisted cursors/events, connector quarantine, cold full rebuild, backlog shedding and rehydration |
| Evaluation | Hot-tenant, million-edge graph, slow reviewer, webhook storm, source outage, rate-limit collapse, regional loss, and noisy-neighbor tests |
| Exit gates | Capacity test sustains forecast plus approved headroom; tenant fairness and blast-radius limits hold; high-risk leaver/revocation lane meets its SLO under overload; full graph rebuild and regional recovery meet RTO/RPO; cost-per-governed-edge/case remains inside budget |

Do not copy the scale claims of Zanzibar or a vendor. Benchmark the actual topology, graph shape, connector limits, and consistency needs.

## Stage 6 — continuous evolution

| Concern | Required design |
| --- | --- |
| Authority | Upgrade cannot add tools, fields, tenants, resource scopes, or effect classes implicitly; authority changes use the I5 path |
| Architecture | Version registry, shadow evaluation, replay corpus, policy tests, schema compatibility checks, failure-mining pipeline, deprecation workflow |
| Inputs and outputs | Incidents, reviewer corrections, appeals, reconciliation misses, drift, cost/latency, connector releases, policy changes in; new regression cases and release decisions out |
| State, events, effects | Immutable behavior manifest links model, prompt, compiler, tools, connector profile, graph schema, policy, eval set, and runtime versions |
| Approvals | Model/tool changes owned by service owner; policy semantics by control owner; connector authority by IAM/security owner; privacy changes by privacy/legal owners |
| Recovery | Roll back or quarantine the changed component; active cases pin or explicitly migrate; impact query locates affected cases/effects |
| Evaluation | Offline comparison, shadow, canary, repeated reliability, slice regression, policy invariants, reviewer blind test, online monitoring |
| Exit gates | No hard-gate regression; no unexplained slice degradation; shadow/canary stop criteria remain clear; migration and rollback tested; new production failure has a reproducer, owner, and prevention/detection/recovery action |

## Stage 0–6 exercise and exit-evidence portfolio

Promotion evidence is a reproducible bundle, not a slide or model demo. Each stage retains the prior stage's hard invariants.

| Stage | Required exercise | Exit evidence to retain |
| --- | --- | --- |
| 0 — qualify | Run the same frozen mover/review sample through the existing IGA/report, a deterministic query/rules baseline, and an isolated model-assisted analyst. Include cases that need no model and ambiguous cases that must abstain. | Dataset/source manifest, policy version, human labels/adjudication, quality/time/cost comparison, privacy approval, recorded `build workflow only`, `add bounded analyst`, or `do not build` decision |
| 1 — bounded loop | Inject duplicate names, stale sources, a six-hop path, prompt injection in a group description, tool-budget exhaustion, and a cross-tenant identifier. | Typed outputs and evidence-link score, every abstention, denied tool/read log, tenant/purpose enforcement result, resource-use distribution, release manifest |
| 2 — useful MVP | Break a snapshot page, expire a cursor, omit a deletion, change an entitlement definition, and run a realistic joiner/mover/leaver/review/orphan set through full rebuild and incrementals. | Connector qualification reports, snapshot manifests, graph/path oracle diff, reviewer study, ambiguity queue, recovery timing, declared coverage and unsupported-capability register |
| 3 — reliable v1 | For one non-privileged removal, kill the worker before reservation, after reservation, during dispatch, and after target commit; also inject stale approval, late cancellation, rate limiting and target propagation delay. | Approval/effect ledger, semantic operation IDs, provider artifacts, target postcondition evidence, zero duplicate/unauthorized effects report, `UNKNOWN` reconciliation outcomes, manual-fallback drill |
| 4 — production readiness | Red-team identity spoofing, confused deputy, injection, secret leakage, insider misuse and tenant escape; revoke connector credentials, quarantine a release, roll back each behavior component, and restore retained/deleted data. | Signed threat/privacy reviews, test traces with no secret/PII leakage, authorization/audit evidence, rollback and impact-query results, on-call runbook drill, ratified SLO/dashboard and launch acceptance |
| 5 — scale and resilience | Replay a termination surge during a large review campaign, hot tenant, provider `429` collapse, graph fan-out spike, regional loss, cold graph rebuild and reconciliation backlog recovery. | Capacity curves and saturation points, fairness/critical-lane SLO report, provider-quota measurements, RTO/RPO proof, reconciliation/backlog drain time, cost per governed item/effect, remaining headroom |
| 6 — continuous evolution | Upgrade model, context compiler, connector profile, graph schema, policy and workflow separately; shadow, canary, abort and restore an active case containing a pending approval and `UNKNOWN` effect. | Before/after slice and repeated-reliability report, compatibility/compaction receipts, canary stop evidence, migration/rollback result, affected-case query, failure-mined fixture and accountable release decision |

An evidence bundle must name tenant fixture or approved production slice, subject/resource classes, source cutoffs, graph/policy/workflow/tool/model versions, clocks, test runner/version, expected oracle, actual outcome, artifacts, reviewer/owner, exceptions, and retention location. A stage has not exited while a required exercise is waived without an accountable, expiring exception and compensating control.

## Accountable ownership matrix

| Decision | Accountable owner | Agent role |
| --- | --- | --- |
| Employment/contractor state | HR or partner system owner | Consume signed/versioned facts; flag contradictions |
| Identity proofing/recovery | IdP/help desk under approved process | No role beyond routing an out-of-scope case |
| Resource business need | Resource/business owner | Prepare evidence and proposal |
| SoD rule semantics | Control owner with legal/audit input as applicable | Execute versioned rule and explain paths |
| Access-review decision | Assigned reviewer/certifier | Prepare packet; never attest |
| Privileged access | PAM/security owner and configured approvers | Proposal-only |
| Provisioning authorization | IAM/IGA policy and authorized approver | Submit only a bound approved operation |
| Revocation verification | IAM service owner plus target owner for exceptions | Collect target evidence and reconcile |
| Compliance conclusion | Independent control/audit function | Export evidence; no compliance claim |
| Production launch and SLO | Service owner | Provide evaluation and operational evidence |

## Stage-gate checklist

- [ ] Authority ceiling and prohibited actions are machine-enforced.
- [ ] Identity and resource facts have named authoritative sources and freshness limits.
- [ ] Evidence, observations, decisions, and effects remain separate.
- [ ] The graph can explain every effective-access result by path and source.
- [ ] Human reviewers see conflicts, missing data, and recommendation provenance.
- [ ] Every enabled effect has idempotency, current authorization, postcondition, and recovery.
- [ ] The stage's hard invariants pass at 100%; empirical thresholds are ratified for this deployment.
- [ ] Failure drills demonstrate the claimed recovery, not merely document it.
- [ ] The next stage adds only the measured capability and authority cells it needs.

## Related guides

- [Blueprint overview](README.md)
- [Reference architecture, connectors, and entitlement graph](02-reference-architecture-connectors-and-entitlement-graph.md)
- [Evaluation, observability, and failure injection](07-evaluation-observability-and-failure-injection.md)
- [Custom loop vs framework vs workflow engine](../../comparisons/custom-loop-vs-framework-vs-workflow-engine.md)
- [Production agent control plane](../../architectures/production-agent-control-plane.md)
