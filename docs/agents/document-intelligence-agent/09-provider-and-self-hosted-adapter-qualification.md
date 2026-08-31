# Provider and Self-Hosted Adapter Qualification

**Purpose:** Turn an OCR, layout, table, handwriting, or multimodal engine into a versioned production route whose capabilities, data boundary, failure semantics, and output evidence are measured rather than assumed.  
**Research baseline:** 2026-08-31

## An adapter is a product boundary

A vendor SDK call or Python model wrapper is not yet a production adapter. The adapter is the boundary that makes a changing engine behave like a stable application capability. It must answer four questions for every request:

1. **Was the exact intended artifact processed?**
2. **Was every expected page or region represented exactly once?**
3. **Can every normalized observation be traced to the raw engine response and exact engine configuration?**
4. **If the answer is uncertain, can the workflow stop or reconcile without silently accepting partial data?**

The application owns these answers. A provider's `SUCCEEDED` status means its job reached a provider-defined terminal state; it does not prove local page completeness, evidence quality, tenant policy, or suitability for a downstream decision.

## Start with a capability contract

Qualify a concrete route, not a product name. `Google Document AI`, `Amazon Textract`, `Azure Document Intelligence`, `Tesseract`, or `Docling` is too broad because APIs, processor/model versions, regions, features, limits, and runtime configurations differ.

```yaml
adapter_capability:
  route_id: invoice-layout-eu-v5
  adapter_release: doc-adapter/5.3.1+sha.19ab
  engine:
    product: example-document-ai
    api_version: "2024-11-30"
    processor_or_model: layout
    immutable_version: "layout-snapshot-2026-06"
    region_or_cell: eu-west
    release_stage: ga
  permitted_inputs:
    formats: [pdf, png, jpeg, tiff]
    max_bytes: 20971520
    max_pages_online: 10
    max_pages_batch: 500
    scripts: [Latn]
    content_classes: [invoice, credit_note]
  expected_outputs:
    page_identity: required
    tokens_and_polygons: required
    reading_order: measured
    tables: measured
    handwriting: unsupported
    raw_response_retained: true
  data_boundary:
    purpose: accounts_payable
    residency: eu
    provider_retention_evidence: assessment_2026_08_15
    training_or_product_improvement_use: prohibited_by_contract
  operational:
    deadline_ms: 90000
    max_attempts: 3
    idempotency_semantics: qualified
    result_lookup_window: P7D
    quota_profile: eu-prod-2026-08
```

Values in this record are deployment evidence with an owner, verification date, and refresh trigger. Do not copy illustrative limits from this guide into configuration.

## Qualification dossier

Require one dossier for every route and material version. A route is not eligible for production until each row is answered or explicitly marked unsupported.

| Area | Evidence to collect | Reject or constrain when |
|---|---|---|
| API and lifecycle | exact endpoint, API version, model/processor version, release maturity, retirement policy | only mutable aliases are available or retirement cannot be monitored |
| Inputs | MIME/signature behavior, byte/page/pixel limits, encryption, archives, office/PDF/image variants | the adapter cannot preflight or safely classify unsupported inputs |
| Page semantics | provider page numbering, page count, sparse pages, pagination, output chunking, rotation | the complete expected page set cannot be reconstructed |
| OCR | printed/handwritten distinction, language/script, vertical text, confidence meaning, geometry | required local slice is unsupported or uncalibrated |
| Layout and tables | reading order, spans, merged cells, selection marks, cross-page tables | normalization loses structures required by the schema |
| Evidence | raw payload, token/text anchors, polygons, coordinate unit/origin, normalized values | accepted fields cannot point to stable page evidence |
| Async protocol | submit identity, accepted/pending/terminal states, callbacks, polling, pagination, retention | retries can duplicate jobs or results can expire before durable capture |
| Data handling | region, temporary storage, logging, support access, retention/deletion, training use, subprocessors | tenant/purpose/legal policy cannot be met or independently evidenced |
| Security | authentication, least-privilege resource scope, network path, customer keys, remote fetch behavior | document content can reach business credentials or arbitrary network |
| Operations | quotas, rate/concurrency units, latency distribution, error taxonomy, maintenance | overload degrades into truncation or an unqualified fallback |
| Economics | billing unit, retry/fallback cost, review impact, storage/egress, minimum capacity | cost is unaffordable at peak/recovery load or cannot be attributed |
| Quality | lawful local corpus, critical slices, calibration, risk at coverage, downstream defects | vendor benchmark substitutes for local evidence |

