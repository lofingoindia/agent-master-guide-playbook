# Scientific Research and Laboratory Operations Agent Blueprint — Research Packet

> **Research status:** production-depth synthesis complete; Pass 2 adapter and worked-flow refinement included  
> **Research cut-off:** 2026-08-31  
> **Primary artifact:** [Scientific Research and Laboratory Operations Agent Blueprint](../../agents/scientific-research-agent/README.md)  
> **Claim level:** engineering design guidance, not a laboratory validation, safety determination, legal opinion, or compliance certification

This packet records the evidence, version checks, contradictions, design decisions, and unresolved limitations used to build the scientific-research-agent area. It exists so future maintainers can distinguish a sourced design choice from a generic agent pattern or unsupported assumption.

## 1. Research scope

The research asked how to build a non-clinical agent that can coordinate computational research and bounded laboratory operations while preserving:

- hypothesis, experiment, protocol, and analysis state;
- ELN/LIMS/instrument/scheduler/repository source-of-truth boundaries;
- sample, material, observation, artifact, and decision provenance;
- metrology, uncertainty, reproducibility, and result limitations;
- safe, idempotent, and recoverable digital and physical effects;
- context/memory integrity and prompt-injection resistance;
- safety, biosecurity, OT security, data governance, and electronic-record scope;
- production deployment, tenancy, queuing, backpressure, observability, incidents, cost, and change control; and
- trajectory, artifact, scientific, security, and failure-recovery evaluation.
- operation-level qualification for representative ELN/LIMS/SDMS, robotics, HPC, notebook/repository, object-storage, identity, safety, and publication-handoff adapters.

Explicit exclusions were literature-only Deep Research, clinical-trial operations, unrestricted wet-lab execution, agent-owned interpretation, electronic signature, authorship, and publication approval.

## 2. Repository context reviewed before research

The blueprint was aligned to the repository's category and staged-program contracts and the canonical guides below:

- [Execution boundaries](../../runtime/execution-boundaries.md)
- [Durable execution and resume](../../runtime/durable-execution.md)
- [Run controls](../../runtime/run-controls.md)
- [Agent state and event contract](../../runtime/agent-state-and-event-contracts.md)
- [Tool contracts](../../tools/tool-contracts.md) and [tool-result contracts](../../tools/tool-results-artifacts-and-provenance.md)
- [Context engineering](../../context-memory/context-engineering.md), [memory architecture](../../context-memory/memory-architecture.md), and [compaction strategies](../../context-memory/compaction-and-continuity.md)
- [Agent threat model](../../security/agent-threat-model.md), [prompt-injection defense](../../security/prompt-injection-and-untrusted-data.md), and [permissions and secrets](../../security/permissions-sandboxing-and-secrets.md)
- [Idempotency, retries, and effect safety](../../reliability/idempotency-and-side-effects.md) and [failure taxonomy](../../reliability/failure-taxonomy.md)
- [Evaluation-driven development](../../evaluation/evaluation-driven-development.md), [trajectory evaluation](../../evaluation/trajectory-and-reliability-evaluation.md), and [observability](../../evaluation/observability-and-tracing.md)
- [Queue architecture](../../operations/queues-scheduling-and-backpressure.md), [deployment](../../operations/deployment-release-and-incident-response.md), [scaling](../../operations/scaling-capacity-and-slos.md), and [cost engineering](../../operations/model-routing-cost-and-latency.md)

Comparable blueprint areas for analytics, data pipelines, MLOps/model operations, QA, and Deep Research were reviewed for structure and boundary discipline. No shared index or registry was edited for this delegated task.

## 3. Method

### 3.1 Source hierarchy

Research prioritized:

1. standards bodies, regulators, public-health/safety institutions, and official specifications;
2. official software, platform, and vendor documentation;
3. peer-reviewed primary research and corrections;
4. official benchmark papers and repositories; and
5. review/consensus works where the field needed a definition or limitation.

Secondary summaries were not used to establish technical, legal, safety, or version claims when a primary source was available.

### 3.2 Research angles

| Angle | Questions investigated |
|---|---|
| Laboratory interoperability | Which standards cover commands, device information, analytical data, study metadata, and provenance? Where do they stop? |
| Source systems | How should ELN/LIMS events, API versions, signed records, and source projections behave? |
| Samples and data | How should physical-sample identity, lineage, native raw data, normalization, and repository deposit work? |
| Measurement | What must accompany a value/unit, uncertainty, calibration, and traceability claim? |
| Compute | What do schedulers/workflow specifications guarantee, and what remains scientific validation? |
| Safety and governance | Which controls must remain independent, and which current policy sources are scoped, draft, or changing? |
| Agent evidence | What have autonomous laboratories and science-agent benchmarks actually demonstrated? |
| Production operations | Which canonical agent controls need workload-specific extensions for physical effects and scarce research resources? |

### 3.3 Synthesis rule

A source was used only for the scope it supports. For example:

- a communication standard did not become a sample-lineage model;
- a job state did not become a scientific result;
- a calibrated instrument did not make every result metrologically traceable;
- FAIR did not become a compliance badge;
- a benchmark did not become evidence of physical safety; and
- a successful domain demonstration did not become a claim of general autonomous-laboratory reliability.

## 4. Executive findings

