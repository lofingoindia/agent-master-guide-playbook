# Workload Fit, Scope, and Accountability

> **Purpose:** Decide whether an agent belongs in the engagement, constrain its authority, and name the humans accountable for every judgment before implementation begins.

## Start with the work, not the model

Compliance evidence work contains several different problem types. Only some benefit from model judgment.

| Work shape | Best default | Model role, if any |
| --- | --- | --- |
| Fixed requirement-to-control relationship in an approved catalog | Versioned database/rules | Explain or search the record; do not remap on every run |
| Scheduled API extraction with a stable schema | ETL/workflow job | None unless a human-readable summary is valuable |
| Evidence requests, reminders, assignment, and escalation | Case/workflow engine | Draft a purpose-limited request |
| Random/systematic selection from a frozen population | Deterministic sampler | None |
| Unstructured policy, ticket, screenshot, or interview record | Workflow plus bounded model worker | Extract candidate facts and cite exact evidence locations |
| Contradictory or incomplete evidence | Human-led case workflow | Organize conflicts and questions; abstain from conclusion |
| Professional judgment about sufficiency, effectiveness, materiality, or opinion | Qualified human | Decision support only |

If structured queries and rules can produce the required evidence package reliably, build that workflow without a model. The agent is justified when measured extraction, cross-record comparison, request drafting, or workpaper preparation improves real reviewer outcomes after review cost and failure risk are included.

## Scope contract

Each deployment must publish one signed scope record before data access:

```yaml
engagement_scope_id: es_01J...
tenant_id: tenant_acme
engagement_id: eng_fy26_soc2
purpose: "Prepare evidence and candidate workpapers for the approved examination"
period:
  starts_at: 2026-01-01T00:00:00Z
  ends_at: 2026-12-31T23:59:59Z
profile:
  profile_id: profile_soc2_org_v7
  content_digest: sha256:...
  approved_by: human:profile_steward_42
in_scope_systems: [aws-prod, github-enterprise, okta-workforce]
in_scope_control_ids: [CC6.1-org-01, CC7.2-org-03]
allowed_data_classes: [configuration, identity-metadata, change-metadata]
prohibited_data: [message-body, source-code-content, credentials, payment-card-data]
authority_ceiling: C2
approved_effect_classes: [evidence_request.create, evidence_request.remind]
residency: in-central
retention_policy_id: retain_audit_7y_v3
legal_hold_policy_id: hold_default_v2
independence_policy_id: assurance_sod_v5
expires_at: 2027-03-31T00:00:00Z
signatures:
  - role: engagement_lead
    subject: human:lead_17
  - role: data_owner
    subject: human:owner_09
  - role: privacy_security
    subject: human:risk_11
```

The runtime rejects a missing, expired, unsigned, or digest-mismatched scope. A natural-language instruction cannot expand it.

## Boundary matrix

| Activity | Agent | Control owner | Independent reviewer / qualified professional |
| --- | --- | --- | --- |
| Inventory approved requirements | Read and index | Consulted | Confirms profile and applicability owner |
| Propose control mappings | Draft with rationale and sources | Explains implementation | Approves/rejects mapping |
| Implement or change a control | Prohibited | Accountable through the organization’s change process | May assess later; must not become the operator |
| Request evidence | Orchestrate within approved scope | Submit or identify source | Reviews sensitive/exceptional requests |
| Collect via API | Execute approved read query | Own source and access approval | Evaluates relevance/reliability |
| Choose population/method/sample size | Draft questions only | Supplies source facts | Approves plan and parameters |
| Select approved sample | Deterministic execution | No selection influence after freeze | Verifies manifest |
| Draft TOD/TOE workpaper | Source-linked proposal | Responds to questions | Performs judgment and signs review |
| Classify or close exception | Recommend only | Supplies remediation/response | Authorized human decides |
| Build package | Deterministic assembly | Verifies representations where required | Approves exact manifest |
| Issue assurance conclusion | Prohibited | May make management assertions | Qualified signer owns conclusion |
| Grant access or waive independence conflict | Prohibited | Cannot delegate to agent | Identity/risk/assurance owner decides |

## Accountable role catalog

Small organizations may combine roles only when the applicable independence policy explicitly permits it. Service-account separation is not enough if the same person controls both identities.

