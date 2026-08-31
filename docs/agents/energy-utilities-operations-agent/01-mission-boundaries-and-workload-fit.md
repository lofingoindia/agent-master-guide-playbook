# Mission, Boundaries, and Workload Fit

> **Last reviewed:** 2026-08-31  
> **Purpose:** turn a broad utility-operations ambition into one testable operating contract with an explicit safety and authority boundary.

The first implementation task is not selecting a model. It is proving that a bounded utility case has identifiable assets and customers, authoritative sources, known time and quality semantics, qualified owners, a manual baseline, and measurable completion.

## Define the operating case

A suitable first case is narrow enough to replay and independently verify. Examples:

- detect a sustained electric feeder outage from SCADA plus AMI and customer evidence, then produce a cited operator briefing;
- reconcile a planned electric interruption across GIS, OMS, CIS, WMS, and notification status;
- correlate a gas pressure/alarm anomaly with current topology, controller notes, and field observations, then hand it to the qualified pipeline controller;
- assemble a water main-break service-impact picture with affected pressure zones, critical customers, crew status, and approved public-message facts;
- reconcile restoration after field work, identifying customers, devices, meters, or reports that still disagree;
- prepare the evidence package for a post-event reliability or emergency report without filing it.

Avoid first cases involving protection, live switching, valve actuation, dispatch, treatment changes, emergency load shed, contamination classification, public-safety release, or a large multi-utility storm. Start read-only in one service territory.

### Candidate scorecard

Score `0` to `2`; do not add model-generated proposals below `20/28`.

| Dimension | 0 | 1 | 2 |
|---|---|---|---|
| Canonical identity | Free text/collisions | Mapping with gaps | Stable IDs plus exception workflow |
| Topology | Unknown/stale | Trace available, weak versioning | Versioned model, validation, freshness and correction |
| Operational state | Single opaque status | Several sources | Source/field authority and quality semantics approved |
| Event semantics | Mutable latest row | Some timestamps | Event/device/ingest time, sequence, correction, gap rules |
| Detection oracle | Anecdotal | Operator review only | Deterministic rule plus labeled replay and coverage |
| Safety boundary | Prompt instruction | Tool allowlist | Physical/network/identity isolation from control |
| Qualified ownership | Informal | Role identified | Named on-duty role and escalation path |
| Constraints | Tacit | Checklist | Machine-checked hard constraints and versioned policy |
| Proposal space | Open ended | Runbook options | Typed, finite catalog with infeasible/unknown states |
| External coordination | Generic write | Narrow draft | Exact artifact, approval, idempotency and read-back |
| Reconciliation | Manual spot check | Periodic compare | Independent postconditions and ageing SLO |
| Simulation | None | Historical examples | Versioned simulator/digital twin with applicability record |
| Evaluation | Demo | Small case set | Multi-trial replay, failures, trajectory and outcome oracles |
| Operations | No owner | On-call exists | SLO, capacity, kill, manual mode, incident and DR drills |

## Authority classes

Classify every capability; enforce the class outside prompt text.

| Class | Meaning | Examples | Category policy |
|---|---|---|---|
| U0 | Offline computation | Replay, simulation, document comparison | Allowed in isolated test environments |
| U1 | Scoped operational read | Topology snapshot, telemetry query, outage case, crew status | Default production ceiling for tools |
| U2 | Isolated draft | Operator brief, work proposal, customer/regulator message draft | Allowed with provenance and review queue |
| U3 | Non-control coordination effect | Create/update a case note, submit an approved work artifact, place a message in a release queue | Only at Stage 5 through a narrow gateway and exact approval |
| U4 | Utility control or safety effect | SCADA command, breaker/valve/pump operation, switching execution, setpoint, dispatch, load shed, clearance, interlock | Permanently prohibited |
| U5 | Governance decision | Change autonomy, policy, topology authority, regulatory position, qualification, emergency declaration | Human/governance owned; agent may draft analysis only |

No amount of U3 evidence promotes an operation to U4. “Human approved” is not a valid reason to route a control command through the model runtime; the human uses the qualified existing control channel.

Every deployed operation also declares one runtime authority tier. `READ` returns attributed data without mutation. `PROPOSE` creates an isolated, immutable candidate with no downstream visibility outside its review queue. `EXECUTE_COORDINATION` may commit only a pre-approved U3 artifact through its dedicated gateway. `EXECUTE_CONTROL` covers commands, setpoints, switching, dispatch, curtailment/load shed, process actuation, alarm acknowledgement/configuration, interlocks and safety/clearance actions and is permanently absent from the agent catalog. The manifest validator rejects an operation when its transport, product permission or downstream behavior is broader than its declared tier.

