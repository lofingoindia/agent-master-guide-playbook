# Mission, Boundaries, Workload Fit, and Stages

Status: production design guide  
Last reviewed: 2026-08-31

The mission is to coordinate a travel transaction safely: capture bounded intent, retrieve current evidence, compare feasible products, obtain exact consent, coordinate approved effects, verify fulfillment, and manage the resulting obligations. The mission is not “make travel decisions for a person.” That framing hides identity, authority, freshness, legal, accessibility, and irreversible-effect boundaries.

## Mission contract

Define one mission contract per operating envelope:

```yaml
mission_contract:
  tenant_id: tenant_acme
  point_of_sale: IN
  supported_journeys: [domestic_air, hotel_only]
  supported_travelers: [verified_employee]
  providers: [air_provider_a, hotel_provider_b]
  permitted_actions:
    search: autonomous
    reprice: autonomous
    hold: exact_approval
    book_and_fulfill: exact_approval
    cancel: exact_approval
    exchange: manual_only
    disruption_rebook: proposal_only
  spend:
    currency: INR
    per_trip_cap: "150000.00"
    approval_policy: travel-policy-2026-08-12
  excluded:
    - unaccompanied_minors
    - group_bookings_over_9
    - medical_clearance
    - visa_or_entry_eligibility_decisions
    - split_tender
    - offline_fare_fulfillment
  manual_owner: queue_travel_ops_india
  kill_switches: [all_writes, provider_write, payment_handoff, disruption_automation]
```

The envelope is the intersection of business policy, provider capability, identity assurance, jurisdiction, tested workflow, and operational support. Marketing claims or an API endpoint's existence do not expand it.

## Scope decomposition

| Lifecycle | Agent responsibility | Source of truth | Stop or handoff boundary |
|---|---|---|---|
| Intent | Convert conversation/form data into explicit constraints and unresolved questions | Traveler-approved `JourneyIntent` | Conflicting principals, missing traveler, or unverified contact |
| Search | Query qualified channels with exact passenger/occupancy, dates, places, point of sale, currency, and service needs | Provider search response as a time-bounded observation | Unsupported traveler/product, stale provider, or incomplete mandatory input |
| Compare | Normalize comparable fields, preserve provider terms, apply hard feasibility, explain trade-offs | Snapshot store plus deterministic policy/feasibility services | Unknown material rule, inaccessible option, or incomparable total price |
| Quote | Reprice/preview/validate a chosen product immediately before approval | Provider's fresh quote or documented hold | Expired offer, changed terms, unavailable inventory, or price beyond policy |
| Approve | Present exact commercial and fulfillment terms; record authenticated decision | Approval service | Ambiguous assent, wrong principal, expired quote, step-up required |
| Commit | Prepare and dispatch one typed, idempotent supplier/payment operation | Effect ledger plus supplier/payment systems | Precondition failure, uncertain dispatch, challenge, or partial result |
| Fulfill | Obtain ticket/EMD/voucher/rail document and confirm services | Supplier read-back | Reservation without fulfillment, missing segment/product, SSR unconfirmed |
| Service | Retrieve schedule, process approved change/cancel/refund, preserve continuity | Supplier order/PNR/booking and after-sales records | Capability gap, involuntary/waiver case, coupon mismatch, manual supplier action |
| Disruption | Detect impact, revalidate remaining journey, propose bounded recovery | Operational provider feeds and current supplier records | Safety/duty-of-care risk, mass event, no safe option, unreachable traveler |
| Reconcile | Match intent, attempts, receipts, supplier state, and financial references | Supplier + payment + finance systems in their domains | Contradiction or aged unknown requires named owner |

## Adjacent-owner contracts

The interface to adjacent systems must be typed, not conversational.

### Executive or personal operations

Receives the broader objective and calendar context. It delegates a travel intent with named travelers, time windows, place constraints, budget/policy reference, and contact channel. The travel agent returns option IDs, itinerary revisions, booking/fulfillment status, and obligations. It does not silently change meetings, expense policy, or personal commitments.

### Customer support

Owns general cases, communication cadence, sentiment, and multi-product support. This agent supplies a signed evidence packet: journey, supplier references, facts versus inferences, effects, outstanding obligations, deadlines, traveler reachability, and recommended next safe actions. Support text cannot authorize a booking effect.

### Payments and finance

Payment service owns credential/token lifecycle, authentication, authorization, capture, reversal, refund movement, and disputes. Finance owns accounting and settlement truth. The travel workflow uses opaque references and observes states. It never declares money returned because a supplier accepted a cancellation or refund request.

### Supplier fulfillment

