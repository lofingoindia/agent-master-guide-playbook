# Identities, State, Events, and Projections

Status: production design guide  
Last reviewed: 2026-08-31

Most dangerous logistics-agent errors look like reasoning failures but begin as state failures: the wrong item alias, a location reused across systems, a split shipment treated as one object, a late carrier scan overwriting newer evidence, or an estimate presented as a fact. The state contract must make those errors representable and blockable.

## Separate five kinds of state

| State kind | Purpose | Mutation model | Authority |
|---|---|---|---|
| Source record | Order, inventory, shipment, booking, receipt, carrier status | Owned by ERP/OMS/WMS/TMS/carrier | Designated external system |
| Observation | What one source asserted at a time | Append-only; corrections link to earlier facts | Evidence, not universal truth |
| Projection | Queryable current operational view | Rebuildable under versioned rules | Derived, with conflicts exposed |
| Exception workflow | Agent lifecycle, timers, owners, attempts, approvals | Versioned state transitions | Durable coordinator |
| Effect record | Intended, attempted, acknowledged, and verified external action | Append-only transition history | Effect ledger plus downstream read-back |

Telemetry is a sixth diagnostic stream. It is not a substitute for any row above.

## Canonical identity model

Identity is namespaced and typed. Use recognized keys when they are actually available—such as GTIN for a trade item, GLN for a party or location, SSCC for a logistics unit, GSIN for a shipment, GINC for a consignment, and UN/LOCODE for trade or transport locations—but preserve internal and partner identifiers too. A standard-shaped string is not proof of a verified mapping.

```yaml
entity_ref:
  entity_type: logistics_unit
  canonical_id: lu_01J7...
  identifiers:
    - namespace: gs1:sscc
      value: "089012345678901234"
      verification: authoritative_master_data
      effective_from: 2026-01-01T00:00:00Z
    - namespace: wms_blr:lpn
      value: LPN-449112
      verification: observed_mapping
      effective_from: 2026-08-30T04:11:09Z
  tenant_id: tenant_acme
  legal_entity_id: in01
  identity_status: resolved
```

### Required entity relationships

Model at least:

- order and order line, including purchase, sales, or transfer order type;
- item, lot/batch, serial, ownership, quality/status segment, and unit of measure;
- party, legal entity, site, sublocation, dock, geospatial point, and timezone;
- shipment, consignment, transport equipment, shipment leg, stop, and carrier/service;
- logistics unit and its packing hierarchy;
- inventory position, reservation, allocation, pick, pack, dispatch, receipt, and discrepancy;
- customer promise, requested time, appointment window, planned milestone, estimate, and observed event.

Relationships are effective-dated. One order line can feed multiple shipments; one shipment can consolidate many order lines; a shipment can have multiple legs and carriers; a logistics unit can be repacked. Never assume a one-to-one mapping.

### Exact entity and effective-time contracts

The canonical layer preserves upstream identity and meaning; it does not manufacture a universal logistics object. Each object carries `tenant_id`, `legal_entity_id`, canonical ID, typed external identifiers, source record and version, `valid_from`/`valid_to` for business applicability, `recorded_at`, and mapping status. `valid_to` is half-open unless an upstream contract explicitly says otherwise. A new source revision appends a version; it never mutates history in place.

