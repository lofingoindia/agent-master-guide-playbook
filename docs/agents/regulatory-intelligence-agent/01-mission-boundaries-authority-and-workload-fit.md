# Mission, Boundaries, Authority, and Workload Fit

## Mission

Reduce the time between an authoritative regulatory change and a well-evidenced, professionally reviewed organizational decision. The system owns evidence acquisition, change cases, hypotheses, version history, review routing, and handoff integrity. It does not own the legal or compliance conclusion.

The useful outcome is not “summarize new rules.” It is a reproducible chain:

```text
authoritative artifact
→ verified change and temporal facts
→ pinpoint provision candidates
→ scoped applicability hypothesis
→ named professional decision
→ accepted obligation candidate
→ acknowledged owner handoff
```

Each arrow is a recorded derivation or decision, not a free-form paragraph.

## Real-agent qualification

An agent loop is justified only if it improves a measured ambiguity that deterministic software cannot solve economically.

| Workload | Deterministic baseline | Potential model value | Default decision |
|---|---|---|---|
| Poll known feeds and identify new IDs | Cursor, digest, and metadata rules | None | Keep deterministic |
| Match exact regulator/jurisdiction/product tags | Rules and reference data | None | Keep deterministic |
| Compare structured amendment XML | Tree-aware diff | Explain complex change, but do not generate the diff | Deterministic first |
| Extract actor/action/condition/exception across varied prose/PDF | Patterns plus review | Candidate extraction and cross-reference synthesis | Bounded model candidate |
| Decide applicability to a company | Rules over complete facts plus professional judgment | Explain predicate evidence and unknowns | Human decision; model cannot own |
| Interpret ambiguous, conflicting, or multilingual provisions | Professional legal analysis | Retrieve and structure competing evidence | Human decision; no model conclusion |
| Update GRC/policy records | Workflow integration | Draft a structured handoff | Deterministic commit after approval |
| Attest compliance | Audit/control evidence | None within this category | Out of scope |

If known rules, metadata, search, and human review meet the coverage, timeliness, and cost objective, stop at Stage 0.

The production split is enforceable, not aspirational:

| Deterministic control plane owns | Bounded model may propose | Model can never supply |
|---|---|---|
| Source admission/status/rights, polling, overlap, reconciliation, identity, bytes/digests and structural diff | Which admitted cross-reference/span to inspect next | Official status, source precedence, missing source text or coverage completeness |
| Legal/knowledge-time records, version/supersession graph and as-of queries | Typed date/relationship candidates with pinpoint evidence | Entry/effect/applicability date without source evidence or professional rule |
| Organization fact snapshots and predicate schema | Explanation of matched/unmatched/unknown facts | A missing fact, exception, threshold state or final applicability decision |
| Case state, budgets, routing policy, reviewer role, approvals and stop conditions | Cited extraction, comparison, hypothesis, question and handoff draft | Legal interpretation, obligation acceptance, approver/destination selection or authority expansion |
| Effect intent, credential issuance, idempotency, remote receipt and reconciliation | Nothing after the payload is sealed | Dispatch, retry, cancellation result, policy/control mutation, filing or attestation |

Every release runs a **model-omission trajectory**: disable the model before and during each permitted call. Official-source capture, coverage/watermarks, deterministic diff, human-readable evidence packet, professional review, existing timers/effects, reconciliation and audit must remain correct. Model availability is not part of the source-coverage SLO, and a model outage never converts degraded analysis into “no regulatory change.”

## Owned and excluded outcomes

### Owned

- approved source catalog, subscription coverage, cursors, and reconciliation watermarks;
- immutable source acquisitions with authenticity, status, language, rights, and provenance;
- instrument/version/provision identity and explicit temporal facts;
- source changes, corrections, withdrawals, consolidations, and unresolved conflicts;
- applicability hypotheses against versioned entity/product/activity facts;
- interpretation questions, review packets, decisions, and supersession history;
- obligation candidates, internal mapping candidates, impact-analysis cases, and owner handoffs;
- effect receipts, source/case incidents, evaluation artifacts, and correction/failure mining.

### Explicitly excluded

