# Steward Cases, Quality Repair, and Merge/Split Workflows

A steward case is a durable, evidence-bound decision process. It converts a detected discrepancy or candidate into an auditable disposition and, when authorized, a set of recoverable external effects. A chat transcript, ticket comment, or model explanation is not the system of record.

## Case types

Use typed workflows because their authority, evidence, and reversal needs differ.

| Case type | Typical trigger | Possible outcomes | Minimum authority |
|---|---|---|---|
| Identity link | Candidate records exceed review threshold | link, non-link, defer, request evidence | Entity steward |
| Merge | Approved cluster change or duplicate masters | merge/alias, partial consolidation, reject | MDM authority; dual approval for high risk |
| Split/unlink | Appeal, contradiction, false merge | split, unlink edge, remap assertions, reject | MDM authority; incident route if propagated |
| Quality repair | SHACL/rule/statistical violation | correct source, correct mapping, accept exception, revise rule | Field/source owner by authority matrix |
| Glossary/vocabulary | New or changed term/mapping | approve, revise, deprecate, reject | Vocabulary owner/reviewer |
| Ontology/schema | Semantic bundle proposal | approve exact release, reject, stage migration | Semantic change board or delegated owner |
| Lineage conflict | Contradictory/missing dependency evidence | accept source, retain both, invalidate observation, instrument source | Platform/data owner |
| Ownership/classification | Weak or conflicting assignment | assign, remove, escalate, no change | Domain owner/security authority |
| Lifecycle/delete | Deprecate, soft delete, hard delete, purge | typed lifecycle effect, reject, legal hold | Owner plus policy-specific approver |

## Case envelope

```yaml
case:
  case_id: case-01K...
  tenant_id: tenant-a
  case_type: entity-split
  state: evidence_ready
  risk_tier: critical
  subject_refs: [urn:party:master:991]
  trigger:
    type: contradiction-detector
    observation_id: obs-01K...
  authority_policy: identity-policy-v12
  evidence_manifest_digest: sha256:...
  proposal_digest: null
  assigned_queue: identity-critical
  created_at: 2026-08-31T09:00:00Z
  due_at: 2026-08-31T13:00:00Z
  version: 7
```

All state transitions use optimistic concurrency on `version`. The case points to immutable evidence; it does not copy mutable source records into reviewer prose.

## State machine

~~~mermaid
stateDiagram-v2
    [*] --> Detected
    Detected --> CollectingEvidence
    CollectingEvidence --> EvidenceReady
    EvidenceReady --> Proposed
    Proposed --> MoreEvidence
    MoreEvidence --> CollectingEvidence
    Proposed --> Rejected
    Proposed --> Approved
    Approved --> Prepared
    Prepared --> Applying
    Applying --> Unknown
    Unknown --> Applied
    Unknown --> NotApplied
    NotApplied --> Prepared
    Applying --> Applied
    Applied --> Reconciling
    Reconciling --> Completed
    Reconciling --> Recovery
    Recovery --> Proposed
    Rejected --> [*]
    Completed --> [*]
~~~

`Approved` means an authorized principal approved an exact proposal digest. It does not mean the target changed. `Applied` requires an authoritative receipt or target observation. `Completed` requires reconciliation and postconditions.

## Evidence manifest and reviewer packet

The packet should let a reviewer reproduce the decision without giving the model unrestricted source access.

```yaml
evidence_manifest:
  manifest_id: evm-01K...
  case_id: case-01K...
  created_from_revision: 7
  entries:
    - evidence_id: ev-1
      type: source-record
      uri: evidence://tenant-a/source/crm/record/442/version/19
      digest: sha256:...
      source_time: 2026-08-28T12:00:00Z
      observed_at: 2026-08-31T09:01:00Z
      visibility_label: restricted-pii
    - evidence_id: ev-2
      type: comparison-features
      uri: evidence://tenant-a/matcher/run-77/pair-8
      digest: sha256:...
  generation:
    policy_version: evidence-pack-v5
    connector_manifest: conn-set-82
  expires_at: 2026-09-02T09:00:00Z
```

The reviewer UI presents:

- raw values beside normalized/computed values;
- evidence provenance, time, source revision, and visibility;
- agreeing and contradicting signals;
- model or matcher score with calibration slice—not as a verdict;
- current cluster or graph neighborhood and the proposed delta;
- downstream blast radius and irreversible consequences;
- permitted actions based on the reviewer's current authority;
- alternative hypotheses, `unknown`, and a request-more-evidence option.

