# Untrusted Content, Forensics, and Data Governance

> **Status:** Research-backed production blueprint  
> **Last researched:** 2026-08-31  
> **Scope:** Indirect prompt injection, malicious artifacts, analysis isolation, evidence integrity, privacy, retention, legal hold, and tenant boundaries.  
> **Section index:** [Security investigation and triage agent](README.md)

Security evidence is adversarial by nature. Logs, email, webpages, documents, repositories, malware strings, CTI reports, case notes, and tool errors can all contain text intended to manipulate analysts or models. An authenticated connector proves where data came from; it does not grant that data authority.

## Security invariant

> **Evidence may influence a finding. It may not change instructions, permissions, tool definitions, policy, credentials, or effect authorization.**

The system must remain safe if the model follows an injected instruction perfectly. Model training and classifiers reduce likelihood; isolation and complete mediation bound impact.

## Source-to-sink threat model

~~~mermaid
flowchart LR
    subgraph Sources["Attacker-influenced sources"]
        EMAIL["Email / ticket / chat"]
        WEB["Web / CTI report"]
        LOG["Log / process / command text"]
        FILE["Document / archive / malware"]
        CASE["Prior case / memory"]
    end

    Sources --> PARSE["Constrained parsing and labeling"]
    PARSE --> EVID["Untrusted evidence lane"]
    EVID --> MODEL["Model proposes"]
    MODEL --> READ["Read-query broker"]
    MODEL --> WRITE["Case/action proposal"]
    READ --> POLICY["Deterministic policy"]
    WRITE --> POLICY
    POLICY --> EXEC["Separated executor"]
    POLICY -. deny .-> AUDIT["Security audit"]
    EXEC --> AUDIT
~~~

Dangerous sinks include more than response actions:

- data transfer to a new destination;
- broad search of sensitive telemetry;
- case notes that trigger automation or influence another analyst;
- durable memory, rule, playbook, prompt, tool, or policy updates;
- active rendering, link fetching, macro execution, or artifact detonation;
- credential requests or permission expansion.

## Typed trust lanes

| Lane | Examples | May issue instructions? | May authorize? |
|---|---|---:|---:|
| Platform policy | Signed deployment configuration, policy service | Yes, within versioned control plane | Through deterministic evaluation |
| Investigation objective | Authenticated analyst request, case assignment | Yes, within user authority | No direct effect |
| Verified facts | Canonical tenant, asset, identity, policy result | No; factual input | No |
| Untrusted evidence | Alerts, logs, email, documents, CTI, tool output | No | No |
| Model proposal | Query, claim, disposition, response plan | No | No |
| Approval | Exact-effect approval record | No natural-language expansion | Only as one policy input |

Serialize lane metadata outside source-controlled text. Labels inside the prompt are useful but are not the enforcement boundary.

## Prompt-injection defenses

### Admission

- Mark every external field with source, tenant, trust, and sensitivity.
- Strip or neutralize active rendering while preserving raw evidence separately.
- Reject recursive archives, extreme compression ratios, malformed encodings, and unsupported active formats.
- Do not follow source-provided URLs during parsing.
- Scan for known injection patterns as a detection signal, not a complete firewall.

### Context assembly

- Keep control instructions and untrusted evidence in separate structured fields.
- Provide excerpts needed for the objective, not entire mailboxes, repositories, or disk images.
- Remove secrets and unrelated personal data before model access.
- Preserve citations to raw evidence so summarization is not trusted.
- Describe allowed transformations and tool purposes explicitly.
- Never place an external document in a system/developer instruction field.

### Tool boundary

- Ignore instructions found in tool output.
- Bind each query to the case objective and a named hypothesis.
- Block tool, scope, tenant, destination, and permission changes derived from evidence.
- Deny egress that is not required for an allowlisted source adapter.
- Keep response tools out of the read-only tool catalog.

### Output boundary

- Validate schemas and citation existence.
- Reject case notes that contain executable markup or active links when plain text is sufficient.
- Mark all model-authored findings as proposals.
- Require independent policy and approval for any sink.
- Gate durable memory, detection-rule, playbook, and prompt changes as privileged effects.

### Evaluation

Test:

