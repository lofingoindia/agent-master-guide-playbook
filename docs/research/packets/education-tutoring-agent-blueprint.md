# Education and Tutoring Agent Blueprint: Research Packet

**Research date:** 2026-08-31  
**Decision horizon:** Revalidate law, safeguarding guidance, assessment rules, standards versions, and provider behavior before each release.  
**Scope:** Bounded learner-facing formative tutoring inside an institution-governed course.  
**Excluded authority:** Final grades, admissions, discipline, special-education identification or placement, diagnosis, high-stakes answers, surveillance or proctoring decisions, and replacement of teacher judgment.

## Executive finding

The safest useful product is not an autonomous teacher and not an answer bot. It is a **bounded formative-practice workflow** whose generative component can explain, question, and choose from approved instructional moves. Identity, enrollment, curriculum, assessment mode, accommodations, consent, effects, and evidence ownership remain deterministic or human-governed.

The primary outcome is credible evidence that a learner can perform later with less assistance. Assisted task completion, conversation length, praise, and model-rated “mastery” are not sufficient. The initial production loop should therefore require an attempt, reveal help progressively, fade support, run an unassisted check when allowed, record the degree of assistance, and route uncertain or consequential conclusions to a teacher.

Three findings materially shape the design:

1. Education interoperability is mature enough to avoid a new universal data model, but no one standard is the authority for everything. CASE carries competency graphs; OneRoster and Ed-Fi carry institutional data; LTI launches tools; QTI carries assessment content and results. Local policy still decides what is authoritative.
2. Learner-facing generative AI needs explicit anti-dependency and relationship-safety controls. Current UK product-safety guidance calls for genuine learner attempts, progressive hints, friction before full solutions, avoidance of personhood or secret-keeping language, time limits, and human escalation.
3. Assisted performance can conceal weaker learning. A 2025 randomized high-school mathematics study found that unguarded GPT assistance improved practice performance but reduced later unassisted performance; the result is not universal evidence against AI tutoring, but it makes delayed, unassisted transfer a mandatory evaluation dimension.

## Research questions

- What is the smallest tutoring workload that adds value without claiming teacher authority?
- Which facts belong to the SIS, LMS, curriculum registry, assessment service, local runtime, or a human?
- What constitutes evidence of learning rather than evidence that the agent supplied an answer?
- How should age, role, consent, jurisdiction, accommodations, language, and retention be represented?
- Which standards and provider interfaces are current, final, or still emerging?
- Where must execution stop, ask, escalate, or reconcile?
- How can seven memory levels remain useful without constructing an uncorrectable learner profile?
- Which safety, accessibility, fairness, academic-integrity, and safeguarding claims require deterministic tests?
- What evidence is required to advance from a read-only pilot to bounded production use?

## Method and evidence rules

Research prioritized primary law and regulator pages, official standards, official provider documentation, government education guidance, official repositories, systematic evidence sources, and peer-reviewed studies. Secondary commentary was excluded where a primary source was available.

Interpretation rules:

- A technical specification describes exchange semantics, not institutional authority.
- A framework or product-safety guide is guidance unless a law, regulation, contract, or local policy makes it binding.
- A narrow trial establishes evidence for its studied context, not a universal pedagogy.
- A provider certification does not prove that every tenant, permission grant, webhook, or failure mode is safe.
- “Mastery” is a claim supported by evidence and policy, not a model confidence score.
- Legal summaries in this packet are engineering constraints, not legal advice.

## Domain boundary and neighboring categories

| Category | Owns | Does not become part of this blueprint |
|---|---|---|
| Education and tutoring | Course-bound formative practice, hinting, explanations, evidence capture, teacher handoff | Institutional search, editorial publishing, HR decisions, support-ticket operations |
| Enterprise knowledge | Retrieval and synthesis over organizational knowledge | Learner modeling or pedagogy |
| Content and editorial | Authoring, review, publication, style, rights workflow | Individual learner adaptation or assessment evidence |
| HR and talent operations | Employee lifecycle and workplace learning administration | Student grading, admissions, or K–12 records |
| Customer support | Case resolution, entitlements, service recovery | Curriculum alignment or learning evidence |

## Current standards and policy snapshot

