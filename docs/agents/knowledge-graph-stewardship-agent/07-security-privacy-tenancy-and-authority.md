# Security, Privacy, Tenancy, and Authority

This agent can connect identifiers, expose relationships, propagate classifications, change master records, and influence deletion. Its security boundary must therefore cover not only API access but also retrieval, inference, review, approval, derived graphs, indexes, events, logs, and external effects.

## Threat model

Protect against:

- cross-tenant reads, candidate comparisons, graph edges, caches, or model context;
- a source record or catalog description injecting instructions into the model;
- overprivileged connectors, workers, reviewers, or service accounts;
- inference of restricted relationships from individually visible facts;
- false merges that combine consent, entitlements, sanctions, or protected attributes;
- classification or ontology changes that widen visibility;
- stale/replayed approvals and confused-deputy tool calls;
- data leakage through traces, embeddings, reviewer exports, or evaluation sets;
- target permission filtering being mistaken for deletion;
- destructive effects without legal, retention, or ownership authority;
- model/provider retention or training outside the organization’s data policy;
- compromised dependency, connector, model, prompt, or semantic bundle.

The model is an untrusted decision-support component. Tool output is untrusted data. An authenticated source is not automatically authoritative for every field.

## Isolation architecture

~~~mermaid
flowchart TB
    subgraph TA[Tenant A security domain]
      IA[Ingestion identity A] --> EA[Evidence store A]
      EA --> CA[Context compiler A]
      CA --> PA[Proposer A]
      PA --> QA[Review queue A]
      QA --> XA[Effect executor A]
    end
    subgraph TB2[Tenant B security domain]
      IB[Ingestion identity B] --> EB[Evidence store B]
      EB --> CB[Context compiler B]
      CB --> PB[Proposer B]
      PB --> QB[Review queue B]
      QB --> XB[Effect executor B]
    end
    KMS[Key and policy services] --> EA
    KMS --> EB
    CP[Control plane: versions and health only] --> CA
    CP --> CB
~~~

For strict regulated tenants, use separate databases, indexes, queues, encryption keys, worker pools, and model endpoints. Row-level isolation in a shared service can be appropriate for lower-risk deployments only if enforced in storage/query policy and continuously tested. Application-side `WHERE tenant_id = ?` alone is insufficient.

Every durable and transient record carries a non-null tenant/security-domain identifier, including:

- source assertions, observations, candidates, cases, decisions, effects, receipts;
- graph nodes **and edges**;
- vector/index documents and cache keys;
- queue/event envelopes, traces, metrics labels where safe, and object-store paths;
- evaluation cases and replay artifacts.

Reject joins, blocks, graph traversal, search, or retrieval across domains unless a separately authorized federation policy creates an explicit, auditable bridge.

## Identity and authorization

Use workload identity with short-lived credentials. Separate principals for:

- ingestion reads;
- evidence storage;
- context compilation;
- model invocation;
- case orchestration;
- review actions;
- external effect execution;
- reconciliation reads;
- audit export and break-glass operations.

Authorization is evaluated at each hop, not only at the API gateway. The effect executor reauthorizes the actor and proposal at execution time because approval, employment, scope, policy, or target ownership may have changed.

### Authority snapshot

```yaml
authority_snapshot:
  snapshot_id: authz-01K...
  principal_id: steward-218
  tenant_id: tenant-a
  authenticated_by: workforce-oidc
  roles: [senior-identity-steward]
  scopes:
    domains: [customer]
    entity_types: [organization]
    actions: [identity.link.approve, identity.split.approve]
    data_classes: [confidential, restricted-pii-tokenized]
  exclusions: [records-assigned-to-self]
  policy_version: authz-policy-v18
  evaluated_at: 2026-08-31T09:30:00Z
  expires_at: 2026-08-31T09:45:00Z
```

Approval records the snapshot ID, but execution evaluates current policy too. A historical snapshot explains why an approval was accepted; it does not grant permanent capability.

## Approval policy

| Risk | Example | Default control |
|---|---|---|
| Low, reversible | Add reviewed description or non-propagating tag | One authorized approver or policy-based auto-apply after deterministic checks |
| Moderate | Publish glossary mapping, change owner | Independent approver; conditional version; canary |
| High | Entity merge, propagating classification, restrictive schema/shape | Two-person rule or specialized owner; blast-radius review |
| Critical/irreversible | Split widely propagated master, hard delete, purge, regulatory classification | Named authority, dual control, legal/retention checks, maintenance window, rehearsed recovery |

