# Context, Memory, Compaction, and Durable Orchestration

Model context is a working representation, not the plant record. Operational truth belongs in authoritative domain systems and durable workflow state. The coordinator may reconstruct context from typed facts and evidence references; it must never treat a remembered summary as proof of asset state, safety, calibration, disposition, approval, or effect outcome.

## Use an exact memory taxonomy

| Memory lifetime | Admit and use | Reject as authority | Retention and deletion | Poisoning controls | Evaluation controls |
|---|---|---|---|---|---|
| Turn/scratch memory | temporary reasoning, schema filling, and response formatting for one call | asset/product state, approval, safety, release, effect result, credential | discard after the call; retain only separately governed trace metadata when necessary | isolate instructions from retrieved data; block secrets; strict tool schemas | seeded injection and secret-canary tests; verify nothing material is required after discard |
| Working/run memory | bounded plan, excerpts, candidate hypotheses, unresolved questions for one run | any fact not linked to a fresh source/version; any permission | expire at run end plus a minimal forensic window; clear on site/role/privilege change | trust labels, byte/token limits, source allowlists, contradiction display | truncation, site-switch, poisoned-note, stale-evidence, and missing-negative-evidence cases |
| Session memory | user interaction continuity, selected view, unresolved conversational question | operational truth, qualification, signature, standing consent, or prior effect outcome | short policy TTL; user correction/deletion; no extension merely because a session is active | bind to user/site/role; prevent cross-session retrieval; reread operational facts | cross-user/site leakage, correction propagation, expired-session, and misleading-prior-turn tests |
| Durable workflow/task memory | canonical orchestration state, event cursor, locks, clocks, approvals, effect ledger, handoffs, next permitted action | physical state, quality disposition, or external-system state without authoritative read-back | workflow, audit, legal-hold, and record schedules; controlled purge that preserves required effect/audit evidence | typed transitions, expected versions, append-only events, transactional outbox/inbox, reconciliation | crash/replay, duplicate event, lost lease, stale approval, backup restore, migration, and invariant-rebuild suites |
| Domain knowledge memory | approved procedures, specifications, mappings, policies, manuals within exact applicability | withdrawn, future, draft, wrong-site, wrong-product, or similarity-only content | document-control schedule; revoke from new retrieval while preserving historical reconstruction | signed releases, effective dating, owner approval, provenance, chunk/index version, withdrawal propagation | historical/current applicability, supersession, multilingual parity, poisoned amendment, and recall tests |
| Long-term/preference memory | consented display, language, accessibility, and notification preferences | identity, credentials, site authority, skill, qualification, approval, or plant fact inferred from behavior | minimum TTL; user-visible access/correction/deletion; tenant-isolated tombstone propagation | allowlisted fields, explicit consent, no free-form plant content, encryption and access audit | deletion completeness, consent withdrawal, cross-tenant isolation, and preference-to-authority escalation tests |
| Episodic/outcome memory | curated, reconciled prior cases for retrieval, evaluation, failure mining, or offline learning | current precedent, automatic policy, unresolved outcome, or unreviewed live transcript | dataset purpose/retention/version; de-identify, honor legal hold, withdraw poisoned/mislabeled cases | human curation, outcome verification, temporal cutoff, provenance, bias/privacy review, no online auto-ingest | leakage-safe time/site splits, label audit, rare-failure coverage, poisoning, representativeness, and removal regression |

Do not create a generic long-term vector memory containing conversations, plant records, and tool results. It mixes authority, retention, access, and poisoning boundaries.

## Keep canonical workflow state outside the model

```yaml
workflow_id: WF-MC-2026-008812
schema: manufacturing-workflow/4.0
site_id: plant-a
case_ref: MC-2026-008812
state: WAITING_FOR_APPROVAL
authority_ceiling: M2
behavior_release: mfg-agent/2026.08.4
resource_locks:
  - key: plant-a:asset:P-204:maintenance-case
    fencing_token: 72
source_versions:
  asset_mapping: 17
  eam_case: 9
  procedure: WI-VIB-014@9
evidence_snapshot: sha256:...
plan_digest: sha256:...
approval: null
pending_effects: []
unknown_effects: []
next_action: request_exact_work_order_draft_approval
stop_conditions: [identity_changed, evidence_expired, active_safety_hold]
version: 31
```

Every transition validates the expected version and policy. A model response proposes a transition; the workflow engine commits it.

## Model long waits as durable clocks and messages

Human approval, parts arrival, maintenance windows, laboratory results, containment confirmation, CAPA effectiveness, and recurrence checks can wait minutes to months. Never keep a worker, model call, session token, or in-memory lock alive for the wait.

