# Research Packet: FinOps and Cloud-Cost Agent Blueprint

**Research date:** 2026-08-31  
**Pass 2 source access date:** 2026-08-31 (all links in this packet unless a page is later marked unavailable)  
**Scope:** Cloud and direct AI-provider cost allocation, anomaly triage, forecasting, budget guardrails, rate/usage optimization, approval-gated effects, and realized-savings verification.  
**Output:** [FinOps and Cloud-Cost Agent Blueprint](../../agents/finops-cloud-cost-agent/README.md)  
**Primary-source register:** 122 sources

## Executive finding

The safe production design is not an autonomous cloud optimizer. It is a deterministic cost-data and workflow control plane with a low-authority reasoning component. Provider exports, immutable evidence, decimal monetary calculations, versioned allocation/forecast/policy services, durable case state, authenticated approval, and effect reconciliation remain authoritative. The language model explains selected evidence, generates labeled hypotheses, and drafts typed proposals.

This boundary follows from five cross-source findings:

1. Cloud cost data is delivered and corrected under provider-specific timing and schema semantics; it is not a single real-time final ledger.
2. FOCUS improves interoperability but provider implementations lag the current specification and retain extensions/gaps.
3. Provider anomalies, forecasts, and optimization recommendations expose useful signals but not the organization's complete SLO, security, commercial, ownership, or change context.
4. Provider budget and recommendation surfaces can lead to high-authority operations, including permission changes, automation, deletion/stop actions, and purchases. Those actions must not be inherited by the agent.
5. FinOps is a collaborative decision discipline. Accounting, procurement, financial authorization, service ownership, security, and infrastructure execution retain accountable human/domain owners.

## Research method

The pass prioritized current primary material:

- FinOps Foundation framework and FOCUS specification/repository;
- AWS, Microsoft Azure, Google Cloud, OpenCost, OpenAI, and Anthropic documentation;
- NIST, SEC, PCAOB, CloudEvents, W3C, and OpenTelemetry standards/guidance;
- two foundational papers for forecast evaluation and production ML failure framing.

Searches covered cost export schemas and delivery, corrections, allocation, anomaly/forecast/budget behavior, rightsizing and commitment recommendation assumptions, IAM/administrative permissions, Kubernetes allocation, direct AI-provider costs, workflow events, observability, evaluation, prompt-injection/security, and financial accountability.

Important claims were compared across providers. Sources were used to derive a common control contract rather than to claim identical provider behavior. Public documentation was treated as a moving compatibility surface; exact provider support must be retested at implementation time.

### Source access, version, and deployment-status log

| Surface | Public status observed on 2026-08-31 | Version/release pinned by the packet | Deployment-specific limitation to retest |
|---|---|---|---|
| FOCUS | Published release | Specification 1.4, ratified/published 2026-06-04 | Conformance belongs to a concrete delivered dataset; internal mapping does not upgrade a provider feed |
| AWS Data Exports | Generally available AWS service documentation | CUR 2.0, FOCUS 1.2 with AWS columns, export manifest/delivery behavior | Export query/table configuration, create-new/overwrite, billing view, organization/Billing Conductor setup, and post-period refresh settings differ by deployment |
| AWS Cost Explorer/Optimization Hub | Current public APIs and user guide | Cost Explorer API; Cost Optimization Hub daily recommendations with IDs valid up to 24 hours | Permissions, enabled organization scope, recommendation preferences, commercial terms, supported resource types, billing transfer, and API cost/quotas |
| Azure Cost Management | Current public exports/APIs and FinOps toolkit docs | Azure FOCUS `1.2-preview`; FinOps toolkit/hubs v12; REST examples at `2025-03-01` and `2026-06-01` where cited | EA/MCA/CSP/MPA offer and scope, credits/taxes/support inclusion, export path/overlap, selected schema, portal/API lookback, tenant/region and toolkit topology |
| Google Cloud Billing | Current docs; several named features remain Preview | BigQuery FOCUS 1.2 Preview; standard/detailed export; Recommender v1 semantics; spend-cap budgets Preview | Dataset location/backfill, export gaps while disabled, no delivery SLA, reseller/subaccount/project permissions, recommender availability, CUD product migration, and Preview terms |
| OpenCost/Kubernetes | CNCF Incubating project plus public spec/API | OpenCost specification and API as accessed; Kubernetes object UID/resourceVersion semantics | Deployed OpenCost release, pricing/cloud-cost integrations, Prometheus history/resolution, cluster UID, idle/shared policy, currency, and invoice reconciliation |
| Sustainability/carbon | Provider estimates with different methodology, lag, granularity, and assurance | AWS carbon export `model_version`; Google methodology/release status; Azure Emissions Impact Dashboard methodology; SCI 1.1 | Account/contract eligibility, historical backfill, methodology revisions, market/location basis, missing regions/services, assurance, and organizational reporting policy |
| Observability | OpenTelemetry core signals plus evolving semantic conventions | OTel semantic conventions 1.44.0; trace/log/metric component status as documented | Language SDK/collector support and individual convention-group stability; GenAI conventions are not assumed stable business state |
| CMDB/ITSM | Vendor APIs with organization-defined schemas and permissions | ServiceNow Australia IRE docs; Jira Cloud REST v3 and current rate-limit docs; Backstage current catalog model | Licensed edition/release, custom CI/issue types and fields, source reconciliation rules, project permissions, residency, webhook/retry behavior, and correlation-field availability |
| Third-party FinOps | Official vendor APIs/docs, not independently certified semantics | Vantage v2 OpenAPI surface, Finout API/Virtual Tags docs, Flexera CCO docs, Cloudability v3 docs | Licensed edition, region, data sources, amortization/FX/rule settings, history restatement, raw-data exit, API permissions/limits, and contractual retention |