| Entity | Identity and version boundary | Effective-time and lifecycle semantics |
|---|---|---|
| Order | Order type + issuing legal entity + source order ID; customer/supplier references are aliases | Header version cannot stand in for line versions; requested, accepted, released, cancelled, and closed are distinct effective states |
| Order line | Order ID + stable line ID, not display sequence alone | Item, quantity, unit, ship-from/to, promise, substitution, cancellation, and fulfillment links are independently versioned; split/merge creates explicit relationships |
| Shipment | Shipment/consignment namespace + source shipment ID; GSIN/GINC may be aliases when verified | Planned movement aggregate; booking/tender status, execution status, and physical observations remain separate; shipment version changes do not rewrite leg history |
| Leg | Shipment ID + stable leg ID; sequence number is an attribute, not durable identity | Origin/destination, mode, carrier/service, planned window, connection dependency, and status have effective intervals; insert/remove/resequence creates a new plan version |
| Load | TMS or carrier load ID scoped to dispatcher/carrier account and mode | A consolidation and capacity assignment, not a shipment synonym; membership, equipment, driver/tractor references, seal, and dispatch status are versioned |
| Container | Owner/operator code + equipment serial/check digit where governed, plus source equipment ID | Physical transport equipment; ownership, type/size, seal, custody, empty/full state, and shipment/load membership are effective-dated; reuse never reuses an old canonical instance blindly |
| Package | Package/logistics-unit ID scoped to issuer; tracking number and SSCC are separate aliases unless the provider equates them | Packing hierarchy, contents, weight/dimensions, label, and parent container can change through repack; old parentage closes and new parentage opens |
| Inventory position | Item + facility/location + owner + lot/serial + quality/status + other source-defined segments | A versioned source snapshot at `as_of`, not an event or reservation; on-hand, ATP, quarantine, in-transit, and projected quantities keep source formulas |
| Reservation | Reservation ID from the authoritative inventory system + linked demand/order line | Requested, soft-held, hard-reserved, consumed, released, expired, and reversed are distinct; quantity, unit, segment, source version, expiry, and offset/consumption link are mandatory |
| Facility | Verified GLN, UN/LOCODE, or internal site ID plus facility type; dock/bin IDs are child locations | Operating calendar, timezone, cut-offs, capabilities, access points, dock geometry, and temporary closure use effective releases; a geocode is not facility identity |
| Carrier/service | Carrier legal party/account ID plus provider service code and contract/region | Eligibility, transit promise, equipment/mode, lane, cut-off, rate terms, and dangerous-goods acceptance are effective-dated; marketing names are display labels only |
| Route | Route plan ID + route version; topology nodes and directed edges reference canonical facilities/points | Planned path differs from observed path; edge mode, allowed direction, connection, schedule, restriction, and validity interval are pinned for each solve |
| Appointment | Facility + appointment ID or provider confirmation ID | Requested window, offered slot, confirmed slot, checked-in, completed, missed, cancelled, and superseded remain distinct; retain local zone, inclusivity, dock/resource, and confirmation version |
| Document | Document type + issuer + issuer document ID/revision; artifact hash proves bytes, not business identity | Draft, issued, amended, superseded, cancelled, accepted, and rejected are separate; issue/effective/expiry times, signer, schema, language, and related shipment/order are preserved |

Control and derived records need equally exact identities:

| Record | Identity and version boundary | Non-negotiable semantics |
|---|---|---|
| Constraint | Constraint ID + governed release; parameters and applicability form part of the version | Source, jurisdiction/contract, hard/soft class, effective interval, scope predicate, units, validator, waiver rule, and superseding release are explicit |
| Forecast/ETA | Forecast ID + target + measure + model/features release + `as_of` | Horizon, cutoff, quantiles/distribution, applicability, calibration slice, and supersession link are mandatory; a refreshed forecast never overwrites the prior estimate |
| Exception | Exception ID + charter version + scoped resource set | Detection episode, state version, owner, severity, decision clock, evidence snapshot, closure reason, and reopen/supersession links are durable |
| Tender | Tender ID + shipment/load/leg + carrier account + tender round; EDI 204 control number is a partner identifier | Prepared, transmitted, syntactically acknowledged, accepted, rejected, expired, withdrawn, and cancelled are distinct; an X12 997/999 is not a 990 business acceptance |
| Proof | Proof ID + proof type + issuer + subject + artifact hash | Delivery scan, signature, photo, receipt advice, and human attestation have different evidentiary weight; capture time, signer/custodian, access scope, verification, dispute, and correction state are preserved |
| Approval | Approval ID + immutable proposal/snapshot/intent hash | Authenticated principal, role, decision, reason, bounds, policy version, grant time, expiry, revocation, and invalidation cause are append-only; approval text is not authority |
| Effect | Effect ID + globally unique semantic operation ID; attempts have separate IDs | Intent hash, resource versions, lease/fence, approval, dispatch attempts, receipts, unknown state, postconditions, reconciliation, compensation, and terminal evidence remain linked |
| Correction | Correction ID + corrected observation/document/proof ID + correcting source/version | States what assertion is withdrawn or replaced, why, and from when; it appends evidence, triggers deterministic rebuild/invalidation, and never erases the corrected record |

If an upstream system lacks a stable identifier or version, the adapter may create a scoped surrogate from immutable source fields, but must mark its collision domain and qualification evidence. A surrogate based only on mutable content, display number, timestamp, address, or sequence is not safe for effects.