| Tier | Who owns the decision | Permitted agent behavior | Mandatory enforcement |
|---|---|---|---|
| `READ` | Source owner defines meaning and access | Query a named resource/snapshot and return quality, coverage and provenance | Read replica/broker where practical; server-side method denial; scoped identity; rate and query-cost limits |
| `PROPOSE` | Qualified operator/engineer/dispatcher/release owner | Produce a sealed option, draft or missing-evidence request | No target-system mutation; typed scenario catalog; deterministic constraint checks; explicit expiry |
| `EXECUTE_COORDINATION` | Human approver plus target-system workflow owner | Post the exact approved U3 note/request/draft and reconcile it | Digest-bound approval, resource/version precondition, semantic idempotency, fencing, read-back, cancellation and kill |
| `EXECUTE_CONTROL` | Qualified operator/controller/authorized employee through established control/safety procedure | None | No endpoint, route, credential, method, schema or generic proxy capable of the action; negative tests and architecture review |

Emergency operations do not change this ceiling. If normal IT/data-plane services fail, the agent narrows to a cited read-only packet or disappears; deterministic alarms, protection/interlocks, operator HMIs, field radio, emergency plans and manual procedures continue independently.

## Commodity-specific non-delegable decisions

| Electric | Gas | Water and wastewater |
|---|---|---|
| Energize/de-energize; execute switching; grant/transfer clearance; change protection; redispatch; curtail or shed load; start blackstart/restoration steps; declare safe working condition | Move a valve; start/stop compressor; alter pressure control; isolate/blow down/purge; classify abnormal or emergency condition; authorize covered task; declare leak safe | Start/stop pumps; move valves; alter treatment/dose/setpoint; bypass process; declare contamination; issue/rescind boil-water or do-not-use advisory; approve discharge or public-health status |

The agent may cite the evidence considered by qualified owners. It cannot assume their legal, regulatory, engineering, or public-health responsibility.

## Boundary with neighboring playbooks

| Trigger | Utility-operations agent response | Owner |
|---|---|---|
| SCADA/OMS software latency or database failure | Mark evidence degraded; use manual mode; attach diagnostic facts | SRE/platform team |
| Control-center WAN or substation communications failure | Record coverage impact and operational handoff | Network operations/telecom and utility operator |
| Plant equipment/process alarm | Consume operator-published availability or hand off | Plant/manufacturing/process operations |
| Generic crew scheduling conflict | Add utility-specific safety/topology/priority evidence | Field service/dispatch owner |
| Suspected cyber manipulation | Preserve raw evidence, reduce authority, notify incident path | OT security and incident command |
| Customer billing or account dispute | Attach outage/service evidence only | Customer care/billing |
| Regulatory interpretation | Draft questions and evidence; freeze affected publication | Legal/regulatory owner |

Typed handoffs include the exact utility, territory, case, assets/customers, evidence references, quality, urgency, requested decision, and prohibited assumptions. They never transfer credentials.

## Jurisdiction and operator pack

Standards and laws are not global defaults. Bind every operating unit to an approved pack:

```yaml
jurisdiction_pack:
  pack_id: us_example_electric_dist_2026_08
  utility_id: util_north_01
  service: electric_distribution
  territory: state_example
  effective_at: 2026-08-15T00:00:00Z
  sources:
    - authority: state_public_utility_commission
      instrument: outage_reporting_rule
      version: "effective-2026-07-01"
    - authority: OSHA
      instrument: 29_CFR_1910_269
      as_of: 2026-08-31
    - authority: NERC
      instrument: applicable_standards_matrix
      version: "utility-registration-specific"
  operator_procedures:
    outage_classification: OP-OUT-014/rev12
    storm_mode: OP-EM-002/rev9
    switching: OP-SW-001/rev31
    customer_updates: OP-COM-008/rev7
  qualified_roles:
    decision: distribution_system_operator
    switching: switching_authority
    public_release: incident_public_information_officer
  critical_load_policy: CL-2026-04
  data_retention: RET-OPS-2026-03
  signature: sigstore:...
```

The pack loader rejects expired, unsigned, overlapping, or mismatched packs. A regulation or operator procedure is never retrieved opportunistically from the public internet during an incident. Proposed updates are reviewed offline and released as signed data.

### Illustrative applicability differences

- NERC reliability standards apply by registered function, facility, jurisdiction, standard version, and effective date; distribution systems are not universally covered by every bulk-power requirement.
- U.S. PHMSA control-room, emergency, operator-qualification, and integrity rules apply by pipeline type and regulated activity; state programs and operator procedures add constraints.
- U.S. AWIA/SDWA risk and emergency-plan duties have system-size and drinking-water applicability; wastewater and local public-health duties differ.
- OSHA electric-power requirements distinguish qualified employees, generation versus transmission/distribution work, and de-energization/grounding processes.
- EU system-operation duties distinguish TSO, DSO, significant grid user, and regional coordination roles.
- India's CERC Grid Code assigns specific functions to NLDC, RLDCs, SLDCs, licensees, and connected entities and has been amended since its 2023 effective date.

