# Reference Architecture and Integration Contracts

> **Purpose:** Put a bounded investigator inside a deterministic, durable case workflow and make every upstream and downstream integration explicit.  
> **Section index:** [Fraud and AML investigation agent](README.md)

## Selected architecture

The default is one case workflow with one bounded reasoning loop. Deterministic services admit alerts, normalize identifiers, retrieve evidence, compute graph features, enforce policy, persist state, and execute approved effects. The model may choose among typed read operations and propose typed artifacts; it never owns identity, authority, case truth, credentials, timers, or effects.

This is deliberately smaller than a multi-agent system. Parallel source retrieval and graph work use deterministic workers. Independent challenge is a separate human or evaluation function, not a conversational agent sharing mutable context.

~~~mermaid
flowchart TB
    A["Alerts / referrals / monitoring events"] --> ADMIT["Admission + deduplication + purpose"]
    ADMIT --> WF["Durable case workflow"]
    WF --> CC["Context compiler"]
    CC --> LOOP["Bounded investigator loop"]
    LOOP --> RB["Typed read broker"]
    RB --> SRC["Authorized source adapters"]
    SRC --> EV["Immutable evidence objects"]
    EV --> CC
    LOOP --> PROP["Typed hypothesis / gap / recommendation proposals"]
    PROP --> WF
    WF --> POL["Policy + jurisdiction + segregation-of-duties gate"]
    POL --> REV["Qualified human review"]
    REV --> DEC["Decision ledger"]
    DEC --> EFF["Effect outbox"]
    EFF --> DEST["Filing / restriction / escalation adapters"]
    DEST --> REC["Receipts + reconciliation"]
    REC --> WF
~~~

The workflow can initially be an ordinary transactional service. Add a durable workflow engine only when cases regularly wait on people or evidence, span deployments, require long timers, or cannot be safely recovered with the existing case platform.

## Plane separation

| Plane | Owns | Must not own |
|---|---|---|
| Source | Records supplied by systems of record | Investigation conclusions |
| Evidence | Immutable source object, acquisition metadata, transform lineage, source snapshot | Source correction or human disposition |
| Derived analytics | Entity candidates, temporal windows, graph edges, features, typology signals | A claim that an identity or offense is proven |
| Case workflow | Alert/case version, assignments, deadlines, hypotheses, decisions, approvals, effect status | Raw model transcript as authoritative state |
| Reasoning | Retrieval choices and typed proposals inside a budget | Credentials, policy decisions, durable truth, or commits |
| Authority | Current policy, eligibility, purpose, jurisdiction, separation of duties, exact target | Open-ended natural-language discretion |
| Effect | Semantic operation, idempotency, attempt/receipt/reconciliation state | Reinterpreting the approved decision |
| Telemetry | Performance, trace correlation, operational health | The evidentiary or audit record |

Keep [execution boundaries](../../runtime/execution-boundaries.md) and [run controls](../../runtime/run-controls.md) enforceable outside the prompt.

## Minimum deployment components

| Component | Minimum responsibility | Scale only when measured |
|---|---|---|
| Admission service | Authenticate producer; validate tenant, purpose, schema, identifiers, and event time; deduplicate | Partition by tenant or alert family when contention appears |
| Case store | Optimistic case version, workflow state, assignments, deadlines, decisions, approvals | Split hot evidence blobs from transactional metadata |
| Evidence store | Content-addressed source objects, encryption, lineage, retention/legal hold | Object storage and tiering for volume; keep searchable metadata small |
| Adapter layer | Narrow typed reads/writes, source-version semantics, rate limits, circuit breakers | Dedicated connector workers for slow or high-volume systems |
| Context compiler | Purpose-aware projection, token budget, redaction, evidence handles, freshness | Precomputed views/caches keyed by source and policy version |
| Reasoning worker | Short, cancellable, retry-aware loop with no destination credentials | Separate pools by data residency, model, or risk tier |
| Policy decision point | Current identity, jurisdiction, policy, action, resource, approval, source freshness | High-availability replica; never embedded only in the model prompt |
| Outbox and reconciler | Semantic effect IDs, receipts, ambiguous-state recovery | Sharded workers and destination-specific queues |
| Audit ledger | Append-only control events with access logging and retention | Separate security domain from general observability |

## Contract envelope

Every adapter request and result uses a common envelope plus a source-specific payload. A prose tool description is not a contract.

