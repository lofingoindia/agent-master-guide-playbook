# Business Intelligence Monitoring and Decision Operations Agent — Research Packet

**Research date:** 2026-08-31  
**Blueprint:** [Business Intelligence Monitoring and Decision Operations Agent](../../agents/business-intelligence-monitoring-agent/README.md)  
**Method:** [Research and Documentation Method](../research-method.md)  
**Registry category:** Business-intelligence monitoring and decision operations  
**Evidence maturity:** primary-source architecture synthesis with workload-specific recommendations; no public end-to-end benchmark validates the complete agent  
**Next planned review:** volatile product/governance sources by 2026-11-29; stable statistical/provenance sources by 2027-02-27, or sooner on a refresh trigger

## Research question

What is the smallest production architecture that can continuously evaluate governed business metrics, reject unfit data, detect material change, organize evidence, route an accountable decision, survive multi-system failure, and verify follow-through—without replacing analytics, metric ownership, source systems, or human decision authority?

Subquestions:

- What semantic and data-quality evidence makes a repeated metric observation trustworthy?
- Which detection methods are appropriate, and how do benchmarks fail to predict business alert utility?
- Which parts benefit from an LLM and which must remain deterministic?
- How should alert identity, acknowledgement, decision, effect, and outcome be modeled?
- What reliability boundary exists across scheduler, semantic engine, model, notification, ITSM, and humans?
- How do privacy, prompt injection, data rights, multi-tenancy, and human oversight change the design?
- Which native BI/alerting capabilities can avoid building this agent?
- What staged evidence supports promotion from a fixed alert to a production decision-operations loop?

## Category boundary established

The [50-category registry](../agent-blueprint-category-registry.md) assigns persistent metric monitoring, semantic-layer contracts, anomaly/change detection, alert routing, decision thresholds, and follow-through to this category. The adjacent [Analytics Agent](../../agents/analytics-agent/README.md) answers bounded questions and performs governed analysis. The [Executive Operations Agent](../../agents/executive-operations-agent/README.md) owns executive briefing, agenda, cross-domain priority, and commitment coordination; it may consume a finalized monitoring packet but does not own the watch, metric truth, or detector decision. The [Data Pipeline Operations Agent](../../agents/data-pipeline-operations-agent/README.md) owns pipeline and data-product operations.

The blueprint therefore:

- owns watch/evaluation/case/acknowledgement/effect/outcome state;
- consumes metric semantics rather than defining them;
- routes data-health failure to DataOps;
- routes bespoke or causal investigation to Analytics;
- supplies finalized evidence/status to Executive Operations without inferring executive priority or commitment;
- creates only low-risk communication/work-item effects under policy;
- never mutates source data or makes unsupported high-impact decisions.

## Research method and saturation

Research followed the repository funnel:

1. inspected registry, expansion program, research method, adjacent blueprints, and canonical runtime/state/tool/context/memory/security/reliability/evaluation/operations guides;
2. mapped terms across semantic layers, data contracts, data quality, lineage, statistical monitoring, observability alerting, BI alerts, decision rules, ITSM, provenance, event envelopes, privacy, and AI governance;
3. prioritized official specifications, documentation, repositories, government guidance, and papers;
4. compared product mechanics with framework-neutral invariants;
5. searched for benchmark flaws, late data, multiple testing, acknowledgement semantics, effect ambiguity, product limits, and licensing;
6. stopped when additional sources no longer changed the architecture, authority boundary, failure taxonomy, staged gates, or principal caveats.

Saturation is temporary. BI products, model/provider behavior, observability conventions, AI governance, and legal guidance are volatile.

## Evidence classification

| Label | Meaning in this packet |
|---|---|
| Mechanic | Directly documented behavior of a specification, standard, API, or product |
| Observed result | Measurement within a stated paper/dataset/workload |
| Engineering inference | Design conclusion derived from multiple mechanics and canonical runtime constraints |
| Recommendation | Project judgment with stated alternatives and promotion evidence |
| Open question | Requires local evidence or jurisdiction/product-specific review |

## Primary-source inventory

### Semantics, contracts, lineage, and data quality

