# Research Packet: Identity and Access Governance Agent Blueprint

> **Research date:** 2026-08-31  
> **Maturity:** Research-backed draft  
> **Method:** Primary standards/specifications, government and regulator guidance, official vendor documentation, foundational papers, and the repository's canonical production guides were compared. Product documentation establishes documented behavior, not independent effectiveness.  
> **Scope boundary:** Entitlement/relationship evidence, JML monitoring, review preparation, SoD analysis, time-bound grant proposals, approval routing, revocation verification, and orphan reconciliation. Identity recovery, conversational identity inference, compliance attestation, and unilateral privileged access are excluded.

## Questions investigated

1. Is identity and access governance a distinct agent workload, or should existing IGA/workflow/rules products own it entirely?
2. Which identity, entitlement, relationship, policy, decision, approval, effect, and verification records must remain deterministic and authoritative?
3. What do SCIM and shared-signal standards establish, and what do they leave implementation/profile specific?
4. How should heterogeneous vendor connectors handle pagination, events, deletion, nesting, eventual consistency, idempotency, and reconciliation?
5. Which parts of JML, access reviews, SoD, time-bound grants, and orphan remediation benefit from a model?
6. How should context, memory, planning, durability, security, privacy, tenancy, evaluation, observability, scale, cost, incidents, and upgrades be specialized for this workload?
7. What evidence would justify expanding from read-only preparation to one approved effect class?

## Research method and saturation

Research followed the repository's [research method](../research-method.md): frame failure and decision questions, establish a primary-source baseline, compare documented product mechanics, inspect foundational formal/production work, classify claim strength, resolve apparent contradictions by version/profile/scope, and stop only when new sources no longer changed the architecture, authority boundary, failure model, evaluation suite, or staged roadmap.

Search angles included:

- NIST identity, access-control, zero-trust, privacy, AI-risk, assessment, and incident guidance;
- SCIM core/protocol and 2025–2026 updates; OAuth security/delegation; OpenID Shared Signals; XACML;
- RBAC, ABAC, ReBAC/relationship graph, static/dynamic SoD, and access-review usability;
- Microsoft Entra, AWS IAM/IAM Identity Center/Access Analyzer, Okta Identity Governance, SailPoint Identity Security Cloud, and Google Cloud documented mechanics and limits;
- CISA/NSA, NCSC, GAO, GDPR, PCI SSC, and jurisdiction/profile boundaries;
- the repository's state, durability, tool, provenance, context, memory, compaction, planning, security, reliability, evaluation, observability, queueing, SLO, release, and incident guidance.

Saturation was reached for a provider-neutral blueprint. It was not reached for any particular organization's connectors, legal duties, SoD policies, target propagation, reviewer behavior, or scale; those require implementation discovery and tests.

## Executive findings

1. **The category is distinct but should not replace IGA.** Existing IGA/IAM/PAM products remain authoritative for lifecycle, approval, policy, and provisioning where deployed. A bounded model is useful for heterogeneous evidence synthesis, effective-path explanation, exception triage, and review-packet preparation.
2. **Identity cannot come from conversation.** Stable authenticated and authoritative bindings establish actors and subjects. NIST SP 800-63-4 informs proofing/authentication/federation but does not authorize a governance model to perform help-desk recovery.
3. **The entitlement graph is a versioned evidence projection.** Direct, inherited, eligible, active, effective, asserted, derived, current, and historical relationships must remain distinguishable and source-backed.
4. **SCIM is not a complete IGA contract.** RFCs 7643/7644 standardize resource schemas and operations; RFC 9865 adds optional cursor pagination and RFC 9967 adds optional SCIM provisioning events/async behavior. Organizational roles, SoD, approvals, correlation, and end-to-end revocation proof remain outside those standards.
5. **Vendor profiles materially differ.** Official AWS and Microsoft documentation shows partial filters/attributes/endpoints, nested-group constraints, attribute-removal caveats, and timing/scale behavior. Capability discovery plus conformance, workload, and reconciliation tests are mandatory.
6. **Events reduce latency; snapshots prove coverage.** Shared Signals and SCIM SETs are useful change signals. They can be duplicated, delayed, reordered, or absent and may convey only a notice. Persist/deduplicate them and confirm current state; retain scheduled full reconciliation.
7. **JML policy is deterministic and human-accountable.** The model may identify mismatches and prepare proposals. HR/sponsor sources establish affiliation facts; policy/owners decide access; IAM/PAM enforces.
8. **Review preparation is not attestation.** The agent supplies actionable access roots, effective paths, source cutoffs, activity limitations, SoD results, and gaps. The assigned reviewer makes the decision; independent audit/compliance functions assess controls.
9. **SoD rules must be versioned control-owner policy.** Analyze effective cross-system paths and all subject types. Standing graph analysis can identify potential dynamic conflicts but cannot prove transaction-level dynamic SoD enforcement.
10. **Time-bound grants need an enforced expiry and verification plan.** Friendly bundle names are insufficient; disclose inherited permissions. Privileged and authority-changing grants remain proposal-only through PAM/administrative controls.
11. **Revocation is a target-state outcome.** A queue acceptance, workflow completion, HTTP success, SCIM response, or IGA job status alone does not prove access is gone. Keep `unknown` and `partial` states and verify alternate paths, propagation, and session/token ownership.
12. **Uncorrelated does not equal orphaned.** Service, shared, emergency, vendor, and intentionally owner-separated accounts require type, purpose, owner/sponsor, activity coverage, and dependency analysis before action.
13. **Most “memory” should be disabled.** Durable typed case state and a governed graph are required. Long-term personalization and live episodic precedent create shadow policy, poisoning, privacy, and bias risk; keep redacted failures in an offline evaluation corpus instead.
14. **One workflow coordinator is the default.** Open-ended planning and department-role agents add failure without measured value. The model gets a bounded read plan and closed completion set.
15. **Read-only compromise is high impact.** The graph maps privileged people, systems, and lateral paths. Tenant, purpose, field, path, row, egress, credential, telemetry, evaluation, and support-access boundaries are essential even before mutation.
16. **Release and evaluation must separate behavior from authority.** A model, prompt, compiler, connector, graph, policy, or tool upgrade does not automatically inherit production approval. Shadow first; promote effect cells independently.
17. **Operational bottlenecks include people and providers.** Connector quotas/propagation, graph fan-out, model capacity, approval staffing, and reconciliation all bind throughput. Priority lanes must protect critical leaver, expiry, and unknown-effect work.
18. **Canonical identity and effective time are control surfaces.** Person, subject, workload, service account, account, group, role, entitlement, grant, assignment, package, credential, session, decision, and effect are not synonyms. Decisions must preserve both business-effective time and what evidence was known at the cutoff.
19. **There are exactly seven memory lifetimes.** Turn/scratch, Working/run, Session, Durable workflow/task, Domain knowledge, Long-term/preference, and Episodic/outcome each need an explicit use/reject decision, retention, correction/deletion path, and poisoning test. The last two are disabled for live authorization by default.
20. **Provider workflow status is not portable revocation proof.** Entra, Okta, and SailPoint document different review, assignment-origin, remediation, aggregation, duplicate, session, and timing behavior. Typed provider adapters and live-tenant qualification are required before any effect cell is enabled.

