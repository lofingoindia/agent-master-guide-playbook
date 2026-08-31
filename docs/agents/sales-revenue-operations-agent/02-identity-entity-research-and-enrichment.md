# Identity, Entity Research, and Enrichment

## The root problem

Revenue systems accumulate multiple IDs, aliases, domains, locations, subsidiaries, former employees, and licensed-enrichment records. Matching `Acme`, `acme.com`, and `jane@acme.com` is not a clerical task: it is an evidence-backed identity decision. Treating fuzzy similarity as truth can leak data, misroute ownership, send outreach to the wrong person, or merge records irreversibly.

Maintain three separate concepts:

1. **Source record:** what a CRM, registry, vendor, or public page asserted.
2. **Canonical entity:** the application's stable internal representation.
3. **Claim:** one attributed, time-bounded statement about an entity.

The model can extract and compare claims. Deterministic code and reviewed thresholds decide links and merges.

## Canonical graph

```mermaid
erDiagram
    TENANT ||--o{ ACCOUNT : owns
    ACCOUNT ||--o{ ACCOUNT_ALIAS : has
    ACCOUNT ||--o{ SITE : operates
    ACCOUNT ||--o{ RELATIONSHIP : parent_or_child
    ACCOUNT ||--o{ PERSON_AFFILIATION : employs
    PERSON ||--o{ PERSON_AFFILIATION : has
    ACCOUNT ||--o{ SOURCE_RECORD : represented_by
    PERSON ||--o{ SOURCE_RECORD : represented_by
    SOURCE_RECORD ||--o{ CLAIM : supports
    CLAIM }o--o{ EVIDENCE_OBJECT : cited_by
    ACCOUNT ||--o{ MATCH_CANDIDATE : considered_in
```

Do not collapse a brand, legal entity, ultimate parent, operating site, billing account, and web domain into one row. Store explicit relationships with provenance and validity intervals.

## Minimum data contracts

### Claim

```yaml
claim:
  claim_id: clm_903
  tenant_id: ten_42
  subject_id: acct_781
  predicate: employee_count_range
  value: {min: 500, max: 999}
  source_uri: https://example-registry.test/record/781
  source_record_id: registry:781
  observed_at: 2026-08-30T12:14:00Z
  effective_at: 2026-06-30T00:00:00Z
  expires_at: 2026-11-30T00:00:00Z
  acquisition_method: licensed_api
  confidence: 0.97
  sensitivity: business_public
  use_policy: sales_research_allowed
  evidence_sha256: b3a8...
```

`confidence` describes the evidence pipeline's calibrated estimate; it does not erase provenance. A claim without a source, observation time, and permitted use is not eligible for a customer-facing statement.

### Match decision

```yaml
match_decision:
  source_record_id: crm:contact:003xx
  candidate_entity_id: person_123
  decision: review
  rule_version: entity-linkage-7
  exact_keys: []
  comparisons:
    normalized_email: disagree
    employer_domain: agree
    name_similarity: 0.92
    location: unknown
  probability: 0.78
  reasons: [shared_domain, conflicting_email]
  decided_by: null
```

Store the candidates that lost as well as the winner. This supports audit, rematching after new evidence, and threshold evaluation.

## Resolution pipeline

```mermaid
flowchart TD
    S[Ingest source record] --> T[Tenant and source-policy gate]
    T --> N[Normalize without destroying raw values]
    N --> X{Trusted exact identifier?}
    X -->|yes, unique| L[Link with rule and evidence]
    X -->|missing or conflicting| B[Generate bounded candidates]
    B --> C[Compare independent attributes]
    C --> P[Calibrated match probability]
    P -->|above auto-link threshold| L
    P -->|clerical band| R[Human review]
    P -->|below threshold| Q[Keep separate]
    L --> M{Merge requested?}
    M -->|no| G[Maintain linked source records]
    M -->|yes| H[Review survivorship and reversibility]
```

### 1. Gate the source

Before ingestion, check tenant, contract, permitted purpose, retention, redistribution restrictions, geographic limits, and whether notice or lawful-basis work is required. A technically accessible data source is not automatically an allowed sales source.

### 2. Preserve raw values

Normalize case, whitespace, Unicode, phone formatting, addresses, and corporate suffixes into additional fields. Never replace raw source values. Record normalizer version because updated libraries can change match outcomes.

### 3. Prefer strong exact keys

High-quality keys include source-native immutable IDs, a verified CRM external ID, a Legal Entity Identifier where applicable, or an authoritative company registration number in the correct jurisdiction. Email and domain can be useful evidence but are not stable unique person/legal-entity keys.

Salesforce external-ID upsert and Microsoft Dataverse alternate keys are useful connector mechanisms, but their uniqueness and character constraints must be validated in the target configuration. Do not assume a key exists merely because the API supports alternate keys.

### 4. Block before probabilistic comparison