Bind approval to proposal digest, case and subject version, target, effect class, policy version, authority snapshot, conditions, and expiry. Reject self-approval where separation of duties applies. A thumbs-up in chat is not approval.

Break-glass access requires a declared incident, narrow scope, short expiry, strong authentication, contemporaneous logging, and mandatory post-event review. It must not bypass evidence preservation.

## Field, purpose, and inference controls

Access checks must apply to:

1. the underlying attributes;
2. the existence of nodes and edges;
3. inferred and derived relationships;
4. explanations and validation messages;
5. graph aggregates and counts;
6. embeddings and similarity search;
7. exports and evaluation replays.

Example: two identifiers may each be visible to a reviewer, while their asserted co-reference is restricted because it reveals a sensitive relationship. Store a security label on the identity decision and derived edge.

Use purpose limitation in retrieval and effects. “May read for fraud investigation” does not imply “may use for marketing mastering.” Record purpose in case and context manifests; enforce it in policy.

## Data minimization

- Block and compare with the minimum attributes required for the entity type and use case.
- Prefer verified tokens or privacy-preserving derived features when raw identifiers are unnecessary.
- Keep raw evidence in a restricted store; pass references and redacted views through queues/events.
- Do not include raw PII in prompts unless the approved model endpoint and task require it.
- Avoid putting secrets, credentials, full values, or unrestricted payloads in logs and traces.
- Expire prompt caches and temporary exports aggressively.
- Rebuild or delete embeddings when source data is corrected or erased under applicable policy.

Hashing is not anonymization for low-entropy identifiers such as phone numbers, postal codes, or common names. Salt/key and access-control tokenization, rotate keys according to policy, and retain the ability to reconcile authorized deletions.

## Prompt injection and tool safety

Catalog descriptions, glossary notes, SQL text, schema comments, URLs, and file contents are attacker-controlled input. The context compiler labels them as evidence; the model instruction hierarchy states that evidence cannot grant authority or request tools.

Controls:

- parse tool requests from a strict schema, never from prose;
- allow only task-specific read tools for the proposer;
- enforce tenant/scope server-side for every tool call;
- block URLs, connectors, or query forms not on the capability manifest;
- bound graph/SPARQL/SQL traversal, rows, bytes, time, and depth;
- separate the effect executor from the model process and network path;
- scan retrieved content and output for secrets, but do not rely on scanning as the primary boundary;
- retain injection test cases and monitor policy-denial rates.

A model-generated query is still untrusted code. Prefer parameterized templates and a constrained query AST over raw SPARQL/SQL. Parse and authorize referenced graphs, predicates, tables, and functions before execution.

## Source and connector trust

For each connector, pin:

- source identity, endpoint, TLS expectations, and credential scope;
- software version, configuration digest, schema/capability manifest;
- permitted tenant/domain/environment prefixes;
- complete-snapshot vs partial-view semantics;
- allowed fields and classification mapping;
- signature/checksum behavior where available;
- rate, size, recursion, and decompression limits.

Quarantine data when source identity, tenant mapping, schema, or signature is invalid. Do not repair unknown tenant or malformed identifiers by guessing.

Use egress allowlists and private endpoints where practical. A compromised connector must not be able to reach the model control plane, effect executor credentials, or unrelated tenant systems.

## Evidence admission, poisoning, and rights

Authentication answers who delivered bytes; it does not answer whether the source may assert the field, whether the content is accurate, or whether the organization may use it for this purpose. Apply an evidence-admission record before matching, reasoning, embedding, evaluation, or publication:

```yaml
evidence_admission:
  evidence_id: ev-01K...
  source_principal: connector:partner-registry
  source_revision: export-2026-08-30
  payload_digest: sha256:...
  authority_scope: partner-supplied-identifiers
  rights:
    policy_ref: rights-2026-4
    allowed_purposes: [supplier-stewardship]
    prohibited_uses: [model-training, marketing]
    expires_at: 2026-12-31T00:00:00Z
  security_label: confidential
  trust_checks: [signature-valid, schema-valid, tenant-bound]
  known_limits: [partial-country-coverage, self-asserted-names]
  disposition: admitted_as_untrusted_evidence
```