- visible and hidden instructions in every supported source type;
- multilingual, encoded, fragmented, and multi-turn attacks;
- instructions embedded in tool errors and analyst-like case notes;
- exfiltration through queries, links, notifications, case writes, or diagnostics;
- attempts to weaken logging, retention, policy, or tenant scope;
- adaptive attacks that know the defenses;
- benign documents discussing prompt injection, malware commands, or response procedures.

Static benchmark success does not prove the system is secure. AgentDojo and InjecAgent establish realistic tool-output attack classes; later research found flaws and weak attacks in several static benchmarks. Maintain adaptive internal tests.

## Artifact handling zones

~~~mermaid
flowchart TD
    IN["Artifact intake"] --> Q["Quarantine object store"]
    Q --> META["Metadata and signature checks"]
    META --> STATIC["Static parser sandbox"]
    STATIC --> SAFE["Sanitized derived artifacts"]
    SAFE --> AGENT["Investigation context"]
    Q --> SPECIAL{"Dynamic analysis justified?"}
    SPECIAL -- No --> HOLD["Preserve / specialist review"]
    SPECIAL -- Yes --> DET["Dedicated detonation environment"]
    DET --> REPORT["Behavior report + captured evidence"]
    REPORT --> SAFE
    DET --> DESTROY["Destroy analysis environment"]
~~~

### Zone 1: quarantine

- Object storage does not render or execute content.
- Access is case- and tenant-scoped.
- Filenames are metadata, not paths.
- Original bytes and initial digest are immutable.
- Automated systems cannot make the object public or attach it to normal collaboration tools.

### Zone 2: static parsing

Run parsers in disposable workers with:

- read-only access to one analysis copy;
- no production mounts, container socket, instance metadata, or response credential;
- default-deny egress;
- CPU, memory, file-count, process, recursion, and wall-clock limits;
- no macros, scripts, external entities, embedded links, or plugin loading;
- pinned parser/tool images and recorded digests;
- captured stdout, stderr, exit status, and output manifest.

Treat parsers as attack surface. File-format exploits are conventional vulnerabilities, independent of prompt injection.

### Zone 3: dynamic analysis

Dynamic malware analysis is a specialist capability, not a normal agent tool. If authorized:

- use dedicated disposable VM or equivalent isolation appropriate to the threat;
- separate management and simulated/detonation networks;
- block access to enterprise, cloud metadata, credentials, and arbitrary internet;
- use controlled services or sinkholes under organizational policy;
- prevent escape through shared clipboard, folders, devices, or host agents;
- snapshot before analysis and destroy after evidence export;
- record environment, tool, image, network, time, and sample digests;
- require a human or pre-approved playbook to authorize detonation.

Do not allow the agent to improvise execution commands or change isolation.

### Example isolation profile

~~~yaml
artifact_worker_policy:
  image_digest: sha256:4f9a...
  filesystem:
    root: read_only
    input_mount: read_only
    output_mount: write_only
    max_output_bytes: 50000000
  process:
    max_processes: 32
    wall_time_seconds: 120
    cpu_seconds: 60
    memory_mb: 1024
  network:
    mode: none
  credentials:
    mounted: false
  active_content:
    macros: disabled
    external_entities: disabled
    embedded_urls: no_fetch
    scripts: disabled
~~~

This is an intent contract; enforce it with mature OS, VM, identity, and network controls and verify the actual boundary.

## Forensic integrity

### Preserve originals, analyze copies

NIST SP 800-86 recommends integrity verification and analysis of copies. NISTIR 8387 discusses hashes, signatures, secure storage, access controls, and preservation. RFC 3227 recommends collection before analysis when possible and prioritizing volatile evidence.

Workflow:

1. Decide acquisition authority and preservation requirements.
2. Identify sources and order by volatility, value, and collection impact.
3. Acquire with documented tool and method.
4. Hash and register original object.
5. Store original under restricted custody.
6. Create a verified working copy.
7. Analyze the copy in an isolated environment.
8. Record every transformation and derived artifact.
9. Re-verify integrity on access, transfer, migration, and scheduled audit.

### Append-only custody event

~~~yaml
custody_event:
  evidence_id: ev_991
  sequence: 7
  event: analysis_copy_created
  occurred_at: 2026-08-31T04:32:12Z
  actor:
    type: workload
    id: forensic-copy-service
  from_location: vault://region-in/case_781/original
  to_location: analysis://job_77/input
  source_sha256: 8b6d...
  destination_sha256: 8b6d...
  tool:
    name: evidence-copy
    version: 2.4.0
    build_digest: sha256:92a1...
  authorization_ref: preservation-order-18
