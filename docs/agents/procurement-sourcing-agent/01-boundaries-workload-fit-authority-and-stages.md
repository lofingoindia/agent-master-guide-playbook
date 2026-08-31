# Boundaries, Workload Fit, Authority, and Zero-to-Production Stages

> **Purpose:** Decide whether model-directed work is justified, establish accountable authority, and define evidence-based gates from ordinary software to production operation.

## Start with the non-agent baseline

Implement or measure the current process before choosing a model:

```text
authenticated requisition
  -> schema and duplicate checks
  -> category/threshold decision table
  -> approved supplier/catalog/framework lookup
  -> deterministic RFx template and workflow
  -> evaluator scorecard
  -> approval and handoff
```

Use that baseline when the input is structured, the route and supplier set are known, the evaluation is formulaic, and exceptions are rare. A rules engine, search UI, spreadsheet with controlled formulas, or sourcing platform can be safer, faster, cheaper, and more explainable than an agent.

A model-directed loop is justified only for measured ambiguity such as:

- mapping free-form demand and documents to a governed category while surfacing uncertainty;
- open-ended discovery across approved market sources with evidence and contradiction handling;
- extracting heterogeneous bid schedules into a fixed comparison schema with precise citations;
- assembling a complex, evidence-backed recommendation where the set of useful read tools depends on findings.

Do not use an agent to calculate thresholds, decide conflicts, open bids, cast official scores, authorize an award, draft legal obligations, or operate orders and shipments.

## Workload contract

Write one contract per sourcing-case class before implementation:

```yaml
case_type: competitive_software_services_sourcing
accountable_owner: strategic_sourcing_director
initiators: [authenticated_requester, approved_portfolio_plan]
authoritative_case_system: procurement_workflow
procurement_regime: private_enterprise_policy_v12
terminal_outcomes:
  - award_decision_handoffs_acknowledged
  - no_award_recorded
  - cancelled_with_reason
value_basis: estimated_total_commitment
currency_policy: treasury_fx_daily_close_v4
maximum_case_age: P180D
autonomous_ceiling: P1_propose
consequential_effects:
  - publish_event
  - invite_supplier
  - broadcast_clarification
  - change_deadline
  - submit_award
  - create_onboarding_handoff
prohibited_effects:
  - change_policy_or_approvers
  - reveal_sealed_or_competitor_bid_data
  - edit_legal_language
  - create_purchase_order
  - change_supplier_bank_details
completion_oracle: acknowledged_handoffs_plus_reconciled_effect_ledger
```

`procurement_regime` is not decoration. Public, regulated, donor-funded, emergency, framework, and private-enterprise events can have different participation, notice, communication, documentation, standstill, challenge, exclusion, and award rules. The application selects an effective-dated profile; the model does not infer one from prose.

## Category separation test

| Seam | Procurement-specific answer | Why a generic neighbor is insufficient |
| --- | --- | --- |
| Environment | Requisition, spend, supplier, risk, e-sourcing, bid, approval, and handoff systems | Generic cases lack sealed competitive-event semantics |
| Authority | Publish, invite, communicate, open, evaluate, award, and onboard under event-specific roles | A normal record update can distort competition or expose confidential bids |
| State and time | Long waits, fixed deadlines, amendments, sealed opening, independent evaluation, challenges | Ordinary workflows do not imply criteria freeze or equal supplier treatment |
| Ground truth | Immutable bid snapshots, published criteria, official scores, approval and platform receipts | A plausible recommendation is not proof of a fair event |
| Recovery | Duplicate invitations, lost response after publish/award, stale approvals, partial handoff | Blind retry can create unequal or contradictory supplier treatment |
| Evaluation | Real event state, confidentiality, policy invariants, normalized math, human procurement judgment | Text quality alone cannot establish correctness |

This passes the registry's distinctness boundary while deliberately reusing canonical workflow, state, effect, security, and evaluation mechanics.

## Authority model

Grant authority per **case type × event × operation × supplier/lot × value/risk band × time**, not to an “agent” in general.