Persist each clock with `clock_id`, clock type, created/due/expiry times, authoritative time source, tolerance, owner, escalation route, cancellation state, and workflow version. Race external events against explicit clocks; deduplicate events by stable event ID because runtimes and queues can redeliver after restart. A late event is recorded but cannot revive an expired approval or intent.

| Wait | Wake events | Expiry behavior |
|---|---|---|
| Exact approval | signed approval, rejection, role revocation, digest/precondition change, deadline | expire and require a new approval; never assume consent |
| Parts/window/qualification | authoritative resource version change, planner update, deadline | recompute feasibility; do not dispatch from cached availability |
| External effect reconciliation | read-back match, authoritative rejection/absence, consistency-window timer | remain fenced; bounded query then accountable escalation |
| Quality/lab result | validated result/correction, sample invalidation, method/calibration change, deadline | stop dependent decision and preserve missing-result state |
| Effectiveness/recurrence | eligible observation window complete, adverse event, data-quality failure, deadline | classify only from predefined evidence; missing window is `INCONCLUSIVE` |

Use logical workflow time supplied by the durable runtime for replay; use trusted wall time and recorded clock uncertainty for domain eligibility. Version the workflow before deployment so old executions either remain on compatible code or migrate through an explicit, tested transition.

## Assemble context by authority and need

Build each call from:

1. immutable mission, authority ceiling, site, and behavior release;
2. current workflow state and permitted next operations;
3. freshly read authoritative facts with versions and timestamps;
4. evidence eligible for the current decision and its quality/lineage;
5. the minimum approved knowledge excerpts with version/effective scope;
6. unresolved conflicts, unknown effects, resource locks, and stop conditions;
7. bounded interaction context required to respond to the user.

Prefer structured facts over copied prose. Separate trusted control instructions from untrusted manuals, notes, emails, vendor attachments, and retrieved text. Do not let retrieved content redefine tool policy.

## Treat compaction as a typed, loss-aware transition

Long workflows require compaction, but a narrative summary can erase uncertainty, negative evidence, expiry, or pending effects. Produce a continuity receipt:

```yaml
receipt_schema: manufacturing-continuity/2.1
receipt_version: 1
receipt_id: CR-01J...
created_at: 2026-08-31T06:12:00Z
workflow_id: WF-MC-2026-008812
case_ref: MC-2026-008812
site_id: plant-a
objective: prepare non-released inspection work-order draft
authority_ceiling: M1
behavior_release: mfg-agent/2026.08.4
version_pins:
  behavior: mfg-agent/2026.08.4
  policy: plant-policy/2026.08.3
  adapter_capabilities: mfg-adapters/2026.08.2
  context_compiler: mfg-context/3
  identity_graph: plant-a-identity/117
  procedure: WI-VIB-014@9
source_event_high_watermark:
  workflow_version: 31
  event_offset: 88210
resource_locks:
  - key: plant-a:asset:P-204:maintenance-case
    fencing_token: 72
sealed_artifacts:
  plan_digest: sha256:...
  approval_digest: null
approval_refs: []
active_clocks:
  - {clock_id: evidence_expiry, due_at: 2026-08-31T06:17:00Z, owner: case-controller}
facts:
  - fact: asset P-204 resolved to object 9509e320 at case time
    authority: identity-service
    source_version: 17
    evidence_ref: ev-id-17
    observed_at: 2026-08-31T03:12:00Z
    expires_at: 2026-08-31T06:17:00Z
unresolved_conflicts:
  - sequence gap before evidence ev-771
effect_ledger:
  confirmed: []
  pending: []
  unknown: []
pending_effect_ids: []
unknown_effect_ids: []
next_safe_action: request human review of sequence-gap impact
stop_conditions: [identity_changed, evidence_expired, policy_unavailable]
omitted_refs:
  - {kind: raw_telemetry_window, ref: evidence-set:ev-700-779, reason: size}
  - {kind: prior_chat, ref: protected-session:segment-1-18, reason: nonauthoritative}
loss_report:
  omitted_chat_turns: 18
  omitted_retrieved_chunks: 42
  material_information_omitted: false
checksum: sha256:...
invariant_hash: sha256:...
```

The receiving runtime verifies schema, checksum, workflow cursor, behavior compatibility, site/role, lock fencing, and freshness. It rereads material facts and all effect outcomes; it does not trust the receipt as current authority.

The receipt is loss-aware only if every omitted material source remains addressable by a protected reference or is declared unavailable. Never place credentials in the receipt or represent a safety/quality attestation only as omitted prose; preserve its authoritative record reference and require a fresh read.

