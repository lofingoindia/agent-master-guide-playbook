# Evidence Intake, Context, and Case State

> **Status:** Research-backed production blueprint  
> **Last researched:** 2026-08-31  
> **Scope:** Alert and evidence contracts, enrichment, claims, hypotheses, timelines, and custody.  
> **Section index:** [Security investigation and triage agent](README.md)

Investigation quality is bounded by evidence quality. Preserve the raw source, record what coverage was available, and make every conclusion traceable to stable evidence. Normalization, enrichment, retrieval, summarization, and model reasoning are derived views; none should erase the original.

## Evidence lifecycle

~~~mermaid
flowchart LR
    S["Authenticated source"] --> A["Admission checks"]
    A --> R["Immutable raw object"]
    A --> N["Versioned normalization"]
    N --> D["Deduplication and correlation"]
    D --> E["Authorized enrichment"]
    E --> C["Case snapshot"]
    C --> H["Hypotheses and claims"]
    H --> V["Analyst validation"]
    V --> O["Disposition / escalation"]
    R --> P["Preservation and custody"]
    N --> P
~~~

Each arrow is a transformation with a version, actor, timestamp, and status. The model receives references or constrained excerpts, not ownership of the evidence lifecycle.

## Intake envelope

Every received alert or artifact needs a system-owned envelope before its content is exposed to a model.

| Field | Purpose | Required behavior |
|---|---|---|
| event_id | Stable logical event identity | Deterministic when source semantics allow; never based on model text |
| delivery_id | Unique transport attempt | Deduplicate repeated delivery without hiding redelivery count |
| tenant_id | Security boundary | Derived from authenticated channel or signed mapping, never payload prose |
| source_system and source_instance | Origin and schema context | Pin connector and parser versions |
| source_record_id | Reconciliation with origin | Preserve exact native identifier |
| event_type and schema_version | Contract selection | Reject or quarantine unsupported versions |
| occurred_at | Source-reported event time | Include timezone and source clock metadata |
| observed_at | Sensor observation time | Distinguish from occurrence and ingestion |
| ingested_at | Platform receipt time | Assigned by trusted intake |
| raw_object_ref and digest | Immutable source material | Digest at acquisition; restrict direct access |
| sensitivity and sharing | Handling policy | Include tenant classification and TLP when applicable |
| lineage | Parent alert, correlation, transform | Append-only references |
| coverage | Source window and completeness | Make gaps and truncation explicit |

Example envelope:

~~~yaml
schema: security-event-envelope/v1
event_id: evt_01JX...
delivery_id: dlv_01JX...
tenant_id: tenant_acme
source:
  system: edr
  instance: prod-us
  connector_version: 4.2.1
  record_id: native-884193
time:
  occurred_at: 2026-08-31T04:17:12.431Z
  observed_at: 2026-08-31T04:17:14.002Z
  ingested_at: 2026-08-31T04:17:16.119Z
  source_clock_offset_ms: 180
content:
  raw_ref: evidence://tenant_acme/sha256/8b6d...
  sha256: 8b6d...
  media_type: application/json
handling:
  classification: confidential-security
  sharing: TLP:AMBER
coverage:
  complete: true
  notes: null
~~~

The example is an application contract, not an OCSF or STIX profile. Map to external schemas at integration boundaries and retain both schema versions.

## Admission and parser safety

Perform before semantic processing:

1. Authenticate the producer and bind it to a tenant and source instance.
2. Enforce maximum body, attachment, archive, nesting, expansion-ratio, row, and field lengths.
3. Identify media type from content and signature, not filename alone.
4. Store raw bytes in a non-executing evidence path and compute an approved digest.
5. Parse in a restricted worker with no response credentials and minimal egress.
6. Preserve parser warnings, dropped fields, decoding errors, and normalization loss.
7. Label all source-controlled strings as untrusted data.
8. Route active, malformed, encrypted, or unsupported content to quarantine or specialist review.

Do not render HTML, follow remote references, resolve external entities, execute macros, import compiled rules from untrusted parties, or preview active content inside the orchestration service.

## Normalization without evidence loss

OCSF provides a vendor-neutral security-event schema and is useful for correlation. It does not eliminate source-specific semantics. Use a three-layer record:

| Layer | Contents | Mutation policy |
|---|---|---|
| Raw | Original bytes, headers, source identifiers | Immutable |
| Normalized | OCSF or application mapping, canonical identifiers, parse diagnostics | New mapping version creates a new view |
| Investigative | Claims, tags, ATT&CK mappings, correlations, summaries | Append or supersede |

Normalization must record:

- mapping name, version, build digest, and execution ID;
- every source field used;
- fields dropped, redacted, coerced, truncated, or defaulted;
- timezone and clock assumptions;
- canonicalization rules for IPs, domains, paths, users, devices, cloud resources, and hashes;
- whether the record passed structural and semantic validation.

Never discard the native field that distinguishes two otherwise similar identities or resources.

## Deduplication and correlation

### Deduplicate transport, not reality

The same event may arrive through retries or multiple sensors. Keep these concepts separate:

- **delivery duplicate:** same source record delivered again;
- **semantic duplicate:** different records describing the same observable event;
- **related event:** distinct event in the same activity sequence;
- **correlated alert:** one or more alerts associated with a case;
- **suppressed alert:** detection intentionally not surfaced under a versioned rule.

Hard deduplication should use authenticated source identifiers or a documented canonical fingerprint. Probabilistic similarity may propose a correlation but must not delete or hide records.

### Correlation safety

Record why items were linked:

- exact shared source record;
- asset or identity identifier;
- session, process, trace, request, or cloud operation identifier;
- temporal and behavioral rule;
- analyst decision;
- model-proposed similarity.

Model-proposed correlations remain provisional until a deterministic rule or analyst confirms them.

## Asset and identity context

An alert severity is not an investigation priority. Priority combines evidence, potential impact, exposure, asset criticality, identity privilege, business timing, and uncertainty.

### Asset context

Retrieve from authoritative sources:

- canonical resource ID, owner, service, environment, region, and tenant;
- business criticality and confidentiality/integrity/availability requirements;
- internet exposure, network zone, dependencies, and blast radius;
- operating system, software inventory, patch state, and end-of-life status;
- sensor coverage and last heartbeat;
- maintenance, deployment, or change windows;
- recovery objective and containment constraints.

### Identity context

Retrieve:

- canonical subject and workload identities;
- account type, lifecycle state, owner, role, groups, and effective privileges;
- authentication method, device/session binding, and risk signals;
- recent role, token, credential, or federation changes;
- expected geography, service use, schedule, and break-glass status;
- aliases and identity-resolution confidence.

Do not infer that two names identify the same person or workload. Preserve the resolution method, authoritative directory, and ambiguity.

### Freshness contract

Every contextual fact should carry:

~~~yaml
value: production
source: cmdb/service-registry
source_record_id: svc-419
observed_at: 2026-08-31T03:55:00Z
retrieved_at: 2026-08-31T04:18:03Z
valid_until: 2026-08-31T04:25:00Z
confidence: authoritative
~~~

At action time, re-read facts that affect authorization or blast radius.

## Threat-intelligence enrichment

Threat intelligence is evidence about external context, not proof that a local event is malicious or attributable.

### Preserve these dimensions separately

| Dimension | Example | Why separate it |
|---|---|---|
| Source reliability | FIRST A–F style rating | A historically reliable source can still provide an uncertain item |
| Information credibility | FIRST 1–6 style rating | Independent corroboration differs from source reputation |
| Analytic confidence | low / moderate / high with rationale | Confidence in the assessment, not probability of occurrence |
| Estimative probability | standardized verbal range | Probability language must be calibrated |
| Freshness | retrieved and valid-through timestamps | Infrastructure and campaigns change |
| Sharing | TLP 2.0 marking and policy | Distribution limits do not express truth |
| Match type | exact hash, domain, behavior, technique | Indicators have different collision and reuse properties |
| Local relevance | asset, geography, sector, exposure | Global intelligence may not change local risk |

STIX 2.1 can structure intelligence and TAXII 2.1 can transport it. Neither proves source quality. Pin the feed snapshot or object version used for the conclusion.

### Indicator handling rules

