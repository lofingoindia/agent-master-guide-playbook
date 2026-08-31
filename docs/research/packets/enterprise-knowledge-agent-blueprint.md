# Enterprise knowledge agent blueprint — research packet

## Packet metadata

| Field | Value |
|---|---|
| Research baseline | 2026-08-31 |
| Pass 2 refresh | 2026-08-31 — connector qualification, explicit memory/state/effect contracts, failure injection, incidents, and governed behavior evolution |
| Scope | Production enterprise knowledge and company-research agents |
| Deliverable | `docs/agents/enterprise-knowledge-agent/` |
| Evidence preference | Standards, official documentation/repositories, peer-reviewed papers, then engineering reports |
| Intended use | Trace important blueprint decisions to research and support future refreshes |

This packet is evidence for the guide set, not a claim that every linked product or technique should be deployed. Product documentation establishes supported behavior but can understate failure modes; research benchmarks establish behavior on their datasets but do not establish production fitness; vendor engineering reports are useful implementation evidence but require independent validation.

## Research questions

The research covered these questions:

1. When should a knowledge product be search, fixed RAG, a durable workflow, an agent loop, or graph-assisted retrieval?
2. How do production connectors converge content, permissions, moves, deletions, and derived indexes?
3. Where must authorization be enforced, and how can search or graph systems leak existence?
4. How should hybrid retrieval, reranking, decomposition, and multi-hop search interact?
5. What evidence model supports citations, temporal truth, contradictions, calculations, and reproducibility?
6. How should finite context, compaction, run state, and memory be separated?
7. How can untrusted retrieved content be prevented from controlling tools or exfiltrating data?
8. What controls are needed for tenant isolation, retention, deletion, legal hold, residency, and audit?
9. What recovery, telemetry, evaluation, latency, cost, and deployment practices make the service operable?
10. Which current specifications or vendor capabilities are unstable enough to require a dated baseline?

## Research method

- Started from standards and first-party documentation for authorization, source change APIs, provenance, security, telemetry, and retention.
- Used original papers and official repositories for retrieval and evaluation methods.
- Cross-checked vendor-specific retrieval and authorization patterns across Microsoft, Google, Elastic, PostgreSQL, and vector-database documentation.
- Compared agent advocacy with explicit simplicity guidance and benchmark limitations.
- Traced important source-sync edge cases to the source API rather than generic connector tutorials.
- Recorded current version/date facts where behavior is likely to change.
- Excluded unsupported universal performance claims, vendor leaderboard conclusions, and architecture recommendations based only on demos.

Research stopped when additional sources repeated established mechanisms without materially changing the decisions below. The packet still identifies empirical questions that can only be answered against the target organization’s corpora and policy model.

## Decision summary

| Decision | Research synthesis |
|---|---|
| Default product shape | Authorized hybrid search plus evidence-grounded synthesis; keep agentic investigation as a bounded route |
| Non-agent alternative | Prefer ordinary search, analytics, or deterministic workflow when the task and evidence path are known |
| Authorization | Enforce below the model at query time, candidate/citation use, and action execution; fail closed |
| Ingestion | Versioned replication with full scan, delta sync, reconciliation, tombstones, and immutable lineage |
| Connector admission | Source-specific capability card and failure test; exclude or narrow sources whose authorization/deletion semantics cannot meet policy |
| Retrieval | Lexical + vector fusion, metadata filters, optional reranking; graph only for measured relationship/global gaps |
| Evidence | Claim-level ledger with exact source version/span, time semantics, authorization, and derivation |
| Contradictions | Preserve alternatives and explain scope/date/definition differences; do not average facts |
| Context | Compact into evidence references, decisions, open slots, and budgets; never treat summaries as new evidence |
| Memory | Treat turn/working/session context, workflow state, domain truth, preferences, episodes, learned assertions, and caches as different lifecycles; reject implicit long-term writes |
| Tools | Narrow schemas behind a policy gateway; separate reads, drafts, commits, and destructive actions |
| Approval | Signed, expiring approval bound to canonical action parameters and idempotency key |
| Reliability | Durable checkpoints, bounded retries, safe degraded modes, and no degraded authorization |
| Evaluation | Component and end-to-end evaluation with deterministic security gates and task-sliced quality metrics |
| Deployment | Version every material layer, shadow incompatible indexes, canary, and retain tested rollback |
| Evolution | Release a complete behavior bundle through held-out eval, shadow, canary, drift monitoring, incident replay, and reversible rollout; no runtime self-modification |

## Evidence map for major claims

### Architecture and agent scope