1. **The useful system is a research-operations control plane, not an autonomous scientist.** Model proposals must remain inside deterministic protocol, policy, identity, unit, effect, budget, and safety boundaries.
2. **No single standard covers the whole laboratory.** SiLA and OPC UA LADS address laboratory device/service interoperability; Allotrope/AnIML address analytical data; ISA can describe investigation/study/assay structure; W3C PROV and RO-Crate support provenance and packaging; domain profiles remain necessary.
3. **Native raw data must be preserved.** Normalization can improve portability but can lose method, audit, precision, or vendor semantics. Store converter versions and loss reports.
4. **The source systems remain authoritative.** ELN/LIMS events can be delayed or out of order; treat them as invalidation hints, refetch the current authorized object, and use version-aware projection updates.
5. **Physical effects are not generic API calls.** A lost response after start requires `UNKNOWN`/`INDETERMINATE` and reconciliation, never a blind retry.
6. **Operational completion is not scientific validity.** Slurm/Kubernetes/process state is followed by artifact, domain, control, lineage, and evidence checks.
7. **Measurement uncertainty and traceability are claims about results.** Instrument calibration alone is insufficient; method, calibration chain, uncertainty contributions, conditions, and raw observations matter.
8. **Reproducibility and replicability differ.** The blueprint distinguishes same-data/code re-execution from independent new-data replication and does not interpret failure to replicate as automatic misconduct.
9. **Safety must be independent and case-specific.** Protocol-driven facility risk assessment, controller limits, hardware interlocks, containment, emergency stops, and qualified operators cannot be delegated to a model.
10. **Current biosecurity policy is a moving boundary.** A July 2026 U.S. high-risk life-science policy is current for its federal scope, while an August 2026 NIH biosafety proposal is draft and not effective. Implement refresh triggers rather than hardcoding a summary.
11. **Electronic-record frameworks have scope.** Part 11, GLP, CGMP, OECD, and MHRA sources do not make a generic agent compliant; regulated intended use needs separate validation and predicate-rule analysis.
12. **Public benchmarks and demonstrations are narrow and mutable.** Benchmark labels have been corrected and an autonomous-lab paper received a material author correction. Release evidence must be local, versioned, trajectory-based, and correction-aware.
13. **A vendor connector is not an authority unit.** Read, draft, submit, cancel, physical start, deposit, and publish operations need separate effective-principal, target, version, ambiguity, and recovery qualification.
14. **Storage and catalogue success are layered.** Metadata registration, byte upload, multipart finalization, checksum validation, authorized read-back, result-package eligibility, and publication are distinct states.
15. **Recovery capacity is part of safety and integrity.** Reconciliation, buffered events, raw artifacts, expired readiness and source corrections return together after an outage; new physical admission waits for convergence and facility re-entry checks.

## 5. Decision record

| ID | Blueprint decision | Evidence basis | Important limit |
|---|---|---|---|
| SR-01 | Keep model, deterministic policy, tool gateway, and durable session state separate | Repository runtime/security contracts; physical-effect analysis | Separation must be implemented, not merely drawn |
| SR-02 | Keep ELN, LIMS, controller, scheduler, code registry, and artifact repository authoritative for their domains | Vendor/API semantics and record-integrity guidance | A source can still be wrong; corrections append with provenance |
| SR-03 | Store observation, evidence, hypothesis, decision, digital effect, and physical effect as different records | PROV concepts, scientific reproducibility definitions, effect safety | Domain profiles must define actual observation/evidence semantics |
| SR-04 | Version and hash hypothesis, protocol, analysis plan, run manifest, policy, adapter, and environment | Reproducibility/provenance sources and correction evidence | Hashing needs canonical serialization and does not prove correctness |
| SR-05 | Use official APIs/events/exports; no direct source database writes | Official vendor/source contracts | Some vendor APIs lack required semantics; constrain or reject them |
| SR-06 | Treat webhook/event payloads as hints and refetch current state | Benchling official event guidance documents late/out-of-order delivery | Other sources require their own contract verification |
| SR-07 | Use standards through tested versioned profiles rather than one universal lab schema | SiLA, LADS, Allotrope, AnIML, ISA scopes differ | Local vendor/technique coverage remains empirical |
| SR-08 | Preserve native raw data alongside normalized representations | Analytical-format scope and data-integrity/provenance needs | Native format longevity can require preservation tooling |
| SR-09 | Keep LIMS sample identity authoritative; optionally link external physical-sample PIDs | DataCite IGSN guidance | IGSN is not a digital-data identifier or full LIMS |
| SR-10 | Use W3C PROV semantic core and export a version-pinned RO-Crate/profile where useful | W3C PROV, RO-Crate, Workflow Run RO-Crate | Profile/base-version compatibility must be tested |
| SR-11 | Perform unit conversions deterministically and preserve source representation | UCUM and metrology guidance | UCUM does not define every domain measurand or arbitrary unit |
| SR-12 | Never invent uncertainty or claim traceability from calibration status alone | BIPM/JCGM and NIST traceability guidance | Qualified domain/metrology review may still be required |
| SR-13 | Distinguish exploratory, confirmatory, and replication workflows and seal confirmatory decisions before outcome access | National Academies and NIH rigor guidance | Exact design rules vary by discipline |
| SR-14 | Let existing Slurm/Kubernetes/workflow infrastructure schedule compute; validate outputs separately | Official scheduler/job/workflow docs | Engine/API behavior and version still need local tests |
| SR-15 | Pin code, environment, workflow, seeds, numerical tolerances, hardware-sensitive dependencies, and outputs | CWL/Workflow Run RO-Crate plus reproducibility definitions | Bitwise reproducibility is not always possible or scientifically necessary |
| SR-16 | Persist effect intent before dispatch and reconcile unknown outcomes; never blind-retry physical commits | Canonical effect-safety contract plus controller semantics | Reconciliation evidence depends on target capability |
| SR-17 | Keep hardware interlocks, E-stop, containment, and operators independent of the agent/cloud | OSHA, BMBL, WHO, NIST OT guidance | The blueprint does not perform a facility hazard analysis |
| SR-18 | Default to digital-only; enable supervised low-risk physical execution only after Stage 4 gates | Safety sources and autonomous-lab evidence limitations | “Low risk” is a local human determination |
| SR-19 | Hard-stop hazardous, high-consequence, novel/unknown-risk, or potentially prohibited biological work | Current NIH/USG/CDC/WHO sources | Jurisdiction/institution-specific interpretation is required |
| SR-20 | Treat external scientific content as untrusted data with no authority to select tools or policy | Repository prompt-injection/security guidance | Content filtering alone is insufficient |
| SR-21 | Disable user and automatic long-term semantic memory by default; curate failure knowledge | Context/memory and scientific correction needs | A governed corpus can still become stale |
| SR-22 | Separate execution approval, scientific interpretation, signature, and repository/publication release | Record-integrity and governance sources | Organizational role mapping is local |
| SR-23 | Evaluate full trajectories, effects, artifacts, security, and recovery; use benchmarks only as supplements | Benchmark scope/corrections and repository eval guidance | Expert rubrics and fixtures require maintenance |
| SR-24 | Measure total cost per valid result, including samples, materials, instruments, staff, recovery, storage, and model use | Self-driving-lab metrics plus operations economics | Opportunity cost may be uncertain and must be labeled |
| SR-25 | Use stages 0–6 to earn capabilities; authority expansion restarts relevant earlier validation | Repository registry and physical-risk evidence | Stage number is not a compliance or safety certification |
| SR-26 | Qualify exact operations with dated, expiring capability manifests instead of certifying vendor-wide connectors | Benchling, Slurm, Opentrons, GitHub, S3, Zenodo and source-specific version/effect semantics | Documentation does not replace target-tenant/account/facility fault tests |
| SR-27 | Bind readiness and evidence to exact project, sample/material, instrument, protocol, analysis, measurement, artifact, and effect identities | LIMS/provenance/effect sources plus provider object/version semantics | Identifier availability and merge/tombstone behavior remain deployment-specific |
| SR-28 | Keep exactly seven memory lifetimes and make compaction omissions loss-aware and recoverable | Canonical context/memory guidance plus long-run safety/effect continuity analysis | Provider-hidden state remains an implementation cache, not durable authority |
| SR-29 | Treat publication as a separately scoped effect; draft/deposit rights do not imply human release authority | Zenodo scope separation, DataCite versioned metadata and repository governance | Each repository has different PID, embargo, correction, withdrawal and access behavior |
| SR-30 | Restore by consequence and reconcile before admitting new physical work | Queue/DR guidance plus facility gateway, raw-artifact and unknown-effect analysis | RTO/RPO and safe re-entry evidence must be technique/facility specific |

