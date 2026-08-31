# Context, Memory, Compaction, and Continuity

Status: production design guide  
Last reviewed: 2026-08-31

A travel journey can last months, outlive model windows and releases, cross disruptions, and contain highly sensitive data. Continuity must come from typed durable records, not an ever-growing transcript or an embedding that “remembers” a booking. Context is a temporary, least-privilege view compiled for one decision.

## Memory is not one store

| Memory lifetime | Use; reject | Authority and writes | Retention and deletion | Poisoning and evaluation controls |
|---|---|---|---|---|
| Turn/scratch memory | Use for current utterance parsing, candidate questions and temporary calculation; reject booking truth, approval, credentials and durable preference | No authority; process-local, allowlisted input only; no durable model write | Destroy at end of model call/turn and verify caches/buffers follow the short-TTL policy | Inject secrets, cross-tenant text, provider instructions and false IDs; require zero durable propagation and zero prohibited-field retention |
| Working/run memory | Use for bounded stage progress, refs, budgets and unresolved conflicts; reject sole copies of effects, deadlines or supplier facts | Coordinator checkpoint authority for progress only; typed reducer writes, reconstructed from ledgers | One bounded run plus recovery window; delete after durable checkpoint/terminal state, subject to incident hold | Crash/replay, stale checkpoint, truncation and forged tool-result tests; compare reconstructed hashes/state with durable projection |
| Session memory | Use for explicit navigation, locale and presentation choices in one authenticated interaction; reject cross-session approval, inferred profile and booking truth | Principal/tenant/session bound; only explicit user/UI writes | Session expiry or stated shorter TTL; logout/revocation clears server/client/cache copies | Session fixation, principal switch, stale login, poisoned preference and cross-device replay; assert no effect/authority survives a new session |
| Durable workflow/task memory | Use for journey/case coordination, source/effect/approval/audit refs, clocks and receipts; reject raw secrets and model-authored truth | Owning systems plus deterministic workflow/effect ledgers; typed trusted services only | Per transaction/dispute/audit/incident schedules with store-specific deletion, restriction, tombstone and legal-hold handling | Event tamper, duplicate/out-of-order, rollback, cross-tenant and missing-obligation properties; deterministic replay/invariant/restore tests gate release |
| Domain knowledge memory | Use for curated provider manuals, policies, advisories, standards and qualified releases; reject transaction state, unreviewed web/provider instructions and timeless legal claims | Versioned corpus with publisher, jurisdiction, effective/superseded dates and owner approval | Source-specific refresh/withdrawal schedule; remove superseded chunks/indexes while retaining governed citation lineage | Corpus poisoning, source impersonation, stale/superseded and conflicting-document suites; measure source selection, applicability, abstention and refresh SLA |
| Long-term/preference memory | Use for explicit stable, scoped traveler choices and loyalty references; reject behavioral/sensitive inference, authority and emergency precedent | Traveler-approved profile store; consented typed write with inspect/edit/revoke controls | Purpose expiry/reconfirmation; deletion/revocation propagates to indexes/caches/exports except governed holds | Provider/model preference injection, co-traveler leakage, stale consent and context-shift tests; measure correct applicability, neutral fallback and deletion proof |
| Episodic/outcome memory | Use only for minimized reviewed incidents, corrections and verified outcomes in offline evaluation/training; reject automatic precedent and raw production replay | Dataset owner approval, lineage, deidentification and opt-in/legal basis; no online write path | Dataset schedule with contributor deletion/restriction, re-derivation and artifact/model lineage controls | Quarantine, label-consensus, backdoor/canary, bias/coverage and memorization tests; promotion requires held-out regression and privacy review |

Source snapshots, effect/audit ledgers, authoritative domain records, and continuity receipts are storage classes inside durable workflow/task memory, not extra memory lifetimes. Each class still retains its own access, correction, deletion/hold, and replay policy.

