# Mission, Boundaries, and Workload Fit

Status: production design guide  
Last reviewed: 2026-08-31

The first engineering task is not choosing a model. It is defining a small operational contract whose facts, authority, and completion conditions are testable. "Optimize the supply chain" is not a contract. "Detect a likely missed delivery promise for one outbound parcel network, assemble cited evidence, propose allowed recovery options, and execute an exactly approved service upgrade" can be one.

## Define the operating unit

An operating unit is the smallest slice that can be isolated, evaluated, and owned. Bind it before implementation:

```yaml
operating_unit:
  tenant_id: tenant_acme
  legal_entity_id: in01
  region: south_asia
  business_flow: outbound_fulfillment
  exception_type: delivery_promise_at_risk
  included_modes: [parcel]
  included_sites: [blr_dc_01]
  authoritative_systems:
    order: oms_prod
    inventory: wms_blr
    shipment: tms_prod
    carrier_milestones: carrier_adapter_set_v3
  owner_role: logistics_exception_manager
  autonomous_ceiling: D1
  maximum_effect_tier: D3
  retention_policy_id: logistics_ops_2026_04
```

The scope keys are security boundaries, routing keys, metric dimensions, and deletion boundaries. They cannot exist only in prompt text.

## Choose a suitable first exception

A good first workload has repeatable inputs, a bounded decision clock, several real but enumerable alternatives, and a downstream effect that can be read back. Examples include:

- a carrier milestone is missing beyond a lane-specific threshold;
- an order promise is at risk after an updated ETA distribution;
- a warehouse short pick conflicts with an existing allocation;
- a tender was rejected and an approved carrier/service fallback exists;
- a planned transfer is no longer feasible because a source inventory position changed;
- a shipment appears delivered but the receiving advice or proof is absent.

Avoid a first workload when it requires open-ended network redesign, unmodeled regulatory judgment, negotiation, plant control, general procurement, or simultaneous writes to many systems without stable idempotency and read-back.

### Choose the least-agentic mechanism

An exception does not justify an agent merely because its input contains text. Choose the least complex mechanism that meets the decision contract:

| Mechanism | Use when | Do not add an agent for |
|---|---|---|
| Deterministic rule | Inputs, threshold, action, and exceptions are enumerable | Missing-scan clocks, stale-source alerts, exact policy tests, or fixed escalation routing |
| Constraint solver or optimizer | The problem is combinatorial but facts, hard constraints, and objective are formal | Load building, vehicle routing, slot assignment, or allocation where prose reasoning adds no value |
| Durable workflow | The path is known but contains waits, callbacks, approvals, retries, and compensations | Tender-response waiting, appointment confirmation, document collection, or cancellation reconciliation |
| Operator workbench | Novel judgment, negotiation, regulatory interpretation, safety disposition, or weak system interfaces dominate | A manual dispatcher can see the relevant evidence and owns the decision under an acceptable SLO |
| Bounded agent inside a workflow | Mixed structured and unstructured evidence must be synthesized and several policy-permitted alternatives need explanation | Any case whose facts, feasibility, authority, or completion would still depend on trusting model prose |

Use a hybrid only when each part earns its place: rules detect, a solver proves feasibility, the workflow persists clocks and effects, the model explains ambiguity, and a named person grants residual authority. Measure the bounded agent against the strongest non-agent baseline. If it does not improve safe resolution rate, operator effort, or deadline performance enough to justify its latency, cost, and new failure modes, remove it.

### Workload scorecard

Score each candidate `0` to `2`; do not start effects below `18/24`.

| Dimension | 0 | 1 | 2 |
|---|---|---|---|
| Identity | Free text or collisions | Mapping exists with gaps | Canonical IDs and collision workflow |
| Source authority | Disputed | Mostly known | Per-field source matrix approved |
| Event quality | Unordered and opaque | Some timestamps/versions | Provenance, freshness, duplicate rules |
| Constraints | Tacit | Partially encoded | Hard/soft constraints versioned |
| Alternatives | Unbounded | Operator playbook | Typed action catalog |
| Effect safety | Generic write | Narrow write, weak read-back | Typed write, stable key, authoritative read-back |
| Reversibility | Irreversible | Forward recovery possible | Staged/reversible or strong compensation |
| Approval | Informal | Role known | Exact, expiring, auditable decision |
| Evaluation | Anecdotes | Historical cases | Simulator/replay and outcome oracle |
| Operations | No owner | On-call exists | SLO, queue, kill switch, incident path |
| Privacy/security | Unknown | Generic controls | Classified fields and scoped credentials |
| Business value | Unclear | Plausible | Baseline and attributable measure |

## Preserve domain ownership

Supply-chain process taxonomies intentionally span broad organizational work. Agent authority should not. Use the following separation when a case crosses teams:

| Trigger | Logistics agent responsibility | Handoff owner | Handoff payload |
|---|---|---|---|
| Supplier cannot meet an awarded order | Report fulfillment risk and physical-flow alternatives | Procurement | Order line, awarded supplier, evidence, required-by date, impact; no supplier recommendation unless requested |
| Factory output is late or quality-blocked | Recompute downstream impact from an authoritative production status | Manufacturing | Material/order links, published availability, shipment dependencies; no equipment or quality action |
| Warehouse database or CDC feed is corrupt | Quarantine affected evidence and reduce authority | Database/DataOps | Source, schema/version, watermarks, affected projections; no pipeline repair |
| Invoice or claim needs processing | Attach logistics evidence | Back-office workflow | Shipment, receipts, proof, discrepancy; no generic case ownership |
| Dangerous-goods classification is uncertain | Stop planning that depends on classification | Qualified dangerous-goods role | Item, existing declaration, route/mode, source citations; never infer classification |
| Sanctions, customs, or export-control status is uncertain | Block the affected option | Compliance/customs | Parties, locations, commodity codes, route, policy version; no model-only determination |

Typed handoffs include the reason, authoritative references, urgency, scope, and requested decision. They do not transfer credentials or silently expand the receiving agent's authority.

Human authority is positive and explicit. Every open exception has one accountable owner, one current queue, a response clock, an escalation target, and a disposition vocabulary. A queue acknowledgement transfers attention, not business authority; an operator may approve only operations allowed by their authenticated role and limit. Safety, dangerous-goods, trade, inventory-control, customer-promise, and spend decisions remain with the qualified role even when the agent assembles all evidence.

## Establish the authority ladder

Use a danger tier for each tool operation, not for the entire connector:

| Tier | Logistics examples | Default control |
|---|---|---|
| D0: isolated computation | Parse a fixture, calculate slack, solve a sandbox route | Autonomous in a resource-limited sandbox |
| D1: read | Read order status, inventory, shipment, carrier event, policy, or rate quote | Autonomous with scoped identity and audit |
| D2: staged/reversible | Create an internal draft, reserve sandbox capacity, place proposal in review queue | Policy-limited; explicit expiry and cleanup |
| D3: consequential commit | Tender, book, reroute, expedite, reserve/reallocate stock, cancel, or send an external notice | Exact approval or preauthorized deterministic runbook |
| D4: authority change | Change policy, approval matrix, connector scope, credential, retention, or autonomous ceiling | Proposal only; separate administrative workflow |

The initial production ceiling should be D1. D2 comes after isolation and cleanup are proven. D3 begins with one narrow operation and exact approval. A D3 runbook can become preauthorized only if inputs, constraints, limits, monitoring, reversal/forward recovery, and owner are deterministic and independently reviewed. D4 never belongs to the runtime agent.

## Specify facts, estimates, and decisions

Model and API outputs often blur different epistemic states. Require an explicit class:

| Class | Example | Required treatment |
|---|---|---|
| Observed | Scan at a facility at an event time | Preserve source, event time, capture time, identity, and quality |
| Declared | Carrier says a delay reason is weather | Attribute to declarant; do not convert to verified cause |
| Planned | Planned departure at 18:00 | Preserve plan version; not evidence of execution |
| Committed | Carrier accepted a tender | Preserve commitment receipt and terms; later verify status |
| Estimated | P90 arrival 2026-09-02 14:00 | Carry as-of, horizon, model/version, quantiles, calibration slice |
| Derived | Order promise risk from ETA and cut-off rules | Preserve formula/rule version and inputs |
| Proposed | Upgrade service or allocate from another site | No effect until feasibility, policy, and authority pass |
| Authorized | Named approver accepted exact proposal hash | Expires on time or material state change |
| Verified | Downstream read-back matches intended postcondition | May close the effect, subject to business completion |

Never turn absence of evidence into a positive event. "No departure scan received" is not "shipment did not depart." It is an evidence gap with a freshness policy.

## Write the exception charter

Every exception class needs a versioned charter:

```yaml
exception_charter:
  id: delivery_promise_at_risk/v1
  detection_rule: p90_arrival_after_customer_promise
  authoritative_inputs:
    - order.promise_time
    - shipment.current_leg
    - eta_forecast.quantiles
  minimum_evidence:
    - canonical_order_line
    - canonical_shipment_leg
    - forecast_as_of_within: PT30M
  allowed_proposals:
    - no_action_monitor
    - carrier_service_upgrade
    - alternate_fulfillment_site
    - split_remaining_quantity
    - customer_contact_draft
  hard_constraints:
    - inventory_nonnegative
    - handling_compatibility
    - dangerous_goods_policy
    - customs_and_sanctions_clearance
    - approval_cost_limit
  forbidden_actions:
    - select_new_supplier
    - relax_regulatory_constraint
    - change_customer_promise_without_authority
  closure:
    - risk_cleared_and_monitored
    - approved_recovery_verified
    - accountable_owner_accepted_residual_risk
```

Policy code validates the charter. The prompt may explain it but cannot redefine it.

## Decision clock and service objective

An exception has a usable decision window, not just a due date:

`decision_deadline = latest_effective_action_time - approval_budget - effect_budget - verification_budget - safety_margin`

When the remaining time is shorter than the model or solver budget, the coordinator must choose a deterministic fallback: notify the on-call owner with a minimal evidence packet, run a preauthorized rule, or take no action. It must not skip approval or constraints to "save time."

