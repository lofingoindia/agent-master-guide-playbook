# Reliability, Observability, Evaluation, and Failure Injection

> **Purpose:** Prove that the complete evidence-and-review workflow is trustworthy under realistic drift, ambiguity, concurrency, outages, and adversarial input—not merely that a model can answer example questions.

## Evaluate claims, controls, trajectories, and outcomes

The system makes different kinds of claims, each needing different evidence:

| Claim | Evaluation object | Example failure |
| --- | --- | --- |
| “This mapping is useful” | Versioned mapping proposal and reviewer label | False equality or wrong version/jurisdiction |
| “This evidence is the requested source record” | Connector/vault/provenance trajectory | Wrong account, truncated pages, overwritten version |
| “This sample is reproducible” | Plan/population/algorithm/seed/manifest | Post-freeze population drift or automatic replacement |
| “This workpaper is supported” | Facts, citations, contradictions, procedure steps, review | Citation exists but does not support claim |
| “The workflow is controlled” | State/event/auth/approval/effect history | Illegal transition or stale approval used |
| “The package is exact” | Manifest, dependencies, render, approval, delivery | Missing artifact or recipient mismatch |
| “The service is operable” | SLOs, queues, recovery, manual capacity, incidents | Reviewer backlog grows while API latency looks healthy |
| “The automation adds value” | End-to-end quality, time, cost, rework, exceptions | Model savings are smaller than review/correction burden |

Follow [evaluation-driven development](../../evaluation/evaluation-driven-development.md) and [trajectory and reliability evaluation](../../evaluation/trajectory-and-reliability-evaluation.md).

## Hard controls versus statistical quality

Do not average hard-control violations into an accuracy score.

### Hard failures

- cross-tenant or out-of-purpose evidence access;
- credential/secret exposure;
- unapproved profile, mapping, plan, source, field, or effect;
- model-authored compliance/effectiveness/attestation conclusion accepted as authoritative;
- proposer/preparer self-review where prohibited;
- untraceable artifact/workpaper/decision/package reference;
- silent truncation, missing lineage, sample mutation, or replacement;
- duplicate/ambiguous external effect without reconciliation;
- package freeze/delivery with stale approval or wrong destination;
- deletion under active hold or use of deleted/withdrawn evidence.

These require zero observed violations in release gates and immediate containment in production. “Zero observed” is not proof of impossibility; maintain layered prevention and detection.

### Statistical/task metrics

Set qualified thresholds by control family, evidence format, connector, language, tenant risk, and task:

- mapping retrieval recall, approved precision, relationship/direction/exclusion accuracy;
- field/temporal comparison precision and recall;
- citation correctness, completeness, and entailment;
- contradiction and missing-evidence recall;
- calibrated abstention/deferral and false-certainty rate;
- reviewer acceptance, correction, disagreement, rework, time, and fatigue;
- request relevance/duplication and custodian burden;
- population reconciliation, source freshness, package build and effect-reconciliation latency;
- cost per accepted evidence item, reviewed control, and frozen/delivered package.

Do not choose a universal threshold from this guide. Establish it from the baseline, consequence, reviewer capacity, and governing methodology in Stage 0.

## Evaluation corpus

Build representative, access-controlled datasets with:

- approved and rejected control mappings, including partial relationships, wrong versions, no relationship, and licensed-source absence;
- structured API records, CSV exports, PDFs, screenshots, tickets, narratives, OCR errors, multilingual material where in scope;
- complete, partial, truncated, stale, duplicated, late, reordered, conflicting, exempt, unknown, and tampered source scenarios;
- TOD walkthroughs with design gaps, bypass paths, dependency failures, and owner representations;
- TOE populations and samples with clean items, deviations, missing selected evidence, invalid population definitions, and mid-period control changes;
- adversarial evidence instructions, exfiltration URLs, secrets, archive bombs, active content, and retrieval poisoning;
- reviewer conflicts, stale decisions, changed inputs, reassignments, deadline pressure, and rubber-stamp patterns;
- effect timeouts before/after commit, duplicate messages, partial pages, webhook loss, rate limits, and source outage;
- package omissions, classification conflicts, late evidence, renderer changes, wrong recipients, and legal holds.

Use synthetic cases for destructive/security tests. Production-derived examples require purpose, minimization, deidentification where feasible, restricted access, retention, and explicit approval. Preserve a sequestered blind set.

## Ground truth and disagreement

Qualified reviewers label independently for high-judgment tasks. Store:

- label and allowed alternatives;
- evidence basis and exact source versions;
- methodology/profile version;
- reviewer identity/qualification and independence status;
- disagreement and adjudication record;
- uncertainty or `cannot_determine` rather than forced binary labels.