Do not create a generic long-term vector memory containing trips, PNR text, chats, document details, or provider error messages. It is difficult to scope, correct, delete, audit, or protect from poisoning.

## Memory policy by data class

```yaml
memory_policy:
  policy_version: travel-memory-2026-08-31
  classes:
    commercial_snapshot:
      allowed_layers: [source_snapshot, continuity_reference]
      model_visibility: normalized_minimum
      ttl: transaction_and_dispute_policy
    traveler_preference:
      allowed_layers: [session_preference, durable_profile]
      durable_write_requires: explicit_confirmation_and_consent
      prohibited_inference: [wealth, health, disability, religion, relationship, risk_tolerance]
    identity_document:
      allowed_layers: [protected_profile_assertion, qualified_adapter]
      prohibited_layers: [turn_scratch, model_prompt, continuity_payload, analytics]
    payment_credential:
      allowed_layers: [pci_scoped_payment_service]
      prohibited_everywhere_else: true
    accessibility_need:
      allowed_layers: [protected_profile, service_request]
      model_visibility: minimum_functional_requirement
      analytics: prohibited_without_specific_governance
    provider_text:
      allowed_layers: [source_snapshot, bounded_context]
      trust: untrusted
      automatic_memory_write: forbidden
```

Enforce policies in storage, context compilation, telemetry, exports, and deletion—not only in a prompt.

## Context assembly pipeline

```mermaid
flowchart LR
    A[Decision request] --> B[Resolve tenant, traveler, journey, stage]
    B --> C[Load pinned release and policy]
    C --> D[Fetch authoritative refs and fresh snapshots]
    D --> E[Verify scope, version, freshness, conflicts]
    E --> F[Apply data minimization and redaction]
    F --> G[Allocate token budgets by evidence class]
    G --> H[Compile typed context + trust labels]
    H --> I[Model call]
    I --> J[Schema, citation, policy, and feasibility validation]
    J --> K[Store output as analysis, never source truth]
```

The compiler has an allowlist per task. A search explanation does not need names, contacts, locators, document data, or payment reference. A cancellation explanation may need masked booking/product refs and the current provider quote, but not unrelated trip history.

### Priority order under pressure

Never truncate indiscriminately. Allocate in this order:

1. tenant, traveler, acting-principal/delegation, journey, stage, and authority ceiling;
2. current exact itinerary revision and hard constraints;
3. active quote/after-sales snapshot, source, freshness, expiry, hashes, and material unknowns;
4. open effect/approval/reconciliation state and deadlines;
5. supplier order/fulfillment/payment status references and conflicts;
6. current policy decisions and forbidden actions;
7. required service/accessibility status and document-information boundary;
8. feasible candidate facts and score explanations;
9. explicit soft preferences;
10. older conversation or background knowledge.

If categories 1–7 do not fit, do not call a model for an effect-adjacent decision. Use a deterministic view or human escalation.

## Token and context budgets

Set hard per-task budgets; measure actual tokenizer behavior for the released model.

| Task | Input target | Reserved output | Evidence emphasis | On overflow |
|---|---:|---:|---|---|
| Intent clarification | 4k tokens | 1k | Current utterance, typed intent gaps, safe defaults | Ask one bounded question or show structured form |
| Option explanation | 12k | 2k | Up to 3–5 feasible options, exact differences, source/freshness | Deterministically reduce dominated options; never drop material term |
| Raw rule explanation | 10k | 2k | Exact scoped clauses plus structured conditions | Chunk by clause; label incomplete; human review |
| Approval presentation draft | 8k | 1.5k | Exact prepared proposal and consequences | Use deterministic template; model optional |
| Disruption proposal | 16k | 2.5k | Current trip, live impact, candidates, deadlines, duties | Prioritize in-travel/critical facts; reduce background |
| Operator escalation summary | 12k | 2k | Facts, unknowns, effects, deadlines, refs, next actions | Emit typed case packet even if prose omitted |