| Tier | Capability | Examples | Production posture |
| --- | --- | --- | --- |
| **P0 — Observe** | Read purpose-limited, event-authorized projections | Spend snapshot, published notice, permitted supplier evidence | First safe deployment after privacy/access tests |
| **P1 — Propose** | Produce typed, cited suggestions | Category candidates, longlist, extraction, comparison narrative | Default useful ceiling |
| **P2 — Stage** | Create reversible internal drafts | RFx draft, review packet, handoff draft | Requires exact target and data-egress controls |
| **P3 — Approved commit** | Execute one exact consequential effect | Publish approved event, invite named suppliers, create handoff, submit award | Independent approval, preconditions, receipt, reconciliation |
| **P4 — Narrow preauthorization** | Low-risk repeatable effect under deterministic policy | Internal reminder or already-approved evidence refresh | Optional; never bid opening, scoring, award, policy, or legal commitment |
| **P5 — Excluded autonomy** | Broad or self-modifying authority | Change criteria/policy, award independently, expose bids, amend contract, issue PO | No tool or credential exists |

The same deployment can be P1 for supplier discovery, P2 for an RFx draft, and P0 for sealed bid data until an authorized opening transition. Human review is meaningful only when the reviewer sees exact evidence and consequence, has time and authority to reject, and is independent under the event's segregation rules.

## Accountable ownership

| Decision or artifact | Agent worker | Requester | Procurement | Evaluator / SME | Compliance / security | Award authority | Legal / onboarding / supply chain |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Business need and acceptance outcome | C | A/R | C | C | C | I | I |
| Category and sourcing strategy | R-propose | C | A/R | C | C | I | I |
| Supplier longlist | R-propose | C | A/R | C | C | I | I |
| Due-diligence disposition | R-evidence | I | C | I | A/R by risk domain | I | I |
| Criteria, weights, and normalization rules | R-draft | C | A/R | C | C | I | C |
| Official qualitative score | R-evidence | I | C | A/R | I | I | I |
| Conflict decision and recusal | I | I | C | R-declare | A/R policy owner | I | I |
| Award recommendation | R-draft | I | A/R | C | C | C | I |
| Award decision | I | I | C | I | C | A/R | I |
| Contract language and obligations | I | I | C | I | C | I | A/R legal |
| Supplier master/onboarding acceptance | I | I | C | I | C | I | A/R onboarding |
| Orders, inventory, shipments, recovery | I | I | I | I | I | I | A/R supply chain |

The “agent” is never accountable. Service identities also cannot replace required human roles.

## Stage 0 — qualify the problem

Deliver:

- the workload contract and jurisdiction/regime map;
- a deterministic process with measured cycle time, rework, error, exception, and reviewer effort;
- representative normal, edge, challenge, cancellation, and recovery cases;
- a data inventory for bids, supplier PII, beneficial ownership, financial evidence, and confidential source-selection information;
- a list of invariants that a model can never own.

**Exit gate:** a specific unstructured or open-ended judgment problem materially harms the baseline, and a bounded model trial improves it without weakening fairness, confidentiality, or control. Otherwise stop at Stage 0.

## Stage 1 — first bounded loop

Build one synchronous, read-only loop for a low-risk task such as category-candidate generation or bid-field extraction from synthetic documents:

1. controller supplies a fixed task contract and schema;
2. model chooses from two or three read-only tools;
3. each tool returns source, freshness, trust label, and evidence ID;
4. output lists facts, citations, uncertainty, missing evidence, and `complete` or `escalate`;
5. hard limits bound turns, tool calls, result bytes, tokens, wall time, and cost.

No supplier communication, platform mutation, persistent cross-case memory, web browser with logged-in authority, or multi-agent delegation exists.

**Exit gate:** on held-out cases, every accepted fact resolves to evidence, unsupported fields abstain, budget exhaustion stops safely, malicious document instructions are ignored, and no write path is reachable.

## Stage 2 — useful MVP

Select one business unit and category with moderate volume, reversible drafts, known reviewers, and no emergency, defense, clinical, employment, or other unusually sensitive procurement.

Add:

- real read adapters with tenant/event authorization and freshness;
- a durable case shell, but keep effects at P0–P2;
- an event-scoped context builder and short-term working state;
- deterministic normalization functions and a cited review packet;
- human correction capture separated from automatic learning;
- an isolated evaluation environment with realistic requisitions, suppliers, bids, and policy versions.

**Exit gate:** shadow processing improves defined quality or handling metrics, official decisions remain human-owned, no bidder data crosses its authorized event/role, and operators can finish every case manually when the model is unavailable.