Airline, hotel, rail carrier/retailer, car-rental supplier/intermediary, GDS, aggregator, accredited agency, or consolidator owns inventory, order/PNR/booking, ticketing, voucher/fulfillment, coupon/product status, and provider rules under its contract. The agent records and reconciles; it does not “correct” the supplier from memory.

## Workload-fit test

Score a candidate workflow before introducing a model.

| Question | Good fit | Bad fit |
|---|---|---|
| Can traveler, tenant, and acting principal be proven? | Stable IDs and scoped delegation | Shared inbox, alias, or inferred passenger |
| Can current price and conditions be refreshed? | Provider has reprice/preview or binding hold | Scraped display with no booking contract |
| Can effects be independently read back? | Lookup by client reference and booking/order retrieval | Only a transient success page or email |
| Can duplicate effects be prevented? | Provider idempotency/client reference plus ledger | No stable reference and no reconciliation path |
| Can post-sales state be retrieved? | Change/cancel/refund quote and status contract | Offline-only workflow with no supported operator |
| Are accessibility needs representable and confirmable? | Raw need, normalized request, supplier acknowledgement | A free-text note presented as guaranteed service |
| Is entry/document guidance sourced and bounded? | Dated authoritative information and escalation | Model-generated eligibility conclusion |
| Is a human owner staffed for the service window? | Named queue, SLA, authority, and contact | “The model will handle exceptions” |
| Is the workflow testable without real harm? | Sandbox/simulator and reversible pilot | First test uses live irreversible inventory |
| Can the exact operation be qualified? | Versioned action-level manifest and destructive/finality tests | One provider-level “integrated” flag |

Any red answer blocks Stage 5. The simplest correct alternative is often a deep link, structured handoff, or operator workbench.

## Risk and authority tiers

| Tier | Examples | Default authority | Required controls |
|---|---|---|---|
| T0: information | Explain baggage fields, show sourced entry-information links, display stored itinerary | Deterministic or read-only | Provenance, freshness, privacy filtering |
| T1: recommendation | Compare current options, explain trade-offs, propose recovery | Model-assisted read-only | Hard feasibility filter, citations, uncertainty, no invented terms |
| T2: preparation | Reprice, build a cart/workbench, request a reversible hold | Automatic reprice; exact approval for supplier hold | Expiry, release path, spend/inventory bounds, effect ledger |
| T3: commitment | Purchase, ticket, change, cancel, refund request, paid ancillary | Exact authenticated approval | Bound commercial snapshot, policy check, idempotency, read-back, audit |
| T4: preauthorized recovery | Narrow re-accommodation within documented disruption policy | Disabled by default | Traveler opt-in, strict cost/time/service caps, safe suppliers, real-time kill, post-action notice, drills |
| T5: excluded | Visa eligibility, medical fitness, unrestricted purchasing, unbounded web action, group/complex fare desk decisions | Never autonomous | Qualified human/authority handoff |

T4 is not “the agent knows the traveler.” It is a versioned standing instruction with authenticated provenance, explicit scope, expiry, revocation, budget, supplier/product constraints, and observable use.

## Deterministic alternatives first

| Need | Default non-model implementation | Add model only for |
|---|---|---|
| Capture airport/date/passenger data | Typed form, calendar/location selector, validation rules | Resolving genuinely ambiguous natural-language intent with confirmation |
| Check policy | Deterministic policy engine | Explaining why a rule applied |
| Find feasible connections | Provider results + minimum-connect-time/route rules | Summarizing trade-offs among already feasible options |
| Rank options | Weighted score/Pareto frontier with visible criteria | Interpreting soft preferences within validated bounds |
| Compare fare/rate conditions | Structured field comparison; human review for raw text | Plain-language explanation with source pointers |
| Track booking | Durable state machine and provider read-back | Condensing evidence for a human |
| Detect disruption | Supplier webhook/feed and rules | Classifying ambiguous evidence, never overriding confirmed operations data |
| Choose re-accommodation | Constraint solver/enumeration and policy | Explaining feasible alternatives or eliciting preference |
| Decide visa/document eligibility | Never | None; provide sourced information and authority handoff |
| Retry a write | Effect state machine and provider contract | None |

Use a browser only as a qualified, contained mechanism when no supported API exists and the supplier permits it. Browser automation does not change authority, freshness, PCI, identity, or reconciliation requirements.

### Choose the least autonomous viable product