| Claim used in blueprint | Strongest evidence | Interpretation |
|---|---|---|
| Use the simplest architecture that satisfies the task | [Anthropic, Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) | The engineering guidance explicitly distinguishes workflows from agents and recommends starting simple; accepted as a design principle, not a benchmark |
| Decomposition and iterative evidence gathering can improve complex enterprise RAG | [Google Research, dependable agentic RAG](https://research.google/blog/unlocking-dependable-responses-with-gemini-enterprise-agent-platforms-agentic-rag/), [IRCoT](https://aclanthology.org/2023.acl-long.557/) | Supports a bounded investigate route; Google’s percentage improvements remain vendor/dataset-specific |
| Self-reflection/correction can improve selected RAG tasks | [Self-RAG](https://openreview.net/forum?id=hSyW5go0v8), [CRAG](https://arxiv.org/abs/2401.15884) | Useful research direction, not justification for an unbounded production loop |
| Graph retrieval can support corpus-wide and relationship questions | [GraphRAG query overview](https://microsoft.github.io/graphrag/query/overview/), [GraphRAG paper](https://arxiv.org/abs/2404.16130) | Include graph as an optional route, with ACL/time lineage and measured adoption gates |
| Graph indexing has substantial cost/complexity trade-offs | [GraphRAG indexing methods](https://microsoft.github.io/graphrag/index/methods/) | The official implementation documents standard versus fast extraction and its cost concentration; do not make graph the default |

### Retrieval and ranking

| Claim used in blueprint | Strongest evidence | Interpretation |
|---|---|---|
| Lexical and vector retrieval are complementary | [Azure hybrid search](https://learn.microsoft.com/en-us/azure/search/hybrid-search-overview), [BEIR](https://github.com/beir-cellar/beir) | Use query-class evaluation; neither representation wins universally |
| Reciprocal-rank fusion is a practical score-independent fusion method | [Azure RRF](https://learn.microsoft.com/en-us/azure/search/hybrid-search-ranking), [Elasticsearch RRF](https://www.elastic.co/docs/reference/elasticsearch/rest-apis/reciprocal-rank-fusion) | Suitable baseline before learned fusion; vendor defaults are not universal tuning values |
| Late interaction and learned sparse retrieval are alternatives for hard domains | [ColBERT](https://github.com/stanford-futuredata/ColBERT), [SPLADE](https://github.com/naver/splade) | Put behind an evaluation gate because serving/storage trade-offs differ |
| Long context does not guarantee evidence use | [Lost in the Middle](https://aclanthology.org/2024.tacl-1.9/) | Retrieve and structure evidence deliberately; do not solve retrieval by filling the window |

### Authorization and tenant isolation

| Claim used in blueprint | Strongest evidence | Interpretation |
|---|---|---|
| Relationship-based authorization can model large nested permission graphs | [Zanzibar paper](https://research.google/pubs/zanzibar-googles-consistent-global-authorization-system/) | Useful conceptual base; implementation choice remains organization-specific |
| Zero trust requires per-request identity/resource policy rather than network trust | [NIST SP 800-207](https://csrc.nist.gov/pubs/sp/800/207/final) | Supports explicit authentication, authorization, least privilege, and continuous policy context |
| Query-time ACL filters are supported but have representation limits | [Azure query-time ACL enforcement](https://learn.microsoft.com/en-us/azure/search/search-query-access-control-rbac-enforcement), [Google data-source access control](https://docs.cloud.google.com/generative-ai-app-builder/docs/data-source-access-control) | Preserve source ACL semantics and test provider limits before selecting an engine |
| Role composition and aggregations can create disclosure surprises | [Elasticsearch document-level security](https://www.elastic.co/guide/en/elasticsearch/reference/current/document-level-security.html) | Do not assume document filters behave like row-level security in every query path |
| Database row-level policy can fail closed by default | [PostgreSQL row security](https://www.postgresql.org/docs/current/ddl-rowsecurity.html) | Good control for authoritative metadata, not a substitute for search-engine enforcement |
| Namespace isolation can reduce tenant-query leakage and scan cost | [Pinecone multitenancy](https://docs.pinecone.io/guides/index-data/implement-multitenancy) | One vendor pattern; evaluate operational limits and keep deterministic tenant binding |
| Policy language and relationship models should live outside prompts | [Cedar documentation](https://docs.cedarpolicy.com/), [OpenFGA modeling](https://openfga.dev/docs/modeling) | Examples of external policy mechanisms; no specific engine is mandated |

### Connectors, changes, and company data

| Claim used in blueprint | Strongest evidence | Interpretation |
|---|---|---|
| A delta feed represents latest state, not necessarily every intermediate mutation | [Microsoft Graph `driveItem: delta`](https://learn.microsoft.com/en-us/graph/api/driveitem-delta?view=graph-rest-1.0) | Upserts must be idempotent and clients must tolerate duplicate/superseded entries |
| Parent permission changes may require descendant refresh | [Google Drive change tracking](https://developers.google.com/workspace/drive/api/guides/about-changes) | Delta events alone may not project effective ACLs correctly; reconciliation is required |
| Change tokens must be stored only after durable processing | [Google Drive manage changes](https://developers.google.com/workspace/drive/api/guides/manage-changes) | Basis for page checkpoint/commit ordering |
| Confluence integrations need source-specific REST/webhook semantics | [Confluence Cloud REST API](https://developer.atlassian.com/cloud/confluence/rest/v1/) | Avoid pretending all connectors share identical change or ACL semantics |
| Confluence restrictions and quotas must be qualified in the deployed app model | [Confluence content restrictions](https://developer.atlassian.com/cloud/confluence/rest/v1/api-group-content-restrictions/), [Confluence rate limiting](https://developer.atlassian.com/cloud/confluence/rate-limiting/) | Capability-test space/content restrictions, app access, scopes, pagination, and `429` behavior |
| Slack events are best effort and retried; history limits depend on app context | [Slack Events API](https://docs.slack.dev/apis/events-api/), [Slack Web API rate limits](https://docs.slack.dev/apis/web-api/rate-limits/) | Acknowledge into an idempotent queue, repair with history/reconciliation, and discover effective installation limits |
| Mail change windows can expire or be folder-scoped | [Gmail synchronization](https://developers.google.com/workspace/gmail/api/guides/sync), [Microsoft Graph message delta](https://learn.microsoft.com/en-us/graph/delta-query-messages) | Test full resync, moves, deletion, retention, delegation, and folder scope instead of treating email as immutable files |
| Object notifications are not ordered exactly-once ledgers | [Amazon S3 event notifications](https://docs.aws.amazon.com/AmazonS3/latest/userguide/notification-how-to-event-types-and-destinations.html), [Cloud Storage Pub/Sub notifications](https://cloud.google.com/storage/docs/pubsub-notifications) | Use object versions/generations, idempotency, inventory reconciliation, and lifecycle-aware deletion semantics |
| Database CDC carries retention and replay obligations | [PostgreSQL logical decoding](https://www.postgresql.org/docs/current/logicaldecoding-explanation.html) | Consumers deduplicate crash replay and operators monitor replication-slot WAL/catalog retention; other databases require their own proof |
| SEC company facts/submissions are structured public primary data | [SEC EDGAR APIs](https://www.sec.gov/search-filings/edgar-application-programming-interfaces) | Use filings and structured facts as high-quality public evidence; preserve filing/form/time context |
| Public sources impose fair-access constraints | [SEC developer resources](https://www.sec.gov/about/developer-resources), [Companies House developer guidelines](https://developer.company-information.service.gov.uk/developer-guidelines) | Connector rate limits, identification, caching, and retry behavior are policy inputs |
| LEI data supports entity and relationship resolution | [GLEIF API](https://www.gleif.org/en/lei-data/gleif-api), [GLEIF Level 2 relationship data](https://www.gleif.org/en/lei-data/access-and-use-lei-data/level-2-data-relationship-record-rr-cdf-2-1-format) | Use identifiers and provenance; do not infer ultimate ownership beyond published relationship semantics |

### Evidence, freshness, and contradictions

| Claim used in blueprint | Strongest evidence | Interpretation |
|---|---|---|
| Provenance should capture entities, activities, derivation, and agents | [W3C PROV overview](https://www.w3.org/TR/prov-overview/), [W3C PROV primer](https://www.w3.org/TR/prov-primer/) | The blueprint uses a smaller application schema compatible with provenance concepts |
| Citation quality has correctness and completeness dimensions | [ALCE](https://aclanthology.org/2023.emnlp-main.398/) | Claim-level citation scoring and verification are necessary; citation presence alone is insufficient |
| RAG quality needs retrieval and generation diagnostics | [RAGAS](https://arxiv.org/abs/2309.15217), [RAGChecker](https://github.com/amazon-science/RAGChecker) | Use as evaluation references, not absolute production oracles |
| Time-sensitive factuality needs explicit fresh evaluation | [FreshQA](https://openreview.net/forum?id=wSvtSOJHRKW) | Store publication/effective/observation/index times and refresh time-sensitive cases |
| Conflicting evidence is a distinct RAG failure mode | [Ragability](https://aclanthology.org/2026.lrec-1.182/), [ConfRAG](https://aclanthology.org/2026.acl-long.11/) | Preserve contradiction sets and resolution rationale rather than silently picking a passage |
| Evidence-backed claim verification can be evaluated separately | [FEVER](https://aclanthology.org/N18-1074/) | Supports atomic claim labels such as supported/refuted/insufficient, adapted to enterprise evidence |

### Context, memory, and compaction

| Claim used in blueprint | Strongest evidence | Interpretation |
|---|---|---|
| Context is finite and should be engineered deliberately | [Anthropic, Effective context engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) | Compact evidence references and state, not merely transcript prose |
| Workflow state and conversational context are different concerns | [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents), [Effective context engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) | Persist durable execution facts independently from model context |
| Evidence in the middle of long inputs may be underused | [Lost in the Middle](https://aclanthology.org/2024.tacl-1.9/) | Structure and select context; verify evidence use |

### Security, tools, and governance

| Claim used in blueprint | Strongest evidence | Interpretation |
|---|---|---|
| Indirect prompt injection needs layered defenses | [Google Security Blog, layered defenses](https://security.googleblog.com/2025/06/), [Anthropic containment](https://www.anthropic.com/engineering/how-we-contain-claude) | Treat content as untrusted and make tools/policy independently safe |
| Embeddings and vector stores can expose or mix sensitive data | [OWASP vector and embedding weaknesses](https://genai.owasp.org/llmrisk/llm082025-vector-and-embedding-weaknesses/) | Derived artifacts inherit sensitivity and require tenant/access controls |
| AI risk management needs governance and measurement, not only model filters | [NIST AI RMF](https://www.nist.gov/itl/ai-risk-management-framework), [NIST AI 600-1](https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.600-1.pdf) | Basis for named risk owners, threat modeling, evals, monitoring, and incident processes |
| MCP authorization must validate audience and avoid credential passthrough | [MCP 2026-07-28 authorization](https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization), [MCP 2026-07-28 changelog](https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/docs/specification/2026-07-28/changelog.mdx) | Use current dated behavior; still require local policy, schema, and destination controls |
| Retention/deletion covers derived data and media sanitization program | [GDPR Article 17](https://eur-lex.europa.eu/eli/reg/2016/679/art_17/oj/eng), [NIST SP 800-88r2](https://csrc.nist.gov/pubs/sp/800/88/r2/final) | Implementation needs jurisdictional counsel and store-specific deletion guarantees |

### Reliability, observability, and evaluation

| Claim used in blueprint | Strongest evidence | Interpretation |
|---|---|---|
| Traces, metrics, and logs are complementary signals | [OpenTelemetry signals](https://opentelemetry.io/docs/concepts/signals/) | Correlate workflow and dependency behavior without indiscriminate payload logging |
| GenAI/agent telemetry conventions are still evolving | [OpenTelemetry GenAI conventions](https://github.com/open-telemetry/semantic-conventions-genai), [agent spans](https://github.com/open-telemetry/semantic-conventions-genai/blob/main/docs/gen-ai/gen-ai-agent-spans.md) | Status is Development at the baseline; pin schema and expect change |
| Baggage can propagate sensitive data downstream | [OpenTelemetry baggage](https://opentelemetry.io/docs/concepts/signals/baggage/) | Do not place document text, tokens, or sensitive identity in baggage |
| Incompatible search versions can be switched atomically through aliases | [Elasticsearch alias updates](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-indices-update-aliases) | One implementation pattern for shadow-index promotion; not a product requirement |
| Search snapshots can reduce derived-index recovery time | [Qdrant snapshots](https://qdrant.tech/documentation/concepts/snapshots/) | Still test source-based rebuild and authorization restoration |
| Standard IR metrics and benchmark tooling are available | [TREC RAG 2024](https://trec.nist.gov/data/rag2024.html), [NIST `trec_eval`](https://github.com/usnistgov/trec_eval), [BEIR](https://github.com/beir-cellar/beir) | Use task slices and enterprise judgments; public benchmarks are not release gates |
| Agent evaluations need layered tasks and production feedback | [Anthropic, Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) | Combine component, workflow, end-to-end, human, and production-derived cases |

## Important implementation findings

### Source sync is replication, not ETL

Microsoft Graph and Google Drive documentation both undermine a simplistic “consume every event exactly once” model. Delta/change feeds can represent current state, return duplicate or superseded records, omit descendant permission events, and require token/cursor lifecycle handling. Therefore the blueprint uses:

```text
periodic full inventory + incremental change feed + reconciliation + tombstones
```

The index is a versioned projection. A source record is not considered converged until content, metadata, effective ACL, derivatives, and deletion state agree.

### Connector qualification is an empirical admission gate

The second research pass broadened connector evidence beyond file stores. Slack documents best-effort event delivery and retries; Gmail documents that history ranges can become unavailable; object stores document duplicated or unordered notifications; PostgreSQL documents crash replay and resource retention for logical slots; Elastic and vector products document different filter/isolation limitations. These mechanisms cannot be compressed into a universal “webhook + ACL” adapter without losing production semantics.

The blueprint therefore requires a per-deployment capability card and test pack covering stable identity, versions, inventory, incremental changes, permissions, deletion, user-visible drill-down, cursor expiry, quotas, data terms, reconstruction, and removal. The safe answer to a failed qualification is often a smaller source subset, live per-user retrieval, source-native links, or exclusion—not a guessed permission projection.

### Authorization is a cross-layer invariant

Search products provide document filters, namespaces, or data-source ACL support, but their semantics differ. Elastic explicitly documents role-combination and aggregation behavior that can surprise implementers. Source systems differ on inherited access. Graph relationships and caches add additional leak paths. Therefore the blueprint rechecks authorization:

1. before query execution where the engine supports deterministic filters;
2. after candidate retrieval and before evidence enters model context;
3. when a citation or artifact is viewed later;
4. before any tool resource is fetched or mutated.

This redundancy is intentional defense in depth, not a reason to retrieve unfiltered data into the application.

### Hybrid retrieval is a baseline, not an endpoint

Exact names, identifiers, error codes, dates, and phrases favor lexical signals; paraphrases and concept matches favor dense representations. RRF is a robust baseline because it combines rank positions without treating incomparable engine scores as calibrated probabilities. Domain rerankers, learned sparse models, and late interaction may improve selected slices but add latency, cost, storage, and operational versions.

### Graph usefulness is query-dependent

GraphRAG distinguishes local/entity-focused, global/community, and DRIFT-style search. Standard graph extraction is materially more expensive than faster/noisier variants. The practical conclusion is not “enterprise knowledge needs a graph”; it is “relationship-heavy or corpus-wide questions may justify a graph after simpler retrieval is measured.” Any graph must inherit temporal, ACL, deletion, and provenance contracts.

### Citation presence is not evidence quality

ALCE and evidence-verification work support separating citation correctness and completeness. A plausible source link can still fail to entail a claim, can point to a superseded version, or can be inaccessible to the viewer. The blueprint therefore stores exact spans and verifies quotes, claim support, source dates, and current access.

### Contradictions and time need first-class state

FreshQA and conflict-oriented RAG research show that a system can be fluent while using stale or conflicting material. Enterprise sources often disagree for legitimate reasons: effective date, jurisdiction, definition, business unit, preliminary/final status, or source authority. The system should preserve alternative claims and their scopes, not average numerical facts or let retrieval rank silently decide truth.

### More context is not reliable retrieval

The long-context literature and context-engineering guidance support a bounded evidence set and explicit compaction. Summaries can omit qualifiers and cannot become new evidence. Durable workflow state should store decisions, evidence IDs, completed steps, open evidence slots, budgets, and versions; the model context should be reconstructed from that state.

### Tool safety must not depend on instruction following

Prompt injection is inherent whenever untrusted content enters context. Classifiers and system prompts may reduce risk but cannot create an authorization boundary. Typed tools, destination controls, short-lived audience-bound credentials, action policies, parameter-bound approvals, idempotency, and receipts make the system safe even when model output is adversarial.

## Areas of disagreement or limited evidence

### Agent loop versus fixed workflow

- Agent engineering reports show value for open-ended tasks where the evidence path is unknown.
- The same guidance warns about cost, latency, and compounding errors, and recommends simple workflows when possible.
- Research benchmarks often use clean tool environments and do not capture enterprise ACL drift, partial connectors, or approval protocols.

**Blueprint resolution:** route by task complexity. Use deterministic search and workflows for known patterns, a bounded plan/evaluate/refine loop for genuine evidence uncertainty, and explicit stop conditions.

### Graph RAG versus hybrid RAG

- Graph methods can improve global thematic and relationship questions.
- Graph construction introduces extraction errors, expensive indexing, temporal ambiguity, ACL propagation, and a second derived corpus to operate.
- Vendor and paper comparisons do not guarantee value on a particular enterprise corpus.

**Blueprint resolution:** no graph in the minimum stack. Add it only after a query-sliced evaluation shows material gain over filters, query expansion, RRF, and reranking.

### Prefilter versus postfilter for vector retrieval

- Prefiltering protects authorization and reduces scanned data but some approximate-nearest-neighbor implementations lose recall under selective filters.
- Postfiltering a small top-k candidate set can starve results and is unsafe if unfiltered content crosses the trusted retrieval boundary.

**Blueprint resolution:** authorization must be enforced before evidence use. Prefer an engine/partition strategy that supports efficient filtered ANN; overfetch and postfilter may be an additional trusted-layer measure but not the sole security boundary.

### Model-as-judge evaluation

- Model graders scale nuanced evaluation and are useful for triage.
- They may share biases with the system, change across versions, accept plausible unsupported text, or mishandle policy.

**Blueprint resolution:** use deterministic oracles for authorization, citations, formulas, schemas, tool effects, budgets, and state. Calibrate model graders against blinded humans for judgment-heavy dimensions.

### Memory scope

- Long-term preferences can personalize answers and reduce repeated setup.
- Persistent memory complicates consent, deletion, authorization invalidation, tenant separation, and reproducibility.

**Blueprint resolution:** begin with short-lived run state and explicit user preferences. Add persistent memory only for a named use case with provenance, scope, expiry, inspection, and deletion.

### Vendor performance claims

Google Research reports substantial improvements for its enterprise agentic RAG design on selected factuality datasets. This is useful evidence that decomposition and sufficiency checks can help, but it is not a transferable guarantee because the full system, corpora, baselines, and evaluation conditions differ.

**Blueprint resolution:** record the mechanism, not the headline percentage; require local evaluation.

## Current version and date baseline

| Item | Baseline used | Why a refresh matters |
|---|---|---|
| Model Context Protocol | Specification/announcement dated 2026-07-28, described as GA; includes stateless operation, per-request metadata/headers, and authorization changes | Tool transport, auth, and compatibility details are moving quickly |
| OpenTelemetry GenAI conventions | Repository state on 2026-08-31; agent spans marked Development | Attribute and span names may change incompatibly |
| NIST SP 800-88 | Revision 2 final, September 2025; supersedes Revision 1 | Retention/sanitization citations should not point to the obsolete revision |
| NIST AI RMF | AI RMF 1.0 materials with NIST revision activity noted in 2026 | Governance guidance may be revised |
| Microsoft Graph drive delta | Documentation viewed for current v1.0 endpoint | Delta, permissions, and header behavior can evolve |
| Google Drive changes | Current Drive API guidance at baseline | Change-feed and ACL propagation behavior is connector-critical |
| Slack Events/Web API | Current delivery, retry, and rate-limit guidance at baseline | Effective history limits depend on app distribution/install context and may change |
| Confluence Cloud | Current restriction and rate-limit guidance at baseline | App model, scopes, app-access rules, quota enforcement, and content types evolve |
| Gmail and Microsoft Graph mail | Current history/delta guidance at baseline | Cursor availability, folder scope, retention, and delegated access affect convergence |
| Object-store notifications | Current Amazon S3 and Google Cloud Storage notification guidance at baseline | Delivery, ordering, lifecycle, version, and delete-marker semantics are source-specific |
| PostgreSQL logical decoding | Current PostgreSQL documentation at baseline | Replication-slot replay, timeout, and WAL-retention behavior is version/configuration-sensitive |
| SEC EDGAR APIs | Current data API and developer guidance at baseline; fair-access ceiling currently documented as 10 requests/second | Public-source rate and identification rules can change |
| Companies House | Developer guidance at baseline; documented default limit 600 requests per five minutes | Limits and application policies affect connector scheduling |
| GraphRAG | Current documentation and repository-linked methods at baseline | Query modes, extraction methods, defaults, and costs evolve |
| GDPR | Consolidated official regulation text linked | Jurisdictional interpretation and organizational obligations require counsel |

The guides intentionally avoid pinning application-library versions because the repository is architecture documentation, not an implementation lockfile. An implementation should add an environment-specific compatibility matrix and exact dependency versions.

## Claims deliberately not made

- No model, embedding model, vector database, graph database, framework, or workflow engine is declared universally best.
- No public benchmark is treated as a release threshold for a private enterprise corpus.
- No context-window size is treated as a substitute for retrieval or evidence verification.
- No prompt, classifier, or guard model is claimed to eliminate prompt injection.
- No shared-index filter is assumed safe without engine-specific authorization tests.
- No connector is assumed to deliver exactly-once, complete, ordered events.
- No external company registry is assumed complete, current, or legally authoritative for every claim.
- No retention mechanism is claimed to satisfy a jurisdiction without legal and records-management review.
- No latency, cost, or accuracy number in the guides is represented as a universal benchmark.

## Source register

### Architecture, retrieval, and orchestration

1. [Anthropic — Building effective agents](https://www.anthropic.com/engineering/building-effective-agents)
2. [Anthropic — Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
3. [Google Research — Unlocking dependable responses with enterprise agentic RAG](https://research.google/blog/unlocking-dependable-responses-with-gemini-enterprise-agent-platforms-agentic-rag/)
4. [Microsoft GraphRAG documentation](https://microsoft.github.io/graphrag/)
5. [Microsoft GraphRAG query overview](https://microsoft.github.io/graphrag/query/overview/)
6. [Microsoft GraphRAG indexing methods](https://microsoft.github.io/graphrag/index/methods/)
7. [GraphRAG paper](https://arxiv.org/abs/2404.16130)
8. [IRCoT](https://aclanthology.org/2023.acl-long.557/)
9. [Self-RAG](https://openreview.net/forum?id=hSyW5go0v8)
10. [Corrective Retrieval-Augmented Generation](https://arxiv.org/abs/2401.15884)
11. [Azure hybrid search overview](https://learn.microsoft.com/en-us/azure/search/hybrid-search-overview)
12. [Azure reciprocal-rank fusion](https://learn.microsoft.com/en-us/azure/search/hybrid-search-ranking)
13. [Elasticsearch reciprocal-rank fusion](https://www.elastic.co/docs/reference/elasticsearch/rest-apis/reciprocal-rank-fusion)
14. [BEIR](https://github.com/beir-cellar/beir)
15. [ColBERT](https://github.com/stanford-futuredata/ColBERT)
16. [SPLADE](https://github.com/naver/splade)
17. [Lost in the Middle](https://aclanthology.org/2024.tacl-1.9/)

### Authorization and policy

18. [Google Zanzibar paper](https://research.google/pubs/zanzibar-googles-consistent-global-authorization-system/)
19. [NIST SP 800-207](https://csrc.nist.gov/pubs/sp/800/207/final)
20. [Azure query-time ACL enforcement](https://learn.microsoft.com/en-us/azure/search/search-query-access-control-rbac-enforcement)
21. [Google data-source access control](https://docs.cloud.google.com/generative-ai-app-builder/docs/data-source-access-control)
22. [Elasticsearch document-level security](https://www.elastic.co/guide/en/elasticsearch/reference/current/document-level-security.html)
23. [PostgreSQL row security](https://www.postgresql.org/docs/current/ddl-rowsecurity.html)
24. [Pinecone multitenancy](https://docs.pinecone.io/guides/index-data/implement-multitenancy)
25. [Cedar documentation](https://docs.cedarpolicy.com/)
26. [OpenFGA modeling](https://openfga.dev/docs/modeling)

### Connectors and public-company sources

27. [Microsoft Graph `driveItem: delta`](https://learn.microsoft.com/en-us/graph/api/driveitem-delta?view=graph-rest-1.0)
28. [Google Drive change tracking](https://developers.google.com/workspace/drive/api/guides/about-changes)
29. [Google Drive manage changes](https://developers.google.com/workspace/drive/api/guides/manage-changes)
30. [Confluence Cloud REST API](https://developer.atlassian.com/cloud/confluence/rest/v1/)

Pass 2 connector-qualification additions:

- [Google Drive sharing and permission inheritance](https://developers.google.com/workspace/drive/api/guides/manage-sharing)
- [Confluence Cloud content restrictions](https://developer.atlassian.com/cloud/confluence/rest/v1/api-group-content-restrictions/)
- [Confluence Cloud rate limiting](https://developer.atlassian.com/cloud/confluence/rate-limiting/)
- [Slack Events API delivery and retries](https://docs.slack.dev/apis/events-api/)
- [Slack Web API rate limits](https://docs.slack.dev/apis/web-api/rate-limits/)
- [Gmail synchronization and history expiry](https://developers.google.com/workspace/gmail/api/guides/sync)
- [Microsoft Graph message delta](https://learn.microsoft.com/en-us/graph/delta-query-messages)
- [Amazon S3 event notification types and delivery](https://docs.aws.amazon.com/AmazonS3/latest/userguide/notification-how-to-event-types-and-destinations.html)
- [Google Cloud Storage Pub/Sub notification guarantees](https://cloud.google.com/storage/docs/pubsub-notifications)
- [PostgreSQL logical decoding concepts](https://www.postgresql.org/docs/current/logicaldecoding-explanation.html)
- [Qdrant multitenancy](https://qdrant.tech/documentation/tutorials/multiple-partitions/)

31. [SEC EDGAR application programming interfaces](https://www.sec.gov/search-filings/edgar-application-programming-interfaces)
32. [SEC developer resources and fair access](https://www.sec.gov/about/developer-resources)
33. [Companies House developer guidelines](https://developer.company-information.service.gov.uk/developer-guidelines)
34. [GLEIF API](https://www.gleif.org/en/lei-data/gleif-api)
35. [GLEIF Level 2 relationship records](https://www.gleif.org/en/lei-data/access-and-use-lei-data/level-2-data-relationship-record-rr-cdf-2-1-format)

### Evidence and evaluation

36. [W3C PROV overview](https://www.w3.org/TR/prov-overview/)
37. [W3C PROV primer](https://www.w3.org/TR/prov-primer/)
38. [ALCE](https://aclanthology.org/2023.emnlp-main.398/)
39. [RAGAS](https://arxiv.org/abs/2309.15217)
40. [RAGChecker](https://github.com/amazon-science/RAGChecker)
41. [TREC 2024 RAG track](https://trec.nist.gov/data/rag2024.html)
42. [NIST `trec_eval`](https://github.com/usnistgov/trec_eval)
43. [FEVER](https://aclanthology.org/N18-1074/)
44. [FreshQA](https://openreview.net/forum?id=wSvtSOJHRKW)
45. [Ragability](https://aclanthology.org/2026.lrec-1.182/)
46. [ConfRAG](https://aclanthology.org/2026.acl-long.11/)
47. [Anthropic — Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents)

### Security, governance, and tool protocols

48. [NIST AI Risk Management Framework](https://www.nist.gov/itl/ai-risk-management-framework)
49. [NIST AI 600-1, Generative AI Profile](https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.600-1.pdf)
50. [NIST SP 800-88 Revision 2](https://csrc.nist.gov/pubs/sp/800/88/r2/final)
51. [OWASP vector and embedding weaknesses](https://genai.owasp.org/llmrisk/llm082025-vector-and-embedding-weaknesses/)
52. [Google Security Blog — layered defenses for indirect prompt injection](https://security.googleblog.com/2025/06/)
53. [Anthropic — How we contain Claude](https://www.anthropic.com/engineering/how-we-contain-claude)
54. [MCP 2026-07-28 GA announcement](https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/blog/content/posts/2026-07-28-spec-ga/index.md)
55. [MCP 2026-07-28 changelog](https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/docs/specification/2026-07-28/changelog.mdx)
56. [MCP 2026-07-28 authorization](https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization)
57. [GDPR Article 17](https://eur-lex.europa.eu/eli/reg/2016/679/art_17/oj/eng)

### Operations and deployment

58. [OpenTelemetry signals](https://opentelemetry.io/docs/concepts/signals/)
59. [OpenTelemetry generative-AI semantic conventions](https://github.com/open-telemetry/semantic-conventions-genai)
60. [OpenTelemetry agent spans](https://github.com/open-telemetry/semantic-conventions-genai/blob/main/docs/gen-ai/gen-ai-agent-spans.md)
61. [OpenTelemetry baggage](https://opentelemetry.io/docs/concepts/signals/baggage/)
62. [Kubernetes Horizontal Pod Autoscaling](https://kubernetes.io/docs/concepts/workloads/autoscaling/horizontal-pod-autoscale/)
63. [Elasticsearch alias updates](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-indices-update-aliases)
64. [Qdrant snapshots](https://qdrant.tech/documentation/concepts/snapshots/)

## Guide coverage map

| Guide | Primary evidence areas |
|---|---|
| `README.md` | All; architecture summary and baseline |
| `product-contract-and-workflows.md` | Agent scope, workflow/agent boundary, risk tiers |
| `architecture-and-stack-selection.md` | Architecture, RAG, graph, language/runtime, MCP |
| `connectors-ingestion-and-corpus-sync.md` | Change APIs, source semantics, external company data |
| `connector-qualification-and-adapter-playbooks.md` | Drive, SharePoint, Confluence, Slack, email, object-store, database, search/vector, and MCP admission/operations |
| `identity-aware-retrieval-and-ranking.md` | Zanzibar, zero trust, search ACLs, hybrid retrieval |
| `research-orchestration-state-and-context.md` | Decomposition, bounded loops, context and compaction |
| `evidence-citations-freshness-and-contradictions.md` | PROV, ALCE, freshness, conflicts, claim verification |
| `security-governance-and-outbound-actions.md` | Injection, tools, MCP auth, governance, deletion |
| `reliability-observability-deployment-and-cost.md` | Durable recovery, OTel, migration, scaling and cost |
| `evaluation-and-acceptance-testing.md` | IR metrics, citation/factuality/conflict evals, release gates |
| `implementation-roadmap.md` | Evidence-gated sequencing and optional complexity |

## Limitations and unresolved empirical questions

- The target organization’s identity provider, group nesting, source ACL semantics, data regions, retention schedule, and regulated-data categories are unknown. The guides define contracts and tests, not a ready policy.
- Search-engine behavior under highly selective ACL filters is workload- and engine-specific. Benchmark authorized recall and p99 latency on representative tenant/group distributions.
- Public-company sources vary by jurisdiction, filing regime, licensing, update cadence, and entity coverage. The included SEC, Companies House, and GLEIF sources are examples, not a global registry strategy.
- Extraction quality for PDFs, spreadsheets, slides, scans, charts, and comments depends on the actual corpus and parser stack.
- Graph value and graph extraction error cannot be estimated without relationship-heavy questions and gold evidence paths.
- Model choice, prompt, token limits, latency, and cost require current provider evaluations; the blueprint intentionally does not select a vendor.
- Prompt injection cannot be eliminated. The design reduces consequences through containment, independent authorization, narrow tools, approvals, and monitoring.
- Legal and regulatory requirements depend on jurisdiction, contractual role, record type, and organization policy. Counsel and records-management owners must approve implementation.
- OTel GenAI semantics and MCP details are current but evolving; implementation schemas need pinned compatibility tests.
- The blueprint contains illustrative objectives and config values. Teams must replace them with measured and approved targets.
- Slack, mail, Drive, SharePoint, Confluence, object-store, database, search/vector, and MCP behavior varies by plan, tenant configuration, scopes, region, and deployed version. The connector guide defines a qualification method, not a universal compatibility promise.

## Refresh triggers

Refresh this packet and affected guides when any of the following occurs:

- a new MCP dated specification or material authorization/security amendment is released;
- OpenTelemetry GenAI agent conventions graduate from Development or change schema;
- a connector changes delta tokens, permission inheritance, webhook guarantees, quotas, or deletion semantics;
- the organization adds a tenant model, data region, model provider, source class, or outbound action;
- a search engine, embedding model, reranker, parser, graph extractor, or ACL representation changes;
- NIST AI RMF, NIST sanitization guidance, privacy law, records policy, or contractual requirements change;
- a security incident reveals a new injection, exfiltration, tenant, cache, approval, or tool failure;
- production evaluation exposes a new important task family or a material slice regression;
- graph or persistent-memory proposals reach adoption review;
- at least six months pass, even if no trigger is reported.

At refresh, re-open primary sources, record the date and changed claims, rerun local acceptance suites, and retain the prior packet through repository history.
