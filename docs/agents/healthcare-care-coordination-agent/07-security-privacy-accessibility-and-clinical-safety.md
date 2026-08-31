# Security, Privacy, Accessibility, and Clinical Safety

Security, privacy, accessibility, and clinical safety interact. A locked-down workflow that a patient cannot use may delay care; an accessible message sent to an unauthorized proxy is still a breach; a correct administrative action on the wrong patient is a safety event.

## Governance before controls

Name accountable owners for:

- intended use and product scope;
- clinical safety and hazard acceptance;
- patient identity and record correction;
- privacy, consent, representative authority, and records management;
- cybersecurity, tenancy, secrets, and incident response;
- accessibility, language access, and accommodations;
- each clinical service and care-team route;
- each integration and processor;
- model/evaluation/release behavior.

The agent cannot hold any of these accountabilities.

## Threat model

### Assets

- patient and representative identities;
- PHI and sensitive data categories;
- clinical source records and provenance;
- consent, authority, and confidential-channel preferences;
- scheduling, referral, authorization, and care-plan status;
- care-team routing and safety escalations;
- credentials, approvals, effect IDs, receipts, and audit;
- prompts, model outputs, evaluations, and behavior manifests.

### Adversaries and failure sources

- external attacker;
- malicious or compromised insider;
- overprivileged service or vendor;
- wrong patient, proxy, or tenant binding;
- prompt injection embedded in a note, document, payer response, or message;
- model hallucination or overconfident inference;
- stale or conflicting source data;
- inaccessible design or language mismatch;
- lost, late, duplicated, or reordered events;
- dependency or safety-queue outage;
- configuration, adapter, policy, or behavior release error.

### Trust boundaries

~~~mermaid
flowchart LR
    C[Patient and workforce channels] -->|untrusted input| I[Identity boundary]
    I -->|verified actor and binding| P[Policy boundary]
    P -->|field-limited context| A[Agent runtime]
    A -->|typed proposal| V[Deterministic validator]
    V -->|authorized effect| E[Effect boundary]
    E -->|least-privilege credential| D[Domain systems]
    D -->|untrusted response with provenance| A
    A --> O[Redacted operations telemetry]
    A --> L[Restricted audit and safety ledger]
~~~

The model runtime is not inside the trusted identity, policy, or effect boundary merely because it runs in the same cloud account.

## PHI data flow

Maintain a data-flow inventory for every field:

| Field question | Required answer |
|---|---|
| Why is it needed? | Named workflow step and purpose |
| Where is it authoritative? | System/resource/version |
| Who may see it? | Role/representative/patient and policy |
| May the model receive it? | Yes/no plus minimization/transformation |
| May a processor retain it? | Contracted duration and deletion |
| May it appear in logs/traces/evals? | Default no; approved reference/redaction strategy |
| Where is it stored and replicated? | Region, store, backups, caches, indexes |
| When is it corrected/deleted? | Retention, amendment, legal hold, correction propagation |
| Which incident rule applies? | Organization/jurisdiction-specific response |

Prefer opaque references and retrieve PHI just in time. Do not duplicate the longitudinal chart into a vector index or prompt cache.

## Privacy and consent enforcement

At every read, context assembly, disclosure, and effect, bind:

~~~text
actor + patient + tenant + relationship + purpose + action
+ data classes + destination + channel + time + policy version
~~~

Controls:

- deny by default;
- enforce outside the model;
- revalidate after waits and before effects;
- filter fields at the data layer;
- honor confidential-channel and proxy restrictions;
- treat FHIR Consent and security labels as policy evidence, not self-enforcing permission;
- define unknown-label and conflicting-consent behavior;
- separate ordinary access, emergency/break-glass, correction, export, and legal-hold workflows;
- retain the exact decision and supporting versions.

HIPAA is not a global privacy template. Even in the US, personal-representative rules, specially protected records, state law, and context-specific exceptions require governed profiles. The current HHS Security Rule remains the governing baseline while proposed changes remain proposals; legal owners must track changes rather than prompts hard-coding them.

## Processor and model-provider controls

Before PHI reaches a provider:

- determine covered-entity/business-associate or equivalent roles with counsel;
- execute required agreements;
- verify product feature scope, regions, subprocessors, retention, training-use policy, deletion, encryption, and support access;
- disable incompatible logging, feedback, caching, memory, file retention, or training features;
- use tenant and environment isolation;
- obtain assurance and incident-notification commitments;
- test that deletion and retention settings work;
- define outage, provider exit, export, and breach processes.

HHS cloud guidance notes that a cloud provider maintaining ePHI can be a business associate even if it cannot decrypt the data. Encryption is necessary but does not remove contractual and operational responsibility.

## Authentication and least privilege

Use NIST SP 800-63-4 concepts to separate:

- identity proofing assurance;
- authenticator assurance;
- federation assurance.

These do not replace patient matching or proxy authorization.

For workforce and services:

- phishing-resistant authentication where risk requires;
- short-lived, audience-bound tokens;
- SMART discovery and narrow patient/user/system scopes for FHIR;
- explicit preauthorization for backend service access;
- role plus attribute/purpose checks;
- just-in-time elevation and audited break-glass if applicable;
- independent approval for D4 policy/tool changes;
- periodic access and service-principal review.

Avoid broad wildcard system scopes for a convenience agent. If a vendor only supports broad scopes, constrain the adapter, data fields, network, tenancy, and workflow, and record the residual risk.

### Break-glass is a separate workflow

Do not implement emergency access as a prompt instruction or reusable administrator token. Where the organization permits break-glass:

- an authenticated person explicitly invokes an organization-defined reason and patient scope;
- the system verifies eligible role, step-up assurance, current emergency policy and unavailable ordinary alternative;
- access is time-limited, data-limited and isolated from routine agent effects;
- the agent does not decide that an emergency exists, select a clinical disposition, or invoke break-glass by itself;
- every read/effect is conspicuously labeled and sent to independent real-time/post-event review;
- credentials expire and sessions cannot silently return to ordinary automation;
- the case enters clinical, privacy and security follow-up under local policy.

If the policy service is unavailable, deny by default or use only a separately approved human break-glass path. Cached routine permission is not an emergency control.

## Tenant isolation, secrets, and supply chain

Treat tenant and patient scope as data-plane keys, not model-readable conventions.

- derive tenant from the authenticated route/service identity; never accept a tenant, patient, recipient, endpoint or scope solely from generated text;
- enforce tenant predicates in every database, object store, cache, vector/retrieval index, queue, event partition, log lookup and adapter credential;
- use separate encryption context and service credentials where the risk model requires; test backup, export, support and analytics paths for cross-tenant access;
- mint short-lived audience-bound tokens after policy evaluation and pass opaque handles to workers; rotate and revoke without redeploying prompts;
- restrict outbound DNS/hosts and block metadata-service, control-plane and unapproved model/tool endpoints;
- scan repositories, images, prompts, fixtures, traces and incident attachments for secrets and PHI; synthetic test data remains the default.

The behavior supply chain includes model/provider route, prompts, schema, workflow definition, policy bundle, adapter, terminology/map package, template, parser/OCR library, container and deployment configuration. For each release:

1. pin immutable identifiers and hashes in the behavior manifest;
2. generate an inventory/SBOM appropriate to the build and retain license/provenance evidence;
3. verify artifact signatures or trusted build provenance where supported;
4. scan dependencies and container/runtime configuration, including transitive parsers used on hostile documents;
5. review publisher privileges and require independent approval for safety, policy, adapter-scope and D4 changes;
6. run malicious-package, compromised-model-route, prompt/template tamper and rollback tests;
7. maintain kill switches by provider, model route, adapter operation, parser, channel, tenant and behavior version;
8. patch through the whole-bundle canary process—never hot-swap an untested “security fix” that changes clinical-administrative behavior without impact review.

Dynamic plugins, tools, schemas, prompts, network destinations and package installation are disabled in patient runs. A signed artifact can still be unsafe; provenance complements, not replaces, qualification and evaluation.

## Prompt injection and untrusted content