| If the actual need is | Prefer | Escalate to an agent only when | Do not use the agent for |
|---|---|---|---|
| Find available products under explicit filters | Deterministic metasearch or direct-supplier search | Natural-language ambiguity or cross-component trade-offs materially improve completion | Claiming whole-market coverage from a limited provider set |
| Enforce corporate travel rules | Forms plus deterministic policy/approval workflow | The user needs a grounded explanation or feasible exception alternatives | Inventing exceptions, policy, or delegation |
| Book a simple known product | Direct supplier deep link or deterministic checkout | Multiple live sources, constraints, and long-running recovery justify coordination | Hiding merchant, supplier, terms, or final price |
| Service an existing booking | Supplier self-service or trained travel adviser | Typed evidence aggregation and workflow continuity reduce operator load | An after-sales action without current quote/read-back |
| Handle complex/group/medical/document-sensitive travel | Human travel adviser with qualified tools | The agent remains a read-only evidence assistant inside the adviser's controls | Autonomous eligibility, medical-clearance, waiver, or group-fare decisions |

Use an agent only if its bounded interpretation and coordination value exceeds the extra model, privacy, security, latency, evaluation, and recovery cost. A deterministic implementation that satisfies the traveler should remain the production default and the degraded-mode fallback.

## Bounded model-loop contract

The model runs inside a deterministic macro-workflow. A recommended initial envelope is:

```yaml
model_loop:
  max_tool_calls_per_turn: 6
  max_replans_per_stage: 2
  max_quote_refreshes_without_user_input: 1
  max_provider_fanout: 4
  max_runtime_seconds: 45
  parallel_reads: true
  parallel_writes: false
  write_serialization_keys: [tenant_id, journey_id, supplier_booking_id]
  effects_from_model_output: forbidden
```

These numbers are starting controls, not universal constants. Tune them using success, latency, cost, and escalation evidence. The coordinator—not the model—enforces every count and deadline.

### Mandatory stop and escalation conditions

Stop local planning and enter a typed state when any of these occurs:

- traveler, tenant, acting principal, point of sale, or credential scope mismatch;
- required quote expired or materially changed;
- a material condition is absent, contradictory, or `unknown`;
- required accessibility/service request is unacknowledged or rejected;
- payment authentication, CVV, or credential collection is needed;
- document, visa, health, safety, sanctions, or legal interpretation exceeds the information boundary;
- an external effect is `unknown`, partially applied, or conflicts with read-back;
- a provider lacks a qualified path for the requested after-sales action;
- cross-tenant data is retrieved or suspected;
- the tool/policy/provider version differs from the qualified release;
- maximum calls, replans, quote refreshes, elapsed time, cost, or queue deadline is reached;
- no feasible option remains, or every option violates a hard constraint;
- the traveler is unreachable inside a disruption deadline.

The terminal result must say why execution stopped, what evidence is reliable, what is unknown, which deadlines remain, and who owns the next action.

## Stage gates

### Stage 0 — deterministic contract and manual baseline

Build typed intent, identity, provider, quote, approval, effect, and escalation schemas. Run the complete workflow manually and record failure modes. No model and no external writes.

Exit only when:

- every mutable fact has one owner and freshness rule;
- supported and excluded traveler/product/jurisdiction combinations are explicit;
- privacy, PCI, accessibility, retention, and legal reviews are recorded;
- manual booking, fulfillment, cancellation, disruption, unknown-effect, and data-incident runbooks have named owners;
- test fixtures contain no real payment credentials or unnecessary identity documents.

### Stage 1 — read-only observation

Integrate search and retrieval for one provider/product envelope. Normalize without losing raw provider references and terms.

Exit only when held-out contract tests show at least:

- 100% tenant/traveler/point-of-sale scope preservation;
- 100% displayed prices and conditions linked to source snapshot and observation time;
- correct duplicate/out-of-order webhook handling;
- zero writes under read-only credentials and network policy;
- deterministic replay produces the same projection from the same event set.

### Stage 2 — cited explanation

Add bounded language interpretation after deterministic filtering. Require citations to stored evidence fields and expose unknowns.

Exit only when the release thresholds in guide 9 pass for grounding, material-condition recall, abstention, prompt injection, accessibility language, and document-boundary behavior. A fluent uncited answer fails.

### Stage 3 — approval-ready proposals

Add live reprice/preview, material-drift comparison, exact approval rendering, and policy-aware alternatives. Still no commits.

Exit only when:

- 100% of approval screens bind the expected quote hash, itinerary revision, traveler set, amount/currency, terms, expiry, payment reference, and service requests;
- every material drift invalidates the prior approval in the test matrix;
- the ranker never promotes a hard-infeasible option;
- operators judge the evidence packet sufficient for the target workflow.

### Stage 4 — staging and qualified holds

Enable one supplier action that is documented as a hold/workbench and has an explicit expiry/release/retrieve path. Do not call a reservation reversible without provider evidence.

Exit only after timeout-after-dispatch, expiry, release, duplicate, quota, and provider-outage drills produce no duplicate live product and no orphaned hold beyond the alert threshold.

### Stage 5 — narrow commitment

