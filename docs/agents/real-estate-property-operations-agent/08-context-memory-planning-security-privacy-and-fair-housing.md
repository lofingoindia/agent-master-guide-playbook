# Context, Memory, Planning, Security, Privacy, and Fair Housing

## Protection rule

Context is a purpose-limited projection, not a data lake. Memory is a set of typed records with owners, retention, deletion, and poisoning defenses—not “everything the user ever said.” Protected, screening, accommodation, VAWA, safety, identity, payment, and access information stays in restricted systems and enters model context only when an approved purpose cannot be met with a safer reference.

## Context assembly

Assemble context on every bounded model call in this order:

1. immutable authenticated scope: tenant, workflow, property/case references, role;
2. pinned behavior, authority, policy, and stop versions;
3. current workflow state, clocks, effect/approval status;
4. minimum fresh authoritative facts with field provenance;
5. the current user input, clearly marked untrusted;
6. a small set of relevant approved domain-knowledge excerpts with citations;
7. tool contracts permitted for this state;
8. required output schema and abstention rules;
9. typed continuity receipt when prior context was compacted.

Never assemble by “top similar chunks” alone. Apply tenant, purpose, role, effective-date, sensitivity, property/jurisdiction, and document-status filters before semantic ranking.

## Token and context budgets

Set budgets per workflow and measure actual distributions. The following is an illustrative starting envelope, not a universal allocation:

| Context class | Target share | Hard behavior on pressure |
|---|---:|---|
| authority, safety, output/tool contracts | 10–15% | never drop; use compact identifiers |
| current state, clocks, approvals, unknown effects | 10–15% | never drop required fields |
| fresh domain facts/provenance | 20–30% | select fields; preserve source/version |
| current input/attachments | 15–25% | chunk with cited spans; defer irrelevant pages |
| domain knowledge | 10–20% | retrieve fewer authoritative excerpts |
| continuity receipt/history | 5–10% | typed compaction; no free-form transcript dump |
| output/reasoning reserve | 20–30% of model window reserved outside input | stop before exhausted |

Runtime limits also include maximum tool calls, retrieved bytes, attachment pages, model calls, replans, wall time, external effects, and spend. On exhaustion, stop with a continuity receipt and an owner—not a compressed guess.

### Loss hierarchy

Compact/drop in this order:

1. duplicate formatting and acknowledged small talk;
2. superseded drafts already represented by final hashes;
3. verbose tool payloads whose typed results and evidence references are stored;
4. low-relevance knowledge excerpts;
5. old conversation text represented by a verified continuity receipt.

Never drop current scope, prohibitions, safety state, unresolved conflicts, clock deadlines, source versions, approvals, effect status/semantic IDs, restricted-record markers, or next-owner commitments.

## Explicit memory classes