## 6. Contradictions, version drift, and resolutions

### 6.1 SiLA 2 and OPC UA LADS

**Finding:** Both are official laboratory interoperability efforts, but they are not interchangeable universal solutions. SiLA 2 describes features, commands, and properties over its specified communication foundation. OPC UA LADS defines a device-agnostic information model with hardware and functional views. The first LADS part explicitly leaves areas such as common dictionaries and richer sample/consumable treatment to later work.

**Resolution:** The blueprint names neither as a mandatory canonical interface. Select the interface supported and qualified for the target device, pin the exact standard/profile/version, retain the vendor-native semantics, and build a narrow adapter.

### 6.2 Analytical data formats

**Finding:** Allotrope ADF/ADM/AFO, Allotrope Simple Model, AnIML, ASTM ANDI/netCDF, and technique-specific formats overlap but differ in scope, maturity, accessibility, implementation, and vendor support. ASTM E1947-98(2022), for example, is specifically chromatographic data interchange, not a universal modern laboratory format.

**Resolution:** Preserve native raw. Normalize only through a tested technique profile, version the converter, validate conformance and loss, and retain both representations.

### 6.3 RO-Crate version mismatch

**Finding:** Base RO-Crate 1.3 is the current long-term release at the research date. Workflow Run RO-Crate profile 0.5 documentation still references RO-Crate 1.1 concepts.

**Resolution:** Pin both the base and profile versions and run compatibility validation. Do not claim “RO-Crate compliant” without specifying the exact profile/version and validator evidence.

### 6.4 FAIR principles versus implementation

**Finding:** The original FAIR paper defines guiding principles, not one data model, repository, security approach, or compliance program.

**Resolution:** Report concrete PID, metadata, provenance, access, format, license, validation, retention, and stewardship properties instead of a FAIR badge.

### 6.5 ELN event versus current record

**Finding:** Official Benchling event documentation says events may arrive late or out of order and their payload may not reflect the latest state; consumers should fetch the current object. This is a useful concrete warning, not proof every ELN behaves identically.

**Resolution:** Make “verify event, deduplicate, refetch, compare-and-set projection” the default integration pattern, then tighten it only when another source contract proves stronger semantics.

### 6.6 Scheduler completion versus result validity

**Finding:** Slurm, Kubernetes Jobs, and workflow engines report execution states. Slurm `COMPLETED` means the process completed with exit code zero; it does not inspect scientific controls, output meaning, or provenance.

**Resolution:** Maintain admission, scheduling, process, artifact, domain, and scientific eligibility layers. A scheduler receipt is an observation.

### 6.7 Measurement guidance changes

**Finding:** JCGM 100:2008 remains foundational, while the GUM family has later supplements and a 2026 amendment concerning measurement-model nonlinearity. The SI Brochure also has current updates.

**Resolution:** Pin the uncertainty procedure and SI/units references actually used. The blueprint avoids embedding one formula or claiming universal uncertainty coverage.

### 6.8 Reproducibility terminology

**Finding:** “Reproducibility” is used inconsistently across fields. The National Academies report uses reproducibility for the same data/code/conditions and replicability for new data addressing the same question.

**Resolution:** Adopt those definitions in this blueprint and always state the exact achieved comparison: re-executable, artifact-identical, numerically equivalent, scientifically consistent, or independent replication.

### 6.9 Biosafety and biosecurity policy transition

**Finding:** The April 2024 NIH Guidelines remain a current referenced policy for their scope. A July 2026 U.S. policy introduced prohibitions and restrictions on defined high-risk life-science research. An August 2026 draft NIH Biosafety Policy proposes replacing the NIH Guidelines but was not effective and remained in comment at the research date.

**Resolution:** The agent hard-stops unknown/flagged work and routes to institutional safety/legal authorities. Production policy uses authoritative effective-date feeds and named owners; the draft is a change-monitoring input only.

### 6.10 NIST OT revision status

**Finding:** NIST SP 800-82 Rev. 3 is final. Rev. 4 activity was pre-draft at the research date.

**Resolution:** Cite and pin Rev. 3 for the blueprint; monitor Rev. 4 without presenting it as final.

### 6.11 A-Lab claims and 2026 correction

**Finding:** The original A-Lab publication's novelty-related claims were corrected in January 2026. The author correction reports manual reanalysis of characterized products, changes the success characterization to 36 of 40 with four inconclusive, and addresses a training-data overlap.

**Resolution:** Autonomous-lab output is not accepted from orchestration success or automated characterization alone. Require independent characterization, raw-artifact review, versioned claim status, and correction monitoring.

### 6.12 Benchmark label drift

**Finding:** ScienceAgentBench's repository added a verified split after identifying false-negative evaluation labels. Its 102 tasks are code/data-driven tasks derived from papers, not physical laboratory operations. LAB-Bench is a multiple-choice practical biology benchmark.

**Resolution:** Pin benchmark commit/split, record limitations, and use public scores only as supplemental evidence. Local trajectory/effect/safety/recovery suites are mandatory.

### 6.13 Electronic-record scope

**Finding:** FDA Part 11 guidance is tied to records required by predicate rules. FDA CGMP data-integrity, OECD GLP, and MHRA GxP guidance likewise have bounded regulated scopes.

**Resolution:** Adopt useful integrity principles generally but never label the blueprint Part 11/GxP/GLP compliant. A regulated deployment performs separate intended-use, predicate-rule, validation, audit, and change-control work.

### 6.14 Benchling V3 documentation drift