~~~

Custody events need an append-only sequence, authenticated actor, trusted timestamp, reason, and integrity protection. Access telemetry alone is not a complete custody record.

### Hash mismatch

On mismatch:

- stop using the affected copy;
- preserve both expected and observed digests;
- quarantine the object and dependent derived artifacts;
- notify the evidence custodian;
- check transfer, storage, migration, and tool logs;
- recover from an independently verified copy when possible;
- document the disposition.

Never overwrite the registered digest to make the check pass.

## Tenant and regional boundaries

Tenant isolation must be enforced before retrieval or ranking:

- derive tenant from authenticated workload and run;
- partition or filter every source query, cache, object key, index, case, trace, and queue;
- include tenant in compound identifiers and idempotency keys;
- use tenant-aware encryption and access policy where risk requires;
- block cross-tenant joins in the query compiler;
- prevent shared embedding or search indices from returning unauthorized chunks;
- keep response executors tenant-specific or strongly policy-partitioned;
- test malicious canonical-ID collisions and confused-deputy calls.

Region is another policy dimension. Model endpoint, logs, traces, evidence store, backup, support access, CTI sharing, and disaster recovery may cross borders even when the primary database does not.

### Managed security providers

For an MSSP:

- never allow one customer's examples, prompts, tools, memory, or analyst notes to become another's context;
- separate customer-specific policies and response identities;
- preserve the identity of both MSSP operator and customer authority;
- treat aggregate benchmarking datasets as a separate governed data product;
- require explicit agreements for cross-customer indicators or learnings;
- apply TLP and other sharing policies to derived intelligence.

## Privacy and data minimization

Security telemetry can reveal employee behavior, communications, location, health, customer activity, and secrets. Define:

- purpose and lawful/organizational basis;
- data classes and fields needed for each alert family;
- allowed subjects and use;
- region and processor/model boundaries;
- default retention and case-based extensions;
- legal-hold override;
- access, correction, export, and deletion procedures;
- secondary-use policy for eval, tuning, or product analytics.

### Minimize at each layer

| Layer | Minimization |
|---|---|
| Intake | Drop unsupported attachments only after policy decision; avoid collecting unrelated sources |
| Normalized event | Keep needed fields while retaining restricted raw object separately |
| Model context | Use relevant excerpts and pseudonymous IDs where identity is not needed |
| Trace | Metadata by default; sensitive content opt-in and access-controlled |
| Eval | De-identify, tokenize, or synthesize while preserving task properties; prevent split leakage |
| Case report | Include necessary observables; restrict personal narrative |

Redaction is a transformation. Record what rule and version produced it; keep the original only where policy permits.

## Retention, deletion, and legal hold

Use separate policies for:

- raw security events;
- evidence under custody;
- normalized/searchable data;
- case records;
- model inputs and outputs;
- tool responses;
- operational traces and metrics;
- eval datasets and annotations;
- backups and replicas.

Retention state machine:

~~~mermaid
stateDiagram-v2
    [*] --> Active
    Active --> ScheduledDeletion: retention expires
    Active --> LegalHold: authorized hold
    ScheduledDeletion --> Deleted: primary and derived copies removed
    ScheduledDeletion --> LegalHold: hold arrives before deletion
    LegalHold --> Active: hold released and retention remains
    LegalHold --> ScheduledDeletion: hold released after retention
    Deleted --> SanitizationVerified
~~~

Deletion must propagate to derived excerpts, embeddings, caches, model-provider storage where contractually supported, replicas, and scheduled backups according to policy. Preserve a minimal deletion audit without retaining the deleted sensitive content. NIST SP 800-88 Rev. 2 is the current media-sanitization baseline; cloud logical deletion still requires provider- and architecture-specific validation.

Legal hold should prevent ordinary deletion, not silently grant broader model access.

## Secret handling

- Do not put API keys, refresh tokens, private keys, session cookies, or broad bearer tokens in prompts, evidence excerpts, environment variables visible to analysis, or traces.
- Detect and redact credentials before model context while preserving restricted original evidence if authorized.
- Use credential brokers to mint short-lived tokens directly to adapters/executors.
- Prevent source-controlled URLs from receiving credentials through redirects or server-side fetch.
- Rotate exposed credentials through a human-directed incident process, not an automatic model side effect.