Save the source links, contract/security artifacts, test report, sample raw payloads, and normalized golden outputs with the dossier. A procurement questionnaire alone is not runtime evidence.

## One normalized observation contract

Keep provider-native responses immutable, then normalize them into an application-owned intermediate representation. Normalization must be loss-aware: an unknown provider field is preserved or quarantined rather than silently discarded.

```yaml
adapter_result:
  adapter_result_id: adr_01K
  request_id: arq_01K
  artifact_id: art_01K
  artifact_sha256: "..."
  requested_page_ids: [pg_1, pg_2]
  returned_page_ids: [pg_1, pg_2]
  completeness: complete
  engine_manifest:
    adapter_release: doc-adapter/5.3.1+sha.19ab
    api_version: "2024-11-30"
    model_or_processor_version: "..."
    region: eu-west
    configuration_sha256: "..."
  provider_job:
    provider_request_id: "..."
    provider_job_id: "..."
    terminal_status: succeeded
    raw_response_artifact_ids: [raw_1, raw_2]
  observations:
    - observation_id: obs_17
      page_id: pg_2
      kind: word
      literal: "1,240.50"
      polygon_normalized: [[0.71, 0.82], [0.83, 0.82], [0.83, 0.85], [0.71, 0.85]]
      provider_native_ref: "raw_2#/blocks/184"
      provider_score_raw: 0.93
  warnings: []
```

The adapter must document coordinate origin, axis direction, rotation, unit, polygon order, and transform to the stored page rendition. Preserve provider-native identifiers only as provenance; application page and observation IDs remain authoritative locally.

## Page, rendition, and result completeness

Qualify completeness under normal, truncated, paginated, and partially failed responses.

```text
requested_page_set
  == provider_result_page_set
  == normalized_observation_page_set
  == durable_raw_response_page_set
```

Set equality is necessary but not sufficient. Also verify:

- the page ordinal maps to the intended source page and stored rendition;
- all response pages and blocks were consumed across pagination/chunks;
- provider warnings, skipped pages, limits, and partial success are first-class results;
- page rotation and coordinate transforms reproduce reviewer highlights;
- a blank page is explicitly present rather than absent;
- output objects and provider results were copied durably before their retrieval window expired;
- retries did not mix chunks from different jobs, versions, or source object generations.

For asynchronous APIs, persist the submit request digest and provider job ID before waiting. Callbacks are wake-up hints, not authoritative completion. Fetch status and every result page using the stored job identity, validate the manifest, and only then mark the activity complete.

Amazon Textract documents a seven-day idempotency-token lifetime for asynchronous start calls and a seven-day default retrieval window for asynchronously stored results. Its result APIs can paginate. Those are adapter concerns: the workflow must persist the token/job, reject changed parameters under the same semantic request, drain every result page, and capture raw output before expiry. Other providers have different semantics and must be qualified separately.

## Provider route playbooks

### Amazon Textract route

Record the exact API operation because text detection, document analysis, expense, lending, ID, and adapter-assisted analysis expose different features and limits. Qualification should cover:

- synchronous versus asynchronous input and page behavior;
- `ClientRequestToken`, parameter mismatch, token lifetime, job ID, SNS/SQS delivery, polling, and duplicate notifications;
- `NextToken` result pagination and maximum returned items;
- `OutputConfig`, bucket policy, KMS policy, default provider result retention, and durable capture;
- block type/relationship graph, geometry, entity types, selection marks, queries, signatures, and layout structures used by the selected operation;
- per-operation and per-region TPS/concurrent-job quotas;
- adapter ID and **adapter version** when custom adapters are used.

Do not convert the block graph directly into flat text. Retain block IDs/relationships and map them to page/token/table evidence. Validate that notification loss, duplicate notification, expired results, throttling, and partial pagination cannot create a locally complete result.

### Google Document AI route

Call a concrete processor-version resource rather than relying on the processor's mutable default when reproducibility matters. The route record should capture:

- processor type, immutable processor version, release channel/maturity, location, endpoint, and active schema version;
- online versus batch path, page and file limits for that exact processor, and project/processor/version quota dimensions;
- `Document` text anchors, page anchors, normalized vertices, detected languages, entities/properties, normalized values, and shard information;
- long-running operation identity, Cloud Storage input/output generations, and every JSON result shard for batch work;
- processor-version deployment state and version-specific region availability;
- whether a generative processor uses a global underlying endpoint even when invoked from a regional Document AI endpoint.

Official processor documentation can list stable and release-candidate versions together, and some generative versions explicitly warn that a global Gemini endpoint does not meet data-residency standards. Release maturity and actual processing path therefore belong in authorization and provenance, not only in an architecture diagram.

### Azure Document Intelligence route

Pin the API version and exact model/custom model identifier. Qualification should test:

- analysis submit, operation-location/result identity, status polling, warnings/errors, result deletion, and expiry behavior;
- spans, pages, polygons, tables, selection marks, handwriting/style, add-on features, and output format used by the selected API/model;
- service resource region, network/private-endpoint posture, identity method, customer storage for training/batch paths, and custom-model lifecycle;
- supported feature differences between cloud API and containers;
- default quotas and successfully granted deployment quotas rather than requested quotas;
- model/API retirement notices and migration compatibility.

Current Microsoft documentation states that analysis inputs and results are temporarily stored in the resource region and automatically deleted after 24 hours, with an API to delete an analysis result earlier in v4.0. Treat this as dated provider evidence: the application still persists required raw results in its own governed store, records deletion attempts where policy requires them, and re-verifies the behavior before release.

## Self-hosted route playbook

Self-hosting changes the trust and operations boundary; it does not make the engine deterministic or automatically private.

### Reproducible package

Pin and inventory:

- source/package/container digest and dependency lock;
- OCR language/model data digests and licenses;
- parser/render/OCR/layout/table/VLM model weights and configuration;
- runtime, native libraries, drivers, accelerator type, precision/quantization, and deterministic flags where available;
- external plugin loading and remote-service settings;
- model cache/download origin, integrity verification, and offline-start behavior;
- SBOM, vulnerability status, patch owner, and end-of-support policy.

Tesseract's code version is only part of its behavior; language data, page segmentation, preprocessing, fonts/scripts, and wrapper versions also affect output. Docling exposes several OCR engines, PDF backends, layout/table options, acceleration settings, external plugins, and remote-service controls. Pin those settings instead of recording only `Docling`.

### Isolation profile

Run untrusted parsing/rendering in a disposable cell with no business credentials and no general outbound network. Self-hosted model workers receive only admitted page renditions or crops, not arbitrary archives or office packages. Disable optional remote services and plugin loading unless a route explicitly qualifies them. Mount model artifacts read-only; use a separate writable scratch volume with byte/inode/time limits; destroy it after the attempt.

GPU access expands the attack and availability surface. Separate hostile parser workers from GPU/model workers, validate image dimensions and decoded-pixel budgets before GPU admission, cap batch and generated-token sizes, and reset unhealthy workers after out-of-memory or device errors.

### Capacity and quality profile

Benchmark production-like distributions, not a single warm image:

| Dimension | Measure |
|---|---|
| cold start | model download disabled, cache present/missing, worker boot and readiness |
| throughput | pages/second by DPI, pixel band, language, OCR/layout/table route and batch size |
| tails | p95/p99 service time, memory/GPU high-water mark, timeout and OOM rate |
| concurrency | safe per-worker batches, queue delay, noisy-tenant isolation, cancellation latency |
| quality | field/table/layout metrics and calibration by critical slice and hardware/runtime profile |
| recovery | worker crash, node loss, model-cache loss, accelerator degradation, replay storm |
| cost | compute, idle headroom, storage, engineering/on-call, review and downstream defects |

Do not promote a GPU/precision/runtime change on throughput alone. Numerical or preprocessing changes can move OCR/layout quality and therefore review load and business risk.

## Adapter state and error taxonomy

Normalize transport and provider outcomes into operational meanings. Do not let the workflow branch on vendor exception strings.

| Result | Required meaning | Runtime behavior |
|---|---|---|
| `rejected_before_acceptance` | provider/local worker proves no job began | correct request or fail; retry only if policy permits |
| `accepted_pending` | durable remote/local job identity exists | wait/poll under deadline; do not resubmit |
| `succeeded_complete` | terminal job plus every expected result captured and validated | publish immutable adapter result |
| `succeeded_partial` | provider says success but pages/blocks/shards/warnings are incomplete | block acceptance; fetch missing result or use qualified fallback |
| `transient_no_acceptance` | no job accepted and retry is safe | retry same semantic request within budget |
| `unknown_acceptance` | timeout/crash occurred across submit boundary | reconcile by request token/job/output before resubmit |
| `definitive_failure` | terminal nonretryable engine outcome | typed exception or review |
| `contract_drift` | response violates pinned schema/invariants | quarantine raw response, disable route/canary, investigate |
| `policy_mismatch` | region, version, feature, or data handling is no longer eligible | fail closed; never downgrade policy |

Every attempt records deadline, retry reason, request digest, bytes/pages billed or processed, provider IDs, raw response artifacts, normalized result, and terminal classification. Provider `4xx` and `5xx` categories are hints; qualification determines whether the request was accepted and whether retry can duplicate work.

## Qualification test suite

### Conformance corpus

Maintain small deterministic fixtures that assert adapter mechanics:

- one-page native PDF, scan, image, multipage TIFF, and supported office/container input;
- blank page, rotated page, mixed page sizes, duplicate ordinal, and missing result page;
- result pagination and output sharding at exact boundaries;
- tables with merged cells and page continuation;
- printed/handwritten mixture and multiple declared scripts;
- malformed and unknown response fields;
- warning-only, partial, timeout, quota, cancelled, and terminal error results;
- source object overwritten between submit and fetch;
- callback duplicated, delayed, forged, or lost;
- provider result expires before a delayed workflow resumes.

### Quality and decision corpus

Use lawful local documents with source/time/template/device holdouts. Compare raw and normalized outputs at token, line, region, reading-order, table, field, and evidence levels. Calibrate acceptance separately per route and fallback. A fallback is not interchangeable because both implement `extract()`.

### Security and privacy suite

Test cross-tenant IDs, wrong resource region, stolen callback identifiers, remote URLs, redirects, oversized images, decompression bombs, polyglots, embedded objects, active content, prompt injection in every content channel, output containing control strings, plugin/model artifact tampering, and deletion/retention propagation. Verify ordinary traces contain identifiers and status—not document bodies, raw tokens, credentials, signed URLs, or provider payloads.

### Failure and recovery suite

Inject failures before submit, after remote acceptance, during polling, between paginated result reads, after raw-result storage, during normalization, and before local completion commit. Assert one semantic job or an explicit reconciled duplicate, exact page-set completeness, durable raw payloads, and no acceptance based on a callback alone.

## Promotion, shadowing, and rollback