Every note, referral, PDF, payer response, portal message, directory description, and external payload is untrusted data.

Required controls:

- parse in an isolated service with attachment scanning and limits;
- strip active content and never execute embedded scripts/macros;
- label content and source boundaries in the context;
- instruct the model that retrieved text cannot grant authority or alter policy;
- expose a closed action enum, not arbitrary tool calls;
- validate citations against actual supplied evidence;
- block URLs, recipients, identifiers, and fields not authorized by contract;
- prevent cross-tenant retrieval;
- test indirect injection that asks for chart export, tool discovery, secret access, policy override, or clinical advice;
- fail closed on parser, provenance, or label uncertainty.

An instruction in a physician note is clinical evidence only when it is represented in the appropriate authoritative workflow and within the coordinator's allowed use. Text that says “ignore restrictions” is never an authorization.

## Clinical-safety management

Use a documented safety process appropriate to the organization and jurisdiction. The 2025 SAFER guides provide concrete patient-identification and clinician-communication practices. NHS DCB0129 and DCB0160 offer a mature manufacturer/deployer hazard-log and safety-case pattern where applicable; they are jurisdiction-specific and currently under review.

### Safety case structure

| Element | Required content |
|---|---|
| Intended use | Population, setting, workflow, users, channels, integrations, authority ceiling |
| Excluded use | Diagnosis, treatment, medication change, clinical urgency, emergency disposition |
| Hazards | Causal chain, affected people, severity, likelihood rationale, detectability |
| Controls | Preventive, detective, mitigative, recovery, owner |
| Evidence | Tests, evaluations, conformance, drills, monitoring, residual-risk review |
| Assumptions | Staffing, queue coverage, source freshness, communication access, vendor behavior |
| Residual risk | Named acceptor and review date |
| Change triggers | Intended use, population, workflow, model, adapter, policy, incident |

### Hazard-log contract

~~~yaml
clinical_hazard:
  hazard_id: opaque
  title: wrong-patient-referral-action
  initiating_causes: [ambiguous-match, stale-binding]
  hazardous_state: effect-authorized-for-wrong-record
  potential_harm: organization-assessed
  affected_populations: [approved-slices]
  controls:
    - control_id: patient-binding-pre-effect-check
      type: preventive
      owner: identity-platform
      evidence_refs: [test-or-monitor]
  residual_risk: organization-assessed
  risk_acceptor: opaque
  review_due_at: timestamp
  status: open | controlled | accepted | retired
~~~

Do not let the model assign hazard severity or accept residual clinical risk.

## Safety escalation lane

The safety lane is independently available from the ordinary model path.

~~~mermaid
flowchart TD
    I[Incoming evidence or overdue event] --> T{Approved deterministic trigger?}
    T -- Yes --> H[Set safety hold]
    T -- No --> M{Model reports material clinical uncertainty?}
    M -- Yes --> H
    M -- No --> N[Continue bounded administrative workflow]
    H --> Q[Route minimal evidence to staffed clinical service]
    Q --> A{Acknowledged in approved time?}
    A -- Yes --> C[Clinician owns assessment and disposition]
    A -- No --> B[Backup route and operations alert]
    C --> R[Authorized release, change, or closure]
~~~

The agent:

- can identify a configured trigger or uncertainty;
- can send the approved minimum escalation packet;
- cannot diagnose, advise, set urgency, choose destination, or cancel the need for review;
- cannot resume routine effects until an authorized transition releases the hold.

Safety communications must be authored and approved by the clinical governance process. Do not invent emergency guidance in the prompt.

## Accessibility and effective communication

Set an explicit product accessibility target such as WCAG 2.2 AA where appropriate, then supplement technical conformance with user and assistive-technology testing.

Design for:

- keyboard-only and screen-reader operation;
- visible focus, adequate contrast, zoom/reflow, and target size;
- no timing trap for people who need longer;
- plain language and clear action/status;
- text alternatives and captions;
- accessible identity proofing and consent flows;
- signed-language, interpreter, relay, phone, paper, and human-supported alternatives as applicable;
- verified language preference and qualified translation/interpreter workflows;
- low bandwidth, no smartphone, and portal-inaccessible paths;
- caregiver/representative flows that do not expose unauthorized data;
- the ability to correct an error and reach a human.