## Memory and learned content

Default to no autonomous long-term memory from investigations. If institutional learning is needed:

- promote only analyst-confirmed, de-identified patterns;
- retain source case, tenant/sharing policy, evidence basis, author, review, freshness, and expiry;
- separate detection content, runbooks, examples, and factual asset data;
- require review for prompt, tool, policy, playbook, rule, or memory changes;
- test poisoned prior cases and delayed activation;
- support revocation, supersession, and deletion.

Past analyst closure is useful evidence but not infallible ground truth.

## Failure matrix

| Threat or failure | Hard boundary | Detection |
|---|---|---|
| Injection asks for new tool/permission | Fixed catalog and policy | Denied scope-change event |
| Log line contains fake system message | Typed evidence lane | Injection canary and output grader |
| Malicious document exploits parser | Disposable restricted parser | Crash, resource, sandbox, and integrity telemetry |
| Sample reaches enterprise network | Default-deny isolated network | Egress policy alert |
| Cross-tenant retrieval | Authorization before search | Tenant mismatch and synthetic canary |
| Secret appears in trace | Redaction and content-off default | DLP scan and access audit |
| Poisoned prior case changes policy | No model-writable policy/memory | Config digest and review gate |
| Evidence object mutates | Immutable storage and digest | Integrity audit |
| Legal-hold data deleted | Hold evaluated before lifecycle transition | Deletion reconciliation alert |
| Deletion leaves embeddings/cache | Deletion lineage and per-store workflow | Tombstone reconciliation report |

## Acceptance checklist

- [ ] All evidence lanes are structurally non-authoritative.
- [ ] Unsupported active content is quarantined before model access.
- [ ] Static and dynamic analysis runtimes have no production credential or unrestricted egress.
- [ ] Every acquired object has provenance, digest, tool/method, and custody.
- [ ] Originals are preserved and analyses use verified copies.
- [ ] Tenant authorization occurs before search, ranking, cache, or model context.
- [ ] Model and trace content collection is minimized and access-controlled.
- [ ] Retention, deletion, legal hold, backup, and provider-storage behavior are tested.
- [ ] Durable memory and configuration updates require reviewed promotion.
- [ ] Adaptive prompt-injection and benign hard-negative tests run before release.

## Related guides

- [Evidence intake, context, and case state](evidence-intake-context-and-case-state.md)
- [Authority, approvals, and constrained response](authority-approvals-and-constrained-response.md)
- [Prompt injection and untrusted data](../../security/prompt-injection-and-untrusted-data.md)
- [Permissions, sandboxing, and secrets](../../security/permissions-sandboxing-and-secrets.md)
- [Memory architecture](../../context-memory/memory-architecture.md)

## Selected sources

- [NIST SP 800-86](https://csrc.nist.gov/pubs/sp/800/86/final)
- [NISTIR 8387, Digital Evidence Preservation](https://www.nist.gov/publications/digital-evidence-preservation-considerations-evidence-handlers)
- [RFC 3227, Evidence Collection and Archiving](https://www.rfc-editor.org/info/rfc3227)
- [ISO/IEC 27037:2012](https://www.iso.org/standard/44381.html)
- [SWGDE Best Practices for Computer Forensic Acquisition](https://www.swgde.org/documents/published-complete-listing/17-f-002-2-1/)
- [NIST SP 800-88 Rev. 2, Media Sanitization](https://csrc.nist.gov/pubs/sp/800/88/r2/final)
- [NIST Privacy Framework](https://www.nist.gov/privacy-framework)
- [NIST AI 600-1, Generative AI Profile](https://www.nist.gov/publications/artificial-intelligence-risk-management-framework-generative-artificial-intelligence)
- [AgentDojo](https://proceedings.neurips.cc/paper_files/paper/2024/hash/97091a5177d8dc64b1da8bf3e1f6fb54-Abstract-Datasets_and_Benchmarks_Track.html)
- [InjecAgent](https://aclanthology.org/2024.findings-acl.624/)
- [OpenAI, Understanding prompt injections](https://openai.com/safety/prompt-injections/)
- [Anthropic, How we contain Claude across products](https://www.anthropic.com/engineering/how-we-contain-claude)