Redact or tokenize fields the reviewer does not need. Never embed secrets or unrestricted PII in explanations, logs, or reusable examples.

## Proposal and approval binding

```yaml
proposal:
  proposal_id: prop-01K...
  case_id: case-01K...
  action: split-canonical-entity
  subject_version: 44
  proposed_partition:
    - canonical_id: urn:party:master:991
      assertion_ids: [a1, a2, a8]
    - canonical_id: pending-new-id
      assertion_ids: [a3, a5]
  aliases_to_rewrite: [alias-17]
  derived_views_to_rebuild: [search, risk-rollup]
  external_effects: [mdm-split, catalog-remap]
  evidence_manifest_digest: sha256:...
  policy_version: identity-policy-v12
  proposal_digest: sha256:...
```

```yaml
approval:
  decision_id: dec-01K...
  proposal_digest: sha256:...
  subject_version: 44
  principal_id: steward-218
  authority_snapshot: authz-01K...
  role: senior-identity-steward
  disposition: approved
  conditions: [second-approver-required]
  decided_at: 2026-08-31T09:30:00Z
  expires_at: 2026-08-31T13:30:00Z
  signature: sig:...
```

Recalculate the digest after any change. Expire approval when the subject version, policy, evidence visibility, target capability, or material blast radius changes. The proposer cannot approve their own high-risk proposal; OpenMetadata's current governance workflow similarly documents self-approval prevention for governed changes, but this blueprint enforces the rule in its own authority layer rather than depending on a target UI.

## Data-quality repair is causal, not cosmetic

A quality observation must retain rule semantics and input scope:

```yaml
quality_observation:
  rule_id: dq.customer.birth_date.not_future
  rule_version: 6
  data_snapshot: customer-master-882
  subject: urn:party:master:991
  observed_value_ref: evidence://.../birth_date
  result: fail
  severity: high
  evaluated_at: 2026-08-31T09:02:00Z
  evaluator_build: dq-engine-2.8.1
```

Then classify likely root cause before proposing repair:

| Cause class | Correct repair target |
|---|---|
| Source value is wrong | System-of-record correction or governed master override |
| Extraction/transformation is wrong | Connector or mapping revision, followed by replay |
| Identity is wrong | Link/merge/split case |
| Rule is wrong or too broad | Rule/shape proposal and revalidation |
| Value is legitimately exceptional | Time-bounded exception with owner and rationale |
| Evidence is insufficient | Remain unknown; request evidence |

Never “repair” the canonical graph alone if the authoritative source will overwrite it on the next sync. Conversely, do not write back to a source that is not authoritative for that field.

## Merge workflow

1. Freeze the candidate set against explicit source/master revisions.
2. Re-evaluate pair evidence and cluster constraints with the current policy.
3. Select survivorship per attribute and record why; do not assume one golden record wins every field.
4. Enumerate source identifiers, aliases, dependent facts, consent/retention constraints, and downstream references.
5. Produce the before/after assertion assignment, not only a target ID.
6. Obtain risk-based approval of the exact proposal.
7. Prepare effects with conditional versions and idempotency keys.
8. Apply in a declared order, recording per-effect receipts.
9. Rebuild derived facts/indexes from retained assertions and decisions.
10. Reconcile targets and monitor appeals/drift.

Attribute survivorship policy can prefer an authoritative source, verified status, recency within a source, completeness, or a reviewed override. IBM's MDM documentation describes matching separately from survivorship and preserves source identifiers/provenance; that separation is the important design principle, regardless of product choice. See [IBM matching algorithms](https://www.ibm.com/docs/en/ws-and-kc?topic=data-matching-algorithms) and [master-data concepts](https://www.ibm.com/docs/en/ws-and-kc?topic=data-concepts-in-master-management).

## Split and unlink workflow

Splits are harder because earlier merges may have propagated a canonical identity and derived values.

1. Stop new high-risk effects involving the affected canonical entity.
2. Snapshot assertion membership, decision history, aliases, and external references.
3. Identify the incorrect edge or cluster decision; preserve the original decision as superseded.
4. Partition **source assertions**, including temporal intervals, instead of copying the current golden row.
5. Recompute survivorship independently for every resulting entity.
6. Classify downstream effects as reversible, compensatable, rebuildable, or irreversible.
7. Obtain approval commensurate with blast radius.
8. Apply source/MDM operations using target-native split/unmerge contracts where supported.
9. Remap catalog/graph references and rebuild derived projections.
10. reconcile and notify owners of irreversibly exported data.