| Source | Evidence used | Classification | Blueprint influence |
|---|---|---|---|
| [MetricFlow metric semantics](https://github.com/dbt-labs/dbt-core/blob/main/crates/dbt-metricflow/docs/metric-semantics.md) | A metric is a named queryable calculation over semantic models and query scope | Mechanic | Watch references a semantic snapshot; it does not treat a column as the full metric contract |
| [MetricFlow repository](https://github.com/dbt-labs/metricflow) | Semantic query planning supports governed metric queries, joins, ratios, cumulative metrics, and time grains | Mechanic | Typed semantic gateway instead of model-authored SQL |
| [Open Data Contract Standard](https://bitol-io.github.io/open-data-contract-standard/latest/) | Contract structure includes schema, data quality, ownership/roles, SLA, and servers | Mechanic | Data-product contract reference and owner/quality/SLA fields |
| [ODCS repository](https://github.com/bitol-io/open-data-contract-standard) | Current repository metadata, schema, governance, and Apache-2.0 licensing | Mechanic | Interoperability option, not mandatory runtime |
| [OpenLineage facets](https://openlineage.io/docs/spec/facets/) | Versioned facets extend run/job/dataset metadata | Mechanic | Versioned provenance references in observation evidence |
| [OpenLineage data-quality assertions](https://openlineage.io/docs/spec/facets/dataset-facets/data_quality_assertions/) | Assertion result, severity, expected/actual evidence can be represented separately | Mechanic | Quality assertion envelope; local policy decides enforcement |
| [OpenLineage data-quality metrics](https://openlineage.io/docs/spec/facets/dataset-facets/data_quality_metrics/) | Dataset/column metrics and update time can be attached to lineage | Mechanic | Evidence input, not automatic fitness proof |
| [OpenLineage object model](https://openlineage.io/docs/spec/object-model/) | Run, job, and dataset event relationships | Mechanic | Observation provenance and data-incident correlation |
| [UK Government Data Quality Framework](https://www.gov.uk/government/publications/the-government-data-quality-framework/the-government-data-quality-framework) | Quality is contextual; completeness, uniqueness, consistency, timeliness, validity, and accuracy can trade off | Government guidance | Per-watch quality contract; no universal gate |
| [Government of Canada data-quality guidance](https://www.canada.ca/en/government/system/digital-government/digital-government-innovations/information-management/guidance-data-quality.html) | Multiple quality dimensions; context determines selection | Government guidance | Reinforces local purpose/fitness decision |
| [ISO/IEC 25024:2015 catalog entry](https://www.iso.org/standard/35749.html) | Standard defines measurement of data quality | Standard metadata | Notes an established measurement model; normative text is paywalled and was not used to invent requirements |

### Time, provenance, and event interchange

| Source | Evidence used | Classification | Blueprint influence |
|---|---|---|---|
| [Apache Flink streaming analytics](https://nightlies.apache.org/flink/flink-docs-stable/docs/learn-flink/streaming_analytics/) | Event time differs from processing time; watermarks trade latency against completeness; late events can revise results | Mechanic | Separate clocks, correction windows, provisional/revision policy |
| [Flink watermark generation](https://nightlies.apache.org/flink/flink-docs-stable/docs/dev/datastream/event-time/generating_watermarks/) | Watermark strategies and idleness affect event-time progress | Mechanic | Streaming variant must make lateness assumptions explicit |
| [W3C PROV overview](https://www.w3.org/TR/prov-overview/) | Entities, activities, and agents form a provenance model for assessing quality/trust | Standard | Evidence lineage among observations, detector activities, and decision actors |
| [CloudEvents specification](https://github.com/cloudevents/spec) | Portable event envelope with stable core specification | Standard/repository | Transport envelope; application domain schema remains authoritative |
| [OpenTelemetry specification](https://opentelemetry.io/docs/specs/otel/) | Traces, metrics, logs, baggage, and evolving semantic conventions | Standard | Correlated telemetry with application-owned event vocabulary |
| [OpenTelemetry logs](https://opentelemetry.io/docs/specs/otel/logs/) | Log/trace correlation and data model | Standard | Trace-event linkage, not state reconstruction |

### Statistical monitoring and anomaly evaluation

| Source | Evidence used | Classification | Blueprint influence |
|---|---|---|---|
| [NIST/SEMATECH process monitoring](https://www.itl.nist.gov/div898/handbook/pmc/section1/pmc12.htm) | Monitoring compares current behavior with a historical process model; Shewhart, CUSUM, EWMA, and multivariate methods differ | Technical reference | Detector ladder and baseline prerequisites |
| [NIST/SEMATECH EWMA](https://itl.nist.gov/div898/handbook/mpc/section2/mpc2211.htm) | EWMA can be more sensitive to small gradual shifts and assumes representative in-control history | Technical reference | Baseline contamination warning and detector choice |
| [NIST/SEMATECH control-chart rules](https://www.itl.nist.gov/div898/handbook/pmc/section3/pmc32.htm) | Additional run rules improve sensitivity but can substantially raise false alarms | Technical reference | Test alert load; avoid indiscriminate rule stacking |
| [NIST/SEMATECH outlier detection](https://itl.nist.gov/div898/handbook/eda/section3/eda35h.htm) | Outlier rules label values for investigation | Technical reference | Anomaly is not cause/action |
| [Numenta Anomaly Benchmark](https://github.com/numenta/NAB) | Labelled univariate series and a timing-aware scoring application profile | Benchmark/repository | Optional detector sanity check, never product gate |
| [TimeEval](https://github.com/TimeEval/TimeEval) | Reproducible algorithm/dataset evaluation tooling | Benchmark toolkit | Candidate comparison with explicit dataset/version |
| [TimeEval paper](https://vldb.org/pvldb/vol15/p3678-schmidl.pdf) | Large-scale evaluation framework and benchmark methodology | Paper | Benchmark reproducibility and diversity considerations |
| [Current time-series anomaly benchmarks are flawed](https://doi.org/10.1109/TKDE.2021.3112126) | Documents benchmark flaws that can create an illusion of progress | Peer-reviewed paper | Local replay/shadow required; leaderboard claims bounded |
| [Online false discovery rate under local dependence](https://papers.neurips.cc/paper_files/paper/2021/hash/def130d0b67eb38b7a8f4e7121ed432c-Abstract.html) | Sequential multiple testing under dependence requires explicit control assumptions | Paper | Slice/watch explosion warning; no generic transfer claim |

### Alerting, decision rules, and operations

| Source | Evidence used | Classification | Blueprint influence |
|---|---|---|---|
| [Prometheus Alertmanager](https://prometheus.io/docs/alerting/latest/alertmanager/) | Grouping, deduplication, routing, silencing, inhibition, HA behavior | Mechanic | Reuse proven notification mechanics; add separate decision/outcome state |
| [Grafana alerting best practices](https://grafana.com/docs/grafana/latest/alerting/guides/best-practices/) | Actionable alerts, ownership, pending periods, grouping, flapping controls, post-incident review | Product guidance | Actionability and alert-fatigue controls |
| [Grafana recovery thresholds](https://grafana.com/docs/grafana/latest/alerting/fundamentals/alert-rules/queries-conditions/) | Recovery thresholds/hysteresis can reduce flapping | Mechanic | Separate trigger and recovery |
| [Google SRE Workbook: Alerting on SLOs](https://sre.google/workbook/alerting-on-slos/) | Multi-window burn-rate patterns and low-traffic trade-offs | Production guidance | Portfolio service SLOs; cautious transfer to sparse business watches |
| [OMG DMN 1.5](https://www.omg.org/spec/DMN/1.5/About-DMN) | Normative decision-model and decision-table notation | Standard | Optional deterministic decision-table representation |
| [ServiceNow operator acknowledge/analyze](https://www.servicenow.com/docs/r/it-operations-management/event-management/operator-phase-acknowledge-analyze.html) | Acknowledgement indicates alert awareness/triage; it does not itself assign or create a task | Product mechanic | Separate acknowledgement, assignment, and action |
| [ServiceNow operator process](https://www.servicenow.com/docs/r/xanadu/it-operations-management/event-management/operator-process.html) | Acknowledge/analyze, triage/action, verify/close phases | Product mechanic | Case lifecycle comparison; vocabulary remains local |
| [ServiceNow alert lifecycle practice](https://www.servicenow.com/docs/r/it-operations-management/event-management/r_EMBestPractice.html) | Message keys, duplicate/open/close/reopen/flapping and incident links | Product guidance | Stable keys and explicit lifecycle states |

### Native BI and event-action alternatives

| Source | Current mechanic observed at research date | Blueprint interpretation |
|---|---|---|
| [Looker creating alerts](https://docs.cloud.google.com/looker/docs/creating-alerts) | Alerts are configured from dashboard tiles with conditions, recipients, permissions, and API support; documentation is actively updated | Strong native option when tile/permission/lifecycle semantics suffice |
| [Looker create alert API](https://docs.cloud.google.com/looker/docs/reference/looker-api/latest/methods/Alert/create_alert) | API exposes alert condition, schedule, destinations, and ownership fields | Verify minimum cadence, edit identity, delivery, and reconciliation locally |
| [Tableau Pulse alerts](https://help.tableau.com/current/online/en-us/pulse_alerts.htm) | Metric alerts include threshold/trend-style behavior and digest/badge presentation | Useful metric consumption; not assumed to be a durable cross-system decision ledger |
| [Tableau Pulse metric creation](https://help.tableau.com/current/online/en-us/pulse_create_metrics.htm) | Metrics are configured from governed data definitions and dimensions | Semantic/watch governance still needs local control |
| [Power BI data alerts](https://learn.microsoft.com/en-us/power-bi/create-reports/service-set-data-alerts) | Alerts depend on refreshed dashboard data, supported visual types, user ownership, and optional automation integration | Suitable for personal/simple threshold alerts; test enterprise ownership and persistence |
| [Fabric Activator overview](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/data-activator/activator-introduction) | Stateful event rules and actions integrate with Fabric/Power Automate | Potential event-rule substrate; not automatically the full approval/effect/outcome controller |
| [Fabric Activator limitations](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/data-activator/activator-limitations) | Documented throughput/rule/lifecycle and feature-maturity limits change over time | Verify at deployment; enforce platform-independent state/effect invariants |
| [Fabric Activator rule creation](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/data-activator/activator-create-activators) | Rule changes/deletions and queued monitoring/actions can have delayed behavior | Disabling a rule is not proof all in-flight effects stopped |

### Adapter and provider contract evidence

| Source | Current mechanic observed at research date | Contract consequence and limit |
|---|---|---|
| [MetricFlow releases](https://github.com/dbt-labs/metricflow/releases) | Release history includes breaking API/compatibility changes; `0.210.0` was published in April 2026 | Pin exact package/hosted observation, semantic manifest, warehouse dialect, and golden query tests; a marketing product name is insufficient |
| [BigQuery `jobs.query`](https://cloud.google.com/bigquery/docs/reference/rest/v2/jobs/query) | Exposes job/query identity, completion, pagination, request ID, dry run, bytes processed/billed, and maximum-byte control | First response can be incomplete; dry-run estimates and external-source cost have documented limits; consume all pages and record final cost |
| [Snowflake query history](https://docs.snowflake.com/en/sql-reference/functions/query_history) | Exposes unique `query_id`, `query_tag`, execution status, role/session/warehouse and runtime/cost fields | Tag is correlation, not idempotency; history visibility, lag and retention require role-specific tests |
| [Databricks SQL Statement Execution API tutorial](https://docs.databricks.com/aws/en/dev-tools/sql-execution-tutorial) | API 2.0 uses an asynchronous statement ID and chunked results; default waiting can return a pending state | Poll terminal state and verify every result chunk; pin cloud/region/client and disposition behavior |
| [OpenLineage run cycle](https://openlineage.io/docs/spec/run-cycle/) and [OpenAPI](https://openlineage.io/apidocs/openapi/) | Versioned run events and schema URLs distinguish start/running/terminal states | Event presence is evidence, not proof of complete lineage or correct/fresh data; test missing, late, duplicate and reordered events |
| [Airflow asset scheduling](https://airflow.apache.org/docs/apache-airflow/stable/authoring-and-scheduling/asset-scheduling.html) and [backfill](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/backfill.html) | `3.3.x` docs expose asset/event scheduling, queued asset events, and explicit backfill reprocessing/concurrency | Asset/DAG success is only readiness input; preserve logical interval, run, source event, and reprocessing identity separately |
| [Jira Cloud REST API v3 issues](https://developer.atlassian.com/cloud/jira/platform/rest/v3/api-group-issues/) | Issue creation is rights/field-schema dependent; REST v3 returns a remote issue and exposes history, while metadata surfaces evolve | Discover fields under the service principal immediately before effects; qualify operation-marker lookup and eventual visibility |
| [Slack `chat.postMessage`](https://api.slack.com/methods/chat.postMessage) | Success returns channel and timestamp ID; membership/scopes and per-channel/workspace rate behavior apply | Store returned remote form/ID and respect `Retry-After`; no deduplication guarantee is assumed |
| [Microsoft Graph `sendMail`](https://learn.microsoft.com/en-us/graph/api/user-sendmail?view=graph-rest-1.0) | Success is `202 Accepted`; delivery remains subject to Exchange limits/throttling | Acceptance is not delivered/read; without a qualified trace/search identity, retain unknown outcome and restrict authority |
| [PagerDuty event management](https://support.pagerduty.com/main/docs/event-management) | Events API v2 groups matching case-sensitive `dedup_key` values | Build a tenant/case/route key, then separately qualify incident lookup, orchestration, suppression, resolve and key lifecycle |

Provider documentation records intended mechanics, not a contractual guarantee for every plan, tenant, region, configuration, or future release. The blueprint therefore requires a dated adapter dossier, service-principal rights matrix, conformance/fault report, behavior-bundle pin, and refresh trigger.

### Security, privacy, governance, and human oversight

| Source | Evidence used | Classification | Blueprint influence |
|---|---|---|---|
| [OWASP AI Agent Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/AI_Agent_Security_Cheat_Sheet.html) | Least privilege, tool validation, memory and prompt-injection risks, human oversight | Security guidance | No raw shell/SQL/HTTP; untrusted data labels; effect controls |
| [NIST AI RMF](https://www.nist.gov/itl/ai-risk-management-framework) | Governance, mapping, measurement, management, and accountable human roles; framework is under revision | Government framework | Risk-based staged gates and named accountability |
| [NIST AI RMF Core](https://airc.nist.gov/airmf-resources/airmf/5-sec-core/) | Organizational roles, oversight, accountability, and risk management outcomes | Government framework | Human authority and release governance |
| [NIST Privacy Framework](https://www.nist.gov/privacy-framework) | Voluntary privacy-risk management approach | Government framework | Purpose, data processing, control, and review structure |
| [GDPR official text](https://eur-lex.europa.eu/eli/reg/2016/679/oj) | EU data-protection obligations where applicable | Law | Purpose, minimization, accuracy, retention, security, rights review |
| [European Commission GDPR principles](https://commission.europa.eu/law/law-topic/data-protection/information-business-and-organisations/principles-gdpr_en) | Plain-language official principles | Government guidance | Design checklist, jurisdiction conditional |
| [EU AI Act official text](https://eur-lex.europa.eu/eli/reg/2024/1689/oj?locale=en) | Risk-based obligations including logs and human oversight for covered high-risk systems | Law | Consequential use requires scoped legal classification and controls |
| [ICO AI and data-protection rights guidance](https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/artificial-intelligence/guidance-on-ai-and-data-protection/how-do-we-ensure-individual-rights-in-our-ai-systems/) | Human review should be active and capable of challenge rather than rubber-stamping | Jurisdiction-specific guidance | Approval UI, capacity, competence, and challengeability |
| [Canada Directive on Automated Decision-Making](https://www.tbs-sct.canada.ca/pol/doc-eng.aspx?id=32592) | Impact assessment and human involvement requirements for in-scope federal administrative systems | Jurisdiction-specific directive | Do not generalize; illustrates higher control with higher impact |

## Claims and resulting design decisions

| Evidence-backed claim | Design decision | Strength/limit |
|---|---|---|
| Metric meaning is a governed semantic calculation, not merely a stored value | Pin metric and semantic snapshot in every evaluation | Strong mechanic; semantic engines differ |
| Data quality is fitness-for-purpose and dimensions can trade off | Watch owns explicit freshness/quality enforcement policy | Strong cross-government guidance; thresholds remain local |
| Event-time completeness is uncertain under late data | Store multiple clocks, watermark/completion evidence, correction window, and revisions | Strong streaming mechanic; batch products expose different metadata |
| Extra statistical rules can raise false alarms | Use simplest detector, measure event-level alert load, require persistence/materiality | Strong statistical reference; exact performance is workload-specific |
| Outliers and anomalies do not identify cause | Model emits hypotheses with citations and no causal authority | Strong conceptual inference; causal analysis is separate |
| Alert managers excel at group/dedup/route but not business outcome state | Reuse them for delivery and maintain a separate case/effect/outcome controller | Engineering inference; product extensions may cover more |
| Acknowledgement is not assignment, action, resolution, or outcome | Model separate authenticated lifecycle states | Strong product/workflow comparison; vocabulary is local |
| Remote writes can commit before response is lost | Intent-first effect ledger, unknown outcome, reconciliation before retry | Canonical distributed-systems invariant; provider lookup quality varies |
| Model/data/tool inputs can be adversarial | Compile minimum context, label trust, validate citations/tools, separate authority | Strong security guidance; no single control eliminates injection |
| Human oversight can become nominal under overload | Alert budgets and meaningful review are release gates | Strong governance/human-factors inference; requires local measurement |
| Public anomaly benchmarks omit business semantics/actions | Local historical replay, shadow, and owner scorecards are mandatory | Strong benchmark critique; local labels can also be biased |
| Built-in BI/event products cover many simple cases | Stage 0 must reconsider a native fixed alert before agent work | Product mechanics change; procurement testing required |

## Key disagreements and reconciliations

### “Data freshness” versus “data quality”

Some product interfaces treat freshness as one quality check; operational teams often treat it as an SLO. The blueprint models freshness evidence explicitly and includes it in the watch's fitness gate. This supports both interpretations without hiding the stop condition.

### “Anomaly” versus “alert”

Statistical literature labels departures from a model; alerting systems prioritize actionable operator attention. The blueprint creates a signal only after deterministic detection, then a case only after materiality, correlation, and policy. This prevents detector output from inheriting business authority.

### Static thresholds versus adaptive detection

Static thresholds align directly with known business limits but can miss context. Adaptive baselines can handle seasonality but introduce contamination, drift, and reproducibility risks. Recommendation: use the least complex detector that meets owner-labelled goals; require versioned baselines and shadow promotion for adaptive methods.

### Alert manager versus decision-operations system

Observability tooling may provide grouping, silencing, routing, notification, and alert state. ITSM adds assignment and workflow. BI products add semantic context. Rather than declare one product insufficient universally, the blueprint defines required invariants. Buy/reuse any component that meets them; build only missing durable case/effect/outcome control.

### LLM-driven investigation versus deterministic workflow

Models are flexible in evidence organization but unreliable as authorities and can amplify prompt injection. Recommendation: keep topology, gates, detection, policy, state, and effects deterministic. Use at most a bounded read-only triage worker. If the evidence path is fixed, omit the model.

### Auto-calibration versus owner governance

Outcome feedback may suggest threshold changes, but outcomes are delayed, confounded, strategically labelled, and affected by interventions. Recommendation: generate offline candidates; require replay, shadow, and owner-approved version promotion. Never self-modify production thresholds or authority.

### Service SLO burn rates versus sparse business metrics

Multi-window burn alerts are well-founded for sufficiently active service-level indicators. Sparse daily/weekly business watches may not produce stable burn statistics. Use portfolio-level service SLOs and per-watch deadlines/materiality; validate any burn-rate transfer locally.

## Benchmark and measurement limits

No reviewed source provides a representative end-to-end benchmark covering:

- governed metric semantics and semantic changes;
- data quality, late data, and backfills;
- local materiality and owner capacity;
- alert grouping, acknowledgement, and decision policy;
- model evidence triage and prompt injection;
- external-effect ambiguity and reconciliation;
- business action and verified realized outcome;
- privacy, multi-tenancy, and jurisdiction.

Consequences:

- detector leaderboards cannot set production thresholds;
- point-wise F1 can over-reward repeated alarms within one event;
- synthetic anomalies may not resemble material business changes;
- curated labels can leak future knowledge or ignore data outages;
- human actionability labels vary by capacity, policy, and available interventions;
- post-action metric change cannot establish causal value.

The roadmap therefore requires local event-level replay, repeated trajectory evaluation, fault/adversarial injection, shadow, canary, and production outcome review.

## Product maturity and drift

| Area | Snapshot at 2026-08-31 | Refresh risk |
|---|---|---|
| Open Data Contract Standard | Repository/docs expose current 3.x standard and Apache-2.0 license | Schema/version can advance; pin used version |
| OpenLineage | Active versioned specification and facets | Facet versions and client support vary |
| CloudEvents | Stable 1.0.x core; broader ecosystem continues | Transport support differs |
| OpenTelemetry | Active specification with evolving GenAI conventions | High; keep application-owned schema |
| DMN | Version 1.5 adopted in 2024 | Tool conformance/support differs |
| Metric/query/event adapters | MetricFlow `0.210.0`, Airflow `3.3.x`, CloudEvents `1.0.2`, and continuously delivered warehouse/SaaS APIs were observed | Very high; pin deployed versions or dated fingerprints and rerun conformance |
| NIST AI RMF | RMF 1.0 is being revised | High for governance references |
| Looker/Tableau/Power BI/Fabric | Active product documentation and frequent changes | Very high; verify cadence, permissions, API, limits, and preview/GA |
| Model providers | Model aliases, retention, region, structured output, and quotas change | Very high; behavior-bundle manifest and canary |
| Privacy/AI law and guidance | Jurisdiction- and use-case-specific, with phased/ongoing change | Very high; counsel review |

Do not copy the source version into a permanent guarantee. Record the version actually deployed and its contract tests.

## Licensing and implementation caveats

Verified repository licenses at research date:

| Component/reference implementation | License observed | Caveat |
|---|---|---|
| [MetricFlow](https://github.com/dbt-labs/metricflow/blob/main/LICENSE) | Apache License 2.0 | Current repository license; dependencies/hosted services differ |
| [Open Data Contract Standard](https://github.com/bitol-io/open-data-contract-standard/blob/main/LICENSE) | Apache License 2.0 | Standard content and implementations may have separate notices |
| [OpenLineage](https://github.com/OpenLineage/OpenLineage/blob/main/LICENSE) | Apache License 2.0 | Integrations have their own dependencies |
| [CloudEvents](https://github.com/cloudevents/spec/blob/main/LICENSE) | Apache License 2.0 | SDK licenses should be checked separately |
| [Prometheus Alertmanager](https://github.com/prometheus/alertmanager/blob/main/LICENSE) | Apache License 2.0 | Managed offerings have commercial terms |
| [Numenta Anomaly Benchmark](https://github.com/numenta/NAB/blob/master/LICENSE.txt) | MIT | Dataset source/usage and benchmark fitness still require review |
| [TimeEval](https://github.com/TimeEval/TimeEval/blob/main/LICENSE) | MIT | Bundled/linked algorithms and datasets can have separate licenses |
| [Grafana](https://github.com/grafana/grafana/blob/main/LICENSING.md) | AGPL-3.0-only by default with documented exceptions | Networked/modified/distributed use requires organization-specific legal review |

Standards availability is not implementation permission. SaaS terms, model-provider data terms, connectors, datasets, fonts/assets, and transitive dependencies require separate review. This is an engineering inventory, not legal advice.

## Jurisdiction and governance limitations

- GDPR and EU AI Act requirements apply only when scope criteria are met; other jurisdictions differ.
- The Canadian Directive source applies to in-scope federal administrative decision systems, not all organizations.
- ICO guidance is UK-specific and can change with legislation and regulatory updates.
- Employment, financial, insurance, healthcare, competition, consumer-protection, records, and sector rules may be more restrictive than general AI/privacy frameworks.
- A business metric can expose sensitive or inferred information even when direct identifiers are absent.
- A human approval is not automatically meaningful oversight if authority, competence, time, information, or ability to reject is missing.
- Legal basis, controller/processor roles, automated-decision classification, notification, retention, and cross-border processing require local counsel.

The blueprint therefore defaults to aggregate evidence, structural purpose/rights controls, model processing disabled for sensitive individual data, and human ownership of consequential decisions.

## Data-quality limitations

- Accuracy often cannot be proved from the same system that produced the value.
- Completeness depends on the expected population, which can itself drift.
- Timeliness can conflict with completeness and accuracy.
- A passing schema/assertion suite does not prove business fitness.
- Lineage can be incomplete or delayed and must have its own freshness.
- No-data, zero, partial, late, and deleted are distinct states.
- Backfills can revise both alerts and detector training data.
- Reference data such as targets, currency, calendars, and hierarchies can be stale independently.

The data gate is therefore a versioned decision with evidence and `accepted/stale/invalid/partial/indeterminate` outcomes, not a single green badge.

## Architecture decisions

| Decision | Selected option | Alternatives rejected or deferred | Change signal |
|---|---|---|---|
| Runtime | Deterministic controller with bounded model worker | Open-ended agent loop | Material measured benefit from additional semantic planning |
| Initial deployment | Modular monolith plus isolated effect/model workers | Microservices/multi-agent system | Proven independent scaling, isolation, or ownership need |
| Metric access | Typed semantic-layer API | Free-form model SQL | No change; raw SQL remains outside model authority |
| State store | Relational authoritative case/effect state plus artifacts | Logs, chat, or BI provider as workflow truth | Provider offers equivalent portable/versioned guarantees and migration path |
| Detection | Complexity ladder starting with owner rule | Default learned anomaly model | Local replay proves a more complex method improves utility |
| Effects | Notifications and work items only | Source-system action | Separate governed system and authority decision; still not default |
| Memory | Durable state and reviewed outcome episodes | Conversation/personal/automatic memory | Explicit governed use case with safety/value evidence |
| Feedback | Offline candidates and reviewed releases | Online self-tuning | No planned change for authority-bearing parameters |
| Scaling | Workload queues, then tenant/cell partitioning | Early distributed fleet | Measured load, residency, or blast-radius requirement |

## Open questions requiring local evidence

- Which recurring decisions are material and truly actionable?
- What are acceptable false-alert, missed-event, and detection-delay costs per watch?
- Which semantic engine exposes stable snapshots and exact generated-query evidence?
- How accurately can data readiness and correction windows be estimated?
- Can notification and ITSM providers search reliably by operation key after timeouts?
- Which destinations support authenticated acknowledgement rather than read receipt?
- Which historical labels are sufficiently reliable and temporally clean?
- What outcome can be independently verified, and what attribution claim is defensible?
- Which model/provider/region is allowed for each data class?
- What owner/channel capacity makes human oversight meaningful?
- Do native BI or event-rule capabilities already satisfy the required invariants?

## Research-to-guide traceability

| Guide | Main evidence families |
|---|---|
| [README](../../agents/business-intelligence-monitoring-agent/README.md) | Registry boundary, canonical runtime, all source families |
| [01 — Mission and architecture](../../agents/business-intelligence-monitoring-agent/01-mission-boundary-and-architecture.md) | BI alternatives, Alertmanager, ITSM, canonical execution boundaries |
| [02 — Contracts](../../agents/business-intelligence-monitoring-agent/02-watch-semantic-freshness-and-quality-contracts.md) | MetricFlow, ODCS, OpenLineage, government data-quality guidance, Flink |
| [03 — Detection and decisions](../../agents/business-intelligence-monitoring-agent/03-detection-triage-routing-and-decision-workflows.md) | NIST/SEMATECH, multiple testing, Alertmanager/Grafana, DMN, ITSM |
| [04 — State and context](../../agents/business-intelligence-monitoring-agent/04-state-events-effects-memory-and-context.md) | Canonical state/memory/context, W3C PROV, CloudEvents |
| [05 — Tools and security](../../agents/business-intelligence-monitoring-agent/05-tools-integrations-security-and-governance.md) | OWASP, NIST, GDPR/EU AI Act/ICO/Canada, licenses |
| [06 — Reliability](../../agents/business-intelligence-monitoring-agent/06-reliability-idempotency-and-reconciliation.md) | Canonical durable execution/idempotency/queues, product effect-lifecycle mechanics |
| [07 — Evaluation](../../agents/business-intelligence-monitoring-agent/07-evaluation-observability-and-failure-injection.md) | NAB, TimeEval, benchmark critique, OTel, SRE |
| [08 — Operations](../../agents/business-intelligence-monitoring-agent/08-deployment-scale-incidents-cost-and-evolution.md) | Canonical operations plus Looker/Tableau/Power BI/Fabric maturity |
| [09 — Roadmap](../../agents/business-intelligence-monitoring-agent/09-zero-to-production-roadmap.md) | Registry expansion stage contract plus all evidence gates |
| [10 — Adapter qualification](../../agents/business-intelligence-monitoring-agent/10-adapter-qualification-and-integration-playbooks.md) | MetricFlow, warehouse query APIs, OpenLineage, Airflow/CloudEvents, Jira, Slack, Graph and PagerDuty primary mechanics plus application-owned conformance gates |

## Residual limitations

1. No live organization, semantic layer, metric corpus, incident history, or provider account was available; all thresholds and SLO examples are illustrative.
2. External links were reviewed at the research date, but SaaS documentation can change without stable versioned URLs.
3. The blueprint provides representative adapter playbooks but had no live provider accounts; exact schemas, plans, rights, consistency windows, quotas, delivery traces, and reconciliation paths still require tenant-specific tests.
4. Public statistical sources do not establish business actionability or causal value.
5. Human-factors guidance cannot predict local alert fatigue or organizational accountability.
6. Legal/licensing summaries are not legal advice.
7. OpenTelemetry GenAI conventions and AI-governance materials are evolving; the application schema intentionally remains independent.
8. Product documentation describes intended mechanics, not availability or correctness under every tenant/region/plan.

## Refresh triggers

Refresh immediately when:

- the category registry or canonical runtime/state/security contracts change;
- a semantic layer, metric definition, data contract, calendar, or lineage interface changes;
- a detector or baseline library changes behavior;
- a model, prompt, context compiler, provider retention policy, region, or tool contract changes;
- a BI/alerting/ITSM platform changes alert identity, cadence, limits, permissions, idempotency, or lifecycle;
- NIST AI RMF revision, OpenTelemetry conventions, DMN, ODCS, OpenLineage, or CloudEvents changes materially;
- a relevant privacy/AI/sector law or regulator guidance changes;
- a production incident reveals a new failure class;
- alert actionability, owner capacity, false/missed events, cost, or outcomes drift;
- a dependency license or SaaS term changes.

## Packet quality check

- [x] Taxonomy, registry, adjacent blueprints, and canonical guides were inspected.
- [x] Primary official sources span semantics, quality, lineage, time, detection, alerting, decisions, BI products, provenance, security, privacy, and operations.
- [x] Important claims are classified and bounded.
- [x] Product mechanics are not presented as universal guarantees.
- [x] Contradictions and conditional recommendations are documented.
- [x] Benchmark, maturity, licensing, jurisdiction, and data-quality limits are explicit.
- [x] Every guide has source traceability.
- [x] Residual uncertainty and refresh triggers are recorded.