### Identity resolution states

| Status | Meaning | Allowed behavior |
|---|---|---|
| `resolved` | One current canonical mapping is supported | Normal reads and scoped proposals |
| `unresolved` | No mapping meets confidence and authority rules | Ask for authoritative evidence; no effect |
| `ambiguous` | Several candidates remain | Present candidates to an authorized resolver |
| `collision` | One external ID maps incompatibly | Quarantine affected observations and page data owner |
| `retired` | Mapping ended or entity superseded | Historical reads only unless explicit successor mapping |

Embedding similarity can suggest candidates. It cannot finalize an item, carrier, or location identity used for inventory or movement effects.

## Observation envelope

Normalize every inbound assertion into an envelope while retaining the source payload as a hashed, access-controlled artifact:

```json
{
  "observation_id": "obs_01J...",
  "observation_type": "shipment.departed",
  "subject": {"type": "shipment_leg", "id": "leg_774_2"},
  "scope": {"tenant_id": "tenant_acme", "legal_entity_id": "in01"},
  "source": {
    "system": "carrier_dhl_gf",
    "record_id": "evt_88921",
    "schema": "adapter:dhl_gf_tracking/v2.4",
    "source_version": "17"
  },
  "event_time": "2026-08-31T08:10:00+05:30",
  "recorded_at": "2026-08-31T08:14:21+05:30",
  "ingested_at": "2026-08-31T02:44:27Z",
  "location": {"id": "loc_blr_air", "namespace": "canonical"},
  "quantities": [],
  "quality": {
    "identity": "resolved",
    "time_precision": "second",
    "source_confidence": "declared_by_carrier"
  },
  "correction": null,
  "payload_artifact": {"id": "art_01J...", "sha256": "..."},
  "dedupe_key": "carrier_dhl_gf|evt_88921|17"
}
```

GS1 EPCIS distinguishes business event time from record time and models error declarations as additions rather than destructive edits. Use the same principle even when the source does not implement EPCIS. Preserve what was received, when it allegedly occurred, when the source recorded it, and when the agent saw it.

## Event-time and ordering rules

There is no trustworthy global order across ERP, WMS, TMS, carrier, EDI, and document streams. The projection uses per-source versions, event-time rules, and explicit conflict policies.

| Condition | Projection behavior | Operational consequence |
|---|---|---|
| Exact duplicate | Retain receipt metric; do not reapply | No new exception/effect |
| Same source ID, higher version | Append new observation; supersede in source-specific view | Recompute affected projections |
| Late but valid event | Insert by event time; preserve ingest time | Re-evaluate only material windows; do not erase later facts |
| Out-of-order milestone | Accept evidence if valid; expose sequence anomaly | May open data-quality exception |
| Source correction | Link correction to original; rebuild relevant projection | Invalidate dependent proposals/approvals |
| Conflicting sources | Apply declared field authority or show conflict | Never ask model to silently pick truth |
| Missing expected event | Record absence/overdue evidence fact | Do not invent the missing physical event |
| Future/impossible time | Quarantine or low-trust flag per rule | Block dependent automation |

Watermarks are per stream and indicate processing completeness, not physical truth. A carrier feed can be fully consumed and still stale operationally.

## Quantity, unit, and inventory semantics

Store decimal quantities with an explicit unit, packaging level, item identity, inventory segment, and conversion source. Never let the model perform implicit `case`, `pallet`, `each`, weight, volume, or dimensional conversions.

Unit conversion is a versioned deterministic operation. Its receipt records source quantity/unit, target unit, item and packaging level, numerator/denominator, rounding mode and scale, master-data release, effective interval, and residual. Item-specific conversions such as `12 EA = 1 CS` cannot be reused for another item or after pack-size change. Dimensional weight preserves the carrier divisor, unit system, service/region, and rate effective date. If conversion provenance is missing, compare in the original unit or stop; never round inventory silently.

An inventory position should distinguish at least:

```yaml
inventory_position:
  item_id: item_1042
  location_id: loc_blr_dc_01
  segments:
    owner: acme_india
    lot: lot_20260817
    quality_status: released
  on_hand: {value: "120.000", unit: EA}
  reserved: {value: "60.000", unit: EA}
  allocated_soft: {value: "20.000", unit: EA}
  available_to_promise: {value: "40.000", unit: EA}
  source: wms_blr
  source_version: "992811"
  as_of: 2026-08-31T10:10:00Z
```