- legal advice or a representation that an output can be relied on as legal advice;
- a model declaring whether legislation, a rule, guidance, standard, exception, or deadline applies;
- a model choosing between conflicting authorities or authentic language versions;
- setting risk appetite, policy, control design, implementation priority, or disclosure position;
- implementing technical, operational, financial, HR, product, or governance controls;
- testing controls, issuing audit findings, certifying, filing, signing, or attesting compliance;
- managing litigation, investigations, legal holds, contracts, negotiations, or privileged matters;
- contacting a regulator, customer, counterparty, court, or the public without a separate approved workflow;
- silently treating commercial summaries, machine translations, search snippets, or editorial consolidations as the law.

## Actors and responsibility model

| Actor | Accountable for | Cannot delegate to the model |
|---|---|---|
| Regulatory-intelligence owner | Source catalog, scope, workflow, coverage objectives, and triage policy | The claim that coverage is comprehensive |
| Legal counsel or qualified legal reviewer | Legal interpretation, authority conflicts, applicability decision where legal judgment is required | Professional judgment or sign-off |
| Compliance/regulatory owner | Operational applicability, obligation acceptance, regulatory risk, and escalation | Acceptance of an obligation or exception |
| Entity/product/activity data owner | Current organization facts and their effective history | Missing or disputed facts |
| Policy owner | Whether and how policy changes | Policy approval |
| Control owner | Control design, implementation, evidence, and operating status | Implementation or effectiveness claims |
| Records/privacy/content-rights owner | Retention, deletion, confidentiality, privilege handling, and licences | Rights clearance or privilege determination |
| Platform/security owner | Runtime, identities, egress, secrets, tenancy, incident controls | Security acceptance and break-glass authority |
| Model | Candidate extraction, comparison, evidence synthesis, and explicit abstention | Any owner decision above |

## Authority classes

These workload classes specialize the repository’s danger tiers. Local governance may lower any ceiling.

| Class | Capability | Default authority | Approval and proof |
|---:|---|---|---|
| R0 | Retrieve an already-authorized artifact or fact snapshot | Autonomous read | Tenant, rights, source, purpose, and as-of checks |
| R1 | Create a candidate source link, extraction, hypothesis, issue, or internal draft | Autonomous proposal | Schema/provenance validation; clearly non-authoritative |
| R2 | Create/update an internal case or send a review request | Pre-authorized bounded effect | Semantic operation ID, exact scope, receipt, reconciliation |
| R3 | Send an accepted obligation/mapping package to a policy/GRC destination | Exact human approval or sealed workflow delegation | Decision ID, proposal digest, destination, expiry, precondition, receipt |
| R4 | Change policy/control status, attest compliance, file/disclose externally, or assert a legal position | Prohibited in this blueprint | Separate qualified human-owned process |

The normal production ceiling is R2. R3 is an internal handoff, not implementation. There is no path from R3 to R4 merely because the system performs well.

## Admission contract

A run is admitted only when trusted application code can establish:

```yaml
admission:
  tenant_id: tenant_acme
  initiating_principal: user_482
  purpose: monitor_and_assess_change
  jurisdiction_scope: [EU]
  entity_scope: [entity_acme_eu]
  product_scope: [payments_api]
  as_of:
    legal_time: 2026-08-31
    knowledge_time: 2026-08-31T08:00:00Z
  source_catalog_release: eu-sources-17
  rights_policy_release: content-rights-9
  authority_ceiling: R1
  confidentiality_ceiling: confidential
  deadlines:
    wall_clock: PT15M
    review_due: 2026-09-02T12:00:00Z
  budgets:
    tool_calls: 20
    retrieved_bytes: 15000000
    model_tokens: 60000
```

The model may not broaden jurisdiction, entity, product, purpose, destination, confidentiality, or authority. “Global,” “all products,” or “all regulations” is rejected unless those scopes resolve to explicit governed sets and capacity.

## Workload risk classes