These are starting targets, not model-independent limits. Also cap tool-result bytes, snapshots per class, candidates, turns, and cumulative model cost per journey/stage. Context length availability is not permission to include more personal data.

## Loss-aware reduction

Reduction occurs by evidence type:

- **Offers:** remove hard-infeasible/dominated options deterministically; retain exact selected option and representative alternatives.
- **Rules:** select clauses by traveler/product/action/applicability; keep source offsets and an `incomplete` marker.
- **Events:** project into current state plus unresolved conflicts and deadlines; preserve event refs.
- **Conversation:** extract only confirmed intent fields and unresolved questions; never convert a model inference into a preference.
- **Provider responses:** pass normalized material fields and restricted raw excerpts; retain raw artifact outside context.
- **History:** summarize old completed stages into signed typed receipts, not free prose.

Each reducer emits:

```yaml
reduction_receipt:
  reducer: offer-frontier-v3.1.0
  input_refs: [ofs_1, ofs_2, ofs_3, ofs_4, ofs_5]
  output_refs: [ofs_1, ofs_3, ofs_5]
  dropped:
    - ref: ofs_2
      reason: hard_infeasible_connection
    - ref: ofs_4
      reason: strictly_dominated_by_ofs_3
  protected_fields_checked:
    - source
    - observed_at
    - expires_at
    - total_and_currency
    - conditions_knowledge
    - service_request_status
  checksum: sha256:...
```

Never use embedding similarity as the sole selector for approval/effect evidence.

## Typed continuity receipt

Compaction or handoff produces a schema-validated, signed receipt:

```yaml
continuity_receipt:
  receipt_version: 1
  schema_version: travel-continuity/v1.0
  receipt_id: ctr_901
  generated_at: 2026-08-31T04:22:00Z
  generated_by: context-compiler-v5.4.0
  release_manifest: travel-release-2026.08.31.2
  version_pins:
    behavior: travel-behavior-2026.08.31.2
    policy: travel-policy-2026.08.31.4
    adapter_capabilities: travel-adapters-2026.08.31.1
    model: approved-model-2026.08
    context_compiler: context-compiler-v5.4.0
  scope:
    tenant_id: tenant_acme
    legal_entity_id: in01
    acting_principal_id: usr_17
    traveler_ids: [trv_01JZ..., trv_01KA...]
    delegation_ref: delegation_71
    journey_id: jny_740
    point_of_sale: IN
  goal:
    objective: book_approved_air_component
    stage: AwaitingApproval
    authority_ceiling: exact_T3
  itinerary:
    revision: 7
    topology_hash: sha256:...
    component_refs: [component_air_1, component_hotel_1]
    hard_constraint_set_ref: constraints://jny_740/rev_7/hard
  active_commercial_evidence:
    snapshot_id: ofs_air_911
    provider: air_provider_a
    observed_at: 2026-08-31T04:20:12Z
    expires_at: 2026-08-31T04:35:12Z
    price_hash: sha256:...
    terms_hash: sha256:...
    topology_hash: sha256:...
    material_unknowns: []
  approval:
    proposal_id: prop_66
    grant_ref: null
    approval_expires_at: 2026-08-31T04:33:12Z
  effects:
    - effect_id: eff_81
      semantic_operation_id: tenant_acme:air_order_create:jny_740:rev_7:prop_66
      state: awaiting_approval
      intent_hash: sha256:...
      latest_attempt_ref: null
      reconciliation_obligation_ref: null
  pending_effect_ids: [eff_81]
  unknown_effect_ids: []
  supplier_state_refs: []
  payment_refs: [paymethod_tok_8]
  service_requests:
    - request_id: sr_204
      state: supported_for_request_not_confirmed
  open_obligations:
    - kind: approval_before_quote_margin
      due_at: 2026-08-31T04:33:12Z
      owner: journey_workflow
  active_clocks:
    - {clock_id: approval_quote_margin, due_at: 2026-08-31T04:33:12Z, owner: journey_workflow}
  unresolved_conflicts: []
  denied_actions: [change, refund, autonomous_disruption_rebook]
  next_safe_actions:
    - obtain_exact_approval
    - expire_proposal_if_deadline_passes
  next_safe_action: obtain_exact_approval
  source_lineage:
    source_event_high_watermark: "1844"
    snapshot_refs: [ofs_air_911]
    policy_refs: [policy://decision/921]
  omitted_refs:
    - ref: session://sess_82/conversation
      class: conversation_history
      reason: confirmed_intent_already_projected
    - ref: vault://provider/air/response_1001
      class: raw_provider_payload
      reason: retained_by_reference_in_restricted_vault
  integrity:
    invariant_hash: sha256:...
    content_hash: sha256:...
    signature: sigstore-or-kms-signature-ref
```