These fields are not universal formulas. Some platforms distinguish soft allocation, reservation, physical reservation, available-to-promise, or virtual pools differently. The adapter must publish the platform semantics and the policy must identify which value supports each effect. A derived `on_hand - reserved` calculation cannot replace the source's availability rules.

## Time semantics

Every time has a kind, timezone, and precision:

```yaml
milestone_time:
  kind: estimated
  milestone: destination_arrival
  value: 2026-09-02T14:00:00+02:00
  timezone_source: facility_master
  precision: hour
  as_of: 2026-08-31T10:00:00Z
  source_ref: forecast_eta_721
```

Do not collapse:

- requested time;
- promised or committed time;
- appointment window;
- planned time;
- estimated time;
- observed event time;
- source record/capture time;
- ingestion and projection time.

Daylight-saving changes, local cut-offs, working calendars, holidays, port/terminal calendars, and time-window inclusivity belong in deterministic calendar services. Convert for computation, but retain original offsets and source timezone.

## Projection contract

A projection is a versioned answer to a defined question. Example:

```yaml
shipment_operational_view:
  projection_id: shipment_view/shp_774
  projection_version: 84721
  built_at: 2026-08-31T10:15:00Z
  rule_version: shipment_projection/v8
  source_watermarks:
    tms_prod: "228184"
    carrier_dhl_gf: "evt_88921"
  identity_status: resolved
  current_leg: leg_774_2
  latest_observed_milestone: origin_departure
  planned_next_milestone: destination_arrival
  eta_forecast_ref: forecast_eta_721
  freshness:
    carrier_status_age_seconds: 331
    policy_status: fresh
  conflicts: []
  evidence_refs: [obs_102, obs_117, obs_130]
```

The projection builder is deterministic for a rule version. Rebuild and compare it during releases. Store enough provenance to explain why a field has its value without retaining sensitive source payloads in every read model.

### Source-of-truth matrix

Define a matrix per operating unit, not globally:

| Field | Primary authority | Corroborating source | Conflict behavior |
|---|---|---|---|
| Customer promise | OMS | ERP order confirmation | Block promise-changing effects; page order owner |
| Current physical inventory | WMS | ERP inventory view | WMS for reservation if charter says so; expose replication lag |
| Shipment plan and tender | TMS | Carrier booking response | TMS until reconciliation rule proves carrier acceptance mismatch |
| Carrier operational milestone | Carrier/forwarder | TMS event copy, EPCIS event | Preserve carrier declaration and transport-system receipt separately |
| Warehouse dispatch | WMS or EPCIS capture owner | Carrier pickup event | Do not infer dispatch from pickup alone without declared rule |
| Dangerous-goods classification | Governed product/compliance master | Shipping document | Conflict blocks affected mode/route |

"Most recent timestamp wins" is not a valid general conflict policy.

Resolve a conflict in this order: validate scope and identity; compare source-specific revision semantics; apply the field-level authority and effective-time rule; test freshness and correction status; retain corroborating or dissenting observations; then emit `resolved`, `provisional`, or `blocked` with the rule version and evidence. Recency is used only inside a source contract that defines it. A carrier milestone cannot overwrite a WMS dispatch quantity, and a replicated ERP balance cannot silently overrule the WMS named for reservation. Material conflicts invalidate dependent forecasts, solver inputs, proposals, approvals, and undispatched intents; committed or unknown effects move to reconciliation rather than being forgotten.

## Exception aggregate contract

The workflow state references evidence instead of copying it:

```yaml
exception:
  exception_id: exc_01J...
  type: delivery_promise_at_risk
  charter_version: delivery_promise_at_risk/v1
  scope: {tenant_id: tenant_acme, legal_entity_id: in01}
  resources:
    order_lines: [ord_882/10]
    shipments: [shp_774]
  state: awaiting_approval
  state_version: 12
  severity: high
  detected_at: 2026-08-31T09:58:10Z
  decision_deadline: 2026-08-31T11:00:00Z
  owner_role: logistics_exception_manager
  snapshot_ref: snapshot_84721
  evidence_refs: [obs_130, forecast_eta_721, solve_419]
  active_proposal_id: prop_66
  active_approval_id: null
  open_effect_ids: []
  replan_count: 1
```