| Risk | Example | Required response |
|---|---|---|
| Coverage risk | A monitored gazette/feed is missing or the cursor is behind | Show coverage watermark; stop claims of completeness; reconcile |
| Authority risk | Nonbinding guidance is treated as a binding rule | Preserve source status; hard validation; reopen affected cases |
| Temporal risk | Publication date is used as effective/applicability date | Keep date types separate; require pinpoint evidence and review |
| Applicability risk | Threshold or product fact is inferred | Mark unknown; request the authoritative fact; professional decision |
| Interpretation risk | Exceptions/cross-references conflict | Preserve competing evidence; escalate; no automatic resolution |
| Rights risk | Licensed standard is embedded or exported without permission | Deny processing/export; quarantine artifacts; notify rights owner |
| Confidentiality risk | Counsel notes enter a general model trace | Block at context compiler; minimize/redact; incident response |
| Effect risk | GRC item was created but receipt recording failed | `outcome_unknown`; reconcile before any retry |
| Reputation/legal risk | Draft output is sent externally as an organizational position | R4 is absent; independent external-communication workflow required |

## Representative journeys

### Normal final-rule change

1. A source adapter sees a new or revised authoritative identifier.
2. The acquisition service stores raw bytes, metadata, signature result, digest, retrieval record, and rights profile.
3. Deterministic comparison creates a change case; the model extracts provision and date candidates with pinpoint citations.
4. The context compiler joins a pinned entity/product fact snapshot. The model emits a hypothesis with `matched`, `not_matched`, and `unknown` predicates.
5. A professional resolves interpretation and applicability. The decision binds the exact source, facts, and policy release.
6. Accepted obligation candidates and mapping candidates go to named owners.
7. An approved handoff is committed once, acknowledged, and later reconciled against destination state.

### Correction or delayed effective date

The new artifact is never folded silently into the old one. It creates a superseding source version, impact traversal finds every derived provision, hypothesis, decision, obligation, and handoff, and affected cases reopen. Previous decisions remain queryable under the knowledge time at which they were made.

### Source outage

The connector records an outage observation, keeps its last successful watermark, backs off under a bounded policy, and triggers an alternative official channel if approved. The user sees `coverage_degraded`; the system does not interpret an empty response as no changes.

### Unofficial translation

The system stores the authentic source-language rendition and the translation as different artifacts. A machine translation may help discovery, but every extracted candidate cites the source-language location and carries translation provenance. A qualified reviewer resolves any material language ambiguity.

### Cancellation during handoff

Cancellation stops new analysis and unsent effects. If dispatch may already have occurred, the case becomes `reconciling`; a late receipt is correlated to the same operation. Cancellation never rewrites a professional decision or source history.

## Stop and escalation matrix

| Condition | Machine action | Human owner |
|---|---|---|
| Missing authoritative artifact or failed signature where signature is expected | Quarantine; retry official source; preserve failed bytes | Source owner/security |
| Source statuses conflict | Present both and configured precedence; no conclusion | Legal/regulatory owner |
| Date is conditional, retrospective, jurisdiction-partial, or event-triggered | Store raw expression and unresolved predicate | Qualified legal reviewer |
| Applicability fact missing/stale | Request fact; hypothesis remains incomplete | Fact owner plus compliance/legal |
| Source text and consolidation disagree | Prefer neither automatically; retrieve affecting acts and official-status policy | Legal/regulatory owner |
| Licensed right expires during a case | Stop retrieval/model use/export as policy requires | Content-rights owner |
| Owner rejects interpretation or obligation | Record rejection and rationale; do not repair it into an approval | Named decision owner |
| Destination cannot prove handoff state | Freeze conflicting handoffs and reconcile | Workflow/GRC owner |

## Boundary acceptance checklist

- [ ] Every outcome has a human or deterministic owner and system of record.
- [ ] Applicability and interpretation decisions name required professional roles.
- [ ] R4 capabilities are absent from tool discovery and credentials.
- [ ] Source coverage is an explicit catalog and watermark, not a marketing claim.
- [ ] Stage-0 deterministic performance and cost are measured before adding a model.
- [ ] A case can finish as awaiting evidence, superseded, or indeterminate without pressure to invent an answer.
- [ ] Compliance audit, legal operations, policy, and control implementation handoffs are documented.
- [ ] Product language says “decision support,” not “automated legal advice” or “guaranteed compliance.”

## Related guides

- [Reference architecture, sources, tools, and integrations](02-reference-architecture-sources-tools-and-integrations.md)
- [Temporal, version, and applicability semantics](04-temporal-version-and-applicability-semantics.md)
- [Provision extraction, obligations, impact, and handoff](05-provision-obligation-impact-and-handoff.md)
- [Execution boundaries](../../runtime/execution-boundaries.md)