**Finding:** The official V3 overview describes alpha/beta access and early-access limitations, while a newer official changelog describes migration to a unified `/v3/` path, a header for beta operations, and different stability notice behavior. Public and tenant-specific documentation can therefore expose different transition snapshots.

**Resolution:** Do not encode a generic “Benchling V3” assumption. Record the exact tenant, route, endpoint stability, schema, app version and observed response contract; subscribe to the changelog and recertify before migration. Prefer the highest stable operation available for critical workflows.

### 6.15 Object identity versus integrity

**Finding:** Amazon S3 documents explicit checksums and conditional operations, but an ETag is not a universal full-object checksum, especially across multipart and other storage cases. A scientific catalogue can also hold metadata and file references while bytes live in another storage system.

**Resolution:** Preserve catalogue/dataset identity, storage object/version identity, expected byte length, declared checksum algorithm/type/value, finalization state and authorized read-back independently. No single `200`, ETag, PID or catalogue row proves the evidence package is intact.

### 6.16 Simulation and robotics version surfaces

**Finding:** The official Slurm REST reference at cut-off reports Slurm 26.05.3 with API paths using `v0.0.45`. Opentrons documents Protocol API 2.29 with distinct supported ranges for Flex and OT-2 and separates protocol API from robot software/app versions. GitHub likewise has an explicit date-versioned REST surface.

**Resolution:** Pin provider server/product, API/profile, SDK/client, adapter and relevant hardware/firmware versions as a compatibility set. “Latest,” a successful offline simulation, or an unversioned request is not production evidence.

## 7. Annotated source register

Source strength labels used here:

- **Normative/official:** standard, regulator, government, or standards-body publication.
- **Official implementation:** official product/project specification, repository, or vendor documentation.
- **Primary research:** peer-reviewed paper, author correction, or benchmark paper/repository.

The “blueprint use” column deliberately states limitations as well as useful facts.

### 7.1 Laboratory device and analytical-data interoperability