The packet records public status, not account capability. Connector acceptance requires a captured response/schema, effective IAM review, negative authorization test, pagination/retry test, monetary reconciliation, correction replay, and an owner/refresh date.

## Evidence-to-decision map

| Evidence finding | Blueprint decision |
|---|---|
| FOCUS 1.4 is the current ratified specification, while cloud-provider FOCUS offerings documented during research are generally 1.2 or Preview | Preserve exact source/version, raw provider data, mapping release, extensions, and gaps; do not claim silent 1.4 conformance |
| FOCUS 1.4 adds stronger correction/delivery/completeness concepts and Billing Period, Contract Commitment, and Invoice Detail datasets | Make delivery/correction first-class; keep cost-and-usage, contract-commitment, billing-period, invoice, and agent workflow records distinct |
| AWS scheduled exports refresh at provider cadence; Azure open periods can be estimated/rerated; Google Cloud documents no latency guarantee and late/backfilled data | Separate event, billing, delivery, and correction times; mark open data provisional; invalidate downstream artifacts after material correction |
| Provider cost categories, tags/labels, and allocation features are useful but effective-date and coverage behavior varies | Use versioned allocation policy, explicit unallocated residual, historical replay, and source-specific semantics |
| Google Cloud GKE allocation uses resource requests and does not backfill; OpenCost has price-source/integration distinctions | State the allocation driver and price basis; reconcile Kubernetes totals to provider billing before treating them as billed cost |
| Provider anomaly and forecast services expose signals, not universal ground truth | Treat results as observations; use a deterministic detector/forecast service, labeled cases, uncertainty, and rolling-origin evaluation |
| Provider budgets notify and some providers support automation/permission actions; Google explicitly says budgets do not cap usage/spend | Baseline is alert-only. Exclude shutdown, billing disablement, IAM/SCP, SSM, and infrastructure action |
| Provider rightsizing recommendations depend on limited metrics/lookbacks and can use retail-rate or product-specific assumptions | Cross-check SLOs, headroom, seasonality, licensing, commitments, security, and change state; expire and refresh proposals |
| Recommendation APIs can surface stop/delete and commitment-purchase actions | Ingest as evidence only; do not expose a general control-plane or purchase tool |
| Commitment recommendation APIs require terms/lookbacks and provider-specific eligibility | Produce explicit no-purchase, coverage/utilization, vacancy, downside, lock-in, and migration scenarios; finance/procurement owns purchase |
| Organization-level AI usage/cost APIs use powerful administrative credentials | Isolate an ingestion broker; never expose admin keys or raw administrative tools to the model |
| Internal token-price calculations and provider cost data answer different questions | Preserve both as estimate versus provider-reported cost and reconcile the gap |
| FinOps invoicing/chargeback intersects finance processes | Keep showback/allocation evidence in scope but ledger posting, invoice certification, and accounting policy outside scope |
| SEC/PCAOB guidance places internal-control responsibility on management for applicable organizations | Preserve authenticated evidence and separation of duties; never describe the agent as the accountable control owner |
| CloudEvents and trace standards solve event/interoperability and correlation concerns, not authoritative workflow state | Use typed durable domain events and effect records; keep sampled traces non-authoritative |

## Current compatibility baseline

### FOCUS

FOCUS 1.4 was ratified on 2026-06-04 and is the current published release at this research date. Its published material adds delivery/correction/completeness and configuration-related capabilities, Billing Period, Contract Commitment, and Invoice Detail datasets, and commitment-related extensions. That makes 1.4 useful as a target vocabulary and research baseline.

It does **not** justify relabeling earlier provider feeds. Conformance is a property of a concrete dataset against a concrete specification, not of a mapper's column names.

### Provider exports

| Provider/source | Publicly documented compatibility during research | Production interpretation |
|---|---|---|
| AWS Data Exports for FOCUS | FOCUS 1.2 table dictionary | Store AWS source/version and extensions; use scheduled export manifest and API reads only for bounded lookup/reconciliation |
| Azure FinOps hubs/toolkit | FOCUS 1.2-oriented datasets and conversion paths | Prevent overlapping export scopes; retain Azure cost semantics and open-period/rerating state |
| Google Cloud BigQuery FOCUS | FOCUS 1.2 Preview linked dataset | Treat Preview/linked-dataset and latency/query-cost constraints explicitly; do not promise final delivery time |
| OpenCost | OpenCost specification/API, optionally integrated with cloud costs | State list/on-demand/negotiated price basis and reconcile to cloud billing |
| Direct AI providers | Provider-specific organization usage/cost APIs | Normalize into an internal AI-cost schema while preserving provider-specific usage dimensions and source cost |

This is not a permanent matrix. The implementation must include a compatibility test and owner for each connector release.

## Detailed findings

### Cost data is revisioned operational evidence

AWS promotes scheduled Data Exports and documents refresh behavior; Azure explains that cost-management data for an open billing period is estimated and can be rerated; Google Cloud says billing-export latency is not guaranteed and documents late data/backfill behavior. These differences make a universal `event_time + amount` record unsafe.

The blueprint therefore stores source artifact/dataset identity, schema version, usage/charge interval, billing/invoice period, delivery time, completeness, correction/supersession, currency, and mapping release. A material correction creates a new evidence snapshot and stale marker; it does not rewrite what a prior approver reviewed.

### Allocation is policy, not probabilistic memory

FinOps Foundation allocation guidance emphasizes assigning shared and direct costs to organizational constructs. Provider features offer useful dimensions and split/allocation mechanisms, but labels/tags, export scope, effective time, and provider behavior differ. GKE cost allocation based on requests and without backfill is a particularly clear example of why the allocation driver and effective date matter.