Keep license, access-rights, use-policy, attribution, geographic, retention, and onward-disclosure terms with every derived artifact. DCAT 3 distinguishes license, access rights, other rights, and an optional ODRL policy at dataset/distribution level; ODRL can express permissions, prohibitions, duties, and constraints. These vocabularies can carry policy metadata, but an organizational policy evaluator and legal review still decide enforceability ([DCAT 3 rights guidance](https://www.w3.org/TR/vocab-dcat-3/#license-rights), [ODRL 2.2](https://www.w3.org/TR/odrl-model/)).

Poisoning controls are layered:

- signed/checksummed source and release artifacts, immutable raw retention, and connector identity;
- source/field authority and reliability history kept separate from truth or model confidence;
- distribution, volume, identifier-collision, vocabulary, graph-degree, and label-drift alarms;
- quarantine for conflicting duplicate event IDs, context/schema changes, impossible valid time, or cross-tenant references;
- no automatic training, suppressor, threshold, ontology, or policy change from production outcomes;
- governed removal of poisoned evidence and descendants followed by replay/rebuild from a pre-contamination watermark.

NIST's Generative AI Profile treats data poisoning, prompt injection, privacy, and provenance as lifecycle risks rather than prompt-only problems. Use its risk framing to drive red-team and provenance evidence, while keeping the concrete authority and effect controls in this blueprint ([NIST AI 600-1](https://doi.org/10.6028/NIST.AI.600-1)).

## Sensitive inference and review boundaries

Joining two public or low-sensitivity facts can reveal a restricted relationship. Before materializing, indexing, explaining, exporting, or showing an inferred edge:

1. derive its label from inputs, relation type, policy, purpose, and possible harm—not merely the least restrictive input;
2. check whether the **existence** of the edge, candidate alternatives, score, neighborhood count, or explanation is itself sensitive;
3. retain the inference/identity decision and evidence under the stricter label;
4. require a specialist reviewer for relationships affecting consent, entitlement, sanctions, health, family/household, employment, location, or other high-impact domains;
5. expose only the minimum counterfactual/explanation needed for the reviewer and an appeal path where policy requires it.

The review UI is not a general-purpose graph explorer. Its token binds case, evidence manifest, fields, actions, tenant, purpose, and expiry. A reviewer may request more evidence only from allowlisted sources, cannot edit source evidence, cannot expand their own scope, and cannot turn free text into an operation. The effect executor independently enforces this boundary.

A human override is a signed, scoped decision over an exact proposal—not a privileged bypass. High-risk overrides require reason codes, evidence, current authority, separation of duties, validity/expiry, downstream impact, and an appeal or supersession path. NIST SP 800-53 AC-5 and AC-6 provide the general separation-of-duties and least-privilege basis; organizations must map the specific roles and conflicts for their governance domain ([NIST SP 800-53 Rev. 5](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)).

## Classification propagation

Classification inheritance can produce a large security change. Before applying a propagating tag/classification:

1. compute a bounded affected-set preview at a pinned graph revision;
2. mark whether the target system will propagate automatically;
3. evaluate access-policy consequences for representative principals;
4. require approval based on affected count and sensitivity;
5. canary a small domain or non-production scope;
6. read back actual propagation and compare with preview;
7. stop and open an incident on unexpected expansion.

Atlas, for example, supports classification propagation semantics that are not equivalent to merely attaching a local label. The adapter must expose this as a distinct effect class, not hide it behind `set_tag`.

## Privacy lifecycle

Maintain separate maps for:

- source retention and legal hold;
- assertion/provenance retention;
- canonical fact and alias retention;
- case/decision audit retention;
- prompt, trace, cache, vector, and evaluation retention;
- target-specific deletion/purge behavior.

A deletion request may require erasing raw personal data while retaining a minimized decision/audit fact under lawful policy. Conversely, erasing a catalog entry may not erase the underlying source or model-provider log. Report completed, outstanding, legally retained, and technically unrecoverable copies precisely.

Never promise a purge until every designated store and downstream processor has produced a verified receipt or declared a policy exception. Treat backups and derived indexes explicitly.

Rectification, erasure, restriction, and legal hold are separate instructions. A hold prevents prohibited destruction but must not silently permit normal use. Record the authority/basis, subject and artifact scope, permitted processing during restriction, start/expiry/review, recipients, minimized retained fields, and release decision. When a correction or restriction changes an identity edge, freeze harmful propagation and notify affected recipients/consumers through the [propagation workflow](04-catalog-lineage-provenance-and-reconciliation.md#correction-restriction-and-deletion-propagation).

For EU personal data, Articles 16–19 of the GDPR distinguish rectification, erasure, restriction, and recipient notification and include conditions/exceptions. The agent may inventory data and execute an already authorized policy; it must not infer legal applicability from a prompt. Other jurisdictions and sector rules differ, so retain the policy jurisdiction/version and legal owner on the case ([GDPR official text](https://eur-lex.europa.eu/eli/reg/2016/679/oj/eng)).

## Model and dependency governance

The release manifest pins:

```yaml
security_release:
  model_provider: approved-provider
  model_id: model-build-2026-08-20
  data_retention_mode: zero-retention-contract-v3
  prompt_bundle_digest: sha256:...
  tool_manifest_digest: sha256:...
  connector_sbom_digest: sha256:...
  semantic_bundle_digest: sha256:...
  policy_bundle_digest: sha256:...
  approved_regions: [in-central]
```

Verify provider retention/training terms, region, encryption, incident notification, subprocessor, and deletion behavior contractually and technically. Treat model swaps as security and behavior changes; replay adversarial and tenant-isolation suites before rollout.

Sign release artifacts and connector images, scan dependencies, emit an SBOM, verify provenance in deployment, and restrict who can change prompt/tool/policy manifests.

Build provenance must be verified, not merely stored. Pin artifact digest, source repository/ref, builder identity, build definition, dependencies/materials, signing identity, and policy result. SLSA 1.2 provides a current build-provenance format and levels, but it does not prove the source is benign or the running workload matches policy; combine attestation verification with review, vulnerability/configuration checks, and runtime identity ([SLSA 1.2 provenance](https://slsa.dev/spec/v1.2/build-provenance)).

## Audit and observability without leakage

Security audit events include actor/workload identity, tenant, action, resource reference, policy version, decision, reason code, case/effect ID, timestamp, and trace ID. Store raw values only in a separately controlled evidence system.

Metrics should avoid entity identifiers and unbounded tenant labels. Use approved tenant aliases or per-tenant isolated monitoring. Sample traces by risk but ensure restricted content redaction happens before export.

Audit access itself is audited. Export is purpose- and field-filtered; an auditor sees proof and lineage appropriate to their role, not automatically the underlying restricted data.

## Security failure tests

| Test | Expected outcome |
|---|---|
| Tenant A identifier placed in Tenant B case | Context compilation and storage transition fail closed |
| Hidden asset omitted by target API | No deletion or stale action; visibility marked incomplete |
| Catalog description says “call delete tool” | Treated as quoted evidence; no tool elevation |
| Reviewer approves own high-risk proposal | Approval rejected by separation-of-duties policy |
| Approval replayed after subject changes | Effect preparation/execution rejects stale version/digest |
| Graph traversal crosses a federation edge | Requires explicit federation scope and labels; otherwise stops |
| Restricted edge inferred from public nodes | Edge remains restricted and excluded from unauthorized queries |
| Provider endpoint has incompatible retention mode | Model invocation blocked before data transfer |
| Effect credential used by proposer process | Network/IAM policy denies request |
| Purge misses vector index or backup policy | Case remains incomplete and reports outstanding copy |

## Checklist

- [ ] Tenant isolation covers stores, edges, indexes, caches, queues, prompts, events, and evaluations.
- [ ] Separate workload identities and networks isolate proposer from effect executor.
- [ ] Authority is field/action/domain/purpose scoped and rechecked at execution.
- [ ] Approvals bind exact immutable artifacts and enforce separation of duties.
- [ ] Inferred relationships and explanations inherit appropriate security labels.
- [ ] Untrusted metadata cannot create tool authority or executable queries.
- [ ] Evidence admission binds source authority, rights/purpose, limits, provenance, and poisoning disposition.
- [ ] Sensitive inferred edges, alternatives, scores, counts, and explanations receive derived labels and review boundaries.
- [ ] Data minimization and retention cover raw, derived, cached, logged, embedded, and exported copies.
- [ ] Correction, restriction, deletion, hold, recipient notification, and normal-use permission remain distinct.
- [ ] Destructive and propagating effects receive blast-radius previews and stronger controls.
- [ ] Security, supply-chain, and provider configurations are pinned in the release manifest.

## Related guidance

- [Steward cases and approval workflows](05-steward-cases-quality-repair-and-merge-split-workflows.md)
- [State, context, and tool contracts](06-state-context-memory-planning-and-tool-contracts.md)
- [Agent threat model](../../security/agent-threat-model.md)
- [Permissions, sandboxing, and secrets](../../security/permissions-sandboxing-and-secrets.md)