Transitions use optimistic concurrency on `state_version`. Timers and callbacks include the expected version so stale work becomes a no-op rather than resurrecting a closed or replanned exception.

## Proposal and approval records

```yaml
proposal:
  proposal_id: prop_66
  exception_id: exc_01J...
  snapshot_ref: snapshot_84721
  snapshot_hash: sha256:...
  action_type: carrier_service_upgrade
  target: {shipment_id: shp_774, leg_id: leg_774_2}
  parameters: {service_level: EXPRESS_12, maximum_cost: {value: "1800.00", currency: INR}}
  constraints_version: parcel_recovery/v12
  feasibility:
    solver_run_id: solve_419
    status: feasible
    optimality_proven: false
  predicted_outcomes:
    arrival_time_quantiles_ref: forecast_scenario_912
  evidence_refs: [obs_130, quote_771, forecast_eta_721]
  created_by: model_proposer/release_2026_08_31
  expires_at: 2026-08-31T10:35:00Z
```

An approval binds `proposal_id`, `snapshot_hash`, action, target, limits, policy version, approver identity/role, reason, and expiry. If a bound source version, quantity, carrier, service, cost, route, regulatory status, or hard constraint changes materially, the approval is invalidated. The model cannot revise an approved object; it creates a new proposal.

## Effect record and unknown state

```yaml
effect:
  effect_id: eff_01J...
  semantic_operation_id: tenant_acme:upgrade:shp_774:leg_2:decision_66
  intent_hash: sha256:...
  action_type: carrier_service_upgrade
  target_system: tms_prod
  expected_resource_version: "shipment-118"
  approval_id: appr_901
  status: effect_unknown
  attempts:
    - attempt: 1
      started_at: 2026-08-31T10:22:01Z
      transport_result: timeout_after_send
      connector_request_id: req_729
  expected_postconditions:
    - path: shipment.leg[2].service_level
      equals: EXPRESS_12
  reconciliation:
    next_at: 2026-08-31T10:22:31Z
    attempts_remaining: 5
```

`effect_unknown` is a normal distributed-systems state. It is neither success nor failure. It blocks a duplicate intent and schedules targeted read-back by semantic operation ID, downstream client reference, or resource state.

## Retention and privacy by state class

| Record | Typical retention principle | Privacy rule |
|---|---|---|
| Raw payload artifact | Shortest period needed for dispute/audit | Encrypt; strict role access; legal hold separately |
| Normalized observation | Business/audit requirement | Minimize personal and commercial text; tokenize identities where possible |
| Projection | Rebuildable and short-lived | Store only operationally necessary fields |
| Workflow/effect ledger | Long enough for audit, incident, and reconciliation | Immutable access trail; field-level classification |
| Prompt/response | Sampled or short-lived unless an approved evidence artifact | Redact or reference; do not log unrestricted documents by default |
| Evaluation episode | Curated, de-identified, reviewed | Separate dataset governance and deletion propagation |

Deletion or correction requests must propagate by subject and source artifact while preserving any legally required minimal audit record. Never copy personal contact data into free-form model memory.

## State validation checklist

- [ ] Every identifier has an entity type, namespace, scope, and resolution status.
- [ ] Split, merge, pack, repack, substitute, leg, and handoff relationships are modeled explicitly.
- [ ] Decimal quantities, units, segments, and conversion provenance are mandatory.
- [ ] Event, record, ingest, projection, plan, estimate, promise, and observation times are distinct.
- [ ] Corrections append and link; history is never silently overwritten.
- [ ] Conflicts are resolved by a field authority matrix or exposed, not model judgment.
- [ ] Projections are rebuildable for a rule version and retain evidence lineage.
- [ ] Workflow transitions use state-version concurrency and stale callbacks no-op.
- [ ] Proposals and approvals bind snapshot hashes, limits, policy, and expiry.
- [ ] Effect unknown is representable, blocks duplicates, and triggers reconciliation.
- [ ] Telemetry cannot substitute for effect or evidence records.

Continue with [uncertainty, constraints, planning, and context](04-uncertainty-constraints-planning-and-context.md). For the generic event envelope and replay rules, see [agent state and event contracts](../../runtime/agent-state-and-event-contracts.md).
