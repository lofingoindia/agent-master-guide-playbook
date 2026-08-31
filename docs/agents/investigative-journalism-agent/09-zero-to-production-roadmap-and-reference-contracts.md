# Zero-to-Production Roadmap and Reference Contracts

## How to use this roadmap

Stages 0–6 are decision gates, not maturity theater. Stop at the lowest stage that solves the newsroom’s actual problem. Every stage below specifies architecture, authority, inputs/outputs, source/claim/case state, events/effects, approvals, recovery, evaluation, and an exit gate.

Do not bring confidential sources, high-risk media, external communication, or publication-adjacent integrations into an earlier stage to create a more impressive demo.

Run the concrete lab and retain the exit artifacts for each stage in [adapter qualification and the worked investigation lifecycle](10-adapter-qualification-and-worked-investigation-lifecycle.md#stage-06-lab-and-exit-evidence). The table is an implementation contract only when the evidence exists; checking a box is not promotion.

## Stage 0 — qualify the problem

Build the deterministic research and records-management baseline first.

| Contract | Stage 0 requirement |
|---|---|
| Architecture | Matter form, source/evidence register, immutable object storage, deterministic capture/import, search/index, OCR queue, claim spreadsheet/database, review checklist; no model loop |
| Authority | Reporters/editors perform research decisions and all external communication; records/source/security/counsel roles are explicit |
| Inputs / outputs | Approved brief and manually acquired evidence → searchable inventory, manually authored claims/timeline, review checklist |
| Source state | Public source register; confidential identities remain in existing approved newsroom practice, not the prototype |
| Claim state | Human-authored `allegation`, `source_statement`, `observation`, `fact`, `inference` labels; no automatic promotion |
| Case state | `draft → admitted → active → review → closed/cancelled`; manual transitions are audited |
| Events / effects | Record acquisitions and human decisions; public-record/contact/publish effects remain outside system or are receipt-only |
| Approvals | Scope, source handling, and review route approved by reporter/editor; security/counsel consulted under existing policy |
| Recovery | Database backup, object fixity, import replay, manual reconciliation; no transcript dependency |
| Evaluation | 30–50 representative matters/tasks; measure manual quality, time, cost, missed contradictions, source-risk incidents, and deterministic search/OCR performance |
| Exit gate | A documented task remains materially slow/error-prone and requires adaptive planning or tool choice; deterministic improvements alone cannot meet the target |

**Successful stop:** A reliable records workspace is a valid final product. Do not add an agent to known forms, monitors, or fixed extraction flows.

## Stage 1 — first bounded advisory loop

Add one public-only, read-only loop over a frozen evaluation corpus.

| Contract | Stage 1 requirement |
|---|---|
| Architecture | Single controller process, one model route, typed search/fetch over frozen/public fixtures, case/claim ledger, hard budgets |
| Authority | Model proposes one action; deterministic policy admits it; no confidential sources, external messages, writes, browser login, or newsroom integration |
| Inputs / outputs | One approved narrow question + frozen public corpus → query/tool trajectory, evidence refs, claim/gap proposals, bounded answer/package preview |
| Source state | Public sources only with origin and capture version; no source identity vault |
| Claim state | Agent proposals start `proposed`; human accepts/rejects; contradictions cannot be deleted |
| Case state | `accepted → running → completed/failed/timed_out/cancelled`; exact terminal result required |
| Events / effects | Model/tool/claim events; only idempotent reads and local derived artifacts; no external effects |
| Approvals | Human approves brief and tool grant before run; no per-read approval if within grant |
| Recovery | Re-run from frozen inputs; duplicate reads safe; no claim accepted from in-memory state alone |
| Evaluation | Compare with deterministic/manual baseline on claim support, contradiction recall, authority, abstention, latency, cost, and repeated reliability |
| Exit gate | Loop improves a declared metric without unsupported claims, source-class confusion, authority violations, or unacceptable cost variance |

Reference loop:

```text
for at most N decisions:
  state = load_case_version()
  if deterministic_completion_or_stop(state): terminal()
  context = compile_one_gap(state)
  proposal = model.propose_one_action(context)
  action = policy.admit(proposal, state)
  result = tool.execute(action)
  ledger.append(result, claim_proposals)
```

## Stage 2 — useful MVP

Connect to approved public environments and produce a reporter review package.

| Contract | Stage 2 requirement |
|---|---|
| Architecture | Control service + relational DB + encrypted object storage + restricted public acquisition + sandboxed document transforms + package renderer |
| Authority | Read public sources; write only internal evidence/claim/package records; reporter reviews every material claim; no source contact or publication |
| Inputs / outputs | Approved matter, public URLs/APIs/records, uploaded non-confidential docs → immutable evidence, OCR/translation derivatives, claims/entities/timeline, reporter package |
| Source state | Public account/publication/record identity, capture provenance, possible dependency graph; confidential sources still excluded |
| Claim state | Typed epistemic classes, evidence edges, contradictions, origin paths, human confirmation, unresolved safe outcome |
| Case state | Add `waiting_reporter`, `package_candidate`, `safety_hold`; durable enough for ordinary restart |
| Events / effects | Acquisition/transform/claim/package events; archive-capture request may be approved effect; no contact/CMS effect |
| Approvals | Matter admission and tool grant; human review of entity merges, material translations/quotes, authenticity assessments, package |
| Recovery | Content-addressed objects, operation IDs, page/coverage checks, resumable transforms, rebuildable indexes/package |
| Evaluation | Live-provider contract fixtures, hostile docs, archive drift, social ancestry, OCR/translation, entity/time, package leakage, cost/latency |
| Exit gate | Reporter can reproduce every material claim from IDs; public acquisition/transform failures are explicit; no model text bypasses the ledger |

MVP ceiling:

- one newsroom/tenant and a few matter classes;
- public, lawfully accessible evidence;
- no confidential source identity or direct contact;
- no autonomous browser forms, records submissions, legal conclusions, or CMS publication;
- no cross-matter long-term or episodic memory.

## Stage 3 — reliable v1

Add multi-day durability, confidential-source compartmentation, and versioned integrations.

| Contract | Stage 3 requirement |
|---|---|
| Architecture | Durable workflow/queue; source identity vault and human-operated intake; quarantine/sensitive-analysis zone; leases/fencing/outbox; versioned records/archive/social/newsroom package adapters |
| Authority | General model sees blind source refs and approved derivatives only; source custodians control identity/export; external request/contact/package export requires scoped human approval; publication remains absent |
| Inputs / outputs | Stage 2 inputs plus human-exported confidential derivatives, source terms, long waits and responses → compartment-aware reporter/editor/specialist packages |
| Source state | Identity, relationship/ground rules, statement, evidence, disclosure and risk records separated; independence and re-identification tracked |
| Claim state | Durable versions, human confirmation, stale/freshness triggers, source-disclosure constraints, correction dependencies |
| Case state | Full waiting reasons, attempt/lease, cancellation, `indeterminate`, package revisions, review decisions, reopen/supersede |
| Events / effects | Stable event/effect IDs; public-record request/contact/package export intents and receipts; ambiguous effects reconcile before retry |
| Approvals | Digest-bound, expiring source export, contact, package, specialist and review approvals; resume revalidates current policy and source safety |
| Recovery | Checkpoints reference ledger IDs; crash-before/after every effect tested; late workers fenced; indexes rebuilt; compaction receipts and repeated-compaction tests |
| Evaluation | Failure injection, identity/mosaic leakage, cross-compartment retrieval, duplicate callbacks/effects, long waits, model/tool/schema upgrades, deterministic fallback |
| Exit gate | Crash/retry/compaction/deployment tests lose no evidence, expose no identity, repeat no effect, and preserve contradictions, terms, approvals, and budgets |

Long-term user memory remains disabled. Source relationship continuity lives in the vault under human governance, not in semantic model memory.

## Stage 4 — production readiness

Establish enforceable security, release, operational, and incident controls.

| Contract | Stage 4 requirement |
|---|---|
| Architecture | Production identity, capability broker, per-matter authorization, separated worker identities/networks, secret broker, audit, redacted tracing, release pipeline, backups/restore, on-call |
| Authority | Least privilege by tenant/matter/compartment/purpose/time; source vault/CMS credentials never enter model plane; newsroom professionals own all high-impact decisions |
| Inputs / outputs | Classified production matters under approved profiles → reviewed packages, bounded unknown/escalation, operational and audit evidence |
| Source state | Retention/hold/deletion, access review, threat plan, compromise/safety-hold and incident states operational |
| Claim state | Release policy enforces type/evidence/invalidation; central claims and corrections receive independent review |
| Case state | State-version fencing, exactly one run terminal, safety holds, approval expiry, deployment pin/migrate/restart policy |
| Events / effects | Transactional intent/receipt/outbox; authenticated resume; CMS publication receipt may be imported but the agent still cannot initiate publish |
| Approvals | Role/segregation-of-duty matrix, step-up identity for vault, emergency authority, commit-time package/policy/recipient revalidation |
| Recovery | Tested restore, incident modes, credential revocation, evidence/package invalidation, source-exposure and erroneous-publication runbooks |
| Evaluation | Offline gates + shadow + canary; red-team source leakage/injection/tenant isolation; load/soak; operator incident exercises; privacy/deletion verification |
| Exit gate | SLOs, alerts, runbooks, rollback, data governance, security review, threat-specific exercises, and hard eval gates pass; model outage leaves deterministic newsroom work available |

Release manifest:

```yaml
production_release:
  release_id: journalism-prod/2026-09-15.1
  runtime_bundle: release_bundle/17
  policy_bundle: newsroom-policy/42
  schema_bundle: journalism-domain/9
  tool_adapter_bundle: tools/31
  eval_report: eval-report/2026-09-12
  approved_matter_profiles: [J0, J1]
  excluded_profiles: [J2, J3, J4]
  migration_policy: pin_existing_runs
  rollback_to: journalism-prod/2026-09-01.3
```

## Stage 5 — scale and resilience

Scale measured bottlenecks, not the number of agents.

| Contract | Stage 5 requirement |
|---|---|
| Architecture | Admission service; behavior-specific bounded queues/pools; fair tenant quotas; workload reservations; tiered governed storage; optional regional placement/failover by policy |
| Authority | Same or narrower capabilities; scaling never broadens source/data/tool access; per-pool identities and egress remain explicit |
| Inputs / outputs | Concurrent matters, larger records/media, scheduled monitoring under owned profiles → deadline-aware partial/complete packages with cost attribution |
| Source state | Matter-local tokens and vault partitions; no shared semantic identity index; capacity pressure cannot downgrade protection |
| Claim state | Deterministic partition/merge by IDs; parallel branches cannot last-write-wins contradictions or evidence |
| Case state | Queue/admission/lease epochs, branch budgets, pause/cancel, disaster-recovery and stale-work semantics |
| Events / effects | Partition keys and per-run order documented; dedupe retention; external-effect queue isolated from read/transform queues |
| Approvals | Human review capacity included in admission; high-risk/correction/source-safety lanes reserved; approval queues expose expiry |
| Recovery | Queue replay, dead-letter/poison work, partial-region/provider failure, storage/index rebuild, retry-storm and late-worker controls |
| Evaluation | Peak + failure reserve load, noisy-tenant, oversized media, provider quota outage, review backlog, regional loss, cost and cleanup tests |
| Exit gate | One tenant/matter/provider/tool cannot exhaust the service; deadlines and safe degradation hold; evidence durability/isolation and costs meet objectives under partial failure |

Scale ladder:

1. optimize queries/transforms and eliminate duplicate work;
2. add worker concurrency within a single region;
3. partition by workload and matter, then add fair admission;
4. reserve source-safety/correction/deadline capacity;
5. add regional/data-residency placement only when legal/security/latency/recovery evidence requires it;
6. add parallel research workers only when evaluation shows net coverage/latency gain after merge and review cost.

## Stage 6 — continuous evolution

Turn confirmed failures and corrections into governed improvement without training on secrets or editorial preference by accident.

| Contract | Stage 6 requirement |
|---|---|
| Architecture | Versioned dataset/fixture store, replay platform, shadow/canary routing, change registry, drift/refresh jobs, affected-output invalidation |
| Authority | Human review admits every fixture, policy, procedure, and memory; acting agent cannot self-modify or promote a successful trajectory |
| Inputs / outputs | Corrections, misses, reviewer edits, incidents, drift, model/tool/corpus releases → minimal fixtures, change proposal, eval report, rollout/rollback/deprecation record |
| Source state | Real source identities/content excluded unless exceptional consent/governance; de-identification and re-identification reviewed; vault relationships never become global memory |
| Claim state | Correction lineage links outcome to affected claim/evidence/decision; editor change reason distinguishes fact, law, safety, style, and judgment |
| Case state | Historical runs pinned and reproducible; affected active runs pin, migrate, restart, or require review |
| Events / effects | `failure.confirmed`, `fixture.approved`, `bundle.evaluated`, `rollout.changed`, `output.invalidated`; rollout actions owned by release workflow |
| Approvals | Dataset owner + security/privacy/editorial review; model/tool/policy/corpus change approval; deprecation and rollback owner |
| Recovery | Roll back bundle/router, disable model/tools/memory independently, re-run affected projections, restore deleted/poisoned indexes from authoritative records |
| Evaluation | Offline/repeated/adversarial/compatibility → shadow → limited canary → progressive rollout; delayed correction and reviewer outcomes monitored |
| Exit gate | Every material unsafe action or confirmed miss has a fixture/control or documented non-automatable owner; stale components/corpora have refresh/deprecation paths; agentic steps can be removed when no longer justified |

## Reference storage layout

```text
relational state
├── matters / matter_revisions / policy_grants / run_budgets / leases
├── blind_sources / ground_rule_refs / source_disclosure_profiles
├── acquisitions / transforms / evidence_items / evidence_access_events
├── claims / claim_versions / evidence_edges / independence_sets
├── contradictions / entities / aliases / entity_merge_decisions
├── timeline_events / temporal_relations / coverage_gaps
├── plans / tasks / tool_commands / tool_results
├── review_packages / review_decisions / fairness_contacts
├── external_effects / effect_receipts / correction_cases
├── application_events / inbox / outbox
└── release_bundles / evaluation_results / invalidations

identity vault (separate authority and keys)
├── source_identities / contact_routes / verification
├── relationship_terms / safety_plans / risk incidents
└── access audit / retention / holds / deletion

governed object storage
├── raw/<matter>/<digest>
├── captures/<matter>/<capture-id>
├── derived/<matter>/<transform-id>/<digest>
├── restricted-source/<compartment>/<digest>
├── review-packages/<audience>/<package>/<revision>
├── released-manifests/<story>/<revision>
└── eval-fixtures/<dataset>/<version>
```

Content-addressing does not remove matter/tenant authorization. Object keys, encryption context, metadata, and index entries must retain scope.

## Reference domain enums

```yaml
epistemic_type:
  - allegation
  - source_statement
  - observation
  - authentic_artifact
  - corroborated_fact
  - inference
  - editorial_conclusion
  - publication_effect

claim_status:
  - proposed
  - attributed
  - unsupported
  - corroborated
  - contradicted
  - unresolved
  - human_confirmed
  - superseded
  - closed_no_finding

case_run_status:
  - admitted
  - active
  - waiting_human
  - package_candidate
  - review_complete
  - external_pending
  - safety_hold
  - indeterminate
  - completed
  - failed
  - cancelled
  - timed_out

evidence_relation:
  - supports
  - contradicts
  - limits
  - contextualizes
  - defines
  - quotes
  - derived_from

external_effect_status:
  - prepared
  - approved
  - dispatched
  - confirmed
  - rejected
  - indeterminate
  - reconciled
  - cancelled
```

Unknown safety-critical enum values fail closed or enter an explicit compatibility path. They never default to corroborated, approved, confirmed, or completed.

## Reference event envelope

```json
{
  "event_id": "evt_01...",
  "event_type": "journalism.review.package_frozen",
  "event_version": 1,
  "occurred_at": "2026-08-31T12:34:56.789Z",
  "tenant_id": "newsroom_ref",
  "matter_id": "matter_204",
  "run_id": "run_44",
  "attempt_id": "attempt_03",
  "step_id": "step_package_7",
  "causation_id": "evt_prior...",
  "sequence": 91,
  "case_version": 58,
  "compartment": "editor_review",
  "sensitivity": "newsroom_confidential",
  "schema_id": "journalism-events/1",
  "release_id": "journalism-prod/2026-09-15.1",
  "policy_version": "newsroom-policy/42",
  "traceparent": "00-...-...-01",
  "data": {
    "package_id": "pkg_220",
    "package_digest": "sha256:...",
    "required_reviews": ["reporter", "editor", "counsel"]
  }
}
```

Never place source identity, raw content, or privileged rationale in the general event envelope. Use protected references and audience-specific projections.

## Reference effect contract

```yaml
external_effect:
  effect_id: effect_contact_33_v1
  semantic_operation: send_fairness_questions
  tenant_id: newsroom_ref
  matter_id: matter_204
  target_ref: newsroom-contact://vendor_9
  payload_ref: artifact://contact-draft/33
  payload_digest: sha256:...
  preconditions:
    case_version: 57
    approval_id: approval_contact_71
    approval_digest: sha256:...
    deadline_not_expired: true
  idempotency_key: matter_204:vendor_9:questions_v1
  status: prepared
  attempts: []
  receipt_ref: null
  reconcile_by: target_message_id
```

The model can propose the draft and effect intent. A human-owned adapter validates and executes it.

## Minimum invariants

Enforce with database constraints plus transactional application logic:

```text
UNIQUE (tenant_id, matter_id, event_sequence)
UNIQUE (tenant_id, effect_id)
UNIQUE (package_id, revision)
UNIQUE (claim_id, version)
UNIQUE (acquisition_id)

claim.status = human_confirmed
  => claim.confirmed_by IS NOT NULL

review_decision.package_digest
  = review_package.digest at commit

publication_effect.status = confirmed
  => receipt_ref IS NOT NULL

derived_evidence
  => input_evidence_ids not empty AND transform_receipt exists

model_plane_source_identity_access = denied
model_plane_publish_capability = absent
```

Also enforce:

- one case-run terminal outcome;
- stale lease cannot write state or attach an artifact;
- allegation/source statement cannot auto-transition to corroborated fact;
- every material draft statement references permitted claim versions;
- dependency paths cannot be counted as independent corroboration;
- package export cannot read fields outside its allowlist/compartment;
- approval expires on material claim/evidence/wording/policy/jurisdiction/recipient change;
- deletion tombstones prevent re-import from stale indexes/backups according to policy;
- a no-result claim requires explicit source coverage and health.

## Approval policy skeleton

```yaml
approval_policy:
  version: newsroom-approvals/8
  rules:
    - when:
        capability: public_read
        matter_profile: [J0, J1]
        target: approved_public
      decision: allow_within_matter_budget

    - when:
        operation: export_source_derivative
      decision: require_source_custodian
      bind: [matter_id, source_ref, artifact_digest, audience, expiry]

    - when:
        operation: [send_records_request, send_fairness_contact]
      decision: require_reporter_or_editor
      bind: [recipient, exact_payload_digest, deadline, channel, idempotency_key]

    - when:
        operation: create_editorial_or_counsel_package
      decision: require_package_role_policy
      bind: [package_digest, audience, fields, expiry]

    - when:
        operation: [publish, unpublish, correct, retract]
      decision: deny_to_agent_plane

    - when:
        operation: [identify_anonymous_source, bypass_access, surveillance, deception]
      decision: deny_and_security_event
```

## Final production checklist

### Mission and authority

- [ ] The deterministic alternative was measured and the loop has a proven advantage.
- [ ] The system is advisory and cannot make source promises, legal/editorial judgments, contact decisions, or publication effects.
- [ ] Every stage has an explicit scope ceiling and rollback to deterministic operation.

### Source and evidence

- [ ] Identity, relationship, statements, raw evidence, and approved derivatives are separately protected.
- [ ] Pseudonyms are matter-local and exports receive re-identification review.
- [ ] Acquisition, integrity, provenance, authenticity, context, and factual support are distinct.
- [ ] OCR, translation, archive, social, metadata, C2PA, and detector limitations remain visible.

### Claims and review

- [ ] The eight epistemic types are never collapsed.
- [ ] Claims, contradictions, origin paths, entities, timelines, gaps, and human decisions are versioned.
- [ ] Review packages use allowlisted audience projections and exact digests.
- [ ] Fairness, public interest, harm, legal, and publication decisions have accountable human owners.

### Runtime and security

- [ ] State, events, effects, artifacts, traces, and source identity have separate authority.
- [ ] Context/compaction cannot launder hostile content or lose contradictions/terms/effects.
- [ ] Memory classes and multi-agent use are explicitly enabled or rejected.
- [ ] Matters/tenants/compartments are isolated before retrieval, caching, or dedupe.
- [ ] External effects are idempotent/reconciled and the model has no CMS/source-channel credentials.

### Operations and evolution

- [ ] SLOs, queues, backpressure, cost, tracing, runbooks, backup/restore, and incident modes are exercised.
- [ ] Eval suites include normal, boundary, adversarial, failure, cancellation, partial-effect, and long-run cases.
- [ ] Source leakage, unauthorized access/contact/publication, and cross-tenant influence are hard release failures.
- [ ] Corrections/failures feed governed fixtures, not automatic memory.
- [ ] Every model/tool/parser/adapter/schema/policy/corpus change is pinned, replayed, shadowed, canaried, and reversible.

## Smallest production-worthy deployment

Begin with one newsroom, public-only J0/J1 matters, one restricted fetch/search adapter, deterministic web/document capture, immutable evidence, a typed claim ledger, one bounded loop, and reporter-only packages. Disable confidential-source intake, direct communication, authenticated social sessions, biometrics, synthetic-media verdicts, long-term/episodic memory, multi-agent fan-out, and every publish capability.

Add each excluded capability only when its threat model, human owner, recovery path, and evaluation gate are independently ready.