| Area | Current finding at research date | Engineering consequence |
|---|---|---|
| Competencies | [CASE 1.1](https://standards.1edtech.org/case/) is a final 1EdTech standard for frameworks, items, associations, and rubrics | Preserve issuer identifiers and framework version; do not flatten a standards graph into model text |
| Rostering | [OneRoster 1.2](https://standards.1edtech.org/oneroster/specifications/standards/v1p2) is final; CSV 1.2.1 is an errata release | Treat membership and class facts as imported institutional state with freshness metadata |
| Tool launch | [LTI 1.3 and LTI Advantage](https://www.1edtech.org/standards/lti) use OIDC, OAuth 2, and JWT; legacy LTI support ended in 2022 | Validate issuer, audience, deployment, nonce, role, resource link, and context; do not accept identity from chat text |
| Assessment exchange | [QTI 3.0](https://www.1edtech.org/standards/qti/index) integrates accessibility and supports adaptive testing; 3.0.1 is the current maintenance update | Preserve item policy and accessibility metadata; never infer that import permission grants high-stakes answer permission |
| Course packages | [Common Cartridge 1.4](https://www.1edtech.org/standards/cc) is Candidate Final, not a final release | Pin tested profiles rather than advertise generic 1.4 compatibility |
| Security | [1EdTech Security Framework 1.1](https://www.1edtech.org/standards/security-framework) is final | Use its OAuth/JWT profiles where applicable, plus local threat modeling and key lifecycle controls |
| K–12 data | [Ed-Fi Data Standard 6.1](https://docs.ed-fi.org/reference/data-exchange/data-standard/) was released in May 2026; the supported-version page must be checked with the deployed ODS/API | Qualify exact API/data-standard combinations; early-access special-education structures do not grant decision authority |
| Accessibility | [WCAG 2.2](https://www.w3.org/TR/WCAG22/) is the current W3C Recommendation; US public-entity Title II requirements presently reference WCAG 2.1 AA | Target WCAG 2.2 AA while mapping jurisdictional conformance separately |
| US Title II timing | The DOJ’s [2026 small-entity guide](https://www.ada.gov/resources/small-entity-compliance-guide/) reflects an interim extension to 2027 for entities of 50,000 or more and 2028 for smaller entities and special districts | Treat dates as a dated legal snapshot and revalidate; do not delay accessibility engineering |
| England safeguarding | [KCSIE 2025](https://www.gov.uk/government/publications/keeping-children-safe-in-education--2) applies through 2026-08-31; the 2026 version starts 2026-09-01 | Version safeguarding policy by effective date and institution; the model does not choose the applicable edition |
| EU education AI | [Regulation (EU) 2024/1689](https://eur-lex.europa.eu/eli/reg/2024/1689/oj?locale=en) lists specified educational access, outcome-evaluation, level-assessment, and test-monitoring uses as high risk | Classify the actual intended use; keep admissions, final grading, level placement, and proctoring outside the tutor |
| Children’s AI safety | UK [Generative AI product safety standards](https://www.gov.uk/government/publications/generative-ai-product-safety-standards/generative-ai-product-safety-standards) were updated in January 2026 | Make progressive hints, anti-offloading, relationship boundaries, time controls, and human escalation testable product behavior |

## Evidence-to-decision register

### ET-01 — Start with a deterministic alternative

**Evidence.** Agent engineering guidance recommends the simplest composable pattern and distinguishes fixed workflows from open-ended agents. Education has strong deterministic assets: item banks, prerequisite graphs, worked examples, spaced schedules, scoring rules, and teacher-authored hint ladders. See [Anthropic’s agent design guidance](https://www.anthropic.com/engineering/building-effective-agents).

**Decision.** Ship a non-generative baseline first: approved content retrieval, fixed hint cards, deterministic attempt scoring, and a teacher queue. Add generation only where it measurably improves explanation or question quality without weakening safety or learning.

**Rejected design.** A general autonomous tutor that invents lessons, assessment policy, and interventions.

### ET-02 — Optimize for independent evidence, not assisted completion

**Evidence.** The [IES practice guide on instruction and study](https://ies.ed.gov/ncee/wwc/PracticeGuide/1) supports retrieval, spacing, interleaving, explanatory questioning, and appropriate use of worked examples. A [2025 PNAS randomized study](https://doi.org/10.1073/pnas.2422633122) found materially different outcomes between ordinary GPT assistance and a guarded tutor in one high-school mathematics context.

**Decision.** Store whether evidence was independent, delayed, transferred to a new item, and how much help preceded it. Product evaluation must include later unassisted measures.

**Limit.** The PNAS result is context-specific; it motivates a guardrail and measurement requirement, not a universal effect-size claim.

### ET-03 — Use one bounded tutoring loop

**Evidence.** UK [product-safety standards](https://www.gov.uk/government/publications/generative-ai-product-safety-standards/generative-ai-product-safety-standards) recommend genuine attempts, progressive hints, restricted full solutions, and monitoring for cognitive offloading.

**Decision.** The first loop handles one teacher-approved goal, one course context, a finite hint ladder, one independent check, and one evidence receipt. Hard caps end or hand off the session.

### ET-04 — Curriculum alignment is a versioned mapping problem

**Evidence.** CASE models framework items and associations but does not select the institution’s curriculum. Jurisdictional sources such as [Common Core](https://corestandards.org/mathematics-standards/), [NGSS](https://www.nextgenscience.org/standards), and the [England national curriculum](https://www.gov.uk/government/publications/national-curriculum-in-england-framework-for-key-stages-1-to-4/the-national-curriculum-in-england-framework-for-key-stages-1-to-4) differ in authority and organization.

**Decision.** Pin `framework_id`, issuer, jurisdiction, version, local course mapping, and teacher-approved goal. The model may propose a mapping; a curriculum owner approves it.

### ET-05 — Teachers retain consequential educational authority

**Evidence.** UK [generative AI guidance for education](https://www.gov.uk/government/publications/generative-artificial-intelligence-in-education/generative-artificial-intelligence-ai-in-education) says technology should not replace the teacher relationship and assigns responsibility to educational professionals. The EU AI Act separately highlights consequential education uses.

**Decision.** The tutor cannot write final grades, decide admissions or discipline, diagnose, determine special-education eligibility or placement, set high-stakes assessment answers, or make proctoring decisions. It produces evidence and drafts for authorized humans.

### ET-06 — Identity, role, age band, consent, and purpose are explicit inputs

**Evidence.** [FERPA](https://studentprivacy.ed.gov/ferpa) protects education records and defines consent requirements; vendor use under the school-official exception requires institutional control and legitimate interest. The FTC’s [COPPA FAQ](https://www.ftc.gov/business-guidance/resources/complying-coppa-frequently-asked-questions) limits school authorization to the educational context and school benefit. [GDPR Article 8](https://eur-lex.europa.eu/legal-content/EN/TXT/?toc=OJ%3AL%3A2016%3A119%20%3ATOC&uri=uriserv%3AOJ.L_.2016.119.01.0001.01.ENG) sets an EU default age of 16 for a specific consent-based online-service case, with member-state variation down to 13.

**Decision.** Resolve identity from institutional authentication, not a claim in conversation. Store the lawful basis/policy basis, authorized purpose, consent artifact where applicable, actor relationship, age band, jurisdiction, and expiry. Do not incorrectly reduce all processing to consent.

### ET-07 — Source ownership and reconciliation must be declared

**Evidence.** OneRoster, LTI, provider APIs, and asynchronous messaging each expose partial and sometimes delayed state. Google Classroom says an integration’s cached coursework can become stale and teachers retain control; [Calendar incremental sync](https://developers.google.com/workspace/calendar/api/guides/sync) can return HTTP 410 and require a full resync.

**Decision.** SIS owns institutional identity and enrollment; LMS owns course/assignment state; curriculum registry owns approved mappings; assessment service owns item/result facts; the local ledger owns tutor runs and effect intents; providers own final delivery state. Every mirror has a cursor, observed time, source version, and reconciliation path.

### ET-08 — A learner model is a corrigible projection over evidence

**Evidence.** No interoperability standard turns heterogeneous attempts into a universally valid mastery score. Evidence strength depends on task, assistance, time, modality, opportunity, and scoring validity.

**Decision.** Maintain an append-only evidence ledger and a separately versioned projection with `insufficient_evidence`, `emerging`, `developing`, `secure_candidate`, and `teacher_confirmed`. Only an authorized teacher can set `teacher_confirmed`; corrections rebuild the projection.

### ET-09 — Accessibility and language support are not diagnosis

**Evidence.** WCAG 2.2 addresses web accessibility but does not cover every learning or language need. [IDEA’s IEP team rule](https://sites.ed.gov/idea/regs/b/d/300.321) assigns decisions to a multi-party team; the [2024 assistive-technology guidance](https://sites.ed.gov/idea/idea-files/at-guidance/) preserves that authority. US civil-rights guidance requires [meaningful access for English learners and communication with limited-English-proficient parents](https://www.ed.gov/laws-and-policy/civil-rights-laws/race-color-and-national-origin-discrimination/race-color-and-national-origin-discrimination-key-issues/equal-education-opportunities-english).

**Decision.** Apply approved accommodations and learner-selected presentation preferences. Offer plain language, translation, multimodal alternatives, keyboard and assistive-technology compatibility. Never infer a disability, language status, IEP need, or placement from behavior.

### ET-10 — Memory needs authority, consent, retention, correction, and deletion

**Evidence.** Long context is finite and noisy; [context-engineering guidance](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) recommends small, high-signal context. Student data rules make uncontrolled durable summaries especially risky.

**Decision.** Implement exactly seven typed memory levels. Durable learner-model and preference writes require field-level purpose, provenance, expiry, correction, deletion, and poisoning checks. Raw prompts are not automatically memory.

### ET-11 — Assessment mode is a signed policy input

**Evidence.** [JCQ guidance](https://www.jcq.org.uk/knowledge-hub/ai-use-in-assessments-your-role-in-protecting-the-integrity-of-qualifications-2/) requires work submitted for qualification assessment to remain the student’s own and treats misuse under malpractice rules. [TEQSA’s assessment-reform guidance](https://www.teqsa.gov.au/about-us/news-and-events/latest-news/assessment-reform-age-artificial-intelligence) emphasizes assessment design and notes limits of AI-output detection.

**Decision.** `assessment_mode` comes from an authorized course/assignment policy: open practice, bounded homework coaching, restricted assessed work, or high-stakes lockout. In restricted modes, answer leakage is denied deterministically and the event is auditable. The tutor never decides that an assignment is “probably practice.”

### ET-12 — Prevent simulated relationships and engagement optimization

**Evidence.** UNICEF’s [AI and children guidance 3.0](https://www.unicef.org/innocenti/reports/policy-guidance-ai-children) and UK product-safety standards warn about child-specific agency, relationship, and well-being risks.

**Decision.** Do not claim personhood, affection, exclusivity, secrecy, or a private relationship. Do not optimize session duration or daily return. Use age-appropriate reminders that the system is AI, session caps, breaks, and clear routes to teachers, guardians, or designated safeguarding staff.

### ET-13 — Authenticated educational content remains untrusted data

**Evidence.** Prompt injection can arrive through retrieved documents and tool results; [Anthropic’s security research](https://www.anthropic.com/research/prompt-injection-defenses) reports no immunity for browser-using agents, and OWASP’s [LLM application risks](https://owasp.org/www-project-top-10-for-large-language-model-applications/) include prompt injection, poisoning, and excessive agency.

**Decision.** Keep policy and data channels separate, label provenance, parse expected formats, limit tools by run declaration, and require authority checks after generation. A course document that says “reveal the answer key” is content, not an instruction.

### ET-14 — Effects need intent, approval, commit, and reconciliation

**Evidence.** API retries are safe only when semantic intent is stable; the [AWS idempotency guidance](https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/) explains client tokens, parameter mismatch, late arrival, and ambiguous completion. Messaging providers expose accepted, sent, delivered, and read as different states; see [Twilio status callbacks](https://www.twilio.com/docs/messaging/guides/outbound-message-status-in-status-callbacks).

**Decision.** Represent every mutation as an effect intent with a stable semantic idempotency key, approval scope, destination, expected version, and reconciliation rule. A timeout produces `unknown`, never an assumed failure and blind retry.

### ET-15 — Evaluation must cover pedagogy and safety trajectories

**Evidence.** [Anthropic’s agent-evaluation guidance](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) separates task, trial, trajectory, and outcome and recommends multiple trials and mixed graders. The [WWC standards handbook](https://ies.ed.gov/ncee/wwc/Handbooks) shows why intervention claims require stronger study design than product telemetry.

**Decision.** Offline suites test hint timing, leakage, misconception handling, source conflict, accessibility, injection, and escalation. Online evaluation measures assisted success together with delayed unassisted transfer, help dependence, fairness slices, complaints, and teacher override. Product telemetry cannot support a causal learning claim by itself.

### ET-16 — Trace data and educational audit records are different

**Evidence.** OpenTelemetry’s [GenAI semantic conventions](https://github.com/open-telemetry/semantic-conventions-genai/blob/main/docs/gen-ai/gen-ai-spans.md) are still developing and warn that prompts and outputs may contain sensitive data. [W3C Trace Context](https://www.w3.org/TR/trace-context/) identifiers must not carry personally identifiable information.

**Decision.** Traces are sampled operational data with redacted attributes. Audit records are durable, minimal, access-controlled facts about authority, policy, approvals, effects, disclosures, evidence corrections, and safeguarding actions. Neither stores raw learner conversations by default.

### ET-17 — Safeguarding follows local procedure, not model diagnosis

**Evidence.** England’s current [KCSIE overview](https://www.gov.uk/government/publications/keeping-children-safe-in-education--2/part-one-overview-for-all-staff) directs staff to act immediately, report to the designated safeguarding lead or deputy, follow local policy, and record actions and rationale. FERPA’s [health-or-safety emergency exception](https://studentprivacy.ed.gov/faq/how-does-school-know-when-health-or-safety-emergency-exists-so-disclosure-may-be-made-under) depends on a case-specific articulable and significant threat determined by the institution.

**Decision.** The product detects a bounded set of concern signals, gives non-diagnostic safety language, pauses ordinary tutoring, and routes a minimal receipt to the configured human path. It does not investigate, promise confidentiality, make a legal determination, or contact emergency services unless a separately authorized institutional workflow explicitly does so.

### ET-18 — Deploy for degradation and reconciliation, not continuous perfection

**Evidence.** Provider events can be delayed, duplicated, reordered, or absent. Canvas documents dynamic [API throttling](https://developerdocs.instructure.com/services/canvas/basics/file.throttling); Google Classroom push messages are notifications rather than full state and require a subsequent read; see [push best practices](https://developers.google.com/workspace/classroom/best-practices/push-notifications).

**Decision.** Queue by risk, reserve capacity for safeguarding and reconciliation, isolate tenants into cells, shed nonessential generation first, and fall back to approved static hints. Offline mode cannot serve stale high-stakes items or finalize effects.

### ET-19 — Roll out immutable behavior bundles

**Evidence.** Model behavior changes when prompts, policies, retrieval, tools, content, or providers change. [Google SRE canary guidance](https://sre.google/workbook/canarying-releases/) treats releases as hypotheses evaluated against control populations and rollback criteria.

**Decision.** A release pins model, prompt, tool schema, policy, curriculum mapping, retrieval index, safety rules, and evaluator versions. Shadow, canary, rollback, drift review, and a controlled feedback path operate on the bundle, not only on the model name.

### ET-20 — Emerging learning-context envelopes are informative, not foundational

**Evidence.** 1EdTech’s [Trusted Portable Learning Context](https://standards.1edtech.org/publications/rfc-trusted-context) is an RFC, not a final standard. It proposes governed, provenance-carrying context assembled from source systems for AI applications.

**Decision.** Track the RFC and borrow its provenance and release-time governance ideas, but do not create a production dependency until the profile stabilizes and passes local qualification.

## Contradictions and reconciliations

| Tension | Resolution |
|---|---|
| Personalization needs history; privacy favors minimization | Keep an evidence ledger and narrow, correctable projections; retrieve only fields needed for the current goal |
| Immediate help improves completion; productive struggle supports learning | Require an attempt, pace hints, and measure delayed independent transfer |
| A learner may prefer direct answers; assessed work may prohibit them | The signed assessment policy outranks conversational preference |
| Adaptive systems need inference; special-education decisions require teams | Use low-stakes pedagogical hypotheses that expire; never infer diagnoses, eligibility, or placement |
| Teachers need visibility; raw transcripts create surveillance and breach risk | Show structured receipts, evidence basis, and sampled excerpts only under policy |
| Provider webhooks reduce latency; provider state remains authoritative | Treat webhooks as wake-up signals, read the source, then reconcile |
| Accessibility standards aid conformance; inclusive learning exceeds conformance | Test WCAG plus language, cognitive load, modalities, assistive technology, and approved accommodations |
| Safety detection may protect learners; false positives can stigmatize | Use minimal labels, non-diagnostic language, human review, correction, and short retention |
| Global products seek one policy; education law is jurisdictional | Compile a signed institution policy bundle from jurisdiction, age, course, and assessment context |

## Rejected architectures

| Rejected approach | Why it fails |
|---|---|
| One omniscient learner profile in a vector database | Loses authority, time, consent, correction, and evidence strength; amplifies poisoning |
| Model selects the curriculum standard from free text | Confuses plausible similarity with approved local alignment |
| Full solution first, explanation second | Encourages offloading and contaminates learning evidence |
| “Mastery probability” as the source of truth | Hides heterogeneous evidence and grants the model educational authority |
| LLM decides whether work is high stakes | An ambiguous classification can leak protected answers |
| Direct gradebook writes with approval prompts | A chat approval is not necessarily grade authority; retries and stale versions remain unsafe |
| Long raw transcripts as teacher dashboards | Creates surveillance, overload, retention, and safeguarding risks |
| AI-output detector as the integrity control | Detection is unreliable and does not replace assessment design or evidence of authorship |
| Engagement-maximizing companion persona | Conflicts with child-centered, anti-dependency, and relationship-safety goals |
| Shared cross-school retrieval namespace | Makes a single filter or injection failure a cross-tenant disclosure |
| Webhook delivery treated as source truth | Notifications can be delayed, duplicated, reordered, or content-light |
| “Offline means everything still works” | Consent, enrollment, assessment mode, and effect state may be stale or unknowable |

## Source register

### Education interoperability and providers

- 1EdTech: [CASE 1.1](https://standards.1edtech.org/case/), [OneRoster 1.2](https://standards.1edtech.org/oneroster/specifications/standards/v1p2), [LTI](https://www.1edtech.org/standards/lti), [LTI implementation guide](https://standards.1edtech.org/lti/guides/implementation_guide/implementation-guide), [QTI](https://www.1edtech.org/standards/qti/index), [QTI accessibility](https://www.1edtech.org/standards/qti/accessibility), [Common Cartridge](https://www.1edtech.org/standards/cc), [Security Framework](https://www.1edtech.org/standards/security-framework).
- Ed-Fi Alliance: [Data Standard](https://docs.ed-fi.org/reference/data-exchange/data-standard/), [supported versions](https://docs.ed-fi.org/reference/roadmap/supported-versions/), [6.1 release](https://docs.ed-fi.org/blog/2026/05/11/).
- Instructure: [Canvas throttling](https://developerdocs.instructure.com/services/canvas/basics/file.throttling), [pagination](https://developerdocs.instructure.com/services/canvas/basics/file.pagination).
- Google: [Classroom coursework integration](https://developers.google.com/workspace/classroom/guides/coursework-integration), [Classroom push notifications](https://developers.google.com/workspace/classroom/best-practices/push-notifications), [Calendar incremental sync](https://developers.google.com/workspace/calendar/api/guides/sync), [Calendar push](https://developers.google.com/workspace/calendar/api/guides/push).
- Microsoft: [Graph education overview](https://learn.microsoft.com/en-us/graph/education-concept-overview), [submission retrieval](https://learn.microsoft.com/en-us/graph/api/educationsubmission-get?view=graph-rest-1.0).
- Twilio: [outbound status callbacks](https://www.twilio.com/docs/messaging/guides/outbound-message-status-in-status-callbacks).

### Privacy, civil rights, accessibility, and authority

- US Department of Education: [FERPA](https://studentprivacy.ed.gov/ferpa), [school-official legitimate interest](https://studentprivacy.ed.gov/faq/what-must-educational-agencies-or-institutions-do-ensure-only-school-officials-legitimate), [online tools and FERPA](https://studentprivacy.ed.gov/faq/i-want-use-online-tool-or-application-part-my-course-however-i-am-worried-it-violation-ferpa), [disclosure records](https://studentprivacy.ed.gov/faq/are-schools-required-record-disclosure-personally-identifiable-information-pii-students), [IDEA IEP team](https://sites.ed.gov/idea/regs/b/d/300.321), [assistive technology](https://sites.ed.gov/idea/idea-files/at-guidance/), [Section 504 FAQ](https://www.ed.gov/laws-and-policy/civil-rights-laws/disability-discrimination/frequently-asked-questions-disability-discrimination), [English learner access](https://www.ed.gov/laws-and-policy/civil-rights-laws/race-color-and-national-origin-discrimination/race-color-and-national-origin-discrimination-key-issues/equal-education-opportunities-english).
- FTC: [COPPA 2025 amendments](https://www.ftc.gov/legal-library/browse/federal-register-notices/16-cfr-part-312-coppa-final-rule-amendments), [COPPA FAQ](https://www.ftc.gov/business-guidance/resources/complying-coppa-frequently-asked-questions).
- European Union: [GDPR](https://eur-lex.europa.eu/legal-content/EN/TXT/?toc=OJ%3AL%3A2016%3A119%20%3ATOC&uri=uriserv%3AOJ.L_.2016.119.01.0001.01.ENG), [AI Act](https://eur-lex.europa.eu/eli/reg/2024/1689/oj?locale=en).
- W3C: [WCAG 2.2](https://www.w3.org/TR/WCAG22/).
- US DOJ: [Title II web and mobile rule](https://www.ada.gov/resources/2024-03-08-web-rule/), [2026 compliance guide](https://www.ada.gov/resources/small-entity-compliance-guide/).
- CAST: [Universal Design for Learning Guidelines 3.0](https://udlguidelines.cast.org/more/downloads/). Treat as design guidance, not a legal conformance standard.

### Pedagogy, assessment, children, and safety

- IES/WWC: [Organizing Instruction and Study to Improve Student Learning](https://ies.ed.gov/ncee/wwc/PracticeGuide/1), [WWC handbooks](https://ies.ed.gov/ncee/wwc/Handbooks).
- National Academies: [How People Learn II](https://nap.nationalacademies.org/initiative/committee-on-how-people-learn-ii-the-science-and-practice-of-learning).
- Bastani et al.: [Generative AI without guardrails can harm learning](https://doi.org/10.1073/pnas.2422633122).
- Kestin et al.: [AI tutoring in college physics](https://pmc.ncbi.nlm.nih.gov/articles/PMC12179260/). Useful but narrow; do not generalize the magnitude.
- UK DfE: [Generative AI in education](https://www.gov.uk/government/publications/generative-artificial-intelligence-in-education/generative-artificial-intelligence-ai-in-education), [product safety standards](https://www.gov.uk/government/publications/generative-ai-product-safety-standards/generative-ai-product-safety-standards), [AI support materials](https://www.gov.uk/government/collections/using-ai-in-education-settings-support-materials), [KCSIE](https://www.gov.uk/government/publications/keeping-children-safe-in-education--2).
- UNICEF: [Guidance on AI and children 3.0](https://www.unicef.org/innocenti/reports/policy-guidance-ai-children), [When AI becomes a friend](https://www.unicef.org/documents/when-ai-becomes-friend-child-rights-risks).
- UNESCO: [Guidance for generative AI in education and research](https://www.unesco.org/en/articles/guidance-generative-ai-education-and-research).
- TEQSA: [Assessment reform for the age of AI](https://www.teqsa.gov.au/about-us/news-and-events/latest-news/assessment-reform-age-artificial-intelligence).
- JCQ: [AI use in assessments](https://www.jcq.org.uk/knowledge-hub/ai-use-in-assessments-your-role-in-protecting-the-integrity-of-qualifications-2/).

### Agent, security, evaluation, and operations foundations

- Anthropic: [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents), [context engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents), [agent evaluations](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents), [prompt-injection defenses](https://www.anthropic.com/research/prompt-injection-defenses).
- NIST: [AI Risk Management Framework](https://www.nist.gov/itl/ai-risk-management-framework), [Generative AI Profile](https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.600-1.pdf). AI RMF 1.0 is under revision; pin the version actually used.
- OWASP: [Top 10 for LLM applications](https://owasp.org/www-project-top-10-for-large-language-model-applications/).
- OpenTelemetry: [GenAI spans](https://github.com/open-telemetry/semantic-conventions-genai/blob/main/docs/gen-ai/gen-ai-spans.md), [agent spans](https://github.com/open-telemetry/semantic-conventions-genai/blob/main/docs/gen-ai/gen-ai-agent-spans.md). Both require maturity checks.
- AWS Builders’ Library: [Making retries safe with idempotent APIs](https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/).
- Google SRE: [Canarying releases](https://sre.google/workbook/canarying-releases/), [SLO foundations](https://sre.google/workbook/part-I-foundations/).

## Provider qualification worksheet

Record one row per provider, tenant, API version, and granted permission set.

| Field | Required evidence |
|---|---|
| Product and tenant | Provider, region, institution tenant, sandbox/production distinction |
| Version profile | API/spec version, release date, deprecated surfaces, certification claim and its limits |
| Authentication | Flow, issuer/audience, rotation, delegated/application permissions, admin consent |
| Identity mapping | Stable IDs, role mapping, guardian proxy rules, merges, deletions, recycled identifiers |
| Scope | Required reads/writes, least-privilege justification, forbidden operations |
| Pagination and delta | Cursor semantics, ordering, expiry, HTTP 410/full-resync behavior |
| Event delivery | Signature, retries, duplication, reordering, gaps, payload completeness, reconciliation |
| Rate and availability | Published limits, observed limits, 429 guidance, outage semantics, residency |
| Effect semantics | Idempotency, conditional writes, read-after-write, cancellation, ambiguous timeout |
| Data lifecycle | Retention, export, correction, deletion, backup expiry, subprocessors |
| Accessibility | Accessibility metadata preservation, keyboard/AT behavior, VPAT/ACR evidence where applicable |
| Test record | Contract tests, fault injection, last qualification date, owner, expiry |

## Open uncertainties and watch list

- Jurisdiction-specific age, consent, education-record, child-safety, and mandated-reporting rules require local counsel and safeguarding owners.
- The 2026 US Title II timing is based on an interim rule and should be rechecked before relying on a deadline.
- KCSIE changes edition one day after this packet’s research date.
- The 1EdTech Trusted Portable Learning Context work is an RFC; its final shape may change.
- OpenTelemetry GenAI semantic conventions remain under development.
- Provider endpoints, quotas, scopes, certification status, and cloud availability change independently.
- The evidence base for generative tutoring remains uneven by age, subject, duration, language, disability, and outcome measure.
- Local assessment rules can be stricter than national guidance and can change per qualification or assignment.

## Refresh triggers

Refresh this packet when any of the following occurs:

- a new legal or regulator effective date;
- a safeguarding-policy edition change;
- a new CASE, OneRoster, LTI, QTI, Ed-Fi, WCAG, or provider major version;
- a change to an LMS/SIS tenant, scope grant, model, prompt, retrieval index, or assessment adapter;
- a material incident, cross-tenant near miss, answer-leakage event, evidence correction, or accessibility complaint;
- a new high-quality study changes the evidence on learning transfer or dependency;
- a new age group, subject, language, jurisdiction, or high-stakes setting enters scope.

## Handoff checklist for guide authors

- [ ] Define the deterministic baseline and smallest bounded loop.
- [ ] Draw control, data, and learning planes.
- [ ] Assign every field to a source and authority.
- [ ] Specify identity, consent, state, event, evidence, effect, and continuity contracts.
- [ ] Preserve exactly seven memory levels and learner-model correction/deletion.
- [ ] Make progressive hinting and unassisted transfer measurable.
- [ ] Encode assessment boundaries before model invocation.
- [ ] Keep grades, admissions, discipline, diagnosis, placement, proctoring, and high-stakes answers outside agent authority.
- [ ] Design teacher, guardian, accessibility, language, and safeguarding handoffs.
- [ ] Qualify provider behavior under retries, delay, duplication, reordering, and ambiguous completion.
- [ ] Separate traces from educational audits and redact learner content by default.
- [ ] Define offline, online, pedagogical, safety, failure-injection, fairness, and accessibility evaluations.
- [ ] Add queues, backpressure, tenant cells, cost controls, HA/DR, recovery, and offline limits.
- [ ] Pin immutable behavior bundles and define shadow, canary, rollback, drift, and feedback controls.
- [ ] Recheck all dated legal, safeguarding, standards, and provider claims before release.