| Role | Accountable decisions | Cannot delegate to the model |
| --- | --- | --- |
| **Profile steward** | Licensed source access, applicability owner, profile version, mapping approval | Meaning of an obligation and jurisdictional applicability |
| **Engagement lead** | Objective, period, systems, procedures, risk, deadlines, package recipient | Engagement acceptance and scope changes |
| **Control owner** | Control description, operation, evidence source, management response | Truthfulness of representation and remediation ownership |
| **Evidence custodian** | Source access, extraction method, completeness explanation | Credential issuance and source-system representation |
| **Tester/preparer** | Procedure execution, workpaper preparation, candidate observation | Self-approval when policy requires independent review |
| **Independent reviewer** | Evidence acceptance, deviations, conclusion support, rework | Independence judgment or review sign-off |
| **Assurance/signing professional** | Materiality, sufficiency, report/opinion/attestation | Professional conclusion and issuance |
| **Privacy/security owner** | Data classes, purpose, residency, retention, legal hold, provider use | Exceptions to privacy or security policy |
| **Platform owner** | Runtime, connectors, release manifest, SLOs, kill switch, recovery | Domain conclusion |
| **Incident commander** | Containment, evidence preservation, correction, notification coordination | Business/audit correction approval outside their authority |

## Risk and autonomy assessment

Score each **effect class**, not the whole application.

| Dimension | Lower-risk example | Higher-risk example |
| --- | --- | --- |
| Reversibility | Draft a request | Deliver a frozen package externally |
| Decision consequence | Suggest a missing artifact | State a control is effective |
| Data sensitivity | Public policy metadata | HR, customer, security-event, or legal-hold evidence |
| Target certainty | Stable source ID | Entity inferred from free text |
| Source reliability | Signed API snapshot | Screenshot or narrative assertion |
| Time pressure | Normal request cycle | Regulatory deadline or incident-driven evidence hold |
| Independence risk | Separate preparer/reviewer | Control operator also reviewing the evidence |
| Detectability | Reconciled receipt | Silent omission from a package |
| Blast radius | One engagement draft | Cross-tenant bulk export |

Any high-consequence, low-reversibility, weakly detectable, or independence-sensitive activity stays proposal-only or human-committed.

## Stage 0: qualification and governance

### Authority

No production data access, connector credential, model call on engagement data, or external effect. Research may use public, licensed-for-that-purpose, synthetic, or explicitly approved historical material.

### Required architecture

- a named system of record for scope, profiles, evidence, decisions, and effects;
- a draft trust-boundary and data-flow diagram;
- a role and segregation-of-duties matrix tied to real identity providers;
- a source/licensing register for standards and framework content;
- an initial connector and data-class inventory;
- a model-free fallback and manual operating procedure.

### Inputs and outputs

Inputs are the candidate engagement type, source standards, organizational policies, data inventory, current manual process, known defects, reviewer capacity, and cost baseline. Outputs are the signed scope contract, authority matrix, risk register, source-rights record, target metrics, staged plan, and explicit no-go conditions.

### State, events, and effects

Only governance states are allowed: `candidate`, `qualifying`, `approved_for_offline_replay`, `rejected`, or `deferred`. Events record owner decisions. External operational effects are forbidden.

### Approvals

At minimum: engagement/process owner, profile steward or qualified professional, data owner, privacy/security owner, platform owner, and independence owner. Procurement/legal approval is required when model or connector terms affect data or licensed content.

### Recovery

Because there are no production effects, recovery means deleting or quarantining unauthorized test data, revoking trial credentials, preserving decision records, and correcting the scope before replay begins.

### Evaluation

- compare current manual and deterministic baselines;
- identify at least one measurable model-assisted task and one task that remains deterministic;
- threat-model prompt injection, cross-tenant access, false equivalence, unsupported claims, and independence bypass;
- verify that every in-scope data class has a purpose, owner, retention rule, and allowed provider path;
- walk through cancellation, source unavailability, reviewer overload, and legal hold.

### Measurable exit gate

Stage 0 passes only when:

1. 100% of in-scope control IDs, systems, data classes, effect classes, and accountable roles are represented in the signed scope;
2. every profile source has an owner, version/effective date, rights classification, and refresh trigger;
3. every approved effect class has a named approver, authorization rule, idempotency key, verification source, and rollback/forward-recovery procedure;
4. zero critical privacy, independence, credential, tenant-boundary, or legal-rights questions remain unowned;
5. a deterministic or manual baseline and target thresholds are approved; and
6. the organization can stop the project without losing a required evidence record.

These are completeness gates for the design record, not assertions that the future system is compliant.

## Jurisdiction and assurance-type limits

The same words can carry different obligations in different contexts:

- [PCAOB AS 2201](https://pcaobus.org/oversight/standards/auditing-standards/details/AS2201) addresses integrated audits of internal control over financial reporting. Its design and operating-effectiveness concepts are useful, but they are not a universal compliance recipe.
- [NIST SP 800-53A Revision 5](https://csrc.nist.gov/pubs/sp/800/53/a/r5/final) provides customizable assessment procedures and examine/interview/test methods; tailoring and assurance requirements still belong to the adopting program.
- The [GAO Green Book](https://www.gao.gov/greenbook), [FISCAM](https://www.gao.gov/products/gao-24-107026), and [Yellow Book](https://www.gao.gov/yellowbook) have U.S. government purposes and effective dates.
- [IIA Global Internal Audit Standards](https://www.theiia.org/en/standards/) govern an internal-audit context, not every external examination.
- SOC materials may be licensed and engagement-specific; PCI DSS, FedRAMP, ISO management-system audits, financial audits, and internal audits have different assessor qualifications, evidence conventions, and report language.

Create separate, version-pinned profile overlays. Do not hide differences behind a generic `framework = compliance` field.

## Requirements discovery checklist

### Engagement and professional requirements

- [ ] Assurance type, intended users, period, boundaries, criteria, and report owner are named.
- [ ] Applicability, materiality/risk, sampling, sufficiency, and conclusion authorities are named.
- [ ] Independence restrictions cover people, services, delegated identities, prior control implementation, and conflicts.
- [ ] Required consultations, management representations, review levels, and retention periods are identified.

### Data and connector requirements

- [ ] Every source has a system owner, stable resource identity, retention window, API/export limit, clock behavior, and completeness test.
- [ ] Data classification, residency, purpose, minimization, redaction, legal hold, deletion, and breach obligations are mapped.
- [ ] Connector credentials are read-only where feasible and scoped to tenant, engagement, fields, time range, and resources.
- [ ] Manual evidence has an authenticated submitter, acquisition record, malware handling, and independent acceptance path.

### Operational requirements

- [ ] Evidence-request and exception queues have owners, deadlines, escalation, pause/cancel, and capacity policies.
- [ ] Reviewer capacity and independence are modeled as hard constraints, not optimistic assumptions.
- [ ] Manual fallback, kill switch, release rollback, restore, and reconciliation drills have owners.
- [ ] SLOs measure freshness, queue age, lineage, reconciliation, and review—not only model latency.

## Anti-patterns

| Anti-pattern | Why it fails | Safer design |
| --- | --- | --- |
| “Map us to every framework” | Treats text similarity as legal and control equivalence | Versioned, relationship-typed proposals approved per profile |
| “Ask the model if we are compliant” | No authoritative scope, evidence standard, independence, or professional judgment | Ask for missing evidence or candidate observations; human concludes |
| Same team implements and independently approves its own control evidence | Service-account separation does not create objectivity | Enforce real-person and organizational conflict rules |
| Sample whatever the agent thinks is representative | Selection is irreproducible and can be biased by available evidence | Approve and freeze population/method/seed before selection |
| Copy all source data “for audit” | Violates minimization and expands breach/retention scope | Purpose-limited fields, immutable versions, and explicit completeness limits |
| One global connector credential | Makes tenant and purpose boundaries unenforceable | Short-lived, engagement-scoped delegated access with policy checks |
| Measure only extraction accuracy | Misses omitted evidence, illegal transitions, duplicate effects, reviewer burden, and package corruption | Evaluate the complete control and workflow trajectory |

## Stage 0 handoff

Do not advance because a demo looks persuasive. Advance only with the signed artifacts and exit evidence above. Stage 1 then implements the architecture in [Reference architecture, runtime, and authority](02-reference-architecture-runtime-and-authority.md) and validates it on approved offline/shadow data.
