# Qualified Adapters and Worked Fraud/AML Investigation Flows

> **Purpose:** Turn a vendor integration into an expiring, operation-level capability and exercise realistic alert,
> investigation, sanctions, filing, override, and recovery paths.
> **Research current through:** 2026-08-31

This is engineering guidance, not legal, regulatory, sanctions, filing, privacy, or customer-action advice. A named
product illustrates documented semantics; it is not an endorsement, supplier assessment, validation conclusion, or
claim that every region, plan, tenant, configuration, or later release behaves the same way.

The model remains a proposal worker. It has no credential or capability to file a SAR/STR, disclose filing status,
contact a customer, clear or confirm a sanctions match, hold/reject/block/freeze a transaction or property, restrict
or close an account, change a risk rating, alter a monitoring rule, or approve its own output.

## Operation-level capability manifest

“Kafka connector,” “case API,” “KYC provider,” or “sanctions feed” is too broad. Admit one operation for one target
configuration and use:

~~~yaml
adapter_capability:
  capability_id: aml-adapter/ledger-window-read/2026-08-31
  intended_use: reproduce a named alert over settled and reversed transactions
  owner: financial-crime-data-platform
  environment_and_tenant: production/regulated-bank-entity-01
  provider_product_release: core-ledger/config-2026.08
  endpoint_region_and_api: eu-west/private-api/v7
  operation: transactions.window.read
  authority:
    workload_identity: investigation-read-broker
    eligible_user_roles: [assigned-investigator]
    purpose: aml-case-investigation
    tenant_legal_entity_scope: [regulated-bank-entity-01]
    case_entitlement_required: true
  object_and_field_scope:
    object: settled-transaction
    keys: [account-id, transaction-id, revision]
    allowed_fields: [amount, currency, event-time, posting-time, status, reversal-ref]
    denied_fields: [authentication-secret, unrelated-customer-profile]
  read_semantics:
    snapshot: repeatable-as-of-token
    pagination_and_limits: {max_records: 5000, max_window_days: 90}
    coverage_proof: control-total-and-source-watermark
  effects: none
  failure_states: [denied, invalid, partial, stale, throttled, unavailable, unknown]
  evidence:
    contract_tests: artifact:ledger-v7-tests
    negative_scope_tests: artifact:ledger-v7-authz
    restore_and_replay_test: artifact:ledger-v7-recovery
  expires_at: 2026-11-30T00:00:00Z
  refresh_triggers: [schema, role, region, plan, limits, source-meaning, incident]
  disable_control: kill-switch/ledger-window-read
~~~

The capability ID is resolved server-side. The model cannot select the tenant, credential, endpoint, API version,
scope, list revision, case state, model alias, notification recipient, filing destination, idempotency key, retry mode,
or effect tier.

## Qualification evidence matrix

| Evidence area | Required proof |
|---|---|
| Authority | Human and workload identity, current role/delegation, case entitlement, tenant/legal entity, purpose, jurisdiction, row/field/action enforcement and revocation latency |
| Exact identity | Party/account/transaction/message/case/alert/rule/model/list-entry/evidence/decision/effect namespaces, revisions, merge/split/reversal/supersession rules and collision tests |
| Time and version | Event/posting/effective/publication/observation times, source watermarks, snapshot mechanism, API/schema/configuration/model/list versions and reproducible historical read |
| Coverage | Pagination, ordering, duplicates, gaps, backfill, corrections, tombstones, control totals, truncation and complete-empty versus unavailable |
| Effects | Server-derived semantic key, immutable payload hash, approval, expiry, destination idempotency, receipt meaning, postcondition, `UNKNOWN`, cancellation and correction path |
| Security/privacy | Classification, encryption, egress, secrets, tenancy, residency, SAR/STR confidentiality, retention/hold/deletion, support access and prompt/data-poisoning tests |
| Operations | Rate/batch limits, concurrency, timeouts, maintenance, change notice, status/support path, circuit breaker, recovery load, DR, decommissioning and tested disable switch |
| Supplier/change | Supplier assessment, subprocessors, SDK/parser/dependency provenance, version/plan/region drift, vulnerability handling, capability expiry and requalification owner |

