# Research Packet: Back-Office Workflow Agent Blueprint

> **Status:** Pass 2 research-backed synthesis  
> **Research date:** 2026-08-31  
> **Scope:** Production back-office agents for long-running cases, business rules, approvals, multi-system writes, reconciliation, exceptions, compliance, privacy, audit evidence, evaluation, deployment, and incidents  
> **Derived guides:** [Production back-office workflow agent blueprint](../../agents/back-office-workflow-agent/README.md)  
> **Method:** Current formal standards, public-sector control guidance, protocol specifications, official product/runtime documentation, official security projects, and original engineering references were compared. Stable engineering conclusions are separated from product mechanisms and jurisdiction-specific legal conclusions.

This packet preserves why the blueprint places a deterministic workflow and effect control plane around bounded model judgment. It is not a regulatory interpretation, procurement endorsement, or claim that every workload needs AI.

## Research questions

1. When is a model or agent justified instead of ordinary rules, workflow, case-management, integration, or RPA software?
2. Which back-office responsibilities must remain deterministic or human-owned?
3. How should structured processes, adaptive cases, and business decisions be represented?
4. What authoritative state, event, fact, decision, approval, and effect contracts survive long waits and upgrades?
5. How can model judgment be bounded, cited, abstaining, and independently validated?
6. How should approval and segregation of duties work across humans, service identities, and delegated automation?
7. How can retries, crashes, partial commits, cancellation, and lost responses avoid duplicate or missing business effects?
8. How should expected intent, downstream receipts, and actual state be reconciled?
9. What data-minimization, automated-decision, retention, provider, and audit boundaries are needed?
10. Which offline, online, failure-injection, control, and business measures gate autonomy?
11. Which runtime and deployment shapes are appropriate without premature distribution or multi-agent complexity?
12. How should model/provider failure, control defects, discrepancies, and privacy incidents be contained and recovered?

## Baseline as researched

| Domain | Baseline used | Status/date note | Blueprint consequence |
| --- | --- | --- | --- |
| Process modeling | OMG BPMN 2.0.2 | Formal, January 2014 | Useful for predefined process flow and events; not a complete deployment/security contract |
| Case modeling | OMG CMMN 1.1 | Formal, December 2016 | Useful for evolving cases and discretionary tasks; human authority still needs explicit controls |
| Decision modeling | OMG DMN 1.5 | Formal, August 2024; DMN 1.7 listed as beta | Use the latest formal baseline for portable rule concepts; pin actual engine conformance |
| Events | CloudEvents 1.0.2 | Latest released core text; `main` described 1.0.3 work in progress | `source + id` supports event duplicate identity; application owns ordering/version semantics |
| Internal control | GAO 2025 Green Book | Effective beginning fiscal year 2026 | Authorization, transaction recording, documentation, control review, and SoD shape business controls |
| Security/privacy controls | NIST SP 800-53 Rev. 5, Release 5.2.0 | Minor release issued August 2025; current at research date | AC-5, AC-6, AU, supply-chain, access, audit, and system-protection controls inform enforceable boundaries |
| AI risk management | NIST AI RMF 1.0 and AI 600-1 | AI RMF revision work was underway; GenAI Profile final July 2024 | Document scope/limits, test under deployment conditions, monitor, allow override/appeal, manage incidents |
| Agent security | OWASP Top 10 for Agentic Applications 2026 | Published December 2025 | Least agency, tool/identity abuse, goal hijack, memory/context, cascading, and human-trust risks are current concerns |
| Privacy law example | EU GDPR consolidated text | Article 5 principles and Article 22 safeguards | Data/purpose minimization and automated-decision applicability require jurisdiction-specific legal review |
| High-risk AI example | EU AI Act 2024/1689 | In force with phased applicability | Human oversight, logging, accuracy/robustness duties may apply depending on scope and role |
| Authorization protocols | OAuth Security BCP RFC 9700; Token Exchange RFC 8693 | Standards/BCP | Preserve actor/delegation, narrow audience/scope/lifetime, avoid ambient credentials |
| Provenance | W3C PROV family | W3C Recommendation/Notes, 2013 | Entity/activity/agent lineage is useful, but does not alone provide integrity or completeness |
| Trace propagation | W3C Trace Context | Recommendation, 2021 | Correlate diagnostics without putting PII/sensitive data in trace headers |
| Telemetry | OpenTelemetry GenAI semantic conventions | Development at research date and moved to a dedicated repository | Pin through an application vocabulary; raw content capture is opt-in and sensitive |
| Incident response | NIST SP 800-61 Rev. 3 | Final, April 2025 | Integrate AI/workflow incidents into preparation, detection, response, and recovery |

“Current” refers to the research date, not a promise that a vendor, law, or draft remains unchanged.

## Evidence tiers

| Tier | Evidence | Use in this packet |
| --- | --- | --- |
| **E1** | Formal standard, law text, final public-sector guidance, protocol BCP | Foundation for definitions and control objectives |
| **E2** | Official product/runtime documentation or official project repository | Concrete mechanisms and documented limits; version-sensitive |
| **E3** | Maintainer issue/release notes or original engineering reference | Failure detail and implementation nuance; cross-check before generalizing |
| **E4** | Emerging research or community discussion | Hypothesis or test inspiration only; not a control guarantee |

No source proves a whole architecture. The recommendations synthesize common invariants across sources.

## Finding 1: an agent is optional; durable process control is not

BPMN explicitly addresses business-process modeling, CMMN addresses less predefined case work driven by evolving information, and DMN addresses decisions. None requires a language model. A structured input mapped through complete rules should remain a normal application or rules service. A known sequence with events, timers, service tasks, and user tasks belongs in workflow software. An adaptive knowledge-worker case belongs in case management.

A model becomes useful when supported operations repeatedly require semantic interpretation of variable documents or free text, conflict synthesis, or a bounded recommendation that deterministic parsers/rules cannot provide economically. The model is therefore a worker inside a process, not the process owner.

**Stable conclusion**

Select the minimum capable layer. Require measured incremental value over the deterministic/manual baseline. “Agent” is not an architecture requirement.

**Strong sources**

