# Customer, Transaction, Entity, and Network Evidence

> **Purpose:** Build a reproducible evidence model for parties, accounts, transactions, ownership, devices, and networks without turning probabilistic linkage into fact.  
> **Section index:** [Fraud and AML investigation agent](README.md)

## Evidence is layered, temporal, and contestable

An investigation must preserve five different things:

1. **Source fact:** what a named system or document reported at a particular revision.
2. **Normalized fact:** a deterministic representation with the transform and original retained.
3. **Derived fact:** a reproducible computation from cited inputs, such as a 30-day total or ownership roll-up.
4. **Identity or relationship candidate:** a fallible linkage with method, score, alternatives, and review state.
5. **Hypothesis:** an investigator or agent interpretation that may be supported, contradicted, unresolved, or rejected.

Do not flatten these into a “customer profile.” A profile assembled from mutable data hides source conflict, valid time, uncertainty, and corrections.

## Canonical identifiers

| Object | Identity requirement | Common collision to prevent |
|---|---|---|
| Party | Institution-scoped stable party ID; party type explicit | Person and legal entity merged because names match |
| Account/product | Stable account/product instance, owner relationship, lifecycle | Reused display/account number treated as same product forever |
| Transaction | Rail/system semantic ID plus revision/status lineage | Authorization, clearing, settlement, reversal, and chargeback counted as separate value transfers |
| Payment message | Message ID and original/amendment/cancellation chain | Multiple intermediaries or replays treated as distinct customer payments |
| External entity | Source namespace + source entry ID + revision | LEI, registry number, tax ID, and watchlist ID treated as interchangeable |
| Device/session | Provider namespace, collection method, time, confidence | Shared/NAT/recycled identifier treated as a person |
| Address/contact | Normalized value plus source and validity | Household, branch, registered agent, or virtual office treated as ownership |
| Rule/typology | Stable definition ID plus immutable release/digest, effective interval, input mapping, thresholds and owner | Display name or current threshold treated as the historical rule |
| Model/inference | Model artifact/version/digest, feature/calibration release, deployment release and inference ID | Mutable registry alias or vendor score treated as a reproducible model decision |
| Watchlist dataset/entry | Authority, list, program/regime, publication/full-or-delta revision, stable entry ID and entry revision | Name/alias or aggregator ID treated as the designated party or current official list state |
| Alert | Stable producer alert ID plus semantic rule/model/subject/window/revision key | Retuned rule or replay opens unrelated duplicate alerts |
| Case | Institution-scoped case ID, generation, version and linked/superseded cases | Alert ID, filing ID or customer ID reused as case identity |
| Evidence object | Content hash + source object ID/revision + acquisition | Changed object silently overwrites the cited version |
| Human decision | Decision ID, generation/version, decision type, review-bundle hash, policy/jurisdiction release, actor/role and effective time | Case disposition or filing outcome treated as timeless ground truth |
| External effect | Server-derived semantic effect ID, immutable request/payload hash, destination, attempt, receipt and reconciliation revision | Transport request ID or HTTP success treated as the business effect |

Identifiers are evidence of linkage, not identity by themselves. Every cross-system join records which identifiers
were used, their namespaces and versions, the deterministic transforms applied, the effective and observation times,
and the authority that accepted the mapping. Fuzzy similarity may create a candidate; it never repairs a missing party,
account, list-entry, case, decision or effect key at commit time.

## Temporal model

At minimum, preserve:

| Time | Meaning | Example use |
|---|---|---|
| `event_time` | When the described activity occurred | Transfer initiated or login observed |
| `effective_from/to` | When a relationship or attribute was valid | Director or beneficial owner tenure |
| `posting_time` | When a financial system recorded value | Ledger-window reconciliation |
| `source_updated_at` | When the source revised the record | KYC refresh or sanctions entry update |
| `observed_at` | When the platform retrieved it | Reproducibility and staleness |
| `case_cutoff_at` | Evidence horizon for a decision | Prevent post-decision facts leaking into evaluation |

Use interval semantics for relationships. Never infer that current ownership, address, employment, or account status existed at the transaction time. For cross-source ordering, use stable event IDs and source sequence where available; timestamps alone may be delayed, rounded, or corrected.

## Evidence object contract

