# Conflicts, Approvals, Security, and Privacy

> **Purpose:** Protect competitive information and enforce event-specific independence, exact approvals, least privilege, and end-to-end data lifecycle controls outside model reasoning.

## Threat model

High-value assets include sealed bids, bidder identities, pricing, source-selection information, evaluator notes/scores, supplier personal and ownership data, budget/strategy, sanctions and allegations, approvals, award intent, credentials, and audit evidence.

Threat actors include a malicious bidder, compromised supplier document/site, conflicted insider, overprivileged evaluator, compromised connector/plugin, cross-tenant operator, model/provider failure, and an attacker exploiting a stale worker or approval. Procurement-specific abuse paths include:

- a bid attachment instructs the model to reveal competitors or change criteria;
- a favored supplier is inserted after longlist or event approval;
- an evaluator accesses bids before opening or outside assigned criteria;
- conflicts are declared under one account while another service account retains access;
- a deadline, weight, FX rule, or award payload changes after approval;
- fuzzy sanctions or adverse-media matches are weaponized to exclude a competitor;
- a connector times out after publishing or awarding and a retry duplicates/inconsistently changes the event;
- raw bids leak through traces, caches, embeddings, eval datasets, or support exports.

See the reusable [agent threat model](../../security/agent-threat-model.md); the controls below specialize it for sourcing.

## Event-scoped identity and permissions

Keep these identities distinct and joinable in evidence:

- initiating user and business owner;
- procurement role and legal delegation;
- evaluator role and assigned criterion/lot;
- conflict reviewer and award authority;
- tenant, legal entity, business unit, and sourcing event;
- workload/service identity and deployed behavior release;
- run, attempt, step, approval, credential, bid snapshot, and effect.

The authorization tuple is:

```text
(principal, tenant/legal_entity, event, resource, operation,
 role/purpose, data_class, current_state/version, constraints, time)
```

Resource names from the model are canonicalized to immutable platform IDs before policy. Deny by default. Recheck revocation and event state on every sensitive read and commit.

## Tool authority matrix

| Canonical operation | Tier | Reversible? | Required identity and approval | Evidence |
| --- | ---: | --- | --- | --- |
| Read approved spend/category snapshot | P0 | N/A | Case-scoped read role | Query/evidence ID, source revision |
| Read one assigned bid after opening | P0-high | N/A; disclosure is irreversible | Evaluator assignment, no conflict, open state | Access event, bid/version, purpose |
| Draft longlist/RFx/comparison | P1/P2 | Yes | Case role; no external visibility | Artifact hash and model/release lineage |
| Publish event or amendment | P3 | Not fully | Procurement authority; exact event digest | Platform receipt and public/private notice state |
| Invite/remove supplier | P3 | Partly | Approved supplier set and event version | Per-supplier delivery/access receipt |
| Open/reveal sealed bids | P3-special | No | Native opening roles and conditions; agent excluded by default | Platform opening audit and participant identities |
| Send clarification/broadcast | P3 | No after delivery | Exact audience/payload and fairness profile | Delivery and content digest |
| Record official score | Human-only | Correctable with history | Assigned evaluator, conflict clearance | Score/rationale version and moderation trail |
| Submit award | P3-critical | Not safely reversible | Independent award authority and fresh full packet | Award receipt, postcondition, decision rationale |
| Create onboarding/legal handoff | P3 | Partly | Approved award and minimized payload | Handoff receipt and acknowledgement |
| Change policy, roles, audit, seal, credentials | P5 | High impact | Separate administrative path | Agent has no operation or credential |

## Conflict-of-interest contract

EU Directive 2014/24/EU Article 24 requires covered contracting authorities to prevent, identify, and remedy conflicts. FAR parts 3 and 9.5 address procurement integrity and personal/organizational conflicts in U.S. federal acquisition. These regimes are not universally applicable; they illustrate why a prompt-level “declare conflicts” reminder is inadequate.

```json
{
  "conflict_case_id": "coi_evt_7812_v7",
  "event_id": "evt_7812",
  "participant_id": "person_991",
  "roles": ["technical_evaluator"],
  "declared_at": "2026-08-28T12:00:00Z",
  "declaration_form_release": "coi_form_5",
  "relationships_checked_through": "2026-08-28",
  "declared_items": [],
  "system_detected_candidates": ["prior_employment_supplier_779"],
  "disposition": "recused_from_supplier_779",
  "disposition_owner": "ethics_officer_21",
  "access_policy_version": 18,
  "expires_at": "2026-10-31T23:59:59Z"
}
```