An adapter release is part of the document behavior bundle. Promotion requires:

1. updated capability dossier and primary-source review;
2. conformance, security, quality, calibration, load, and recovery reports;
3. normalization diff against the current route, including unknown fields and lost information;
4. offline replay, then effect-free shadow traffic under separate quotas;
5. canary by named tenant/source/document-class cohorts;
6. capacity and reviewer headroom for the observed fallback/review shift;
7. an available rollback route and known treatment for already accepted candidate outputs.

Rollback changes new routing. It does not erase accepted results. Identify affected runs by exact adapter/engine/configuration manifest, freeze unsafe automatic acceptance or effects, then create versioned reprocessing and downstream correction plans.

## Refresh triggers

Re-qualify when any of these change:

- API, SDK, processor/model, release stage, mutable alias, response schema, feature, region, endpoint, quota, retention, deletion, data-use, or price;
- parser, renderer, OCR engine, language/model data, layout/table model, runtime, driver, accelerator, precision, plugin, or remote-service setting;
- local source/template/language/device distribution, error cost, schema, critical field, authority tier, or residency/retention policy;
- provider incident, security advisory, contract change, unexplained calibration drift, completeness failure, or downstream defect cohort.

Automate documentation/contract monitors where possible, but a change alert opens review; it does not auto-promote a new route.

## Production checklist

- [ ] The route is identified by exact adapter, API, engine/model/processor, configuration, region/cell, and release stage.
- [ ] Raw responses and normalization lineage are durable and deletion-aware.
- [ ] Requested, returned, normalized, and stored page sets reconcile exactly.
- [ ] Async submit, lookup, pagination, callbacks, result expiry, and retries have tested semantics.
- [ ] Data residency, temporary retention, early deletion, training use, logging, and support access have dated evidence.
- [ ] Self-hosted weights, language data, dependencies, hardware, plugins, and remote-service controls are pinned.
- [ ] Provider scores are calibrated locally and never treated as portable correctness probabilities.
- [ ] Fallback routes satisfy the same policy and are evaluated as separate populations.
- [ ] Contract drift fails closed and can disable only the affected route.
- [ ] Promotion includes conformance, safety, quality, cost, capacity, shadow, canary, and rollback evidence.

## Sources

- [Amazon Textract asynchronous operations](https://docs.aws.amazon.com/textract/latest/dg/api-async.html)
- [Amazon Textract quotas and concurrent jobs](https://docs.aws.amazon.com/textract/latest/dg/limits-quotas-explained.html)
- [Amazon Textract document limits](https://docs.aws.amazon.com/textract/latest/dg/limits-document.html)
- [Amazon Textract adapter inference](https://docs.aws.amazon.com/textract/latest/dg/textract-adapter-inference.html)
- [Google Document AI processor catalog and versions](https://docs.cloud.google.com/document-ai/docs/processors-list)
- [Google Document AI processor-version management](https://docs.cloud.google.com/document-ai/docs/manage-processor-versions)
- [Google Document AI REST resources](https://docs.cloud.google.com/document-ai/docs/reference/rest)
- [Google Document AI batch processing](https://docs.cloud.google.com/document-ai/docs/reference/rest/v1/projects.locations.processors/batchProcess)
- [Google Document AI quotas](https://docs.cloud.google.com/document-ai/quotas)
- [Azure Document Intelligence service limits](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/service-limits)
- [Azure Document Intelligence data, privacy, and security](https://learn.microsoft.com/en-us/azure/foundry/responsible-ai/document-intelligence/data-privacy-security)
- [Azure Document Intelligence documentation and API lifecycle](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/?view=doc-intel-4.0.0)
- [Azure Document Intelligence containers](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/containers/install-run?view=doc-intel-4.0.0)
- [Tesseract OCR repository](https://github.com/tesseract-ocr/tesseract)
- [Docling pipeline options](https://docling-project.github.io/docling/reference/pipeline_options/)