Enable one exact-approved booking-and-fulfillment path. Keep changes, refunds, complex fares, unsupported carriers, and disruption automation manual until each earns its own gate.

Exit only when:

- duplicate confirmed bookings are zero across the adversarial suite;
- every ambiguous outcome reaches verified, proved-absent, recovery-required, or named escalation within the SLO;
- supplier read-back proves all expected tickets/vouchers/fulfillments and service-request state;
- payment challenge and failure paths do not expose credentials or fabricate booking success;
- kill, reconciliation, compensation, and incident drills pass.

### Stage 6 — scaled operations and governed evolution

Add providers, products, regions, and only then narrow preauthorized disruption actions. Each new cell has isolated credentials, quotas, queues, SLOs, and rollback.

Exit criteria are continuous: canary gates, drift detection, incident drills, privacy/access reviews, supplier contract refresh, cost budgets, reconciliation backlog, and operator feedback remain within policy. Missing evidence demotes the affected capability; it does not widen fallback behavior.

### Stage exercise and exit-evidence ledger

| Stage | Required measurable exercise | Exit evidence retained |
|---|---|---|
| 0 | Operators complete at least the agreed representative air/hotel/rail/car and after-sales scenarios manually, including one document, accessibility, partial-trip, and data-incident case | Scenario inventory, owner/RACI, median/p95 handling time, failure taxonomy, approved schemas, reviews, and runbook sign-off |
| 1 | Replay each provider fixture with stale, malformed, duplicate, out-of-order, wrong-tenant, wrong-market, and unknown-enum variants | Contract report showing 100% scope/provenance invariants, zero network writes, parser/version pins, and deterministic projection hashes |
| 2 | Run held-out grounded explanations plus injection, sensitive-inference, legal/document abstention, and assistive-technology sessions | Per-slice claim precision/material recall/abstention, zero hard-boundary failures, human-factor findings, and deterministic fallback comparison |
| 3 | Mutate every approval-bound field individually and across quote-expiry/payment-step-up races | Exhaustive mutation report with 100% material invalidation, preserved nonmaterial rendering, exact accessible approval artifacts, and operator usefulness score |
| 4 | Fault the hold at every transport boundary; advance virtual time through release/expiry; restart every worker | Effect/read-back trace proving no duplicate or orphan beyond the declared threshold, restored clocks, and a staffed recovery receipt |
| 5 | Run a declared high-volume randomized create/fulfill suite with partial traveler/product results, unknown outcomes, payment divergence, and kill activation | Zero duplicate confirmed effects, 100% audit linkage, bounded unknown/fulfillment age, exact supplier read-back, and incident commander sign-off |
| 6 | Run seasonal peak, mass disruption, provider degradation, cell failover, reconciliation surge, behavior-bundle canary, and rollback game days | Measured SLO/error-budget/cost/operator-load results, RTO/RPO proof, one-writer evidence, traveler-impact review, and current requalification records |

Counts, percentile targets, and acceptable operator load are declared before the exercise from the risk envelope; they are not selected after observing results. Each artifact identifies release manifest, dataset, provider/action cell, owner, execution date, limitations, and raw evidence references.

## Anti-patterns

- Treating a search response as guaranteed inventory or a remembered price as current.
- Asking the model to “check fare rules” without a provider rule payload and explicit unknown state.
- Storing a card, passport scan, health detail, or authentication answer in a prompt or vector store.
- Calling an unfulfilled PNR “confirmed” or a refund request “money returned.”
- Reusing a generic approval after price, traveler, date, product, or service request changes.
- Giving the model a generic `browser()` or `execute_api()` tool with inherited session credentials.
- Retrying after a timeout with a new idempotency key.
- Assuming cancellation reverses payment or that multi-supplier rollback is atomic.
- Inferring accessibility needs or diagnoses from text instead of confirming the traveler's requested assistance.
- Encoding one jurisdiction's refund or passenger-rights rule in the system prompt.
- Treating a supplier's supported standard version as proof that every carrier/content source supports every operation.
- Adding multiple agents to simulate organizational departments while sharing the same credentials and mutable state.

## Review checklist

- [ ] The mission envelope names supported travelers, products, providers, jurisdictions, actions, and exclusions.
- [ ] Adjacent owners accept typed handoffs and cannot authorize through prose.
- [ ] Deterministic implementations exist for feasibility, policy, state, retries, and effects.
- [ ] Model calls, replans, time, cost, quote refresh, and fan-out are bounded.
- [ ] Stop conditions map to typed terminal or escalation states.
- [ ] Each Stage 0–6 promotion has measurable evidence and an owner.
- [ ] A useful read-only/manual mode survives model and write disablement.

Continue with [reference architecture, control/data planes, and runtime](02-reference-architecture-control-data-planes-and-runtime.md).
