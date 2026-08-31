# Workload Fit, Authority, and Stage 0–6 Gates

> **Purpose:** Qualify the workload, establish the human and deterministic authority boundary, and define an evidence-gated path from ordinary software to governed production.  
> **Section index:** [Fraud and AML investigation agent](README.md)

## Start with the outcome, not “AI”

Name one measurable investigator outcome before selecting a model:

- reduce time to assemble a complete case snapshot for a specific alert family;
- improve the proportion of material claims with valid evidence lineage;
- surface connected entities or transactions that current tooling makes hard to retrieve;
- improve consistency of first-pass hypotheses while preserving investigator discretion;
- reduce avoidable alerts without reducing sensitivity on agreed severe-risk slices;
- prepare a technically valid, cited filing package after—not before—a qualified person decides to file.

“Automate AML,” “reduce headcount,” or “find criminals” is not a valid task contract. It hides the legal threshold, source coverage, error costs, and decision owner.

## Stage 0 decision record

Compare at least four options on the same time-bounded, permission-clean case set.

| Option | What it does well | Main limit | Promote only if |
|---|---|---|---|
| Rules and deterministic workflow | Repeatable thresholds, routing, deadlines, schema checks, known typologies | Brittle across heterogeneous narratives and novel combinations | It remains the baseline and owns every invariant it can express |
| Search, dashboards, and graph UI | Gives investigators direct evidence access without model error | Manual query and synthesis burden | Training/UX cannot meet the target alone |
| Single-call extraction or summarization | Produces one typed artifact with low orchestration risk | Cannot adaptively retrieve missing evidence | A loop adds no measured value |
| Bounded read-only loop | Chooses among narrow queries and revises hypotheses | Stochastic path, greater security/eval burden | It improves the named outcome with acceptable missed-risk and false-positive cost |
| Multi-agent system | Can isolate or parallelize independent domains | Coordination, duplicated access, hidden disagreement, cost | A controlled trial shows benefit over deterministic fan-out and one investigator |

Use a blind paired review where possible. Compare investigator time, evidence coverage, unsupported claims, correction rate, escalation rate, severe false-negative review, subgroup burden, latency, and total cost. Do not promote because summaries “look good.”

## Authority and accountability matrix

| Record or decision | Model | Case workflow | Investigator / designated officer | Legal | Independent compliance/audit |
|---|---|---|---|---|---|
| Source fact and lineage | May cite; cannot alter | Validates and stores | Challenges or accepts relevance | Advises on restrictions | Samples completeness and control operation |
| Entity resolution | Proposes candidates and conflicts | Applies deterministic keys/thresholds | Resolves material ambiguity | Advises where legal identity matters | Tests methodology independently |
| Hypothesis / typology | Proposes with alternatives | Stores versioned proposal | Owns investigative judgment | Advises on legal characterization | Tests process, not live case decision |
| Alert disposition / case escalation | Recommends | Enforces eligible transitions | Owns according to policy | Consulted as required | Does not co-own disposition |
| SAR/STR decision and narrative | Drafts a cited package | Enforces schema, deadline, and segregation | Designated human owns decision/attestation | Owns legal advice and special escalation | Independently tests filing control |
| Hold/block/freeze/reject/offboard | Impact-aware proposal only | Performs current-policy and approval gate | Separate authorized owner decides | Advises/approves where required | Tests control after the fact |
| Monitoring policy or behavior release | No authority | Enforces release manifest and rollout | Product/operations sign off | Reviews legal impact | Independently challenges and validates |

No confidence score, “high risk” label, or prior investigator disposition can substitute for the named accountable role.

## Workload and autonomy profile

Before Stage 1, record:

| Dimension | Required answer |
|---|---|
| Population | Products, rails, customer/entity types, countries, languages, and alert families included |
| Time | Interactive review, batch enrichment, maximum case duration, evidence and filing deadlines |
| Load | Alerts/day, bursts, graph query fan-out, document volume, concurrent investigators, approval capacity |
| Error costs | Missed suspicious activity, needless escalation, customer harm, delayed payments, confidentiality breach, and reviewer overload |
| Evidence | Authoritative sources, coverage gaps, update cadence, lineage, and reconstruction requirement |
| Authority | Autonomous ceiling, forbidden actions, decision owners, segregation of duties, and approval invalidation |
| Data | Purpose, legal basis, tenancy, residency, sensitive classes, retention, deletion, legal hold, and allowed providers |
| Recovery | Crash boundaries, duplicate intake, stale case state, cancellation, unknown effects, and deadline breach |