Negative tests are noncompensating. An adapter fails admission if it leaks one other tenant's record, treats a partial
page as complete, loses an official-list revision, silently changes an entity link, lets a model choose a sink, retries
an unknown consequential effect, or cannot reconstruct an investigator's decision-time evidence.

## Provider and system qualification notes

### Transaction, warehouse, search, and streaming systems

The transaction system of record owns rail-native identity and lifecycle: authorization, capture, clearing, settlement,
return, reversal, chargeback, amendment and cancellation may be different records representing one economic event.
Qualify exact schemas and counts before alert logic. A warehouse/search projection can accelerate investigation but is
not automatically current, complete, permission-equivalent or authoritative.

Apache Kafka 4.3.1 was released in June 2026; installed versions and compatibility must be pinned. Kafka's official
design documentation warns that “exactly once” has a scope: transactional read/process/write within Kafka requires the
corresponding producer/consumer configuration, while an external database, case service, filing gateway or payment
control needs its own atomicity/idempotency and reconciliation. Persist source topic/partition/offset, event ID/schema,
producer and ingestion time, correction/tombstone semantics, consumer generation and output transaction. Never infer
that a committed offset proves the case/evidence write or downstream effect occurred.

OpenSearch illustrates two separate concerns. A point-in-time search can hold a stable search view for pagination, but
the index remains a derived projection whose coverage and source watermark must be proven. OpenSearch audit logging is
configurable and disabled by default in current documentation; enabling it does not turn application logs into case
evidence or a complete access-control audit. Pin index/alias, mapping, analyzer, PIT, query, sort/tie-breaker, security
plugin/configuration, source watermark and deleted-document behavior.

### Case management and workflow

Qualify alert/case creation, dedupe/link/supersede, optimistic version, assignment, queue, deadline, evidence attachment,
hypothesis, human decision, approval, override, effect handoff, confidentiality masking and audit export separately.
The case platform must reject stale writes and preserve earlier values, actor, reason and time. A workflow-complete state
does not mean evidence is correct, a filing is required, a destination accepted it, or an account action is authorized.

Do not give the model a generic `update_case`, comment, email, task or browser tool. Expose typed D2 proposals such as
`case.propose_hypothesis`, `case.propose_evidence_gap` and `case.create_review_draft`; the server binds case/version,
validates citations and policy, and attributes the proposal to the behavior release.

### Sanctions and official watchlists

OFAC's Sanctions List Service (SLS) is now the primary OFAC application for list files/data and provides SDN,
consolidated non-SDN, customized datasets and archived delta files. The official source has changed XML namespaces and
schemas before, so list ingestion must validate rather than silently accept the prior parser. Persist authority, exact
list/program, publication/retrieval time, full/delta identity, dataset hash/signature where available, entry ID/revision,
aliases/identifiers, removals/corrections, parser/normalizer and complete population counts. Periodically reconcile a
full official snapshot even when deltas appear healthy.

A vendor aggregator may aid matching but cannot silently replace the official authority required by policy. A potential
match package preserves all candidates, transliteration/normalization, exact and conflicting identifiers, ownership/
control paths, transaction/property state, applicable jurisdiction profile and deadline. The model cannot decide a true
match or whether to reject, block, freeze, license, report or release.

The FATF Recommendations were amended in June 2026, including a revised Recommendation 16 payment-transparency
surface and Recommendation 6 changes. EU AML responsibilities also moved from EBA to AMLA on 2026-01-01 while existing
EBA instruments remain until replaced; AMLA's live regulatory-instrument register must be monitored. These facts are
refresh triggers, not executable global law. Local legal/compliance owners compile the applicable policy.