The design uses deterministic, effective-dated rules: direct hierarchy, verified service mapping, valid tags, approved shared-cost formula, then explicit unallocated. The model may suggest an owner but cannot activate or backdate a rule. Coverage is paired with residual amount, mapping freshness, direct/inferred share, and sampled accuracy to prevent a broad default rule from gaming the metric.

### Anomaly signals need a durable case layer

FinOps anomaly management and provider detectors support detection and management, but provider outputs vary in scope and assumptions. Duplicate/reordered notifications and data corrections can create false operational urgency. An expected deployment is not a permanent detector exception.

The blueprint deduplicates signals into a case while preserving every source detector. Suppression requires a narrow, owned, expiring annotation. The model produces hypotheses with supporting, contradicting, and missing evidence. Evaluation reports label coverage, precision, cost-weighted recall proxies, owner burden, and time-to-detect/acknowledge rather than a context-free “accuracy.”

### Forecasts and budgets are different decisions

FinOps framework guidance treats forecasting and budgeting as related capabilities. Provider forecast services can supply numerical signals, but the adopted forecast remains an accountable planning decision. The Hyndman/Koehler accuracy paper supports the choice to avoid relying on MAPE and to compare against scale-aware baselines.

The blueprint uses deterministic/statistical forecasting, rolling-origin tests, MAE, MASE, bias, and interval coverage. The model explains drivers and assumptions only. Budget proposals bind amount, currency, period, threshold, recipients, and action. The baseline action is notification only.

Google Cloud explicitly notes budgets do not cap usage or spending and its programmatic notifications can be delivered at least once/out of order. AWS budget actions can apply IAM/SCP or run Systems Manager automation. Those capabilities demonstrate why a “budget tool” is not inherently low risk.

### Optimization must preserve service and security constraints

AWS Compute Optimizer and Azure Advisor document lookbacks, signals, headroom/rightsizing behavior, and caveats. Azure cost recommendations may use retail-rate assumptions that do not fully represent commitments. AWS Cost Optimization Hub returns action types that include stop/delete and purchase-related suggestions, and recommendation IDs can be time-limited. These are valuable observations but incomplete execution authority.

The blueprint requires SLO/criticality, resilience role, workload cycles, utilization dimensions, growth, changes/incidents, licenses, commitments, security/retention, headroom, rollback, and target freshness. Missing required production evidence causes abstention. Infrastructure owners decide and execute through their own change controls.

### Commitment recommendations are financial scenarios

AWS Savings Plans and reservation recommendation APIs and Azure savings-plan/reservation guidance use provider-specific eligibility, lookbacks, terms, coverage, and utilization assumptions. They cannot know the organization's migration probability, concentration policy, liquidity preference, procurement constraints, or full demand scenarios.

The agent therefore produces a no-purchase baseline, existing-commitment context, coverage/utilization, term/payment/flexibility, break-even, vacancy/downside, forecast assumptions, lock-in, expiry, and evidence. The runtime has no purchase API or purchasing credential.

### AI cost requires privileged ingestion and two reconciled views

OpenAI organization usage/cost endpoints and Anthropic administrative usage/cost reporting expose useful direct-vendor evidence. Administrative API keys have broad organization scope and require stronger isolation than a general tool credential.

Provider-reported cost and internal token/request allocation are retained separately. Internal multiplication by a public or contract price is an estimate; provider reports can contain credits, aggregation, delayed adjustments, or dimensions not available to the request path. The system reconciles rather than overwrites either view.

### Identity and version semantics are reconciliation boundaries

Provider resource names are not universal durable identifiers. Azure resource IDs change when a resource moves across a resource group or subscription; Kubernetes names can be reused while UIDs identify object incarnations; Google project names are mutable while project numbers are generated stable identifiers. Recommendations and billing artifacts also have their own lifetimes: Google Recommender couples a recommendation name with state and `etag`, while AWS Cost Optimization Hub recommendation IDs can expire within 24 hours.

The canonical model therefore separates business keys from source incarnation keys and source revisions. Every decision-bearing entity has an explicit identity, version/effective interval, source lineage, and supersession rule. Moves, deletes/recreates, recommendation refreshes, corrected exports, and policy changes produce lineage or a new version rather than silent identity reuse. This applies to billing scopes, exports, line items, resources, services/workloads, tags, allocation rules, budgets, forecasts, anomaly cases, commitments, recommendations, proposals, effects, and savings verifications.

### Cost basis is a named contract, not a display option

AWS documents differences between Bills and Cost Explorer, including amortization, service grouping, timing, rounding, refunds, credits, and taxes. Azure Cost Management generally excludes taxes and some credits and can rerate an open period. Google detailed export emits corrective rows and distinguishes usage time from invoice month. A bare `cost` field therefore cannot support approval, chargeback, or realized-savings claims.

Every monetary aggregate in the blueprint binds a named basis, charge inclusion policy, currency/FX policy, time basis, source and mapping release, open/final state, and correction cutoff. Cash, unblended/list-equivalent, effective/amortized, net, allocated, and invoice/reconciled amounts remain distinct. Credits and refunds are signed charges with classification and applicability; taxes are a separately governed inclusion; commitment fees and negations are preserved rather than flattened. Shared-cost formulas are effective-dated and reconcile to a declared pool. Unit economics includes metric source, denominator quality, zero-denominator behavior, and allocation version, preventing a model from inventing denominators or dividing silently by zero.

### Stateful reliability requires seven explicit memory lifetimes

The guide uses exactly seven memory lifetimes: turn/scratch, working/run, session, durable workflow/task, domain knowledge, long-term/preference, and episodic/outcome. This taxonomy was selected because retention, authorization, poisoning, deletion, and restart requirements differ materially across them. Financial policy and source-of-truth data are not “learned memory”; they stay in governed versioned systems. Preferences cannot override policy, and outcomes cannot become precedent until curated.