- Canonicalize and validate before lookup.
- Preserve the raw observable and the normalized lookup key.
- Record exact provider response, retrieval time, and license/sharing restrictions.
- Treat a positive match as a lead whose meaning depends on context and time.
- Treat no match as unknown, not benign.
- Never perform active scanning, beaconing, detonation, or victim notification as an implicit enrichment step.
- Keep attribution language separate from technical activity assessment.

ATT&CK mappings describe behaviors and hypotheses. As of ATT&CK v19.2, use detection strategies, analytics, and data components rather than building new systems around the data-source objects deprecated in v18.

## Case model

### Minimum entities

| Entity | Required content |
|---|---|
| Case | Tenant, objective, status, priority, owners, policy scope, retention, timestamps |
| Alert link | Source alert, relation, correlation reason, disposition |
| Evidence item | Raw/derived reference, digest, source, acquisition, coverage, access |
| Observable | Typed value, canonicalization, first/last seen, source references |
| Claim | Precise assertion, status, evidence for/against, author, version |
| Hypothesis | Explanation, prior rationale, discriminating tests, outcome |
| Timeline event | Time interval, uncertainty, event/evidence links, ordering basis |
| Query | Intent, typed arguments, source, authorization, result coverage |
| Recommendation | Expected benefit, risks, alternatives, evidence prerequisites |
| Action | Exact effect, policy, approval, idempotency, attempt, observed outcome |
| Decision | Human or deterministic decision, rationale, superseded record |

### Claim state

~~~mermaid
stateDiagram-v2
    [*] --> Proposed
    Proposed --> Corroborated: independent evidence supports
    Proposed --> Contradicted: evidence conflicts
    Proposed --> Unresolved: evidence insufficient
    Corroborated --> AnalystConfirmed: accountable review
    Corroborated --> Contradicted: later evidence
    AnalystConfirmed --> Superseded: corrected or new evidence
    Contradicted --> Superseded: revised claim
    Unresolved --> Corroborated: new evidence
    Unresolved --> Contradicted: new evidence
~~~

Confidence does not replace state. A high-confidence proposed claim is still model-authored; an analyst-confirmed claim can later be superseded.

### Evidence-grounded claim example

~~~yaml
claim_id: clm_204
case_version: 17
statement: The session used a newly registered OAuth application.
status: corroborated
confidence:
  level: high
  rationale: Directory audit and application inventory agree.
supports:
  - evidence_ref: ev_991
    locator: jsonpath:$.app.createdDateTime
  - evidence_ref: ev_1044
    locator: row:18
contradicts: []
gaps:
  - Consent grant provenance is not available for the first 12 minutes.
author:
  type: agent
  run_id: run_71
supersedes: null
~~~

The statement is deliberately narrow. “The tenant was compromised by actor X” would require many more claims and is often unjustified.

## Hypothesis-driven investigation

Maintain at least:

- a leading malicious explanation;
- the strongest benign or administrative explanation;
- a sensor, parser, or configuration-failure explanation when plausible;
- an unknown/insufficient-evidence branch.

For each hypothesis record:

| Field | Question |
|---|---|
| Scope | Which assets, identities, sessions, and time interval does it explain? |
| Supporting evidence | Which stable evidence references increase plausibility? |
| Contradictions | Which observations should not exist if it were true? |
| Predictions | What else should be observable? |
| Discriminator | Which lowest-cost, authorized query best separates alternatives? |
| Coverage | Could the needed source observe it during the interval? |
| Outcome | Supported, weakened, falsified, or unresolved |

Do not ask the model only for “the most likely cause.” That framing encourages premature convergence and confirmation bias.

## Timeline model

Security timelines contain multiple clocks and uncertain times. Record:

- event time from the source;
- sensor observation time;
- collector ingestion time;
- acquisition and analysis time;
- timezone;
- known source offset or synchronization status;
- precision and uncertainty interval;
- whether ordering comes from a trusted sequence number, causal identifier, or timestamp only.

Represent uncertain time as an interval when necessary. Do not fabricate millisecond ordering from second-granularity logs. A useful timeline item can say “between 04:15:00 and 04:17:30 UTC” and explain why.

### Causal links

Use explicit relations such as:

- spawned;
- authenticated;
- assumed-role;
- connected-to;
- downloaded;
- modified;
- triggered;
- followed-by-observation;
- shares-session-with;
- analyst-associated.

“Occurred before” is not “caused.”

## Evidence integrity and custody