Reviewer disagreement may reveal ambiguous criteria or insufficient evidence, not model error. Report both raw agreement and adjudicated metrics; do not train away legitimate professional disagreement as noise.

## Evaluation ladder

| Layer | When | What must pass |
| --- | --- | --- |
| Schema/unit | Every change | ID/version/digest validation, transition rules, relationship types, canonicalization, sampler, manifest, claim vocabulary |
| Component | Every adapter/model/prompt/profile change | Connector pagination/schema/rate handling; extraction/mapping/citation/abstention slices; renderer compatibility |
| Workflow simulation | Every release candidate | Complete requests, collections, samples, reviews, exceptions, freeze/delivery trajectories with faults |
| Security/privacy | Every relevant change and periodic red team | Tenant/purpose isolation, injection, secrets, malicious files, provider route, retention/hold, package egress |
| Shadow/replay | Before authority increase and major upgrade | Historical/parallel proposals and would-effects against human baseline |
| Canary | Production promotion | Bounded tenants/profiles/effects, live SLOs, manual review, rollback ready |
| Continuous | Production | Drift, control violations, reconciliation, reviewer outcomes, queue/cost signals |
| Incident regression | After every material incident/near miss | Minimal reproducer plus adjacent/variant tests remains in governed suite |

## End-to-end trajectory assertions

An evaluator should be able to assert:

```yaml
trajectory_assertions:
  - before: evidence_request.dispatched
    requires: [authorization.allow, effect.reserved]
  - before: evidence_version.registered
    requires: [tenant_match, content_digest, provenance_manifest, retention_policy]
  - before: sample.selected
    requires: [plan.approved, population.frozen, seed_policy.valid]
  - before: workpaper.awaiting_review
    requires: [procedure_version, exact_input_manifest, citations_resolve]
  - before: review_decision.recorded
    requires: [human_authentication, independence.allow, input_not_stale]
  - before: package.frozen
    requires: [manifest_valid, required_reviews_complete, classification_valid]
  - before: package.delivered
    requires: [exact_freeze_approval, destination_allowlisted, commit_authorized]
  - always_forbidden:
      - model_output_changes_authoritative_conclusion
      - selected_item_silently_replaced
      - trace_used_as_evidence_source_of_truth
```

## Failure-injection matrix

Run controlled experiments in lower environments and bounded canaries. Preserve expected state, observed state, recovery time, operator actions, and any evidence/business correction.

