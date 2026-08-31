# Research Packet: Document Intelligence Agent Blueprint

**Status:** Completed synthesis for the production blueprint  
**Research date:** 2026-08-31  
**Scope:** Intake, identity, parsing, OCR, layout, tables, images, handwriting, classification, extraction, validation, review, security, privacy, tenancy, retention, durable workflows, material effects, evaluation, deployment, cost, and staged delivery.  
**Derived guide:** [Document Intelligence Agent](../../agents/document-intelligence-agent/README.md)

## Research objective

Determine the smallest production architecture that can convert hostile, heterogeneous business documents into evidence-backed structured records and, where separately authorized, controlled external actions. The research explicitly tested whether an autonomous agent should be the default architecture and how claims of accuracy, safety, durability, and “exactly once” behavior survive real failure boundaries.

## Method

Research prioritized current primary sources:

1. standards and government publications for provenance, security, privacy, media annotation, signatures, and sanitization;
2. official cloud/service documentation for response models, limits, version status, evaluation, quotas, and regional caveats;
3. official parser/runtime documentation and repositories for isolation, embedded content, retries, replay, and idempotency;
4. original benchmark and calibration/selective-prediction papers for evaluation semantics;
5. vendor security engineering only where it contributed a concrete threat/control model.

Claims were cross-checked at control boundaries. Service features and versions were recorded as a dated baseline, not treated as permanent. Marketing accuracy numbers, universal confidence thresholds, and unqualified exactly-once claims were excluded.

Search/review families included:

- document AI response schemas, OCR/layout/table/signature/handwriting capabilities, limits, processor versions, regional behavior, and evaluation;
- hostile file upload, parser sandboxing, embedded resources, archive limits, macros, PDF active content, prompt injection, and content disarm/reconstruction;
- provenance, byte identity, annotation selectors, OCR exchange formats, immutable retention, legal holds, and sanitization;
- workflow determinism, retry semantics, transaction authorization, idempotency, unknown external outcomes, and reconciliation;
- calibration, selective automation, public document benchmarks, observability, failure injection, and adversarial ML.

## Dated technology baseline

| Area | Baseline observed on 2026-08-31 | Design implication | Volatility |
|---|---|---|---|
| Apache Tika | 4.0.0 current download; security docs require external isolation for untrusted content | parser runs disposable, without network/credentials, under hard limits | medium |
| AWS Textract | documented sync/async page and size limits; block graph includes geometry, confidence, handwriting/signature/layout types | adapter normalizes provider graph and preflights documented limits | high |
| Google Document AI | stable and preview/RC processor versions coexist; response uses anchors, pages, entities, normalized vertices, confidence | pin immutable processor versions and verify region/data-use behavior per version | high |
| Azure Document Intelligence | v4.0 `2024-11-30` GA baseline in current service docs | pin API/model version; locally evaluate and calibrate | high |
| JSON Schema | Draft 2020-12 current referenced core/validation vocabulary | typed output and validation contract | low |
| W3C PROV / Web Annotation | stable W3C Recommendations | provenance graph and page/region evidence selectors | low |
| BagIt | RFC 8493 | optional package manifest for transfer/fixity, not a workflow model | low |
| NIST SP 800-53 | Release 5.2.0 current cited control catalog | control mapping, separation of duties, audit and access design | medium |
| NIST SP 800-88 | Rev. 2 final, September 2025 | sanitization program and deletion verification evidence | low/medium |
| ETSI PAdES | EN 319 142-1 V1.2.1 referenced | preserve exact signed bytes and validate signature semantics separately | medium |
| NIST AI security | AI 100-2 E2025 final | adversarial ML/prompt-injection threat baseline | medium |
| NIST AI RMF profile | AI 600-1 final | lifecycle risk/evaluation baseline, not a product certification | medium |

Preview, release-candidate, processor-version, quota, region, price, and data-handling statements require deployment-time revalidation.

## Synthesized findings

### 1. The default architecture is a durable workflow, not a free-form agent loop

Document processing has stable stages, typed contracts, completeness requirements, and high-cost failure modes. Deterministic services should own byte admission, identity, page manifests, state transitions, schemas, validation, authorization, retention, and effects. Models assist with perception and semantic interpretation. A bounded agent is justified only for measured exceptions requiring multi-step adaptive investigation.

