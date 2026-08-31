# Customer Support and Service Resolution Agent Blueprint Research Packet

**Status:** Research-backed production packet; Pass 2 complete  
**Research date:** 2026-08-31  
**Applies to:** [Customer support and service resolution blueprint](../../agents/customer-support-agent/README.md)  
**Registry category:** 29  
**Evidence posture:** Primary specifications, official documentation/repositories, and original engineering/research sources were preferred; vendor behavior is treated as provider-specific unless independently standardized.

## Executive finding

The best practical customer-support design is a deterministic case and effect workflow around one bounded model resolver. Customer support becomes an agent workload only where language ambiguity, adaptive evidence gathering, troubleshooting, and multi-turn continuity exceed a rules engine or fixed self-service flow. Identity, tenant and object binding, policy eligibility, monetary/time calculations, SLA clocks, approval, effect commit, provider reconciliation, case closure, and audit remain application-owned.

This conclusion is supported by four converging evidence groups:

1. support platforms expose versioned ticket/audit/SLA concepts and asynchronous or duplicate event behavior rather than a single conversational truth;
2. payment, subscription, commerce, and messaging providers have distinct idempotency, pending, asynchronous, cancellation, and delivery semantics;
3. identity and security guidance separates authentication, recovery, session assurance, authorization, data minimization, and incident controls from conversational confidence;
4. current agent guidance recommends bounded tools, explicit approval boundaries, environment-backed evaluation, and simple orchestration, while warning that context, tracing, guardrails, and hosted state do not replace application controls.

## Questions researched

- What makes customer support category 29 distinct from browser automation, IT service desk, generic back-office cases, enterprise knowledge, and sales/revenue operations?
- Which tasks justify model-directed planning, and which should remain deterministic?
- What should be authoritative for customer identity, case/conversation state, policy, provider state, delivery, effects, and closure?
- What event, effect, approval, memory, handoff, and evidence contracts are required?
- How do current support, messaging, payment, subscription, and commerce APIs actually behave under duplicates, lag, async completion, cancellation, and retries?
- How should helpdesk/CRM, contact-center, email/chat/voice, identity, knowledge/status, commerce/shipment, subscription/billing/refund capabilities be qualified and recertified independently?
- Which voice call, recording, transcription, association, consent, retention, delivery and deletion states must remain separate?
- How should prompt injection, social engineering, permissions, tenancy, privacy, secrets, and payment/authentication data be handled?
- What evaluation evidence is stronger than transcript quality or public benchmark score?
- How should the system deploy, scale, degrade, reconcile, trace, respond to incidents, and evolve without self-modifying from raw cases?

## Research method and saturation

Research began with the repository's category registry, expansion contract, cross-cutting control packet, and adjacent blueprints. External research then proceeded by evidence group: official OpenAI agent/runtime guidance; official support-platform APIs and operational documentation; official messaging, identity, payment, billing, and commerce documentation; web and observability standards; durability/retry engineering sources; public customer-service benchmark papers/repositories/issues; and production evaluation/context/release guidance.

Sources were cross-checked for the following failure-sensitive claims: ticket/audit authority, optimistic concurrency and propagation delay, SLA clock semantics, webhook delivery guarantees, message-delivery meaning, provider idempotency, asynchronous effect states, cancellation consequences, conversation storage and compaction, multi-agent trade-offs, trace privacy, and benchmark validity. Research stopped when additional sources repeated these controls without changing the design. Product-specific details still require connector certification against the exact account, API version, contract, and region used by a deployment.

## Boundary decision

| Category | Owns | Customer-support interaction |
|---|---|---|
| Customer support and service resolution | Authenticated customer case, policy-grounded answer/troubleshooting, customer effect request, channel continuity, SLA route, handoff, quality, verified outcome | This blueprint remains case owner |
| Browser automation | Generic browser/UI observation and manipulation | Receives one typed bounded legacy-UI operation; returns evidence/result; does not decide policy or close the case |
| IT service desk | Employee/workforce accounts, endpoints, fleet, corporate access, internal recovery | Receives employee or endpoint cases; consumer product troubleshooting remains support-owned |
| Back-office workflow | Generic organizational cases and multi-department processing | Receives typed fulfillment dependency; support retains customer communication/outcome tracking |
| Enterprise knowledge | Cross-corpus search, access-aware retrieval platform, knowledge operations | Supplies governed support knowledge; support owns case-specific use and claims |
| Sales and revenue operations | Leads, opportunities, quotes, pipeline, expansion, revenue communication | Receives acquisition/expansion work; support credits or retention actions remain policy-bound support effects, not CRM opportunity changes |
| Marketing operations | Campaign populations, acquisition journeys, bulk/promotional messaging, campaign spend and attribution | Receives opt-in marketing or campaign work; support never exports case/customer populations or turns a service conversation into promotional outreach |

The distinction is by authoritative object and outcome, not interface. Using a browser does not turn a support case into browser automation; opening an internal fulfillment task does not turn it into generic back-office ownership; a frustrated customer mentioning an upgrade does not authorize support to operate the sales pipeline.

## Evidence-to-decision register

### Decision CS-01 — qualify against a deterministic baseline

- **Class:** Product and architecture.
- **Decision:** Fixed lookup, form, macro, policy calculator, incident notice, and routing paths are the default. Add a model resolver only to slices requiring adaptive interpretation/evidence gathering and only when verified outcomes improve.
- **Evidence:** Anthropic's engineering guidance recommends starting with the simplest composable pattern and adding agentic complexity only when it improves outcomes; current OpenAI orchestration guidance similarly starts with a single agent and adds specialization only for demonstrated benefit. See [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) and [OpenAI agent orchestration](https://developers.openai.com/api/docs/guides/agents/orchestration).
- **Limit/contradiction:** These are vendor engineering recommendations, not controlled evidence that one architecture wins for every support queue. The organization's deterministic baseline and eval suite decide.
- **Consequence:** Stage 0 requires a measured non-agent path. More agent turns, roles, or automation are not maturity signals.
- **Refresh trigger:** Material new evidence on support-specific agent architecture or changed orchestration capabilities.