Candidate detection may use authoritative HR, ownership, gift/hospitality, prior-work, or relationship data that policy permits. A model may summarize evidence, never adjudicate or infer private relationships. The disposition immediately changes access and assignment; it is not merely a note.

Recheck on supplier-set change, participant change, role change, ownership update, new disclosure, and before award. Preserve correction and appeal routes.

## Segregation of duties

Model SoD as a graph of identities and incompatible capabilities, including delegated accounts and service principals.

| Combination | Default rule |
| --- | --- |
| Requester + sole requisition/route approver | Forbidden above policy-defined conditions |
| RFx/criteria author + sole final award authority | Require independent approval or review |
| Supplier inviter + sealed-bid opener | Separate where regime/policy requires |
| Evaluator + undisposed supplier relationship | Recuse or remediate before access/score |
| Proposal/model worker + approval | Always separate; model cannot approve |
| Effect gateway + policy/role administrator | Separate administrative control plane |
| Reconciliation operator + ability to rewrite original receipt | Forbidden; corrections append |

Evaluate both static role conflicts and dynamic event conflicts. Two nominal accounts controlled by the same person are one actor for SoD. Emergency overrides are time-limited, reasoned, independently reviewed, and visible in the audit package.

## Exact approval contract

```yaml
approval:
  approval_id: appr_01K...
  action: submit_award
  event_id: evt_7812
  event_version: 9
  award_packet_digest: "sha256:..."
  suppliers_and_lots:
    - {supplier_id: supplier_779, lot_id: lot_software}
  evaluated_value: {amount: "418750.00", currency: USD}
  criteria_release: criteria_5
  normalization_release: norm_11
  bid_snapshot_ids: [bid_204_rev3, bid_221_rev2, bid_230_rev1]
  due_diligence_snapshot_ids: [dd_31, dd_32, dd_33]
  conflict_clearance_id: coi_evt_7812_v7
  policy_release: sourcing_policy_2026_08_15
  approver_id: person_104
  approver_authority_release: delegation_33
  issued_at: 2026-08-31T11:00:00Z
  expires_at: 2026-08-31T15:00:00Z
  use_count: 1
```

Commit-time policy revalidates all fields plus budget, event state, current revocations, and remote version. Any material change creates a new packet and approval. A chat “looks good,” email reply, or task emoji is not approval.

## Sealed-bid and competitor isolation

Enforce isolation in platform, broker, storage, retrieval, cache, and model session layers:

- no raw bid appears before the source platform's authorized opening event;
- each bidder's originals and derived artifacts have separate access labels;
- evaluator access is limited by event, lot/criterion, role, purpose, and time;
- individual evaluation contexts contain one bid unless the approved method calls for comparison;
- no cross-event or cross-tenant embedding collection, shared retrieval cache, prompt cache, or eval dataset contains unredacted bids;
- exports, support tools, shadow traffic, backups, and incident bundles inherit the same classification;
- publication/debrief views are generated from an explicit disclosure policy, not the internal award packet.

A supplier-facing assistant, if one exists, is a separate product and identity plane. It never shares a context, tool credential, cache key, or trace content with the buyer-side agent.

## Untrusted content and execution isolation

Supplier PDFs, spreadsheets, archives, websites, email, and connector fields can contain malicious instructions, macros, formulas, links, tracking content, oversized objects, and exploit payloads.

Use:

1. content-type and size validation, malware scanning, archive limits, and quarantine;
2. immutable original plus safe rendered/parsed derivatives;
3. sandboxed document processing with no credentials, restricted filesystem/process, and deny-by-default egress;
4. explicit untrusted labels through tool results and context;
5. trusted sink gates for every read expansion, external fetch, message, upload, or effect;
6. canary tests for prompt injection, data exfiltration, formula injection, and cross-bid retrieval.

Detection can reduce bad proposals; it cannot authorize effects. Follow [Prompt injection and untrusted data](../../security/prompt-injection-and-untrusted-data.md).

## Data and privacy lifecycle

| Data class | Examples | Default handling |
| --- | --- | --- |
| Source-selection confidential | Raw bids, prices, scores, evaluator notes | Event/role isolated; content telemetry off; retain per policy/regime |
| Supplier confidential | Product, financial, security, pricing, trade-secret material | Purpose-limited; no reuse for unrelated model training |
| Personal data | Contacts, beneficial owners, declarations, references | Minimize, restrict, correct/delete where applicable, avoid prompt duplication |
| Highly sensitive financial | Bank/tax/payment instructions | Exclude from model; onboarding control system handles it |
| Public notice/award data | Approved publishable fields | Separate view and release workflow; do not infer publication permission |
| Audit/control evidence | IDs, versions, decisions, hashes, approvals, receipts | Unsampled, integrity-protected, restricted, retention-aligned |
| Diagnostic telemetry | Latency, counts, errors, redacted summaries | Sampled; content off by default; never authoritative |