| Injection | Expected system behavior | Evidence of recovery |
| --- | --- | --- |
| Worker dies after event commit | Replacement worker resumes next durable node; no duplicate state | Event version and replay trace |
| Worker dies after downstream commit before receipt | Effect becomes `unknown`; reconciliation finds existing commit | One downstream object and reconciled receipt |
| Duplicate create/reminder/delivery message | Same operation ID deduplicates or downstream idempotency returns original | Attempt ledger and one semantic outcome |
| Source returns partial page then 500 | Partial artifact not finalized as complete; cursor/backoff resumes | Page ledger, final counts, limitation if incomplete |
| Rate limit/throttle storm | Per-source circuit breaker/backoff; fair queues; deadlines visible | No retry storm; queue/SLO record |
| Webhook lost/expired | Periodic authoritative query finds state/event; webhook renewed | Reconciliation gap and correction event |
| Events arrive late/out of order | Overlap/watermark/dedupe; completeness delayed | Stable record count and watermark decision |
| Connector schema/enum changes | Adapter marks incompatible; raw bytes preserved; no false mapping | Quarantine alert and compatibility block |
| Adapter was never qualified for this tenant/configuration | Capability denied before collection; no downstream evidence use | Qualification decision and approved alternate/manual route |
| Source retention window missed | Explicit unrecoverable/alternate-source limitation | Human scope/procedure decision |
| Wrong account/project/org | Resource binding rejects before use; alert | Denied policy decision; no artifact registered |
| Cross-tenant ID/cached result | Access denied and security alert; no model exposure | Isolation audit record |
| Evidence contains instructions/exfiltration URL | Treated as content; output/effect validator blocks | Injection test result and no egress |
| Malicious archive/oversize file | Quarantine and bounded processing; workers stay healthy | Quarantine record and capacity metrics |
| Secret appears in evidence | Stop indexing/model path; restrict, notify, rotate, sanitize | Incident and derived redaction lineage |
| Population count mismatch | Population cannot freeze; reconciliation work item | Approved correction/limitation decision |
| Population changes after freeze | New population version; old sample remains pinned | Impact decision and no mutation |
| Missing selected evidence | Item remains selected and explicit; no auto-replacement | Missing-evidence/authorized replacement history |
| Model invents citation or conclusion | Schema/citation/claim validator rejects and records failure | Proposal rejection reason |
| Context compaction drops contradiction | Continuity validator fails; rebuild from durable inputs | Exact contradiction restored before use |
| Reviewer account is preparer alias | Real-person SoD denies decision | Independence check denial |
| Reviewer decision input changes | Decision becomes stale; affected work reopens | Impact event and new review version |
| Renderer crashes mid-build | Candidate not frozen; deterministic retry uses same manifest | Same output digest after recovery |
| Immutable store times out after freeze | `unknown`; reconcile object/version/digest before retry | One frozen version and receipt |
| WORM governance-bypass permission appears | Quarantine preservation route; security/records review mode and existing versions | Permission removed or explicitly accepted; version-scoped retention/hold reverified |
| E-signature webhook trims document/participant data | Treat notification as hint; fetch authoritative agreement/document version | Complete source receipt and exact signed-document digest or explicit limitation |
| Warehouse audit view is delayed or omits statement class | Hold freshness/completeness decision; reconcile complementary source | Qualified coverage statement, watermark, count and limitation receipt |
| MCP task ID is retrieved across authorization contexts | Deny, alert, quarantine server/tool release; no result registration | Tenant/auth binding test, task access review and credential rotation as needed |
| New evidence arrives after freeze | Package impact event; no in-place change | Reopen/supplement/no-impact human decision |
| Wrong package destination requested | Policy denies; no dispatch | Denial and alert |
| Legal hold races deletion | Hold wins; deletion pauses and reconciles | Artifact retained; decision history |
| Model/provider outage | Queue/manual fallback; durable deadlines and human actions continue | No state loss and measured backlog recovery |
| Telemetry backend outage | Execution continues with bounded local buffering; audit events unaffected | Backfill/alert without duplicate domain event |
| Regional restore misses committed ledger tail | Restored cell remains read/effect-disabled; compare signed watermarks and source/destination state | Complete state/effect reconciliation or declared RPO breach and business correction |
| Recovery backlog exceeds reserved drain capacity | Admission control sheds optional work; protect source windows, unknown effects, holds and review deadlines | Measured net drain rate clears backlog within approved recovery objective |

## Observability model

Use [observability and tracing](../../evaluation/observability-and-tracing.md), but separate three planes:

| Plane | Purpose | Examples | Retention/access |
| --- | --- | --- | --- |
| Domain/audit record | Reconstruct engagement and accountable actions | state events, evidence provenance, review decisions, effects, package manifests | Engagement policy/legal hold; restricted assurance access |
| Security audit | Detect/access-investigate use of sensitive systems | auth decisions, vault reads, grant issuance, key/admin action, export | Security policy; highly restricted |
| Diagnostic telemetry | Operate/debug service | spans, queue age, latency, errors, tokens, adapter status | Shorter/minimized; sampled where allowed; not evidence truth |

Trace IDs may link to domain IDs. A trace does not replace the domain record because it can be sampled, redacted, dropped, or deleted. Evidence content should not be placed in span attributes; use opaque IDs and reason codes.

## Required correlation dimensions

`tenant_id`, `engagement_id`, `run_id`, `task_id`, `aggregate_type/id/version`, `event_id`, `operation_id`, `attempt_id`, `artifact_id/version`, `plan/population/sample IDs`, `workpaper/decision/exception/package IDs`, `profile/release/connector/model/prompt/schema versions`, actor/workload identity, and source/destination class.

High-cardinality identifiers belong in traces/logs or exemplars, not unbounded metric labels.

## Service indicators and SLOs

Define user/assurance-facing indicators rather than only infrastructure uptime:

| Indicator | Definition | Error condition |
| --- | --- | --- |
| Request dispatch correctness | Authorized request effects reconciled to exact downstream request | duplicate, wrong target/payload, unresolved unknown beyond bound |
| Evidence freshness | Age from approved source observation/cutoff to accepted collection | exceeds procedure/source freshness bound |
| Lineage completeness | Downstream-used artifact versions with required manifest fields and resolvable provenance | any required field/ref missing |
| Population readiness | Approved population freezes before test deadline | reconciliation gap or late freeze |
| Review latency | Time from complete workpaper to qualified review decision, excluding documented pause | exceeds engagement risk tier bound |
| Exception age | Time in each accountable exception state | due/escalation policy breach |
| Package integrity | Candidate/frozen packages whose references, classifications, digests, and approvals validate | any invalid/stale/missing dependency |
| Delivery reconciliation | Authorized deliveries reaching verified terminal state | duplicate, wrong destination, unresolved unknown |
| Manual fallback capacity | High-priority work that can be handled during model/source outage | backlog exceeds approved recovery capacity |