If an external system cannot unmerge, retain a glue/alias record and an explicit dispute mapping rather than falsifying success. Recovery may require a forward correction, not rollback.

## Worked stewardship flows

Every flow uses the same separation: observe immutable evidence, create a versioned proposal, decide under current authority, prepare target-specific effects, then reconcile. The examples name concrete failure behavior so an implementation can turn them into executable scenarios.

~~~mermaid
sequenceDiagram
    participant S as Source/producer
    participant L as Assertion + case ledgers
    participant A as Analysis/proposer
    participant H as Human/authority
    participant E as Effect executor
    participant T as Target + consumers
    S->>L: versioned observation/assertion
    L->>A: authorized evidence manifest
    A->>L: typed proposal + alternatives
    L->>H: exact digest + impact packet
    H->>L: approve/reject/defer
    L->>E: prepared effect + fence
    E->>T: conditional target operation
    T-->>E: receipt or ambiguous result
    E->>L: reconcile + invalidate/rebuild
~~~

### Flow A — admit a new entity without inventing identity

**Situation:** a procurement source emits supplier `erp-eu/vendor/V-204` revision 9, not found in the canonical MDM crosswalk.

1. Intake verifies tenant, namespace, entity type, source revision, rights, valid interval, and payload digest; it stores the source record and assertions.
2. Exact registry/crosswalk checks return no match. Blocking produces two candidates and records that the corporate registry identifier is unavailable.
3. The scorer places both in the review band. The packet shows alternatives, one conflicting incorporation date, source visibility, and the cost of a false merge.
4. The steward chooses `NEW_ENTITY`, not “best of two.” The canonical owner mints `org-1204`; the decision binds assertion membership and alias plan.
5. The executor creates the canonical entity and source crosswalk under target preconditions, then reads both back.
6. Search and catalog projections rebuild from the committed decision. The case closes with the minted ID, MDM version, projection checkpoints, and no unresolved effect.

**Failure boundary:** if the registry lookup was forbidden rather than empty, the result is insufficient evidence; admission cannot silently reinterpret it as “no existing entity.”

### Flow B — preserve and resolve a fact conflict

**Situation:** CRM revision 71 asserts country `DE`; the legal registry revision 12 asserts `AT` for the same accepted legal entity.

1. Both assertions remain active within their source authority; the system creates a conflict observation rather than overwriting either.
2. The authority matrix says the registry owns legal domicile while CRM owns service address. The proposed repair changes the mapping, not the CRM source value.
3. A steward confirms the field-semantics mismatch and approves a canonical-fact recomputation plus a CRM schema-description repair request.
4. The canonical projection publishes legal domicile `AT` derived from registry assertion 12; the CRM assertion remains queryable as service-address evidence.
5. Consumers of the old canonical fact receive invalidations and acknowledge rebuild.

**Close evidence:** two retained assertions, mapping correction, new canonical-fact version and valid interval, source-owner ticket, consumer acknowledgements, and an explained prior fact.

### Flow C — merge two canonical entities

**Situation:** `org-501` and `org-884` have distinct members but a verified, namespace-correct tax identifier and overlapping legal validity.

1. Freeze member/source revisions and validate the identifier issuer, temporal overlap, cannot-link suppressors, source-cardinality rules, and cluster constraints.
2. Preview survivor `org-501`, retiring `org-884`, per-attribute survivorship, aliases, policies, contracts, cases, lineage, and all downstream references.
3. Two independent authorized reviewers approve the exact proposal digest because the risk tier is high.
4. The executor rechecks target versions, applies the target-native merge, and records survivor/retired IDs and target operation ID.
5. Reconciliation compares observed membership and aliases with the proposal. Derived facts are recomputed; dependent indexes and consumers advance to the new decision watermark.

**Abort rule:** a new source revision, hard conflict, expired approval, or changed target version invalidates the proposal before dispatch.

### Flow D — split or unmerge after a false merge

**Situation:** an appeal proves a “glue” record joined two legal organizations; exports already used the merged ID.