## Stage 3 — reliable v1

Add only the P3 effects required for one complete slice, commonly event publication or an onboarding/contract handoff—not all effects at once.

Required capabilities:

- authoritative state, append-only domain events, compare-and-swap versions, timers, cancellation, and release compatibility;
- deterministic continuity snapshots and event/bid-isolated context compaction;
- exact approvals, conflict/SoD enforcement, effect intents, receipts, unknown outcomes, and reconciliation;
- versioned connector adapters, explicit downstream idempotency semantics, rate limits, timeouts, and schema tests;
- curated domain memory and closed-case evaluation artifacts only where justified.

**Exit gate:** kill-after-every-boundary, duplicate delivery, reordering, expired approval, connector drift, deadline change, and timeout-after-commit tests converge to one valid business outcome with no blind retry.

## Stage 4 — production readiness

Add:

- user, workload, tenant/legal-entity, role, run, policy, approval, credential, event, bid, and effect identity lineage;
- sealed-bid isolation, least privilege, short-lived credential brokering, egress policy, attachment sandboxing, and privacy retention/deletion;
- unsampled audit evidence plus redacted diagnostic traces, metrics, SLOs, runbooks, and on-call ownership;
- immutable behavior manifests and offline, adversarial, compatibility, shadow, canary, and rollback gates;
- independent stop-admission, disable-effects, revoke, quarantine, and evidence-preservation controls.

**Exit gate:** procurement, security, privacy, compliance, platform, and operations owners sign the control profile; severe policy failures are zero-tolerance release blockers; incident and kill-switch drills pass without agent cooperation.

## Stage 5 — scale and resilience

Introduce cells and queues only after measured need. Partition at least by tenant/legal entity and consider stricter cells for regulated regimes or sealed-bid workloads. Separate intake, document, discovery, model, effect, and reconciliation worker pools so a large attachment or slow registry cannot starve event-deadline work.

Set numeric targets for admission, queue age, event-deadline slack, connector quota, OCR/model concurrency, evidence bytes, unknown-effect age, manual fallback capacity, and spend per case. Test regional/cell loss and authoritative-store restore.

**Exit gate:** load and failure tests prove bounded queues, fair tenant scheduling, no cross-cell cache or vector leakage, safe degradation under provider/source outage, and recovery within organization-approved RTO/RPO and event deadlines.

## Stage 6 — continuous evolution

Govern the whole behavior bundle: code, model and provider, instructions, tool schemas, connector versions, policy and taxonomy releases, normalization formulas, context builder, memory sources, and evaluators.

Production feedback creates a proposed dataset item, not a direct prompt or memory mutation. Domain reviewers adjudicate failures and counterfactuals. Every change passes critical procurement slices repeatedly, then shadow/canary release. Active sourcing events are pinned, migrated through a tested path, or quarantined; they do not silently change scoring behavior mid-event.

**Exit gate:** the team can explain why a change was proposed, which evidence supports it, how it affects active events and historical reproducibility, which critical slices passed, and how to roll back or deprecate it.

## Consolidated stage-gate checklist

- [ ] A deterministic alternative and its measurements are recorded.
- [ ] The regime profile, accountable owner, authority ceiling, non-goals, and completion oracle are explicit.
- [ ] Criteria, calculations, conflicts, approvals, official scores, and award authority remain outside model control.
- [ ] Bid and supplier information has event-, tenant-, role-, purpose-, retention-, and deletion-aware access.
- [ ] Every accepted model claim has a source or is labeled an assumption/proposal.
- [ ] Every P3 effect has exact intent, current preconditions, semantic identity, receipt, verification, and reconciliation.
- [ ] Human fallback, stop controls, and incident ownership do not depend on the model/provider.
- [ ] Outcome, control, reliability, latency, cost, confidentiality, and escalation gates pass on repeated trials.
- [ ] Active cases have a release pin, migration, or quarantine strategy.

## Related guides

Continue with [Reference architecture, runtime, and integrations](02-reference-architecture-runtime-and-integrations.md). The [README](README.md) provides the full guide map. Universal architecture trade-offs are covered in [Custom loop vs framework vs workflow engine](../../comparisons/custom-loop-vs-framework-vs-workflow-engine.md).