~~~json
{
  "contract_version": "evidence.read.v1",
  "request_id": "req_01...",
  "run_id": "run_01...",
  "case_id": "case_01...",
  "case_version": 17,
  "tenant_id": "tenant_01...",
  "actor": {"type": "service", "id": "investigation-runtime"},
  "purpose": "aml_case_investigation",
  "jurisdiction_profile": "us-bank-2026-08@4f2c...",
  "operation": "transactions.list",
  "resource_scope": {"account_ids": ["acct_..."], "from": "...", "to": "..."},
  "budget": {"max_records": 500, "deadline_at": "..."},
  "cursor": null,
  "source_snapshot": "ledger-view@2026-08-31T08:00:00Z",
  "data_classification": ["customer-confidential", "financial"],
  "trace_id": "00-..."
}
~~~

A successful result must add `source_system`, `source_schema_version`, `source_event_time`, `observed_at`, `completeness`, `next_cursor`, `record_ids`, `content_hashes`, `transform_id`, `freshness`, `coverage`, `warnings`, and `provenance_refs`. A failed result must be typed as denied, invalid, unavailable, throttled, partial, stale, version-conflict, or unknown—not collapsed into an empty list.

### Contract invariants

- Tenant, purpose, jurisdiction, actor, case, resource scope, and deadline are required and reauthorized server-side.
- The adapter returns stable opaque identifiers; the model never invents or dereferences raw SQL, URLs, filesystem paths, or vendor query languages.
- Pagination is explicit. A partial page is not “no evidence.” Coverage and truncation are first-class.
- Source time and acquisition time are distinct. Mutable lookups identify the snapshot or revision used.
- Free text is untrusted data and remains tagged through retrieval, context, output, and display.
- Results disclose classification, retention, residency, export, and redistribution constraints.
- Each D2/D3 proposal binds the case version and evidence hashes it reviewed. Any material change invalidates review.

## Integration inventory and admission decision

This inventory identifies systems to evaluate; it does not qualify a vendor or connector. Production admission is for
one exact operation, tenant/application configuration, role, object/field scope, region, API/schema, intended use and
failure/effect contract, with an owner and expiry.

| Integration | Include when | Required semantics | Reject or contain when |
|---|---|---|---|
| Core/ledger transaction systems | Always for relevant accounts and rails | Transaction, posting, reversal, status, account, counterparty, source time, currency, amount, rail identifiers, completeness | No stable transaction identity, silent backfill, or ambiguous reversal semantics |
| Payment messages | Cross-border, correspondent, wire, card, or instant-payment evidence matters | Versioned message definition, original fields, intermediary chain, UETR/message identifiers where applicable, amendment/cancellation lineage | Parsed narrative without original message and schema version |
| Customer/account/KYC master | Always | Party/account relationships, verification method/status, valid time, record owner, source documents, change history | A flattened “customer risk” field with no contributors or effective date |
| Case/alert management | Always | Alert lineage, case version, assignment, deadlines, disposition vocabulary, approvals, duplicate/supersession rules | Model transcript used as the case record |
| Sanctions and official watchlists | When screening or escalation is in scope | Issuer, program/regime, entry ID, aliases, identifiers, list revision, publication time, applicable restrictions | Aggregator as sole legal source; latest-only data without reproducible snapshot |
| PEP and adverse-media providers | When lawful and approved | Provider/version, source link, publication/event date, language, entity candidate, confidence and limitations | Opaque score, scraped sensitive data without legal basis, or unverifiable article summary |
| Identity/device/fraud telemetry | Fraud scope or cross-channel evidence | Device/account binding method, event time, precision, consent/purpose, spoofing limits | Identifier treated as a person or shared across purposes without authorization |
| Company registries/LEI/beneficial ownership | Entity or ownership analysis | Registry jurisdiction, record/revision, valid time, ownership type/percentage, source quality | LEI treated as proof of current beneficial ownership; stale or challenged record hidden |
| Graph/feature platform | Repeated network analysis | Node/edge type, temporal validity, construction job/version, truncation, source refs | Model creates durable edges directly or opaque embeddings replace reviewable relationships |
| Filing/FIU gateway | Only after approved decision | Current schema, amendment/correction rules, receipt/status lookup, deadline, credential owner, reconciliation | Browser automation, credential in prompt, or submit endpoint exposed to the model |
| Restriction/payment-control workflow | Only for a separately authorized action | Exact object, action, legal/policy basis, duration, owner, approval, status query, compensation limits | Generic “block customer” tool or retry without current state check |

