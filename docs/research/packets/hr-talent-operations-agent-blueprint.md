# Research Packet: HR and Talent Operations Agent Blueprint

> **Status:** Pass 1 research-backed synthesis  
> **Research date:** 2026-08-31  
> **Scope:** Candidate/worker identity, requisitions, recruiting/interview evidence, human employment decisions, sensitive records, fairness, accessibility, onboarding/offboarding coordination, HRIS/ATS integrations, reliability, retention/deletion, deployment, and governed evolution  
> **Derived guides:** [Production HR and talent operations agent blueprint](../../agents/hr-talent-operations-agent/README.md)  
> **Method:** Primary law/regulator text, public-sector guidance, formal standards, official product/API documentation, and repository control guides were compared. Legal applicability is not inferred from a source example; it must be determined for the deployed employer, person, process, and jurisdiction by qualified owners.

This packet preserves the evidence and decisions behind category #36. It is not legal advice, a bias audit, a validation study, an accessibility conformance report, or an endorsement of any HR/model vendor.

Universal identity, danger-tier, effect, durability, telemetry, and incident invariants are specialized from the [cross-cutting blueprint controls](agent-blueprint-cross-cutting-controls.md); this packet records only the HR/talent-specific evidence and decisions.

## Research questions

1. Which HR/talent work benefits from a model-directed loop rather than rules, ATS/HRIS workflow, integration, retrieval, or trained human casework?
2. Which employment decisions and effects must remain deterministic, policy-owned, IAM-owned, or human-owned?
3. How should candidate, application, person, worker, employment, position, requisition, case, and account identity relate?
4. Which source, event, decision, approval, and effect records remain authoritative across future/backdated changes, rehire, correction, and replay?
5. What evidence protocol supports requisition, selection, interview, offer, onboarding, move, and offboarding work?
6. How should fairness, job-related validity, accessibility, accommodation, notice, contestability, and human oversight be evaluated?
7. Which data compartments, purposes, credentials, retention/deletion/hold paths, and provider controls are needed?
8. What do current ATS/HRIS APIs, webhooks, and identity standards guarantee—and what remains application-owned?
9. How should timeouts, duplicates, partial effects, ambiguous outcomes, cancellation, correction, and reconciliation work?
10. Which context, compaction, memory, planning, and multi-agent choices are justified or rejected?
11. Which evaluation, failure-injection, tracing, SLO, capacity, cost, incident, release, and refresh controls gate production?
12. Where do law, regulator guidance, professional practice, technical metrics, and vendor mechanisms conflict or remain unsettled?

## Baseline as researched

| Domain | Current baseline | Date/status boundary | Blueprint consequence |
|---|---|---|---|
| U.S. selection | EEOC Uniform Guidelines and employment-test guidance | Long-standing federal guidance; applicability/facts require counsel | Treat every opportunity-shaping component as a governed selection procedure; job-related validation and impact evidence are local responsibilities |
| U.S. disability | ADA text, DOJ and EEOC technical resources | DOJ guidance current on official site; guidance is not binding law by itself | Test for disability screen-out, provide accessible accommodation path, and separate medical information |
| U.S. background checks | FTC FCRA employer guidance | Current official guidance; state/local rules may add duties | Prevent vendor results from directly changing status; preserve authorization, pre-adverse, dispute, and adverse-action steps where applicable |
| NYC AEDT | Local Law 144 resources/rules | Enforcement since 2023; official page current at research date | Covered use may require annual bias audit/public summary/notices; this is not a universal fairness certificate |
| California employment ADS | Civil Rights Council regulations | Effective 2025-10-01; official final text/approval | ADS use can implicate FEHA; automated-decision data included in four-year employment-record baseline described by CRD |
| Illinois AI video interview | 820 ILCS 42 | Current compiled statute at research date | Disclosure, explanation, consent, sharing, deletion, and reporting conditions may apply to covered video-interview use |
| Colorado ADMT | SB 26-189 as described by Colorado AG | New law effective 2027-01-01; rulemaking active Aug–Oct 2026 | Do not implement against superseded SB24-205 assumptions; maintain a refreshable jurisdiction profile |
| EU AI | Regulation 2024/1689 plus 2026 AI Omnibus implementation pages | Act in force; Commission states Annex III employment high-risk rules apply 2027-12-02 after Omnibus | Employment AI classification/obligations and timeline are version-sensitive; workplace emotion inference is prohibited except narrow stated exception |
| EU privacy | GDPR and EDPB endorsed automated-decision/DPIA guidance | In force; Member State employment rules matter | Purpose/minimization/storage limitation, rights, DPIA, special data, and meaningful human involvement require deployment-specific review |
| UK privacy/recruitment | ICO recruitment AI audit and draft recruitment guidance | ICO guidance under review for Data (Use and Access) Act; final update expected after research date | Use audit findings as operational evidence; refresh legal/guidance claims before UK deployment |
| Accessibility | WCAG 2.2 W3C Recommendation | 2024 Recommendation; W3C notes it does not meet every need | Use WCAG as technical baseline plus assistive-tech, disability-led, and accommodation-path testing |
| AI risk/bias | NIST AI RMF 1.0, NIST AI 600-1, SP 1270 | AI RMF 1.0 revision underway in 2026 | Treat fairness as socio-technical; document limits, human roles, TEVV, appeals, and continuous monitoring |
| Interview practice | OPM structured-interview resources | Current official public-sector guidance; organization must validate its own procedure | Predetermined job-related questions, common rating anchors, trained assessors, and independent evidence improve consistency |
| ATS mechanics | Greenhouse Harvest API/webhooks | Official docs current but plan/version specific | Endpoint scopes can still expose broad data; signed retrying webhooks are hints requiring dedupe and source reread |
| HRIS mechanics | Workday business-process event APIs | Official current docs; tenant configuration/version specific | Effective dates, event IDs/statuses, pending steps, cancellation/rescind mechanics require adapter tests |
| Identity interoperability | SCIM RFC 7643/7644 | Standards Track RFCs; provider semantics vary | SCIM can carry enterprise attributes/conditional updates, but HR agent must not own IAM access policy/effects |
| Operations | NIST SP 800-61r3, W3C Trace Context, repository runtime/effect guides | Current stable general controls; GenAI OTel conventions remain version-sensitive | Preserve privacy-safe correlation, independent containment, effect reconciliation, and whole-behavior releases |

“Current” means the research date. Regulations, regulator positions, statutes, APIs, plans, product features, and model behavior can change independently.

## Pass-2 volatile standards and provider refresh

These primary surfaces were rechecked on 2026-08-31. Their public behavior informs adapter qualification; it does not establish a live tenant's plan, configuration, contractual rights, local selection validity, fairness, accessibility, legal sufficiency or operational guarantee:

| Surface/status observed | Engineering evidence | Limitation and refresh trigger |
|---|---|---|
| [Greenhouse Harvest API](https://docs.greenhouse.io/harvest.html), [webhooks](https://docs.greenhouse.io/webhooks.html), and [Assessment API](https://docs.greenhouse.io/assessment.html) | Harvest endpoint-wide data access, Link pagination/rate headers/ephemeral attachments; retrying signed event hints; assessment definition/instance/status mechanics and Jul 2025 candidate/application-ID change | Continuously delivered docs and tenant/plan features; refresh on change log, endpoint/field/permission, webhook/signature, assessment flow or contract change |
| [SAP effective-dated OData v2](https://help.sap.com/docs/successfactors-platform/sap-successfactors-api-reference-guide-odata-v2/effective-dated-query-in-odata) | `asOfDate`, `fromDate`/`toDate`, history/future time slices and default today behavior | Nonstandard query/expand/sequence semantics require exact entity fixtures; refresh with SuccessFactors release and tenant metadata |
| [SAP CompoundEmployee API 1H 2026](https://help.sap.com/docs/successfactors-employee-central/employee-central-compound-employee-api/compoundemployee-api-general-information-about-full-snapshot-and-delta-transmission-modes) | Approved employee extraction with full/snapshot/effective and period delta modes for payroll/benefits replication | SOAP, supported fields, auditing and consumer time-slice support vary; future and retroactive changes behave differently by mode |
| [Oracle Fusion Cloud HCM assignment REST](https://docs.oracle.com/en/cloud/saas/human-resources/farws/api-workers-work-relationships-assignments.html) | Public `11.13.18.05` worker relationship/assignment resources and as-of semantics | Resource path is not proof of quarterly tenant configuration, privileges, ETags or correction/action behavior; contract-test the deployed tenant |
| [Workday Events REST APIs](https://developer.workday.com/documentation/GUID-0df5cd55-e578-43d3-b58f-ae98825d1df0-enHYPHENus) | Official business-process event surface | Public page is dynamically delivered and exact API/release/tenant/security-domain behavior was not independently retrievable; use contracted docs and live conformance evidence |
| [Microsoft Graph calendar v1.0](https://learn.microsoft.com/en-us/graph/api/calendar-post-events?view=graph-rest-1.0) and [`sendMail`](https://learn.microsoft.com/en-us/graph/api/user-sendmail?view=graph-rest-1.0) | Event identity/update/cancel and client `transactionId`; mail returns asynchronous `202 Accepted` | Creation/acceptance does not prove attendance, interview completion or message delivery; delegated rights, time zones and Exchange processing need tenant tests |
| [Checkr API v1](https://docs.checkr.com/) | Credentialing/staging, candidate-hosted invitations, report/webhook status and POST idempotency keys | Provider compliance flow/result is not employer adjudication or local-law assurance; package/work location/tenant behavior and human process require qualification |
| [Acrobat Sign REST API v6](https://developer.adobe.com/acrobat-sign/docs/overview/developer_guide/apiusage) and [agreement events](https://developer.adobe.com/acrobat-sign/docs/overview/acrobat_sign_events/webhookeventsagreements) | Agreement identity/status/history, document versions and webhooks | Webhook details may be omitted during size/processing limits; sent/signed is not proof of correct terms, signer authority or HR acceptance |
| [SCIM RFC 7643](https://www.rfc-editor.org/info/rfc7643/), [RFC 7644](https://www.rfc-editor.org/info/rfc7644/), [RFC 9865](https://www.rfc-editor.org/info/rfc9865/) and [RFC 9967](https://www.rfc-editor.org/info/rfc9967/) | Core/protocol plus 2025 cursor-pagination and 2026 security-event-profile updates | Provider support/conformance remains optional/variable; cursor possession never grants access and SCIM attributes do not define employment/access policy |
| [OpenTelemetry specifications](https://opentelemetry.io/docs/specs/) | Specification 1.60.0, OTLP 1.11.0 and semantic conventions 1.44.0 shown at research time | GenAI conventions moved to a separate repository and version-selection work was development status; pin emitted schemas and keep HR content out |
| [WCAG 2.2](https://www.w3.org/TR/WCAG22/) and [W3C WCAG overview](https://www.w3.org/WAI/standards-guidelines/wcag/) | W3C Recommendation republished Dec 2024; ISO/IEC 40500:2025 corresponds to the Oct 2023 version | W3C records later errata and explicitly says WCAG cannot meet every disability need; use exact conformance target plus end-to-end affected-user/accommodation testing |

### Recorded contradictions and status limits

- SCIM is still commonly called “RFC 7643/7644” or “SCIM 2.0,” but those RFCs now show updates from RFC 9865 and RFC 9967. A provider claiming SCIM 2.0 may support neither update; discover and test exact capabilities rather than assuming the label is complete.
- WCAG 2.2 has both a Dec 2024 W3C Recommendation incorporating errata and ISO/IEC 40500:2025 corresponding to the Oct 2023 WCAG 2.2 text. Record the exact conformance reference; neither is a complete employment-process accessibility certificate.
- Workday's official public developer page identifies the Events surface but did not expose a stable static version or enough contract detail to this research environment. No tenant mechanic was inferred from product reputation.
- SAP OData queries with no explicit date can default to today's effective record, while CompoundEmployee effective-delta and period-delta modes deliberately expose different future/retroactive slices. “Current employee record” is not a universal integration semantic.
- Greenhouse offers both write endpoints and assessment scores, but API capability is not agent authority or evidence that a procedure is job-related, fair or accessible.
- Provider `completed`, `signed`, `clear`, `accepted` or HTTP success states each have provider-specific meaning. None is promoted into an employment decision or downstream completion without the application-owned human/effect oracle.

## Evidence classification

| Class | Meaning | Examples in this packet |
|---|---|---|
| **Law/standard mechanic** | Normative text or formal protocol behavior | EU AI Act, GDPR, state statutes, SCIM |
| **Regulator/public guidance** | Authoritative interpretation/guidance within stated limits | EEOC, DOJ, FTC, ICO, CRD, NYC DCWP, Colorado AG |
| **Product mechanic** | Vendor-documented API/product behavior | Greenhouse, Workday, Microsoft Entra |
| **Observed regulator finding** | Evidence from an audit/enforcement report, not universal prevalence | ICO recruitment AI audit outcomes |
| **Engineering inference** | Architecture derived by comparing sources and failure boundaries | Proposal-only model, effect ledger, reconciliation, compartmentation |
| **Recommendation** | Chosen production posture | No autonomous high-impact employment decisions; workflow-centered hybrid |
| **Open question** | Applicability/evidence unresolved at packet scope | Exact employer/jurisdiction obligations, local validity, works-council rules |

One source can prove a mechanic or that a failure occurred; it cannot prove a universal rate or whole-system safety.

## Material decision record

All entries were researched on 2026-08-31. “Strongest sources” identifies the primary basis; the later source register records scope in more detail.

| Decision | Claim class | Strongest sources | Boundary, contradiction, or limitation | Blueprint consequence and refresh |
|---|---|---|---|---|
| Use a deterministic HR workflow with bounded model workers | Engineering inference/recommendation | [NIST AI RMF Core](https://airc.nist.gov/airmf-resources/airmf/5-sec-core/), [Workday event APIs](https://developer.workday.com/documentation/GUID-0df5cd55-e578-43d3-b58f-ae98825d1df0-enHYPHENus) | Sources define risk/process mechanics, not one required architecture | Require Stage 0 baseline; refresh if workflow/model capability changes the measured fit |
| Prohibit autonomous high-impact employment decisions | Recommendation grounded in law/guidance | [EU AI Act](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX:32024R1689), [EEOC selection guidance](https://www.eeoc.gov/laws/guidance/employment-tests-and-selection-procedures), [DOJ disability guidance](https://www.ada.gov/resources/ai-guidance/) | Human review can itself be biased or superficial; source applicability varies | Enforce H5 prohibition and meaningful human-decision contract; refresh on law/use/authority change |
| Treat recruiting as a versioned selection-evidence protocol | Engineering inference/recommendation | [EEOC Uniform Guidelines Q&A](https://www.eeoc.gov/laws/guidance/questions-and-answers-clarify-and-provide-common-interpretation-uniform-guidelines), [OPM structured interviews](https://www.opm.gov/policy-data-oversight/assessment-and-selection/structured-interviews) | OPM practice does not prove local validity; Uniform Guidelines are U.S.-specific | Version job analysis/procedure/rubric/cohort and capture independent human evidence |
| Do not certify fairness with one audit or metric | Engineering inference | [EEOC Uniform Guidelines Q&A](https://www.eeoc.gov/laws/guidance/questions-and-answers-clarify-and-provide-common-interpretation-uniform-guidelines), [NYC AEDT resources](https://www.nyc.gov/site/dca/about/automated-employment-decision-tools.page), [NIST SP 1270](https://www.nist.gov/publications/towards-standard-identifying-and-managing-bias-artificial-intelligence) | Required calculations, validity, disability barriers, intersections, and sample uncertainty differ | Keep legal and product-safety gates distinct; refresh metrics with procedure/jurisdiction/population changes |
| Make accessibility and accommodation an end-to-end release gate | Recommendation | [DOJ disability guidance](https://www.ada.gov/resources/ai-guidance/), [WCAG 2.2](https://www.w3.org/TR/WCAG22/) | WCAG does not cover every disability or employment obligation | Test assistive tech, affected users, equivalent path, and no-penalty recovery; refresh with surface/vendor changes |
| Separate sensitive HR data compartments | Engineering inference grounded in privacy/employment guidance | [EEOC medical confidentiality](https://www.eeoc.gov/pre-employment-inquiries-and-medical-questions-examinations), [GDPR](https://eur-lex.europa.eu/eli/reg/2016/679/oj) | Exact categories, basis, access, and labor rules vary | Field-purpose authorization and separate stores/views/keys/context; refresh with data/use/jurisdiction changes |
| Compute retention/deletion by record class and copy inventory | Recommendation | [EEOC recordkeeping](https://www.eeoc.gov/employers/recordkeeping-requirements), [USCIS I-9 guide](https://www.uscis.gov/sites/default/files/document/guides/E3en.pdf), [Illinois AI Video Interview Act](https://www.ilga.gov/Legislation/ILCS/Articles?ActID=4015&ChapterID=68&Print=) | Sources show different clocks; none is a complete global schedule | Policy engine plus deletion/hold propagation and proof; refresh on law/claim/vendor/storage change |
| Require local evidence and operational contracts for third parties | Recommendation/observed-result response | [ICO audit report](https://ico.org.uk/media/about-the-ico/documents/4031620/ai-in-recruitment-outcomes-report.pdf), [UK recruitment assurance guide](https://www.gov.uk/government/publications/responsible-ai-in-recruitment-guide/responsible-ai-in-recruitment) | ICO audit sample does not establish prevalence; guidance is not legal assurance | Vendor admission, local pilot, version lock, audit/deletion/exit terms; refresh on every vendor release/contract change |
| Treat webhooks as hints and preserve an `unknown` effect state | Product mechanic/engineering inference | [Greenhouse webhooks](https://docs.greenhouse.io/webhooks.html), [AWS idempotent APIs](https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/) | Retry delivery and idempotency do not prove ordered or exactly-once business outcome | Persist/dedupe/reread source; semantic operation ledger, status query, reconciliation; refresh per adapter version |
| Keep HR lifecycle facts separate from IAM access effects | Boundary recommendation | [Microsoft HR-driven provisioning](https://learn.microsoft.com/en-us/entra/identity/app-provisioning/what-is-hr-driven-provisioning), [SCIM RFC 7643](https://www.rfc-editor.org/info/rfc7643/) | Product docs show integrated automation, but access policy remains an IAM concern | Publish minimal source-versioned facts; IAM executes and proves access state; refresh with IAM/provisioning design |
| Reject long-term person profiling and default multi-agent role simulation | Recommendation | [NIST AI RMF Core](https://airc.nist.gov/airmf-resources/airmf/5-sec-core/), repository [memory architecture](../../context-memory/memory-architecture.md) | No source proves these mechanisms are always harmful; decision is based on workload authority/privacy cost | Typed task memory and one coordinator; reconsider only with measurable benefit and full controls |
| Release the whole behavior bundle and reconcile after rollback | Engineering inference/recommendation | [NIST AI 600-1](https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.600-1.pdf), [NIST SP 800-61r3](https://csrc.nist.gov/pubs/sp/800/61/r3/final) | General lifecycle/incident sources do not define HR-specific cohorts/effects | Manifest model/prompt/tool/policy/rubric/vendor/runtime; shadow/canary/rollback/impact review; refresh on any component change |

## Finding 1: the deterministic workflow is the baseline; the agent is optional

ATS/HRIS products and workflow engines already represent requisitions, approvals, interviews, offers, worker events, and tasks. Stable calculations, eligibility, deadlines, and routing are better expressed as rules. A model adds value only where variable language/evidence requires bounded extraction, comparison, drafting, or exception planning that measurably improves the baseline.

**Decision:** recommend a model worker inside an authoritative workflow, not an “AI recruiter.” Stage 0 must measure ordinary workflow, parser, retrieval, scheduler, and manual alternatives.

**Limit:** product docs show mechanisms, not that a specific product/process is correct or accessible.

## Finding 2: consequential employment decisions remain human-owned

Employment selection, termination, promotion, compensation, accommodation, and similar outcomes materially affect people. EU law treats many employment AI uses as high-risk; U.S. federal/state/local sources show discrimination, disability, notice, audit, recordkeeping, and due-process-like concerns; NIST requires explicit human-AI roles and oversight.

A human click is not meaningful oversight when the reviewer lacks authority/time/evidence, sees only the model recommendation, cannot choose alternatives, or routinely rubber-stamps.

**Decision:** the model may assemble evidence and proposals. An authenticated, trained, eligible human independently makes each high-impact employment decision. The application, not the prompt, enforces this ceiling.

**Refresh trigger:** law/guidance or an approved use expands/changes automated-decision boundaries.

## Finding 3: identity requires person and employment semantics plus effective time

SCIM distinguishes user attributes and enterprise extensions, while HRIS APIs expose worker/business-process and effective-date semantics. Neither makes email a stable person key. Rehire, concurrent employment, contractor conversion, global transfer, name changes, and migration create legitimate many-to-many/time-varying relationships.

**Decision:** keep candidate, application, person, worker, employment, position, requisition, account, case, run, and effect identities distinct. Record effective, observed, and transaction time. Ambiguity stops dependent effects.

**Limitation:** each HRIS tenant's rehire, rescind, effective dating, and external-ID behavior requires local contract fixtures.

## Finding 4: jurisdiction and policy must be versioned data, not model interpretation

The 2026 research itself exposed volatility: Colorado's original 2024 AI act was delayed and then replaced/re-enacted in 2026 with a 2027 effective date; the EU AI Omnibus changed the Annex III timeline; UK ICO recruitment guidance was under statutory update. Location, legal entity, residence, worker type, collective agreement, and process can affect applicability.

**Decision:** admission resolves a governed jurisdiction profile and policy version or stops. Legal owns interpretation. The model may retrieve/explain approved policy with citations; it may not decide which law applies.

**Open question:** country/state/local and collective-bargaining coverage must be researched for each rollout; this packet is a control blueprint, not a global obligations register.

## Finding 5: recruiting is an evidence protocol, not a ranking problem

The EEOC Uniform Guidelines and OPM assessment guidance emphasize selection procedures, job-related evidence, validation, common procedures, and records. A model summary can omit negative/contradictory evidence or anchor reviewers. Historical hiring decisions may encode prior practice rather than job truth.

**Decision:** version job analysis, essential functions, criteria, selection procedures, questions, rubrics, thresholds, cohort, and evidence. Interviewers record independent cited observations/scores before optional model synthesis. The agent never ranks/shortlists/rejects.

**Limitation:** OPM is public-sector practice guidance, not proof that one structured interview design is valid for every employer/job.

## Finding 6: fairness cannot be reduced to the four-fifths rule or one parity metric

EEOC describes the four-fifths comparison as a rule of thumb and notes that smaller differences can matter with practical/statistical significance and larger numbers; small samples need context. NYC's law defines covered audit calculations, while NIST SP 1270 frames bias as systemic, human, and computational. Disability barriers can exist without a useful aggregate parity calculation.

**Decision:** separate jurisdiction-required calculations from product-safety evaluation. Test procedure validity, data/measurement, component errors, funnel outcomes, intersections, uncertainty, human behavior, accessibility, contest, and longitudinal drift. No single metric is a release certificate.

**Contradiction resolved:** a vendor “bias audit passed” is evidence for that audit scope/date/data, not proof of lawful, valid, accessible, or fair local use.

## Finding 7: accessibility requires an equivalent process and accommodation seam

DOJ guidance gives examples where facial/voice analysis and inaccessible interview technology can screen out qualified people with disabilities. It emphasizes reasonable accommodation and evaluating job skill rather than disability. WCAG 2.2 is a stable technical standard, but W3C states it does not cover every need.

**Decision:** combine WCAG target, automated tests, assistive-technology tests, disability-led usability, notice, no-penalty reschedule, and a confidential accommodation/equivalent path. The selector/model sees only approved process status or adjustment, not medical rationale.

**Recommendation:** reject emotion, gaze, face, voice, accent, and personality inference for recruiting/worker decisions.

## Finding 8: medical, accommodation, demographic, background, and ordinary HR data need separate compartments

EEOC guidance states applicant/employee medical information is confidential and maintained separately, with narrow disclosures. DOJ warns hiring tools can elicit disability information. Fairness monitoring may need demographic data that selectors/recruiters should not see. FCRA-related screening has its own authorization and adverse-action process where applicable.

**Decision:** use distinct stores/views/roles/keys/context paths for recruiting, selection evidence, demographic monitoring, accommodation/medical, background, personnel, compensation, employee relations, I-9, and IAM. Passing all fields to one model is prohibited.

**Open question:** exact special/sensitive data categories and labor/works-council constraints vary; privacy/legal owners configure each profile.

## Finding 9: data minimization must include model, trace, memory, evaluation, and backup copies

GDPR principles, NIST privacy/risk guidance, Google/Microsoft-style API data policies, and regulator audit findings all point beyond the source database. Prompts, tool results, cache, vector indexes, provider logs, traces, eval sets, exports, and backups can replicate an employment record.

**Decision:** inventory every copy and carry purpose, source ACL, retention, deletion key, region, and provenance. Default trace content off. Use controlled evidence references instead of raw content in telemetry.

**Limitation:** provider retention/training/deletion/residency terms change by product, contract, region, and configuration; verify at procurement and release.

## Finding 10: retention/deletion is a record-class policy engine

Official examples have materially different clocks: EEOC personnel/employment records, payroll/equal-pay records, USCIS I-9 rules, California automated-decision employment records, Illinois video deletion requests, GDPR/UK purpose/storage/rights frameworks, plus claims/holds/contracts. One maximum period over-retains; one minimum period can destroy required evidence.

**Decision:** compute disposition by record class, trigger, jurisdiction/coverage, purpose, and hold. Propagate to source, derived artifacts, model/provider copies, search, trace, eval, export, vendors, and backups with terminal evidence. Holds preserve only authorized scope and do not grant broad access.

**Open question:** deployment counsel/records owners must produce the actual schedule; the blueprint intentionally does not supply universal periods.

## Finding 11: third-party assurances do not transfer employer accountability

The UK government's recruitment guide recommends purpose definition, impact/DPIA work, supplier evidence, model cards, accuracy/validity and bias audits, pilots, monitoring, transparency, and contestability. The ICO recruitment-tool audit found good practices and gaps, including insufficient accuracy testing and attempts to shift compliance responsibility to recruiters.

**Decision:** every assessment/model/ATS/HRIS/background/communication/MCP/iPaaS vendor receives a purpose- and operation-specific admission packet, local pilot, version lock, contract tests, monitoring, incident/deletion/exit terms, and replacement/manual path.

**Recommendation:** reject a vendor that cannot expose intended use, features/construct, version changes, local validation evidence, accessibility, data flow, retention/deletion, APIs/status, and audit evidence.

## Finding 12: APIs and webhooks require gap recovery and narrow scopes

Greenhouse documents endpoint permissions while warning that data access within an endpoint can be broad, API rate limits/pagination, ephemeral attachment URLs, signed webhook delivery IDs, and retries. Workday exposes business-process event IDs, statuses, effective/due/completed dates, steps, cancellation/rescind operations, but exact services and tenant behavior vary.

**Decision:** webhook = authenticated hint. Persist/dedupe, then reread authoritative state. Adapters must define stable IDs, pagination/cursors, versions/effective dates, scopes, limits, receipts, status/read-back, consistency, cancellation, and schema lifecycle. Run scheduled gap/full reconciliation.

**Limitation:** official docs do not promise end-to-end exactly once or organizational correctness.

## Finding 13: external effects need semantic identity and an unknown state

Workflow journals and HTTP responses cannot resolve a lost response after a remote system committed. Provider idempotency is limited by key scope, parameters, retention, and concurrency. Email/API acceptance may not prove delivery or business completion.

**Decision:** reserve a stable semantic operation ID and canonical payload hash, revalidate authority/source/policy at commit, persist receipts, model `unknown`, query status/read-back before retry, and issue linked corrections/compensations. A transactional outbox closes only the local state/message gap.

**Failure requirement:** explicitly test “HRIS/IAM/vendor effect happened but receipt recording failed.”

## Finding 14: HR is source evidence for IAM; IAM implements access changes

Microsoft documents HR-driven provisioning where HR systems are sources of authority for identity facts and IAM provisioning/lifecycle workflows act on joiner/mover/leaver changes. SCIM standardizes user resources/operations but leaves provider meanings and access policy outside the HR agent.

**Decision:** the HR agent publishes a minimal authenticated person/employment/legal-entity/effective lifecycle fact with source version and reconciles IAM acknowledgement. IAM maps policy, grants/revokes access, and reports authoritative outcome. HR cannot bypass an IAM denial or manipulate accounts by email.

**Boundary:** the [IAM blueprint](../../agents/identity-access-governance-agent/README.md) owns entitlement graphs, SoD, access reviews, and revocation proof.

## Finding 15: context and memory must not create a shadow employee profile

Long HR cases need durable task state, not full conversational memory. Policies/job analyses/rubrics belong to governed domain sources. Cross-run recruiter preferences or model-created candidate/employee traits can become hidden selection criteria; raw outcomes can poison future behavior.

**Decision:** use task-specific turn context, typed run working state, minimal session continuity, required durable case state, governed domain knowledge, no user/person long-term profiling by default, and curated/de-identified episodic incidents only after review. Compaction retains identity, effective dates, negative/conflicting evidence, decisions, approvals, and effects.

**Recommendation:** no vector store over full personnel files unless a narrowly justified purpose and complete ACL/deletion/compartment design exists.

## Finding 16: fixed orchestration is safer for normal HR work

Requisitions, interview rounds, onboarding checklists, offboarding deadlines, notices, and approvals are policy-defined. Dynamic planning is useful only for bounded evidence/exception discovery. Simulated recruiter/manager/legal agents do not create real organizational authority or segregation of duties.

**Decision:** fixed workflow for normal cases; bounded model plan for allowed exception-resolution reads/drafts; one coordinator; no default multi-agent topology. Human roles stay in identity/policy systems.

**Change signal:** consider specialist workers only if typed delegation improves measurable quality/latency and can be isolated without inherited authority.

## Finding 17: evaluation must cover people, process, state, effects, and severe tails

NIST AI RMF calls for deployment-representative TEVV, human roles, monitoring, and appeal/feedback. Employment-specific sources require job/selection/accessibility context. Agent evaluation must include trajectories and state changes, not only answer style.

**Decision:** seed an HR world with ATS/HRIS/vendor/IAM simulators; include normal, boundary, fairness, disability/accessibility, adversarial, authority, tool-failure, partial-effect, cancellation, correction, deletion, and novelty cases. Combine deterministic outcome/policy graders, statistical analysis, calibrated model graders, and qualified human review.

**Hard failures:** any autonomous high-impact decision, wrong-person/tenant effect, prohibited inference/sensitive leak, critical accessibility barrier, stale/unauthorized commit, or unreconciled critical effect.

## Finding 18: observability must remain less sensitive than the evidence plane

W3C Trace Context supports distributed correlation but trace headers/baggage are propagated broadly. HR prompts and artifacts contain high-risk personal data. Sampled diagnostics cannot replace an authoritative decision/effect ledger.

**Decision:** trace IDs and structured control metadata; content capture off; separate evidence, audit, diagnostic, and fairness planes; field redaction before export; deletion-aware telemetry. SLOs measure deadline completion/safe escalation, effect convergence, IAM acknowledgement, freshness, rights, and cost per correct case.

**Limitation:** OpenTelemetry GenAI semantic conventions are evolving; preserve an application-owned stable vocabulary.

## Finding 19: scale is deadline, isolation, and recovery—not only throughput

Hiring seasons, acquisitions, reorganizations, annual cycles, and mass offboarding create bursty load and aggregate authority. Vendor limits, human review, webhook full sync, and reconciliation backlogs can dominate. A model outage must not stop required employment work.

**Decision:** per-tenant/workflow/risk admission, deadline queues, person/employment serialization, bounded workers, regional/data boundaries, reserved reconciliation/rights capacity, manual/deterministic fallback, and safe degradation that disables enrichment first.

**Cost decision:** count model, storage, connectors, workflow, human review, fairness/accessibility/privacy assurance, reconciliation, exceptions, vendor licenses, and incidents per correctly completed case.

## Finding 20: every behavior change is a release

Model alias, prompt, context, memory, tool schema, adapter, vendor assessment feature, rule/policy, job rubric, UI default, evaluator, redaction, retention, and region configuration can change outcomes. Rolling back code cannot undo a sent message, offer, HRIS event, or IAM action.

**Decision:** signed behavior manifest, full critical eval, active-case/cohort policy, shadow, canary, rollback, effect reconciliation, decision-impact review, and deprecation. Do not change a selection procedure mid-cohort without governed impact handling.

**Incident boundary:** HR coordinates process; security contains; IAM remediates access; privacy/legal determine obligations; SRE restores service; compliance independently reviews.

## Architecture alternatives and conditional decision

| Alternative | Evidence-supported use | Rejection reason/change signal |
|---|---|---|
| Deterministic ATS/HRIS workflow | Stable rules, steps, timers, structured integrations | Default; add model only for measured semantic residual |
| Retrieval/single model call | Policy explanation or one-shot cited draft | Prefer when no tool/planning loop is required |
| Custom controller | Bounded read/proposal pilot | Add durability when waits/partial effects dominate |
| Agent SDK/framework | Typed model workers and tracing | Never rely on it for HR authority/durability/effects |
| Durable workflow engine | Multi-day waits, human tasks, replay/recovery | Use when operational benefit exceeds engine complexity |
| Hybrid | Native ATS/HRIS authority + durable cross-system control + model workers | Recommended production path |
| RPA/browser automation | Supervised read/low-risk bridge when API absent | Reject for consequential writes, weak identity/receipts, inaccessible UI |
| Multi-agent role simulation | No default use | Reconsider only for independently owned typed subsystems with measured benefit |

## Material contradictions and resolved positions

### Human review versus automated decision support

Some product descriptions call any human involvement “human in the loop.” GDPR/EDPB and broader governance sources distinguish meaningful involvement from fabricated review; human-factors evidence warns about automation bias.

**Resolved position:** independent evidence access, score/observation timing, authority, alternatives, time, accountability, and contest are enforceable properties. A click does not change the model's decision into a human one.

### Bias audit versus fairness/validity

NYC specifies a particular audit regime; EEOC uses selection-procedure and adverse-impact/validity concepts; NIST describes broader socio-technical bias; accessibility law/guidance covers disability barriers.

**Resolved position:** perform each applicable legal calculation, but do not merge them into a universal pass. Product release requires validity, subgroup/error, accessibility, human, process, and severe-tail evidence.

### Demographic data minimization versus disparity measurement

Fairness measurement may require protected-characteristic data; selection privacy requires preventing misuse.

**Resolved position:** lawful, separately governed monitoring compartment, pseudonymous joins, minimum access/cells, and no selector/recruiter exposure. Lack of permitted/adequate data is reported as an evidence limitation, not “fairness passed.”

### Transparency versus gaming/privacy

Candidates need notice, meaningful information, and contestability, but exposing personal data, security controls, or manipulable item banks can cause harm.

**Resolved position:** explain purpose, data categories, role of automation, material criteria/process, limitations, alternatives, rights, and contact. Protect other people's data and assessment security; do not use secrecy to hide inability to validate.

### Provider-managed state versus application state

Agent/model providers offer sessions/background execution; ATS/HRIS vendors offer workflows/events.

**Resolved position:** use provider mechanisms as replaceable execution aids. Application-owned case/decision/approval/effect contracts and domain systems remain authoritative.

### Exactly-once claims versus remote effects

Workflow/platform literature may describe exactly-once processing inside a boundary; networks and external systems leave ambiguity.

**Resolved position:** name the exact boundary. For remote effects use semantic idempotency, receipts, `unknown`, read-back, fencing, and reconciliation.

### Retain for defense/audit versus delete/minimize

Employment recordkeeping, claims, audit, privacy rights, vendor terms, and storage limitation can point in different directions.

**Resolved position:** record-class policy with triggers, applicable minima/maxima, holds, access, and copy inventory. Legal/records owners resolve conflicts; the model never chooses.

### HR source of authority versus IAM source of access truth

HR systems originate employment facts; directories/IAM own account/entitlement state.

**Resolved position:** source and effect authority are different. HR publishes versioned lifecycle facts; IAM decides/executes access effects and proves real state.

### Current policy versus approved historical policy

Long-running cases need reproducibility, but safety revocations or new law/policy can supersede old authority.

**Resolved position:** retain historical versions for audit; re-evaluate restrictive current policy/revocations at commit. New leniency does not silently expand old approvals.

## Claims deliberately excluded

- that an agent is required for HR transformation;
- that a human reviewer automatically makes an automated decision safe or lawful;
- that the four-fifths rule, a vendor bias audit, statistical parity, a model card, or an impact assessment certifies fairness;
- that historical hiring/performance/retention outcomes are ground truth for model training;
- that face, voice, gaze, emotion, personality, or “culture fit” inference is reliable or appropriate;
- that WCAG conformance alone satisfies every accessibility/accommodation obligation;
- that one privacy law, retention period, or notice applies globally;
- that a vendor's API, webhook, agent, plugin, or MCP annotation guarantees least privilege, idempotency, or correctness;
- that SCIM/HR-driven provisioning transfers access authority to HR;
- that a workflow engine or provider background mode makes remote effects exactly once;
- that full prompts, interview transcripts, or personnel records must be logged for audit;
- that multi-agent organization mirroring improves segregation of duties;
- that counsel/compliance/audit roles can be simulated by model personas.

## Jurisdictional uncertainty register

| Uncertainty | Why unresolved here | Deployment action |
|---|---|---|
| Employer/worker/contractor coverage | Statutes differ by size, entity, relationship, and facts | Counsel-approved applicability profile |
| Multi-location/remote work | Job location, residence, employing entity, and processing location may interact | Resolve facts; legal interpretation; multiple-applicable policy |
| Works council/collective bargaining | Information/consultation/consent rights vary | Labor-relations owner before pilot |
| Automated-decision definition | “Substantially assist,” material influence, profiling, and human involvement vary | Inventory actual use and reviewer behavior; do not rely on product label |
| Bias audit/impact assessment | Scope, auditor independence, publication, timing, data, and metric differ | Jurisdiction-specific procedure and evidence owner |
| Retention/deletion/holds | Record class and claims/charge/litigation rules differ | Versioned records schedule and hold workflow |
| Notice/consent/contest | Timing/content/rights vary; consent may be problematic in employment power imbalance | Lawful basis and affected-person design by jurisdiction |
| Cross-border/vendor processing | Transfer, residency, subprocessor, training, and access terms vary | Data map, contracts, configuration and verification |
| Background/medical/biometric use | Federal/state/local and special-data rules overlap | Separate restricted workflow; counsel and specialist review |
| AI Act implementation | Guidance/standards/timelines continued evolving in 2026 | Track final Commission/member-state guidance and standards |

## Research gaps and limitations

- No public source can establish the validity, fairness, accessibility, or security of a future local deployment; those require local artifacts and tests.
- Official vendor documentation is incomplete evidence for failure prevalence, tenant configuration, product tier, contractual guarantees, and undocumented behavior.
- Many employment/AI rules are rapidly changing; current official pages may lag enacted amendments or future effective dates.
- EEOC/DOJ/FTC/ICO/DSIT guidance documents have different legal weight and can be revised or rescinded; the blueprint uses them as control evidence, not as substitute law.
- Protected-characteristic data availability, quality, disclosure risk, and small samples can limit statistical conclusions.
- Disability is heterogeneous; aggregate metrics and WCAG checks cannot replace accommodation and affected-user evidence.
- Research did not attempt an exhaustive country-by-country labor/privacy/AI law survey, union contract analysis, or professional validation study.
- No primary source supports autonomous hiring/firing/promotion/compensation as a safe default; the blueprint prohibits it.
- Provider model quality, data terms, residency, caching, safety, and retention must be reverified for the exact enterprise contract and release.

## Refresh triggers

Review immediately when:

- EEOC, DOJ, FTC, DOL, state/local regulators, EU institutions, UK ICO/EHRC, or another deployed jurisdiction changes employment/AI/privacy/accessibility guidance or enforcement;
- the EU AI Act/Omnibus timeline, high-risk classification guidance, harmonized standards, or national enforcement rules change;
- Colorado ADMT rules finalize or another U.S. jurisdiction enacts/amends automated-employment requirements;
- an ATS, HRIS, assessment, background, identity, communication, model, MCP/plugin, or iPaaS vendor changes API, scope, model, feature, subprocessors, retention/training, region, price, or support;
- a job analysis, selection procedure, rubric, threshold, policy, retention schedule, collective agreement, or organization structure changes;
- a fairness/accessibility signal, complaint, contest, correction, regulator inquiry, incident, near miss, or cross-person/tenant error occurs;
- the agent gains a new data compartment, region, worker type, cohort selector, external recipient, write effect, long-lived memory, or multi-agent delegation;
- quarterly review is due for selection-support deployments or semiannual review for administrative-only use.

## Source register

### Employment selection, disability, and records

| Source | Evidence used | Limitation |
|---|---|---|
| [EEOC Uniform Guidelines Q&A](https://www.eeoc.gov/laws/guidance/questions-and-answers-clarify-and-provide-common-interpretation-uniform-guidelines) | Selection procedure, adverse impact, four-fifths caveats, validity, alternatives, records | U.S. federal context; application requires facts and qualified interpretation |
| [EEOC employment tests and selection procedures](https://www.eeoc.gov/laws/guidance/employment-tests-and-selection-procedures) | Job-related tests, ADA/Title VII/ADEA concerns, post-offer medical boundary | General guidance, not local validation |
| [DOJ algorithms, AI, and disability discrimination in hiring](https://www.ada.gov/resources/ai-guidance/) | Screen-out, facial/voice risks, accommodation, accessible alternatives | Technical assistance; states its nonbinding guidance status |
| [ADA statute/resources](https://www.ada.gov/law-and-regs/ada/) | Employment scope, qualification standards, accommodation baseline | Coverage and case law require counsel |
| [EEOC pre-employment medical questions](https://www.eeoc.gov/pre-employment-inquiries-and-medical-questions-examinations) | Stage limits and separate confidential medical files | U.S. federal context; exceptions/applicability vary |
| [EEOC recordkeeping requirements](https://www.eeoc.gov/employers/recordkeeping-requirements) | Personnel, payroll, benefit/merit record examples | Not a complete records schedule |
| [FTC background checks for employers](https://www.ftc.gov/business-guidance/resources/background-checks-what-employers-need-know) | FCRA authorization and pre-/post-adverse-action process | State/local rules and exact use require review |
| [USCIS Form I-9 employer guide](https://www.uscis.gov/sites/default/files/document/guides/E3en.pdf) | I-9 retention formula | Exact current form/process and immigration rules must be refreshed |
| [OPM structured interviews](https://www.opm.gov/policy-data-oversight/assessment-and-selection/structured-interviews) | Same job-related questions, rating scale, structured procedure | Federal HR practice, not universal legal/validation proof |

### Automated employment and AI regulation

| Source | Evidence used | Limitation |
|---|---|---|
| [NYC automated employment decision tools](https://www.nyc.gov/site/dca/about/automated-employment-decision-tools.page) | Annual bias audit/public summary/notices for covered use | Coverage/definitions/rules are NYC-specific |
| [California CRD ADS regulation announcement](https://calcivilrights.ca.gov/2025/06/30/civil-rights-council-secures-approval-for-regulations-to-protect-against-employment-discrimination-related-to-artificial-intelligence/) | Effective date, FEHA application, four-year ADS-employment record statement | Use final codified text and counsel for exact obligations |
| [California final ADS regulation text](https://calcivilrights.ca.gov/wp-content/uploads/sites/32/2025/06/Final-Text-regulations-automated-employment-decision-systems.pdf) | Definitions and covered employment-agent/ADS concepts | California only; amendments/codification should be checked |
| [Illinois AI Video Interview Act](https://www.ilga.gov/Legislation/ILCS/Articles?ActID=4015&ChapterID=68&Print=) | Notice, explanation, consent, sharing, deletion, reporting | Narrow covered scenario; current Public Acts may update compiled text |
| [Colorado AG ADMT rulemaking](https://coag.gov/ai/) | SB26-189 replacement and 2027-01-01 effective date; active rulemaking | Rules were not final on research date |
| [EU Artificial Intelligence Act](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX:32024R1689) | Employment high-risk category, workplace emotion prohibition, system obligations | Amended application timeline must be read with 2026 Omnibus |
| [European Commission AI Act framework/timeline](https://digital-strategy.ec.europa.eu/en/policies/regulatory-framework-ai) | 2026 Omnibus and 2027-12-02 Annex III timeline | Implementation pages evolve; legal text controls |
| [European Commission AI Omnibus in force](https://digital-strategy.ec.europa.eu/en/news/ai-omnibus-enters-force) | July 2026 entry and extended high-risk timeline | Summary page; consult final legal text |

### Privacy, accessibility, and assurance

| Source | Evidence used | Limitation |
|---|---|---|
| [GDPR](https://eur-lex.europa.eu/eli/reg/2016/679/oj) | Purpose/minimization/storage, sensitive data, automated decisions, DPIA, rights | Member State employment law and applicability matter |
| [EDPB automated decision-making/profiling guidance](https://www.edpb.europa.eu/documents/guideline/automated-decision-making-and-profiling_en) | Solely automated decisions and meaningful human involvement concepts | Endorsed older guidance; monitor statutory/case-law updates |
| [ICO AI recruitment audit report](https://ico.org.uk/media/about-the-ico/documents/4031620/ai-in-recruitment-outcomes-report.pdf) | Accuracy/bias/privacy/provider responsibility findings | Consensual audit sample; cannot establish market prevalence |
| [ICO recruitment and selection guidance](https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/employment/recruitment-and-selection/) | Recruitment data lifecycle and automated decisions | Draft/under review for UK statutory changes at research date |
| [UK responsible AI in recruitment](https://www.gov.uk/government/publications/responsible-ai-in-recruitment-guide/responsible-ai-in-recruitment) | Procurement, impact/DPIA, supplier evidence, audit, pilot, transparency | Guidance, not legal assurance |
| [W3C WCAG 2.2](https://www.w3.org/TR/WCAG22/) | Stable accessible web criteria and stated coverage limits | Web conformance does not cover every person/process/obligation |
| [NIST AI RMF Core](https://airc.nist.gov/airmf-resources/airmf/5-sec-core/) | Roles, oversight, TEVV, appeals, third parties, monitoring | Voluntary; AI RMF revision underway |
| [NIST SP 1270](https://www.nist.gov/publications/towards-standard-identifying-and-managing-bias-artificial-intelligence) | Systemic, human, computational bias | General framework, not an employment metric standard |
| [NIST AI 600-1](https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.600-1.pdf) | Generative-AI lifecycle risk and evaluation controls | Cross-sectoral; local profile required |

### HRIS, ATS, identity, and reliability mechanics

| Source | Evidence used | Limitation |
|---|---|---|
| [Greenhouse Harvest API](https://docs.greenhouse.io/harvest.html) | Endpoint permissions, broad endpoint data, rate limits, pagination, temporary attachment URLs | Plan/version/account behavior varies |
| [Greenhouse recruiting webhooks](https://docs.greenhouse.io/webhooks.html) | Signatures, delivery ID, retries, payload/source reread | Does not provide full ordered event log/exactly once |
| [Workday Events REST APIs](https://developer.workday.com/documentation/GUID-0df5cd55-e578-43d3-b58f-ae98825d1df0-enHYPHENus) | Business-process IDs/status/effective/due dates/steps/cancel/rescind | Tenant/service/version configuration varies |
| [SAP effective-dated OData v2](https://help.sap.com/docs/successfactors-platform/sap-successfactors-api-reference-guide-odata-v2/effective-dated-query-in-odata) and [CompoundEmployee 1H 2026](https://help.sap.com/docs/successfactors-employee-central/employee-central-compound-employee-api/compoundemployee-api-general-information-about-full-snapshot-and-delta-transmission-modes) | Explicit time-slice/default-date and payroll/benefits replication modes | Release, entity, field, auditing and tenant behavior require qualification |
| [Oracle HCM assignment REST](https://docs.oracle.com/en/cloud/saas/human-resources/farws/api-workers-work-relationships-assignments.html) | Worker relationship/assignment resource and action surface | Public resource version does not prove live-tenant configuration |
| [Greenhouse Assessment API](https://docs.greenhouse.io/assessment.html) | Definition/instance/status flow and active-candidate limitations | Provider result does not establish validity/fairness/accessibility |
| [Microsoft Graph calendar v1.0](https://learn.microsoft.com/en-us/graph/api/calendar-post-events?view=graph-rest-1.0) | Event lifecycle and client transaction identity | Does not prove attendance/interview completion |
| [Microsoft Graph `sendMail` v1.0](https://learn.microsoft.com/en-us/graph/api/user-sendmail?view=graph-rest-1.0) and [Slack `chat.postMessage`](https://api.slack.com/methods/chat.postMessage) | Representative email/chat acceptance and remote identity | Provider success does not prove delivery, reading, review or durable HR record |
| [Checkr API v1](https://docs.checkr.com/) | Invitation, report/webhook and idempotency mechanics | Background result is not employer adjudication |
| [Acrobat Sign API usage](https://developer.adobe.com/acrobat-sign/docs/overview/developer_guide/apiusage) | Agreement/status/document-version mechanics | Signed status does not prove HR acceptance or correct terms |
| [SCIM core schema RFC 7643](https://www.rfc-editor.org/info/rfc7643/) | User active/enterprise manager/employeeNumber semantics | Service provider defines important meanings |
| [SCIM protocol RFC 7644](https://www.rfc-editor.org/info/rfc7644/) | PATCH atomicity, ETags/If-Match, resource operations | Provider support/conformance must be tested |
| [SCIM cursor update RFC 9865](https://www.rfc-editor.org/info/rfc9865/) and [security-event profile RFC 9967](https://www.rfc-editor.org/info/rfc9967/) | Post-2015 SCIM updates and current capability/status boundary | Provider support must be discovered; neither transfers IAM authority to HR |
| [Microsoft HR-driven provisioning](https://learn.microsoft.com/en-us/entra/identity/app-provisioning/what-is-hr-driven-provisioning) | HR source facts feeding JML provisioning | Microsoft product mechanism; HR still must not own access policy |
| [AWS idempotent APIs](https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/) | Caller intent, semantic idempotency, retry reasoning | Engineering guidance; adapter-specific behavior controls |
| [W3C Trace Context](https://www.w3.org/TR/trace-context/) | Trace correlation propagation | Do not place PII/secrets in propagated context |
| [NIST SP 800-61 Rev. 3](https://csrc.nist.gov/pubs/sp/800/61/r3/final) | Incident preparation, detection, response, recovery | General incident framework; HR runbooks specialize it |

## Traceability to derived guides

| Evidence/decision area | Applied in |
|---|---|
| Category fit, human authority, Stage 0–6 gates | [01 — Workload fit, authority, and stages](../../agents/hr-talent-operations-agent/01-workload-fit-authority-and-stages.md) |
| Hybrid architecture, runtime/model choice, vendor admission, connector mechanics | [02 — Architecture, runtime, models, and integrations](../../agents/hr-talent-operations-agent/02-reference-architecture-runtime-models-and-integrations.md) |
| Identity graph, effective state, context/compaction/memory, planning | [03 — Identity, lifecycle state, context, memory, and orchestration](../../agents/hr-talent-operations-agent/03-identity-lifecycle-state-context-memory-and-orchestration.md) |
| Selection procedures, interview evidence, human decisions, communications | [04 — Requisitions, recruiting, interviews, and decisions](../../agents/hr-talent-operations-agent/04-requisitions-recruiting-interviews-and-human-decisions.md) |
| JML tasks, IAM boundary, effects, idempotency, reconciliation, recovery | [05 — Onboarding, offboarding, tools, effects, and recovery](../../agents/hr-talent-operations-agent/05-onboarding-offboarding-tools-effects-and-recovery.md) |
| Compartments, threat model, privacy, retention/deletion/holds | [06 — Sensitive records, security, privacy, retention, and deletion](../../agents/hr-talent-operations-agent/06-sensitive-records-security-privacy-retention-and-deletion.md) |
| Fairness/accessibility metrics, evaluation, failure injection, tracing/SLOs | [07 — Fairness, accessibility, evaluation, and observability](../../agents/hr-talent-operations-agent/07-fairness-accessibility-evaluation-and-observability.md) |
| Deployment, scale/cost, degradation, release, incidents, continuous evolution | [08 — Deployment, scale, incidents, and evolution](../../agents/hr-talent-operations-agent/08-deployment-scale-incidents-and-governed-evolution.md) |
| Provider-specific adapters, identities, finality, lifecycle worked flows and conformance tests | [09 — Adapter qualification and lifecycle playbooks](../../agents/hr-talent-operations-agent/09-adapter-qualification-and-lifecycle-playbooks.md) |

## Research saturation statement

Additional primary-source searches stopped changing the architecture boundary, authority ceiling, identity/effect model, fairness/accessibility posture, data compartments, or staged roadmap. They continued to reveal jurisdiction and timeline volatility, which is represented as a policy registry and refresh obligation rather than collapsed into static global advice.

Pass 2 is suitable as a production engineering and qualification reference, not as legal sign-off, a validation study, a bias audit, an accessibility conformance report or proof of live-provider behavior. An implementer must validate exact product/contract versions and tenant configuration, obtain local professional decisions, and incorporate reviewed incident/contest evidence without exposing personal data.