### Required receipt semantics

- IDs are stable references, never regenerated during compaction.
- `omitted_refs` lists every dropped artifact/class by stable reference and reason; an explicit empty list is required when nothing was omitted.
- Current commercial evidence includes source, observation, expiry, and separate hashes.
- Effect records include unknown/partial states, latest attempt, and reconciliation obligation.
- Open deadlines and owners survive.
- Denied actions and authority ceiling survive.
- The receipt carries source/event cursor and release manifest so replay can detect staleness.
- Integrity covers the canonical serialized form; access is authenticated and audited.
- Protected values remain references, not copied secrets.

## Compaction algorithm

1. Pause new model calls for the workflow stage; do not pause independent supplier reconciliation.
2. Load current deterministic projection and open obligation/effect ledgers.
3. Resolve and verify tenant/traveler/journey scope.
4. Select the schema version supported by both current and resume runtime.
5. Populate protected fields from authoritative projections—not model text.
6. Attach current evidence references and explicitly classify conflicts/unknowns.
7. Reduce optional history by typed reducers and collect omission receipts.
8. Validate schema, cross-field invariants, source existence, freshness, and scope.
9. Canonicalize, hash, sign, store, and append `continuity.receipt_created`.
10. Resume by reloading authoritative current state; compare event cursor/versions and invalidate stale proposal/approval as needed.

### Effect-adjacent compaction gate

Block preparation, approval, and commit if the active receipt lacks or conflicts on any required field:

- tenant/traveler/acting principal/delegation;
- journey and itinerary revision/topology hash;
- exact provider capability and resource IDs;
- quote/after-sales source, expiry, price/terms/topology hashes;
- material unknowns and service-request state;
- policy, approval, intent hash, and authority ceiling;
- current effect state, attempt, and reconciliation obligations;
- expected source versions and write fencing key;
- open deadlines and denied actions.

No model may reconstruct a missing field from prose.

## Resume and handoff protocol

```text
verify receipt signature/schema/scope
→ load current event/effect projections from receipt cursor
→ apply later events deterministically
→ re-read live supplier/commercial state when freshness requires
→ compare pinned and current release compatibility
→ invalidate stale approvals/proposals
→ restore timers/obligations
→ continue only from allowed state
```

If the receipt conflicts with current supplier truth, retain both and prefer the fresh qualified supplier observation for its domain. If it conflicts with an effect ledger, block writes and investigate tampering or projection error.

Human handoff uses the same receipt plus an accessible case view. Do not paste raw PNR, card, document, or accessibility text into a general ticketing system.

### Fail-closed resume verifier

Resume produces a verification receipt before any model call or external write:

```text
verify canonical signature, receipt_version and supported schema
AND verify tenant/legal entity/principal/delegation/travelers and access purpose
AND load event/effect ledgers at source_event_high_watermark, then apply every later event
AND verify all version pins or an explicitly tested compatibility migration
AND resolve every snapshot/approval/effect/clock/obligation/omitted ref and its scope
AND recompute invariant_hash from current canonical projection
AND re-read supplier/payment/commercial state required by freshness/finality policy
AND prove pending/unknown effects have fencing and reconciliation owners
AND validate next_safe_action against current state, authority, policy and kill controls
ELSE block writes, preserve evidence and create a typed recovery/security case
```

| Verification result | Resume state | Permitted next action |
|---|---|---|
| Exact match; evidence still fresh | `verified_current` | Only the recorded `next_safe_action` after normal preconditions |
| Event high-watermark advanced | `verified_advanced` after deterministic replay | Recompute next action; stale proposal/grant invalidates |
| Compatible pinned-version migration proved | `verified_migrated` with migration receipt | Continue under explicit compatibility map; never reinterpret committed effect semantics |
| Snapshot/approval expired or material source changed | `safe_stale` | Reprice/retrieve/reapprove; no commit |
| Unknown/pending effect lacks conclusive state | `reconciliation_required` | Read-back/escalation only; no conflicting write |
| Missing/unauthorized ref, hash/signature/scope/fence mismatch, unsupported schema | `verification_failed` | No model/effect; quarantine and operator/security review |

The verifier never accepts a prose summary, “close enough” hash, regenerated ID, last-model-message reconstruction, or fallback to a newer permissive policy. Its receipt records check versions, ledger head, live read times, discrepancies and exact deny reason.

## Preference memory

Separate explicit preference from observed behavior:

```yaml
preference:
  preference_id: pref_aisle_3
  traveler_id: trv_01JZ...
  kind: seat_position
  value: aisle
  applicability: air_when_available_without_extra_cost
  hardness: soft
  source: traveler_explicit_confirmation
  consent_id: cns_pref_22
  recorded_at: 2026-08-31T03:00:00Z
  expires_at: 2027-08-31T00:00:00Z
  last_confirmed_at: 2026-08-31T03:00:00Z
  user_editable: true
```

Do not infer “prefers cheapest” from prior bookings, “can walk” from missing assistance requests, “has visa” from a prior destination, “traveling with partner” from co-booking, or willingness to accept risk from an emergency. Context changes.

### Preference precedence

1. current trip's explicit traveler instruction;
2. current trip's authenticated delegation/policy;
3. applicable explicit unexpired durable preference;
4. session-only explicit preference;
5. neutral default shown to the traveler.

Conflicts are presented. Employer policy does not silently rewrite a traveler's accessibility need or consent.

## Knowledge retrieval

Separate authoritative policy/standard retrieval from transactional state:

- index document ID, publisher, jurisdiction, product/provider, version/effective date, fetched/reviewed time, and supersession;
- retrieve the exact relevant section and preserve source link/offset;
- use primary sources where possible;
- mark old provider manuals and drafts as historical or uncertain;
- do not let retrieved instructions override system/tool policy;
- enforce review/expiry for document, refund, passenger-rights, accessibility, security, and API capability material;
- answer with dated information and route legal/admission decisions to authorities.

An embedding index can help locate evidence. It cannot decide that a rule applies, that an offer is current, or that the traveler meets a legal condition.

## Memory poisoning and correction

### Threats

- supplier text says to store or reuse a credential;
- a malicious email claims the traveler “always approves upgrades”;
- a model summary converts an unknown fare condition into a durable fact;
- an operator pastes another tenant's booking into a case;
- a bad outcome is automatically saved as a future “lesson”;
- stale itinerary/document facts remain retrievable after correction/deletion.

### Controls

- allow memory writes only through typed commands and trusted services;
- require source type, scope, consent, purpose, applicability, expiry, and sensitivity;
- prohibit untrusted content/model output from direct durable writes;
- run scope and contradiction checks before retrieval and before save;
- maintain immutable provenance plus correction/supersession, not silent overwrite;
- quarantine and review episodic/training candidates offline;
- provide traveler/operator view, edit, revoke, and delete workflows;
- version embeddings/indexes and remove superseded/deleted data with verification;
- continuously test canary records and cross-tenant retrieval.