Declare separate objectives for:

- source-event freshness and projection lag;
- exception detection latency;
- evidence-compilation latency;
- proposal and solver latency;
- human approval latency;
- effect acknowledgement and authoritative verification;
- maximum age of `effect_unknown`;
- manual queue age at each severity.

Business metrics such as on-time-in-full delivery, perfect-order performance, premium-freight spend, and inventory turns are important but confounded. They cannot substitute for agent reliability measures.

## Stage gates by authority

### Observation gate

- Canonical identities survive split orders, split shipments, merged loads, substitute items, logistics units, and location aliases.
- Event time, capture/record time, and ingestion time are distinct.
- Duplicate, late, out-of-order, corrected, and missing events are replay-tested.
- The source-of-truth and freshness matrix is approved per field.
- Tenant and legal-entity boundaries are enforced outside the prompt.

### Explanation gate

- Every material claim links to a typed observation, rule, forecast, or solver result.
- Untrusted carrier messages, EDI free text, documents, emails, and notes cannot issue instructions.
- Unsupported cause claims are labeled hypotheses.
- The agent abstains on identity collision, stale mandatory evidence, or policy uncertainty.

### Proposal gate

- Hard constraints are validated outside the model.
- Forecasts expose uncertainty and are evaluated by lane, horizon, mode, carrier, and disruption regime.
- Solver status distinguishes optimal, feasible, timeout/unknown, invalid, and infeasible.
- At least one `no action / monitor` alternative is considered when safe.
- Cost, service, inventory, and downstream effects are shown as ranges or scenarios when uncertain.

### Commit gate

- The exact approved intent includes canonical resources, expected versions, limits, expiry, and postconditions.
- Credentials are scoped to the operation and operating unit.
- A stable semantic operation ID exists before the first write attempt.
- Unknown outcomes are reconciled before retry.
- Cancellation, partial failure, and downstream unavailability drills pass.

### Scale gate

- Per-resource write serialization and connector backpressure are load-tested.
- Manual-review demand fits staffed capacity during bursts.
- Kill switches, read-only degradation, DR replay, and evidence retention are exercised.
- Each new region, mode, carrier, item class, or exception type has an explicit validation delta.

## Production anti-patterns

- Treating the agent, transcript, vector store, or event stream as the inventory/shipment source of truth.
- Using one generic ERP, carrier, SQL, browser, email, or `update` tool across tenants and effect tiers.
- Resolving conflicts by latest timestamp, flattening units/segments, or treating missing events as physical non-occurrence.
- Asking the model to invent ETA confidence, prove feasibility, relax hard constraints, interpret law, or choose its own credential.
- Approving a narrative and then allowing the runtime to change resource, carrier, quantity, route, cost, or action.
- Retrying a timeout with a new idempotency key, or marking a 2xx/202/EDI technical acknowledgement as verified business completion.
- Letting independent shipment agents compete for shared inventory, capacity, slots, or operator attention.
- Passing an ever-growing transcript, retrieving raw prior incidents, or writing production outcomes directly into persistent memory.
- Optimizing average cost/service while hiding starvation, allocation harm, operator workload, or non-canary network effects.
- Canarying only a model or prompt while adapters, context, policy, solver, thresholds, dependencies, and UI change independently.

## Rejection criteria

Reject or redesign the workload if any of these remain true:

- there is no accountable owner for residual risk;
- success is defined only as the absence of complaints;
- the only interface is a generic UI automation path for a critical effect;
- the downstream system cannot accept a client reference and cannot be searched after timeout;
- regulatory or safety constraints exist only in model context;
- a natural-language item or location name is considered sufficient identity;
- the organization expects the model to resolve disagreements between authoritative systems silently;
- a single approval is intended to authorize changing resources, costs, or actions after the fact;
- production traces must contain unrestricted shipment documents, personal data, or commercial terms;
- the rollout has no shadow period, simulator, replay set, or failure-injection environment;
- disabling the agent would stop normal logistics operations rather than return them to a manual path.

## Boundary review checklist

- [ ] One operating unit and one exception charter are named.
- [ ] Procurement, manufacturing, data platform, back-office, safety, and compliance boundaries are signed off.
- [ ] Canonical item, location, party, carrier, order, shipment, and logistics-unit namespaces are declared.
- [ ] Each mutable field has an authoritative source and freshness rule.
- [ ] All forecasts and derived facts carry provenance and version.
- [ ] Hard constraints and forbidden actions are machine-enforced.
- [ ] D0-D4 tiers are assigned per operation.
- [ ] Human owners, approval limits, expiries, and escalation clocks are configured.
- [ ] Effect postconditions and read-back paths exist.
- [ ] Manual fallback and kill behavior are acceptable to operations.
- [ ] The business baseline and agent-specific SLOs were measured before rollout.

Continue with [reference architecture and runtime selection](02-reference-architecture-and-runtime-selection.md). The broader authority model is defined in the [cross-cutting controls research packet](../../research/packets/agent-blueprint-cross-cutting-controls.md).