| Source | Version/status checked | Blueprint use and limit |
|---|---|---|
| [SiLA standards](https://sila-standard.com/standards/) and [downloads](https://sila-standard.com/downloads/) | Official site; SiLA 2 current family, SiLA 1.x marked obsolete; Part A v1.1 available | Features, commands, and properties can shape adapter profiles. Do not assume device support or use obsolete SiLA 1.x semantics. |
| [SiLA 2 Part A v1.1](https://sila-standard.com/wp-content/uploads/2022/03/SiLA-2-Part-A-Overview-Concepts-and-Core-Specification-v1.1.pdf) | Official specification PDF, v1.1 | Foundation for exact capability/version profiles; not a full research-data or safety model. |
| [OPC UA LADS Part 1](https://reference.opcfoundation.org/specs/OPC-30500-1) | Official online reference; 1.0.1 release listed 2024-01-18 | Device-agnostic analyzer/device model with hardware and functional views. Initial part leaves some dictionaries, PubSub, sample, and consumable topics for future work. |
| [OPC UA LADS profile](https://profiles.opcfoundation.org/document/37) | Official OPC Foundation profile | Use to verify claimed conformance components; conformance still needs the target product/profile/version. |
| [Allotrope Framework technical reports](https://docs.allotrope.org/) | Official framework docs; ADF v1.5.3 pages observed | ADF, ADM, and AFO provide analytical-data format/model/ontology components. Access, implementation, and exact technique coverage need local assessment. |
| [Allotrope Data Format v1.5.3](https://docs.allotrope.org/Allotrope%20Data%20Format.html) | Official technical report, v1.5.3 | Supports analytical data, contextual metadata, data package/cube/description, audit/checksum concepts. Does not justify discarding vendor-native raw. |
| [AnIML overview](https://animl.org/overview) and [current schemas](https://www.animl.org/current-schema) | Official project site; current schemas linked to project repository | Core and technique schemas plus technique definitions can support XML analytical-data exchange. The site describes AnIML as emerging; check exact ASTM and technique-definition status rather than claiming universal adoption. |
| [ASTM E1947-98(2022)](https://store.astm.org/standards/e1947) | Active, last updated 2022 | Chromatographic data interchange using the specified protocol/netCDF family. Its scope is chromatography, not all analytical or laboratory data. |
| [ISA specification](https://isa-tools.org/format/specification.html) | ISA model specification 1.0, published 2016 | Investigation–Study–Assay structure and ISA-Tab/ISA-JSON can help life-science metadata profiles. It is not a universal instrument/run schema. |

### 7.2 Provenance, packaging, identifiers, and repositories

| Source | Version/status checked | Blueprint use and limit |
|---|---|---|
| [W3C PROV overview](https://www.w3.org/TR/prov-overview/) and [PROV-DM](https://www.w3.org/TR/prov-dm/) | W3C Recommendations | Entity/activity/agent relationships provide a stable provenance semantic core. Domain details, access, and packaging remain separate. |
| [RO-Crate specification](https://www.researchobject.org/ro-crate/specification) | Base specification 1.3 current long-term release at cut-off | Result/research-object packaging with JSON-LD metadata. Pin version/profile and validate; a crate does not prove scientific completeness. |
| [Workflow Run RO-Crate](https://www.researchobject.org/workflow-run-crate/) and [profile 0.5](https://www.researchobject.org/workflow-run-crate/profiles/workflow_run_crate/) | Profile 0.5; documentation references RO-Crate 1.1 | Useful for workflow inputs, outputs, code, environment, and run provenance. Test the documented base/profile version mismatch. |
| [CWLProv profile](https://github.com/common-workflow-language/cwlprov/blob/main/prov.md) | Official project repository | Additional provenance option for CWL runs; do not require it when another qualified workflow provenance profile is used. |
| [FAIR Guiding Principles](https://doi.org/10.1038/sdata.2016.18) | Original peer-reviewed 2016 paper | Establishes findable, accessible, interoperable, reusable principles. It is not an implementation, security framework, or certification. |
| [DataCite metadata schema](https://support.datacite.org/docs/datacite-metadata-schema) | Official support documentation; schema 4.7 current at cut-off | PID metadata and relationships for research objects. Always pin schema version and repository rules. |
| [DataCite IGSN IDs](https://support.datacite.org/docs/igsn-ids) | Official current documentation | IGSN identifies physical/material samples or features of interest and connects them to research objects. It explicitly does not identify digital data. |
| [IGSN metadata recommendations](https://support.datacite.org/docs/igsn-id-metadata-recommendations) | Official guidance | Supports better physical-sample metadata. Local LIMS lineage and domain metadata remain authoritative. |
| [NIH selecting a data repository](https://www.grants.nih.gov/policy-and-compliance/policy-topics/sharing-policies/dms/selecting-a-data-repository) | Official page updated 2026-08-20 | Useful criteria: PID, sustainability, metadata, curation, access, security, provenance, retention, and sensitive-data capabilities. Applies to NIH-supported context, not every project. |
| [Final NIH Data Management and Sharing Policy](https://grants.nih.gov/grants/guide/notice-files/not-od-21-013.html) | Effective 2023-01-25 | Supports prospective data-management planning and appropriate sharing for covered NIH research, with justified limits. Not a universal sharing mandate. |
| [protocols.io API](https://apidoc.protocols.io/) | Official API documentation | Demonstrates a concrete versioned protocol service with DOI/URI and version concepts. It is an example adapter, not the blueprint's protocol standard. |

### 7.3 ELN/LIMS event and source-system behavior

| Source | Version/status checked | Blueprint use and limit |
|---|---|---|
| [Benchling entries](https://docs.benchling.com/docs/working-with-entries) | Official current product docs | Establishes supported entry operations for one ELN. Draft/write semantics must be validated for the actual tenant and workflow. |
| [Benchling events getting started](https://docs.benchling.com/docs/events-getting-started) | Official event docs | States event payloads should not be treated as current object truth and events can arrive late/out of order; refetch the API resource. This informed the conservative generic pattern. |
| [Benchling webhooks](https://docs.benchling.com/docs/getting-started-with-webhooks) | Official webhook docs | Webhook/event identifiers and delivery behavior support deduplication design. Exact retention/retry behavior must be version-checked. |
| [Benchling webhook verification](https://docs.benchling.com/docs/webhook-verification) | Official security docs | Supports signature/timestamp verification at ingress. Verification does not make payload content trusted scientific instructions. |
| [openBIS data modelling](https://openbis.readthedocs.io/en/20.10.12-plus/user-documentation/advance-features/openbis-data-modelling.html) | Official documentation for an older 20.10.12-plus line | Illustrates another ELN/LIMS-style domain model. Because the referenced documentation is old, it was not used to prescribe a current API. |

### 7.4 Compute schedulers and workflow specifications

| Source | Version/status checked | Blueprint use and limit |
|---|---|---|
| [Slurm REST API](https://slurm.schedmd.com/rest_api.html) and [slurmrestd](https://slurm.schedmd.com/rest.html) | Official current docs; versions change across Slurm releases | Prefer the supported REST/OpenAPI version and pin it. `slurmrestd` is a scheduler interface, not scientific validation. |
| [Slurm job state codes](https://slurm.schedmd.com/job_state_codes.html) | Official current docs | `COMPLETED` describes process/job completion with zero exit status. It does not establish output integrity or scientific success. |
| [Slurm APIs](https://slurm.schedmd.com/api.html) | Official current docs | Documents API/version boundaries; reinforces the need for an adapter compatibility matrix. |
| [Kubernetes Jobs](https://kubernetes.io/docs/concepts/workloads/controllers/job/) and [batch/v1 Job API](https://kubernetes.io/docs/reference/kubernetes-api/batch/job-v1/) | Official current docs | Job completions, retries, indexed jobs, and failure policy are operational semantics. Keep domain output validation separate. |
| [CWL v1.2 workflow specification](https://www.commonwl.org/v1.2/Workflow.html) and [specification portal](https://www.commonwl.org/specification/) | v1.2.1 current release observed, released 2024-01 | Vendor-neutral workflow DAG and declared version help reproducibility. Structural validation does not prove correct scientific design or results. |
| [CWL v1.2 repository](https://github.com/common-workflow-language/cwl-v1.2) | Official specification repository | Version and implementation evidence; use exact runner/version and conformance tests. |

### 7.5 Measurement, units, uncertainty, and traceability

| Source | Version/status checked | Blueprint use and limit |
|---|---|---|
| [BIPM/JCGM Guides](https://www.bipm.org/en/publications/guides) | Official current publication index; JCGM 100:2008 plus later guides/supplements and 2026 amendment | Measurement uncertainty requires a declared model, contributors, and versioned procedure. The blueprint does not encode one universal calculation. |
| [JCGM Working Group 1](https://www.bipm.org/en/committees/jc/jcgm/wg/jcgm-wg1-gum) | Official current program page | Confirms the evolving GUM family, including later work. Use as a refresh source. |
| [SI Brochure](https://www.bipm.org/en/publications/si-brochure/) | 9th edition with updates, current page at cut-off | Source for SI definitions and updates. Domain units and measurands still need profiles. |
| [UCUM 2.2](https://ucum.org/ucum) | Version 2.2, dated 2024-06-17 | Machine-readable unit codes and formal syntax. Does not define the complete semantic basis/measurand for every quantity. |
| [NIST metrological traceability](https://www.nist.gov/metrology/metrological-traceability) | Official current guidance | Traceability applies to a measurement result through a documented unbroken calibration chain with uncertainty contributions. Calibration status alone is insufficient. |

### 7.6 Reproducibility, replicability, and rigor

| Source | Version/status checked | Blueprint use and limit |
|---|---|---|
| [National Academies, Reproducibility and Replicability in Science, chapter 3](https://www.nationalacademies.org/read/25303/chapter/3) | 2019 consensus report | Supplies explicit reproducibility/replicability definitions and cautions that non-replication has multiple causes and does not automatically imply misconduct. Exact terminology still varies by field. |
| [NIH rigor and reproducibility](https://www.grants.nih.gov/policy-and-compliance/policy-topics/reproducibility) | Official current NIH overview | Supports rigorous, unbiased design, methodology, analysis, interpretation, and reporting for NIH context. It is not a complete cross-domain design standard. |
| [NIH principles and guidelines for reporting preclinical research](https://www.grants.nih.gov/policy-and-compliance/policy-topics/reproducibility/principles-guidelines-reporting-preclinical-research) | Official NIH guidance | Supports explicit distinction of biological/technical replicates, resource authentication, controls, and reporting details. Scope is preclinical research and must not be generalized mechanically. |

### 7.7 Laboratory safety, biosecurity, and OT security

| Source | Version/status checked | Blueprint use and limit |
|---|---|---|
| [CDC/NIH BMBL, 6th edition](https://www.cdc.gov/labs/bmbl/index.html) | Current official BMBL landing page at cut-off | Emphasizes protocol-driven risk assessment and layered biosafety practice. It cannot be compiled into a universal name-to-risk lookup. |
| [WHO Laboratory Biosafety Manual, 4th edition](https://www.who.int/publications/b/55063) | Official 4th-edition publication | Risk- and evidence-based, case-specific biosafety foundation. Local regulation, facility, agent, procedure, and competence still govern. |
| [OSHA 29 CFR 1910.1450, Occupational Exposure to Hazardous Chemicals in Laboratories](https://www.osha.gov/laws-regs/regulations/standardnumber/1910/1910.1450) | Current U.S. regulation page at cut-off | Establishes Chemical Hygiene Plan and related controls in its legal scope. It is jurisdiction- and applicability-specific, not a global software checklist. |
| [NIH Office of Science Policy biosafety and biosecurity policy page](https://osp.od.nih.gov/policies/biosafety-and-biosecurity-policy/) | Official page, updated 2026 at cut-off | Current hub for NIH Guidelines and policy transition. Use as a monitored source rather than copying policy text into prompts. |
| [NIH Guidelines for Research Involving Recombinant or Synthetic Nucleic Acid Molecules](https://osp.od.nih.gov/wp-content/uploads/NIH_Guidelines.pdf) | April 2024 current edition identified at cut-off | Important current requirements for covered work. Applicability and institutional review are not model decisions. |
| [U.S. Government Policy for Stopping High-Risk Life Sciences Research](https://www.whitehouse.gov/wp-content/uploads/2026/07/USG-Policy-for-Stopping-High-Risk-Life-Sciences-Research_July-2026.pdf) | Issued July 2026 | Current federal prohibitions/restrictions for defined covered work. This is a hard refresh/stop signal, not a universal global taxonomy. |
| [NIH implementation notice NOT-OD-26-101](https://grants.nih.gov/grants/guide/notice-files/NOT-OD-26-101.html) | Released 2026 | Confirms NIH implementation posture for the 2026 high-risk policy and that further guidance was developing. Institutional interpretation remains necessary. |
| [Draft NIH Biosafety Policy, NOT-OD-26-112](https://grants.nih.gov/grants/guide/notice-files/NOT-OD-26-112.html) | Draft released 2026-08-19; comment period through 2026-10-19 at cut-off | Change-monitoring source only. It was not effective policy at the research date and must not replace current rules in production. |
| [NIST SP 800-82 Rev. 3](https://csrc.nist.gov/pubs/sp/800/82/r3/final) | Final, September 2023 | OT security foundation that preserves safety, reliability, and availability constraints. Laboratory instrument qualification and hazards remain local. |

### 7.8 Electronic records and data integrity

| Source | Version/status checked | Blueprint use and limit |
|---|---|---|
| [FDA Part 11 scope and application guidance](https://www.fda.gov/media/75414/download) | Official final 2003 guidance | Part 11 scope depends on predicate-rule records; guidance supports risk-based validation and audit considerations. It does not make every electronic lab record regulated. |
| [FDA Data Integrity and Compliance With Drug CGMP](https://www.fda.gov/media/119267/download) | Official final guidance, December 2018 | Useful data-integrity expectations in drug CGMP scope. The blueprint uses integrity principles without making a CGMP claim. |
| [OECD GLP Data Integrity](https://www.oecd.org/en/publications/glp-data-integrity_45779212-en.html) | OECD Series on Principles of GLP and Compliance Monitoring | Risk-based data-lifecycle and criticality guidance for GLP environments. Applicability requires an actual GLP context. |
| [OECD application of GLP principles to computerized systems](https://www.oecd.org/content/dam/oecd/en/publications/reports/2016/04/application-of-glp-principles-to-computerised-systems_7d05366e/e3c9cda5-en.pdf) | Official OECD advisory document | Supports intended-use, lifecycle, validation, security, audit, and continuity thinking in GLP scope. It is not an agent-specific certification path. |
| [MHRA GxP data-integrity guidance](https://www.gov.uk/government/publications/guidance-on-gxp-data-integrity) | Official current U.K. guidance page | Useful GxP data-governance principles with explicit scope and cross-reference to later OECD GLP material. Jurisdiction and regulated use matter. |

### 7.9 Autonomous laboratories and science-agent evaluation

| Source | Version/status checked | Blueprint use and limit |
|---|---|---|
| [Coscientist: Autonomous chemical research with large language models](https://www.nature.com/articles/s41586-023-06792-0) | Peer-reviewed Nature article, 2023 | Evidence that a tool-integrated LLM system can plan/execute selected semi/autonomous chemistry tasks. Six-task/domain demonstration is not general safety or production reliability evidence. |
| [A-Lab: An autonomous laboratory for the accelerated synthesis of novel materials](https://www.nature.com/articles/s41586-023-06734-w) | Peer-reviewed Nature article, 2023; page carries later update/correction | Demonstrates an integrated domain-specific materials platform. Claims must be read with the 2026 correction. |
| [Author Correction to A-Lab](https://doi.org/10.1038/s41586-025-09992-y) | Published January 2026 | Corrects novelty/training-overlap interpretation and reports manual reanalysis with 36 of 40 successes and four inconclusive results. Strong evidence for independent characterization and correction-aware provenance. |
| [ScienceAgentBench repository](https://github.com/OSU-NLP-Group/ScienceAgentBench) | Official benchmark repository; verified split added 2026-04-30 | 102 code/data-driven tasks from 44 papers; repository change documents false-negative labels. Pin commit and split. It does not test physical effects or lab safety. |
| [ScienceAgentBench paper](https://openreview.net/pdf?id=6z4YKr0GK6) | Primary benchmark paper | Useful for scientific code/data agent evaluation design and limitations. Local tools, data, and safety remain out of scope. |
| [LAB-Bench](https://arxiv.org/abs/2407.10362) | Primary benchmark preprint, 2024 | More than 2,400 multiple-choice questions across practical biology topics. Knowledge breadth is not trajectory, instrument, or effect evidence. |
| [Performance metrics to unleash the power of self-driving labs](https://www.nature.com/articles/s41467-024-45569-5) | Peer-reviewed Nature Communications article, 2024 | Useful metric dimensions such as autonomy, lifetime, throughput, precision, material use, accessible parameter space, and optimization efficiency. Not a universal release threshold or certification. |

### 7.10 Representative adapter qualification sources

| Source | Version/status checked | Blueprint use and limit |
|---|---|---|
| [Benchling stability policy](https://docs.benchling.com/docs/stability) | Official current documentation checked 2026-08-31 | Distinguishes stable, beta, alpha, deprecated and SDK-version behavior. Exact endpoint/tenant behavior still needs qualification. |
| [Benchling V3 overview](https://docs.benchling.com/docs/v3-api-overview) and [unified V3 changelog](https://docs.benchling.com/changelog/v3-rest-api-unified-api-new-stability-guidelines-and-breaking-changes) | Official dynamic pages; transition details differ | Evidence that product-transition documentation can drift; pin the observed tenant route/stability instead of repeating a generic V3 claim. |
| [Benchling app versioning](https://docs.benchling.com/docs/managing-and-sharing-benchling-apps) | Official current documentation | App installations/versions and legacy/GxP differences are relevant to deployment identity. App version does not establish scientific record approval. |
| [openBIS 7.x V3 API transition](https://openbis.readthedocs.io/en/7.x/software-developer-documentation/apis/java-javascript-v3-api.html) | Official 7.x documentation | Records continuing V3 API and DSS-to-AFS transition. Local server release and data model remain decisive. |
| [SciCat project](https://www.scicatproject.org/) and [data model](https://www.scicatproject.org/documentation/Development/v4.x/Data_Model.html) | Project page reports 3.1.0 at cut-off; documentation exposes v4.x model family | Useful SDMS/catalog distinction among metadata, raw/derived datasets, storage references, lifecycle, jobs and publication. It is not byte storage or universal lab metadata. |
| [Opentrons Protocol API versioning](https://docs.opentrons.com/python-api/versioning/) | Protocol API 2.29; robot software 9.1.1; documented Flex/OT-2 ranges at cut-off | Concrete robotics compatibility example. Offline analysis/simulation does not prove physical setup or safety. |
| [Slurm REST API](https://slurm.schedmd.com/rest_api.html) | Slurm 26.05.3 and REST path family `v0.0.45` observed at cut-off | Pin scheduler and REST/data-parser versions. Job acceptance/completion remains operational evidence only. |
| [Jupyter Server REST API](https://jupyter-server.readthedocs.io/en/stable/developers/rest-api.html) | Official stable docs | Contents/checkpoint operations support bounded notebook work. A checkpoint is not a governed commit, environment or approval. |
| [GitHub REST API versions](https://docs.github.com/en/rest/about-the-rest-api/api-versions) | `2026-03-10` and `2022-11-28` supported at cut-off | Pin the version header and immutable commit; unversioned requests default differently and repository permissions remain operation-specific. |
| [Amazon S3 conditional requests](https://docs.aws.amazon.com/AmazonS3/latest/userguide/conditional-requests.html) and [object integrity](https://docs.aws.amazon.com/AmazonS3/latest/userguide/checking-object-integrity.html) | Official current documentation | Supports conditional operations and explicit checksums. ETag/multipart/version semantics require exact qualification; S3 is only one representative object store. |
| [Zenodo REST API](https://developers.zenodo.org/) | Official current developer documentation | Distinguishes deposit write and deposit action/publish scopes plus draft/published deposit records. Repository release is still human-governed. |
| [DataCite Metadata Schema](https://support.datacite.org/docs/datacite-metadata-schema) | Schema 4.7 current at cut-off | Versioned PID metadata for publication packages. It does not validate authorship, scientific claims, access choice or repository suitability. |

## 8. Evidence-to-blueprint coverage

| Blueprint guide | Primary evidence families used |
|---|---|
| [01 — Mission and authority](../../agents/scientific-research-agent/01-mission-boundary-requirements-and-authority.md) | BMBL, WHO, OSHA, NIH policy, record-integrity scope, autonomous-lab limitations |
| [02 — Architecture and integrations](../../agents/scientific-research-agent/02-reference-architecture-integrations-and-runtime.md) | SiLA, LADS, Allotrope, AnIML, Benchling events, Slurm/Kubernetes/CWL, repository canonical boundaries |
| [03 — Hypothesis and experiment state](../../agents/scientific-research-agent/03-hypothesis-protocol-experiment-state-and-planning.md) | National Academies, NIH rigor, provenance sources, canonical state/effect guidance |
| [04 — Samples and reproducibility](../../agents/scientific-research-agent/04-samples-measurements-artifacts-lineage-and-reproducibility.md) | DataCite/IGSN, PROV, RO-Crate, ISA, FAIR, BIPM, UCUM, NIST traceability, repository guidance |
| [05 — Tools and effects](../../agents/scientific-research-agent/05-instruments-simulations-tools-effects-and-recovery.md) | SiLA/LADS, scheduler/workflow semantics, OT/safety sources, canonical idempotency/effect safety |
| [06 — Context, security, and safety](../../agents/scientific-research-agent/06-context-memory-security-safety-and-data-governance.md) | CDC/WHO/OSHA/NIH policy, NIST OT, FDA/OECD/MHRA, canonical security/context/memory guidance |
| [07 — Evaluation and incidents](../../agents/scientific-research-agent/07-observability-evaluation-failure-injection-and-incidents.md) | ScienceAgentBench, LAB-Bench, autonomous-lab papers/correction, self-driving-lab metrics, canonical evaluation guidance |
| [08 — Operations and evolution](../../agents/scientific-research-agent/08-deployment-scaling-cost-and-continuous-evolution.md) | Canonical deployment/queue/scaling/cost guidance plus workload evidence on scarce physical resources and version drift |
| [09 — Stages](../../agents/scientific-research-agent/09-zero-to-production-stages-and-exit-gates.md) | Repository staged program plus all safety/effect/evaluation evidence |
| [10 — Schemas and checklists](../../agents/scientific-research-agent/10-implementation-schemas-checklists-and-anti-patterns.md) | Synthesized contracts from the full evidence set; examples are explicitly non-normative |
| [11 — Integration qualification and worked flows](../../agents/scientific-research-agent/11-integration-qualification-and-worked-flows.md) | Benchling/openBIS/SciCat, SiLA/LADS/Opentrons, Slurm, Jupyter/GitHub, S3, Zenodo/DataCite and canonical identity/effect/recovery controls |

## 9. What was deliberately not concluded

The research does **not** establish:

- that any particular instrument, ELN, LIMS, scheduler, repository, or vendor API is suitable for a target laboratory;
- that SiLA, LADS, Allotrope, AnIML, ISA, RO-Crate, or another standard preserves all required semantics for a chosen technique;
- that a given protocol, organism, chemical, material, scale, facility, or instrument is safe or low risk;
- that the agent can perform clinical, human-subject, diagnostic, therapeutic, or unrestricted biological work;
- that any generic architecture is Part 11, GLP, GxP, CGMP, BMBL, NIH, OSHA, FAIR, or other framework compliant;
- that a model's confidence is measurement/statistical uncertainty;
- that a scheduler state or autonomous-lab benchmark score establishes scientific validity;
- that bitwise reproducibility is possible or necessary for every workflow;
- that a public repository is appropriate for sensitive, consent-limited, sovereign, security-sensitive, or proprietary data; or
- that physical authority is necessary for a valuable production deployment.

## 10. Known research limitations

1. **No live integration qualification.** Official documentation was reviewed, but no vendor tenant, controller, instrument, LIMS, scheduler, or repository was contract-tested.
2. **No domain-specific protocol validation.** The blueprint spans scientific domains; an actual technique needs qualified domain schemas, controls, uncertainty models, and acceptance criteria.
3. **No facility hazard assessment.** Safety sources informed the authority boundary, not a Chemical Hygiene Plan, biosafety risk assessment, process-hazard analysis, or equipment safety case.
4. **Policy is time- and jurisdiction-sensitive.** U.S. sources were used to expose current transition and scope, not to prescribe global law. The August 2026 NIH draft is explicitly not effective.
5. **Some standards are paywalled or implementation-limited.** ASTM detail and some Allotrope resources may require access; product conformance varies.
6. **Benchmark results were not independently rerun.** Papers, repositories, and corrections were inspected for scope; the blueprint does not reproduce their reported scores.
7. **No universal SLO thresholds were chosen.** Valid latency, repeatability, queue, and incident targets depend on technique and consequence; the guide requires local baselines.
8. **Human factors require local study.** Operator workload, alarm design, approval comprehension, and separation of duties need observation in the target facility.
9. **Record-retention details are deployment inputs.** Law, funder, contract, institutional policy, legal hold, and scientific-integrity needs determine actual periods.
10. **Native-format longevity remains technique-specific.** Preserving native bytes is necessary but may not be sufficient for future readability; emulation, viewer retention, or migration may be required.
11. **Representative providers are not recommendations.** Benchling, openBIS, SciCat, Opentrons, Slurm, Jupyter, GitHub, S3 and Zenodo illustrate semantics; none was compared comprehensively or qualified for a live laboratory.
12. **Publication and identity policy remain local.** Authorship, consent/ethics, embargo, IP/export, repository selection, institutional identity, training and delegated authority require accountable organizational decisions.

## 11. Required deployment research

Before implementation, the target team must answer:

- Which one protocol family, scientific domain, facility, and value hypothesis define Stage 0?
- Which systems own protocol approval, notebook/signature, sample identity, inventory, instrument state, raw data, code, and final interpretation?
- Which exact API, schema, firmware, profile, export, and authentication versions are supported?
- Which exact read/draft/submit/cancel/start/stop/deposit/publish capabilities can prove negative access, target state, ambiguous outcomes, cancellation, reconciliation, expiry, outage and catch-up behavior?
- What data classifications, use restrictions, providers, regions, retention, deletion, and repositories apply?
- What local hazard/risk categories, interlocks, operator qualifications, stop conditions, and emergency procedures apply?
- Is the intended use regulated, and which predicate rules or validation duties apply?
- Which uncertainty, unit, measurand, calibration, control, and result-validity profiles apply to the technique?
- What counts as a planned replicate, retry attempt, valid result, reproducible result, and acceptable disagreement?
- What are the observed capacity, sample-stability, operator, storage, cost, and recovery constraints?
- Which evaluation fixtures and independent reviewers can establish safe usefulness?

These are not questions for the model to answer from general knowledge.

## 12. Refresh plan

| Trigger | Required review |
|---|---|
| NIH/USG/CDC/WHO/OSHA or local safety-policy change | Safety owner assesses effective date, scope, current holds, protocols, and agent prohibitions before work continues |
| NIST OT or facility security baseline change | Security/facility owners update gateway/network controls and rerun adversarial/availability tests |
| SiLA/LADS/Allotrope/AnIML/RO-Crate/CWL/DataCite/UCUM version change | Data/integration owners assess compatibility, loss, schema migration, and historical readers |
| ELN/LIMS/repository API or event-contract change | Adapter owner updates contract tests, replay fixtures, reconciliation, and rollback |
| Adapter certification expiry, scope/credential/plan/region change, undocumented schema behavior, or negative-permission regression | Disable the affected operation and repeat qualification; vendor-wide connectivity remains insufficient |
| Instrument firmware/method/controller change | Instrument owner returns adapter to the appropriate read-only/simulator/shadow qualification stage |
| Scheduler/workflow-engine upgrade | Platform owner pins API/runner behavior and reruns duplicate, cancellation, checkpoint, and output-validation tests |
| Model/provider/prompt/retrieval change | Evaluation owner creates a new behavior manifest and runs offline, shadow, and canary gates |
| Benchmark label, paper correction, or source withdrawal | Research owner updates claims, fixtures, and affected decisions; historical citation remains versioned |
| Incident, near miss, or reviewer disagreement | Incident owner contains first, then curates a scoped regression fixture after authorized review |
| New protocol, instrument, facility, project class, data class, or effect class | Restart the relevant earlier stage; no inheritance by similarity alone |
| Publication repository, PID schema, object-store checksum/finalization, notebook/repository API, robotics software, or scheduler REST version change | Revalidate the exact affected capability, historical reader, ambiguity path and rollback/hold behavior |

For volatile policy and vendor sources, monitor continuously or before each relevant release. For stable specifications, review at least at adapter/release change and on a scheduled documentation refresh. “Last checked” dates should be updated only after actually reopening the authoritative source.

## 13. Research completion assessment

The packet was considered sufficient for this general blueprint when additional primary-source searches stopped changing the following core decisions:

- bounded model authority and independent safety;
- source-of-truth and adapter boundaries;
- versioned hypothesis/protocol/state/effect records;
- no blind retry of physical effects;
- native raw preservation and explicit normalization loss;
- sample/artifact provenance and uncertainty;
- layered scheduler/artifact/domain/scientific completion;
- trajectory and recovery evaluation;
- exact seven-lifetime memory inventory and loss-aware compaction continuity;
- operation-level adapter qualification and version/permission/ambiguity evidence;
- recovery-load and publication-handoff boundaries;
- staged authority with physical execution optional; and
- strong claims limited by source scope and current corrections.

The unresolved items in Section 11 are intentionally local and cannot be resolved responsibly in a technology-neutral repository blueprint.

## Navigation

Return to the [Scientific Research and Laboratory Operations Agent Blueprint](../../agents/scientific-research-agent/README.md).