A restart-safe receipt records tenant and workflow epoch, event and source high-watermarks, every behavior/evidence version, approval and proposal digests, effect state including `outcome_unknown`, cancellation state, invariants, and the next safe action. Resume validates those fields against authoritative stores before dispatching any effect. Cancellation is cooperative and durable: stop new work, reconcile in-flight effects, emit a receipt, and release the lease. This is what prevents transcript compaction or a worker takeover from losing a financial control boundary.

### Sustainability signals remain a separate governed objective

AWS, Azure, and Google publish carbon/emissions data under different methodologies, lags, scopes, granularity, eligibility, and revision practices. Google states that methodology changes can revise current and historical results; AWS exports expose `model_version`; Azure's dashboard has contract-history and methodology constraints. The Software Carbon Intensity specification answers a different question from provider account-level estimates.

The system preserves provider, methodology/model version, market- or location-based basis, covered scope/services/regions, estimation flags, reporting period, release time, and restatement lineage. Carbon is not converted into currency or combined with savings in a single score unless an approved policy supplies that valuation and the UI exposes it. A proposal reports cost, service risk, and sustainability outcomes as separate dimensions.

### Enterprise and third-party adapters require semantic qualification

OpenTelemetry provides correlation signals, but its semantic conventions have mixed stability and telemetry is not an approval or financial audit ledger. ServiceNow's Identification and Reconciliation Engine has explicit source and authority rules; direct table writes bypass them. Jira issue creation depends on project fields/permissions and is subject to burst, points, and per-issue limits. Backstage supplies organizational catalog relationships, not cloud-provider resource identity.

Commercial FinOps platforms can accelerate ingestion and analysis, but their amortization, FX, allocation, history-restatement, region, edition, and export semantics remain deployment-specific. Flexera documents allocation-rule changes that can reallocate historical data, for example; Finout documents limits in Virtual Tags reallocation behavior. The connector gate therefore captures a real schema/response, permissions, negative authorization, pagination/retry, correction replay, monetary reconciliation, raw-data exit, retention/deletion behavior, and an owner/refresh date. Vendor outputs are evidence, never silent replacements for immutable approval snapshots.

### Durable state and exact approval are necessary

Long-running work spans export delivery, owner review, approval, ticket/change execution, and post-change verification. At-least-once delivery and timeouts make attempt-level deduplication insufficient. The blueprint follows the repository's canonical [state/event](../../runtime/agent-state-and-event-contracts.md), [durable execution](../../runtime/durable-execution.md), and [idempotency](../../reliability/idempotency-and-side-effects.md) guidance: semantic operation identity, intent hash, target precondition, outbox, receipt, `outcome_unknown`, and reconciliation.

An authenticated chat response is not sufficient approval. Approval binds the actor's current scope/role to the exact proposal digest, policy, amount, target version, allowed effect, and expiry.

### Security is dominated by data disclosure and authority composition

NIST access-control, zero-trust, and AI risk guidance supports least privilege, explicit authorization, separation of trust from network location, and governed AI risk. Cost data and model context introduce additional exposure through negotiated prices, resource names, change tickets, and AI attribution.

Provider descriptions, labels, tickets, and retrieved text are untrusted input. A tag that says “ignore policy” has no authority. The model receives minimal tenant-scoped aggregates, never administrative credentials, general SQL, or general cloud-control endpoints.

### Production evolution needs versioned behavior and real outcomes

Production ML systems accumulate data and control dependencies beyond model code, as described in the technical-debt literature. For this agent the behavior release includes connector/schema mapping, allocation, analytical configuration, policy, context builder, prompt/model, tool schemas, and eval corpus.

Model/provider changes are compared offline, shadowed, canaried, and rolled back independently from authority. Continuous evaluation mines production failures into fixtures and measures delayed outcomes: corrected anomaly labels, forecast actuals, verified savings, and service-health impact.

## Contradictions and resolutions

| Tension | Resolution |
|---|---|
| “Use the latest FOCUS” versus provider 1.2/Preview feeds | Use 1.4 as the current reference vocabulary, preserve exact source version, and map only supported semantics |
| Provider dashboard appears current versus documented late/rerated billing | Show source-specific freshness/completeness; do not infer finality from query availability |
| Provider recommendation says “save” versus service owner says risk is unknown | Recommendation is an observation; SLO/security/owner evidence dominates |
| FinOps seeks action versus separation of financial/infrastructure authority | Agent owns evidence and proposal workflow; accountable domains approve/execute |
| Budget threshold implies control versus provider budget features with powerful actions | Enable notifications by default; treat configuration writes as optional F3 and exclude operational enforcement |
| OpenCost allocation versus invoice truth | Use it as a driver/estimate with documented price basis and reconcile to provider billing |
| Token price estimate versus organization cost report | Preserve and reconcile two views; label estimate and provider-reported cost |
| Model memory improves personalization versus financial policy must remain current and reviewable | Put rules/preferences in versioned policy; retain only curated outcome memory with provenance/expiry |
| Trace observability versus audit completeness | Maintain complete domain/effect evidence separately from sampled/redacted telemetry |
| AWS create-new/overwrite exports, Azure daily overwrite/rerating, and Google append-style correction rows | Preserve the provider artifact and manifest, normalize correction/supersession explicitly, and replay source-specific fixtures instead of imposing one ingestion behavior |
| Dashboard cost versus invoice/accounting cost | Bind every aggregate to a cost basis, charge-inclusion policy, time basis, currency/FX policy, source release, and correction cutoff |
| Provider “potential” or “realized” savings versus internally verified outcomes | Preserve the vendor metric name and assumptions; calculate a separate verified outcome with demand, service health, corrections, and coverage |
| Cost reduction versus carbon reduction | Report separate objective dimensions; combine only under an explicit approved valuation policy |
| Vendor allocation configuration can restate history versus immutable approval evidence | Freeze the vendor export, mapping/rule version, and evidence digest used for the decision; a restatement creates a new verification run |