1. Fence new effects on the canonical ID and open a critical incident-linked split case.
2. Reconstruct pre-merge assertion membership, then include post-merge assertions and facts; do not restore an old snapshot as if nothing changed.
3. Partition source assertions by valid interval, recompute survivorship for both resulting entities, and classify each downstream reference as reassignable, duplicated-with-warning, review-required, or irreversible.
4. Approvers accept a forward split plan. If the MDM supports unmerge, execute and read it back; otherwise create owner-minted canonical IDs plus explicit dispute/redirect records and disclose that physical unmerge is unavailable.
5. Invalidate facts derived from mixed membership, rebuild projections, notify export owners, and attach compensation evidence for irreversible disclosures.

**Close evidence:** no assertion is orphaned, every alias/reference has a disposition, both clusters pass constraints, unknown exports remain open, and the erroneous decision is superseded rather than erased.

### Flow E — publish an ontology change

**Situation:** the ontology owner proposes making `ex:legalDomicile` functional and adding a disjointness axiom.

1. Build an exact semantic bundle and compute dependency, inference, SHACL, query, classification, and consumer deltas on a representative snapshot.
2. Results show 31 functional conflicts and two new inconsistencies. The model may summarize them; the reasoner and validators produce the evidence.
3. The owner changes the proposal to an advisory shape plus a migration, then reviewers approve the revised digest.
4. Shadow/canary compares old/new query results, closure size, latency, and case volume. Publication advances the bundle pointer only after gates pass.
5. Old derived entailments are invalidated and rebuilt; source assertions are never deleted. Consumers acknowledge the new bundle and migration state.

**Rollback:** route reads to the previous bundle and rebuild derived closure. Persistent published IRIs are deprecated/superseded, not reused.

### Flow F — propagate a source correction or deletion

**Situation:** a person disputes an inaccurate phone number and the authorized privacy workflow requests correction plus restricted processing during review.

1. Verify subject/source identity and authority; append restriction and correction events. Do not alter historical receipts.
2. Freeze matching features and decisions that depend on the phone assertion. Traverse pinned provenance/lineage and record unknown coverage.
3. Supersede the source assertion, recompute candidates and canonical facts, delete/re-embed authorized vectors, invalidate caches/evaluation examples, and notify known recipients.
4. A legal hold retains a minimized decision record but prohibits normal use; that exception is visible in the propagation case.
5. Close only when each required target is verified corrected/deleted/restricted or has an authorized retention disposition.

**Safety rule:** deletion never follows an unreviewed fuzzy identity link; over-deletion is also a privacy and integrity failure.

### Flow G — invalidate downstream consumers

**Situation:** a split changes the canonical supplier used by risk scoring, procurement reports, search, and an external export.

1. At a pinned lineage revision, classify descendants by derivation relation and coverage. High-degree or truncated traversal creates follow-up partitions.
2. Emit versioned invalidations with affected fact versions, minimum source watermark, security label, action, and deadline.
3. Internal projections rebuild from assertions/decisions; the risk model is held from new scoring until features are recomputed; the external recipient receives a correction notice.
4. Reconciler verifies projection checkpoints, consumer acknowledgements, and sampled content. It never treats message delivery as rebuild completion.

**Terminal states:** verified rebuilt, authorized not-applicable, retained under policy, or explicitly unresolved. An unresponsive critical consumer prevents case completion.

### Flow H — reconcile an unknown external effect

**Situation:** an approved Atlas propagating-classification request times out after dispatch.

1. Persist `unknown` with request digest, target GUID/version, propagation flags, remote correlation data, and fence; open the target write circuit for that effect key.
2. A separate principal reads the entity, classifications, propagated descendants, and audit trail. It compares actual affected GUIDs with the approved preview.
3. If exact postconditions hold, mark applied and continue reconciliation. If no effect is visible and the adapter's consistency window has elapsed, mark not-applied and allow a fenced retry.
4. If partially or unexpectedly propagated, keep the effect unknown/recovery-required, stop similar writes, and execute the approved removal/forward-correction runbook.

**Prohibited response:** retrying merely because the HTTP client timed out.

### Flow I — apply a human override without creating a hidden rule

**Situation:** a senior steward rejects a 0.995 match because two regulated identifiers prove the records are different.