### KYC, identity, beneficial ownership, PEP, and adverse media

Identity proofing, authentication, KYC/CDD, beneficial ownership, PEP/adverse-media research and transaction identity
are distinct. NIST SP 800-63-4, finalized in 2025, provides useful identity-proofing, authentication and federation
concepts but does not satisfy a financial institution's KYC/CDD duty or prove beneficial ownership. Qualify provider
account/region, subject and verification-session IDs, evidence method/version, assurance/decision vocabulary,
manual-review path, fraud/deepfake controls, webhook authenticity/order, retention, redress and correction.

Preserve provider assertion versus institutionally verified fact. An identity-provider group is not AML training,
case assignment, filing authority or segregation of duties. A customer-risk score is not suspicion evidence; an
adverse-media result is not verified fact; PEP status is not wrongdoing; registry/LEI data is not guaranteed current
beneficial ownership.

### Graph and entity-resolution systems

Graph edges are derived, temporal and source-backed. Neo4j's current operations documentation states that the default
is read-committed and that non-repeatable reads can occur; bookmarks support causal ordering but are opaque tokens, not
evidence IDs. Qualify database/release, transaction isolation, bookmark/snapshot strategy, Cypher templates, parameters,
query bounds, timeouts, node/edge construction version, source watermarks, truncation and correction propagation.

Only deterministic queries run. The model supplies approved typed parameters, never arbitrary Cypher. A graph path or
shared attribute proposes a relationship to investigate; it cannot confirm common control, beneficial ownership,
collusion or guilt.

### Rules, feature stores, and model registries

Pin rule definition, compiler/digest, input schema, threshold, effective interval, typology release and population. Pin
model artifact digest, feature definitions and point-in-time values, training/evaluation lineage, calibration/threshold,
runtime and inference ID. MLflow's current registry supports version lineage, signatures, aliases and tags, but aliases
are mutable references. Resolve an approved alias to an immutable version/artifact at case admission and store that
version; an alias name cannot reproduce a score later.

Rules and scores produce signals, not decisions. Test point-in-time feature correctness, delayed labels, population
shift, adversarial adaptation, missingness, calibration and slice burden. A model deployment or registry approval does
not approve the agent workflow, jurisdiction policy, human review, or downstream action.

### Filing, escalation, restriction, and notification systems

Separate draft package, human decision, approval, submission/restriction intent, dispatch, receipt, postcondition,
correction/release and reconciliation. FinCEN's BSA E-Filing path and SAR confidentiality/supporting-document rules are
specific to covered U.S. workflows. They do not authorize a generic filing API or legal rule for other institutions.
The credentialed adapter is outside the reasoning plane, and the exact current schema, acknowledgement, deadline and
request-verification process are locally qualified.

Notifications must not tip off a subject or disclose that a SAR/STR was considered or filed. The agent never composes
or sends customer communications about investigation status. Twilio's current Message API illustrates provider states
such as queued, sent, delivered and undelivered, evolving callback fields, per-channel behavior and queuing/rate limits.
A provider delivery state does not prove the intended human read/understood the message or that disclosure was lawful.
Bind approved template/purpose, recipient identity, channel/consent, sender, provider message ID, callback verification,
status query, retention and cancellation; separate delivery from the underlying case/effect.

## Qualification pipeline

~~~mermaid
flowchart LR
    I[Inventory exact operation and owner] --> P[Pin tenant config version role scope]
    P --> C[Contract and semantic fixtures]
    C --> N[Negative scope and confidentiality tests]
    N --> F[Failure ambiguity correction poisoning tests]
    F --> R[Read-only replay and population reconciliation]
    R --> S[Proposal-only shadow]
    S --> D[Draft-only sandbox effect]
    D --> G[Game day restore drift and kill switch]
    G --> A[Expiring capability approval]
~~~