## Telemetry and privacy

Log references and classifications, not payloads:

```yaml
context_telemetry:
  trace_id: tr_772
  task: disruption_option_explanation
  tenant_hash: hmac:...
  journey_hash: hmac:...
  compiler_version: context-compiler-v5.4.0
  input_tokens: 11322
  output_tokens: 1288
  evidence_counts: {supplier: 3, policy: 2, traveler: 1}
  oldest_material_evidence_age_seconds: 18
  redaction_counts: {direct_contact: 2, document: 1, payment: 1}
  reduction_receipt_refs: [red_77, red_78]
  continuity_receipt_ref: ctr_901
  sensitive_payload_logged: false
```

Model/provider observability does not justify storing prompts. Use allowlisted structured fields and protected artifact access when incident investigation genuinely requires content.

## Failure matrix

| Failure | Safe behavior |
|---|---|
| Receipt signature invalid | Quarantine receipt, load durable ledger, block effects, security escalation |
| Receipt schema newer than runtime | Use compatible upgrader or manual handoff; no lossy coercion |
| Event cursor advanced | Apply later events, revalidate quote/approval and obligations |
| Snapshot expired after resume | Mark proposal expired and reprice; never rely on compacted value |
| Effect state missing/ambiguous | Load effect ledger; block conflicting writes |
| Protected source unavailable | Do not substitute conversation copy; escalate or retrieve anew |
| Context over budget | Deterministically reduce optional classes; if protected facts do not fit, skip model |
| Preference source/consent absent | Treat as no preference and ask neutrally |
| Deletion request intersects legal hold | Apply owning policy, restrict processing, explain precise status; no ad hoc decision |
| Model output cites absent source | Reject output and fall back to deterministic evidence view |

## Anti-patterns

- “Remember everything” as a product feature.
- Treating the system prompt, transcript summary, or vector database as the current itinerary.
- Reusing prior names, document values, loyalty IDs, or accessibility details without source/version/consent.
- Summarizing away quote expiry, terms, unknown effects, or approval hashes.
- Letting the same model generate and validate its compaction summary.
- Saving provider errors, emails, or model recommendations as trusted future instructions.
- Fitting more context by removing provenance or redaction.
- Copying raw prompts into telemetry “for debugging.”
- Assuming deletion from the primary store removes embeddings, caches, exports, and evaluation datasets.

## Exercises and exit criteria

1. Compact at every transition from quote through commit unknown. Exit: protected fields, unknown effect, and reconciliation deadline survive byte-for-byte by reference/hash.
2. Resume after the provider changed the booking out of band. Exit: later event/read-back supersedes the receipt, approval invalidates, and no restoring write occurs.
3. Inject a fake preference in provider text and in model output. Exit: neither reaches profile/session memory.
4. Request erasure of an old itinerary with one finance legal-hold record. Exit: each store applies its policy, access is restricted, and the outcome is auditable without corrupting settlement evidence.
5. Overflow the model context with rules and 200 offers. Exit: deterministic filtering/reducers preserve material conditions and receipt omissions; no silent truncation.
6. Try to retrieve another tenant's similar journey through vector similarity. Exit: structural tenant scope yields zero cross-tenant result and triggers detection on canary attempts.

Continuity is production-ready when any fresh worker can resume the exact safe state from signed typed records, every omission is visible, stale evidence invalidates action, and no prohibited personal/payment/document data crosses into model or general memory.

Continue with [disruptions, duty of care, documents, and escalation](08-disruptions-duty-of-care-documents-and-escalation.md). See the shared [memory architecture](../../context-memory/memory-architecture.md), [context engineering](../../context-memory/context-engineering.md), and [compaction/continuity guide](../../context-memory/compaction-and-continuity.md).