NIST SP 800-86 and NISTIR 8387 support acquisition documentation, integrity verification, secure storage, and analysis of copies. RFC 3227 emphasizes order of volatility and collecting before analyzing when possible.

For every evidence acquisition record:

- who or which authenticated service collected it;
- authority and case purpose;
- source system, device, tenant, and location;
- start and completion times in UTC;
- acquisition tool name, version, configuration, and build digest;
- exact command or API request in a safe structured form;
- source and destination object identifiers;
- digest algorithm and value;
- errors, omissions, filters, sampling, and known transformations;
- storage location, access policy, retention, and legal-hold status;
- every custody transfer or analysis-copy creation.

Use an organization-approved hash. NISTIR 8387 recommends a NIST-approved algorithm and separate secure storage of hash data. Do not silently “repair” a mismatch.

### Evidence versus investigative artifact

| Object | Example | Handling |
|---|---|---|
| Acquired evidence | Exported audit log, disk image, memory image | Immutable, hashed, custody-tracked |
| Derived evidence | Parsed event table, extracted strings | Reproducible transform linked to source |
| Investigative artifact | Timeline, graph, model summary | Versioned interpretation, not original evidence |
| Operational telemetry | Tool latency, token count, policy denial | Auditable system record; not automatically case evidence |

If the agent transcript becomes relevant to an incident involving the agent itself, promote a preserved copy under a documented acquisition process instead of assuming ordinary trace storage satisfies evidence requirements.

## Context assembly

Build each model context from a deterministic manifest:

~~~yaml
context_manifest:
  case_id: case_781
  case_version: 17
  objective: Determine whether the sign-in sequence warrants escalation.
  authority_tier: supervised_read_only
  policy_version: soc-investigation/12
  evidence:
    - ref: ev_991
      trust: untrusted_source_data
      excerpt: excerpt://ev_991/3
      complete: false
  facts:
    - ref: fact_asset_44
      trust: authoritative
      valid_until: 2026-08-31T04:25:00Z
  open_hypotheses: [hyp_7, hyp_8, hyp_9]
  known_gaps:
    - Identity logs unavailable from 04:01 to 04:13 UTC.
  budgets:
    tool_calls_remaining: 8
    bytes_remaining: 200000
    deadline_at: 2026-08-31T04:23:00Z
~~~

The manifest makes omissions visible and allows evaluation of what the model actually saw.

### Compaction and memory lifecycle

| Information class | Default decision | Investigation rule |
|---|---|---|
| Turn context | Enable, evidence-bounded | Compile objective, tenant/case/version, authority, selected evidence/facts, hypotheses, gaps, coverage, budgets, and pending decisions/effects for one call |
| Short-term working memory | Enable as typed case projection | Track hypotheses, contradictions, open queries, rejected explanations, tasks, evidence IDs, coverage, and remaining budget; rebuild it from durable state and never turn model confidence into a fact |
| Conversation/session memory | Minimize and non-authoritative | Retain only what supports the current analyst interaction; resume from case/evidence/effect state and current source coverage, not a chat transcript or framework checkpoint |
| Durable task memory | Required as application state | Intake, evidence/custody, observations, claims, hypotheses, decisions, approvals, effects, receipts, and terminal/supersession records remain model-independent and schema-versioned |
| Domain or semantic memory | Source-controlled only | Asset/identity truth, detections, threat intelligence, runbooks, policies, tools, and ownership retain native source, version, sharing, freshness, and review status; retrieval never promotes a summary to authority |
| General long-term model memory | Reject for case truth and authority | Do not let provider memory, a vector index, or free-form model-written notes influence tenant, verdict, evidence, retention, approval, or action; approved domain artifacts belong in governed source systems |
| User preference memory | Disable by default | Display preferences may be explicit and deletable, but cannot change tenant, evidence access, authority, containment, retention, or legal hold |
| Cross-run episodic memory | Curated, derived, and isolated | Analyst-confirmed, de-identified incidents/corrections may become eval fixtures or proposed detections/runbooks after review; raw case content, personal data, attacker strings, and model conclusions never self-promote or become live instructions |