Temporal documents deterministic workflow replay and treats external work as activities; DBOS similarly ties durability to recorded/checkpointed execution. Neither removes the need for idempotent external effects. See [Temporal Workflow Definition](https://docs.temporal.io/workflow-definition), [Temporal Workflow Execution](https://docs.temporal.io/workflow-execution), and [DBOS architecture](https://docs.dbos.dev/architecture).

### 2. Originals, observations, and interpretations must remain distinct

The system preserves immutable original bytes and creates immutable derivatives with lineage. A rendered page, CDR output, OCR text, layout graph, normalized field, and accepted record are different entities. W3C PROV supplies entity/activity/agent relations; Web Annotation supplies selectors for evidence regions. See [W3C PROV overview](https://www.w3.org/TR/prov-overview/) and [Web Annotation Data Model](https://www.w3.org/TR/annotation-model/).

This enables correction without rewriting history and allows a reviewer to inspect the literal evidence that produced a normalized value.

### 3. Upload, artifact, document, revision, run, and effect identities solve different problems

A SHA-256 digest answers exact-byte identity, not business identity. Two render-equivalent PDFs can have different bytes; two similar invoices can be distinct obligations; one invoice can arrive by multiple channels; one corrected invoice can be a new revision. Preserve every intake event and classify duplicate relations explicitly.

BagIt demonstrates fixity manifests for transferred payloads but is not a business-deduplication scheme or workflow. See [RFC 8493](https://www.rfc-editor.org/rfc/rfc8493).

### 4. A parser is not a security boundary

Apache Tika explicitly states that it is not designed as a security boundary. Untrusted parsing requires process/container isolation, no network, no secrets, CPU/memory/time/decompression limits, and an external kill mechanism. Tika also documents embedded-document handling and configurable limits. See [Tika security model](https://tika.apache.org/security-model.html), [Tika security](https://tika.apache.org/docs/4.0.x/security.html), and [Tika setting limits](https://tika.apache.org/docs/4.0.x/advanced/setting-limits.html).

One subtle adapter requirement: a parser may return a nominal success while logging that embedded extraction was truncated. Truncation and limit hits therefore become first-class `INCOMPLETE` evidence, not log-only warnings.

### 5. File admission requires layered evidence

OWASP recommends allowlisting extensions, generating storage names, validating file signatures/types, enforcing size limits, storing outside the web root, authorizing uploaders, and scanning/sandboxing; it also emphasizes that no single check is sufficient. See [OWASP File Upload Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html).

Macros being blocked by default in some desktop Office scenarios does not make them safe in a server pipeline. The server never opens active content in an interactive privileged desktop process. See [Microsoft macro security guidance](https://learn.microsoft.com/en-us/microsoft-365-apps/security/internet-macros-blocked).

### 6. Use a native-text → OCR → specialized/visual cascade

Born-digital text should be extracted first and checked for page coverage, ordering, glyph mapping, and evidence geometry. OCR is needed for scans, images, broken text layers, or selected regions. Tables, handwriting, figures, signatures, checkboxes, and dense layout require specialized routes or visual interpretation.

AWS, Google, and Azure expose materially different response structures and capabilities. AWS Textract models blocks and relationships; Google Document AI uses text/page anchors and normalized vertices; Azure returns layout/document structures. Normalize into a provider-independent intermediate representation while preserving raw responses and provider-specific version provenance. See [AWS Block API](https://docs.aws.amazon.com/textract/latest/APIReference/API_Block.html), [Google Document response model](https://docs.cloud.google.com/document-ai/docs/reference/rest/v1/Document), and [Azure layout model](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/prebuilt/layout).

### 7. Page completeness is a hard invariant

Processing “most pages” is not success. Create an authoritative expected page/embedded-object manifest, attach stable page identities, and join by exact set equality. Duplicate page 4 cannot compensate for missing page 5. Provider page/size limits and asynchronous job semantics make explicit completeness checks necessary. See [AWS Textract limits](https://docs.aws.amazon.com/textract/latest/dg/limits-document.html) and [asynchronous processing](https://docs.aws.amazon.com/textract/latest/dg/async.html).

### 8. Typed extraction needs evidence and semantic null states

JSON Schema Draft 2020-12 is suitable for structural validation, but business validation and provenance require additional contracts. Each field should retain literal text, normalized value, page/region evidence, extraction method, version, and confidence features. `missing`, `illegible`, `inferred`, `conflicting`, and `not_applicable` are not one `null`.

See [JSON Schema Core 2020-12](https://json-schema.org/draft/2020-12/json-schema-core) and [JSON Schema Validation 2020-12](https://json-schema.org/draft/2020-12/json-schema-validation).

### 9. Confidence is local decision evidence, not truth

Vendor confidence values are not comparable across providers, fields, versions, or document populations, and a value such as 0.8 is not a universal review threshold. Calibrate the final acceptance decision on held-out local data per field/class/slice. Measure risk at accepted coverage and retain abstention.

Google Document AI evaluation uses precision/recall/F1 and thresholds; Azure's transparency guidance recommends local pilots and treats numerical thresholds as examples. General calibration and selective-classification research support the local risk/coverage model. See [Google Document AI evaluation](https://docs.cloud.google.com/document-ai/docs/evaluate), [Azure transparency note](https://learn.microsoft.com/en-us/azure/foundry/responsible-ai/document-intelligence/transparency-note), [Guo et al. calibration](https://proceedings.mlr.press/v70/guo17a.html), and [selective classification](https://proceedings.mlr.press/v130/gangrade21a.html).

### 10. Prompt injection is contained by authority architecture

Document content can instruct a model through visible text, hidden text, metadata, OCR, images, barcodes, links, or embedded objects. A classifier cannot perfectly label every attack. Therefore the model lane receives a trusted typed task, treats document content as untrusted data, has no credentials or material-effect tools, and produces schema-constrained proposals validated by deterministic services.

This follows the security conclusion that prevention filters alone are insufficient: constrain reachable assets and actions even if manipulation succeeds. See [NIST AI 100-2 E2025](https://csrc.nist.gov/pubs/ai/100/2/e2025/final), the [indirect prompt-injection paper](https://arxiv.org/abs/2302.12173), and [OpenAI's agent security design](https://openai.com/index/designing-agents-to-resist-prompt-injection/).

### 11. CDR creates a derivative and may break authenticity evidence

Content disarm/reconstruction can remove active features and produce a safer review copy. It is not proof that a file is harmless, and the derivative is not the original. Rewriting a PDF can change exact bytes, interactive features, revision evidence, and signature validity. Preserve the original and signature-verification evidence; use the derivative for narrowly defined viewing/processing purposes.

PAdES requirements make exact signed-document semantics important. See [ETSI EN 319 142-1](https://www.etsi.org/deliver/etsi_EN/319100_319199/31914201/01.02.01_60/en_31914201v010201p.pdf).

### 12. Tenant, purpose, retention, holds, and deletion form one data-lifecycle contract

Tenant and purpose travel on every artifact, derivative, cache entry, review, evaluation sample, provider request, workflow, and effect. Access checks are server-side and non-enumerating. The data map includes originals, derivatives, logs, caches, provider copies, evaluation sets, exports, indexes, and backups.

WORM/immutable storage protects records from premature mutation but can conflict with deletion obligations if retention is misconfigured. Object locks are version-specific and legal holds are governance decisions. See [Amazon S3 Object Lock](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock.html), [Azure immutable blob storage](https://learn.microsoft.com/en-us/azure/storage/blobs/immutable-storage-overview), and [Google Cloud Bucket Lock](https://docs.cloud.google.com/storage/docs/bucket-lock).

NIST SP 800-88 Rev. 2 provides current sanitization-program guidance. GDPR, HIPAA, and PCI sources inform scope but do not make the system compliant by citation. See [NIST SP 800-88 Rev. 2](https://csrc.nist.gov/pubs/sp/800/88/r2/final), [GDPR](https://eur-lex.europa.eu/eli/reg/2016/679/oj/eng), [HIPAA Security Rule](https://www.hhs.gov/hipaa/for-professionals/security/index.html), and [PCI DSS library](https://www.pcisecuritystandards.org/document_library/).

### 13. Document acceptance and effect authorization are separate decisions

Extraction review answers “does this structured record reflect the evidence?” Effect approval answers “may this exact principal cause this exact filing, signature, payment, release, or deletion now?” The second decision binds material arguments, source revision, accepted extraction, destination, policy, approver identity/role, expiry, and one-use semantics.

OWASP transaction authorization supports binding authorization to material transaction data and enforcing it server-side. NIST SP 800-53 provides separation-of-duties controls. See [OWASP Transaction Authorization](https://cheatsheetseries.owasp.org/cheatsheets/Transaction_Authorization_Cheat_Sheet.html) and [NIST SP 800-53 Release 5.2.0](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final).

### 14. Exactly-once external effects require qualification

A local transaction can reserve an operation once. A remote call can be accepted while its acknowledgement is lost. The correct state is `UNKNOWN`; reconcile at the destination before retry. A stable idempotency key represents one semantic operation and is stored with a canonical request hash so different parameters cannot reuse it.

Stripe and AWS document these idempotency principles, while workflow runtimes document retries rather than universal atomicity. See [Stripe idempotent requests](https://docs.stripe.com/api/idempotent_requests) and [AWS idempotent operations guidance](https://docs.aws.amazon.com/wellarchitected/latest/framework/rel_prevent_interaction_failure_idempotent.html).

When a destination supplies neither idempotency nor a reliable query, ambiguous outcomes require manual recovery.

### 15. Evaluation must measure the whole system on production-like slices

Public datasets cover components: FUNSD forms, DocLayNet/PubLayNet layout, DocVQA question answering, PubTables table structure, CORD/SROIE receipts, and IAM handwriting. They do not represent a particular tenant's sources, languages, capture devices, validation rules, workflow, or effects.

Use lawful production-like corpora with temporal/source/device/template holdouts, ordinary and hard cases, semantic nulls, duplicates/revisions, hostile inputs, and crash/effect ambiguity. Grade page completeness, OCR, layout, fields, evidence, calibration, tables, workflow transitions, tenant isolation, and destination receipts separately.

Primary dataset sources: [FUNSD](https://arxiv.org/abs/1905.13538), [DocLayNet](https://arxiv.org/abs/2206.01062), [PubLayNet](https://arxiv.org/abs/1908.07836), [DocVQA](https://openaccess.thecvf.com/content/WACV2021/papers/Mathew_DocVQA_A_Dataset_for_VQA_on_Document_Images_WACV_2021_paper.pdf), [PubTables-1M](https://openaccess.thecvf.com/content/CVPR2022/papers/Smock_PubTables-1M_Towards_Comprehensive_Table_Extraction_From_Unstructured_Documents_CVPR_2022_paper.pdf), [CORD](https://github.com/clovaai/cord), [SROIE](https://arxiv.org/abs/2103.10213), and [IAM](https://fki.tic.heia-fr.ch/databases/iam-handwriting-database).

### 16. Observability, audit, and evaluation data are different assets

Telemetry can be sampled and short-lived; audit records prove policy-relevant events; evaluation corpora contain governed evidence and labels. Keep raw document data and secrets out of ordinary traces and OpenTelemetry baggage. See [OpenTelemetry signals](https://opentelemetry.io/docs/concepts/signals/), [baggage security](https://opentelemetry.io/docs/concepts/signals/baggage/), and [context propagation security](https://opentelemetry.io/docs/concepts/context-propagation/).

### 17. Cost optimization is a routing and review problem

Cost includes pages, regions, model units, storage, compute, reviewer minutes, reprocessing, and downstream defects. Native-text-first parsing, selective OCR, bounded specialized routes, correct versioned caching, and calibrated abstention are practical controls. An apparently cheap model can be expensive if it increases review or defects.

Provider limits and quotas are mutable deployment evidence. See [Google Document AI quotas](https://docs.cloud.google.com/document-ai/quotas), [AWS Textract limits](https://docs.aws.amazon.com/textract/latest/dg/limits-document.html), and [Azure service limits](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/service-limits).

### 18. Capability should advance in risk-gated stages

The recommended sequence is corpus/manual baseline → safe extraction-only pipeline → calibrated structured automation → durable operations/governance → one controlled effect at a time → optional bounded agentic exception handling. Each stage has acceptance and failure drills. Effects are not enabled merely because extraction quality is good.

### 19. Qualify concrete adapter routes, not product names

The reliable abstraction is a versioned route: adapter build + operation/API + processor/model snapshot + configuration + region/cell + data policy + normalization contract. Managed services differ in synchronous/asynchronous behavior, result pagination/sharding, job identity, release maturity, temporary result retention, regional availability, response graphs, and quotas. Self-hosted routes differ by language/model data, parser backend, model weights, runtime, accelerator/driver/precision, plugin policy, and remote-service settings.

AWS documents asynchronous request tokens, job IDs, result pagination, and a default seven-day result window. Google exposes processor-version resources and lists stable and release-candidate versions, including version-specific regional caveats. Microsoft documents same-region temporary analysis storage and a 24-hour deletion window plus an earlier-delete API for current Document Intelligence. These are examples of mutable adapter evidence, not universal defaults. See [Amazon Textract asynchronous operations](https://docs.aws.amazon.com/textract/latest/dg/api-async.html), [Google processor versions](https://docs.cloud.google.com/document-ai/docs/manage-processor-versions), and [Azure Document Intelligence data privacy and security](https://learn.microsoft.com/en-us/azure/foundry/responsible-ai/document-intelligence/data-privacy-security).

### 20. Context compaction and memory are projections over durable records

Each model call receives a recorded, tenant- and purpose-authorized context manifest containing exact artifact/evidence IDs, trust labels, versions, budgets, and tool limits. Compaction produces a schema-validated receipt with source event range, retained IDs, unresolved contradictions, pending/unknown effects, deadlines, budgets, explicit omissions, and compiler/compactor versions. It never replaces originals, observations, decisions, approvals, or effect ledgers.

Turn, working/run, session, durable workflow, domain, long-term, and episodic memory have distinct authority and retention. Durable workflow state is mandatory; raw conversational/vector memory is rejected by default. Long-term and episodic reuse is admitted only through reviewed, scoped, deletion-aware release governance and never makes a prior extraction authoritative for a new document.

### 21. Release the whole behavior bundle and mine failures through governance

Parser, renderer, OCR/layout/table engines, model, prompt, schema, validator, calibration, router, context builder, compactor, memory policy, adapter, reviewer UI, workflow, security policy, and effect adapter interact. Promote their exact combination as one behavior bundle with a signed/integrity-protected manifest, evaluation report, rollout cohort, and rollback target. Component-level improvements do not prove the combined route is safe.

Reviewed corrections, abstentions, incidents, hostile inputs, contract drift, and downstream defects should become minimized, lawful, versioned evaluation cases only after causality, privacy/security, licensing, labeling, leakage, and retention review. They must not write prompts, thresholds, long-term memory, or training sets automatically. Rollback changes future routing; accepted history remains immutable and an affected cohort uses explicit reprocessing/correction workflows.

## Resolved tensions and trade-offs

| Tension | Resolution |
|---|---|
| autonomous agent vs deterministic workflow | deterministic workflow by default; agent only inside a bounded exception lane with measured benefit |
| one provider vs multi-provider abstraction | normalize essential semantics and retain raw provider output; add fallback only for an actual reliability/coverage requirement |
| full-page vision model vs OCR/layout cascade | native text/OCR/layout first for cost, provenance, and determinism; visual route for pages/regions that need it |
| vendor confidence threshold vs local policy | calibrate final field/slice decision locally; measure risk at coverage |
| CDR safety vs authenticity | preserve immutable original and signature evidence; treat CDR as a purpose-specific derivative |
| WORM retention vs erasure | retention/hold policy owns the conflict; lock exact versions intentionally and record unavoidable residuals |
| deduplicate storage vs preserve receipt events | reuse eligible artifact content, but retain every intake and explicit duplicate relation |
| retries vs duplicate effects | reserve stable semantic operation, bind approval, use destination idempotency, mark ambiguity `UNKNOWN`, reconcile before retry |
| public benchmarks vs production confidence | use public sets for component smoke tests; gate on lawful production-like temporal/source/device slices |
| provider fallback vs residency | fallback must meet the same region/data-use/contract policy or fail closed |
| human review vs straight-through processing | calibrate selective acceptance per field/slice; review ambiguity and high-impact cases, then measure downstream defects |
| managed service vs self-hosting | qualify the full route and measured operating burden; self-hosting changes trust/cost/control but does not remove versioning, safety, or evaluation work |
| quick component rollback vs reproducibility | route new work to the prior behavior bundle; preserve accepted history and explicitly migrate or reprocess in-flight/affected work |

## Claims deliberately excluded

The guide does not claim:

- “99% document accuracy” without a defined unit, corpus, slice, and ground truth;
- that a universal confidence threshold is safe;
- that an antivirus or file signature check makes an upload safe;
- that a parser library is a sandbox;
- that CDR output is the original or retains signature validity;
- that a model can reliably validate its own unsupported extraction;
- that agreement among models proves correctness;
- that prompt-injection detection is a complete security boundary;
- that durable workflow replay makes every external write exactly once;
- that a public benchmark demonstrates production readiness;
- that a cited privacy/security standard makes a deployment compliant;
- that an autonomous multi-agent system is inherently more capable or reliable.

## Version- and deployment-sensitive findings

Reverify before implementation and on every material upgrade:

- provider processor/model/API version status (GA, preview, RC, deprecated);
- supported formats, page/byte/pixel limits, languages/scripts, handwriting and signature behavior;
- batch/synchronous/asynchronous behavior, quotas, rate limits, and price;
- processing region, global endpoints, failover routing, retention, training/data-use, logging, and deletion behavior;
- mutable model aliases and response schema changes;
- parser/renderer/scanner security releases;
- workflow-runtime replay/versioning and retry defaults;
- destination idempotency retention, lookup, finality, and reversal behavior;
- jurisdiction-specific signature, record-retention, legal-hold, deletion, privacy, health, and payment obligations.

One concrete caution from the reviewed Google processor catalog: stable and preview/RC versions coexist, and at least one preview path documents use of a global Gemini endpoint that does not satisfy regional residency. Version and region therefore belong in policy and provenance, not a generic provider name.

## Source register

### Document parsing and open-source components

- [Apache Tika download/current release](https://tika.apache.org/download) — release baseline.
- [Apache Tika security model](https://tika.apache.org/security-model.html) — explicit non-security-boundary statement.
- [Apache Tika security](https://tika.apache.org/docs/4.0.x/security.html) — isolation guidance.
- [Apache Tika setting limits](https://tika.apache.org/docs/4.0.x/advanced/setting-limits.html) — timeout/limit configuration.
- [Apache Tika embedded documents](https://tika.apache.org/docs/4.0.x/advanced/embedded-documents.html) — embedded extraction semantics.
- [Apache Tika unpack configuration](https://tika.apache.org/docs/4.0.x/pipes/unpack-config.html) — fork/unpack/truncation behavior.
- [Tesseract OCR repository](https://github.com/tesseract-ocr/tesseract/blob/main/README.md) — self-hosted OCR baseline and language-data context.
- [Docling model catalog](https://github.com/docling-project/docling/blob/main/docs/usage/model_catalog.md) — open document-understanding component inventory.

### Managed document AI

- [AWS Textract document layout](https://docs.aws.amazon.com/textract/latest/dg/how-it-works-document-layout.html) — layout response concepts.
- [AWS Textract Block API](https://docs.aws.amazon.com/textract/latest/APIReference/API_Block.html) — block types, relationships, geometry, confidence.
- [AWS Textract limits](https://docs.aws.amazon.com/textract/latest/dg/limits-document.html) — formats, pages, bytes, language and feature limits.
- [AWS Textract asynchronous processing](https://docs.aws.amazon.com/textract/latest/dg/async.html) — async job semantics.
- [AWS Textract asynchronous API protocol](https://docs.aws.amazon.com/textract/latest/dg/api-async.html) — request-token lifetime, result retention, output configuration, notification, and pagination semantics.
- [AWS Textract quota model](https://docs.aws.amazon.com/textract/latest/dg/limits-quotas-explained.html) — operation/region TPS, concurrent jobs, and adapter quota dimensions.
- [AWS Textract adapter inference](https://docs.aws.amazon.com/textract/latest/dg/textract-adapter-inference.html) — adapter ID/version use at inference.
- [Google Document AI Document model](https://docs.cloud.google.com/document-ai/docs/reference/rest/v1/Document) — anchors, pages, entities, geometry, language and confidence.
- [Google Document AI handle response](https://docs.cloud.google.com/document-ai/docs/handle-response) — traversal and normalized response use.
- [Google Document AI processor list](https://docs.cloud.google.com/document-ai/docs/processors-list) — processor/version/status and regional caveats.
- [Google Document AI evaluation](https://docs.cloud.google.com/document-ai/docs/evaluate) — precision/recall/F1 and confidence threshold evaluation.
- [Google Document AI quotas](https://docs.cloud.google.com/document-ai/quotas) — mutable quotas/limits.
- [Google Document AI processor-version management](https://docs.cloud.google.com/document-ai/docs/manage-processor-versions) — stable/RC lifecycle and version resources.
- [Google Document AI batch API](https://docs.cloud.google.com/document-ai/docs/reference/rest/v1/projects.locations.processors/batchProcess) — long-running batch operation and Cloud Storage output contract.
- [Google layout parser](https://docs.cloud.google.com/document-ai/docs/layout-parse-chunk) — layout-aware parsing/chunking behavior.
- [Azure Document Intelligence layout](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/prebuilt/layout) — layout and API-version baseline.
- [Azure Document Intelligence service limits](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/service-limits) — API lifecycle and service constraints.
- [Azure Document Intelligence transparency note](https://learn.microsoft.com/en-us/azure/foundry/responsible-ai/document-intelligence/transparency-note) — limitations, local evaluation, human review.
- [Azure Document Intelligence data, privacy, and security](https://learn.microsoft.com/en-us/azure/foundry/responsible-ai/document-intelligence/data-privacy-security) — regional processing, temporary result retention, and early deletion behavior.
- [Azure Document Intelligence containers](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/containers/install-run?view=doc-intel-4.0.0) — container/API/model support boundary.
- [Docling pipeline options](https://docling-project.github.io/docling/reference/pipeline_options/) — OCR/backend/model, acceleration, plugin, remote-service, timeout, and batching controls.

### Provenance, identity, schemas, and signatures

- [W3C PROV overview](https://www.w3.org/TR/prov-overview/) — provenance-family overview.
- [W3C PROV constraints](https://www.w3.org/TR/prov-constraints/) — validity/constraint model.
- [W3C Web Annotation Data Model](https://www.w3.org/TR/annotation-model/) — selectors and evidence annotations.
- [ALTO description](https://www.loc.gov/standards/alto/description.html) — OCR text/layout exchange concepts.
- [RFC 8493 BagIt](https://www.rfc-editor.org/rfc/rfc8493) — payload manifests and fixity.
- [JSON Schema Draft 2020-12 Core](https://json-schema.org/draft/2020-12/json-schema-core) — schema language.
- [JSON Schema Draft 2020-12 Validation](https://json-schema.org/draft/2020-12/json-schema-validation) — validation vocabulary.
- [ETSI EN 319 142-1 PAdES](https://www.etsi.org/deliver/etsi_EN/319100_319199/31914201/01.02.01_60/en_31914201v010201p.pdf) — PDF signature profile.
- [European Commission eSignature FAQ](https://ec.europa.eu/digital-building-blocks/sites/spaces/DIGITAL/pages/880312429/eSignature%2BFAQ) — EU signature terminology and context.

### File, model, privacy, and storage security

- [OWASP File Upload Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html) — layered upload defense.
- [Microsoft Office macro security](https://learn.microsoft.com/en-us/microsoft-365-apps/security/internet-macros-blocked) — macro behavior and enterprise controls.
- [NIST AI 100-2 E2025](https://csrc.nist.gov/pubs/ai/100/2/e2025/final) — adversarial ML terminology/taxonomy.
- [Indirect prompt injection paper](https://arxiv.org/abs/2302.12173) — document/web indirect-instruction threat.
- [OpenAI: Designing agents to resist prompt injection](https://openai.com/index/designing-agents-to-resist-prompt-injection/) — limit impact through authority/data-flow design.
- [GDPR consolidated text](https://eur-lex.europa.eu/eli/reg/2016/679/oj/eng) — EU personal-data legal baseline.
- [HHS HIPAA Security Rule](https://www.hhs.gov/hipaa/for-professionals/security/index.html) — US electronic protected-health-information safeguards.
- [PCI Security Standards document library](https://www.pcisecuritystandards.org/document_library/) — payment-card standard source.
- [NIST SP 800-53 Release 5.2.0](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final) — security/privacy control catalog.
- [NIST SP 800-88 Rev. 2](https://csrc.nist.gov/pubs/sp/800/88/r2/final) — media sanitization program guidance.
- [NIST Privacy Framework 1.0](https://www.nist.gov/privacy-framework/privacy-framework) — current final privacy-risk framework baseline; later drafts are not mislabeled final.
- [Amazon S3 Object Lock](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock.html) — version-level WORM retention/legal hold.
- [Azure immutable blob storage](https://learn.microsoft.com/en-us/azure/storage/blobs/immutable-storage-overview) — time retention/legal hold behavior.
- [Google Cloud Bucket Lock](https://docs.cloud.google.com/storage/docs/bucket-lock) — retention-policy locking behavior.

### Workflow, authorization, and external effects

- [Temporal Workflow Execution](https://docs.temporal.io/workflow-execution) — durable histories and execution.
- [Temporal Workflow Definition](https://docs.temporal.io/workflow-definition) — determinism/replay constraints.
- [Temporal Retry Policies](https://docs.temporal.io/encyclopedia/retry-policies) — retry model and defaults.
- [DBOS architecture](https://docs.dbos.dev/architecture) — durable execution/checkpoint model.
- [Stripe idempotent requests](https://docs.stripe.com/api/idempotent_requests) — semantic key and parameter consistency.
- [AWS Well-Architected idempotent operations](https://docs.aws.amazon.com/wellarchitected/latest/framework/rel_prevent_interaction_failure_idempotent.html) — safe mutating retries.
- [OWASP Transaction Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Transaction_Authorization_Cheat_Sheet.html) — bind authorization to material data and enforce server-side.

### Evaluation and observability

- [Google Document AI evaluation](https://docs.cloud.google.com/document-ai/docs/evaluate) — provider evaluation semantics.
- [On Calibration of Modern Neural Networks](https://proceedings.mlr.press/v70/guo17a.html) — confidence calibration.
- [Selective Classification via One-Sided Prediction](https://proceedings.mlr.press/v130/gangrade21a.html) — selective prediction/risk control.
- [NIST AI RMF Generative AI Profile](https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.600-1.pdf) — generative-AI lifecycle risks and actions.
- [NIST AI RMF core](https://airc.nist.gov/airmf-resources/airmf/5-sec-core/) — govern/map/measure/manage functions.
- [OpenTelemetry signals](https://opentelemetry.io/docs/concepts/signals/) — traces, metrics, logs, baggage context.
- [OpenTelemetry baggage](https://opentelemetry.io/docs/concepts/signals/baggage/) — propagation and disclosure risk.
- [OpenTelemetry context propagation](https://opentelemetry.io/docs/concepts/context-propagation/) — untrusted inbound propagation concerns.
- [FUNSD paper](https://arxiv.org/abs/1905.13538) — form understanding dataset.
- [DocLayNet paper](https://arxiv.org/abs/2206.01062) — diverse document layout dataset.
- [PubLayNet paper](https://arxiv.org/abs/1908.07836) — large scientific-layout dataset.
- [DocVQA paper](https://openaccess.thecvf.com/content/WACV2021/papers/Mathew_DocVQA_A_Dataset_for_VQA_on_Document_Images_WACV_2021_paper.pdf) — document visual QA dataset.
- [PubTables-1M paper](https://openaccess.thecvf.com/content/CVPR2022/papers/Smock_PubTables-1M_Towards_Comprehensive_Table_Extraction_From_Unstructured_Documents_CVPR_2022_paper.pdf) — table detection/structure dataset.
- [CORD repository](https://github.com/clovaai/cord) — receipt dataset.
- [SROIE paper](https://arxiv.org/abs/2103.10213) — scanned receipt OCR/information extraction dataset.
- [IAM Handwriting Database](https://fki.tic.heia-fr.ch/databases/iam-handwriting-database) — English handwriting corpus.

## Guide coverage map

| Research area | Guide |
|---|---|
| scope, authority, architecture, build/buy/language/runtime decisions | [Purpose, Architecture, and Technology Selection](../../agents/document-intelligence-agent/01-purpose-architecture-and-technology-selection.md) |
| admission, identity, deduplication, revision, provenance, originals/derivatives/storage | [Intake, Document Identity, Lineage, and Storage](../../agents/document-intelligence-agent/02-intake-document-identity-lineage-and-storage.md) |
| native text, OCR, layout, tables, images, handwriting, signatures, page completeness | [Document Understanding](../../agents/document-intelligence-agent/03-document-understanding-ocr-layout-tables-images-and-handwriting.md) |
| schemas, extraction, normalization, validation, confidence, calibration, review | [Classification, Extraction, Validation, Confidence, and Review](../../agents/document-intelligence-agent/04-classification-extraction-validation-confidence-and-review.md) |
| hostile files, prompt injection, tenant/purpose, providers, privacy, retention, holds, deletion | [Security, Privacy, Tenancy, Retention, and Deletion](../../agents/document-intelligence-agent/05-security-privacy-tenancy-retention-and-deletion.md) |
| durable states, page fan-out/join, approvals, effects, retries, unknown outcomes, reconciliation | [Workflow State, Approvals, Effects, and Recovery](../../agents/document-intelligence-agent/06-workflow-state-approvals-effects-and-recovery.md) |
| telemetry/audit, graders, corpora, public benchmarks, slices, failure injection, release gates | [Observability, Evaluation, and Failure Testing](../../agents/document-intelligence-agent/07-observability-evaluation-and-failure-testing.md) |
| latency, cost, caching, backpressure, topology, rollouts, incidents, roadmap | [Performance, Cost, Deployment, and Roadmap](../../agents/document-intelligence-agent/08-performance-cost-deployment-and-roadmap.md) |
| managed-provider and self-hosted route qualification, normalization, async completeness, lifecycle, and promotion | [Provider and Self-Hosted Adapter Qualification](../../agents/document-intelligence-agent/09-provider-and-self-hosted-adapter-qualification.md) |

## Refresh triggers

Refresh this packet and affected guide sections when any of the following occurs:

- a provider version is deprecated, promoted from preview, changes region/data-use behavior, or changes response schema;
- parser, PDF/Office renderer, OCR, scanner, workflow runtime, or model receives a material security/reliability release;
- a new document class, tenant tier, language/script, region, effect, or regulated data category enters scope;
- evaluation shows calibration drift, novel template/source/device shift, or critical-slice regression;
- a destination changes idempotency, lookup, receipt, finality, or reversal behavior;
- retention, legal-hold, signature, privacy, health, payment, or recordkeeping obligations change;
- an incident exposes an unmodeled parser, injection, tenant, deletion, workflow, or effect failure.

## Remaining limitations

- The packet synthesizes architecture and controls; it does not select a provider for a specific region, corpus, budget, or procurement regime.
- No public source can establish local extraction quality. A production-like lawful corpus and measured baseline remain mandatory.
- Regulatory sections identify engineering boundaries and evidence, not jurisdiction-specific legal advice or compliance certification.
- Some closed provider behaviors, destination finality rules, and data-deletion guarantees can only be verified through contract, deployment configuration, and acceptance tests.
- Emerging multimodal and agent-security techniques continue to change quickly. The durable authority, provenance, and evaluation boundaries are intentionally provider-independent.