## Evidence and decision records

The access date for every record below is 2026-08-31 unless a publication date is part of the boundary.

| # | Claim or decision | Class | Strongest evidence | Limitation / conflicting evidence | Blueprint consequence |
| ---: | --- | --- | --- | --- | --- |
| 1 | Account management, SoD, and least privilege are explicit controls, not model judgments | Mechanic / recommendation | [NIST SP 800-53 Rev. 5](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final), especially AC-2/AC-5/AC-6 | NIST control catalog is tailorable and not automatically applicable outside its adoption context; publication remains Rev. 5 while NIST's 2025 supplemental release is 5.2.0 | Deterministic policy/authorization and accountable owners remain outside the model |
| 2 | Control assessment and evidence preparation are different from control operation/attestation | Mechanic / boundary | [NIST SP 800-53A Rev. 5](https://csrc.nist.gov/pubs/sp/800/53/a/r5/final), [GAO 2025 Green Book](https://www.gao.gov/greenbook) | Assessment methods and legal/audit independence depend on adopted framework/jurisdiction | Agent prepares evidence; compliance/audit category attests |
| 3 | Identity proofing/authentication/federation require explicit risk-managed processes | Mechanic | [NIST SP 800-63-4](https://csrc.nist.gov/pubs/sp/800/63/4/final), final July 2025 | Government online-service guidance is not a universal workforce-IAM mandate; it does not cover IGA policy | Exclude help-desk recovery and conversational identity inference |
| 4 | Zero trust rejects implicit trust based on location/ownership and focuses on resources and explicit identity | Mechanic / recommendation | [NIST SP 800-207](https://csrc.nist.gov/pubs/sp/800/207/final), [SP 800-207A](https://csrc.nist.gov/pubs/sp/800/207/a/final) | Zero trust is an architecture approach, not a product or direct IGA workflow specification | Reauthorize each read/effect; separate user/workload/downstream identities |
| 5 | ABAC evaluates subject/object/action/environment attributes against policy | Mechanic | [NIST SP 800-162](https://csrc.nist.gov/pubs/sp/800/162/upd2/final) | Attribute quality/freshness and policy administration remain deployment problems | Keep attributes sourced/versioned; use deterministic policy engine |
| 6 | RBAC formally includes role hierarchies and SoD concepts | Mechanic / foundational | [NIST IR 6192](https://csrc.nist.gov/pubs/ir/6192/final), [NIST RBAC history](https://csrc.nist.gov/nist-cyber-history/identity-access-management/chapter), [NIST mutual-exclusion paper](https://tsapps.nist.gov/publication/get_pdf.cfm?pub_id=916538) | Role models can hide direct/resource/attribute/relationship paths and may drift | Preserve role/permission assignments and static/dynamic SoD distinctions |
| 7 | Relationship tuples and consistency can support large authorization graphs | Observed result / foundational | [Zanzibar paper](https://www.usenix.org/conference/atc19/presentation/pang) | Google's reported scale/latency/availability is workload-specific; the paper is authorization, not enterprise IGA | Use relationship paths conceptually; do not copy architecture or claims without benchmark |
| 8 | Conditional relationship tuples can express time-bound/context conditions in one current implementation | Mechanic / product-specific | [OpenFGA conditions](https://openfga.dev/docs/modeling/conditions), [OpenFGA concepts](https://openfga.dev/docs/concepts) | Product semantics/version are implementation-specific; using it is optional | A graph engine is a choice, not a requirement; time expiry still needs target enforcement |
| 9 | XACML standardizes policy language/decision architecture; it is not the only viable policy engine | Mechanic | [OASIS XACML 3.0 plus errata](https://www.oasis-open.org/standard/xacmlv3-0/), [XACML RBAC profile](https://www.oasis-open.org/standard/xacml3-0-core/) | Mature but not universally adopted; product interoperability and policy ergonomics vary | Require a versioned deterministic PDP contract, not XACML specifically |
| 10 | SCIM core models extensible Users/Groups and the protocol provides CRUD/query/discovery operations | Mechanic | [RFC 7643](https://www.rfc-editor.org/rfc/rfc7643.html), [RFC 7644](https://www.rfc-editor.org/rfc/rfc7644.html) | Point-to-point schema/protocol does not standardize enterprise roles, SoD, approvals, or correlation | Profile/test each connector; avoid calling SCIM “full IGA interoperability” |
| 11 | Cursor pagination became a standards-track SCIM update in October 2025 | Mechanic / maturity | [RFC 9865](https://www.rfc-editor.org/rfc/rfc9865.html) | Capability is optional; providers/clients may use index, cursor, or both | Discover pagination, persist opaque cursor, bind identical query, test expiry |
| 12 | SCIM SET events and optional async completion became a proposed standard in May 2026 | Mechanic / emerging | [RFC 9967](https://www.rfc-editor.org/rfc/rfc9967.html) | New adoption surface; event configuration is optional and events can be notice-only | Treat events as hints, persist before acknowledge, retrieve state and reconcile |
| 13 | Shared Signals Framework 1.0 became final in August 2025 | Mechanic / maturity | [OpenID SSF 1.0 Final](https://openid.net/specs/openid-sharedsignals-framework-1_0-final.html), [CAEP 1.0 Final](https://openid.net/specs/openid-caep-1_0-final.html) | Final specification does not imply provider support or complete provisioning semantics | Use only after capability/conformance review; do not replace inventory reconciliation |
| 14 | OAuth current BCP deprecates weaker patterns and recommends stronger client/token protections | Mechanic | [RFC 9700](https://www.rfc-editor.org/rfc/rfc9700.html) | Provider support for sender constraint/asymmetric auth and token scopes varies | Broker audience/resource/tenant-scoped connector credentials outside model context |
| 15 | OAuth token exchange defines delegation/impersonation mechanics but leaves authorization decisions to deployment | Mechanic | [RFC 8693](https://www.rfc-editor.org/rfc/rfc8693.html) | Token exchange can amplify stolen tokens if clients/targets are poorly restricted | Do not treat token exchange as delegation proof; application policy remains decisive |
| 16 | Vendor SCIM profiles can implement documented subsets | Mechanic / product-specific | [AWS IAM Identity Center SCIM limitations](https://docs.aws.amazon.com/singlesignon/latest/developerguide/limitations.html) | Documentation can change by product release/region; another provider differs | Connector registry records exact supported filters/endpoints/attributes/group behavior |
| 17 | Provisioning through a central IdP can have nested-group, deletion, mapping, and timing constraints | Mechanic / product-specific | [Microsoft provisioning mechanics](https://learn.microsoft.com/en-us/entra/identity/app-provisioning/how-provisioning-works), [known SCIM issues](https://learn.microsoft.com/en-us/entra/identity/app-provisioning/application-provisioning-config-problem-scim-compatibility), [Entra-to-AWS considerations](https://docs.aws.amazon.com/singlesignon/latest/userguide/idp-microsoft-entra.html) | Limits are flow/product-specific, not universal Microsoft or SCIM behavior | Test actual end-to-end profile and never infer deletion/attribute removal |
| 18 | IGA sources may support read-only aggregation separately from provisioning and different aggregation modes | Mechanic / product-specific | [SailPoint source configuration](https://documentation.sailpoint.com/saas/help/sources/config_sources.html), [account aggregation](https://documentation.sailpoint.com/saas/help/accounts/loading_data.html) | Product feature/licensing/configuration varies | Prefer read-only first; explicitly enable/segregate write connectors; full vs delta decision recorded |
| 19 | JML is a recognized identity-management process and should include non-employees/third parties | Recommendation | [NCSC IAM guidance](https://www.ncsc.gov.uk/collection/10-steps/identity-and-access-management), [NCSC SaaS guidance](https://www.ncsc.gov.uk/collection/cloud/using-cloud-services-securely/using-saas-securely) | UK guidance is not a universal legal frequency/implementation mandate | Build source-authoritative JML monitoring, including guests/service identities where in scope |
| 20 | Lifecycle products derive access changes from authoritative lifecycle states but have retries/exceptions and product-specific semantics | Mechanic / product-specific | [SailPoint lifecycle states](https://documentation.sailpoint.com/saas/help/provisioning/lifecycle.html), [Microsoft lifecycle workflows hub](https://learn.microsoft.com/en-gb/entra/id-governance/) | State names, retries, precedence, manual override, and provisioning differ | Do not hard-code vendor state semantics into the canonical model |
| 21 | Entitlement-management products can combine policies, approvals, expiration, reviews, and bundles | Mechanic / product-specific | [Microsoft Entra entitlement management](https://learn.microsoft.com/en-us/entra/id-governance/entitlement-management-overview) | Supported resource types/features/licensing and preview status change | Prefer existing product; model prepares an exact cataloged proposal, not a novel entitlement |
| 22 | Review scope/remediation differs by product and assignment origin | Mechanic / contradiction | [Microsoft access-review deployment](https://learn.microsoft.com/en-us/azure/active-directory/governance/deploy-access-reviews), [Okta campaigns](https://help.okta.com/oie/en-us/Content/Topics/identity-governance/access-certification/campaigns.htm), [SailPoint certification behavior](https://documentation.sailpoint.com/saas/help/certs/understanding_certifications.html) | Leaf access can be encapsulated by roles/lifecycle; automatic remediation and self-review behavior differ | Canonical review item exposes revocation root, effective leaves, origin, and product action semantics |
| 23 | Access-review quality depends on context and human factors | Observed result / foundational | [USENIX SOUPS access-review study](https://www.usenix.org/conference/soups2014/proceedings/presentation/jaferian) | 2014 study and tested interfaces do not establish current universal effect sizes | Measure reviewer comprehension/bias locally; provide evidence/history/paths, not transcript |
| 24 | Recommendation systems in IGA products are advisory inputs | Mechanic / inference | [SailPoint access recommendations](https://documentation.sailpoint.com/saas/help/ai/access_recs/recommendations.html), [Microsoft review recommendations context](https://learn.microsoft.com/en-us/azure/active-directory/governance/deploy-access-reviews) | Vendor methods and reviewer behavior differ; peer patterns can encode historic privilege | Keep recommendation separate from evidence and decision; test automation bias |
| 25 | Unused-access analysis is useful but has resource coverage and analysis latency/limits | Mechanic / product-specific | [AWS IAM Access Analyzer concepts](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-concepts.html), [findings](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-findings.html) | Policy-derived possible access and activity-derived unused access answer different questions; product scope/price/limits change | Usage is one bounded signal; lack of use is not automatic revocation proof |
| 26 | Stale/inactive account heuristics require context | Mechanic / recommendation | [Microsoft inactive accounts](https://learn.microsoft.com/en-us/entra/identity/monitoring-health/howto-manage-inactive-user-accounts), [stale guest cleanup](https://learn.microsoft.com/en-us/entra/identity/users/clean-up-stale-guest-accounts) | Activity windows and legitimate inactivity vary; service identities behave differently | Show observation coverage; route to review; do not auto-delete unmatched/inactive accounts |
| 27 | Least privilege and regular review should include service/system identities | Recommendation | [CISA/NSA IAM administrator guidance](https://www.cisa.gov/sites/default/files/2023-12/ESF%20IDENTITY%20AND%20ACCESS%20MANAGEMENT%20RECOMMENDED%20BEST%20PRACTICES%20FOR%20ADMINISTRATORS%20PP-23-0248_508C.pdf), [NCSC CAF B2](https://www.ncsc.gov.uk/collection/cyber-assessment-framework/caf-objective-b/principle-b2-identity-and-access-control) | Guidance profiles and review frequencies differ by adoption/risk | Model person and non-person identities distinctly; require owners/sponsors and coverage |
| 28 | Privileged access requires stronger separation and monitoring | Recommendation / product-specific | [Microsoft secure admin practices](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/security-planning), [Entra privileged review](https://learn.microsoft.com/en-us/entra/id-governance/privileged-identity-management/pim-create-roles-and-resource-roles-review) | Microsoft product specifics are not universal PAM requirements | I5 proposal-only; route privileged access to PAM/security owners |
| 29 | Revocation can leave alternate paths and sessions even after one assignment changes | Engineering inference | NIST least privilege/SoD, vendor nested/profile docs, graph/authorization papers | No universal protocol defines end-to-end revocation across heterogeneous systems | Define postcondition per mechanism; recompute effective paths; name session/token owner |
| 30 | Durable execution does not remove external-effect ambiguity | Engineering inference / recommendation | Repository [durable execution](../../runtime/durable-execution.md), [idempotency and side effects](../../reliability/idempotency-and-side-effects.md), official connector async/status mechanics | Provider-specific idempotency and status can strengthen one operation but not every target | Persist semantic operation ID; `unknown`; reconcile before retry |
| 31 | Context must be a purpose-limited projection, not a full entitlement dump | Recommendation | Repository [context engineering](../../context-memory/context-engineering.md), NIST least privilege/privacy guidance | Optimal projection size is workload-specific | Compile typed lanes with source freshness, omissions, row/path/token budgets |
| 32 | Long-term personalization/episodic precedent is not justified by default | Recommendation | Repository [memory architecture](../../context-memory/memory-architecture.md), privacy and control evidence above | A specialized deployment could prove value under governance | Disable live long-term/episodic memory; retain durable task/domain state and offline failures |
| 33 | Sensitive content must not be default telemetry | Recommendation | [W3C Trace Context](https://www.w3.org/TR/trace-context/), [OpenTelemetry GenAI conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/), repository [observability guide](../../evaluation/observability-and-tracing.md) | GenAI conventions are evolving and backend defaults differ | Stable application event schema; content off by default; audit separate from traces |
| 34 | Privacy duties cover derived copies, not only source systems | Mechanic / jurisdiction-limited | [GDPR consolidated text](https://eur-lex.europa.eu/eli/reg/2016/679/2016-05-04), [NIST Privacy Framework](https://www.nist.gov/privacy-framework) | GDPR applicability/lawful basis/rights and other laws require counsel; NIST PF is voluntary | Data map includes prompts, graph, artifacts, caches, evals, logs, backups; no compliance claim |
| 35 | Current regulatory/industry profiles differ on access review frequency and evidence | Mechanic / jurisdiction-profile limit | [NCSC CAF B2](https://www.ncsc.gov.uk/collection/cyber-assessment-framework/caf-objective-b/principle-b2-identity-and-access-control), [PCI DSS v4.0.1 library](https://www.pcisecuritystandards.org/document_library/?class=pcidss&doc=pci_dss), [GAO Green Book](https://www.gao.gov/greenbook) | These apply only in their respective scopes and do not create a universal annual/quarterly rule | Policy stores jurisdiction/profile, resource population, frequency, owner, evidence requirement |
| 36 | Evaluation must separate deterministic controls, model judgment, effects, human review, and operations | Recommendation | [NIST AI RMF Core](https://airc.nist.gov/airmf-resources/airmf/5-sec-core/), [NIST AI 600-1](https://doi.org/10.6028/NIST.AI.600-1), repository evaluation guides | Neither NIST document supplies workload-specific thresholds | Use separate hard gates, local task data, repeated/failure/online evaluation |
| 37 | SLOs should reflect business deadlines and error budgets | Recommendation | [Google SRE SLO chapter](https://sre.google/sre-book/service-level-objectives/), [SLO alerting workbook](https://sre.google/workbook/alerting-on-slos/) | SRE examples are not IAM thresholds | Define source freshness, JML/expiry, verification, unknown-effect, review-queue SLOs locally |
| 38 | Incident response must include containment, recovery, and improvement | Mechanic / recommendation | [NIST SP 800-61 Rev. 3](https://csrc.nist.gov/pubs/sp/800/61/r3/final) | General incident guidance does not specify target access correction | Add connector/release quarantine, impact query, target reconciliation, approved repair, failure mining |
| 39 | OpenID Connect identifies an authenticated end user by issuer and subject for a relying party; it is not an account correlation or provisioning protocol | Mechanic / boundary | [OpenID Connect Core 1.0 incorporating errata set 2](https://openid.net/specs/openid-connect-core-1_0.html) | Pairwise/public subject configuration, issuer, audience and client differ; an OIDC subject is not automatically an application's local account ID | Bind interactive actor by exact issuer × subject and resolve beneficiary/target account separately |
| 40 | Token revocation and session/application revocation are different outcomes | Mechanic / product-specific | [RFC 7009](https://www.rfc-editor.org/rfc/rfc7009.html), [Microsoft emergency access revocation](https://learn.microsoft.com/en-us/entra/identity/users/users-revoke-access) | RFC 7009 intentionally returns success even for an invalid token and allows implementation-specific cascades; Microsoft states application-issued sessions are application-owned and access may persist until token/session expiry | Define R0–R5 evidence levels, residual lifetime and issuer/application/PAM ownership; never claim global logout from one receipt |
| 41 | Entra review decision, result application, directory change, downstream provisioning and session expiry are separate stages | Mechanic / product-specific | [Entra access-review FAQ](https://learn.microsoft.com/en-us/entra/id-governance/access-reviews-faqs), [application review preparation](https://learn.microsoft.com/en-us/entra/id-governance/access-reviews-application-preparation), [complete an access review](https://learn.microsoft.com/en-us/entra/id-governance/complete-access-review) | Current docs say decisions can wait until scheduled end; nested or on-premises-origin access can remain; downstream app/session behavior depends on integration. Tenant configuration, cloud, license and propagation were not tested | Entra adapter preserves review/decision/apply/assignment/provisioning/session states and verifies each declared target |
| 42 | Entra provisioning uses initial/incremental cycles and product-specific disable/delete semantics; on-demand coverage is narrower | Mechanic / product-specific | [How Entra provisioning works](https://learn.microsoft.com/en-us/entra/identity/app-provisioning/how-provisioning-works), [on-demand provisioning limitations](https://learn.microsoft.com/en-us/entra/identity/app-provisioning/provision-on-demand), [ID Governance service limits](https://learn.microsoft.com/en-us/entra/id-governance/governance-service-limits) | Service limits and intervals can change or be raised; gallery mappings and target soft-delete semantics differ; no live job/cursor was observed | Record exact tenant capability/profile, watermark and target behavior; do not use on-demand success as full reconciliation |
| 43 | Okta campaign remediation depends on assignment origin and some `Successful` report states do not mean an automatic target removal occurred | Mechanic / product-specific | [Okta campaign limits](https://help.okta.com/oie/en-us/content/topics/identity-governance/access-certification/ac-get-started.htm), [remediation behavior](https://help.okta.com/oie/en-us/content/topics/identity-governance/access-certification/remediation.htm), [past campaign report fields](https://help.okta.com/oie/en-us/content/topics/identity-governance/columns-campaign-details.htm) | Group rules, app-sourced groups, resource collections, entitlements and service accounts have different/manual paths; feature flags, subscriptions and tenant configuration were not tested | Resolve grant origin and attempted remediation; verify source and downstream target instead of trusting campaign summary |
| 44 | Okta event hooks are best-effort, at-least-once, can be delayed/out of order, and have a bounded retry | Mechanic / product-specific | [Okta event-hook concepts](https://developer.okta.com/docs/concepts/event-hooks/), [hook best practices](https://developer.okta.com/docs/guides/hooks-best-practices/) | Current official docs state at most one retry and no maximum delivery delay; daily/feature limits can change and were not tenant-tested | Persist/deduplicate by event ID, order by source metadata, poll authoritative APIs/System Log and retain full reconciliation |
| 45 | Okta exposes granular OAuth scopes for governance APIs, while separate legacy API-token guidance recommends a long-lived super-admin service account for continuity | Mechanic / documented contradiction | [Okta Governance OAuth scopes, updated 2026-05-15](https://developer.okta.com/docs/api/iga/oauth2), [Okta API-token management](https://help.okta.com/oie/en-us/content/topics/security/api.htm) | Endpoint coverage, Beta status and org subscription vary. The continuity recommendation conflicts with least-privilege blast-radius goals | Prefer a scoped OAuth service application where supported; isolate any unavoidable legacy token and do not generalize vendor continuity advice into architecture |
| 46 | SailPoint access-request submission is asynchronous and may not reject rapid duplicates | Mechanic / product-specific | [SailPoint submit access request](https://developer.sailpoint.com/docs/api/v3/create-access-request/), [access-request status/API behavior](https://developer.sailpoint.com/docs/tools/sdk/go/accessrequests/methods/access-requests/) | API versions and experimental endpoints differ; current docs recommend pre-query but do not provide a universal provider idempotency guarantee; no tenant call was made | Reserve a semantic operation ID, query current/pending access before submit, treat acceptance as queued, and reconcile duplicate provider objects |
| 47 | SailPoint delta/single-account aggregation and direct model objects do not provide universal deletion or revocation proof | Mechanic / product-specific | [SailPoint account loading](https://documentation.sailpoint.com/saas/help/accounts/loading_data.html), [access-profile behavior](https://documentation.sailpoint.com/saas/help/access/access-profiles.html), [certification behavior](https://documentation.sailpoint.com/saas/help/certs/understanding_certifications.html) | Current docs say account deletions are processed on full aggregation and role/profile overlaps or automated assignment can change revocability; connector and tenant configuration vary | Select the real revocation root, use full aggregation where required, preserve manual remediation, and verify target/effective paths |
| 48 | SailPoint official pages currently disagree on the default API gateway rate-limit key | Mechanic / documented contradiction | [Current rate-limit page](https://developer.sailpoint.com/docs/api/rate-limit/) says `client_id` × API version; [getting-started page](https://developer.sailpoint.com/docs/api/getting-started/) says access token | Documentation can be revised; specific APIs may differ; no live headers/load test were captured | Use response headers and conservative tenant tests, store documentation revision, and avoid hard-coded global quota assumptions |
| 49 | Connector/model/workflow dependencies and administrative artifacts are supply-chain attack surfaces | Recommendation | [NIST SP 800-218 SSDF 1.1](https://csrc.nist.gov/pubs/sp/800/218/final), [NIST SP 800-161 Rev. 1 Update 1](https://csrc.nist.gov/pubs/sp/800/161/r1/upd1/final) | General guidance does not attest a specific artifact or supplier | Pin and inventory artifacts, protect build/release identities, require provenance/review/tests, and support quarantine/impact query/rollback |

## Resolved tensions and contradictions

### Standards-compliant versus interoperable enough

SCIM core/protocol, cursor pagination, and event profiles are complementary standards. A provider can legitimately support a limited workload/profile, extensions, or only older surfaces. Vendor documentation demonstrates that filters, endpoints, multivalued fields, group membership, nesting, role provisioning, deletion, async events, and pagination cannot be assumed from “SCIM 2.0.”

**Resolved position:** record a connector capability/profile contract, pin documentation/version, exercise conformance and workload fixtures, and run full reconciliation. Do not label a vendor categorically compliant/noncompliant from one unsupported optional feature.

### Event-driven lifecycle versus periodic inventory

Events reduce latency and can carry transaction/version information. RFC 9967 itself distinguishes notices from full data and supports callback retrieval; event delivery specifications have persistence/recovery/security considerations. Events do not prove absence or complete coverage.

**Resolved position:** events open/coalesce cases and trigger reads. Durable event persistence, deduplication, source-version handling, and periodic full inventory remain required.

### Central IGA state versus target-system truth

IGA products coordinate access and may expose successful provisioning/remediation status. Targets can have local assignments, delayed propagation, sessions, multiple access paths, or connector blind spots.

**Resolved position:** IGA owns workflow/decision/provisioning status; each target owns its actual account/assignment state. Completion requires effect-specific target verification and declared coverage.

### OAuth/OIDC identity versus governance identity

OIDC establishes an end-user subject for an issuer and relying party; OAuth grants an API client bounded access. Neither protocol proves that a workforce person is the same object as an ERP account, defines an entitlement, approves access, or proves provisioning/revocation.

**Resolved position:** bind the initiating actor by validated issuer × subject and audience, use a separate authoritative correlation pipeline for beneficiary and target accounts, and treat connector OAuth credentials as workload authority with narrow audience/scope. Never use email/display name or an ID token as the mutation key.

### Review completed versus access revoked

Current Microsoft, Okta, and SailPoint documentation gives different meanings to reviewer submission, campaign completion, result application, automated/manual remediation, provisioning completion and target/session effects. Assignment origin can make the apparent review leaf non-revocable.

**Resolved position:** preserve provider-native states but normalize them only into `decision_recorded`, `effect_accepted`, `direct_state_observed`, `effective_paths_cleared`, and `session_token_handled`. Close the governance effect at the highest evidenced revocation level, with exceptions for lower levels.

### Vendor documentation contradiction versus live behavior

Official pages can lag, contradict adjacent pages, describe a different API version, or assume a feature/license/configuration. The SailPoint rate-limit key and Okta legacy-token-versus-scoped-OAuth guidance are concrete examples; neither can be resolved by choosing the more convenient sentence.

**Resolved position:** record page revision/access date and contradiction, configure to the least authority/conservative limit, test the actual tenant and response headers, and make capability promotion an accountable connector-release decision.

### Automatic remediation versus accountable review

Products can auto-apply review outcomes and lifecycle policy. That does not make a model recommendation an authorized decision, nor does a human click always prove meaningful review.

**Resolved position:** deterministic pre-authorized policy may automate an approved low-risk class. Model output remains advisory. High-risk/privileged access uses meaningful independent approval; reviewer utility and automation bias are evaluated.

### RBAC, ABAC, ReBAC, and graph databases

RBAC maps users/permissions through roles; ABAC evaluates attributes; relationship systems evaluate typed tuples/paths; real enterprises use mixtures plus resource policies and local accounts. No model alone solves incomplete source data or policy governance.

**Resolved position:** canonical graph represents facts and derivations across models; a deterministic PDP evaluates the organization's versioned policy. Choose relational, graph, or product storage by tested query/operational needs.

### Static versus dynamic SoD

Formal RBAC work distinguishes assignment/activation constraints. Enterprise SoD can also span systems and transaction steps. A graph of standing assignments can show potential reach but not prove which roles/capabilities were activated in one business transaction.

**Resolved position:** enforce static assignment conflicts at governance time; surface dynamic potential; enforce transaction-level dynamic SoD in the relevant runtime/PAM/business system.

### Current policy versus reproducible historical decision

Audit reconstruction needs the historical graph/policy/approval. Safety requires current revocations and tighter policy at commit. Re-evaluating old model outputs during replay rewrites history.

**Resolved position:** preserve historical versions and typed decisions; revalidate current safety/authorization before pending effects; explicitly migrate active cases; never replay model calls to recreate historical decisions.

### Inactivity as evidence versus decision

Vendor tools expose last-use/inactivity and unused-permission findings, but coverage, retention, resource support, seasonal work, break-glass, and service identities limit inference.

**Resolved position:** activity is one sourced observation with coverage metadata. It can prioritize review but not prove access unnecessary.

### Rich context versus privacy/reconnaissance

Reviewers need useful job, access, path, and history context. Full graph/prompt capture exposes sensitive organizational security information and can overwhelm model/reviewer attention.

**Resolved position:** purpose-limited evidence-first packets, bounded paths, structured omissions, reference-based artifacts, content-off telemetry, and local human evaluation.

## Workload-specific reference decisions

| Area | Selected default | Rejected default | Adoption test / change signal |
| --- | --- | --- | --- |
| Product strategy | Extend existing IGA/IAM/PAM plus durable workflow and bounded analyst | Replace IGA with autonomous agent | Change only if current product cannot meet measured workflow/control needs |
| Controller | Deterministic state machine/durable workflow | Conversational loop as process owner | Model-directed planning must show material benefit on bounded exception evals |
| Model authority | Read-only evidence synthesis and typed proposals | Identity/policy/approval/effect authority | No new authority without hard-gate and incident evidence |
| Graph store | Relational/materialized or current IGA analytics first | Dedicated global ReBAC system by default | Benchmark path depth/fan-out/latency/consistency/ops before adoption |
| Connector posture | Official supported API/profile; read/write split; profile tests | Browser mutation or generic admin tool | Legacy read-only RPA only with temporary exception and reconciliation |
| Signals | Event/delta for latency plus periodic full snapshots | Event-only or poll-only by ideology | Tune by provider retention/quota/change rate and convergence tests |
| Memory | Exactly seven named lifetimes: Turn/scratch, Working/run, Session, Durable workflow/task, Domain knowledge, Long-term/preference, Episodic/outcome; last two disabled for live authority by default | Ad hoc “short/long-term memory” or live case-precedent retrieval | Every lifetime needs use/reject, retention, correction/deletion, poisoning and value tests |
| Multi-agent | Disabled | Department-role swarm | Enable only for real isolation/tool ownership with measured handoff benefit |
| Approval | Exact, expiring, version-bound, independent | “Human in loop” generic confirmation | Adjust per effect cell and reviewer-capacity evidence |
| Effects | Proposal-only first; one low/non-privileged I3 cell at v1 | Broad directory admin or privileged grants | Expand cell-by-cell after duplicate/unknown/verification/incident gates |
| Privilege | PAM/security administrative path, agent proposal-only | Agent grants/administers privilege | No maturity stage makes I5 autonomous by default |
| Compliance | Evidence export only | Agent attestation or guarantee | Independent compliance/audit review owns conclusion |

## Derived guide set

| Guide | Specialized content |
| --- | --- |
| [Blueprint README](../../agents/identity-access-governance-agent/README.md) | Production position, category boundary, invariants, authority, lifecycle, guide map |
| [Boundaries, authority, and stages 0–6](../../agents/identity-access-governance-agent/01-boundaries-authority-and-zero-to-production.md) | Non-agent baseline, authority progression, every-stage architecture/state/approval/recovery/eval/exit gates |
| [Reference architecture, connectors, and entitlement graph](../../agents/identity-access-governance-agent/02-reference-architecture-connectors-and-entitlement-graph.md) | Build/product choice, canonical identity/time ontology, typed adapters, SCIM/OAuth/OIDC, Entra/Okta/SailPoint qualification and worked effect chains, graph/evidence/correlation contracts |
| [State, context, memory, and orchestration](../../agents/identity-access-governance-agent/03-state-context-memory-and-orchestration.md) | Typed state/events/effects, exactly seven memory lifetimes, context lanes, restart-safe compaction receipt, bounded planning |
| [JML, reviews, SoD, and time-bound access](../../agents/identity-access-governance-agent/04-jml-access-reviews-sod-and-time-bound-access.md) | Core workflows, review packets, policy-owned SoD, proposals, orphan classification |
| [Approvals, effects, reconciliation, and recovery](../../agents/identity-access-governance-agent/05-approvals-effects-reconciliation-and-recovery.md) | Exact delegation, commit checks, effect state, revocation proof, ambiguity, runbooks |
| [Security, privacy, identity, and tenancy](../../agents/identity-access-governance-agent/06-security-privacy-identity-and-tenancy.md) | Identity separation, credential broker, threats, injection, tenancy, privacy, audit |
| [Evaluation, observability, and failure injection](../../agents/identity-access-governance-agent/07-evaluation-observability-and-failure-injection.md) | Multi-plane eval, hard gates, reviewer testing, traces/SLOs, extensive fault suite |
| [Deployment, scale, operations, cost, and evolution](../../agents/identity-access-governance-agent/08-deployment-scale-operations-cost-and-evolution.md) | Cells/queues/backpressure, degradation, capacity, releases/upgrades, incidents/DR/cost/drift |

## Refresh triggers

Review the affected decisions immediately when:

- NIST revises SP 800-53/53A, SP 800-63-4, SP 800-162, SP 800-207/207A, Privacy Framework, AI RMF, or incident-response guidance;
- IETF publishes errata, updates, or successors for SCIM 7643/7644/9865/9967, SET delivery/subject identifiers, OAuth security/token exchange, or related provisioning standards;
- OpenID SSF/CAEP/RISC final specifications receive errata/new versions or chosen providers add/change conformance;
- the selected IGA/IdP/PAM/cloud/application changes connector APIs, pagination, filters, nesting, deletion, async events, quotas, status, idempotency, propagation, limits, licensing, or region support;
- an authoritative source changes identity/correlation/lifecycle semantics;
- role/entitlement/SoD/approval/resource-owner policy changes;
- model/provider changes model IDs/aliases, tool behavior, context, retention, residency, caching, subprocessors, or safety behavior;
- OpenTelemetry GenAI conventions stabilize or change schema/content guidance;
- applicable privacy, employment, financial, health, critical-infrastructure, records, audit, or cybersecurity obligations change;
- a cross-tenant incident, wrong correlation, unauthorized/duplicate effect, delayed leaver, missed expiry, false review recommendation, prompt injection, unexplained access path, or reconciliation divergence occurs;
- workload scope adds a tenant, jurisdiction, subject type, high-risk resource, privileged access, batch selector, connector, model memory class, or effect cell;
- 90 days pass for volatile protocols/connectors/security/provider behavior or 180 days for stable workload architecture without a targeted refresh.

## Research limitations

- No live organization, HRIS, directory, IGA, PAM, application, or cloud tenant was available. Connector semantics and permissions were not conformance-tested.
- No Entra, Okta, or SailPoint tenant was available to validate feature flags, licenses, clouds/regions, API permissions, stable IDs, pagination, rate headers, review/remediation transitions, aggregation behavior, job retention, event loss, propagation, or target/session postconditions. The provider profiles are qualification hypotheses only.
- No authoritative directory/HR truth model was supplied. Stable source IDs, worker reuse/rehire, contingent workers, future/backdated/corrected lifecycle facts, manager/sponsor precedence, service-account ownership, merge/split and deletion semantics remain organization-specific.
- No executable organization policy was supplied. Birthright mappings, SoD semantics, approver authority, overlap, exception, non-response, protected access, expiry and emergency actions require versioned control-owner decisions and tests.
- No global revocation proof is possible from the researched standards or product docs. At most, a deployment can prove stated direct/effective/session postconditions for declared systems at observed times; bearer tokens, caches, local/offline sessions and uncovered applications can preserve residual access.
- Public vendor documentation cannot prove configured tenant behavior, internal controls, support-tier behavior, actual propagation, availability, or future compatibility.
- Product licenses, previews, regions/clouds, quotas, retention, pricing, and feature limits can change and must be verified at procurement/deployment time.
- The packet is jurisdiction-neutral. It does not determine GDPR lawful basis, employment-law obligations, sector rules, records schedules, audit independence, legal hold, notification, or admissibility.
- NIST, CISA, NCSC, GAO, PCI SSC, EU law, and vendor sources have different scopes. Referencing them does not make all of them applicable or establish compliance.
- No organization-specific SoD matrix, role model, risk appetite, protected population, reviewer authority, or access-review frequency was defined.
- No performance benchmark was run. The blueprint deliberately does not reuse Google Zanzibar or vendor scale/latency numbers as local claims.
- No task-quality threshold is universal. Hard policy/tenant/effect invariants are zero-tolerance; empirical quality/SLO/cost thresholds require local ratification.
- The access-review usability paper is foundational but older and not a substitute for testing the selected UI, reviewers, data, and recommendation behavior.
- Emerging SCIM cursor/event and OpenID Shared Signals specifications are current standards but provider adoption is not assumed.
- The blueprint is provider-neutral and does not compare model vendors. Exact model/tool/data terms must be evaluated at release time.

## Primary and authoritative source register

### Identity, access control, governance, privacy, and incident guidance

- [NIST SP 800-53 Rev. 5](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)
- [NIST SP 800-53A Rev. 5](https://csrc.nist.gov/pubs/sp/800/53/a/r5/final)
- [NIST SP 800-63-4](https://csrc.nist.gov/pubs/sp/800/63/4/final)
- [NIST SP 800-162](https://csrc.nist.gov/pubs/sp/800/162/upd2/final)
- [NIST SP 800-207](https://csrc.nist.gov/pubs/sp/800/207/final)
- [NIST SP 800-207A](https://csrc.nist.gov/pubs/sp/800/207/a/final)
- [NIST IR 6192](https://csrc.nist.gov/pubs/ir/6192/final)
- [NIST role-based access control history](https://csrc.nist.gov/nist-cyber-history/identity-access-management/chapter)
- [NIST mutual exclusion and SoD paper](https://tsapps.nist.gov/publication/get_pdf.cfm?pub_id=916538)
- [NIST Privacy Framework](https://www.nist.gov/privacy-framework)
- [NIST AI RMF Core](https://airc.nist.gov/airmf-resources/airmf/5-sec-core/)
- [NIST AI 600-1](https://doi.org/10.6028/NIST.AI.600-1)
- [NIST SP 800-61 Rev. 3](https://csrc.nist.gov/pubs/sp/800/61/r3/final)
- [CISA/NSA IAM best practices for administrators](https://www.cisa.gov/sites/default/files/2023-12/ESF%20IDENTITY%20AND%20ACCESS%20MANAGEMENT%20RECOMMENDED%20BEST%20PRACTICES%20FOR%20ADMINISTRATORS%20PP-23-0248_508C.pdf)
- [CISA Zero Trust Maturity Model Version 2](https://www.cisa.gov/resources-tools/resources/zero-trust-maturity-model)
- [NCSC identity and access management](https://www.ncsc.gov.uk/collection/10-steps/identity-and-access-management)
- [NCSC Cyber Assessment Framework B2](https://www.ncsc.gov.uk/collection/cyber-assessment-framework/caf-objective-b/principle-b2-identity-and-access-control)
- [NCSC SaaS secure user management](https://www.ncsc.gov.uk/collection/cloud/using-cloud-services-securely/using-saas-securely)
- [GAO 2025 Green Book](https://www.gao.gov/greenbook)
- [GDPR consolidated text](https://eur-lex.europa.eu/eli/reg/2016/679/2016-05-04)
- [PCI DSS v4.0.1 document library](https://www.pcisecuritystandards.org/document_library/?class=pcidss&doc=pci_dss)

### Protocols and standards

- [RFC 7643: SCIM Core Schema](https://www.rfc-editor.org/rfc/rfc7643.html)
- [RFC 7644: SCIM Protocol](https://www.rfc-editor.org/rfc/rfc7644.html)
- [RFC 9865: SCIM Cursor Pagination](https://www.rfc-editor.org/rfc/rfc9865.html)
- [RFC 9967: SCIM Profile for Security Event Tokens](https://www.rfc-editor.org/rfc/rfc9967.html)
- [RFC 8417: Security Event Token](https://www.rfc-editor.org/rfc/rfc8417.html)
- [RFC 8935: Push-Based SET Delivery](https://www.rfc-editor.org/rfc/rfc8935.html)
- [RFC 8936: Poll-Based SET Delivery](https://www.rfc-editor.org/rfc/rfc8936.html)
- [RFC 9493: Subject Identifiers for SETs](https://www.rfc-editor.org/rfc/rfc9493.html)
- [OpenID Shared Signals Framework 1.0 Final](https://openid.net/specs/openid-sharedsignals-framework-1_0-final.html)
- [OpenID CAEP 1.0 Final](https://openid.net/specs/openid-caep-1_0-final.html)
- [OAuth 2.0 Security BCP, RFC 9700](https://www.rfc-editor.org/rfc/rfc9700.html)
- [OAuth 2.0 Token Exchange, RFC 8693](https://www.rfc-editor.org/rfc/rfc8693.html)
- [OAuth 2.0 Token Revocation, RFC 7009](https://www.rfc-editor.org/rfc/rfc7009.html)
- [OpenID Connect Core 1.0 incorporating errata set 2](https://openid.net/specs/openid-connect-core-1_0.html)
- [OASIS XACML 3.0](https://www.oasis-open.org/standard/xacmlv3-0/)
- [OASIS XACML RBAC profile](https://www.oasis-open.org/standard/xacml3-0-core/)
- [W3C PROV overview](https://www.w3.org/TR/prov-overview/)
- [W3C Trace Context](https://www.w3.org/TR/trace-context/)
- [OpenTelemetry semantic conventions](https://opentelemetry.io/docs/specs/semconv/)
- [OpenTelemetry GenAI conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/)

### Official vendor and implementation documentation

- [Microsoft Entra entitlement management](https://learn.microsoft.com/en-us/entra/id-governance/entitlement-management-overview)
- [Microsoft Entra access-review deployment](https://learn.microsoft.com/en-us/azure/active-directory/governance/deploy-access-reviews)
- [Microsoft Entra identity-governance security practices](https://learn.microsoft.com/en-us/entra/id-governance/best-practices-secure-id-governance)
- [Microsoft Entra ID Governance service limits](https://learn.microsoft.com/en-us/entra/id-governance/governance-service-limits)
- [Microsoft Entra access-review FAQ](https://learn.microsoft.com/en-us/entra/id-governance/access-reviews-faqs)
- [Microsoft Entra application access-review preparation](https://learn.microsoft.com/en-us/entra/id-governance/access-reviews-application-preparation)
- [Microsoft Entra complete an access review](https://learn.microsoft.com/en-us/entra/id-governance/complete-access-review)
- [Microsoft application provisioning mechanics](https://learn.microsoft.com/en-us/entra/identity/app-provisioning/how-provisioning-works)
- [Microsoft on-demand provisioning limitations](https://learn.microsoft.com/en-us/entra/identity/app-provisioning/provision-on-demand)
- [Microsoft SCIM compliance issues](https://learn.microsoft.com/en-us/entra/identity/app-provisioning/application-provisioning-config-problem-scim-compatibility)
- [Microsoft emergency access revocation](https://learn.microsoft.com/en-us/entra/identity/users/users-revoke-access)
- [Microsoft inactive-account detection](https://learn.microsoft.com/en-us/entra/identity/monitoring-health/howto-manage-inactive-user-accounts)
- [Microsoft privileged-access security planning](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/security-planning)
- [AWS IAM Identity Center SCIM limitations](https://docs.aws.amazon.com/singlesignon/latest/developerguide/limitations.html)
- [AWS IAM Identity Center automatic provisioning](https://docs.aws.amazon.com/singlesignon/latest/userguide/provision-automatically.html)
- [AWS Entra-to-IAM Identity Center considerations](https://docs.aws.amazon.com/singlesignon/latest/userguide/idp-microsoft-entra.html)
- [AWS IAM Access Analyzer concepts](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-concepts.html)
- [AWS IAM Access Analyzer findings](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-findings.html)
- [Okta Identity Governance campaigns](https://help.okta.com/oie/en-us/Content/Topics/identity-governance/access-certification/campaigns.htm)
- [Okta access-certification setup and limits](https://help.okta.com/oie/en-us/content/topics/identity-governance/access-certification/ac-get-started.htm)
- [Okta access-certification remediation](https://help.okta.com/oie/en-us/content/topics/identity-governance/access-certification/remediation.htm)
- [Okta past campaign detail fields](https://help.okta.com/oie/en-us/content/topics/identity-governance/columns-campaign-details.htm)
- [Okta Governance API](https://developer.okta.com/docs/api/iga)
- [Okta Governance OAuth scopes](https://developer.okta.com/docs/api/iga/oauth2)
- [Okta event-hook concepts](https://developer.okta.com/docs/concepts/event-hooks/)
- [Okta hook best practices](https://developer.okta.com/docs/guides/hooks-best-practices/)
- [Okta API-token management](https://help.okta.com/oie/en-us/content/topics/security/api.htm)
- [SailPoint source configuration](https://documentation.sailpoint.com/saas/help/sources/config_sources.html)
- [SailPoint account aggregation](https://documentation.sailpoint.com/saas/help/accounts/loading_data.html)
- [SailPoint access-profile behavior](https://documentation.sailpoint.com/saas/help/access/access-profiles.html)
- [SailPoint lifecycle states](https://documentation.sailpoint.com/saas/help/provisioning/lifecycle.html)
- [SailPoint certifications](https://documentation.sailpoint.com/saas/help/certs/understanding_certifications.html)
- [SailPoint access recommendations](https://documentation.sailpoint.com/saas/help/ai/access_recs/recommendations.html)
- [SailPoint submit access request](https://developer.sailpoint.com/docs/api/v3/create-access-request/)
- [SailPoint access-request status and operations](https://developer.sailpoint.com/docs/tools/sdk/go/accessrequests/methods/access-requests/)
- [SailPoint current API rate limiting](https://developer.sailpoint.com/docs/api/rate-limit/)
- [SailPoint API getting started](https://developer.sailpoint.com/docs/api/getting-started/)
- [Google Cloud identity and access governance patterns](https://docs.cloud.google.com/architecture/patterns-practices-identity-access-governance-google-cloud)
- [OpenFGA concepts](https://openfga.dev/docs/concepts)
- [OpenFGA conditions](https://openfga.dev/docs/modeling/conditions)
- [Open Policy Agent deployment/PDP model](https://www.openpolicyagent.org/docs/deploy)

### Foundational and operational evidence

- [Zanzibar: Google's Consistent, Global Authorization System](https://www.usenix.org/conference/atc19/presentation/pang)
- [USENIX SOUPS: Helping Users Review Access Policies](https://www.usenix.org/conference/soups2014/proceedings/presentation/jaferian)
- [AWS Builders' Library: Making retries safe with idempotent APIs](https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/)
- [Google SRE: Service level objectives](https://sre.google/sre-book/service-level-objectives/)
- [Google SRE Workbook: Alerting on SLOs](https://sre.google/workbook/alerting-on-slos/)
- [Temporal worker versioning](https://docs.temporal.io/production-deployment/worker-deployments/worker-versioning)
- [NIST SP 800-218: Secure Software Development Framework 1.1](https://csrc.nist.gov/pubs/sp/800/218/final)
- [NIST SP 800-161 Rev. 1 Update 1](https://csrc.nist.gov/pubs/sp/800/161/r1/upd1/final)

## Pass 2 quality statement

This refinement adds a canonical identity/effective-time ontology, exactly seven governed memory lifetimes, restart-safe compaction receipts, typed system/provider adapter qualification, current Entra/Okta/SailPoint worked effect chains and contradictions, effect/revocation evidence levels, realistic JML/review/emergency scenarios, supply-chain and insider controls, explicit evaluation baselines/slices/telemetry separation, stage exercises, recovery-load budgets, and rollback semantics. Promotion beyond research-backed draft still requires live connector conformance, authoritative HR/directory truth mapping, executable organization policy, representative reviewer testing, target/session propagation and revocation experiments, capacity/DR/incident drills, and independent security/privacy/control-owner review.