Compaction changes only the model's working projection. Persist a structured receipt with input event/evidence ranges, tenant/case/version, retained claim/hypothesis/evidence/task IDs, contradictions and coverage gaps, custody/legal-hold constraints, approvals/effects, budgets, explicit omissions, compiler/model version, and digest. The receipt is a continuity artifact, not evidence or authority. Repeated-compaction tests must prove that source coverage, benign alternatives, timeline uncertainty, chain of custody, containment authority, unknown effects, and stop/escalation conditions cannot disappear. Raw evidence and canonical case history remain separately governed and recoverable.

Use retrieval, not long-term conversational memory, to re-enter an old case. Recompile current authoritative facts because ownership, credentials, CTI, sensor health, asset criticality, policy, and source retention may have changed since the previous run. Any episodic example retrieved from another case is untrusted advisory material and must not carry identifiers, approvals, or conclusions into the current tenant.

## Common failure modes

| Failure | Consequence | Control |
|---|---|---|
| Raw data discarded after normalization | Cannot reproduce or repair parser error | Immutable raw reference and mapping lineage |
| One timestamp field | False sequence and impossible causality | Multiple clocks, offset, precision, uncertainty |
| CTI hit treated as verdict | False attribution or escalation | Match semantics, local context, source and item confidence |
| No result treated as benign | Silent false negative during outage or retention gap | Coverage object and explicit unknown |
| Model summary replaces source | Lost nuance, injection persistence, unverifiable claim | Source citation and versioned derived artifact |
| Alert dedupe uses semantic similarity only | Distinct attacks collapse | Delivery identity plus visible probabilistic correlation |
| Asset name used as tenant boundary | Cross-tenant collision | Canonical tenant-scoped resource identifier |
| Evidence copied into general vector store | Retention and access boundary bypass | Authorized evidence index with deletion and provenance |
| Case closure freezes truth | Late evidence ignored | Supersession and reopen transitions |

## Acceptance checklist

- [ ] Every alert can be traced to raw bytes and a native source record.
- [ ] Unsupported or lossy mappings are visible to analysts.
- [ ] Query results declare time coverage, pagination, truncation, and source health.
- [ ] Claims cite evidence for and against and expose unresolved gaps.
- [ ] Hypotheses include a realistic benign alternative.
- [ ] Timeline ordering does not exceed source precision.
- [ ] CTI source reliability, item credibility, confidence, freshness, and sharing are distinct.
- [ ] Evidence access and derived-index access obey the same tenant and retention policy.
- [ ] Custody records are append-only and integrity failures generate an incident.

## Related guides

- [Investigation reasoning, tools, models, and runtime](investigation-reasoning-tools-and-runtime.md)
- [Untrusted content, forensics, and data governance](untrusted-content-forensics-and-data-governance.md)
- [Tool results, artifacts, and provenance](../../tools/tool-results-artifacts-and-provenance.md)
- [Context engineering](../../context-memory/context-engineering.md)

## Selected sources

- [NIST SP 800-86, Guide to Integrating Forensic Techniques into Incident Response](https://csrc.nist.gov/pubs/sp/800/86/final)
- [NISTIR 8387, Digital Evidence Preservation](https://www.nist.gov/publications/digital-evidence-preservation-considerations-evidence-handlers)
- [RFC 3227, Guidelines for Evidence Collection and Archiving](https://www.rfc-editor.org/info/rfc3227)
- [ISO/IEC 27037:2012](https://www.iso.org/standard/44381.html)
- [SWGDE Best Practices for Digital Evidence Collection](https://www.swgde.org/documents/published-complete-listing/18-f-002-2-0/)
- [OCSF schema 1.8.0](https://github.com/ocsf/ocsf-schema/releases/tag/1.8.0)
- [OASIS STIX 2.1 and TAXII 2.1 standards](https://www.oasis-open.org/2021/06/23/stix-v2-1-and-taxii-v2-1-oasis-standards-are-published/)
- [FIRST source evaluation and information reliability](https://www.first.org/global/sigs/cti/curriculum/source-evaluation)
- [FIRST communicating uncertainties in CTI reporting](https://www.first.org/global/sigs/cti/curriculum/cti-reporting)
- [FIRST TLP 2.0](https://www.first.org/tlp/)
- [MITRE ATT&CK v18 defensive-model changes](https://attack.mitre.org/resources/updates/updates-october-2025/)