| Canonical lifetime | Use | Reject | Retention, correction, and deletion | Poisoning and evaluation controls |
|---|---|---|---|---|
| Turn/scratch | One call's scoped instructions, selected evidence excerpts, typed tool results and disposable candidates | Credentials, screening reports, door codes, payment data, approvals, durable facts or unrestricted attachments | Destroy after the call except separately governed prompt/output evidence; legal hold applies only to that retained record | Inject hostile lease/message/vendor text, wrong-property facts and secret requests; assert no tool/authority/state change and no retained cross-case payload |
| Working/run | Typed hypotheses, source refs, plan progress, budgets, validation and open questions for one bounded run | Free-form diary, final lease/property state, access authority, decision, consent or state required after a crash | Short TTL; checkpoint material refs/hashes; correct derived state without rewriting sources; delete on completion/cancel | Reorder tool results, seed stale unit/lease/vendor versions and no-progress loops; score support, contradiction preservation, bounded steps and clean rebuild |
| Session | Authenticated UI focus, recent interaction and unsaved draft for one interaction segment | Authentication/assignment, housing decision, legal notice, entry permission, approval, effect receipt or evidence that a message was delivered | Expire on logout/idle/handoff; explicitly promote governed drafts; support lawful user deletion without erasing business records | Resume as another user/tenant/property and alter cached UI selections; verify reauthentication, resource binding, no hidden personalization and no implicit approval |
| Durable workflow/task | Case state, owners, timers, event offsets, source versions, facts/decisions/effects, approvals, cancellation, correction and recovery | Transcript or model prose as source of truth, mutable shadow lease, secrets or raw restricted data unnecessary for the task | Business-record schedule with bitemporal history, correction lineage, legal hold, deletion proof and backup/vendor propagation | Duplicate/reorder/corrupt events, omit clocks and inject stale approvals/unknown effects; replay, restore, invariant and reconciliation tests fail closed |
| Domain knowledge | Reviewed procedures, housing/safety rules, authority matrices, templates, calendars, property policies and tool contracts | Model-written rules, withdrawn guidance as current authority, uncited web text, mixed jurisdictions or embeddings as truth | Effective-dated registry with owner, approval, source digest and archived supersession; deletion only through governance | Poison corpus, swap jurisdiction/program/lease edition and stale calendar; test exact-version retrieval, precedence, citation, quarantine and abstention |
| Long-term/preference | Normally avoid; where lawful, read explicit channel, language or accessible-format preference from its authoritative consent/customer system | Protected-trait inference, neighborhood/household profile, screening/selection history, door/access pattern, payment propensity or reviewer shortcut | Purpose/consent/TTL, correction, revocation/export and propagated deletion; tenant/subject bound | Poison source, revoke consent, swap person/tenant and request sensitive inference; test freshness, correction, deletion and zero eligibility/service/price/access influence |
| Episodic/outcome | Curated, minimized/de-identified reviewed incidents/cases for offline evaluation or governed nonbinding operational lessons | Raw household replay, automatic precedent, cross-case prospect/resident lookup or online self-learning | Dataset manifest, purpose/license, lineage, exclusion/hold, TTL, deletion and retraining/index propagation; freeze holdouts | Plant memorized identifiers, biased historic outcomes and mislabeled incidents; test leakage, subgroup/temporal slices, provenance and nonbinding behavior |

Every memory record contains `memory_class`, `purpose`, `source`, `tenant_id`, `subject_refs`, `sensitivity`, `created_at`, `expires_at`, `deletion_state`, `allowed_consumers`, and `integrity/provenance`.

## Typed, loss-aware compaction

Compaction produces a **continuity receipt**, not a prose summary:

```yaml
continuity_receipt:
  receipt_version: 1
  schema_version: continuity/1.3
  receipt_id: cr_881
  created_at: 2026-08-31T06:50:00Z
  source_event_high_watermark: 100
  reason: context_budget_checkpoint
  scope:
    tenant_id: tnt_17
    portfolio_id: pf_4
    case_id: case_72
    workflow: maintenance_intake
  versions:
    workflow_state: 14
    behavior_bundle: reops/2026-08-31.2
    authority_grant: auth-2026-08-31.3
    policies: [gas-gate/6, maint-priority/22, maint-sla/8]
    source_objects: [pms/unit_5c@etag:a19, pms/occupancy_12@etag:b6]
    tools: [pms-read/4.2, cmms-write/7.1, cmms-status/3.0]
  objective: "Create and acknowledge one verified non-emergency work order"
  current_state: creation_unknown
  authoritative_facts:
    - field: unit_id
      value_ref: unit_5c
      source: pms/unit_5c
      source_version: etag:a19
      observed_at: 2026-08-31T06:44:00Z
  completed:
    - step: safety_gate
      outcome: clear_non_emergency
      evidence_ref: evt_100
  inflight_effects:
    - semantic_operation_id: wo:case_72:create:v1
      state: unknown
      intent_hash: sha256:3c...
      provider_correlation_ref: req_91
  pending_effect_ids: [wo:case_72:create:v1]
  unknown_effect_ids: [wo:case_72:create:v1]
  approvals:
    - approval_id: apr_904
      intent_hash: sha256:3c...
      expires_at: 2026-08-31T07:00:00Z
  clocks:
    - clock_id: sla_case_72_response
      rule_version: maint-sla/8
      target_at: 2026-08-31T07:05:00Z
      timezone: America/New_York
      owner: property_ops_queue
  unresolved:
    - id: u_9
      type: effect_outcome
      owner: reconciliation_worker
      next_action: reconcile_by_semantic_operation_id
  prohibitions:
    - blind_retry
    - grant_physical_access
    - include_resident_sensitive_details_in_vendor_payload
  next_permitted_actions:
    - reconcile_work_order
    - request_human_reconciliation
  next_safe_action: reconcile_work_order
  context_compiler_release: reops-context/5
  compactor_release: reops-compactor/3
  invariants_hash: sha256:...
  omitted_state_refs:
    - ref: pms/case_72/communication_archive
      reason: not_required_for_resume
      retrieval_policy: authorized_on_demand
  discarded:
    - class: superseded_drafts
      count: 2
      digest: sha256:d1...
  source_transcript_digest: sha256:7f...
  receipt_digest: sha256:canonical_receipt_without_this_field
```

### Compaction validation

Before accepting a receipt:

- schema validates and all references are tenant-scoped;
- current workflow version matches;
- every nonterminal effect appears exactly once;
- approvals bind an included intent hash;
- clocks have owner and instant/timezone;
- unresolved conflicts and stops are preserved;
- facts include source/version/freshness;
- discarded classes are permitted and digested;
- omitted state has stable tenant-scoped references and an authorized retrieval path;
- a replay test can choose the same next permissible action;
- a redaction scan finds no prohibited data.

If validation fails, do not continue autonomously. Checkpoint the full encrypted state reference and route to an operator.

Restart verification is automated: authenticate and reauthorize the new worker; verify receipt/schema/digest compatibility and monotonic event watermark; reread current property/unit/occupancy/lease/work-order, approvals, clocks and source versions; retrieve any omitted state required by the next transition; recompute the invariant hash; invalidate stale drafts/plans/approvals; and reconcile every pending or unknown effect before retry. Crash after each persistence/dispatch boundary in the fault suite and prove no timer, approval, restriction or effect vanishes and that any changed authoritative state produces a named blocking diff rather than a silent resume.

## Bounded planning

### Macro versus micro

- The **macro-plan** is the deterministic lifecycle and allowed transitions.
- A **micro-plan** is a typed proposal of at most a few actions within the current transition.
- An **effect plan** is prepared by deterministic services and separately approved.

The model cannot create new tools, modify policy, delegate authority, remove a stop, or continue after a terminal condition.

### Planning contract

```json
{
  "plan_version": "micro-plan/2",
  "state": "clarification_needed",
  "goal": "obtain missing location without delaying the SLA",
  "facts_used": [
    {"field": "symptom", "evidence_ref": "msg_998#char=20-58"}
  ],
  "actions": [
    {"type": "ask_approved_question", "template_id": "maintenance-location/5"},
    {"type": "wait_for_event", "event_type": "resident.reply", "until": "2026-08-31T07:00:00Z"}
  ],
  "stop_if": ["safety_trigger", "identity_conflict", "deadline_exceeded"],
  "replan_on": ["new_resident_event", "template_delivery_failure"],
  "max_replans_remaining": 1
}
```

The runtime validates every action. Stop after the goal, required human handoff, effect unknown, budget limit, repeated no-progress, policy conflict, or new safety/security trigger.

## Fair-housing and anti-discrimination controls

### Runtime prohibitions