~~~json
{
  "evidence_id": "ev_01...",
  "case_id": "case_01...",
  "source": {
    "system": "payments-ledger",
    "object_id": "txn_...",
    "revision": "7",
    "schema": "transfer.v3",
    "event_time": "...",
    "observed_at": "..."
  },
  "content_hash": "sha256:...",
  "classification": ["customer-confidential", "financial"],
  "purpose": "aml_case_investigation",
  "jurisdiction": ["..."],
  "coverage": {"window": ["...", "..."], "complete": true},
  "transform": {"id": "normalize-transfer@sha256:...", "input_refs": ["raw_..."]},
  "quality": {"status": "accepted", "warnings": []},
  "retention": {"class": "case-evidence", "legal_hold": false}
}
~~~

The evidence object points to protected content; prompts and traces normally receive a minimized projection and opaque reference. Corrections append a new revision. Deletion or restriction produces a tombstone and policy-aware behavior rather than falsifying the historical audit record.

## Derived fact contract

A derived fact is valid only when computation and scope are reproducible.

~~~json
{
  "derived_fact_id": "df_01...",
  "predicate": "outbound_amount_sum",
  "subject_ref": "acct_...",
  "value": {"amount": "125000.00", "currency": "USD"},
  "window": {"from": "...", "to": "...", "clock": "event_time"},
  "filters": ["status=settled", "exclude_reversals=true"],
  "method": "cashflow-features@sha256:...",
  "source_refs": ["ev_..."],
  "coverage": {"complete": true, "excluded_count": 2},
  "computed_at": "..."
}
~~~

Currency conversion, reversal handling, duplicated messages, missing intermediaries, partial windows, and late postings must be visible. A chart or narrative never replaces the underlying fact contract.

## Entity resolution is a hypothesis service

Entity resolution produces candidates, not canonical truth. Separate deterministic equivalence from probabilistic similarity:

- **Exact, governed link:** a system-of-record key or verified identifier under documented semantics.
- **Rule-derived candidate:** normalized name plus date of birth, address, registration number, or other attributes.
- **Model-derived candidate:** learned similarity or graph inference.
- **Human-adjudicated link:** accepted/rejected candidate with reviewer, reasons, time, and scope.

~~~json
{
  "candidate_id": "link_01...",
  "left_ref": "party_...",
  "right_ref": "registry:...",
  "relationship": "same_entity",
  "method": "entity-resolver@2.4.1+sha256:...",
  "features": [
    {"name": "name_similarity", "value": 0.94, "source_refs": ["ev_..."]},
    {"name": "dob_exact", "value": false, "source_refs": ["ev_..."]}
  ],
  "score": 0.71,
  "threshold_profile": "legal-entity-default@8",
  "alternatives": ["registry:..."],
  "status": "unresolved",
  "limitations": ["transliterated-name", "missing-registration-id"]
}
~~~

High-impact matches require independent evidence appropriate to the source and action. Never use a single global threshold across names, scripts, jurisdictions, data quality, and consequences. Measure false merges and false splits separately, including language and entity-type slices.

## Graph model

Use typed, temporal, source-backed edges. A useful minimum vocabulary is:

| Node | Example edges | Required qualifiers |
|---|---|---|
| Party/legal entity | owns, controls, directs, employed-by, related-to | source, valid interval, direct/indirect, percentage/type, confidence |
| Account/wallet/product | owned-by, authorized-user, funded-by, beneficiary-of | role, valid interval, provider, verification status |
| Transaction/message | sender, receiver, intermediary, device, location | rail, status, amount/currency, event time, direction |
| Address/contact/device | used-by, registered-at, observed-with | observation method/time, sharing/reuse risk |
| Alert/case | triggered-by, contains, supersedes, related-case | rule/model version, case access restrictions |
| Evidence/source | supports, contradicts, derived-from | source revision, transformation, custody, purpose |

~~~mermaid
flowchart LR
    P1["Party A"] -- "owns 60%; valid interval; registry ref" --> C["Company C"]
    C -- "owns account; KYC ref" --> A1["Account 1"]
    A1 -- "originates; ledger ref" --> T["Transfer"]
    T -- "beneficiary; message ref" --> A2["Account 2"]
    D["Device candidate"] -. "observed-with; provider ref; uncertain" .-> P1
    T --> E1["Evidence objects"]
    P1 --> E1
    C --> E1
~~~

The display must differentiate confirmed system relationships, deterministic derivations, probabilistic candidates, and analyst annotations visually and in export.

## Safe network queries

Expose bounded query types rather than general graph traversal:

| Query | Bounds and output |
|---|---|
| Transaction neighborhood | Direction, rail, time window, max depth/nodes/edges; includes truncation and source coverage |
| Ownership/control roll-up | Jurisdiction-specific relationship types, direct/indirect percentages, cycles, unknown portions, source revisions |
| Shared-attribute candidates | Attribute type, minimum evidence, time overlap, known sharing/reuse behavior; candidate status only |
| Flow path | Settled-value semantics, max hops/time gap, reversal treatment, currency method, alternative paths |
| Peer comparison | Cohort definition/version, inclusion criteria, privacy threshold, missingness, distribution—not only percentile |
| Typology feature bundle | Feature definitions, job version, window, thresholds, inputs, calibration, exclusions |