### Decision CS-02 — use one bounded resolver inside an application workflow

- **Class:** Architecture.
- **Decision:** One resolver chooses evidence queries, safe diagnostic steps, questions, and structured proposals. The deterministic workflow owns identity, state, deadlines, policy, approval, effects, reconciliation, delivery, and closure.
- **Evidence:** OpenAI's orchestration documentation describes manager/tool and handoff patterns but explicitly frames extra agents as a trade-off; its guardrail/approval guidance places approval before consequential tools and distinguishes tool-level enforcement. [OpenAI guardrails and approvals](https://developers.openai.com/api/docs/guides/agents/guardrails-approvals) documents mechanisms, not a complete business authorization system.
- **Limit/contradiction:** SDK approvals and guardrails can make a run resumable or validate inputs/outputs, but they do not establish customer identity, current policy, approver entitlement, or external outcome.
- **Consequence:** Agent SDK or durable workflow features are implementation choices behind application contracts. Multi-agent delegation is rejected by default.
- **Refresh trigger:** Evaluations show a bounded specialist improves capability or isolation enough to justify extra state, traces, and approval surfaces.

### Decision CS-03 — support platform plus workflow ledger, with explicit authority

- **Class:** State and integration.
- **Decision:** Keep the support platform authoritative for the business case where feasible; keep application workflow/effect state in a durable ledger; reconcile through versioned projections and optimistic concurrency.
- **Evidence:** Zendesk's [Tickets API](https://developer.zendesk.com/api-reference/ticketing/tickets/tickets/) describes ticket fields/statuses, audits, update visibility delays, and optimistic-locking behavior. Its [Ticket Audits API](https://developer.zendesk.com/api-reference/ticketing/tickets/ticket_audits/) exposes an immutable-style history of ticket updates and their events. Zendesk's [2025–2026 changelog](https://developer.zendesk.com/api-reference/changelog/changelog/) demonstrates that update/concurrency and object behavior evolve.
- **Limit/contradiction:** Zendesk is an example, not the prescribed support platform. “Solved” and “closed,” audit retention, write conflicts, and propagation behavior differ by provider and plan.
- **Consequence:** Connector certification documents state mappings, concurrency, propagation, audit, pagination, and change-management behavior. Last-write-wins is prohibited for concurrent human/model/effect updates.
- **Refresh trigger:** Support-platform API/version/plan change or a change in system-of-record ownership.

### Decision CS-04 — channel participation is not customer identity

- **Class:** Security and identity.
- **Decision:** Bind tenant, subject, customer, account, channel address, case, and provider object separately. Use the organization's identity service and risk-based step-up for protected disclosures/effects; recovery stays outside the model.
- **Evidence:** NIST SP 800-63B-4 separates authentication, session management, authenticator events, and recovery and discusses social-engineering risk in subscriber/customer support. See [SP 800-63B-4](https://pages.nist.gov/800-63-4/sp800-63b.html) and [authenticator lifecycle events](https://pages.nist.gov/800-63-4/sp800-63b/events/).
- **Limit/contradiction:** NIST assurance guidance must be mapped to the organization's jurisdiction, risk, customer population, authenticators, and contractual identity system; it does not prescribe refund thresholds.
- **Consequence:** Email address, phone number, caller ID, ticket number, order detail, or familiar conversation is not sufficient proof by itself. Identity, customer intent, eligibility, and organizational approval remain distinct records.
- **Refresh trigger:** Identity/recovery/authenticator policy or NIST revision changes.

### Decision CS-05 — filter knowledge by access, version, and applicability before rank

- **Class:** Grounding and privacy.
- **Decision:** Every outcome-changing claim keeps evidence source, object/version, effective interval, locale, access decision, retrieval time, and freshness. Tenant/access/publication/applicability filters precede semantic rank.
- **Evidence:** Zendesk's [Help Center Articles API](https://developer.zendesk.com/api-reference/help_center/help-center-api/articles/) exposes draft/publication, locale, update, permission, and user-segment properties; its [Help Center API introduction](https://developer.zendesk.com/api-reference/help_center/help-center-api/introduction/) documents permission-filtered responses. Salesforce's [Knowledge article version model](https://developer.salesforce.com/docs/service/salesforce-knowledge-dev-guide/guide/knowledge-development-object-managing-articles.html) separates article identity from draft/online/archive versions.
- **Limit/contradiction:** Vendor fields do not create a universal policy precedence rule. Semantic retrieval scores cannot adjudicate two valid conflicting authorities.
- **Consequence:** A deterministic policy service decides eligibility. Material conflicts block the affected claim/effect and create a knowledge-quality route.
- **Refresh trigger:** Knowledge platform, access model, policy-source hierarchy, or indexing behavior changes.

### Decision CS-06 — keep context minimal and memory typed

- **Class:** Context, memory, and privacy.
- **Decision:** Build a task-specific context; separate turn, working, session, durable task, domain, long-term customer, and episodic memory; disable ungoverned raw cross-case memory.
- **Evidence:** Anthropic's [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) frames context as finite and recommends high-signal selection, just-in-time retrieval, structured notes, and careful compaction. OpenAI's [conversation state guide](https://developers.openai.com/api/docs/guides/conversation-state) documents provider-managed state and retention/billing behavior that differs from an application record.
- **Limit/contradiction:** Context guidance is engineering experience, not a fixed token formula. Provider storage settings and lifetimes can change and differ by API/configuration.
- **Consequence:** Authoritative case/provider facts are fetched by reference. Long-term customer memory is typed, purpose-bound, consent/retention-controlled where applicable; episodic production patterns are curated offline.
- **Refresh trigger:** Provider retention/state contract, privacy assessment, context-window, or memory architecture changes.

### Decision CS-07 — compaction is continuation, not evidence

- **Class:** Durability and context.
- **Decision:** A schema-validated continuation package preserves goals, verified evidence refs, open questions, attempts, active approvals/effects, deadlines, and event watermark. Resume revalidates all versions.
- **Evidence:** OpenAI's [compaction guide](https://developers.openai.com/api/docs/guides/compaction) documents server-side compaction and opaque compaction items intended to continue context. Anthropic notes that compaction can lose subtle details.
- **Limit/contradiction:** Opaque provider compaction may be effective for model continuity precisely because it is not a readable audit or authoritative ledger.
- **Consequence:** Raw case events/evidence and approval/effect receipts remain in application systems. A summary can be discarded and rebuilt.
- **Refresh trigger:** Compaction implementation/schema or provider state semantics change.

### Decision CS-08 — provider events are duplicate-, loss-, lag-, and reorder-tolerant

- **Class:** Events and reliability.
- **Decision:** Verify callbacks, deduplicate stable provider event IDs, retain event/ingestion time, make consumers replay-safe, and reconcile important state through provider lookup.
- **Evidence:** Zendesk's [webhook operational guide](https://developer.zendesk.com/documentation/webhooks/creating-and-monitoring-webhooks/) describes best-effort delivery, retries, duplication/loss considerations, and circuit breaking; its [webhook verification guide](https://developer.zendesk.com/documentation/webhooks/verifying/) documents signature verification. Zendesk's [event integration introduction](https://developer.zendesk.com/api-reference/integration-services/trigger-events/introduction/) explicitly documents at-least-once delivery and duplicates. Stripe's [webhook guide](https://docs.stripe.com/webhooks) warns that events can duplicate and are not guaranteed in order.
- **Limit/contradiction:** Retry schedules, signature schemes, delivery guarantees, and ordering metadata differ by provider and can change.
- **Consequence:** No callback directly applies an effect twice or overwrites a newer case state. High-impact completion is verified against current provider state.
- **Refresh trigger:** Any webhook/event version, verifier, secret rotation, or provider delivery-policy change.

### Decision CS-09 — accepted, sent, delivered, and read are different

- **Class:** Channel continuity.
- **Decision:** Track channel-specific delivery states and define the best observable success per channel; never repeat the underlying effect because its notice failed.
- **Evidence:** Twilio's [Message resource](https://www.twilio.com/docs/messaging/api/message-resource) exposes queued/sending/sent/delivered/undelivered/failed-style states, while its [webhook security guide](https://www.twilio.com/docs/usage/webhooks/webhooks-security) warns webhook parameters can evolve and recommends maintained signature validation. Zendesk's [delivery event documentation](https://developer.zendesk.com/documentation/conversations/messaging-platform/programmable-conversations/delivery-events/) distinguishes delivery to a channel from delivery to a person and notes channel-dependent confirmation/order behavior.
- **Limit/contradiction:** State names and assurance vary by channel/provider; delivery often does not prove the intended person read or understood a notice.
- **Consequence:** Closure predicates and required-notice runbooks use declared channel capabilities and alternate-channel policy.
- **Refresh trigger:** Channel provider status/callback/consent behavior changes.

### Decision CS-10 — SLA calculation is application-owned and reconciled

- **Class:** Service operations.
- **Decision:** Compute contract/priority/calendar/pause deadlines with a versioned application rule and compare platform metric events. Sentiment is only a tone/review hint.
- **Evidence:** Zendesk's [SLA policy guide](https://support.zendesk.com/hc/en-us/articles/5600997516058-About-SLA-policies-and-how-they-work) documents first-match policies, priority dependence, business/calendar hours, and a limitation involving AI-agent tickets. Its [Ticket Metric Events API](https://developer.zendesk.com/api-reference/ticketing/tickets/ticket_metric_events/) exposes reply/work/wait/resolution and breach-related events.
- **Limit/contradiction:** This is a useful counterexample to assuming a vendor's human-ticket SLA covers automation; the exact current limitation and plan behavior must be checked for the deployment.
- **Consequence:** Automated containment cannot silently remove a case from contractual tracking. SLA, internal SLO, priority, and sentiment remain distinct.
- **Refresh trigger:** Support contract, calendar, platform SLA plan/API, or automation eligibility changes.

### Decision CS-11 — exact D3 authorization is independent of the model

- **Class:** Authority and effects.
- **Decision:** Customer/object binding, deterministic eligibility, organizational authority, and commit-time checks are separate gates. Approval binds the exact request digest and expires on relevant changes.
- **Evidence:** OpenAI's approval guidance identifies cancellations and edits as examples needing approval boundaries, while its safety guide recommends human review for high-impact domains and constraining inputs/outputs. See [OpenAI safety best practices](https://developers.openai.com/api/docs/guides/safety-best-practices).
- **Limit/contradiction:** Model-provider guardrails and SDK approvals are useful mechanisms but not proof of identity, business eligibility, approver role, separation of duties, or current provider state.
- **Consequence:** A model never self-approves, chooses approval identity, edits approved fields, or commits through a broad connector. Narrow deterministic pre-authorization is possible only when independently contained and tested.
- **Refresh trigger:** Authority policy, risk thresholds, operation consequence, approval service, or SDK behavior changes.

### Decision CS-12 — idempotency begins with semantic intent and ends with reconciliation

- **Class:** Reliability and effects.
- **Decision:** Persist one semantic effect intent before calling a provider; reuse its provider idempotency identity for the exact request; reject parameter drift; model ambiguous outcomes as `unknown`; verify exact postconditions.
- **Evidence:** AWS's [Making retries safe with idempotent APIs](https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/) discusses caller-supplied request identity, semantic equivalence, late requests, and mismatch. [RFC 9110](https://datatracker.ietf.org/doc/html/rfc9110) limits automatic retry assumptions for non-idempotent methods. Stripe's [idempotent request guide](https://docs.stripe.com/api/idempotent_requests) documents provider-specific key/result and parameter-mismatch behavior.
- **Limit/contradiction:** Provider idempotency retention, error caching, mismatch behavior, and business semantics vary. HTTP or workflow-engine durability does not make arbitrary external effects exactly once.
- **Consequence:** Timeout after a possible commit triggers lookup/reconciliation, not a new key. Compensation is a separately authorized effect.
- **Refresh trigger:** Provider idempotency contract/API version or effect workflow changes.

### Decision CS-13 — effect semantics are provider-versioned

- **Class:** Integration and financial/service effects.
- **Decision:** Refund, credit, subscription/order cancellation, replacement, entitlement, and notification operations each have a certified provider-specific contract and exact postcondition.
- **Evidence:** Stripe's [refund creation](https://docs.stripe.com/api/refunds/create) and [refund object](https://docs.stripe.com/api/refunds/object) expose partial/multiple refund rules and pending/action/succeeded/failed/canceled-style states. Stripe's [subscription cancellation guide](https://docs.stripe.com/billing/subscriptions/cancel) documents immediate versus period-end and invoice/proration consequences. Shopify's current [refundCreate](https://shopify.dev/docs/api/admin-graphql/latest/mutations/refundCreate) and [orderCancel](https://shopify.dev/docs/api/admin-graphql/latest/mutations/orderCancel) show store-credit/restock choices, irreversibility/preconditions, and asynchronous job behavior; Shopify also documents [idempotent requests](https://shopify.dev/docs/api/usage/idempotent-requests).
- **Limit/contradiction:** These examples deliberately conflict with any universal “refund/cancel succeeded” abstraction. Exact fields, versions, idempotency requirements, and states change over time.
- **Consequence:** The connector registry pins API versions and records async jobs, terminal states, reversibility, dependent objects, lookup, reconciliation, and cancellation/compensation semantics.
- **Refresh trigger:** Provider API/changelog, merchant/account configuration, currency/region, or business policy changes.

### Decision CS-14 — durable orchestration is optional; durable effects are mandatory

- **Class:** Runtime and durability.
- **Decision:** Start with a relational state machine, durable queue/scheduler, and separate workers when sufficient. Adopt a durable workflow engine when long waits, signals, replay, and worker versioning justify its complexity.
- **Evidence:** Temporal's [documentation](https://docs.temporal.io/) describes crash-resilient workflow execution; Restate's [workflow documentation](https://docs.restate.dev/tour/workflows) describes journaled workflows and keyed execution. Both demonstrate implementation options for durable application progress.
- **Limit/contradiction:** A durable engine cannot prove whether an uncooperative external provider applied an effect. Replay compatibility, determinism, worker/version migration, and operations remain engineering obligations.
- **Consequence:** The blueprint specifies state/effect contracts rather than requiring a vendor. All implementations retain provider reconciliation and versioned migrations.
- **Refresh trigger:** Case wait/recovery complexity, runtime choice, or workflow-engine compatibility model changes.

### Decision CS-15 — evaluate authoritative outcomes and multi-trial reliability

- **Class:** Evaluation.
- **Decision:** A task includes environment, expected state, invariants, and faults; hard security/policy/effect gates precede trajectory and response quality; stochastic tasks use multiple trials.
- **Evidence:** OpenAI's [agent evaluation guide](https://developers.openai.com/api/docs/guides/agent-evals) describes reproducible evals and trace grading. Anthropic's [Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) distinguishes tasks, trials, transcripts, outcomes, graders, and suites and emphasizes environment outcomes. The [τ-bench paper](https://arxiv.org/abs/2406.12045) and current [τ-bench lineage repository](https://github.com/sierra-research/tau2-bench) provide customer-service policy/tool interaction tasks.
- **Limit/contradiction:** The original [τ-bench repository](https://github.com/sierra-research/tau-bench) now points readers to a newer lineage, and public repository issues document evaluator/task realism defects. Benchmarks are revision-sensitive and omit much of production identity, privacy, async effects, delivery, capacity, and incident behavior.
- **Consequence:** Pin dataset revision, inspect tasks/graders, use benchmarks only as seeds, and add environment-backed organization-specific failure injection.
- **Refresh trigger:** Benchmark revision/issues, workload drift, grader/model change, or a production incident.

### Decision CS-16 — separate control audit from diagnostic traces

- **Class:** Observability and privacy.
- **Decision:** Identity/access, policy, approval, effect, delivery, and state-transition evidence is unsampled. Metrics and sampled traces use opaque references with raw content off by default.
- **Evidence:** [W3C Trace Context](https://www.w3.org/TR/trace-context/) cautions against sensitive information in trace context. OpenTelemetry's [generative-AI agent span conventions](https://github.com/open-telemetry/semantic-conventions-genai/blob/main/docs/gen-ai/gen-ai-agent-spans.md) are marked as development, and its [generative-AI span guidance](https://github.com/open-telemetry/semantic-conventions-genai/blob/main/docs/gen-ai/gen-ai-spans.md) treats input/output content as sensitive and potentially large. OpenAI's [integrations and observability guide](https://developers.openai.com/api/docs/guides/agents/integrations-observability) describes traces and hosted/local integration choices.
- **Limit/contradiction:** Tracing products can make content capture convenient, but convenience is not a legal basis, access policy, or durable audit. Semantic-convention fields may still change.
- **Consequence:** Use an internal versioned telemetry schema and evidence references. Trace IDs correlate but never authorize.
- **Refresh trigger:** OTel convention stabilization, exporter/provider contract, privacy assessment, or audit policy changes.

### Decision CS-17 — deploy through shadow, assist, and separately gated canaries

- **Class:** Release and operations.
- **Decision:** Every behavior dependency is versioned in one manifest; release progresses from offline to shadow, human assist, low-risk D1/D2 canary, and separate D3-class canary with independent rollback/kill switches.
- **Evidence:** Google's [Canarying Releases](https://sre.google/workbook/canarying-releases/) recommends bounded exposure, control comparison, and checking absolute reliability criteria. Current model guidance recommends pinning versions where consistency matters and evaluating upgrades; see [OpenAI model selection guidance](https://developers.openai.com/api/docs/guides/latest-model).
- **Limit/contradiction:** Statistical improvement against a control cannot waive a hard control breach. Model aliases, hosted tools, policies, and knowledge can change outside application code.
- **Consequence:** Manifest includes model snapshot, prompt, schema, tool/connector, policy, knowledge, safety, workflow, compaction, eval, and runtime versions. In-flight work drains or migrates compatibly.
- **Refresh trigger:** Any manifest dependency, release system, or model/provider lifecycle changes.

### Decision CS-18 — scale does not increase authority

- **Class:** Capacity and governance.
- **Decision:** Stage 5 adds admission, fairness, queues, cells/regions, disaster recovery, graceful degradation, and cost controls but no new authority merely because volume grows.
- **Evidence:** Google's [Implementing SLOs](https://sre.google/workbook/implementing-slos/) provides a service-level framework; the blueprint applies queueing and capacity principles to support-specific deadlines and unknown effects. Provider rate-limit and async behavior in the sources above reinforce the need for dependency-aware backpressure.
- **Limit/contradiction:** No universal SLO, headroom, RTO/RPO, or region layout fits every support contract. Arbitrary copied targets would be unsupported.
- **Consequence:** Teams set targets from contract/risk/baseline/load tests, reserve reconciliation and safety capacity, and measure cost per verified resolution.
- **Refresh trigger:** Workload/provider mix, contracts, regional requirements, topology, or economics change.

### Decision CS-19 — qualify provider capabilities, not vendor connectors

- **Class:** Integration and operations.
- **Decision:** Admit each helpdesk/CRM, identity, knowledge/status, email/chat/voice, commerce/shipment and billing/effect operation through a dated, expiring capability declaration with target-account permission, schema, version, state, failure, privacy and recovery evidence.
- **Evidence:** Intercom documents API-version-dependent webhook serialization and ticket `resolved`/conversation `closed` behavior; Salesforce documents different access paths for on- and off-platform conversation data and recommends a newer Conversation Data GET API for new data-access integrations; Shopify's GraphQL model exposes related order, fulfillment-order, fulfillment and tracking objects with nontrivial cancellation behavior. See the added primary sources below.
- **Limit/contradiction:** Provider documentation describes products, not the deployment's plan, tenant configuration, roles, custom automations, contracts or real failure path. A passed OAuth flow proves neither business scope nor negative object/field isolation.
- **Consequence:** “Zendesk connector” or “commerce tool” is never an authority unit. Writes remain disabled until an exact postcondition/reconciliation query and ambiguity tests exist. Certification expires on schedule and immediately after relevant version, scope, config, incident or drift changes.
- **Refresh trigger:** Provider API/schema/product/plan/region, credential scope, source-system automation, event, quota, retention or reconciliation behavior changes.

### Decision CS-20 — voice, recording and transcript states remain separate

- **Class:** Channel, privacy and evidence.
- **Decision:** Track call/participant, call completion, recording-policy decision, recording processing/availability, transcription, transcript association/correction, retention/deletion and case resolution as separate provider/application records. Recording consent is not inferred from service-channel participation.
- **Evidence:** Twilio documents separately subscribed call-progress events, out-of-order callback arrival, recording processing/status callbacks and legal consent considerations. Salesforce documents that voice conversation entries may associate with the voice-call record only after the call ends. Provider media/transcription/configuration behavior is product-specific.
- **Limit/contradiction:** Provider controls do not determine which recording/monitoring law, notice, consent, workforce rule, PCI/health restriction or retention period applies to an organization or participant set. Transcript and speaker quality are not guaranteed.
- **Consequence:** Raw media is restricted; the resolver receives only necessary time-coded utterances with speaker/provenance uncertainty. Transcript text cannot establish identity, policy, intent, effect or truth without owning validation. Denied/withdrawn recording consent routes to unrecorded or alternate support where possible.
- **Refresh trigger:** New channel/region/participant class, recording/transcription provider or model, notice/consent rule, media configuration, storage/retention/deletion/security feature or recording incident.

## Contradictions and important limits

| Tempting claim | Evidence-based correction | Blueprint treatment |
|---|---|---|
| “The ticket transcript is the state.” | Support platforms expose versioned tickets, audits, metrics, comments, and propagation behavior; provider objects have separate truth. | Case, event, evidence, workflow, delivery, and effect records are explicit. |
| “A signed webhook happens once and in order.” | Official providers document retries, duplicates, best-effort behavior, and missing order guarantees. | Authenticate, deduplicate, make consumers replay-safe, reconcile. |
| “The message API succeeded, so the customer got it.” | Providers distinguish request acceptance, sent, channel delivery, failure, and sometimes read. | Track channel-specific assurance and alternate-notice workflow. |
| “A refund/cancel call is a simple reversible action.” | Providers expose partial, pending, failed, irreversible, deferred, proration, restock, and async-job semantics. | Certify each provider/version and verify exact postconditions. |
| “Idempotency means exactly once.” | Provider key retention/semantics vary; late/ambiguous outcomes remain. | Stable semantic intent plus provider key plus reconciliation. |
| “An SDK approval makes the action authorized.” | Approval nodes can pause/resume runs but do not prove identity, eligibility, approver rights, or current state. | Application-owned exact authority and commit gates. |
| “Hosted conversation state is durable case memory.” | Provider response/conversation storage, lifetime, billing, and compaction differ and may be opaque. | Application case/effect ledger and readable continuation package. |
| “More agents mirror the support organization better.” | Current orchestration guidance treats extra agents as complexity justified by evaluated specialization. | One resolver by default; deterministic services/tools for specialist facts. |
| “High retrieval similarity resolves policy conflicts.” | Access/publication/version/effective-date semantics precede rank; two valid authorities can conflict. | Block affected claim/effect and route to content/policy owner. |
| “Vendor SLA automatically covers automated tickets.” | Support-platform rules can treat automation differently. | Application SLA record reconciled with provider events. |
| “Benchmark success proves deployability.” | Public tasks/evaluators evolve and issues expose realism/robustness gaps. | Revision-pinned seed plus production-specific environment/fault tests. |
| “Full traces are necessary for accountability.” | Raw model/tool content is sensitive and conventions are evolving; sampled traces cannot prove every effect. | Unsampled structured control audit; privacy-filtered diagnostic traces. |
| “An OAuth connection means the adapter is qualified.” | OAuth proves a protocol exchange, not the effective object/field scope, business authority, account configuration, plan behavior, negative isolation, or recovery path. | Certify each operation and target tenant/account through an expiring capability declaration and fault suite. |
| “A public status page proves this customer is affected.” | A published component incident is scoped communication; tenant configuration and symptoms can differ. | Correlate public status with authenticated tenant-safe telemetry and preserve uncertainty. |
| “The transcript proves who spoke and what must happen.” | Recording, speaker attribution, transcription, identity, intent, policy and provider effects have independent state and error modes. | Use transcript text only as attributed evidence; revalidate identity, current state, authorization and effect results. |
| “Restoring workers means recovery is complete.” | Backlogs, callbacks, reconciliations, approvals, notifications and provider quotas can amplify load after dependency recovery. | Reserve recovery capacity, restore by consequence, reconcile before replay, and declare recovery only after state convergence. |

## Rejected designs

### Pure chatbot with direct business APIs

Rejected because conversational context would become implicit identity, policy, state, and authority; provider ambiguity would be invisible; and there would be no durable reconciliation owner.

### Model-generated policy decisions

Rejected because retrieval and generation cannot reliably apply exact monetary, temporal, jurisdictional, plan, exception, and effective-version rules. The model may explain a versioned deterministic decision.

### One broad support service credential

Rejected because read, public message, protected disclosure, refund, cancellation, administration, and cross-tenant operations have different consequences. Separate principals and exact effect commands reduce blast radius.

### Multi-agent department simulation

Rejected for the base design because it copies organizational labels without creating authoritative domain separation. Typed deterministic services and human queues provide clearer ownership. A specialist model requires independent evaluation evidence.

### Provider callback as completion

Rejected because callbacks can be duplicate, late, lost, reordered, or refer to partial/async state. Completion is an exact verified postcondition.

### Generic vendor connector certification

Rejected because one provider exposes read, internal note, public reply, status change, export, refund, cancellation, recording and administration capabilities with different scopes and consequences. Certification attaches to an exact operation, target account, principal, version and evidence suite, not a vendor logo.

### Transcript, caller ID, or case number as identity

Rejected because these are lookup or conversation evidence, not authentication. Material disclosure and effects bind to an identity-service assurance decision and current object authorization.

### Automatic promotion of successful cases into memory

Rejected because positive feedback, case closure, tone or low handle time can coexist with unauthorized exceptions, hidden repeat contact, biased treatment or wrong effects. Candidates enter a quarantined failure/feedback-mining pipeline and require authoritative outcome checks, privacy eligibility, review and regression evidence.

### Full transcript and raw history in every prompt

Rejected because it increases cost, latency, data exposure, prompt-injection surface, and attention dilution. Minimum evidence plus versioned references and on-demand reads are safer.

### Cross-case free-form customer memory

Rejected by default because it can fossilize errors, leak data, bias service, and evade purpose/retention/deletion controls. Only typed, explicitly governed customer preferences are eligible.

### Optimizing deflection or CSAT alone

Rejected because an unauthorized exception can score well and a correct policy denial can score poorly. Verified outcomes and noncompensating control gates come first.

## Source register

All sources below were accessed on 2026-08-31. Dynamic documentation should be rechecked against the deployment's pinned versions.

### Agent runtime, context, approvals, and evaluation

| Source | Primary contribution | Boundary |
|---|---|---|
| [OpenAI model selection guidance](https://developers.openai.com/api/docs/guides/latest-model) | Model snapshots/aliases, Responses API guidance, evaluation-led selection | OpenAI-specific and dynamic |
| [OpenAI conversation state](https://developers.openai.com/api/docs/guides/conversation-state) | Stored response/conversation lifecycle and token implications | Not an application case ledger |
| [OpenAI compaction](https://developers.openai.com/api/docs/guides/compaction) | Server-side compaction and continuation | Opaque compacted state is not audit evidence |
| [OpenAI orchestration](https://developers.openai.com/api/docs/guides/agents/orchestration) | Manager/handoff choices and single-agent starting point | Framework guidance, not universal proof |
| [OpenAI guardrails and approvals](https://developers.openai.com/api/docs/guides/agents/guardrails-approvals) | Input/output/tool guardrail placement and resumable approvals | Does not replace application authorization |
| [OpenAI integrations and observability](https://developers.openai.com/api/docs/guides/agents/integrations-observability) | Hosted/local integrations and tracing | Connection convenience is not permission |
| [OpenAI agent evals](https://developers.openai.com/api/docs/guides/agent-evals) | Reproducible evaluation and trace grading | Must add environment outcome checks |
| [OpenAI safety best practices](https://developers.openai.com/api/docs/guides/safety-best-practices) | Red teaming, human review, input/output constraints | Generic controls require workload-specific enforcement |
| [Anthropic effective context engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) | High-signal context, just-in-time retrieval, structured notes, compaction trade-offs | Engineering guidance, provider-authored |
| [Anthropic demystifying agent evals](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) | Task/trial/outcome/grader/suite model and environment outcomes | Engineering guidance, not a standard |

### Support and channel systems

| Source | Primary contribution | Boundary |
|---|---|---|
| [Zendesk Tickets API](https://developer.zendesk.com/api-reference/ticketing/tickets/tickets/) | Ticket states, requester/submitter, audits, optimistic updates and visibility | Zendesk-specific |
| [Zendesk Ticket Audits API](https://developer.zendesk.com/api-reference/ticketing/tickets/ticket_audits/) | Read-only update history and events | Retention/pagination still provider-specific |
| [Zendesk Ticket Metric Events](https://developer.zendesk.com/api-reference/ticketing/tickets/ticket_metric_events/) | Reply/work/wait/resolution and SLA events | Platform semantics are not the customer contract by themselves |
| [Zendesk SLA policies](https://support.zendesk.com/hc/en-us/articles/5600997516058-About-SLA-policies-and-how-they-work) | First-match, priority, calendars, automation limitation | Plan/current behavior must be recertified |
| [Zendesk Articles API](https://developer.zendesk.com/api-reference/help_center/help-center-api/articles/) | Publication, locale, permission, update metadata | Knowledge policy still organization-owned |
| [Zendesk webhook operations](https://developer.zendesk.com/documentation/webhooks/creating-and-monitoring-webhooks/) | Best-effort delivery, retries, duplicate/loss and breaker considerations | Retry behavior may evolve |
| [Zendesk webhook verification](https://developer.zendesk.com/documentation/webhooks/verifying/) | HMAC verification pattern | Use exact current provider algorithm/library |
| [Twilio Message resource](https://www.twilio.com/docs/messaging/api/message-resource) | Messaging lifecycle/status | Channel/provider semantics differ |
| [Twilio webhook security](https://www.twilio.com/docs/usage/webhooks/webhooks-security) | Signature validation and evolving parameters | Validate through maintained SDK/configuration |
| [Intercom webhook models and topics](https://developers.intercom.com/docs/references/webhooks/webhook-models) | API-version-dependent payload serialization, ticket-resolution/conversation-closure distinction, response deadline, retries and signatures | Workspace configuration and target API version still require certification |
| [Intercom REST API](https://developers.intercom.com/docs/references/rest-api/api.intercom.io) | Current versioned API surface and regional endpoints | Dynamic documentation; pin the deployment version and region |
| [Salesforce conversation data access](https://developer.salesforce.com/docs/service/messaging-object-model/guide/messaging-object-model-access-data.html) | On-platform/off-platform access paths, API choice and post-call voice association | Org permissions, licenses and object/field configuration are deployment-specific |
| [Salesforce Enhanced Chat server-sent events](https://developer.salesforce.com/docs/service/messaging-api/references/about/server-sent-events-structure.html) | Evolving event schemas and safe handling of unknown event types | Messaging product/version specific |
| [Twilio Voice webhooks](https://www.twilio.com/docs/usage/webhooks/voice-webhooks) | Separate call-progress and recording callbacks and asynchronous recording processing | Callback selection and account configuration vary |
| [Twilio Recording resource](https://www.twilio.com/docs/voice/api/recording) | Recording lifecycle, media/channel controls, deletion and consent warning | Legal basis, participant notice/consent and retention remain organization-specific |
| [Atlassian Statuspage API types](https://support.atlassian.com/statuspage/docs/what-are-the-different-apis-under-statuspage/) | Separation of public status reads and authenticated management operations | Published component state is not tenant-impact proof |

### Identity, effects, reliability, and standards

| Source | Primary contribution | Boundary |
|---|---|---|
| [NIST SP 800-63B-4](https://pages.nist.gov/800-63-4/sp800-63b.html) | Authentication, sessions, recovery, social-engineering considerations | Must be risk/jurisdiction mapped |
| [Stripe refunds](https://docs.stripe.com/api/refunds/create) | Partial/multiple refund creation semantics | Stripe/version/account specific |
| [Stripe refund object](https://docs.stripe.com/api/refunds/object) | Refund lifecycle states and next action | Normalization must retain provider detail |
| [Stripe idempotent requests](https://docs.stripe.com/api/idempotent_requests) | Key/result/mismatch behavior | Retention and behavior are provider-specific |
| [Stripe subscription cancellation](https://docs.stripe.com/billing/subscriptions/cancel) | Timing, invoice, proration, and terminal implications | Billing configuration dependent |
| [Shopify refundCreate](https://shopify.dev/docs/api/admin-graphql/latest/mutations/refundCreate) | Refund/store-credit/restock options and current idempotency behavior | Current Admin GraphQL version only |
| [Shopify orderCancel](https://shopify.dev/docs/api/admin-graphql/latest/mutations/orderCancel) | Irreversibility, preconditions, dependent work, async job | Current Admin GraphQL version only |
| [Shopify Fulfillment object](https://shopify.dev/docs/api/admin-graphql/latest/objects/Fulfillment) | Order/fulfillment/tracking relationships and multiple-fulfillment semantics | Current Admin GraphQL version; carrier/provider detail varies |
| [Shopify fulfillmentOrderCancel](https://shopify.dev/docs/api/admin-graphql/latest/mutations/fulfillmentOrderCancel) | Fulfillment cancellation, replacement work and completion race | Current Admin GraphQL version and fulfillment-service behavior |
| [AWS retries with idempotent APIs](https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/) | Semantic request identity, late requests, mismatch | Engineering pattern, not provider guarantee |
| [RFC 9110](https://datatracker.ietf.org/doc/html/rfc9110) | HTTP idempotent-method and retry semantics | Business effect semantics remain application/provider-specific |
| [W3C Trace Context](https://www.w3.org/TR/trace-context/) | Interoperable correlation and privacy caution | Trace context is not identity or authorization |
| [OpenTelemetry generative-AI spans](https://github.com/open-telemetry/semantic-conventions-genai/blob/main/docs/gen-ai/gen-ai-spans.md) | Emerging span/content conventions and sensitivity | Development/stability status must be monitored |
| [Google SRE canarying releases](https://sre.google/workbook/canarying-releases/) | Bounded canary and absolute/control evaluation | Needs support-specific hard gates |

### Research benchmarks

| Source | Primary contribution | Boundary |
|---|---|---|
| [τ-bench paper](https://arxiv.org/abs/2406.12045) | Policy/tool/customer interaction benchmark framing | Research environment, not production proof |
| [Current τ-bench lineage repository](https://github.com/sierra-research/tau2-bench) | Maintained task/evaluator lineage and releases | Pin commit and inspect issues/tasks |
| [Original τ-bench repository](https://github.com/sierra-research/tau-bench) | Historical implementation and migration notice | Explicitly superseded; do not treat as current default |

## Provider certification worksheet

Complete this before enabling any real connector operation:

```yaml
provider_certification:
  provider: "..."
  product_and_plan: "..."
  provider_account_and_region: "..."
  api_and_schema_version: "..."
  certified_at: "..."
  expires_at: "..."
  owner: "..."
  operations:
    - name: "..."
      authority_class: "D1|D2|D3|D4"
      source_role: "authoritative|projection|customer_claim|derived"
      credential_scope: "..."
      identity_and_tenant_binding: "..."
      positive_and_negative_object_field_tests: "..."
      request_and_response_schemas: "..."
      money_time_locale_rules: "..."
      timeout_and_rate_limit: "..."
      idempotency_key_semantics: "..."
      conditional_update_or_precondition: "..."
      asynchronous_states_and_jobs: "..."
      terminal_states: "..."
      callback_authentication: "..."
      duplicate_reorder_loss_behavior: "..."
      read_after_write_and_lag: "..."
      cancellation_reversal_compensation: "..."
      reconciliation_query_and_postcondition: "..."
      evidence_and_redaction_fields: "..."
      recording_consent_and_artifact_lifecycle: "n/a|policy-and-test-ref"
      sandbox_fault_tests: "..."
      changelog_subscription: "..."
```

## Remaining uncertainties

- Exact SLA obligations, pause rules, automated-case coverage, business calendars, and remedies are organization- and contract-specific.
- Identity assurance, recovery, consent, privacy, retention, data residency, recording/transcription, and regulated-case requirements require organizational legal/security decisions.
- Refund/credit/cancellation limits, exception rights, separation of duties, and compensation are policy decisions, not model settings.
- Provider semantics depend on account configuration and pinned API/product versions; documentation alone does not replace sandbox and failure testing.
- Voice recording, monitoring, transcription, biometrics, workforce review and secondary-use requirements differ by participant location, purpose and organization; provider feature availability is not legal authorization.
- Status publication, carrier tracking, transcript confidence and channel delivery are evidence with bounded assurance, not universal proof of tenant impact, physical receipt, speaker identity or human comprehension.
- No public benchmark located covers the full combination of authenticated omnichannel support, adversarial knowledge, concurrent human ownership, durable waits, exact financial effects, delivery uncertainty, privacy, and incident recovery.
- SLO, RTO/RPO, headroom, queue priority, review sampling, and unit-economic targets require measured workload and customer contracts; the blueprint intentionally does not invent them.
- Model and tool behavior is snapshot-dependent. A release must evaluate the exact manifest used in production.

## Refresh triggers

Refresh this packet immediately when any of these occur:

- model snapshot/API, conversation storage, compaction, tool calling, guardrail, approval, MCP, or tracing behavior changes;
- support, identity, knowledge, channel, payment, subscription, commerce, or workflow provider version/changelog changes;
- adapter scope, target account, plan, region, event schema, object/field visibility, recording/transcription configuration, certification expiry, or unexplained provider drift changes;
- authority limits, refund/cancellation policy, SLA contract, calendar, identity/recovery, privacy/retention, or regional policy changes;
- a new channel, locale, tenant tier, customer population, effect type, or regulated/safety case enters scope;
- optimistic-concurrency, event, callback, rate-limit, idempotency, async-job, delivery, or reconciliation semantics change;
- public benchmark tasks/graders are revised or new issues invalidate a relied-on result;
- a production incident reveals an unsupported assumption, missing invariant, or evaluator blind spot;
- workload, latency, cost, human capacity, repeat-contact, or failure distribution drifts materially;
- OpenTelemetry generative-AI conventions stabilize or change incompatibly;
- a proposed specialist/multi-agent architecture seeks production admission.

At minimum, review dynamic vendor sources and the behavior manifest before each relevant release. Perform a broader evidence review on the repository's normal research cadence even when no incident occurs.

## Research handoff checklist

- [ ] Blueprint claims that depend on provider behavior link to the exact official source and identify the provider boundary.
- [ ] No vendor-specific behavior is presented as a universal guarantee.
- [ ] Contradictions around SLA, delivery, idempotency, compaction, approvals, benchmarks, and tracing are preserved.
- [ ] Exact connector/account/API versions are certified separately from this general packet.
- [ ] Each production operation has a dated capability declaration, negative visibility tests, reconciliation evidence, owner, expiry, outage mode and recovery-load test.
- [ ] Call, participant, recording-consent, recording, transcript, deletion and case-resolution state are modeled independently.
- [ ] Unsupported legal, contractual, assurance, financial, SLO, and capacity values remain organization decisions.
- [ ] Future changes update the packet, relevant guide, eval suite, behavior manifest, and provider certification together.