Map prompts, provider requests, caches, checkpoints, memory, artifacts, indexes, traces, shadow runs, eval sets, support exports, and backups. Verify provider retention, training use, residency, subprocessors, encryption, deletion, and incident terms for the exact service and contract. Applicable privacy law and contractual duties require legal review; this blueprint does not invent universal retention periods.

## Procurement control declaration

```yaml
control_profile:
  task_boundary:
    authoritative_event_system: e_sourcing_platform
    out_of_scope: [contract_language, purchase_orders, shipments, payments]
  authority:
    autonomous_ceiling: P1
    p3_effects: [publish_event, invite_supplier, send_clarification, submit_award, create_handoff]
    p5_effects: [change_policy, change_roles, reveal_bids, edit_contract, create_order]
  execution:
    document_isolation: hardened_sandbox_no_credentials_no_egress
    model_egress: through_read_broker_only
    credentials: short_lived_audience_and_event_scoped
  effects:
    approval_invalidation: [event_version, supplier_set, criteria, value, conflict, policy, expiry]
    outcome_unknown: reconcile_from_platform_before_retry
  data:
    raw_bid_memory: prohibited
    content_telemetry: off
  kill_switches: [stop_admission, disable_reads, disable_effect_class, revoke_connector, quarantine_event]
```

Production values must name actual services, owners, and tests. “Uses OAuth,” “has HITL,” or “runs in a container” is not a control declaration.

## Security and privacy failure matrix

| Failure | Immediate containment | Recovery owner/evidence |
| --- | --- | --- |
| Cross-bid or cross-tenant disclosure | Revoke sessions/connector, freeze affected events, preserve access evidence | Security + procurement/legal decide scope and event remedy |
| Unauthorized early opening | Disable bid reads, preserve native platform log | Procurement/legal and platform owner |
| Conflicted evaluator retains access | Revoke assignment and all linked credentials | Ethics owner; re-evaluation evidence |
| Approval replay or payload substitution | Deny by digest/use/state checks | Approval service incident and effect reconciliation |
| Secret appears in model/trace | Revoke/rotate, stop export, quarantine records | Security/privacy; deletion and exposure evidence |
| Supplier correction not propagated | Block affected disposition/award | Data owner updates cases, indexes, memory, eval sets |
| Audit store unavailable | Halt P3 effects; read/propose may degrade | Platform owner restores unsampled evidence path |
| Connector/plugin scope expands | Disable integration | Re-admit version, scopes, schemas, data flows, tests |

## Readiness checklist

- [ ] Event, bidder, bid, evaluator, tenant, role, purpose, and time are enforceable authorization dimensions.
- [ ] Conflict declarations, detected candidates, dispositions, recusal, expiry, and rechecks change access in real time.
- [ ] SoD evaluates humans and all accounts/service principals they control.
- [ ] P3 approvals bind exact versions, data, amount, criteria, conflicts, policy, expiry, and one use.
- [ ] Sealed-bid isolation covers platform, broker, storage, context, cache, telemetry, backup, shadow, and eval paths.
- [ ] Supplier content is sandboxed and cannot reach dangerous sinks without trusted policy.
- [ ] Credentials are short-lived, scoped, audience-bound, attached outside model context, and independently revocable.
- [ ] Data purpose, minimization, provider use, residency, retention, correction, deletion, and incident paths are approved.
- [ ] Audit evidence is unsampled and distinct from redacted diagnostic telemetry.
- [ ] Independent stop, revoke, quarantine, and evidence-preservation drills pass.

## Sources and next step

- [EU Directive 2014/24/EU](https://eur-lex.europa.eu/eli/dir/2014/24/oj/eng)
- [FAR Part 3 — Improper Business Practices and Personal Conflicts](https://www.acquisition.gov/far/part-3)
- [FAR Subpart 9.5 — Organizational and Consultant Conflicts](https://www.acquisition.gov/far/subpart-9.5)
- [GAO 2025 Green Book](https://www.gao.gov/greenbook)
- [WTO GPA Article XVII confidentiality provisions](https://www.wto.org/english/docs_e/legal_e/rev-gpr-94_01_e.htm)
- [NIST SP 800-161 Rev. 1 update 1](https://csrc.nist.gov/pubs/sp/800/161/r1/upd1/final)

Continue with [State, context, memory, planning, and reliable effects](07-state-context-memory-planning-and-reliable-effects.md). Return to the [guide map](README.md#guide-map).