Admission is per operation. A read may be approved while a write is rejected; one object/field set may be approved
while another remains inaccessible; a capability may expire when the provider, configuration, region, role, schema,
limits, receipt semantics, source meaning or jurisdiction profile changes.

## Worked flow one — rapid-movement alert triage

1. Admit a semantic alert keyed by tenant, producer, rule/model release, subject scope, window and revision; link known
   duplicates without erasing their producer events.
2. Pin case cutoff, ledger/message snapshots, jurisdiction profile, behavior release and investigator entitlement.
3. Deterministically reproduce the alert with rail-specific settled/reversal/currency semantics and complete coverage.
4. Retrieve bounded KYC/account purpose and counterpart/network views. Show unavailable sources and truncated graph
   results; do not translate missing data into innocence or suspicion.
5. The model proposes cited hypotheses, benign alternatives and discriminating evidence gaps. It stops when identity,
   purpose, coverage, deadline or authorization is ambiguous.
6. An investigator accepts, edits, rejects or requests evidence. Their decision cites the frozen review bundle and
   applicable policy; the alert/disposition remains a workflow outcome, not ground truth.

**Pass gate:** reversal double-counting, partial pagination, late posting, corrected KYC, huge graph and benign payroll
pattern cannot create a false “complete” investigation or uncited accusation.

## Worked flow two — potential sanctions match

1. Bind authority/list/program, official full/delta revision, parser, entry ID/revision, customer/party revision,
   transaction/property state, jurisdiction profile and response deadline.
2. Verify official-list freshness and complete ingestion; reconcile an aggregator result to the official source.
3. Build all identity candidates with names/scripts, dates, places, documents, vessels/entities, aliases, conflicts and
   ownership/control paths. Similarity is a candidate, not a match.
4. The model organizes evidence and unresolved questions within the blinded/authorized case projection. An eligible
   sanctions reviewer makes the match decision; a separate legal/operational workflow decides the effect.
5. Persist the decision-time snapshot. A list correction/removal or entity-link correction opens the reviewed
   rescreen/release/correction path without rewriting the earlier record.

**Pass gate:** missed delta, changed XML namespace, removed entry, transliteration collision, common name, ownership
cycle and expired reviewer role all stop or escalate. No model path blocks, freezes, rejects or releases.

## Worked flow three — investigation and proportionate evidence request

1. Freeze party/account/beneficial-owner identities, effective relationships, case cutoff, CDD version and lawful
   purpose. Separate customer assertion, provider assertion, verified fact, derived feature and investigator hypothesis.
2. Evaluate existing expected-activity, source-of-funds/wealth, PEP/adverse-media and transaction evidence with
   freshness, coverage, contradictions and jurisdiction-specific relevance.
3. The model may propose a narrow missing-evidence request with purpose, fields, period, alternatives, burden and reason.
4. Policy checks necessity/proportionality and confidentiality; an eligible human chooses whether and how the approved
   customer or third-party process communicates. The agent has no contact tool.
5. New evidence is quarantined, verified, versioned and linked. It may resolve, contradict or leave the hypothesis
   unresolved; inability to obtain it is not automatically evidence of misconduct.

**Pass gate:** hidden sensitive-field request, wrong legal entity, overly broad period, injected document, adverse-media
allegation and PEP family-name guess are rejected or visibly qualified.

## Worked flow four — filing decision, dispatch, and unknown outcome

1. The model prepares a cited narrative and structured draft from a frozen evidence bundle and pinned current schema.
2. Deterministic validation checks required fields, internal consistency, unsupported claims, confidentiality and
   deadline. A designated human makes the filing decision and attests through the approved workflow.
3. The effect service derives one semantic filing intent, payload hash and destination reference after current policy,
   separation-of-duties, approval-expiry and case-version checks.
4. If the response is lost after dispatch, state becomes `UNKNOWN`; keep the legal clock/escalation visible and never
   create a new filing intent or tell the model to retry.