## Designs rejected after research

1. **Autonomous cloud optimization.** Provider signals lack complete organizational context and include destructive/purchase actions.
2. **A general cloud or billing SDK as a model tool.** It makes action scope dependent on prompt/tool generation instead of policy and least privilege.
3. **FOCUS-only storage with raw provider data discarded.** It prevents correction replay and loses extensions/gaps.
4. **One current-cost table overwritten on delivery.** It destroys decision provenance and hides rerating/corrections.
5. **Model-generated monetary calculations or SQL.** It is harder to reproduce, authorize, bound, and evaluate than deterministic code.
6. **Provider recommendation as approval evidence.** Recommendation and accountable decision have different authority and context.
7. **Automatic budget shutdown.** A cost threshold alone is insufficient evidence to disrupt service or change permissions.
8. **Commitment purchase integration.** Procurement/financial authorization is deliberately outside the agent category.
9. **Browser-console automation.** It is fragile, hard to reconcile, and inherits interactive authority.
10. **Unbounded conversation/vector memory.** It creates stale-policy, confidentiality, provenance, injection, and deletion risks.
11. **Provider-specific subagents.** Typed adapters solve source differences with less state and coordination risk.
12. **Potential savings as the primary KPI.** It rewards overclaiming; verified savings with intact SLOs is the meaningful outcome.

## Source register

### FinOps Framework and FOCUS