The current federal Fair Housing Act overview lists race, color, national origin, religion, sex, familial status, and disability; state/local/program rules may add more. Maintain a current deployment-specific protected-class registry, but do not pass those fields into ordinary ranking or service-priority context.

Block:

- protected-class inference from names, language, images, geography, household statements, disability/safety records, or external enrichment;
- ranking, eligibility, availability, showing, service, renewal, price, promotion, or enforcement differences based on protected traits or proxies;
- “neighborhood fit,” demographic steering, or personalized discouragement;
- model-created screening or occupancy criteria;
- training feedback that learns from historical discriminatory outcomes;
- general retrieval over accommodation, VAWA, harassment, screening, or legal records.

### Permitted controlled uses

Legally sourced protected labels may be used only in a segregated evaluation/compliance environment under documented purpose, counsel/compliance approval, strict access, retention, and aggregate reporting. The production agent does not see them.

Counterfactual tests create controlled pairs outside production and hold relevant operational facts constant. A changed protected/proxy cue should not change inventory, tone, options, priority, effort, escalation, or outcome unless a reviewed rule requires a specific supportive path, such as fulfilling an approved accessibility request. Those exceptions must be explicit, beneficial/purpose-limited, and separately evaluated.

No fairness metric or counterfactual test alone proves legal compliance. Causal counterfactual fairness depends on the validity of a causal model and unmeasured-confounding assumptions; use it as one test family alongside process review, slices, redress, audits, and legal analysis.

## Sensitive-request isolation

Accommodation, assistance-animal, VAWA, harassment, retaliation, safety, and medical-adjacent requests:

1. trigger a restricted case type;
2. record only minimum operational facts in the ordinary case;
3. place supporting evidence in a separate encrypted store/queue;
4. restrict staff roles and model access;
5. avoid detailed medical records when less information suffices under reviewed policy;
6. preserve confidentiality and controlled disclosure;
7. provide accessible human review and redress;
8. propagate retention/deletion without erasing required audit evidence.

VAWA controls apply to covered programs and situations; do not assume universal applicability, and check state/local protections.

## Security and privacy controls

### Least privilege

- workload identity per component and tenant/connector;
- short-lived credentials and secrets manager;
- read and write credentials separated;
- no accounting, pricing, screening-decision, BAS-control, or access-control scope;
- ABAC/RBAC using authenticated context, never model output;
- row/index/cache/queue/backup isolation;
- just-in-time administrative access with immutable audit;
- regular access recertification and emergency revocation.

### PII minimization

Tokenize person/contact IDs before model use. Keep raw documents, SSNs/government identifiers, financial account/card data, detailed consumer reports, biometrics, door credentials, and medical/supporting records out of model and general logs. Use provider-hosted payment collection; a token does not automatically remove PCI obligations for systems that can affect security.

### Prompt-injection and knowledge poisoning

Treat instructions in messages, leases, PDFs, images, websites, vendor notes, telemetry labels, and retrieved documents as untrusted text. Controls:

- channel/content delimiters and provenance;
- allowlisted tools enforced outside the model;
- retrieval only from signed published manifests;
- malware/content scanning and quarantine;
- no tool invocation from document text;
- schema and policy validation of output;
- secret and cross-tenant egress filters;
- suspicious “ignore policy,” credential, or data-export requests trigger security evidence;
- source review, two-person publication, versioning, and rapid rollback for domain knowledge;
- poisoning canary tests in every release.

## Approval, audit, and supply-chain controls

Consequential operations preserve distinct proposer, decision owner, approver, effect identity and reconciler. Selection/adverse action, lease/legal interpretation, deposit disposition, refund/disbursement, safety dispatch exception, vendor conflict, inspection/compliance conclusion and physical-access grant require the deployment's named qualified human/domain owner. A service account retains the initiating human/workflow lineage; a model identity can never satisfy a human approval field.