For payment schemas, pin the applicable [ISO 20022 repository](https://www.iso20022.org/iso20022-repository/e-repository) release or other rail definition rather than assuming a field name is stable. For legal-entity data, preserve registry provenance and surface challenged or stale records; [GLEIF explicitly provides a challenge process](https://www.gleif.org/en/lei-data/gleif-data-quality-management/challenge-lei-and-vlei-data).

Use the operation manifest, provider-specific limits, qualification pipeline and end-to-end exercises in
[Qualified adapters and worked investigation flows](11-qualified-adapters-and-worked-investigation-flows.md). A read
capability does not imply write authority, and a provider receipt never expands the human/legal meaning approved in the
case workflow.

## Connector acceptance test

Before an integration can enter a release manifest, verify:

1. **Identity:** What is the semantic object? Can it be merged, reversed, corrected, superseded, or reused?
2. **Time:** Which fields mean occurrence, effective, posting, publication, observation, and ingestion time?
3. **Coverage:** Can the source prove a complete page/window? How are delayed and deleted records represented?
4. **Version:** Can a reviewer reproduce the case view after the source changes?
5. **Authorization:** Are tenant, role, purpose, jurisdiction, field, row, and export restrictions enforced at the source boundary?
6. **Reliability:** Which failures are safe to retry? How are rate limits, timeouts, partials, unknown outcomes, and recovery exposed?
7. **Privacy:** What may enter prompts, caches, traces, evaluation sets, and cross-border transfers? How do retention and legal hold interact?
8. **Security:** Can source content alter instructions, invoke links, or smuggle secrets? Are outbound destinations allowlisted?
9. **Operations:** Who owns the schema, credentials, rotation, breaking changes, incident contact, and decommissioning?
10. **Evidence:** Do contract tests cover live-like pagination, backfill, correction, permission denial, staleness, and malformed content?

## Failure containment

| Failure | Unsafe interpretation | Required behavior |
|---|---|---|
| Empty source result | “No suspicious activity” | Distinguish complete-empty from denied, filtered, partial, stale, and unavailable |
| Schema field disappears | Continue with null | Quarantine adapter version, mark evidence gap, block affected conclusion |
| Cursor repeats/skips | Treat assembled page set as complete | Detect cursor cycle/coverage break; stop or retry from a safe checkpoint |
| Source updates mid-run | Mix revisions silently | Pin a snapshot or record revision; restart affected analysis on material change |
| Duplicate alert/referral | Open independent cases and effects | Use a semantic dedupe key; link/supersede under reviewed policy |
| Entity provider changes candidate | Rewrite prior case fact | Append new revision; preserve earlier evidence and re-evaluate dependent claims |
| Free text contains instructions | Follow it | Render as quoted evidence; never elevate to control instructions |
| Adapter timeout after a write | Blind retry | Mark outcome unknown and reconcile by semantic operation ID |
| Destination reports success but no receipt | Assume committed | Query authoritative status; keep effect pending/reconciling |
| Model provider unavailable | Bypass controls or lose case | Pause reasoning, retain durable state, use deterministic/manual degradation path |

## Explicitly rejected shortcuts

- A generic database/query tool, unrestricted browser, shell, email, or document connector in the reasoning plane.
- Credentials, access tokens, filing forms, or destination URLs in prompts or model-visible memory.
- A vector store as the canonical customer, transaction, list, policy, or case database.
- Vendor “risk scores” or generated summaries without inputs, version, calibration, and limitations.
- An aggregator as the only sanctions authority or an adverse-media hit as proof of wrongdoing.
- Direct model-to-model delegation as a substitute for typed source adapters.
- Best-effort logs as audit evidence, or observability retention as the case retention policy.

## Readiness checklist

- [ ] One bounded loop and one authoritative case workflow are documented.
- [ ] Every connector has an owner, contract version, authorization rule, purpose, classification, residency, and lifecycle.
- [ ] Evidence distinguishes event time, observed time, snapshot/revision, coverage, and transformation.
- [ ] Pagination, correction, backfill, deletion, duplicate, and unknown-result behavior have tests.
- [ ] The model has no generic query/write tool or destination credential.
- [ ] Policy, approval, decision, effect, receipt, reconciliation, and telemetry records are separated.
- [ ] Failure of a source makes uncertainty visible and cannot silently strengthen a conclusion.

## Sources and next guide

- [W3C PROV Overview](https://www.w3.org/TR/prov-overview/)
- [ISO 20022 e-Repository](https://www.iso20022.org/iso20022-repository/e-repository)
- [FATF — Guidance on Beneficial Ownership of Legal Persons](https://www.fatf-gafi.org/en/publications/Fatfrecommendations/Guidance-Beneficial-Ownership-Legal-Persons.html)
- [AWS Builders' Library — Making retries safe with idempotent APIs](https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/)
- [Tool contracts](../../tools/tool-contracts.md)
- [Tool results, artifacts, and provenance](../../tools/tool-results-artifacts-and-provenance.md)
- [Qualified adapters and worked investigation flows](11-qualified-adapters-and-worked-investigation-flows.md)

Next: [Customer, transaction, entity, and network evidence](03-customer-transaction-entity-and-network-evidence.md).