Every result reports `truncated`, `unavailable_sources`, `coverage_window`, `as_of`, and construction version. “No path” is meaningful only within those bounds.

## Evidence assembly algorithm

1. Pin the case cutoff, jurisdiction profile, customer/account scope, and source snapshots.
2. Resolve exact institutional identifiers first; retain unresolved external candidates separately.
3. Retrieve raw/normalized facts through authorized typed adapters.
4. Verify completeness, corrections, reversals, pagination, and source freshness before computation.
5. Run deterministic time-window and graph jobs with versioned definitions.
6. Assemble both inculpatory and exculpatory evidence plus known gaps.
7. Let the investigator loop request only bounded follow-up evidence.
8. Freeze the review bundle by evidence hashes and case version; invalidate it on material change.

## Common analytical traps

| Trap | Why it fails | Control |
|---|---|---|
| Current-state leakage | Later KYC, list, or ownership data changes what was knowable | Case cutoff and bitemporal snapshots |
| Double-counted value | Auth/settlement/reversal/message records represent one economic event | Rail-specific transaction lifecycle mapping |
| Graph guilt by association | Shared addresses/devices/intermediaries can be common or benign | Typed relationship, time overlap, base-rate and alternative explanation |
| Peer-group circularity | Risk score defines the cohort used to call activity anomalous | Independently governed cohort definition and sensitivity tests |
| Missingness as innocence | Source outage or incomplete fields suppress signals | Explicit missingness/coverage and conclusion blockers |
| Missingness as guilt | Sparse KYC or foreign data becomes an adverse inference | Separate data-quality remediation from suspicion evidence |
| Latest-record overwrite | Investigation cannot be reproduced | Immutable revisions and decision-time snapshot |
| Score laundering | Opaque vendor/model output is restated as fact | Inputs, method/version, calibration, limitations, human review |
| Alert feedback loop | Prior alerts/cases become features and reinforce historical selection | Outcome/selection-bias review; isolate operational labels |

## Scale and cost guidance

Do graph construction, deduplication, currency normalization, temporal windows, list ingestion, and common features in batch or streaming data systems. The reasoning loop should read small referenced views, not scan raw transaction histories. Cache only immutable or version-keyed projections and keep purpose/tenant in the cache key. Bound graph exploration by risk-relevant time, depth, node/edge count, and source. Show truncation instead of silently sampling.

Public graph datasets and synthetic generators can test pipelines and exploratory methods, but they do not establish production detection quality. The [Elliptic Bitcoin study](https://research.ibm.com/publications/anti-money-laundering-in-bitcoin-experimenting-with-graph-convolutional-networks-for-financial-forensics) is a narrow labeled research setting, and [AMLSim](https://github.com/IBM/AMLSim/) generates synthetic transactions; neither reproduces an institution's legal duties, data missingness, investigator behavior, or adversarial adaptation.

## Checklist

- [ ] Source, normalized, derived, candidate, hypothesis, decision, and effect objects are distinct.
- [ ] Every material object has stable identity, valid/event/observed time, source revision, and lineage.
- [ ] Transaction lifecycle semantics prevent double counting across authorization, settlement, reversal, and messages.
- [ ] Entity links expose method, alternatives, score/threshold where applicable, limitations, and review status.
- [ ] Graph queries are typed, bounded, temporal, coverage-aware, and reproducible.
- [ ] Review packages include contradictory, exculpatory, missing, stale, and truncated evidence.
- [ ] Evaluation prevents future-data leakage and operational-label feedback loops.

## Sources and next guide

- [W3C PROV Overview](https://www.w3.org/TR/prov-overview/)
- [FATF — Guidance on Beneficial Ownership of Legal Persons](https://www.fatf-gafi.org/en/publications/Fatfrecommendations/Guidance-Beneficial-Ownership-Legal-Persons.html)
- [GLEIF — Challenge LEI and vLEI Data](https://www.gleif.org/en/lei-data/gleif-data-quality-management/challenge-lei-and-vlei-data)
- [ISO 20022 e-Repository](https://www.iso20022.org/iso20022-repository/e-repository)

Next: [Alerts, cases, typologies, hypotheses, and decisions](04-alert-case-typology-and-investigation-reasoning.md).