5. The independent reconciler queries acknowledgement/status by stored IDs and payload. It confirms, proves not
   submitted before an authorized bounded retry, or routes unresolved ambiguity to controlled manual resolution.
6. Correction/amendment is a new linked decision/effect. It does not delete or silently replace the original filing.

**Pass gate:** crash at every boundary, duplicate queue delivery, expired approval, changed case evidence, schema drift,
destination lag and conflicting receipt produce one explainable ledger history and no duplicate filing.

## Worked flow five — human override, correction, and handoff

1. Show the reviewer the model proposal, cited evidence, gaps, benign alternatives, versions and downstream burden;
   never hide uncertainty behind a score or fluent narrative.
2. Record accept/edit/reject/abstain as separate structured actions with actor/role, reason codes, free-text rationale,
   case version and review-bundle hash. A reviewer cannot approve their own restricted action where SoD forbids it.
3. An override changes the governed case decision, not source evidence. Correct the source/entity/rule/model artifact in
   its owning system and propagate invalidation through claims, eval fixtures, decisions and pending effects.
4. Handoff to fraud operations, sanctions, legal, cyber/SOC, payment operations or another legal entity uses minimum
   typed references and explicit purpose/sharing authority; it never copies unrestricted case memory.
5. Feedback enters training/retrieval only after independent adjudication, minimization, selection-bias review,
   retention/expiry and contamination controls.

**Pass gate:** automation-bias controls detect selective acceptance, override authority is current, source truth is not
rewritten, confidential filing status does not enter customer/support paths, and correction reaches every dependent.

## Cancellation, ambiguity, and recovery matrix

| Operation | Before provider commit | After dispatch or ambiguous response | Recovery evidence |
|---|---|---|---|
| Case draft/task | Cancel/supersede the local generation | Refetch exact case/version; preserve comments and assignment history | Case audit plus semantic artifact key |
| KYC evidence request | Cancel unsent approved request | Do not assume customer/vendor non-receipt; reconcile workflow and communication | Request/workflow/provider IDs and approved purpose |
| Sanctions review action | Stop before separately authorized effect | Freeze dependent work; sanctions/legal owner reconciles exact property/transaction state | Official list snapshot, decision and destination state |
| Regulatory filing | Cancel only a known unsubmitted intent | Treat as potentially submitted; query acknowledgement/destination | Payload hash, client/receipt IDs, status and case filing ledger |
| Hold/block/freeze/reject | Cancel only if authoritative system proves no action | Do not compensate automatically; restrict new action and escalate | Exact object/amount/status, audit and release/correction authority |
| Notification | Cancel queued message if provider supports it | Observe provider state; never repeat underlying investigation/effect | Provider message ID/status plus authorized communication record |

For every `UNKNOWN`, freeze the semantic intent, preserve clocks and emergency paths, query with local and provider IDs,
compare authoritative state and audit, obtain required human disposition, and re-evaluate policy/approval before resume.

## Exercises and measurable gates

| Exercise | Faults to inject | Passing evidence |
|---|---|---|
| Stream-to-case completeness | Duplicate/reordered events, offset rollback, tombstone, schema change, late backfill and external-store timeout | Control totals/watermarks converge; no offset or Kafka guarantee is mistaken for case/effect success |
| Official-list freshness | Missed delta, full-list mismatch, namespace/parser change, removal and publication during active review | Bounded freshness alert, affected-operation disable/rescreen, complete snapshot and no stale confirmed match |
| Case and graph scope | Cross-tenant ID, stale case version, nonrepeatable read, huge component and correction | Zero disclosure, explicit truncation, version conflict and dependent-link invalidation |
| Filing ambiguity | Lost response after accept, duplicate dispatch event, expired approval and region failover | One semantic intent; `UNKNOWN` reconciles before retry; deadline/manual escalation remains active |
| Tipping-off and privacy | Customer asks about case, message template includes filing state, support trace captures narrative | No disclosure/send, restricted evidence plane, incident trail and independent authorized response path |
| Human factors and fairness | Fluent wrong draft, counterfactual names/scripts/languages, reviewer fatigue and conflicting reviewers | Severe errors caught; calibrated disagreement/adjudication; burden/error slices meet approved gates |
| Restore and recovery load | Case/effect/list/index restore with source/provider backlog and staff shortage | Critical clocks/authority/effects restore first; safe throttled drain meets recovery objective without starvation |

