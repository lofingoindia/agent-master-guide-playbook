# Security Investigation Agent Blueprint — Research Packet

> **Status:** Completed research packet with retained uncertainties  
> **Research date:** 2026-08-31  
> **Scope:** Production design for a defensive security alert-triage and investigation agent with optional, separately governed containment. Offensive autonomy is out of scope.  
> **Primary guides:** [Security investigation and triage agent](../../agents/security-investigation-agent/README.md)

This packet records the evidence used to build the guide set, the decisions synthesized from it, and the claims that remain unresolved. It is not a source dump. Standards define interoperable structures and control expectations; operational architecture must still reconcile their gaps, age, and differing scopes.

## Research questions

1. What tasks belong to alert triage, incident analysis, coordination, containment, and remediation?
2. How should alerts, raw evidence, normalized events, claims, hypotheses, and timelines relate?
3. Which current standards are useful for events, CTI, playbooks, response commands, evidence, identity, and telemetry?
4. Where may a model reason, and where must deterministic systems retain authority?
5. How should untrusted security content and malware be isolated without destroying forensic integrity?
6. What reliability, idempotency, tenancy, privacy, retention, and recovery contracts are required?
7. What evaluation evidence exists for LLM-assisted SOC work, and what does it not establish?
8. What staged architecture minimizes risk while generating evidence for later expansion?
9. What must be qualified separately for SIEM, SOAR, EDR, cloud, IAM, CTI, and ticket/case adapters?

## Research method

The review prioritized:

1. current official standards and government guidance;
2. official standard/project repositories and release histories;
3. original peer-reviewed papers and public benchmark repositories;
4. current vendor engineering reports for implementation lessons, explicitly treated as self-reports;
5. current preprints only where evidence was unavailable elsewhere, labeled as emerging.

Important version and status claims were checked against current release/history pages. Findings were cross-checked across incident-response, forensic, CTI, agent-security, durable-execution, and evaluation sources. Unsupported universal rates, marketing claims, and offensive-agent designs were excluded.

## Current version and status baseline