## Stage 0–6 gates

### Stage 0 — qualify the problem

Build the deterministic baseline first: source ingestion, case state, entity keys, graph/query views, policy deadlines, and investigator UI. Create a representative evaluation set before a prompt.

Exit only when:

- the target alert family, jurisdictions, and outcome are explicit;
- rules/search/single-call alternatives were measured;
- a qualified owner accepts the label limitations and error-cost model;
- the proposed model step has no external authority;
- privacy, SAR/STR confidentiality, and source rights permit the planned processing.

### Stage 1 — first bounded investigator

Use one curated alert envelope and a fixed evidence bundle or at most a few typed read tools. Require cited claims, one plausible benign alternative, explicit missing evidence, a stop reason, and hard turn/tool/token/time budgets.

Exit only when:

- every output validates against a closed schema;
- fabricated locators and unsupported allegations fail the trial;
- the loop cannot access generic search, SQL, customer lookup, filing, or restriction tools;
- repeat trials meet minimum evidence and abstention gates;
- investigator review remains mandatory.

### Stage 2 — useful MVP

Integrate the real case environment with purpose-bound reads, context assembly, run-scoped working state, versioned typology material, and draft case artifacts. Add normal, benign, suspicious, ambiguous, stale, incomplete, and hostile-content cases.

Exit only when:

- tenant and field filters are enforced before retrieval/ranking;
- source facts, derived features, model proposals, and human decisions are visually and structurally distinct;
- case-time and source-time semantics are correct;
- investigators can correct, reject, or request evidence without rewriting history;
- operational metrics include review time and queue age, not just model latency.

### Stage 3 — reliable v1

Add durable run/checkpoint state only for real waits or recovery. Version every connector, parser, graph build, context compiler, prompt, model, policy, and schema. Add compaction, cancellation, fencing, effect identity for draft/internal writes, and reconciliation.

Exit only when:

- process death after every durable boundary neither loses a case transition nor repeats an effect;
- compacted context preserves case goal, approved scope, IDs, evidence references, contradictions, unresolved gaps, deadlines, and next action;
- unknown external outcomes cannot be treated as failed;
- active cases have a pin/migrate/quarantine policy for every behavior component;
- third-party failures, pagination gaps, schema drift, revoked access, and stale sources route safely.

### Stage 4 — production readiness

Add workload identity, user delegation, tenancy, least privilege, egress control, credential brokerage, redacted tracing, SLOs, immutable behavior manifests, security and privacy review, shadow/canary/rollback, and incident runbooks.

Exit only when:

- unauthorized disclosure or D3/D4 effect is a non-compensating release failure;
- independent validation covers detection/ranking components and the agent workflow;
- audit records reconstruct decisions without hidden chain-of-thought or sampled telemetry;
- kill, revoke, quarantine, propose-only, and last-known-good rollback drills pass;
- production can operate safely during model/provider/trace/evaluator outages.

### Stage 5 — scale and resilience

Partition admission, retrieval, graph computation, model work, review, and effects. Bound queues by age and case priority, reserve capacity for urgent legal deadlines, and model the investigator/approver bottleneck.

Exit only when:

- overload degrades to deterministic routing or evidence-only bundles, never weaker authority checks;
- one tenant, typology burst, connector, or huge network cannot starve others;
- cell/region failure, backlog redrive, source rebuild, and disaster recovery are tested;
- cost is measured per verified investigator outcome and severe-risk slice;
- fairness and false-positive burden remain acceptable under load shedding and prioritization.

### Stage 6 — continuous governed evolution

Mine corrected cases, abstentions, novel patterns, incidents, overrides, unknown effects, missed deadlines, and complaints into a governed evaluation backlog. Treat model, prompt, typology, rule, list, source mapping, entity-resolution logic, connector, schema, and policy changes as behavior releases.

Exit is continuous evidence, not a one-time launch:

- release and rollback decisions use critical slices plus repeated trials;
- investigator feedback is reviewed for label leakage, bias, retaliation, and confirmation bias before reuse;
- drift monitoring separates population, source, rule, model, investigator, and outcome changes;
- independent compliance/audit retains its own sampling and testing path;
- deprecated behavior, schemas, and active-case migrations have owners and deadlines.

## Stage exercises and exit evidence