1. The UI requires a reason code, evidence references, relation/scope, valid interval, and whether the decision is an expiring exception or a durable non-link.
2. Policy verifies authority and separation of duties. The decision supersedes the proposal; it does not edit the model score or source evidence.
3. A versioned suppressor prevents immediate reproposal under the same evidence/policy while preserving an appeal path.
4. Evaluation owners may later curate the case into a de-identified gray/gold set after independent adjudication. Production thresholds do not learn directly from the click.

**Review boundary:** a reviewer may decide only the displayed proposal and scoped follow-on effect. Free-text rationale cannot grant broader mutation authority.

## Exceptions and waivers

An exception is a governed fact, not a comment:

```yaml
exception:
  exception_id: exc-01K...
  rule_id: dq.customer.birth_date.not_future
  scope: urn:source:test-fixtures
  reason_code: synthetic-future-dates
  owner: team-data-quality
  approved_by: reviewer-88
  valid_from: 2026-08-31
  expires_at: 2026-09-30
  compensating_controls: [exclude-from-mastering]
  evidence_refs: [ev-71]
```

Require bounded scope and expiry. Track exception count, age, repeated renewals, and downstream use. An exception does not alter the underlying assertion or prove it correct.

## Queue design and service levels

Route cases by tenant, domain, case type, risk, skill, data classification, and conflict of interest. Keep workload balanced without changing authority.

Suggested priority score:

```text
priority = harm_if_wrong * propagation_radius * time_sensitivity
           * evidence_readiness * policy_multiplier
```

Use explicit bands, not a mysterious model-generated number. Age boosts may prevent starvation, but never bypass approval. Separate queues for urgent false merges/security classifications from low-risk vocabulary cleanup.

Track:

- time to evidence-ready, first review, decision, effect, and reconciliation;
- reopen, appeal, disagreement, and reversal rates;
- queue age and oldest case by risk tier;
- reviewer throughput and calibrated agreement by slice;
- fraction of cases lacking adequate evidence;
- effects blocked by stale approval or target version.

## Appeals and audit

An appeal creates a linked new case. It does not edit the old decision. The audit view reconstructs:

```text
trigger -> evidence manifest -> proposal -> authority snapshot -> decision
        -> effect attempts/receipts -> reconciliation -> later supersession/appeal
```

Audit export must preserve identifiers and digests but enforce tenant and field-level visibility. Auditors do not automatically receive raw PII.

## Failure and recovery matrix

| Failure | State | Required response |
|---|---|---|
| Evidence changes during review | proposal stale | Refresh packet and proposal; approval invalid |
| Reviewer lacks current scope | unauthorized | Reject transition; retain attempt in security audit |
| Approval expires before effect | expired | Re-evaluate and reapprove |
| First of several effects succeeds | partial | Stop, reconcile each target, run declared compensation/forward repair |
| Write times out | unknown | Read target; never blind-retry |
| Merge later proven false | appealed/incident | Freeze propagation, execute split workflow, evaluate affected outputs |
| Target cannot reverse | recovery-required | Use compensating mapping/glue record; disclose limitation |
| Model invents evidence | invalid proposal | Reject missing evidence IDs; log evaluation failure |
| Queue exceeds capacity | degraded | Keep ingestion/evidence durable; reduce auto-proposals; prioritize high risk |

## Checklist

- [ ] Every case type has its own inputs, dispositions, authority, and recovery rules.
- [ ] Case transitions use durable state and optimistic concurrency.
- [ ] Evidence is immutable, digest-bound, provenance-rich, and least-privilege.
- [ ] Approval binds proposal, subject version, policy, authority snapshot, and expiry.
- [ ] Quality repair targets the causal layer and authoritative source.
- [ ] Merge and split operate on retained assertions, not only a golden row.
- [ ] Admission, conflict, merge, split, ontology, correction, invalidation, unknown-effect, and override flows are exercised end to end.
- [ ] Partial and irreversible effects have explicit recovery paths.
- [ ] Exceptions expire; appeals preserve previous history.

## Related guidance

- [Identity resolution and candidate graphs](02-identity-resolution-and-candidate-graphs.md)
- [State, context, memory, planning, and tool contracts](06-state-context-memory-planning-and-tool-contracts.md)
- [Security, privacy, tenancy, and authority](07-security-privacy-tenancy-and-authority.md)
- [Permissions, sandboxing, and secrets](../../security/permissions-sandboxing-and-secrets.md)