## Define what compaction may never erase

- site/tenant, object identities, effective time, and authority ceiling;
- plan and approval digests, approval expiry, and governing versions;
- safety/quality boundaries and stop conditions;
- unresolved identity/evidence conflicts and negative evidence;
- resource locks and fencing tokens;
- every effect intent, attempt, confirmation, rejection, and `UNKNOWN` outcome;
- cancellation/recovery state and external correlation IDs;
- evidence source/version/time/quality needed for pending decisions;
- next accountable human, deadline, and escalation route.

If the compactor cannot represent a material item in the receipt schema, mark `material_information_omitted: true` and stop automated continuation.

## Make durable execution replay-safe

Workflow code must be deterministic under replay or explicitly versioned. Model calls and external operations are activities whose results are recorded; they are not rerun silently during recovery. Use:

- transactional outbox/inbox around intent and message publication;
- semantic operation IDs and adapter reconciliation;
- deterministic timers and deadlines;
- versioned workflow definitions and migration plans;
- heartbeats for long activities without treating a lost heartbeat as proof the external operation stopped;
- bounded retries and explicit nonretryable failure classes;
- encrypted durable state, backup, restore, retention, and legal hold where applicable.

Do not put raw sensitive prompts, secrets, full manuals, or large telemetry windows into workflow history. Store protected references and hashes.

## Govern approved knowledge

Each knowledge item should carry:

```yaml
document_id: WI-VIB-014
revision: 9
owner: plant-a-reliability-engineering
site_scope: [plant-a]
object_scope: [pump-family-204]
effective_from: 2026-06-01T00:00:00Z
effective_to: null
approval_ref: doc-control-9912
supersedes: WI-VIB-014@8
classification: internal-controlled
content_hash: sha256:...
retrieval_chunker: controlled-doc-chunker/2.0
```

Withdrawn documents remain available for historical reconstruction but cannot support new work. Retrieval must filter by site, object, role, event time, current decision time, and approval state before similarity ranking.

## Resist memory and context poisoning

Controls include:

- allowlisted ingestion sources and signed knowledge releases;
- explicit trust labels and source boundaries in the prompt/context protocol;
- extraction of data into schemas without executing instructions found in content;
- human review before a past case becomes episodic memory;
- scanning, content limits, and quarantine for attachments;
- provenance-aware retrieval and contradiction display;
- deletion, correction, supersession, and re-index propagation tests;
- periodic canaries that prove cross-site and withdrawn-content exclusion;
- no autonomous “learn from every outcome” write-back.

An attacker, faulty vendor note, or copied email can state “ignore safety rules” or mimic tool syntax. Such text remains data.

## Design resume validation

On every resume after compaction, failover, pause, or human delay:

1. authenticate the actor and reestablish site/role scope;
2. validate the receipt schema/version/checksum and prove its source-event high-water mark is neither ahead of nor ambiguously behind the durable event store;
3. replay from that cursor, rebuild canonical state, recompute the invariant hash, and fail closed on any mismatch;
4. reacquire or validate locks with a strictly newer fencing token and prove the prior owner cannot write;
5. reconcile every pending/unknown effect and partial multi-system step from authoritative sources;
6. reread target identity, source versions, holds, procedure/specification status, approvals, qualifications, and all material omitted references;
7. rebuild active clocks from durable time, expire late approvals/intents, and reevaluate evidence eligibility and clock uncertainty;
8. verify the complete behavior release and adapter manifests remain allowed for this workflow version;
9. compare the receipt's proposed next safe action with freshly computed permitted operations; continue only on an exact allowed match, otherwise stop with a typed reason and human handoff.

Resume is a new authorization check, not continuation by inertia.

After worker or regional restart, effect dispatch remains disabled until this sequence completes. A restored workflow whose local history predates a vendor-accepted effect is expected; reconciliation repairs knowledge without redispatching the intent.

## Memory verification drills

- withdraw a procedure while a case is paused;
- correct a QMS result after compaction;
- replace an asset component and reuse a tag;
- revoke a user/site permission between approval and resume;
- corrupt or omit one continuity-receipt field;
- restore workflow state from backup while the vendor system retained the effect;
- inject an instruction into a maintenance note and OEM PDF;
- request deletion of a user preference without deleting controlled evidence;
- attempt cross-site retrieval through a shared embedding index;
- change behavior release while an old workflow remains active.

Pass only when authoritative rereads and policy stop unsafe continuation while preserved ledgers still reconstruct prior events.

## Read next

Memory controls are part of the security program. Continue with [Security, governance, controlled procedures, and audit](09-security-governance-controlled-procedures-and-audit.md).