A checklist item passes only with reproducible evidence. Re-enter an earlier stage when intended use, authority,
jurisdiction, tenant, alert family, source, list, rule/model, adapter, memory or effect scope changes materially.

| Stage | Required exercise | Measurable gate | Evidence retained |
|---:|---|---|---|
| 0 | Compare rules/search/workflow, single-call synthesis and a bounded loop on the same time-correct cases | The model improves a named investigator outcome and error-cost profile; deterministic software still owns exact rules, identity, clocks and authority | Workflow map, deterministic baseline, intended/prohibited use, label limitations, risk/burden assessment and manual owner |
| 1 | Attack one read-only alert-family loop with cross-tenant IDs, prompt injection, missing/partial data, stale versions and budgets | Zero unauthorized reads/effects; every claim cites an allowed revision; every ambiguity stops in its expected state | Signed task/tool/output contracts, invariant traces, rejected-input corpus, baseline comparison and reviewer rubric |
| 2 | Run rapid-movement, entity, sanctions and benign cases in a representative sandbox with blinded reviewers | Severe-error threshold is zero; evidence fidelity, abstention, benign-alternative quality, reviewer time and calibrated agreement meet approved targets | Versioned cases/cutoffs, expected-outcome lineage, reviewer calibration, slice/burden results and unresolved-risk disposition |
| 3 | Crash/restart around checkpoints and writes; compact repeatedly; reorder/correct events; cancel; lose a response and reconcile | No duplicate semantic effect; every `UNKNOWN` is reconciled/owned; clocks, evidence, contradictions, versions and next safe action survive | State/event/effect export, continuity receipts, adapter dossiers, correction propagation, audit reconstruction and recovery report |
| 4 | Rehearse proposal-only shadow, constrained canary, revocation, confidentiality incident, kill switch, manual path and rollback | No shadow mutation or tipping-off; independent stops and evidence preservation meet objectives; no severe policy/effect failure | Validation dossier, signed whole-behavior manifest/diff, approvals/training, runbooks, incident exercise and independent challenge |
| 5 | Load all priority tiers and provider limits; fail a tenant cell/region; restore with source/list/effect/reviewer backlog | Critical deadline capacity and fairness hold; no cross-scope leakage; RPO/RTO, reconciliation and recovery-drain objectives pass with headroom | Capacity/cost model, fairness/starvation tests, isolation evidence, DR/recovery-load report and operating limits |
| 6 | Change a list/parser, rule/model, graph/entity logic, policy, provider API and context compiler; detect and roll back each | Semantic diff finds affected cases/populations; gates catch seeded regressions; rollback preserves records and never replays unknown effects | Change record, refresh decision, controlled failure fixtures, drift/reviewer report, rollout/rollback results and migration/deprecation plan |

## Promotion blockers

Do not move forward while any of these remain true:

- “filed SAR,” “confirmed fraud,” “chargeback,” or case disposition is used as unqualified ground truth;
- the product cannot calculate false-positive burden and missed-risk review by meaningful slice;
- the model sees broader data than the investigator is authorized to see;
- sanctions, PEP, adverse-media, and internal watchlist hits are collapsed into one risk flag;
- active cases cannot identify which sources, lists, rules, prompts, and models produced their artifacts;
- review capacity is assumed infinite;
- the team cannot stop effects without asking the agent to cooperate.

## Checklist

- [ ] A deterministic alternative is implemented and measured.
- [ ] Scope and out-of-scope adjacent owners are signed off.
- [ ] Every decision and effect has one accountable owner.
- [ ] The autonomous ceiling is D1 reads and D2 draft artifacts.
- [ ] Stage gates use evidence, not feature count.
- [ ] Human capacity and both sides of error cost are modeled.
- [ ] Later maturity cannot silently broaden authority.

## Sources and next guide

- [FATF — risk-based approach and financial inclusion changes](https://www.fatf-gafi.org/en/publications/Fatfrecommendations/update-standards-promote-financial-conclusion-feb-2025.html)
- [FFIEC BSA/AML Manual — Suspicious Activity Reporting](https://bsaaml.ffiec.gov/manual/AssessingComplianceWithBSARegulatoryRequirements/04)
- [FDIC and U.S. agencies — risk-based customer relationships and CDD](https://www.fdic.gov/news/financial-institution-letters/2022/fil22028.html)
- [Anthropic — Building effective agents](https://www.anthropic.com/engineering/building-effective-agents)

Next: [Reference architecture and integration contracts](02-reference-architecture-and-integration-contracts.md).