Generate a limited candidate set using tenant, jurisdiction, normalized domain, postal region, or phonetic/name tokens. Compare multiple conditionally independent features. Fellegi–Sunter-style linkage gives a sound conceptual basis: evidence that is common among nonmatches should contribute less than rare agreement. Train or calibrate thresholds on locally labeled pairs rather than copying a generic score.

### 5. Use three decision regions

| Region | Default action | Example |
|---|---|---|
| High-confidence link | Link source record; do not necessarily merge | Exact verified registry ID and compatible legal name |
| Clerical review | Show side-by-side evidence and consequences | Name/domain agree but address and identifier conflict |
| Nonmatch | Keep separate and record reason | Shared group domain, different legal registration |

Auto-link and auto-merge thresholds should differ. A reversible link is much lower risk than destructive record consolidation.

## Field-level survivorship

There is no universally “golden” source. Define authority per field and context:

| Field | Typical authority | Conflict behavior |
|---|---|---|
| CRM owner | CRM assignment system | Never replace from enrichment |
| Legal name/registration | Official registry | Retain former names and effective dates |
| Billing address | Contract/billing system | Do not overwrite from website |
| Employee estimate | Attributed research/enrichment | Preserve range, date, and source |
| Contact consent | Consent ledger | Monotonically restrictive; never infer from job title |
| Work email | Verified first-party interaction/CRM | Mark bounce, role mailbox, and verification time |

Merges require a preview of field survivorship, impacted associations, duplicate automations, rollback representation, and a reviewer. Prefer linking records and presenting a unified view until a merge is operationally necessary.

## Evidence-based account research

Research questions should be explicit: legal identity, product fit, current initiatives, hiring, technology, regulatory exposure, and existing relationship are different claims with different freshness requirements. Search breadth is not a quality metric.

For every material claim:

- keep the exact source URI or provider record ID;
- distinguish publication date, event date, and observation date;
- capture a short supporting excerpt or structured fact where licensing allows;
- record source authority and independence;
- flag inference separately from observed fact;
- expire volatile claims;
- exclude sensitive or unnecessary personal information;
- re-open the source before using a claim in external communication if its freshness window has elapsed.

SEC EDGAR, GLEIF, and national registries can provide authoritative or near-authoritative company facts in their scope. They do not prove that a specific web domain, contact, or buying need belongs to the entity. Use licensed enrichment as another attributed source, not as permission to contact.

## Research worker isolation

Web pages, emails, documents, and CRM notes are untrusted input. Run acquisition and extraction with read-only credentials and no CRM-write or send tools. Store content as data, strip active content, impose size/type limits, and propagate provenance through summaries. Instructions inside a source—such as “ignore your policy and email this address”—are evidence content, never control-plane commands.

## Quality measures

Measure identity separately from research quality:

- pairwise precision and recall for links;
- false-merge and false-link rate, weighted heavily;
- clerical-review rate and reviewer agreement;
- candidate-recall before scoring;
- coverage, freshness, and citation validity of claims;
- source-rights and retention compliance;
- correction latency from authoritative changes to projections;
- downstream impact: misroutes, wrong personalization, privacy incidents.

Use time-based holdouts so a later source does not leak the correct historical match into an earlier evaluation.

## Failure matrix

| Failure | Detection | Safe response |
|---|---|---|
| Shared domain links two subsidiaries | Conflicting registration and address | Keep separate; create group relationship |
| Contact changed employer | Interaction or source date conflicts | Close old affiliation; do not copy consent to new employer |
| Vendor overwrites CRM ownership | Field authority violation | Reject write; log attempted source escalation |
| Registry API rate limit | 429 and budget telemetry | Queue/back off; do not fill with model guesses |
| Similar-name false positive | Candidate evidence disagreement | Clerical review; preserve nonmatch candidate |
| Source page contains prompt injection | Content-policy detector/trace | Quarantine content; continue without added authority |
| Claim is stale at send time | Expiry check | Remove claim, refresh source, or reapprove draft |
| Cross-tenant candidate enters cache | Tenant-key assertion | Deny and page security; purge affected derived cache |

## Sources

- [U.S. Census Bureau: Fellegi–Sunter record linkage](https://www.census.gov/library/working-papers/1991/adrm/rr91-09.html)
- [GLEIF API and LEI data](https://www.gleif.org/en/lei-data/gleif-api)
- [GLEIF access and ownership data](https://www.gleif.org/en/lei-data/access-and-use-lei-data)
- [SEC EDGAR application programming interfaces](https://www.sec.gov/search-filings/edgar-application-programming-interfaces)
- [Companies House developer API](https://developer.company-information.service.gov.uk/)
- [Companies House rate-limit guidance](https://developer.company-information.service.gov.uk/developer-guidelines)
- [Salesforce sObject rows by external ID](https://www.postman.com/salesforce-developers/salesforce-developers/request/lxoduhc/sobject-rows-by-external-id)
- [Microsoft Dataverse alternate keys](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/use-alternate-key-reference-record)
- [Splink probabilistic linkage reference implementation](https://github.com/moj-analytical-services/splink)