1. FinOps Foundation, [FinOps Framework](https://www.finops.org/framework/).
2. FinOps Foundation, [FinOps capabilities](https://www.finops.org/framework/capabilities/).
3. FinOps Foundation, [Data ingestion](https://www.finops.org/framework/capabilities/data-ingestion/).
4. FinOps Foundation, [Allocation](https://www.finops.org/framework/capabilities/allocation/).
5. FinOps Foundation, [Anomaly management](https://www.finops.org/framework/capabilities/anomaly-management/).
6. FinOps Foundation, [Forecasting](https://www.finops.org/framework/capabilities/forecasting/).
7. FinOps Foundation, [Budgeting](https://www.finops.org/framework/capabilities/budgeting/).
8. FinOps Foundation, [Rate optimization](https://www.finops.org/framework/capabilities/rate-optimization/).
9. FinOps Foundation, [Usage optimization](https://www.finops.org/framework/capabilities/usage-optimization/).
10. FinOps Foundation, [Reporting and analytics](https://www.finops.org/framework/capabilities/reporting-analytics/).
11. FinOps Foundation, [Invoicing and chargeback](https://www.finops.org/framework/capabilities/invoicing-chargeback/).
12. FinOps Foundation, [FinOps for AI technology category](https://www.finops.org/framework/technology-categories/ai/).
13. FOCUS, [What is FOCUS?](https://focus.finops.org/what-is-focus/).
14. FOCUS, [Specification 1.4](https://focus.finops.org/docs/specification/v1-4/).
15. FOCUS, [1.4 changelog](https://focus.finops.org/docs/specification/v1-4/changelog/).
16. FOCUS, [1.4 features](https://focus.finops.org/docs/specification/v1-4/features/).
17. FOCUS, [Correction handling](https://focus.finops.org/docs/specification/v1-4/attributes/correction-handling/).
18. FOCUS, [Delivery handling](https://focus.finops.org/docs/specification/v1-4/attributes/delivery-handling/).
19. FOCUS, [Schema metadata](https://focus.finops.org/docs/specification/v1-4/sections/metadata/schema/).
20. FOCUS specification repository, [releases](https://github.com/FinOps-Open-Cost-and-Usage-Spec/FOCUS_Spec/releases).

### Amazon Web Services

21. AWS, [What is AWS Data Exports?](https://docs.aws.amazon.com/cur/latest/userguide/what-is-data-exports.html).
22. AWS, [FOCUS 1.2 table dictionary](https://docs.aws.amazon.com/cur/latest/userguide/table-dictionary-focus-1-2-aws.html).
23. AWS, [Creating data exports](https://docs.aws.amazon.com/cur/latest/userguide/dataexports-create.html).
24. AWS Cost Explorer API, [GetCostAndUsage](https://docs.aws.amazon.com/aws-cost-management/latest/APIReference/API_GetCostAndUsage.html).
25. AWS Cost Anomaly Detection API, [GetAnomalyMonitors](https://docs.aws.amazon.com/aws-cost-management/latest/APIReference/API_GetAnomalyMonitors.html).
26. AWS Cost Anomaly Detection API, [GetAnomalies](https://docs.aws.amazon.com/aws-cost-management/latest/APIReference/API_GetAnomalies.html).
27. AWS Cost Explorer API, [GetSavingsPlansPurchaseRecommendation](https://docs.aws.amazon.com/aws-cost-management/latest/APIReference/API_GetSavingsPlansPurchaseRecommendation.html).
28. AWS Cost Explorer API, [GetReservationPurchaseRecommendation](https://docs.aws.amazon.com/aws-cost-management/latest/APIReference/API_GetReservationPurchaseRecommendation.html).
29. AWS Cost Optimization Hub API, [GetRecommendation](https://docs.aws.amazon.com/aws-cost-management/latest/APIReference/API_CostOptimizationHub_GetRecommendation.html).
30. AWS, [What is AWS Compute Optimizer?](https://docs.aws.amazon.com/compute-optimizer/latest/ug/what-is-compute-optimizer.html).
31. AWS Compute Optimizer, [Rightsizing recommendation preferences](https://docs.aws.amazon.com/compute-optimizer/latest/ug/rightsizing-preferences.html).
32. AWS, [Budget actions](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-controls.html).
33. AWS Budgets API, [Action](https://docs.aws.amazon.com/aws-cost-management/latest/APIReference/API_budgets_Action.html).
34. AWS Cost Categories API, [CostCategorySplitChargeRule](https://docs.aws.amazon.com/aws-cost-management/latest/APIReference/API_CostCategorySplitChargeRule.html).
35. AWS, [Billing and Cost Management permissions reference](https://docs.aws.amazon.com/cost-management/latest/userguide/billing-permissions-ref.html).

### Microsoft Azure

36. Microsoft, [Configure FinOps hub scopes](https://learn.microsoft.com/en-us/cloud-computing/finops/toolkit/hubs/configure-scopes).
37. Microsoft, [FinOps hubs overview](https://learn.microsoft.com/en-us/cloud-computing/finops/toolkit/hubs/finops-hubs-overview).
38. Microsoft Azure Advisor, [Cost recommendations](https://learn.microsoft.com/en-us/azure/advisor/advisor-cost-recommendations).
39. Microsoft Azure Advisor, [Cost recommendation reference](https://learn.microsoft.com/en-us/azure/advisor/advisor-reference-cost-recommendations).
40. Microsoft, [Understand Cost Management data](https://learn.microsoft.com/en-us/azure/cost-management-billing/costs/understand-cost-mgt-data).
41. Microsoft, [Cost allocation overview](https://learn.microsoft.com/en-us/azure/cost-management-billing/costs/cost-allocation-introduction).
42. Azure Cost Management REST API, [Create or update allocation rule](https://learn.microsoft.com/en-us/rest/api/cost-management/cost-allocation-rules/create-or-update?view=rest-cost-management-2025-03-01).
43. Azure Cost Management REST API, [Budgets list](https://learn.microsoft.com/en-us/rest/api/cost-management/budgets/list?view=rest-cost-management-2026-06-01).
44. Azure Cost Management REST API, [Alerts list](https://learn.microsoft.com/en-us/rest/api/cost-management/alerts/list?view=rest-cost-management-2026-06-01).
45. Azure Cost Management REST API, [Benefit recommendations list](https://learn.microsoft.com/en-us/rest/api/cost-management/benefit-recommendations/list?view=rest-cost-management-2026-06-01).
46. Microsoft, [Azure savings plan purchase recommendations](https://learn.microsoft.com/en-us/azure/cost-management-billing/savings-plan/purchase-recommendations).
47. Microsoft, [Reservation purchase recommendations](https://learn.microsoft.com/en-us/azure/cost-management-billing/reservations/reserved-instance-purchase-recommendations).
48. Microsoft, [Manage access to Microsoft Cost Management data](https://learn.microsoft.com/en-us/azure/cost-management-billing/costs/assign-access-acm-data).

### Google Cloud

49. Google Cloud, [Export Cloud Billing data to BigQuery](https://docs.cloud.google.com/billing/docs/how-to/export-data-bigquery).
50. Google Cloud, [Set up the FOCUS BigQuery export](https://docs.cloud.google.com/billing/docs/how-to/export-data-bigquery-focus-setup).
51. Google Cloud, [Cloud Billing export data tables](https://docs.cloud.google.com/billing/docs/how-to/export-data-bigquery-tables).
52. Google Cloud, [Detailed usage cost data schema](https://docs.cloud.google.com/billing/docs/how-to/export-data-bigquery-tables/detailed-usage).
53. Google Cloud, [Programmatic budget notifications](https://docs.cloud.google.com/billing/docs/how-to/budgets-programmatic-notifications).
54. Google Cloud, [Create, edit, or delete budgets and budget alerts](https://docs.cloud.google.com/billing/docs/how-to/budgets).
55. Google Cloud, [View and manage cost anomalies](https://docs.cloud.google.com/billing/docs/how-to/manage-anomalies).
56. Google Cloud, [Forecasted costs](https://docs.cloud.google.com/billing/docs/how-to/reports/forecasted-costs).
57. Google Cloud Recommender, [Key concepts](https://docs.cloud.google.com/recommender/docs/key-concepts).
58. Google Cloud, [Committed use discount recommender](https://docs.cloud.google.com/docs/cuds-recommender).
59. Google Cloud, [Analyze committed use discounts](https://docs.cloud.google.com/billing/docs/how-to/cud-analysis).
60. Google Kubernetes Engine, [View GKE costs](https://docs.cloud.google.com/kubernetes-engine/docs/how-to/cost-allocations).
61. Google Cloud, [Overview of Cloud Billing access control](https://cloud.google.com/billing/docs/how-to/billing-access).

### Kubernetes and direct AI costs

62. OpenCost, [Specification](https://opencost.io/docs/specification/).
63. OpenCost, [API and integrations](https://opencost.io/docs/integrations/api/).
64. OpenCost, [GitHub repository](https://github.com/opencost/opencost).
65. OpenAI API, [Organization usage and costs](https://platform.openai.com/docs/api-reference/usage).
66. OpenAI API, [Admin API keys](https://platform.openai.com/docs/api-reference/admin-api-keys).
67. Anthropic API, [Get Messages Usage Report](https://docs.anthropic.com/en/api/admin-api/usage-cost/get-messages-usage-report).
68. Anthropic, [Pricing and usage accounting](https://docs.anthropic.com/en/docs/about-claude/pricing).

### Security, workflow, observability, evaluation, and accountability

69. NIST, [SP 800-53 Rev. 5, Security and Privacy Controls](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final).
70. NIST, [SP 800-207, Zero Trust Architecture](https://csrc.nist.gov/pubs/sp/800/207/final).
71. NIST, [AI Risk Management Framework](https://www.nist.gov/itl/ai-risk-management-framework).
72. NIST, [AI RMF Generative AI Profile](https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.600-1.pdf).
73. Cloud Native Computing Foundation, [CloudEvents specification](https://github.com/cloudevents/spec/blob/main/cloudevents/spec.md).
74. W3C, [Trace Context](https://www.w3.org/TR/trace-context/).
75. Rob J. Hyndman and Anne B. Koehler, [Another look at measures of forecast accuracy](https://doi.org/10.1016/j.ijforecast.2006.03.001).
76. D. Sculley et al., [Hidden Technical Debt in Machine Learning Systems](https://papers.nips.cc/paper/2015/hash/86df7dcfd896fcaf2674f757a2463eba-Abstract.html).
77. U.S. SEC, [Management's Report on Internal Control over Financial Reporting](https://www.sec.gov/rules-regulations/2003/03/managements-report-internal-control-over-financial-reporting-certification-disclosure-exchange-act).
78. PCAOB, [AS 2201: An Audit of Internal Control Over Financial Reporting](https://pcaobus.org/oversight/standards/auditing-standards/details/AS2201).

### Pass 2 provider delivery, identity, optimization, and sustainability sources

79. AWS, [Understanding export delivery](https://docs.aws.amazon.com/cur/latest/userguide/dataexports-export-delivery.html).
80. AWS, [Differences between Billing and Cost Explorer data](https://docs.aws.amazon.com/cost-management/latest/userguide/differences-billing-data-cost-explorer-data.html).
81. AWS, [Line item details](https://docs.aws.amazon.com/cur/latest/userguide/Lineitem-columns.html).
82. AWS, [Cost Optimization Hub overview](https://docs.aws.amazon.com/cost-management/latest/userguide/cost-optimization-hub.html).
83. AWS Cost Optimization Hub API, [GetRecommendation](https://docs.aws.amazon.com/aws-cost-management/latest/APIReference/API_CostOptimizationHub_GetRecommendation.html).
84. AWS, [Viewing Cost Optimization Hub recommendations](https://docs.aws.amazon.com/cost-management/latest/userguide/coh-view-recommendations.html).
85. AWS, [Cost Optimization Hub optimization strategies](https://docs.aws.amazon.com/cost-management/latest/userguide/coh-optimization-strategies.html).
86. AWS, [Reservation recommendations](https://docs.aws.amazon.com/cost-management/latest/userguide/ri-recommendations.html).
87. AWS, [Carbon emissions export columns](https://docs.aws.amazon.com/cur/latest/userguide/carbon-emissions-columns.html).
88. AWS, [Troubleshooting carbon emissions exports](https://docs.aws.amazon.com/cur/latest/userguide/troubleshooting-carbon-emissions.html).
89. AWS, [Sustainability key concepts](https://docs.aws.amazon.com/sustainability/latest/userguide/key-concepts.html).
90. Microsoft, [Create and manage Cost Management exports](https://learn.microsoft.com/en-us/azure/cost-management-billing/costs/tutorial-improved-exports).
91. Microsoft, [FinOps toolkit changelog](https://learn.microsoft.com/en-us/cloud-computing/finops/toolkit/changelog).
92. Microsoft, [FinOps hubs compatibility](https://learn.microsoft.com/en-us/cloud-computing/finops/toolkit/hubs/compatibility).
93. Microsoft Azure Advisor, [Cost recommendations](https://learn.microsoft.com/en-us/azure/advisor/advisor-cost-recommendations).
94. Microsoft, [Azure savings plan purchase recommendations](https://learn.microsoft.com/en-us/azure/cost-management-billing/savings-plan/purchase-recommendations).
95. Microsoft, [Reservation purchase recommendations](https://learn.microsoft.com/en-us/azure/cost-management-billing/reservations/reserved-instance-purchase-recommendations).
96. Microsoft, [Move Azure resources to a new resource group or subscription](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/move-resource-group-and-subscription).
97. Microsoft, [Group and allocate costs using tag inheritance](https://learn.microsoft.com/en-us/azure/cost-management-billing/costs/enable-tag-inheritance).
98. Microsoft, [Connect to the Emissions Impact Dashboard](https://learn.microsoft.com/en-us/power-bi/connect-data/service-connect-to-emissions-impact-dashboard).
99. Google Cloud, [FOCUS cost and usage data schema](https://docs.cloud.google.com/billing/docs/how-to/export-data-bigquery-tables/focus-export).
100. Google Cloud, [Detailed usage cost data schema](https://docs.cloud.google.com/billing/docs/how-to/export-data-bigquery-tables/detailed-usage).
101. Google Cloud, [FinOps hub](https://docs.cloud.google.com/billing/docs/how-to/finops-hub).
102. Google Cloud Recommender, [Key concepts](https://docs.cloud.google.com/recommender/docs/key-concepts).
103. Google Cloud, [Committed use discount recommender](https://docs.cloud.google.com/docs/cuds-recommender).
104. Google Cloud, [Analyze committed use discounts](https://docs.cloud.google.com/billing/docs/how-to/analyze-cuds).
105. Google Cloud, [Create and manage budgets](https://docs.cloud.google.com/billing/docs/how-to/budgets).
106. Google Cloud, [Cloud Billing spend caps (Preview)](https://docs.cloud.google.com/billing/docs/how-to/budgets-spend-caps).
107. Google Cloud, [Forecasted costs](https://docs.cloud.google.com/billing/docs/how-to/reports/forecasted-costs).
108. OpenCost, [OpenCost specification source](https://github.com/opencost/opencost/blob/develop/spec/opencost-specv01.md).
109. OpenCost, [API documentation](https://opencost.io/docs/integrations/api/).
110. Kubernetes, [Object names and IDs](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/).
111. OpenTelemetry, [Specification status](https://opentelemetry.io/docs/specs/status/).
112. OpenTelemetry, [Semantic conventions 1.44.0](https://opentelemetry.io/docs/specs/semconv/).
113. ServiceNow, [Identification and Reconciliation Engine API](https://www.servicenow.com/docs/r/api-reference/rest-apis/c_IdentifyReconcileAPI.html).
114. Atlassian, [Jira Cloud REST API v3 issues](https://developer.atlassian.com/cloud/jira/platform/rest/v3/api-group-issues/).
115. Atlassian, [Jira Cloud rate limiting](https://developer.atlassian.com/cloud/jira/platform/rate-limiting/).
116. Backstage, [The Backstage system model](https://backstage.io/docs/features/software-catalog/system-model/).
117. Vantage, [API documentation](https://docs.vantage.sh/api).
118. Finout, [Virtual Tags API](https://docs.finout.io/configuration/finout-api/virtual-tags-api).
119. Flexera, [Cloud cost allocation rules](https://docs.flexera.com/flexera-one/cloud/using-cloud-cost-optimization/analyzing-cloud-costs-in-billing-centers/allocation-rules).
120. IBM Cloudability, [Getting started with Cloudability v3 API](https://www.ibm.com/docs/en/cloudability-commercial/cloudability-enterprise/saas?topic=api-getting-started-cloudability-v3).
121. Google Cloud, [Carbon Footprint methodology](https://docs.cloud.google.com/carbon-footprint/docs/methodology).
122. Green Software Foundation, [Software Carbon Intensity specification](https://sci.greensoftware.foundation/).

## Limitations

- Public provider documentation changes independently of deployed account capabilities, contracts, regions, preview status, and API versions. Connector tests and effective IAM inspection are still required.
- FOCUS provider support can change after the research date. The compatibility table is a dated baseline, not a support guarantee.
- No organization-specific contract, negotiated pricing agreement, tax rule, accounting policy, service SLO, data-residency requirement, or materiality threshold was available. The blueprint provides decision contracts, not those values.
- Provider anomaly/forecast internals and recommendation algorithms are partly opaque. The design therefore evaluates observed outcomes and preserves provider releases/assumptions where exposed.
- Provider carbon data is estimated, methodology-dependent, incomplete for some services/regions/scopes, delayed, and potentially restated. It is not treated as audited accounting truth or silently merged with currency.
- Preview features, including the provider surfaces named Preview in the status log, remain outside an irreversible production dependency until contract, regional availability, rollback, and change-notice terms are accepted.
- Third-party platform documentation cannot prove the licensed tenant's edition, semantics, rate limits, history-restatement behavior, or raw-data portability. Those properties require connector acceptance evidence.
- Resource moves and delete/recreate cycles can change provider identities. Deployment-specific lineage tests are required before historical attribution is trusted.
- Complete anomaly labels and counterfactual realized-savings truth are inherently limited. Evaluation must report label/verification coverage and uncertainty.
- The source pass establishes architecture and controls; it is not legal, accounting, audit, tax, procurement, or investment advice.
- API endpoint and schema details should be generated from the provider's current specification/SDK at implementation time rather than copied from prose into code.

## Refresh triggers

Re-run the relevant research and compatibility tests when:

- FOCUS publishes or ratifies a new release or conformance changes;
- AWS, Azure, Google Cloud, OpenCost, OpenAI, or Anthropic changes export/API schema, preview/GA status, pagination, retention, or correction behavior;
- provider billing/IAM roles or recommendation/budget actions change;
- recommendation identity, expiry, lookback, eligible product, rate basis, or purchase/cancellation terms change;
- a provider changes a carbon methodology/model, coverage, assurance statement, release cadence, or historical-restatement practice;
- OpenTelemetry convention stability or a CMDB/ITSM/third-party API, reconciliation rule, or rate limit changes;
- a new cloud, SaaS, AI provider, currency, contract, or allocation driver is added;
- a model provider, model identifier, data-processing boundary, or retention policy changes;
- accounting, internal-control, privacy, security, or procurement requirements change;
- anomaly/forecast/optimization outcome drift exceeds a gate;
- an incident reveals a source assumption not represented in fixtures;
- the agent receives a new effect type or proposed authority tier.

## Packet quality check

- [x] Current FOCUS release and provider-version lag were explicitly reconciled.
- [x] Provider delivery, correction, allocation, anomaly, forecast, budget, optimization, commitment, and IAM behavior were researched.
- [x] Kubernetes and direct AI cost sources were included.
- [x] Authority, financial/accounting boundary, security, durability, evaluation, and operations were connected to primary sources and canonical guides.
- [x] Competing designs and provider contradictions were recorded rather than hidden.
- [x] The blueprint uses implementable contracts while avoiding invented provider APIs or a mandatory third-party stack.
- [x] Identity/version semantics cover every decision-bearing entity and delete/move/recreate or supersession behavior.
- [x] Cost basis, amortization, signed charges, taxes, shared cost, and unit-economics denominator quality are explicit.
- [x] Exactly seven memory lifetimes, restart receipts, cancellation, and takeover behavior are specified.
- [x] Current provider, carbon, observability, CMDB/ITSM, and commercial FinOps surfaces carry dated status and qualification limits.
- [x] Worked anomaly and commitment flows plus Stage 0–6 exercises connect research to implementation evidence.