The blueprint therefore supplies control patterns, not a universal compliance answer.

## Critical-load and customer constraints

Critical-load status is sensitive and often contested. Never infer it from a facility name or an old outage ticket.

```yaml
critical_service_record:
  utility_id: util_north_01
  service_point_id: sp_7731
  classification: health_and_medical
  authority: critical_load_registry
  policy_id: CL-2026-04
  effective_from: 2026-04-01T00:00:00Z
  expires_at: 2027-04-01T00:00:00Z
  validation_status: VERIFIED
  contact_access_class: restricted_ops
  restoration_priority_effect: advisory_input_only
```

Priority is a constraint input, not a promise of restoration order. Physical feasibility, crew safety, system stability, public safety, mutual dependencies, and incident command still govern. Communications must not expose a critical facility, medical dependency, vulnerable customer, or security-sensitive asset beyond approved need-to-know.

## Stop and escalation rules

The runtime stops analysis or lowers output authority when any hard condition is true:

| Condition | Required response |
|---|---|
| Utility, asset, meter, service point, circuit/zone, or case identity unresolved | Quarantine evidence; ask the named data steward/operator |
| Topology version missing, invalid, dirty beyond policy, or incompatible with operational state | No affected-customer count or plan; present conflict |
| Clock skew, sequence reset, quality flag, or coverage gap invalidates correlation | Mark interval unknown; do not infer occurrence/absence |
| Alarm flood or telemetry loss exceeds validated envelope | Switch to alarm-flood/degraded runbook; deterministic prioritization only |
| Safety, clearance, isolation, contamination, or emergency classification is requested | Stop and hand to qualified authority |
| Forecast or simulation is outside applicability or non-convergent | Exclude it; present manual alternatives |
| Critical-load registry unavailable or stale | Do not make a priority claim; escalate |
| Operator procedure or jurisdiction pack unavailable/ambiguous | No proposal requiring it |
| Any coordination effect has unknown outcome | Freeze same-resource effects; reconcile |
| Prompt injection or untrusted document requests tool/authority changes | Ignore instruction; preserve artifact and alert security |
| On-duty qualified owner cannot be established | Read-only evidence packet; no approval routing |

Stopping is a successful safety outcome, not an availability failure.

## Measurable baseline and value

Record the manual/deterministic baseline before deploying a model:

- time from first attributable evidence to case creation;
- time to a topology- and quality-checked affected-customer estimate;
- operator minutes spent gathering evidence;
- duplicate, merged, split, and false outage-case rates;
- restoration confirmation lag and “nested outage” miss rate;
- communication correction/retraction rate;
- unreconciled evidence and coordination effect age;
- operator workload during steady state and storm peaks;
- safety, privacy, security, and regulatory escapes;
- utility-defined reliability and service measures, reported with method and inclusion rules.

The agent must beat a rule/template/search baseline on operator time or decision quality without increasing false certainty, missed constraints, queue age, or operational risk. Business metrics such as restoration duration require causal caution; weather severity, damage, access, crew availability, and system design dominate many outcomes.

## Stage 0 exit gate

Do not proceed until all answers are “yes”:

- [ ] One utility, commodity, service territory, control center, case type, and owner are named.
- [ ] The no-agent baseline has been measured.
- [ ] Field-level sources of truth, time, quality, coverage, and conflict rules are approved.
- [ ] The agent is network-, identity-, and tool-isolated from U4 endpoints.
- [ ] Qualified roles and on-duty resolution are deterministic.
- [ ] The jurisdiction/operator pack is signed, versioned, and testable.
- [ ] Critical-load and sensitive-customer handling is approved.
- [ ] A historical replay set and independent completion oracle exist.
- [ ] Stop, manual, storm, security, and emergency paths are rehearsed.
- [ ] Operations, safety, OT security, field, customer, regulatory/legal, data, and platform owners accept the boundary.

## Primary evidence

- [NERC Reliability Standards and jurisdiction/effective-date views](https://www.nerc.com/standards/reliability-standards)
- [OSHA 29 CFR 1910.269, electric power generation, transmission, and distribution](https://www.osha.gov/laws-regs/regulations/standardnumber/1910/1910.269)
- [PHMSA control room management](https://www.phmsa.dot.gov/pipeline/control-room-management/control-room-management)
- [PHMSA operator qualification](https://www.phmsa.dot.gov/pipeline/operator-qualifications/about-operator-qualification)
- [EPA emergency-response plans for water utilities](https://www.epa.gov/waterutilityresponse/develop-or-update-emergency-response-plan)
- [EU Commission Regulation 2017/1485 on electricity system operation](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=celex%3A32017R1485)
- [CERC Indian Electricity Grid Code Regulations, 2023](https://www.cercind.gov.in/Regulations/180-Regulations.pdf)

Continue with [reference architecture and staged delivery](02-reference-architecture-and-staged-delivery.md).