| Area | Baseline on 2026-08-31 | Status and implication |
|---|---|---|
| Incident response | [NIST SP 800-61 Rev. 3](https://csrc.nist.gov/pubs/sp/800/61/r3/final), April 2025 | Final; supersedes Rev. 2 and frames incident response across CSF 2.0. |
| Federal operational playbook | [CISA Incident and Vulnerability Response Playbooks](https://www.cisa.gov/sites/default/files/2024-08/Federal_Government_Cybersecurity_Incident_and_Vulnerability_Response_Playbooks_508C.pdf) | Operationally useful beyond federal contexts, but organization-specific thresholds and authorities still apply. |
| SOC/CSIRT service taxonomy | [FIRST CSIRT Services Framework 2.1](https://www.first.org/standards/frameworks/csirts/csirt_services_framework_v2.1) | Current official 2.1 page labels the framework “for review”; it defines services/outcomes, not implementation or maturity. |
| Digital forensics | [NIST SP 800-86](https://csrc.nist.gov/pubs/sp/800/86/final), 2006 | Final but old; still useful for acquisition, integrity, analysis, and reporting. Supplement with newer preservation guidance. |
| Digital evidence preservation | [NISTIR 8387](https://www.nist.gov/publications/digital-evidence-preservation-considerations-evidence-handlers), 2022 | Final; strong evidence-storage, hash, access, and preservation guidance. |
| Evidence collection | [RFC 3227 / BCP 55](https://www.rfc-editor.org/info/rfc3227), 2002 | Old but still a BCP; useful for order of volatility, collection before analysis, reproducibility, and custody. |
| International evidence guidance | [ISO/IEC 27037:2012](https://www.iso.org/standard/44381.html) | Published; ISO page shows a revision is planned/underway. Avoid claiming the 2012 text is permanently current. |
| Media sanitization | [NIST SP 800-88 Rev. 2](https://csrc.nist.gov/pubs/sp/800/88/r2/final), September 2025 | Final; supersedes Rev. 1 and focuses on an enterprise sanitization program. |
| Log management | [NIST SP 800-92 Rev. 1 initial public draft](https://csrc.nist.gov/pubs/sp/800/92/r1/ipd), 2023 | Still draft; NIST project page says comments are being addressed. Do not cite as final. |
| Zero trust | [NIST SP 800-207](https://csrc.nist.gov/pubs/sp/800/207/final) and [SP 800-207A](https://csrc.nist.gov/pubs/sp/800/207/a/final) | Final; supports per-request resource and workload-identity policy rather than network-location trust. |
| Security controls | [NIST SP 800-53 Rev. 5, current derivative release 5.1](https://csrc.nist.gov/Projects/risk-management/sp800-53-controls/downloads) | Control catalog covering access, audit, incident response, media, privacy, and integrity; tailoring remains organizational. |
| Privacy | [NIST Privacy Framework 1.0 and 1.1 initial public draft](https://www.nist.gov/privacy-framework) | 1.0 final; 1.1 is draft. Use for risk management, not jurisdiction-specific legal advice. |
| Adversary/detection knowledge | [MITRE ATT&CK 19.2](https://attack.mitre.org/resources/versions/) | Current. ATT&CK v18 replaced technique detections with detection strategies/analytics and deprecated data-source objects. |
| Defensive techniques | [MITRE D3FEND 1.5.0](https://d3fend.mitre.org/) | Knowledge graph; MITRE explicitly says it does not prescribe or rate control effectiveness. |
| Security-event schema | [OCSF 1.8.0](https://github.com/ocsf/ocsf-schema/releases/tag/1.8.0), March 2026 | Current release. It is a vendor-neutral schema, not a storage, ETL, or evidence-integrity system. |
| CTI format/transport | [STIX 2.1 and TAXII 2.1](https://www.oasis-open.org/2021/06/23/stix-v2-1-and-taxii-v2-1-oasis-standards-are-published/) | OASIS standards; structure and transport do not establish intelligence truth or handling rights. |
| Sharing marking | [FIRST TLP 2.0](https://www.first.org/tlp/) | Current and authoritative from August 2022; sharing boundary, not classification or reliability score. |
| CTI uncertainty | [FIRST source evaluation](https://www.first.org/global/sigs/cti/curriculum/source-evaluation) and [uncertainty communication](https://www.first.org/global/sigs/cti/curriculum/cti-reporting) | Useful separate concepts for source reliability, information credibility, confidence, and estimative probability. |
| Vulnerability severity | [CVSS 4.0, specification document 1.2](https://www.first.org/cvss/v4.0/specification-document) | Current. Base is not organizational priority; Threat and Environmental metrics improve local relevance. |
| Exploit likelihood | [FIRST EPSS](https://www.first.org/epss/) | Daily probability of in-the-wild exploitation in the next 30 days; not incident evidence or impact. |
| Detection rules | [Sigma specification 2.1.0](https://sigmahq.io/sigma-specification/specification/sigma-rules-specification.html), August 2025 | Current official specification; translation and local field/log semantics still need testing. |
| Security playbooks | [CACAO 2.0](https://www.oasis-open.org/standard/cacao-security-playbooks-v2-0/), November 2023 | OASIS Committee Specification with versioning, workflow objects, markings, and signatures. It does not authorize local effects. |
| Response command vocabulary | [OpenC2 Language 1.0 CS02](https://www.oasis-open.org/committees/tc_home.php?wg_abbrev=openc2) | Current 1.0 language revision; action/target/actuator semantics and responses, not local policy or rollback. |
| Distributed correlation | [W3C Trace Context](https://www.w3.org/TR/trace-context/), 2021 Recommendation | Stable correlation format with explicit privacy/security caveats; trace context is not authority. |
| Agent observability | [OpenTelemetry GenAI agent conventions](https://github.com/open-telemetry/semantic-conventions-genai/blob/main/docs/gen-ai/gen-ai-agent-spans.md) | Development status. Use as a mapping target, not the sole durable audit contract. |

## Evidence ledger

### Incident response and SOC operating model

| Source | Evidence retained | Design consequence | Limitation |
|---|---|---|---|
| [NIST SP 800-61 Rev. 3 announcement and final status](https://www.nist.gov/news-events/news/2025/04/nist-revises-sp-800-61-incident-response-recommendations-and-considerations) | Rev. 3 supersedes Rev. 2 and integrates response with all six CSF 2.0 functions. | Treat the agent as one capability within organization-wide preparation, detection, response, and recovery. | High-level risk-management profile, not a detailed agent architecture. |
| [FIRST CSIRT Services Framework 2.1](https://www.first.org/standards/frameworks/csirts/csirt_services_framework_v2.1) | Separates monitoring/detection, event analysis, incident intake, triage, detailed analysis, coordination, and situational awareness; explicitly values context and false-alarm qualification. | Separate alert disposition from incident analysis and response authority. Include asset/identity context and continuous detection-quality feedback. | Framework intentionally avoids implementation, maturity, and capacity prescriptions. |
| [CISA response playbooks](https://www.cisa.gov/sites/default/files/2024-08/Federal_Government_Cybersecurity_Incident_and_Vulnerability_Response_Playbooks_508C.pdf) | Standardized detection/analysis, containment, eradication/recovery, post-incident work; containment considers evidence preservation, availability, constraints, and duration. | Response proposals must include preservation, business impact, duration, and verification—not just a technical block. | Federal responsibilities and reporting thresholds are not universal. |

### Evidence and forensics

| Source | Evidence retained | Design consequence | Limitation |
|---|---|---|---|
| [NIST SP 800-86](https://www.nist.gov/publications/guide-integrating-forensic-techniques-incident-response-0) | Identify/acquire/protect, process, analyze, and report; verify integrity; document acquisition and custody; analyze copies. | Separate immutable raw evidence from working copies and investigative artifacts. | 2006 technology examples are dated. |
| [NISTIR 8387](https://nvlpubs.nist.gov/nistpubs/ir/2022/NIST.IR.8387.pdf) | Digital files are easy to change; recommends approved hashes, separate secure hash storage, copies, access controls, and preservation planning. | Register digest, independently protect integrity metadata, test migrations and mismatch handling. | Oriented partly to law-enforcement evidence management; local organizational context differs. |
| [RFC 3227](https://datatracker.ietf.org/doc/html/rfc3227) | Order of volatility; minimize changes; collect before analyze; transparent and reproducible methods; detailed custody. | Make volatile acquisition a specialist decision, record method/tool/version, and never let an agent silently alter a live source. | Old BCP and not a complete modern cloud-forensics guide. |
| [ISO/IEC 27037:2012](https://www.iso.org/standard/44381.html) | Identification, collection, acquisition, and preservation guidance for potential digital evidence. | Use as an international vocabulary and governance baseline. | Paywalled detail and currently marked for revision. |
| [SWGDE digital evidence collection](https://www.swgde.org/documents/published-complete-listing/18-f-002-2-0/) and [computer forensic acquisition](https://www.swgde.org/documents/published-complete-listing/17-f-002-2-1/) | Contemporaneous custody, unique evidence identifiers, volatile/ancillary data, and contextual metadata. | Require acquisition manifests and custody events, not only object hashes. | Best-practice documents still require organization/legal tailoring. |
| [NIST SP 800-88 Rev. 2](https://csrc.nist.gov/pubs/sp/800/88/r2/final) | Current media sanitization program guidance and 2025 replacement of Rev. 1. | Retention/deletion design must include media and logical-storage sanitization verification. | Does not by itself define deletion across SaaS/model providers, embeddings, or backups. |

### Security events, CTI, and detection knowledge

| Source | Evidence retained | Design consequence | Limitation |
|---|---|---|---|
| [OCSF overview](https://github.com/ocsf/ocsf-schema) and [1.8.0 release](https://github.com/ocsf/ocsf-schema/releases/tag/1.8.0) | Vendor-neutral event categories, classes, objects, and attributes; 1.8 added AI operation-related objects/profile. | Normalize for investigation and analytics, but preserve native events and pin schema/extensions. | Mapping is lossy; OCSF is format-agnostic and not an evidence store. Agent-specific schema work continues. |
| [MITRE ATT&CK v18 changes](https://attack.mitre.org/resources/updates/updates-october-2025/) and [current version history](https://attack.mitre.org/resources/versions/) | Detection strategies and analytics replaced technique detections; data sources deprecated; current version 19.2. | New guides should use strategies, analytics, and data components; pin datasets. | ATT&CK is a knowledge base, not proof of a local technique or actor. |
| [ATT&CK detection strategies](https://attack.mitre.org/detectionstrategies/) and [analytics](https://attack.mitre.org/analytics/) | High-level detection approaches separated from platform-specific logic. | Preserve distinction between behavioral hypothesis and concrete local analytic. | Local telemetry and rule validation remain essential. |
| [MITRE D3FEND FAQ](https://d3fend.mitre.org/faq/) | D3FEND is a defensive technique knowledge graph and does not prioritize or establish effectiveness. | Use it as vocabulary or mapping support, never as automatic remediation selection. | Coverage and referenced mechanisms are not effectiveness evidence. |
| [STIX/TAXII 2.1](https://oasis-open.github.io/cti-documentation/resources.html) | Structured objects and RESTful exchange; STIX and TAXII are independent. | Preserve object version, producer, marking, retrieval snapshot, and local assessment. | Syntax/transport do not imply quality, timeliness, license, or relevance. |
| [FIRST TLP 2.0](https://www.first.org/tlp/) | Four valid sharing labels and authoritative handling definitions. | Enforce sharing policy across prompts, cases, traces, and derived intelligence. | TLP is not a secrecy classification, source reliability, or permission to process. |
| [FIRST source evaluation](https://www.first.org/global/sigs/cti/curriculum/source-evaluation) | Separates source reliability from information credibility. | Store them separately and retain unknown states. | Rating practice needs analyst training and calibration. |
| [FIRST CTI uncertainty reporting](https://www.first.org/global/sigs/cti/curriculum/cti-reporting) | Uses words of estimative probability and levels of confidence to reduce ambiguous language. | Define a local rubric instead of free-form “likely” and “high confidence.” | Organizations may tailor language; cross-org comparability can still fail. |
| [CVSS 4.0 specification](https://www.first.org/cvss/v4.0/specification-document) and [EPSS](https://www.first.org/epss/) | CVSS separates Base, Threat, Environmental, and Supplemental; EPSS estimates 30-day exploitation probability. | Keep technical severity, local importance, current exploitation, and observed compromise separate. | Neither score is a case verdict or substitute for evidence. |
| [Sigma specification 2.1.0](https://sigmahq.io/sigma-specification/specification/sigma-rules-specification.html) | Portable detection-rule structure, taxonomy, filters, and correlation specification. | Agent-generated rule drafts must retain source/evidence and be compiled/tested against local data before promotion. | Backend translation and inconsistent field semantics can change behavior. |

### Product API integration evidence

These vendor/API facts are examples used to derive adapter qualification tests. They are volatile and apply only to the documented surface on the research date.

| Source | Evidence retained | Design consequence | Limitation |
|---|---|---|---|
| [Microsoft Graph `alerts_v2`](https://learn.microsoft.com/en-us/graph/api/security-list-alerts_v2?view=graph-rest-1.0) and [security incidents](https://learn.microsoft.com/en-us/graph/api/resources/security-incident?view=graph-rest-1.0) | Alerts expose documented OData filtering/paging and environment-retention bounds; incident objects contain correlated vendor state and support distinct read/write operations. | Follow native paging links, retain alert/incident IDs and source/version, keep vendor correlation distinct from local case truth, and separate read from incident-write scopes. | Permissions, national-cloud support, available fields, and retention vary; vendor incident correlation is not evidence of causality. |
| [Microsoft Graph sign-in list](https://learn.microsoft.com/en-us/graph/api/signin-list?view=graph-rest-1.0) and [throttling guidance](https://learn.microsoft.com/en-us/graph/throttling) | Sign-in queries support time filters and pagination; Graph advises `Retry-After` handling for throttling. | Bound identity queries by time and fields, preserve page/permission/coverage state, and partition retry budgets by tenant/operation. | Missing conditional-access or identity fields may reflect permission/licensing; API throttling behavior is service-specific. |
| [Microsoft Defender isolate machine](https://learn.microsoft.com/en-us/defender-endpoint/api/isolate-machine) and [machine action](https://learn.microsoft.com/en-us/defender-endpoint/api/machineaction) | Isolation submission returns a machine-action resource; vendor documentation warns full-tunnel VPNs can impair service reachability after isolation. | Model isolation as asynchronous submitted/observed state, preserve native action ID, verify management path, and test unisolate/rollback on target fleets. | Platform/license/device eligibility and action semantics change; a `201` response does not prove target isolation. |
| [AWS CloudTrail `LookupEvents`](https://docs.aws.amazon.com/awscloudtrail/latest/APIReference/API_LookupEvents.html) | Current event-history lookup covers 90 days, returns at most 50 events per request, uses `NextToken`, and limits requests to two per second per account per Region. | Encode source scope, retention, pagination, region, and rate limit in the coverage receipt; use a lake/trail for broader history. | The convenience lookup is not a complete management/data/network evidence source. |
| [AWS Security Hub customer finding updates](https://docs.aws.amazon.com/securityhub/latest/userguide/finding-update-batchupdatefindings.html) | Customer updates operate on existing findings and a constrained field set; SIEM/ticket/SOAR tools can act on behalf of a customer. | Expose exact typed finding fields, keep source finding identity, separate proposed and confirmed workflow state, and read back authoritative outcome. | Update permissions and accepted fields differ from creation/provider behavior; batch/partial outcomes need per-item receipts. |
| [Google Cloud Logging `entries.list`](https://cloud.google.com/logging/docs/reference/v2/rest/v2/entries/list) | Reads are scoped by resource names, can fan out, use stable page tokens, and may return an empty entries array with a non-empty next token. | Empty intermediate pages cannot terminate the query; record source resources, filters, order, cursor, regional fan-out, and completeness. | Event availability, field visibility, and regional behavior depend on configured buckets/views and IAM. |
| [Google Security Command Center API](https://cloud.google.com/security-command-center/docs/reference/rest) | Finding state, mute state, marks, and external-system state are separate operations over hierarchical resource names. | Map each semantic write to a distinct tool and canonical organization/folder/project/location/source/finding identity. | Versions and supported resource scopes differ; mute is not investigation closure or remediation. |
| [OASIS TAXII 2.1 pagination](https://docs.oasis-open.org/cti/taxii/v2.1/os/taxii-v2.1-os.html) | TAXII can use opaque `next` or `added_after` plus date-added headers while retaining original filters. | Preserve transport cursor separately from STIX object time/version; test deletion and empty-page edge cases. | Server implementations may differ within allowed behavior; transport completeness does not prove intelligence quality. |
| [Jira REST v3 introduction](https://developer.atlassian.com/cloud/jira/platform/rest/v3/intro/) and [rate limits](https://developer.atlassian.com/cloud/jira/platform/rate-limiting/) | Collection paging is operation-specific; limits may change; rate-limit responses use `429` and retry metadata; issue-level security affects access. | Follow returned paging metadata, qualify fields/permissions per tenant, respect write concurrency/rate limits, and preserve changelog/readback. | Jira issues are not security case truth unless the organization deliberately defines that ownership. |
| [ServiceNow REST API](https://www.servicenow.com/docs/r/api-reference/rest-api-explorer/c_RESTAPI.html) and [rate limiting](https://www.servicenow.com/docs/r/api-reference/rest-api-explorer/inbound-REST-API-rate-limiting.html) | REST endpoints may be versioned; ACLs can omit inaccessible fields; instance-defined limits return `429`/`Retry-After`. | Pin endpoint version, restrict tables/fields, test invisible fields, and treat instance configuration as part of the adapter contract. | Generic table access can be much broader than the intended security workflow and differs between instances/releases. |

### Agent security and containment

| Source | Evidence retained | Design consequence | Limitation |
|---|---|---|---|
| [NIST AI 600-1](https://www.nist.gov/publications/artificial-intelligence-risk-management-framework-generative-artificial-intelligence) | Cross-sector generative-AI risk profile across lifecycle. | Maintain governance, measurement, documentation, and ongoing risk treatment around the technical controls. | Voluntary high-level profile; not an agent-specific authorization design. |
| [AgentDojo](https://proceedings.neurips.cc/paper_files/paper/2024/hash/97091a5177d8dc64b1da8bf3e1f6fb54-Abstract-Datasets_and_Benchmarks_Track.html) | Tool-returned untrusted data can inject instructions; dynamic environment evaluates task utility and security. | Test source-to-sink behavior with real tools and preserve benign utility. | Domains and attacks do not cover SOC artifacts or every adaptive strategy. |
| [InjecAgent](https://aclanthology.org/2024.findings-acl.624/) | Broad tool-integrated indirect-injection benchmark and demonstrated vulnerability on historical systems. | Treat every alert, log, CTI report, and tool response as untrusted. | Historical model rates are stale and benchmark attacks are bounded. |
| [Indirect Prompt Injections: Are Firewalls All You Need, or Stronger Benchmarks?](https://arxiv.org/abs/2510.05244) | Reports benchmark bugs/weak attacks and shows simple filters can saturate static tests while remaining bypassable. | Require adaptive attacks and do not claim prompt injection solved from static scores. | 2025 preprint; exact results require replication. |
| [OpenAI, Designing AI agents to resist prompt injection](https://openai.com/index/designing-agents-to-resist-prompt-injection/) | Frames attacks through social engineering and source/sink analysis; warns intermediary classifiers struggle with developed attacks. | Reduce reachable sinks and apply deterministic source-sink controls. | Vendor report and product-specific defenses. |
| [OpenAI, Understanding prompt injections](https://openai.com/safety/prompt-injections/) | Layered defenses, least access, explicit tasks, and confirmations for consequential actions. | Combine training/monitoring with least privilege and effect review. | General product guidance, not a formal guarantee. |
| [Anthropic containment engineering report](https://www.anthropic.com/engineering/how-we-contain-claude) | Filesystem/network isolation, credential separation, approval fatigue, and blast-radius control; describes real implementation failures. | Keep credentials outside analysis, prefer hard containment, and avoid approval floods. | Vendor self-report; product rates and architectures are not universal. |
| [NIST SP 800-207](https://csrc.nist.gov/pubs/sp/800/207/final) and [800-207A](https://csrc.nist.gov/pubs/sp/800/207/a/final) | No implicit network-location trust; identity-tier and network-tier policy for user/service/resource. | Authorize each query/effect by user, workload, tenant, resource, action, and current state. | General ZTA does not define agent approval or effect semantics. |

### Playbooks, effects, and reliability

| Source | Evidence retained | Design consequence | Limitation |
|---|---|---|---|
| [CACAO 2.0](https://docs.oasis-open.org/cacao/securityplaybooks/v2.0/security-playbooks-v2.0.html) | Versioned workflow steps, actions, commands, agents/targets, markings, and signature support. | Use for interchange and signed/versioned playbooks, then map to local typed steps. | Secure transport, authorization, idempotency, rollback, and local safety remain external. |
| [OpenC2 Language 1.0](https://docs.oasis-open.org/openc2/oc2ls/v1.0/oc2ls-v1.0.html) and [Architecture 1.0](https://docs.oasis-open.org/openc2/oc2arch/v1.0/oc2arch-v1.0.html) | Action, target, arguments, actuator, response, and request correlation. | Useful vocabulary for narrow response adapters and observed responses. | The standard explicitly does not cover sensing, analytics, or choosing courses of action. |
| [Google Cloud Eventarc duplicate-event guidance](https://cloud.google.com/eventarc/docs/retry-events) | At-least-once delivery, source+ID uniqueness under CloudEvents, idempotent handlers, external idempotency records. | Deduplicate alert delivery and make all side effects independently idempotent. | Product-specific mechanics; general principle transfers. |
| [Google Cloud Workflows error guidance](https://cloud.google.com/workflows/docs/reference/syntax/error-types) | Connection failures differ from failures after connection; some retries may not be idempotent. | Represent effect outcome as unknown and reconcile before retry. | Product-specific error taxonomy. |
| [Temporal event history](https://github.com/temporalio/documentation/blob/main/docs/encyclopedia/event-history/event-history.mdx) | Durable log and replay reconstruct workflow state after worker failure. | Durable runtime is appropriate for long investigations/approvals, with results persisted at effect boundaries. | Workflow durability does not make external effects idempotent. |
| [Temporal retry policies](https://github.com/temporalio/documentation/blob/main/docs/encyclopedia/retry-policies.mdx) | Activities retry and should be idempotent; whole-workflow retry is different from deterministic replay. | Keep model/tool calls in activities, cap retries, and test replay compatibility. | Runtime-specific semantics and operational overhead. |
| [W3C Trace Context](https://www.w3.org/TR/trace-context/) | Interoperable request correlation plus privacy, information-exposure, and DoS considerations. | Use trace IDs for correlation, validate headers, and never treat them as authority. | Trace propagation is not durable audit or chain of custody. |
| [OpenTelemetry GenAI spans](https://github.com/open-telemetry/semantic-conventions-genai/blob/main/docs/gen-ai/gen-ai-spans.md) and [agent spans](https://github.com/open-telemetry/semantic-conventions-genai/blob/main/docs/gen-ai/gen-ai-agent-spans.md) | Current vocabulary for inference, retrieval, memory, tool, plan, workflow, and agent spans. | Map application events to OTel for operations. | Development status; content capture is sensitive and causal/grouping questions remain open. |
| [OTel causal tool-link issue 309](https://github.com/open-telemetry/semantic-conventions-genai/issues/309) | Current conventions do not fully express which model output triggered a tool execution. | Keep application-owned causal IDs in the audit schema. | Open issue, not normative specification. |

### Malware and parser isolation

| Source | Evidence retained | Design consequence | Limitation |
|---|---|---|---|
| [NIST SP 800-83 Rev. 1](https://csrc.nist.gov/pubs/sp/800/83/r1/final) | Malware incident prevention and handling recommendations. | Treat malware handling as a prepared incident capability, not an improvised model tool. | 2013 and desktop/laptop-focused. |
| [SWGDE computer forensic examination](https://www.swgde.org/wp-content/uploads/2025/09/2018-07-11-SWGDE-Best-Practices-for-Computer-Fo.pdf) | Examination workstations should provide an isolated, known environment. | Use isolated, recorded analysis environments. | General forensic guidance; sandbox strength remains implementation-specific. |
| [YARA command-line documentation](https://yara.readthedocs.io/en/v3.10.0/commandline.html) | Warns that untrusted compiled rules can execute malicious code and requires an explicit compiled-rule flag. | Treat rules, plugins, parsers, and “security content” as untrusted supply-chain inputs. | Documentation page is for an older YARA version; retain the security principle, verify current tooling. |

### Evaluation evidence

| Source | Evidence retained | Design consequence | Limitation |
|---|---|---|---|
| [SecAlertBench repository](https://github.com/Dxsssu/SecAlertBench) | 8,322 alerts from three enterprises, 241 alert types, 16 models; repository reports promising average TPR but high average FPR. | Measure false-negative and false-positive trade-offs by alert family; do not deploy standalone from aggregate accuracy. | Artifact/paper status and enterprise sampling need independent review; binary Tier-1 labels are not full investigation. |
| [SIABench thesis](https://spectrum.library.concordia.ca/id/eprint/996238/) | 25 deep-analysis and 35 triage scenarios across network/memory/malware/phishing/log tasks. | Include heterogeneous tools and false-alert scenarios in local evals. | Small master's-thesis benchmark; not broad production evidence. |
| [SecRespond paper](https://arxiv.org/abs/2607.26791), [dataset](https://huggingface.co/datasets/Alibaba-NLP/SecRespond), and [repository harness](https://github.com/Alibaba-NLP/qqr/tree/main/data/secrespond) | Ten post-compromise cyber ranges, disk snapshots, alerts/checks, 23 models; authors report alert-exposed findings are easier than silent intrusions and no complete range. | Test silent related activity, comprehensive evidence, and verified remediation—not only alert explanation. | July 2026 preprint, only ten ranges, license restrictions, and model-judge methodology require caution. |
| [NIST CAISI evaluation-cheating examples](https://www.nist.gov/caisi/cheating-ai-agent-evaluations/2-examples-cheating-caisis-agent-evaluations) | Agents may exploit benchmark artifacts or solution leakage. | Harden eval environments, hold out solutions, and inspect unexpected paths. | Examples are not SOC-specific. |

## Synthesized production architecture

~~~mermaid
flowchart TD
    SRC["Alerts and authorized security sources"] --> IN["Authenticated intake"]
    IN --> RAW["Immutable evidence and lineage"]
    IN --> CASE["Versioned case state"]
    RAW --> COMP["Least-data context compiler"]
    CASE --> COMP
    COMP --> MODEL["Single bounded investigator"]
    MODEL --> QB["Tenant-aware read-query broker"]
    QB --> SRC
    QB --> RAW
    MODEL --> CLAIM["Proposed claims, hypotheses, gaps"]
    CLAIM --> CASE
    MODEL -. exact action proposal .-> PDP["Commit-time policy"]
    PDP --> APR["Effect-bound approval"]
    APR --> EX["Separate response executor"]
    EX --> LED["Effect ledger and observed-state reconciliation"]
    LED --> CASE
    CASE --> EVAL["Outcome, trajectory, security, and human eval"]
~~~

### Stable conclusions

1. **Alert triage, incident analysis, and response are different authority domains.** FIRST and CISA reinforce the operational separation.
2. **The case is a versioned interpretation; evidence is immutable source material.** Forensic guidance supports acquisition integrity, working copies, and custody.
3. **No-data, no-match, denied, partial, and unavailable are different results.** Source coverage must be first-class to control false negatives.
4. **Asset and identity context changes priority and authorization.** Severity, exploit probability, asset criticality, and observed compromise should not collapse into one score.
5. **CTI is a lead with provenance and uncertainty.** STIX/TAXII/TLP structure transport and sharing, not truth.
6. **The model proposes; deterministic systems authorize and commit.** This follows complete mediation, zero-trust, source-sink, and idempotency evidence.
7. **Read access is sensitive and needs a broker.** Investigation queries can expose cross-tenant, employee, customer, secret, or broad historical data.
8. **Response needs a separate identity and executor.** No response credentials should be reachable from the model or artifact-analysis runtime.
9. **Approval binds an exact, current effect.** Broad or stale consent is not reliable authority.
10. **Unknown effect outcomes must reconcile before retry.** Durable orchestration does not create exactly-once external effects.
11. **Security artifacts are hostile content.** Prompt injection and conventional parser/malware attacks require both typed context and execution isolation.
12. **Evaluation must measure material misses, noise, evidence, safety, analyst correction, and cost.** Public benchmarks remain too narrow for autonomous deployment claims.
13. **Each source instance requires its own qualification manifest.** Vendor names and HTTP status classes do not establish tenancy, pagination, coverage, permission, idempotency, reconciliation, or schema-drift semantics.

## Architecture decision record

### Default to one investigator

**Decision:** Use one bounded investigation loop plus deterministic validators.  
**Reason:** Multi-agent designs add message injection paths, inconsistent case versions, duplicate tool use, and cost. A separate verifier is justified only when it has independent evidence or a measured rubric advantage.  
**Rejected default:** Router, planner, researcher, critic, and response agents for every alert.

### Default to read-only

**Decision:** Start offline/advisory, then add tenant-aware read queries.  
**Reason:** It produces value and evaluation evidence while keeping production effects outside the model.  
**Rejected default:** Autonomous alert closure or endpoint/account/network action.

### Separate evidence, case, audit, trace, and workflow histories

**Decision:** Give each an explicit owner and retention contract.  
**Reason:** They have different integrity, privacy, size, mutability, and legal requirements.  
**Rejected default:** One transcript/vector store as memory and source of truth.

### Prefer typed query templates

**Decision:** Start with parameterized recurring investigation queries, then add a constrained abstract query tree if needed.  
**Reason:** Easier authorization, cost estimation, source compatibility, and evaluation.  
**Rejected default:** Model-generated arbitrary SQL/SIEM/EDR query language or shell.

### Terminate vendor semantics in qualified adapters

**Decision:** Keep SIEM, SOAR, EDR, cloud, IAM, CTI, and ticket/case pagination, identity, permission, consistency, error, and write semantics behind independently versioned adapters.  
**Reason:** Primary product documentation shows materially different paging, retention, field visibility, rate limiting, asynchronous action, and write surfaces. A uniform model-facing contract is safe only when it preserves those differences as coverage and effect receipts.  
**Rejected default:** Direct vendor APIs, generic HTTP/MCP tools, or one permissive connector that mixes read, case-write, collection, and response authority.

### Use durable workflows only when waits/effects justify them

**Decision:** Begin with durable intake, queues, idempotent case state, and bounded workers; adopt workflow replay for long cases, approvals, timers, and effects.  
**Reason:** Durability is valuable, but replay/versioning and operational overhead are unnecessary for a single short read-only request.  
**Rejected default:** A heavyweight durable workflow per trivial duplicate or malformed alert.

## Disagreements and trade-offs

### Automatic closure versus analyst confirmation

High-volume SOCs want automatic closure, but public benchmark evidence shows substantial false-positive/false-negative trade-offs and incomplete investigation. Closing benign-looking alerts also hides labels needed for improvement. The blueprint keeps “proposed benign” distinct from accountable closure until alert-family-specific shadow evidence supports a deterministic, reviewable rule. Even then, periodic sampling and reopen monitoring remain necessary.

### Approval versus containment

Approval helps with ambiguous intent and business impact but becomes weak when frequent or poorly explained. Containment caps blast radius even when review fails. The design uses hard isolation and policy always, effect-bound approval selectively, and no approval flood as the primary firewall.

### Dynamic malware analysis versus evidence safety

Detonation can reveal behavior unavailable from static analysis, but it adds escape, network, cost, and evidence-handling risk. The normal investigation agent receives only sanitized reports. Dynamic analysis requires a separately authorized specialist environment, not a general tool call.

### Long context versus evidence retrieval

Large context can reduce retrieval misses but increases sensitive-data exposure, prompt-injection surface, cost, and distraction. The blueprint uses a deterministic evidence manifest, progressively disclosed excerpts, and explicit omissions. Full originals remain available through controlled specialist workflows.

### Model-generated detection and response content

Models can draft Sigma/CACAO/OpenC2-like artifacts quickly. The standards ensure shape and interoperability, not correctness. Promotion requires source citations, schema validation, local compilation, replay/backtest, blast-radius analysis, owner review, signing, and canary deployment.

### OCSF versus source-native events

OCSF enables cross-source reasoning, but local/native fields often contain decisive semantics. Keep raw source and mapping lineage; never normalize away the only authoritative identifier.

### OTel versus application audit

OTel accelerates operations and backend portability. Development GenAI conventions and sampling/content policies are not sufficient for custody, approval, or effect reconstruction. Maintain an application-owned audit envelope and map it to OTel.

## Unresolved claims and research gaps

| Unresolved question | Why unresolved | Required evidence |
|---|---|---|
| Which alert families are safe for automatic benign closure? | Public aggregate benchmarks do not transfer to local prevalence, controls, and labels. | Longitudinal shadow study by family/tenant with re-adjudication and sampled closures. |
| Which reversible containment action can be pre-authorized? | Business impact and rollback differ by asset/identity architecture. | Scenario-specific drills, owner sign-off, false-action budget, and observed rollback reliability. |
| Does a second model materially reduce dangerous errors? | Correlated errors and shared evidence can create false agreement. | Blinded ablation against one-model plus deterministic verifier, including cost/latency. |
| What context size maximizes security investigation quality? | Model, alert family, evidence position, and injection surface interact. | Retrieval/context ablations with hidden relevant evidence and adversarial distractors. |
| Can current prompt-injection classifiers safely admit active security content? | Adaptive attacks and benign hard negatives remain difficult. | Source-specific adaptive red team and utility/security curves. |
| How should model confidence map to operational confidence? | Verbal confidence is model- and prompt-dependent. | Local calibration against expert-adjudicated repeated trials. |
| How reliable are SecAlertBench and SecRespond rates across organizations? | Dataset sampling, synthetic components, labels, and judge choices constrain transfer. | Independent replication and local benchmark mapping. |
| What is the best cross-source case/event standard? | OCSF, STIX, CACAO, vendor case APIs, and emerging agent telemetry cover different layers. | Interoperability prototypes and loss/round-trip tests. |
| Can deleted evidence be removed from all model/provider-derived storage? | Provider retention, backups, cache, abuse monitoring, and regional behavior vary. | Contract and technical deletion verification per provider/deployment. |
| What evidence is legally admissible? | Jurisdiction, procedure, authority, and tool validation vary. | Local legal/forensic review and validated SOPs. |
| How should model/provider outage affect incident SLAs? | Depends on staffing, fallback logic, and alert mix. | Load/failure drills and operational capacity measurements. |
| Does dynamic analysis improve triage enough to justify risk/cost? | Malware families and source artifacts differ. | Specialist-run controlled comparison against static-only path. |
| Which adapter guarantees transfer across tenants or vendor releases? | Permissions, schemas, rate limits, retention, regional topology, and case workflows are instance-specific and volatile. | Qualification manifests, production-shaped contract tests, shadow dual-read, and per-instance failure drills. |

## Claims deliberately excluded

- “Ninety-nine percent of SOC alerts are false positives” as a universal rate.
- “AI agents reduce mean time to respond by X percent” without a comparable workload and human baseline.
- “A high ATT&CK, CVSS, EPSS, reputation, or model-confidence score proves compromise.”
- “Two agents agreeing is independent corroboration.”
- “STIX/OCSF/CACAO/OpenC2 compliance makes the data, playbook, or action safe.”
- “A connector, signed report, or authenticated source contains trusted instructions.”
- “Prompt injection is solved” from one static benchmark or classifier.
- “Containerization alone safely contains malware or a compromised agent.”
- “A successful HTTP response proves containment.”
- “Exactly once” without target idempotency and observed-state reconciliation.
- “Deletion from the case database deletes prompts, traces, embeddings, caches, replicas, and provider storage.”
- “Current public defensive-agent benchmarks justify standalone incident response.”
- Offensive retaliation, exploitation, credential capture, persistence, stealth, or autonomous lateral movement.

## Guide mapping

| Guide | Packet evidence used |
|---|---|
| [README](../../agents/security-investigation-agent/README.md) | Stable architecture, maturity ladder, current baseline |
| [Operating model and reference architecture](../../agents/security-investigation-agent/operating-model-and-reference-architecture.md) | NIST incident response, FIRST CSIRT, CISA playbooks, zero trust, CACAO/OpenC2 |
| [Evidence intake, context, and case state](../../agents/security-investigation-agent/evidence-intake-context-and-case-state.md) | OCSF, ATT&CK, STIX/TAXII, TLP, FIRST CTI, forensics and custody |
| [Security integrations and adapter qualification](../../agents/security-investigation-agent/security-integrations-and-adapter-qualification.md) | Microsoft Graph/Defender, AWS CloudTrail/Security Hub, Google Logging/SCC, STIX/TAXII, Jira, ServiceNow, source-specific reliability evidence |
| [Investigation reasoning, tools, models, and runtime](../../agents/security-investigation-agent/investigation-reasoning-tools-and-runtime.md) | Typed broker synthesis, agent security, durable runtime, benchmark constraints |
| [Authority, approvals, and constrained response](../../agents/security-investigation-agent/authority-approvals-and-constrained-response.md) | Zero trust, CISA containment, CACAO/OpenC2, idempotency/reconciliation |
| [Untrusted content, forensics, and data governance](../../agents/security-investigation-agent/untrusted-content-forensics-and-data-governance.md) | Prompt injection, sandbox/containment reports, evidence preservation, privacy/sanitization |
| [Reliability, observability, scaling, and operations](../../agents/security-investigation-agent/reliability-observability-scaling-and-operations.md) | Temporal, Eventarc/Workflows, OCSF, OTel, W3C Trace Context, NIST audit/log guidance |
| [Evaluation, rollout, and build roadmap](../../agents/security-investigation-agent/evaluation-rollout-and-build-roadmap.md) | SecAlertBench, SIABench, SecRespond, AgentDojo, InjecAgent, NIST evaluation warnings |

## Refresh triggers

Refresh this packet when:

- NIST finalizes SP 800-92 Rev. 1 or publishes new agentic-security guidance;
- ISO publishes the replacement/revision of ISO/IEC 27037;
- FIRST changes CSIRT Services Framework 2.1 review status, TLP, CVSS, EPSS, or CTI uncertainty guidance;
- MITRE releases a new ATT&CK major version or changes detection-strategy/data-component structures;
- OCSF changes agent/security-finding schemas or publishes a new major/minor used by the implementation;
- OASIS revises STIX/TAXII, CACAO, or OpenC2;
- OpenTelemetry GenAI agent conventions move from Development or resolve causal/grouping semantics;
- a major public defensive-investigation benchmark is peer reviewed, independently replicated, or substantially revised;
- new adaptive prompt-injection, memory-poisoning, parser-escape, or agent-runtime incidents change the threat model;
- model, provider, runtime, source systems, authority tier, tenancy, region, or legal requirements change.
- any SIEM, SOAR, EDR, cloud, IAM, CTI, ticket/case, or CMDB API changes version, authentication, permission, pagination, retention, rate-limit, ordering, field, or asynchronous-action behavior.

## Research limitations

- Some 2026 agent-security and defensive-agent evidence is recent vendor reporting or preprint research and has not had long independent validation.
- Several foundational forensic standards are old; their principles remain useful, but cloud/SaaS acquisition and jurisdictional practice require local specialist guidance.
- Public benchmark datasets are far smaller and cleaner than the range of production SOC cases and may contain synthetic elements or label noise.
- Product API examples were reviewed only to derive qualification patterns; exact identity models, case schemas, entitlements, tenant configuration, data residency, rate limits, retention, and write semantics remain deployment-specific and must be re-tested.
- This packet did not assess offensive cyber-agent capabilities because they are unnecessary and outside the defensive scope.

## Compact source index

### Standards and government

- [NIST SP 800-61 Rev. 3](https://csrc.nist.gov/pubs/sp/800/61/r3/final)
- [NIST SP 800-86](https://csrc.nist.gov/pubs/sp/800/86/final)
- [NISTIR 8387](https://www.nist.gov/publications/digital-evidence-preservation-considerations-evidence-handlers)
- [NIST SP 800-207](https://csrc.nist.gov/pubs/sp/800/207/final)
- [NIST SP 800-207A](https://csrc.nist.gov/pubs/sp/800/207/a/final)
- [NIST SP 800-88 Rev. 2](https://csrc.nist.gov/pubs/sp/800/88/r2/final)
- [NIST AI 600-1](https://www.nist.gov/publications/artificial-intelligence-risk-management-framework-generative-artificial-intelligence)
- [CISA response playbooks](https://www.cisa.gov/sites/default/files/2024-08/Federal_Government_Cybersecurity_Incident_and_Vulnerability_Response_Playbooks_508C.pdf)
- [FIRST CSIRT Services Framework 2.1](https://www.first.org/standards/frameworks/csirts/csirt_services_framework_v2.1)
- [ISO/IEC 27037:2012](https://www.iso.org/standard/44381.html)
- [RFC 3227](https://www.rfc-editor.org/info/rfc3227)

### Security data and orchestration

- [MITRE ATT&CK](https://attack.mitre.org/)
- [MITRE D3FEND](https://d3fend.mitre.org/)
- [OCSF](https://github.com/ocsf/ocsf-schema)
- [STIX/TAXII](https://oasis-open.github.io/cti-documentation/resources.html)
- [FIRST TLP](https://www.first.org/tlp/)
- [FIRST CVSS](https://www.first.org/cvss/)
- [FIRST EPSS](https://www.first.org/epss/)
- [Sigma specification](https://sigmahq.io/sigma-specification/)
- [CACAO 2.0](https://www.oasis-open.org/standard/cacao-security-playbooks-v2-0/)
- [OpenC2](https://www.oasis-open.org/committees/tc_home.php?wg_abbrev=openc2)

### Product integration references

- [Microsoft Graph security alerts](https://learn.microsoft.com/en-us/graph/api/security-list-alerts_v2?view=graph-rest-1.0)
- [Microsoft Defender endpoint isolation](https://learn.microsoft.com/en-us/defender-endpoint/api/isolate-machine)
- [AWS CloudTrail `LookupEvents`](https://docs.aws.amazon.com/awscloudtrail/latest/APIReference/API_LookupEvents.html)
- [AWS Security Hub finding updates](https://docs.aws.amazon.com/securityhub/latest/userguide/finding-update-batchupdatefindings.html)
- [Google Cloud Logging `entries.list`](https://cloud.google.com/logging/docs/reference/v2/rest/v2/entries/list)
- [Google Security Command Center API](https://cloud.google.com/security-command-center/docs/reference/rest)
- [Jira Cloud REST API v3](https://developer.atlassian.com/cloud/jira/platform/rest/v3/intro/)
- [ServiceNow REST APIs](https://www.servicenow.com/docs/r/api-reference/rest-api-explorer/c_RESTAPI.html)

### Agent security, runtime, and evaluation

- [AgentDojo](https://proceedings.neurips.cc/paper_files/paper/2024/hash/97091a5177d8dc64b1da8bf3e1f6fb54-Abstract-Datasets_and_Benchmarks_Track.html)
- [InjecAgent](https://aclanthology.org/2024.findings-acl.624/)
- [OpenAI prompt-injection guidance](https://openai.com/safety/prompt-injections/)
- [Anthropic containment engineering](https://www.anthropic.com/engineering/how-we-contain-claude)
- [Temporal event history](https://github.com/temporalio/documentation/blob/main/docs/encyclopedia/event-history/event-history.mdx)
- [OpenTelemetry GenAI conventions](https://github.com/open-telemetry/semantic-conventions-genai)
- [SecAlertBench](https://github.com/Dxsssu/SecAlertBench)
- [SIABench](https://spectrum.library.concordia.ca/id/eprint/996238/)
- [SecRespond](https://arxiv.org/abs/2607.26791)