- [OMG BPMN 2.0.2](https://www.omg.org/spec/BPMN/2.0.2/)
- [OMG CMMN 1.1](https://www.omg.org/spec/CMMN/1.1/)
- [OMG DMN 1.5](https://www.omg.org/spec/DMN/1.5/)
- [Repository: custom loop vs framework vs workflow engine](../../comparisons/custom-loop-vs-framework-vs-workflow-engine.md)

## Finding 2: BPMN, CMMN, and DMN are complementary but not execution guarantees

BPMN 2.0.2 formalizes process concepts and interchange but notes that operational monitoring and deployment are out of its scope. CMMN models case files and activities that may occur in unpredictable order. DMN specifies decision models, FEEL, and decision-table semantics including hit policies.

Vendor products implement subsets/extensions and add their own job, incident, migration, authorization, and operations behavior. A diagram can communicate a control flow while leaving effect idempotency, tenant access, audit retention, and active-instance compatibility undefined.

**Stable conclusion**

Use standards to make process/case/decision semantics reviewable. Test the chosen engine's executable behavior and keep application-owned state/effect/control contracts.

**Strong sources**

- [BPMN 2.0.2 specification PDF](https://www.omg.org/spec/BPMN/2.0.2/PDF)
- [CMMN overview](https://www.omg.org/cmmn/index.htm)
- [DMN 1.5 specification PDF](https://www.omg.org/spec/DMN/1.5/PDF)
- [Camunda business rule tasks](https://docs.camunda.io/docs/components/modeler/bpmn/business-rule-tasks/)

## Finding 3: the case record must be authoritative and model-independent

Long-running cases receive duplicate, delayed, and conflicting inputs; wait on people; cross deployments; and may be reassigned, cancelled, reopened, or corrected. CloudEvents gives a portable event envelope and states that `source + id` identifies distinct events/duplicates, but it does not define application ordering, aggregate versions, or legal transitions.

Durable runtimes journal or persist workflow progress, timers, and external signals. Those mechanics are useful, but the business case still needs explicit case, state version, owner/fence, facts, decisions, approvals, effects, deadlines, and terminal outcome. A model transcript or provider thread cannot safely own these semantics.

**Stable conclusion**

Maintain one application-owned case aggregate plus append-only semantic events. Distinguish case, run, attempt, proposal, approval, operation, event, and trace identity.

**Strong sources**

- [CloudEvents 1.0.2](https://github.com/cloudevents/spec/blob/ce%40v1.0.2/cloudevents/spec.md)
- [Temporal documentation](https://docs.temporal.io/)
- [AWS Step Functions callback pattern](https://docs.aws.amazon.com/step-functions/latest/dg/connect-to-resource.html)
- [Azure Durable Task external events](https://learn.microsoft.com/en-us/azure/durable-task/common/durable-task-external-events)
- [Repository: agent state and event contracts](../../runtime/agent-state-and-event-contracts.md)

## Finding 4: model output is a proposal with evidence, uncertainty, and abstention

NIST AI RMF calls for documenting task scope, limits, human oversight, test conditions, performance, monitoring, and risk response. OWASP agentic guidance highlights goal hijack, tool misuse, identity abuse, memory/context poisoning, cascading failure, and human-agent trust exploitation. Source documents and tickets are both evidence and untrusted input.

The safe interface is a closed, versioned schema with allowed labels, evidence locators, conflict and missing-data outcomes, task/prompt/model/tool versions, and an abstain/exception path. Confidence can help route review only after deployment-representative calibration. It cannot grant effect authority.

**Stable conclusion**

Validate the proposal independently, then let deterministic rules or an authorized human decide the transition. The model receives no write credentials.

**Strong sources**

- [NIST AI RMF Core](https://airc.nist.gov/airmf-resources/airmf/5-sec-core/)
- [NIST AI 600-1 Generative AI Profile](https://doi.org/10.6028/NIST.AI.600-1)
- [NIST AI 100-2 E2025 adversarial machine learning](https://csrc.nist.gov/pubs/ai/100/2/e2025/final)
- [OWASP Top 10 for Agentic Applications 2026](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/)
- [OWASP LLM06:2025 Excessive Agency](https://owasp.org/www-project-top-10-for-large-language-model-applications/2_0_vulns/LLM06_ExcessiveAgency.html)

## Finding 5: business invariants and rules remain deterministic

Amounts, thresholds, calendar computations, required evidence, eligibility, approval routes, and forbidden state combinations are testable logic. DMN offers an established representation for decision requirements, tables, hit policies, and FEEL; deterministic code is also valid.

Natural-language policy can be ambiguous or inconsistent. A model may help a policy owner analyze or draft a candidate rule, but production decisions must reference approved, versioned logic and typed inputs. No-match, null, conflict, and effective-date behavior must be explicit.

**Stable conclusion**

Models can explain rule results or provide candidate facts. They must not silently become the rule engine.

**Strong sources**

- [OMG DMN 1.5 formal baseline](https://www.omg.org/spec/DMN/1.5/About-DMN)
- [DMN 1.5 decision-table specification](https://www.omg.org/spec/DMN/1.5/PDF)

## Finding 6: approval and segregation of duties require system enforcement

The GAO Green Book describes authorization, complete/accurate/timely transaction recording, documentation, control review, and segregation of duties. NIST SP 800-53 AC-5 calls for identifying duties requiring separation and defining access authorization to support it. These controls apply across an entire transaction and its system identities, not just one UI.

Approval must bind the exact intent, target, current object versions, policy/risk version, evidence digest, approver eligibility, time, and expiry. The requester/preparer/model-assisted operator must not approve its own high-consequence result where SoD requires independence. Commit-time revalidation catches changed roles, target state, policy, or intent.

**Stable conclusion**

Treat approval as a versioned, expiring authorization input. Treat SoD as a policy over actor history and relationships across systems.

**Strong sources**

- [GAO 2025 Green Book](https://www.gao.gov/greenbook)
- [NIST SP 800-53 Rev. 5, Release 5.2.0](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)
- [NIST AI RMF human-oversight outcomes](https://airc.nist.gov/airmf-resources/airmf/5-sec-core/)
- [EU AI Act Article 14](https://eur-lex.europa.eu/eli/reg/2024/1689/oj)

## Finding 7: human review must be meaningful and operable

Formal approval can still fail through automation bias, missing evidence, unqualified reviewers, high queue load, or a UI that only permits acceptance. EU and UK regulatory guidance in applicable contexts emphasizes competent, authorized human oversight and ability to intervene or contest. NIST AI RMF calls for defined human-AI roles and oversight.

Human work needs eligibility, assignment, evidence, permitted decisions, deadlines, escalation, independence, and appeal semantics. A typed exception queue is a normal subsystem, not a dumping ground.

**Stable conclusion**

Design the review surface around source evidence, conflicts, exact consequence, and alternatives. Measure sampled decision quality and queue capacity, not click-through alone.

**Strong sources**

- [ICO automated decision-making guidance](https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/individual-rights/individual-rights/rights-related-to-automated-decision-making-including-profiling/)
- [EU AI Act human oversight](https://eur-lex.europa.eu/eli/reg/2024/1689/oj)
- [NIST AI RMF Core](https://airc.nist.gov/airmf-resources/airmf/5-sec-core/)

## Finding 8: external effects need stable semantic identity and an unknown state

AWS's idempotent API reference emphasizes caller-provided request identity, semantic equivalence, and late requests. Stripe documents a concrete contract with key scope, parameter comparison, stored results, and retention. Those semantics vary across providers.

A remote effect can commit while its response is lost. Retrying with a new key duplicates the effect; treating it as success can lose required verification. “Exactly once” is not a useful cross-system promise without naming the boundary.

**Stable conclusion**

Create a tenant-scoped `operationId` before the first attempt, bind it to canonical intent and target, reuse it across attempts, and represent `unknown` until status/receipt/read-back reconciliation resolves it.

**Strong sources**

- [AWS Builders' Library: Making retries safe with idempotent APIs](https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/)
- [Stripe idempotent requests](https://docs.stripe.com/api/idempotent_requests)
- [Repository: idempotency and side-effect safety](../../reliability/idempotency-and-side-effects.md)

## Finding 9: multi-system writes are an explicit consistency problem

A local database transaction cannot atomically commit arbitrary third-party SaaS, ERP, payment, email, or UI actions. The outbox pattern can atomically persist local state and publication intent, while consumers still deduplicate. Sagas coordinate local commits and compensation but introduce partial visibility, concurrency anomalies, and application-specific recovery. Compensation can fail and may not restore the original state.

For many back-office flows, a single authoritative system plus event-fed projections is safer than attempting synchronous multi-master writes. When multiple commits are necessary, forward recovery may be more correct than reversal after a business pivot.

**Stable conclusion**

Select single authority, local transaction, orchestrated saga, forward recovery, compensation, or manual reconciliation per business operation. Identify irreversible steps and persist each receipt.

**Strong sources**

- [Debezium outbox event router](https://debezium.io/documentation/reference/stable/transformations/outbox-event-router.html)
- [Azure saga pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/saga)
- [Azure compensating transaction pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/compensating-transaction)

## Finding 10: reconciliation is a product capability, not an afterthought

Internal-control guidance requires reliable reporting, transaction recording, documentation, and control activities. Distributed-effect evidence shows that local intent and external state can diverge even when retries are careful.

Reconciliation must compare intended operations, provider receipts/events, and authoritative snapshots. It must identify missing, unexpected, duplicate, divergent, pending, partial, and corrected effects by business/value dimensions. Reconciliation itself needs checkpoints, completeness evidence, ownership, deadlines, and independent control totals for high-risk domains.

**Stable conclusion**

Do not mark a consequential case complete until required postconditions and reconciliation pass or an authorized exception accepts the known state.

**Strong sources**

- [GAO 2025 Green Book](https://www.gao.gov/greenbook)
- [Azure compensating transaction pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/compensating-transaction)
- [Amazon SQS at-least-once delivery](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/standard-queues-at-least-once-delivery.html)

## Finding 11: delegation, least privilege, and short-lived credentials preserve accountability

RFC 8693 distinguishes delegation, where the actor remains visible as acting for a subject, from impersonation. RFC 9700 updates OAuth security best practices. NIST AC-6 and OWASP excessive-agency guidance support minimum privilege/functionality/autonomy and complete downstream mediation.

A generic service identity across tenants and systems destroys useful actor lineage and increases blast radius. The model should never see bearer credentials. A trusted gateway can mint or obtain short-lived, target/audience/scope-bound credentials after current policy and approval checks.

**Stable conclusion**

Preserve human/business subject and automated service actor separately. Scope credentials per adapter, tenant/resource, effect class, and time; enforce authorization again downstream.

**Strong sources**

- [OAuth 2.0 Token Exchange, RFC 8693](https://www.rfc-editor.org/rfc/rfc8693.html)
- [OAuth 2.0 Security BCP, RFC 9700](https://www.rfc-editor.org/rfc/rfc9700.html)
- [NIST SP 800-53 Rev. 5, Release 5.2.0](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)
- [OWASP LLM06:2025 Excessive Agency](https://owasp.org/www-project-top-10-for-large-language-model-applications/2_0_vulns/LLM06_ExcessiveAgency.html)

## Finding 12: privacy requires field/purpose boundaries and provider-specific verification

GDPR Article 5 states purpose limitation, data minimization, accuracy, storage limitation, integrity/confidentiality, and accountability principles. Article 22 contains safeguards for specified solely automated significant decisions. NIST Privacy Framework treats privacy across a data-processing ecosystem and lifecycle.

Prompts, artifacts, model outputs, traces, evaluation sets, caches, replicas, and third-party provider systems can create new copies and purposes. Full-content telemetry for convenience can undermine minimization and retention. Provider data use/retention differs by product, account, region, and contract.

**Stable conclusion**

Create purpose-limited field projections, explicit provider boundaries, data-class retention/hold rules, correction/appeal paths, and deletion evidence across every copy. Obtain domain legal/privacy review.

**Strong sources**

- [GDPR consolidated text](https://eur-lex.europa.eu/eli/reg/2016/679/2016-05-04)
- [NIST Privacy Framework](https://www.nist.gov/privacy-framework)
- [ICO automated decision-making guidance](https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/individual-rights/individual-rights/rights-related-to-automated-decision-making-including-profiling/)

## Finding 13: audit evidence and telemetry serve different guarantees

NIST AU-3 requires audit records to establish event type, time, location, source, outcome, and associated identities/entities. NIST AU-9 and log-management guidance address protection and management. W3C PROV models entity/activity/agent provenance. W3C Trace Context is designed for correlation and prohibits PII/sensitive values in trace headers.

Traces can be sampled, redacted, dropped, or reconfigured. An audit ledger must be unsampled for required control records and preserve rule/policy/model/prompt/evidence/approval/effect/reconciliation versions and relationships. A hash detects change only relative to a trusted known digest; it does not prove completeness or truth.

**Stable conclusion**

Use a protected audit evidence manifest over authoritative records and a separate redacted diagnostic telemetry plane. Neither private chain-of-thought nor full prompt capture is required for defensible decision evidence.

**Strong sources**

- [NIST SP 800-53 Rev. 5, Release 5.2.0](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)
- [NIST SP 800-92](https://csrc.nist.gov/pubs/sp/800/92/final)
- [W3C PROV overview](https://www.w3.org/TR/prov-overview/)
- [W3C Trace Context](https://www.w3.org/TR/trace-context/)
- [RFC 3161 time-stamp protocol](https://www.rfc-editor.org/rfc/rfc3161.html)

## Finding 14: evaluation must combine judgment, workflow, controls, effects, and business outcomes

NIST AI RMF calls for documented TEVV, representative deployment conditions, safety/security/reliability evaluation, post-deployment monitoring, third-party risk management, feedback, appeal/override, incident response, recovery, and change management. A classification accuracy number does not cover workflow state, authorization, effect duplication, or reconciliation.

Model evaluation needs per-field/class metrics, evidence support, conflict detection, abstention/selective risk, calibration, supported-population slices, adversarial content, and human utility. Deterministic tests cover state, timers, SoD, approval, authorization, and version migration. Effect fault tests cover commit ambiguity, partial success, cancellation, and repair. Business evaluation covers correct terminal state, timeliness, harm, appeal/reversal, and total cost.

**Stable conclusion**

Use separate hard gates for each plane. Never average a control bypass away with cost or accuracy improvements.

**Strong sources**

- [NIST AI RMF Core](https://airc.nist.gov/airmf-resources/airmf/5-sec-core/)
- [NIST AI 600-1 Generative AI Profile](https://doi.org/10.6028/NIST.AI.600-1)
- [NIST: Challenges to monitoring deployed AI systems](https://www.nist.gov/news-events/news/2026/03/new-report-challenges-monitoring-deployed-ai-systems)
- [Repository: trajectory and reliability evaluation](../../evaluation/trajectory-and-reliability-evaluation.md)

## Finding 15: operations must measure age, value, unknown state, and manual capacity

Google SRE guidance defines SLIs/SLOs around user-valued behavior and error-budget alerting. Back-office users care about complete/correct processing before deadlines, not only endpoint uptime. Queue backlog may contain different business values and deadlines; a count alone hides risk. Human review is a finite capacity pool.

OpenTelemetry's GenAI conventions remained in Development and warn that inputs/outputs may be large or sensitive. They offer useful vocabulary but should sit behind an application stability/redaction layer.

**Stable conclusion**

Define case durability, deadline, decision, control, effect verification, reconciliation, human-queue, and fallback SLOs. Alert on risk-weighted age/value and actionable error-budget burn.

**Strong sources**

- [Google SRE: Service level objectives](https://sre.google/sre-book/service-level-objectives/)
- [Google SRE Workbook: Alerting on SLOs](https://sre.google/workbook/alerting-on-slos/)
- [OpenTelemetry GenAI agent spans](https://github.com/open-telemetry/semantic-conventions-genai/blob/main/docs/gen-ai/gen-ai-agent-spans.md)
- [OpenTelemetry Collector security practices](https://opentelemetry.io/docs/security/config-best-practices/)

## Finding 16: deployment must version the whole decision system and preserve active cases

Long-running workflows may remain active while application code, workflow definitions, rules, policy, prompt/model routes, tool schemas, adapters, and data obligations change. Durable runtimes document replay/versioning constraints because historical execution must remain compatible.

Re-evaluating old model proposals during replay changes business history. Upgrading a tool schema can alter a retried step. A new policy may apply immediately, at the next decision, or only to new cases depending on approved semantics.

**Stable conclusion**

Ship an immutable behavior bundle spanning all decision dependencies. Pin, effective-date, or explicitly migrate active cases; never let a default upgrade policy emerge accidentally.

**Strong sources**

- [Temporal worker versioning](https://docs.temporal.io/production-deployment/worker-deployments/worker-versioning)
- [Azure Durable Task documentation](https://learn.microsoft.com/en-us/azure/durable-task/)
- [NIST AI RMF change-management outcomes](https://airc.nist.gov/airmf-resources/airmf/5-sec-core/)

## Finding 17: incidents require out-of-band containment and business correction

NIST SP 800-61 Rev. 3 integrates incident response with wider cybersecurity risk management and recovery. NIST AI RMF includes post-deployment monitoring, override, decommissioning, incident response, recovery, and communication. An agent-specific incident can be a wrong decision, unauthorized or duplicate effect, privacy disclosure, prompt injection, rule defect, stuck case, reconciliation gap, or audit-evidence failure.

Stopping a model does not resolve committed downstream changes. Recovery needs an impact query across versions and cases, authoritative reconciliation, correction/compensation workflows, stakeholder/affected-person communication where required, regression tests, and independent re-enable review.

**Stable conclusion**

Kill switches, credential revocation, version blocks, and effect/reconciliation controls must operate outside the model loop. Use the organization's incident process and business-control owners.

**Strong sources**

- [NIST SP 800-61 Rev. 3](https://csrc.nist.gov/pubs/sp/800/61/r3/final)
- [NIST AI RMF Core](https://airc.nist.gov/airmf-resources/airmf/5-sec-core/)
- [NIST SP 800-160 Vol. 2 Rev. 1](https://csrc.nist.gov/pubs/sp/800/160/v2/r1/final)

## Finding 18: identity and memory lifetime must be explicit before resume is safe

CloudEvents' event identity, Temporal's Workflow/Run identities, Camunda's definition/instance/job/task keys, SaaS record IDs, queue messages, and model/provider run IDs refer to different scopes. Temporal specifically documents that a Run ID can change across retries and other chain operations; Camunda documents process-definition and instance metadata and product-specific migration behavior. None defines the application's case, approval, represented principal, external-record version, or semantic effect intent.

Likewise, “memory” hides different retention and authority regimes. One-inference scratch state, run progress, reviewer session state, durable workflow records, governed domain knowledge, preferences, and curated outcomes cannot share promotion/deletion rules. OWASP's current agentic guidance treats memory/context poisoning as a security concern, reinforcing the need for provenance, writer controls, and governed promotion.

Compaction is lossy by design. A useful receipt must therefore identify the source-event high watermark and prefix digest, version pins, active approvals/clocks, pending and unknown effects, invariant hash, explicit omissions/retrieval references, and the next safe action. Resume must re-read authoritative stores, revalidate approvals/clocks/policy, reconcile uncertain effects, verify hashes/pins, and compile a fresh projection; a summary never becomes a substitute state snapshot.

**Stable conclusion**

Use separate semantic identities with exact version/reuse rules, and one canonical seven-lifetime memory policy. Treat a compaction receipt as a continuity proof and retrieval map, not authority.

**Strong sources**

- [Temporal Workflow ID and Run ID](https://docs.temporal.io/workflow-execution/workflowid-runid)
- [Camunda 8.9 messages and process instance identity](https://docs.camunda.io/docs/components/concepts/messages/)
- [CloudEvents 1.0.2](https://github.com/cloudevents/spec/blob/ce%40v1.0.2/cloudevents/spec.md)
- [OWASP Top 10 for Agentic Applications 2026](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/)

## Finding 19: platform and connector qualification is operation-specific

Current product documentation demonstrates incompatible contracts. Temporal has durable workflow and Worker Deployment Version mechanisms but application activities still own idempotent effects. Camunda 8.9 supplies jobs, user tasks, messages, incidents, migration, and backups with version/element/storage-specific limits. ServiceNow's REST/Table surface is versioned and ACL-bound but its general CRUD description does not define a universal idempotency key. Salesforce Composite can roll back subrequests in one Salesforce transaction, not a remote ERP. A documented SAP S/4HANA Sales Order OData V4 API requires ETags for updates, which cannot be generalized to every SAP operation.

Document services also differ: Textract documents a seven-day start-token/result window for the referenced async APIs; Azure Document Intelligence's current GA v4.0 API is versioned and asynchronous. DocuSign Connect may skip intermediate envelope notifications. SQS Standard is at least once; FIFO producer deduplication is time-bounded. Twilio delivery callbacks can arrive out of order and initial acceptance is not delivery.

**Stable conclusion**

Maintain an enforced manifest per semantic read/propose/stage/commit/reconcile/compensate operation. Qualify target identity, version precondition, idempotency scope/retention, definitive no-commit versus unknown, receipt/status/read-back, callback security/order, retries, cancellation, compensation, credentials, data, limits, and evidence expiry under the exact tenant configuration.

**Strong sources**

- [Temporal Worker Versioning](https://docs.temporal.io/production-deployment/worker-deployments/worker-versioning)
- [Camunda 8.9 job workers](https://docs.camunda.io/docs/components/concepts/job-workers/)
- [ServiceNow Zurich Table API](https://www.servicenow.com/docs/r/zurich/api-reference/rest-apis/c_TableAPI.html)
- [Salesforce REST Composite](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/resources_composite_composite_post.htm)
- [SAP S/4HANA Cloud Sales Order OData V4 operations](https://help.sap.com/docs/SAP_S4HANA_CLOUD/03c04db2a7434731b7fe21dca77440da/b7db7f7b302643d0a4e56fdfbfa6e5db.html)
- [Amazon Textract asynchronous operations](https://docs.aws.amazon.com/textract/latest/dg/api-async.html)
- [Azure AI Document Intelligence overview/version support](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/overview?view=doc-intel-4.0.0)
- [DocuSign Connect guidance](https://www.docusign.com/blog/developers/dsdev-adding-webhooks-application)
- [Amazon SQS visibility timeout and delivery behavior](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-visibility-timeout.html)
- [Twilio outbound status callbacks](https://www.twilio.com/docs/messaging/guides/outbound-message-status-in-status-callbacks)

## Finding 20: evaluation must test trajectories, invariants, people, and recovery load

Business outcome quality is delayed and selection-biased; model accuracy alone cannot expose illegal paths, control bypass, duplicate effects, reviewer automation bias, or an unstaffable incident. NIST AI RMF separates governance, mapping, measurement, and management outcomes and expects deployment-representative TEVV and monitoring. SRE guidance grounds SLOs in user-valued outcomes and error budgets. OpenTelemetry's GenAI/agent conventions remain developmental and content fields are opt-in, so telemetry needs an application-owned stable/redacted vocabulary.

**Stable conclusion**

Gate releases separately on judgment, allowed semantic trajectory, always-on invariants, effect/recovery behavior, terminal outcomes, and human factors. Fault-test combined failures and measure impact discovery, reconciliation throughput, human correction hours, restore catch-up, communication/appeal load, and recovery toil.

**Strong sources**

- [NIST AI RMF Core](https://airc.nist.gov/airmf-resources/airmf/5-sec-core/)
- [Google SRE service-level objectives](https://sre.google/sre-book/service-level-objectives/)
- [OpenTelemetry GenAI agent conventions](https://github.com/open-telemetry/semantic-conventions-genai/blob/main/docs/gen-ai/gen-ai-agent-spans.md)

## Finding 21: controlled evolution releases a whole behavior bundle

Long-running cases and external effects make “model rollback” an incomplete concept. Workflow/schema/rules/policy/context/memory/compaction/prompt/model/tool/capability/reconciliation changes jointly determine behavior. NIST SSDF and SP 800-53 supply software-development and supply-chain control objectives; runtime versioning documentation shows why active executions need explicit pin/patch/migrate behavior.

Production failures are valuable but unsafe training material. Corrections, appeals, incidents, unknown effects, and near misses are selectively observed, may contain sensitive data, and can carry poisoned instructions or wrong reviewer labels. They need quarantine, purpose authorization, deduplication, adjudication, lineage, leakage checks, held-out evaluation, accountable change approval, canary, and full-bundle rollback. They must not self-modify production.

**Stable conclusion**

Promote through measurable Stage 0–6 evidence and a signed immutable behavior bundle. Controlled failure mining proposes reviewed changes; it never bypasses release, privacy, SoD, or active-case compatibility gates.

**Strong sources**

- [NIST SP 800-218 Secure Software Development Framework](https://csrc.nist.gov/pubs/sp/800/218/final)
- [NIST SP 800-53 Rev. 5, Release 5.2.0](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)
- [Temporal Worker Versioning](https://docs.temporal.io/production-deployment/worker-deployments/worker-versioning)
- [Camunda 8.9 process migration](https://docs.camunda.io/docs/components/concepts/process-instance-migration/)

## Architecture alternatives and conditional choices

| Choice | Prefer when | Avoid/limit when |
| --- | --- | --- |
| Deterministic application | Rules complete and inputs structured | Semantic ambiguity is the measured bottleneck |
| Rules/DMN service | Decisions can be specified and tested | Policies are unresolved prose or need human interpretation |
| BPM/workflow engine | Flow, timers, tasks, and operations visibility dominate | Team only needs a small database state machine |
| Case-management platform | Knowledge workers choose evolving activities | Process is actually stable and high-volume |
| RPA | No supported API and bounded UI is unavoidable | Consequential writes lack confirmation/reconciliation |
| Model-assisted workflow | Unstructured evidence must become typed facts/proposals | Model is being added only for marketing or “autonomy” |
| Agent graph inside a step | Bounded iterative semantic work needs tool calls | Used as the sole case/audit/effect runtime |
| Custom DB state machine | Few simple case types and strong engineering ownership | Long complex waits/migrations/ops UI exceed team capacity |
| Single authoritative write | One system can own business truth | Legitimate independent authorities must commit |
| Orchestrated saga | Multi-system local commits and recovery are unavoidable | Strong atomicity is required or compensation is undefined |
| Manual reconciliation | High-risk legacy ambiguity cannot be automated safely | Volume makes backlog itself unsafe without process redesign |

## Material tensions and resolved positions

### “Human in the loop” versus risk-tiered autonomy

Requiring approval for every trivial step creates fatigue and backlog; removing it from high-consequence decisions creates uncontrolled risk. Resolve by granting authority per action/risk cell, with meaningful review for applicable effects and pre-authorized bounded automation only after evidence.

### Current policy versus policy at proposal/approval time

Audit needs historical policy evidence; security/control needs current authorization. Persist proposal/approval policy versions, then re-evaluate current commit policy and explicitly resolve conflicts.

### Pinned rules versus current effective rules

Reproducibility favors pinning; mandatory changes may require current rules. Choose per rule class and record every version. Migrate active cases explicitly when required.

### Orchestration versus choreography

Choreography can fit simple decoupled flows, but complex cases, human waits, compensation, and audit benefit from an explicit coordinator. Use events for integration while keeping one authoritative case owner.

### Compensation versus forward recovery

“Rollback” sounds safe but can be lossy or wrong after an irreversible pivot. Domain owners predefine whether to finish, compensate, pause, or accept partial state.

### Full trace capture versus audit/privacy

Full content improves debugging but expands sensitive copies and still does not guarantee audit completeness. Keep minimal authoritative evidence and opt-in protected debug capture under an approved purpose.

### Model confidence versus human workload

High thresholds reduce auto-volume but may overload review; low thresholds increase harmful errors. Optimize selective risk under explicit harm and queue-capacity constraints, never confidence alone.

### Multi-agent organization mirroring versus typed workflow participants

Department-role agents appear intuitive but add handoff, state, identity, and cascading-failure complexity. Use one coordinator, deterministic services, bounded model workers, and real human roles unless independent agents provide measured isolation/capability value.

## Claims deliberately excluded

- That an LLM can reliably interpret all business policy from natural language.
- That a high confidence score proves a correct or safe action.
- That a human click automatically constitutes meaningful oversight.
- That a workflow engine, agent SDK, or durable runtime provides exactly-once external effects.
- That event sourcing is mandatory for every case system.
- That a hash chain alone creates immutable, complete, or legally admissible audit evidence.
- That a compensation restores the exact prior world state.
- That deleting sensitive fields from prompts eliminates inference or fairness risk.
- That one regulatory framework applies to every back-office workload or jurisdiction.
- That vendor enterprise terms, retention, region, or model behavior are uniform across products/accounts.
- That multi-agent architecture improves a back-office process by default.
- That the highest autonomy level is an appropriate maturity destination.

## Derived guide set

| Guide | Specialized content |
| --- | --- |
| [Blueprint README](../../agents/back-office-workflow-agent/README.md) | Production position, invariants, authority, lifecycle, promotion gates, anti-patterns |
| [Workload fit, requirements, and autonomy](../../agents/back-office-workflow-agent/01-workload-fit-requirements-and-autonomy.md) | Alternatives, decomposition, requirements, risk and admission contract |
| [Reference architecture and runtime selection](../../agents/back-office-workflow-agent/02-reference-architecture-and-runtime-selection.md) | Trust/component boundaries, runtime/BPMN/CMMN/DMN choice, identity, topology |
| [Case state, events, rules, and model judgment](../../agents/back-office-workflow-agent/03-case-state-events-rules-and-model-judgment.md) | Contracts, versioning, missing/conflict/novelty, concurrency, upgrade behavior |
| [Approvals, segregation of duties, and exceptions](../../agents/back-office-workflow-agent/04-approvals-segregation-of-duties-and-exceptions.md) | Exact approval, SoD, review UI, exception queues, break glass |
| [Tools, effects, idempotency, and reconciliation](../../agents/back-office-workflow-agent/05-tools-effects-idempotency-and-reconciliation.md) | Semantic commands, ledger/outbox, unknown/partial outcomes, sagas, reconciliation |
| [Data, privacy, compliance, and audit evidence](../../agents/back-office-workflow-agent/06-data-privacy-compliance-and-audit-evidence.md) | Purpose projection, provider boundary, automated decisions, evidence manifest, retention |
| [Observability, evaluation, and failure injection](../../agents/back-office-workflow-agent/07-observability-evaluation-and-failure-injection.md) | Five evaluation planes, SLOs, corpus, hard gates, fault matrix |
| [Deployment, operations, incidents, and roadmap](../../agents/back-office-workflow-agent/08-deployment-operations-incidents-and-roadmap.md) | Behavior bundle, active cases, queues, kill switches, incidents, DR, and staged authority |

## Refresh triggers

Re-research and revise when:

- OMG publishes a new formal BPMN, CMMN, or DMN version;
- the chosen workflow/case engine changes replay, migration, human-task, or incident semantics;
- CloudEvents releases after 1.0.2 or application event compatibility changes;
- NIST publishes a revised AI RMF, Privacy Framework, SP 800-53, or agent-security guidance;
- OWASP Agentic Top 10, AISVS, or related official agent guidance materially changes;
- applicable automated-decision, AI, privacy, employment, finance, health, records, or audit law/guidance changes;
- a model/provider changes data use, retention, residency, subprocessors, model aliases, or support terms;
- a downstream API changes idempotency, receipt, status, concurrency, or retention behavior;
- OpenTelemetry GenAI conventions stabilize or change repositories/schema materially;
- production corrections, appeals, injection incidents, discrepancies, or reviewer behavior reveal a new failure class;
- an authority cell, effect class, tenant, data category, or workflow population expands.

## Research limitations

- The packet is domain-neutral. It does not encode sector-specific accounting, lending, insurance, healthcare, employment, public-benefit, tax, records, or evidentiary law.
- Product feature tiers and defaults were not exhaustively compared; implementation must verify selected versions and contracts.
- BPMN/CMMN/DMN conformance does not imply interoperability of every executable extension.
- Public documentation cannot prove a provider's internal controls or the organization's configured behavior.
- Emerging agent-security taxonomies provide useful threat labels but are not a substitute for workload threat modeling and testing.
- Model quality thresholds cannot be universal; each case/effect class needs representative data and harm-based criteria.

## Pass 2 source annotations

All web sources in this table were accessed **2026-08-31**. “Current” means current on that date. Product behavior remains subject to tenant configuration, plan, region, extensions, and later releases; the qualification packet must pin the deployed reality.

| Source/baseline | Material evidence used | Version and interpretation limit |
| --- | --- | --- |
| [OMG BPMN 2.0.2](https://www.omg.org/spec/BPMN/2.0.2/), [CMMN 1.1](https://www.omg.org/spec/CMMN/1.1/), [DMN 1.5](https://www.omg.org/spec/DMN/1.5/) | Separates predefined process flow, adaptive case work, and deterministic decision models; supplies event/task/decision vocabulary | Formal standards do not define a product's durability, security, deployment, API, connector, or audit implementation; executable subsets/extensions vary |
| [CloudEvents 1.0.2](https://github.com/cloudevents/spec/tree/ce%40v1.0.2) | `source + id` event identity and portable envelope context | Core release baseline; does not supply aggregate order, case version, actor/evidence, delivery, or business transition semantics |
| [NIST SP 800-53 Rev. 5, Release 5.2.0](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final), [SP 800-53A Rev. 5](https://csrc.nist.gov/pubs/sp/800/53/a/r5/final), [SP 800-218](https://csrc.nist.gov/pubs/sp/800/218/final) | Least privilege, SoD, audit protection, assessment, acquisition/supply-chain, and secure-development control objectives | Control catalogs and practices require organizational tailoring and assessment; they do not certify this architecture, a provider, or legal compliance |
| [NIST AI RMF Core](https://airc.nist.gov/airmf-resources/airmf/5-sec-core/), [NIST AI 600-1](https://doi.org/10.6028/NIST.AI.600-1) | Scope/limits, roles, TEVV, monitoring, human oversight, incident/recovery, and change-management outcomes | Voluntary risk-management guidance, not universal thresholds or sector law; AI RMF revision activity is a refresh trigger |
| [OWASP Agentic Top 10 2026](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/), [Prompt Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/LLM_Prompt_Injection_Prevention_Cheat_Sheet.html) | Goal/tool/identity/memory risks and defense-in-depth for indirect injection in documents/tool results | Threat guidance, not proof that filtering or a guard model prevents injection; application least privilege and independent policy/effect gates remain primary |
| [Temporal Workflow/Run IDs](https://docs.temporal.io/workflow-execution/workflowid-runid), [Worker Versioning](https://docs.temporal.io/production-deployment/worker-deployments/worker-versioning), [cancellation](https://docs.temporal.io/develop/go/workflows/cancellation) | Runtime identity scope, Run ID chain changes, pinned/auto-upgrade deployments, patching need, cancellation/heartbeat mechanics | Current docs span platform and SDK-specific pages; qualify selected SDK/server/cloud versions, retention, codecs, history, HA/DR, and activity effect semantics. Application case/effect identity remains external |
| [Camunda 8.9 jobs](https://docs.camunda.io/docs/components/concepts/job-workers/), [user tasks](https://docs.camunda.io/docs/components/modeler/bpmn/user-tasks/), [messages](https://docs.camunda.io/docs/components/concepts/messages/), [migration](https://docs.camunda.io/docs/components/concepts/process-instance-migration/), [backup/restore](https://docs.camunda.io/docs/self-managed/operational-guides/backup-restore/backup-and-restore/) | Process/job/task/message identities, waits/human work, retries/incidents, migration limitations, compensation, and component restore behavior | Baseline is Camunda 8.9/current docs; SaaS/self-managed and secondary-store paths differ. Public-API stability covers only the declared surface; model semantics do not provide adapter idempotency |
| [ServiceNow Zurich Table API](https://www.servicenow.com/docs/r/zurich/api-reference/rest-apis/c_TableAPI.html), [REST versioning/security](https://www.servicenow.com/docs/r/api-reference/rest-api-explorer/c_RESTAPI.html), [Scripted REST API](https://www.servicenow.com/docs/r/zurich/api-reference/rest-api-explorer/c_CustomWebServices.html) | Versioned resource paths, `sys_id`, CRUD, caller roles/ACLs, and explicit versioning for custom semantic APIs | Zurich/API reference baseline; instance release, domain separation, ACLs, business rules, plugins, and customizations control reality. The cited generic Table API does not document a universal idempotency-key guarantee |
| [Salesforce REST Composite v67.0](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/resources_composite_composite_post.htm), [integration patterns](https://developer.salesforce.com/docs/atlas.en-us.integration_patterns_and_practices.meta/integ_pat_tempate.htm) | Same-user composite subrequests, configurable rollback within Salesforce, external-ID/upsert patterns, API/security/limit considerations | v67.0 example/current developer page; triggers/Flows, field/object security, uniqueness, stale-write/read-back, and org limits require tenant tests. Composite atomicity does not cross systems |
| [SAP S/4HANA Cloud 2608 Sales Order OData V4](https://help.sap.com/docs/SAP_S4HANA_CLOUD/03c04db2a7434731b7fe21dca77440da/b7db7f7b302643d0a4e56fdfbfa6e5db.html) | Entity keys and required ETag/`If-Match` optimistic concurrency on documented updates | One API/release example only; never generalize to other SAP/ERP entities, actions, communication arrangements, side effects, or receipts |
| [Amazon Textract async operations](https://docs.aws.amazon.com/textract/latest/dg/api-async.html) | Async job/status pattern, seven-day `ClientRequestToken` behavior and default result availability, notifications, quota/permission cautions | Applies to the referenced asynchronous APIs; exact operation/region/account quotas, output storage, IAM, pagination, and later changes must be pinned |
| [Azure AI Document Intelligence overview](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/overview?view=doc-intel-4.0.0), [confidence guidance](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/concept/accuracy-confidence?view=doc-intel-4.0.0) | v4.0 (`2024-11-30`) GA status, asynchronous operation/result identity, model/field confidence behavior | Current GA baseline at access date; confidence availability varies by result type and is not business authorization. Region, retention, identity, quota, and model configuration remain qualification items |
| [DocuSign Connect guidance](https://www.docusign.com/blog/developers/dsdev-adding-webhooks-application), [Connect 2.0 structure](https://www.docusign.com/blog/developers/connect-20) | Envelope/account webhook configuration, event/status identity, retrieval pattern, skipped intermediate notifications | Official developer guidance but product plan/configuration and API schema must be checked; webhook receipt does not alone prove exact document/recipient intent or eliminate duplicate-create risk |
| [Amazon SQS visibility timeout](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-visibility-timeout.html), [outage recovery](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/designing-for-outage-recovery-scenarios.html) | At-least-once consumer behavior, visibility, DLQ, FIFO ordering scope, and five-minute producer-deduplication window | Queue delivery properties do not provide exactly-once business effects; consumer idempotency, redrive, retention, tenant policy, and recovery after dedupe expiry remain application responsibilities |
| [Twilio message resource](https://www.twilio.com/docs/messaging/api/message-resource), [outbound callbacks](https://www.twilio.com/docs/messaging/guides/outbound-message-status-in-status-callbacks) | Message SID/status lifecycle, callbacks, signature/evolving-payload guidance, and accepted/queued versus delivered distinction | Channel/carrier/region and compliance behavior vary; callbacks may be out of order, and delivery status does not prove the underlying business decision was authorized |
| [OpenTelemetry GenAI agent spans](https://github.com/open-telemetry/semantic-conventions-genai/blob/main/docs/gen-ai/gen-ai-agent-spans.md) | Candidate span vocabulary for agent/workflow/tool operations and opt-in content attributes | Dedicated repository and **Development** status at access date; pin an internal schema and never make telemetry an execution or business-audit dependency |

The annotations intentionally state what each source does **not** establish. No single source validates the full system or substitutes for executable qualification under the organization's case types, providers, controls, and laws.

## Selected primary and authoritative sources

### Process, case, decision, and event standards

- [OMG specifications catalog](https://www.omg.org/spec)
- [BPMN 2.0.2](https://www.omg.org/spec/BPMN/2.0.2/)
- [CMMN 1.1](https://www.omg.org/spec/CMMN/1.1/)
- [DMN 1.5](https://www.omg.org/spec/DMN/1.5/)
- [CloudEvents 1.0.2](https://github.com/cloudevents/spec/tree/ce%40v1.0.2)
- [W3C PROV overview](https://www.w3.org/TR/prov-overview/)

### Internal control, security, privacy, and AI risk

- [GAO 2025 Green Book](https://www.gao.gov/greenbook)
- [NIST SP 800-53 Rev. 5, Release 5.2.0](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)
- [NIST SP 800-53A Rev. 5](https://csrc.nist.gov/pubs/sp/800/53/a/r5/final)
- [NIST SP 800-92](https://csrc.nist.gov/pubs/sp/800/92/final)
- [NIST Privacy Framework](https://www.nist.gov/privacy-framework)
- [NIST AI RMF Core](https://airc.nist.gov/airmf-resources/airmf/5-sec-core/)
- [NIST AI 600-1](https://doi.org/10.6028/NIST.AI.600-1)
- [NIST AI 100-2 E2025](https://csrc.nist.gov/pubs/ai/100/2/e2025/final)
- [OWASP Top 10 for Agentic Applications 2026](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/)
- [GDPR consolidated text](https://eur-lex.europa.eu/eli/reg/2016/679/2016-05-04)
- [EU AI Act](https://eur-lex.europa.eu/eli/reg/2024/1689/oj)

### Identity, effects, durability, and operations

- [OAuth 2.0 Security BCP, RFC 9700](https://www.rfc-editor.org/rfc/rfc9700.html)
- [OAuth 2.0 Token Exchange, RFC 8693](https://www.rfc-editor.org/rfc/rfc8693.html)
- [RFC 3161 time-stamp protocol](https://www.rfc-editor.org/rfc/rfc3161.html)
- [AWS idempotent API guidance](https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/)
- [Stripe idempotent requests](https://docs.stripe.com/api/idempotent_requests)
- [Debezium outbox event router](https://debezium.io/documentation/reference/stable/transformations/outbox-event-router.html)
- [Azure saga pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/saga)
- [Azure compensating transaction pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/compensating-transaction)
- [Temporal documentation](https://docs.temporal.io/)
- [AWS Step Functions callback tasks](https://docs.aws.amazon.com/step-functions/latest/dg/connect-to-resource.html)
- [Azure Durable Task external events](https://learn.microsoft.com/en-us/azure/durable-task/common/durable-task-external-events)
- [W3C Trace Context](https://www.w3.org/TR/trace-context/)
- [OpenTelemetry GenAI semantic conventions](https://github.com/open-telemetry/semantic-conventions-genai)
- [Google SRE SLO guidance](https://sre.google/sre-book/service-level-objectives/)
- [NIST SP 800-61 Rev. 3](https://csrc.nist.gov/pubs/sp/800/61/r3/final)

## Pass 2 quality statement

This pass adds exact cross-system identity/version semantics, the canonical seven-lifetime memory policy, loss-aware compaction/resume, durable runtime and operation-level connector qualification, worked lifecycle/effect flows, supply-chain and tenant/secret controls, separated outcome/trajectory/invariant/human-factor evaluation, recovery-load engineering, a signed behavior bundle, and measurable Stage 0–6 exercises. Current official product examples are version-limited and annotated rather than treated as endorsements.

The remaining work is deployment-specific: specialize at least two real case domains with process/control/privacy/legal owners; execute qualification against the chosen Temporal/Camunda/ServiceNow/Salesforce/ERP/document/e-sign/queue/communication configurations; set evidence-based thresholds and RPO/RTO; and perform independent security, privacy, accessibility, audit, human-factors, and contradiction reviews. Public documentation cannot establish configured tenant behavior or sector-specific legal sufficiency.