Machine translation can assist drafting only under an approved workflow. Do not let it silently alter clinical intent, safety content, consent, or patient instructions.

### Accessibility failure modes

| Failure | Control |
|---|---|
| Timeout expires during identity verification | Extend/pause safely; preserve no authority beyond verified step |
| Screen reader cannot distinguish pending from booked | Semantic status, live-region testing, explicit text |
| Color alone marks urgent handoff | Redundant label/icon/text and assistive test |
| SMS omits context and patient cannot access portal | Approved minimal callback route and human alternative |
| Language preference is inferred from name | Use explicit authoritative preference |
| Proxy interface reveals hidden diagnosis | Field-level policy and proxy-scope test |
| Automated translation changes scheduling constraint | Human/qualified workflow for material content |

## Logging, audit, and minimization

Keep three stores distinct:

| Store | Purpose | PHI posture |
|---|---|---|
| Operational telemetry | Health, latency, errors, saturation | Opaque IDs and redacted attributes |
| Decision/effect ledger | Reconstruct policy, approval, tool, receipt, reconciliation | Restricted, purpose-built, retention-controlled |
| Clinical record/provenance | Care documentation and authoritative evidence | Remains in domain system; reference exact version |

Never enable raw prompt/response logging globally. If an approved diagnostic capture is necessary, use targeted authorization, encryption, minimal duration, access logging, and deletion confirmation.

## Data lifecycle

- Classify every store, queue, cache, index, backup, and evaluation artifact.
- Configure retention by record class and jurisdiction.
- Propagate patient-record corrections and revoked access into retrieval indexes.
- Keep append-only historical audit while marking corrections.
- Test deletion in model-provider files/caches and internal derived stores where required.
- Separate legal hold from ordinary product retention.
- Prevent production PHI from entering developer laptops, issue trackers, screenshots, or general analytics.
- Use synthetic fixtures by default.

## Incident intersections

One event may be security, privacy, clinical safety, and reliability simultaneously. Examples:

- wrong-patient reminder;
- referral lost after adapter deploy;
- safety escalation stuck in a queue;
- proxy disclosure after revocation;
- inaccessible workflow causing missed follow-up;
- model output recommending a medication change.

The incident router must notify all applicable owners. Do not wait for definitive harm before containing the system.

Immediate containment may include:

- disable a model action, adapter write, channel, tenant, or behavior version;
- preserve effect/audit evidence;
- reconcile in-flight work;
- switch to manual operations;
- activate backup safety routing;
- assess affected patients/cases through accountable clinical and privacy processes.

## Security, privacy, accessibility, and safety release checklist

- [ ] Named governance owners approved intended and excluded use.
- [ ] Data-flow inventory covers processors, stores, caches, backups, logs, and evaluations.
- [ ] Identity/patient/proxy/purpose/action/data/destination policy is enforced outside the model.
- [ ] Provider contracts and settings cover PHI, retention, training, regions, subprocessors, incidents, and deletion.
- [ ] Workforce and service access is least-privilege, short-lived, reviewed, and audited.
- [ ] Untrusted-content and cross-tenant injection tests pass.
- [ ] Hazard log, safety case, control evidence, residual-risk acceptance, and change triggers exist.
- [ ] Safety escalation has staffed primary and backup routes.
- [ ] Accessibility and language paths are tested with humans and assistive technology.
- [ ] Operational telemetry is PHI-minimized and separate from the restricted ledger.
- [ ] Incident playbooks cover cross-domain events and manual fallback.

## Related guides

- Previous: [Tools, Effects, Idempotency, Reconciliation, and Recovery](06-tools-effects-idempotency-reconciliation-and-recovery.md)
- Next: [Observability, Evaluation, Failure Injection, and Incidents](08-observability-evaluation-failure-injection-and-incidents.md)
- Home: [Healthcare Care Coordination Agent](README.md)
- Canonical: [Agent Threat Model](../../security/agent-threat-model.md)