| Control | Production evidence |
|---|---|
| Segregation of duties | Role/effect/value/property matrix; proposer/approver/committer/reconciler and owner/vendor/payee conflict tests; current assignment and expiry checked at commit |
| Exact approval | Tenant/property/unit/person target, source/lease/work versions, rule, recipient/vendor/payee, payload/amount/window, intent hash, expiry and use count |
| Audit record | Actor and workload lineage, purpose, source/evidence refs, behavior/tool/adapter/policy/template versions, decision/reason, approval, attempts, receipt/read-back, correction and final owner |
| Audit versus telemetry | Required business evidence is application-owned, integrity-protected, unsampled and held per record policy; traces/logs are redacted diagnostics and cannot prove legal notice, entry or money movement |
| Break glass | Named incident/safety policy, strong authentication, bounded resource/time, immediate evidence and retrospective independent review; never model-triggered |

The behavior supply chain includes source/build artifacts, runtime dependencies, models, prompts, schemas, connector mappings, housing/safety policies, lease templates/abstract rules, communication translations, geospatial/building ontologies, vendor/qualification feeds and evaluation data.

| Object | Before-release proof | Runtime containment |
|---|---|---|
| Code/image/dependency | Immutable reviewed revision, dependency lock/SBOM, signed digest, vulnerability/license status, build provenance | Allowlisted digest, minimal runtime, no package install, rollback artifact retained |
| Model/provider | Exact route/version or alias semantics, region/project/data use/subprocessors/retention, evaluated fallback and change notice | Egress allowlist, scoped project, route kill switch; fallback receives no automatic authority |
| Prompt/schema/tool/adapter | Immutable digest, owner, compatibility and contract/adversarial/fault tests | Bundle pin, reject unknown versions, credentials outside model worker |
| Rule/template/knowledge | Authoritative source, legal/operations owner, effective interval, locale/jurisdiction/program/property scope, digest and supersession | Signed registry retrieval, precedence check, no model write-back, rapid withdrawal |
| External data/ontology | Owner/license, collection/update time, geography/coverage, version, lineage and permitted caching/reuse | Source label, TTL, purpose/tenant filter; no score/geometry/telemetry-to-authority conversion |
| Evaluation corpus | License/purpose, minimization/de-identification, slice distribution, split/contamination manifest and expiry | Isolated access, immutable gates, locked holdout and deletion/retraining propagation |

[SLSA 1.2](https://slsa.dev/spec/v1.2/) provides a current approved source/build provenance vocabulary, but provenance alone does not prove that a housing rule, lease interpretation, model, vendor record or outcome is correct. Fault tests replace a signed adapter, mutate a prompt under the same bundle, serve withdrawn guidance as current, poison an index, alter an ontology/unit mapping and roll a model alias. The route must block or narrow, identify the cohort, preserve safety/intake/clocks and leave evidence for rollback and resident/applicant remediation.

## Retention and deletion

Maintain a data inventory mapping purpose, authority, sensitivity, source, processor, region, retention, legal hold, deletion, export, and backup behavior. Deletion is a workflow:

1. authenticate and authorize request/policy trigger;
2. resolve subject/tenant scope without overmatching;
3. freeze conflicting reuse;
4. delete or de-identify eligible records across stores, indexes, caches, model files, vendors, and backups per schedule;
5. retain only legally/operationally required evidence under restricted access;
6. record completion and exceptions without retaining deleted content;
7. test restoration does not resurrect deleted retrievable records.

## Decision gate

The context/memory design fails if:

- “memory” is one untyped vector store;
- protected/screening/access/payment data can enter ordinary prompts;
- tenant filters are model-controlled;
- a compaction summary can omit effects, clocks, approvals, or conflicts;
- long-term preferences are inferred rather than explicit;
- episodic memory reuses raw household cases;
- domain documents can publish without provenance and review;
- fairness evaluation runs only on aggregate accuracy;
- deletion does not propagate to retrieval indexes and vendors;
- a model can override a stop or expand a plan.