Set targets and error budgets per risk tier and engagement deadlines. As an illustrative internal starting point—not an external benchmark—an organization might require 100% lineage/package referential checks, zero unauthorized effects, and a time-bound reconciliation target while allowing a small error budget for non-consequential draft latency. Record the approved values in the service policy, not this guide.

## Alerts that matter

- any hard-control violation or near miss;
- cross-tenant/purpose denial surge or successful anomaly;
- unknown effect/delivery older than its reconciliation objective;
- connector cursor stalled near source retention expiry;
- pagination/count/watermark completeness regression;
- reviewer or exception queue age/capacity breach;
- abnormal approve-without-open duration, low evidence access, or override pattern suggesting rubber stamping;
- profile/connector/release drift or compatibility expiration;
- evidence/package integrity mismatch, hold/deletion conflict, or destination anomaly;
- model unsupported-claim, citation, contradiction, abstention, or cost drift by slice;
- manual fallback backlog beyond recovery capacity.

Each alert has a runbook, owner, severity, suppression rule, and link to exact domain state—not just raw logs.

## Failure mining and continuous evaluation

1. Capture structured rejection, rework, override, contradiction, abstention, reconciliation, incident, and near-miss reason codes.
2. Triage whether the root cause is requirement/profile, source data, connector, schema, context, model/prompt, workflow, policy, UI, reviewer capacity, or operations.
3. Preserve the minimal reproducer with authorized/deidentified data.
4. Add the failure plus neighboring variants to the appropriate test layer.
5. Fix the deterministic boundary first when possible; do not prompt around an authorization or state flaw.
6. Re-evaluate the affected slice, full hard-control suite, and end-to-end trajectory.
7. Shadow/canary the release and monitor for displacement to a different error class.
8. Record corrective/preventive action, owner, due date, and effectiveness review.

Do not automatically train on reviewer approvals. Approvals can be wrong, jurisdiction-specific, conflicted, or tied to an old profile. Curate learning data through governance.

## Evaluation ownership

| Owner | Accountable evaluation |
| --- | --- |
| Profile steward / qualified professional | mapping/procedure/conclusion boundaries and labels |
| Connector/source owner | query semantics, completeness, retention, schema, source controls |
| Security/privacy | isolation, injection, secrets, provider route, retention/hold, malicious content |
| Independent reviewers | workpaper usability, disagreement, burden, automation-bias signals |
| Platform/SRE | state/effect recovery, capacity, SLOs, deployment/rollback |
| Assurance/risk leadership | thresholds, residual risk, authority promotion, exceptions |
| FinOps/product owner | cost/value baseline and review-capacity economics |

The model team cannot self-certify the system for broader authority.

## Release evaluation checklist

- [ ] Hard controls are separate, all pass, and are not averaged into task quality.
- [ ] Metrics are sliced by control type, evidence source/format, tenant risk, language, exception, and version.
- [ ] Gold labels preserve disagreement, uncertainty, basis, and methodology version.
- [ ] End-to-end trajectories assert ordering, authorization, exact inputs, effects, and recovery.
- [ ] Failure injection covers source, connector, evidence, sample, model, context, review, package, tenant, hold, and telemetry failures.
- [ ] Domain, security, and diagnostic records are separate and access/retention controlled.
- [ ] SLOs cover evidence freshness, review/exception age, lineage, reconciliation, fallback, and package integrity.
- [ ] Every incident/near miss produces a governed regression when technically possible.
- [ ] Promotion authority belongs to accountable risk/assurance owners, with rollback ready.

## Anti-patterns

| Anti-pattern | Failure | Correction |
| --- | --- | --- |
| One model “accuracy” number | Hides hard controls, slices, abstention, workflow, and reviewer cost | Claim-specific metrics plus hard gates and trajectories |
| Use production approvals as automatic truth | Encodes old/conflicted/wrong decisions | Curated qualified labels with versions and disagreement |
| Happy-path connector tests | Misses truncation, retention, order, rate, timeout, and schema failure | Fault matrix and source compatibility tests |
| Sampled traces as audit record | Trace loss/redaction changes reconstruction | Application-owned domain/effect/evidence records |
| Alert on every model error | Fatigue obscures consequential failures | Risk-tier alerts with runbooks and aggregation |
| Fix authorization/state bugs in prompts | Nondeterministic text cannot enforce invariants | Deterministic policy/workflow fix plus regression |