## Final qualification checklist

- [ ] Each operation has intended use, exact configuration/version, authority/scope, owner, expiry and disable control.
- [ ] Exact party/account/transaction/case/alert/rule/model/list/evidence/decision/effect identities survive correction and replay.
- [ ] Complete-empty, partial, stale, denied, throttled, unavailable and unknown are distinguishable.
- [ ] Official-list full/delta ingestion and emergency rescreen are reconciled and refresh-monitored.
- [ ] Rules/models/graphs remain derived signals with point-in-time inputs, immutable versions and challenge paths.
- [ ] Case proposals are typed D2 artifacts; human decision, approval, external effect and receipt are separate.
- [ ] Anti-tipping-off, confidentiality, tenant/purpose, secrets, poisoning and supplier-change tests pass.
- [ ] `UNKNOWN`, cancellation, correction, override, handoff, recovery load and DR are exercised.
- [ ] Provider status meanings are documented and never inflated into legal, customer, filing or investigation outcomes.

## Current primary sources

- [FATF Recommendations, amended June 2026](https://www.fatf-gafi.org/en/publications/Fatfrecommendations/Fatf-recommendations.html)
- [FFIEC BSA/AML Manual](https://bsaaml.ffiec.gov/manual)
- [FinCEN SAR supporting-documentation guidance](https://www.fincen.gov/resources/statutes-regulations/guidance/suspicious-activity-report-supporting-documentation)
- [FinCEN BSA E-Filing information](https://bsaefiling.fincen.gov/filing-information)
- [OFAC Sanctions List Service](https://ofac.treasury.gov/sanctions-list-service)
- [OFAC SLS schema/namespace change notice](https://ofac.treasury.gov/recent-actions/20240507_44)
- [AMLA regulatory-instrument register](https://www.amla.europa.eu/policy/regulatory-instruments_en)
- [EBA–AMLA 2026 handover](https://www.eba.europa.eu/publications-and-media/press-releases/eba-and-amla-complete-handover-amlcft-mandates)
- [NIST SP 800-63-4 Digital Identity Guidelines](https://www.nist.gov/publications/nist-sp-800-63-4-digital-identity-guidelines)
- [Apache Kafka 4.3.1 release announcement](https://kafka.apache.org/blog/2026/06/25/apache-kafka-4.3.1-release-announcement/)
- [Apache Kafka message-delivery semantics](https://kafka.apache.org/42/design/design/)
- [OpenSearch point-in-time search](https://docs.opensearch.org/latest/search-plugins/searching-data/point-in-time/)
- [OpenSearch audit logs](https://docs.opensearch.org/latest/security/audit-logs/index/)
- [Neo4j transaction behavior](https://neo4j.com/docs/operations-manual/current/database-internals/)
- [Neo4j causal-consistency bookmarks](https://neo4j.com/docs/query-api/current/bookmarks/)
- [MLflow Model Registry workflows](https://www.mlflow.org/docs/latest/ml/model-registry/workflow/)
- [Twilio Message resource](https://www.twilio.com/docs/messaging/api/message-resource)

Return to the [category index](README.md), [integration contracts](02-reference-architecture-and-integration-contracts.md),
[effect safety](07-filings-effects-idempotency-and-recovery.md), or [research packet](../../research/packets/fraud-aml-investigation-agent-blueprint.md).
