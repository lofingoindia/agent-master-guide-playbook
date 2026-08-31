# Connector Qualification and Adapter Playbooks

> **Purpose:** Decide whether a source can safely join the enterprise corpus, then implement its identity, change, permission, deletion, capacity, and reconciliation semantics without pretending every API behaves alike.

## Qualify before you integrate

A connector is admitted only after a production-like proof demonstrates that it can preserve the product's source, authorization, freshness, and deletion contracts. A successful OAuth flow and a page of search results prove almost nothing.

Create a versioned capability card for every deployed tenant and connector version:

```yaml
connector_capability:
  connector_type: google_drive
  adapter_version: drive_v3_7
  tenant_binding: workspace_customer_id
  source_identity: file_id
  version_identity: revision_id_or_content_hash
  inventory: full_list_with_shared_drives
  incremental: changes_page_token
  notification_role: latency_hint_only
  permissions:
    model: direct_inherited_domain_anyone_limited_access
    effective_acl_derivable: conditional
    live_recheck: files_get_capabilities
  deletion:
    source_states: [trashed, permanently_deleted, access_lost]
    derivative_cascade: lineage_key
  quotas: discovered_at_installation
  residency_and_terms: reviewed
  reconcile_interval: PT6H
  last_capability_probe: "2026-08-31T00:00:00Z"
```

The values above illustrate the contract, not a universal Drive configuration. Record the actual API edition, scopes, tenancy model, plan/license, feature flags, region, and observed behavior. Re-run the probe after provider, permission-model, or app-installation changes.

Reject or restrict a connector when it cannot:

- bind every observation to a stable tenant and source object;
- distinguish content revision, metadata change, move, deletion, and loss of connector access;
- produce or safely recheck the access policy needed for the product's authorization objective;
- enumerate enough of the corpus to reconcile missed changes;
- cascade deletion and revocation through every derived store;
- operate within documented quotas without starving revocations;
- satisfy source terms, retention, residency, audit, and user-notice obligations.

If authorization cannot be projected reliably, safer options are per-user live retrieval, a deliberately public subset, source-native search links, or excluding the source.

## Qualification sequence

Use the same admission sequence for every source while keeping source-specific semantics inside the adapter.

1. **Scope one tenant and one collection.** Inventory object types, volumes, size distribution, permission patterns, update rate, languages, attachments, and deletion paths.
2. **Build a capability probe.** Exercise every required API with least-privilege delegated and application identities. Record absent fields, plan-dependent behavior, pagination, quotas, and error distinctions.
3. **Freeze adversarial fixtures.** Include nested groups, external guests, public links, direct-plus-inherited access, moves, edits during pagination, large objects, malformed attachments, deletion, restore, and credential revocation.
4. **Prove initial convergence.** Take a scan boundary when possible, enumerate by stable ID, replay changes, and compare counts, revisions, permission roots, and sampled hashes.
5. **Prove incremental convergence.** Duplicate, delay, reorder, and drop notifications. Expire a cursor. The adapter must converge through idempotent reads and reconciliation.
6. **Prove restrictive change priority.** Revoke a group, restrict a parent, remove a sharing link, delete an item, and deauthorize the connector while a backfill runs.
7. **Prove query-time behavior.** Test allowed and denied principals, zero-result/existence leakage, citation drill-down, cache invalidation, group staleness, and long-running requests that cross a revocation.
8. **Prove rebuild and removal.** Rebuild projections from the source ledger, then uninstall the connector and execute the tenant's disposition policy.
9. **Load and cost test.** Measure source calls, bytes, parse CPU, embedding work, queue age, revocation lag, and reconciliation duration at expected and burst volumes.
10. **Assign owners and objectives.** Name the source owner, adapter owner, security owner, records owner, on-call route, freshness objective, and revocation/deletion objective.

No provider badge substitutes for this test pack.

## Source-family qualification matrix

| Source family | Stable identity and revision | Permission trap | Change/deletion trap | Safe starting mode |
|---|---|---|---|---|
| Google Drive | File ID plus revision or content/metadata digest | Direct, inherited, domain, anyone, shared-drive, and limited-access semantics differ | Parent moves and permission changes require hierarchy-aware refresh; tokens can become unusable | One shared drive or folder tree with reconciliation |
| SharePoint / OneDrive | Site, drive, and item IDs; eTag/cTag where applicable | Hierarchical sharing scans can require elevated application permission; user drill-down is still separate | Delta reports latest state, can repeat items, and needs stable-ID tracking | One site/drive with least-privilege scan and query-time ACL tests |
| Confluence | Cloud/site, space, content ID, version | Space permission, content restriction, guests, external collaborators, and app-access rules combine | Webhooks and APIs are hints/views; restrictions and deletion must be re-read and reconciled | Selected spaces, read-only, explicit restriction projection |
| Slack | Enterprise/workspace, conversation ID, message timestamp, thread timestamp | Token type, scopes, app membership, channel type, Connect membership, and retention affect visibility | Events are best effort and retried; edits, deletes, threads, retention, and history availability must converge | Selected approved channels; exclude DMs by default |
| Email | Tenant/mailbox, folder, immutable message ID when available, provider change token | Mailbox delegation, shared mailboxes, labels/folders, BCC, sensitivity labels, and per-user purpose | Change windows/cursors expire; move, soft delete, hard delete, retention, and journaling differ | Selected functional mailbox or case folder, not every employee mailbox |
| Object store | Account/bucket/key/version or generation | Bucket policy, object ACL, tags, encryption context, signed links, and requester identity differ | Notifications may be duplicated, delayed, unordered; delete markers are not physical deletion | Versioned prefix with inventory and lifecycle-aware reconciliation |
| Database / warehouse | Database/schema/table/primary key plus transaction or snapshot position | Row/column policy, view logic, masking, session identity, and purpose cannot be flattened into chunk ACLs casually | CDC retention can expire or retain logs indefinitely; schema changes and hard deletes need contracts | Governed view, read replica, or approved report before CDC |
| Search / vector system | Index/namespace/point ID plus projection generation | Filters, role composition, aggregates, caches, and approximate search have engine-specific limits | It is derived state; delete/upsert acknowledgements need read-back or reconciliation | Retrieval projection rebuilt from the source ledger |
| MCP server | Server identity, dated protocol, capability/schema hash, resource/tool identity | Protocol authorization does not define application policy; token audience and tenant binding remain local controls | Capabilities and schemas can change; remote state/effects may be ambiguous | Narrow read-only tools behind the same adapter and policy gateway |

The matrix is a test agenda, not a guarantee. Hosted edition, plan, administrator settings, API scopes, and source configuration can change behavior within the same product family.

## Drive and SharePoint playbook

### Google Drive

Google Drive permissions can be direct or inherited and may apply to users, groups, a domain, or anyone. Limited-access folders add another state that a connector must capability-probe. Preserve permission origin and link/discovery behavior rather than reducing access to a flat list of emails.

Use `fileId` as identity; paths and names are attributes. Model My Drive and shared drives explicitly. After a move or parent permission change, recompute the affected inheritance region or deny it until verified. A parent revocation does not remove an independently granted child permission, so a subtree assumption can create false denials or false access.

The changes feed is an incremental observation channel, not an authorization oracle. Commit the next durable checkpoint only after page effects are recorded. A full or partitioned inventory must remain available to repair missed hierarchy, access, and deletion state.

### SharePoint and OneDrive

Bind items to tenant, site, drive, and item IDs. Microsoft Graph `driveItem: delta` reports latest state, may repeat an item, and documents cases where descendant paths are not replayed after a folder rename. Apply the last observation by stable ID and verify current source state before publishing a disputed version.

Treat hierarchical permission scanning as a separately approved indexing capability. Graph documents permission-change scan headers and notes that the strongest scan path can require `Sites.FullControl.All`; that is materially broader than ordinary content reading. If the organization will not accept that scope, choose per-user live retrieval, a constrained site subset, or another source-native mechanism instead of hiding the gap.

For both families, distinguish source deletion, recycle-bin/trash state, lost application access, user revocation, and tenant deauthorization. They have different restoration and disposition rules.

## Confluence and Slack playbook

### Confluence

Inventory spaces, pages, blog posts, comments, whiteboards/databases if supported, and attachments as distinct types. Preserve content ID, version, ancestors, space, status, canonical URL, body representation, and restrictions. A page-level restriction is not the whole permission model: effective access can also depend on space permissions, user status, guest/external-collaborator rules, and app access.

Use webhooks to reduce latency, then fetch the authoritative current representation and restrictions. Reconcile page inventories, versions, restrictions, attachments, and removed content. Rate-limit every request family independently and honor `429`/reset guidance. The deployment card must state whether it uses Forge, OAuth, or another integration model because scopes and app-access rules differ.

### Slack

Decide first whether indexing conversational content is proportionate. Direct messages, private channels, Slack Connect, deleted messages, legal holds, and employee-monitoring concerns usually need narrower policy than ordinary documents.

If admitted, preserve workspace, channel, message timestamp, thread root, edit state, deletion state, author, membership context, and captured time. A thread is an ordered conversation projection, not a stable factual document. Do not promote chat claims into durable domain memory.

Slack documents Events API delivery as best effort with retries and requires quick acknowledgement; enqueue idempotently, deduplicate by event identity, and use history reads plus reconciliation to repair gaps. History access and quotas depend on token type, scopes, app membership, and distribution model. Discover effective limits at installation and honor `Retry-After`; never hard-code a marketing-era rate table as a capacity guarantee.

## Email and object-store playbook

### Email

Prefer a shared functional mailbox, case folder, or explicitly opted-in account over organization-wide ingestion. Preserve provider message identity, internet message ID as secondary evidence, mailbox/folder, thread/conversation identity, sent/received time, participants by role, headers needed for provenance, attachment lineage, label state, and deletion state.

Threads are presentation constructs: subject changes, aliases, forwarding, BCC, and provider-specific conversation grouping can make them unreliable evidence identity. Cite a captured message version and attachment, not only a mutable thread.

Gmail documents that `historyId` ranges can expire and require a full synchronization. Microsoft Graph exposes message delta per folder, so moves and folder scope must be tested explicitly. Treat notifications as hints. Qualify soft delete, hard delete, retention/hold, delegated mailbox access, sensitivity labels, and credential revocation with the organization's mail and records owners.

### Object stores

Use bucket/container plus object key and version ID or generation. Record etag/checksum semantics, metadata/tags, encryption/key context, storage class, retention/lock state, and lifecycle policy. An overwrite, new version, delete marker, permanent version delete, lifecycle expiry, and loss of decryption permission are distinct observations.

Amazon S3 and Google Cloud Storage document at-least-once notification behavior; ordering is not a safe assumption. Use version/generation preconditions, idempotent observation keys, inventory or listing reconciliation, and lifecycle-specific events where available. Never infer that a delete marker removed retained versions, backups, derived chunks, or evaluation artifacts.

## Database, search, vector, and MCP playbook

### Databases and warehouses

Start with governed SQL views, read-only stored reports, or exported snapshots. Use a free-form query agent only when the analytics boundary explicitly owns query generation; enterprise knowledge should consume approved structured evidence and preserve query, parameters, schema version, snapshot/as-of time, and result digest.

Before CDC, qualify primary keys, transaction ordering, snapshot handoff, update-before/delete data, schema evolution, row/column policy, masking, and backfill. PostgreSQL documents that logical slots can replay data after crash and can retain WAL/catalog resources; consumers must deduplicate and operators must monitor slot lag and storage. Equivalent guarantees vary by database and managed CDC product.

Never turn a database row into globally visible text because the indexing service account could read it. Either project an effective authorization policy with tested semantics or query a governed view at request time.

### Search and vector systems

Treat lexical, vector, and graph indexes as replaceable projections. The connector contract targets the source ledger; index writers publish a generation with source version, ACL version, parser/chunker version, embedding model, and projection receipt.

Capability-test mandatory tenant and ACL predicates on every query path, including hybrid search, reranking fetches, aggregates, suggestions, scrolling, backups, and debug endpoints. Elastic documents role-composition and aggregate-information limitations for document-level security. Vector systems expose different namespace, shard, and metadata-filter isolation patterns; none removes the need for server-injected tenant binding and cross-tenant tests.

Deletion requires enumerating all point/chunk IDs by lineage, issuing the mutation, checking completion semantics, and reconciling counts/samples. A successful delete response is not evidence that every replica, cache, snapshot, graph edge, or retained trace satisfies the data policy.

### MCP servers

MCP is an interoperability boundary, not a trust shortcut. Pin the dated protocol negotiated in production and retain a capability manifest containing server identity, advertised resources/tools, schemas, annotations, cache metadata, and authorization configuration. Run contract tests when that manifest changes.

Expose enterprise retrieval as narrow operations such as `search_authorized_corpus`, `fetch_evidence`, and `get_source_status`. Keep tenant, subject, purpose, policy, quotas, and citation reauthorization in server-owned state. The current MCP authorization design requires audience-aware tokens and prohibits token passthrough; local policy still decides which resources and tools the subject may use.

The 2026-07-28 protocol removes protocol-level sessions; application continuity must use explicit handles and durable state. Do not infer that a connection or model conversation is an authorization session. Treat tool results as untrusted inputs, validate schemas and sizes, and keep external writes behind the exact approval/effect protocol.

## Permission and deletion convergence

Measure convergence as a join across source and projections, not as connector job success:

```text
converged(object_version) =
    source_observation durable
    AND content/version identity consistent
    AND effective access version publishable
    AND lexical/vector/graph generations agree
    AND citation target resolves under current identity
    AND no newer restrictive or deletion observation is pending
```

Use a restriction-first two-phase publication rule:

1. A broadening change becomes searchable only after content and authorization projections are ready.
2. A restriction, delete, legal disposition, or access-loss observation removes visibility immediately or marks the scope fail-closed.
3. Background work cascades the change to chunks, vectors, graphs, caches, evidence manifests, saved artifacts, evaluation copies, and governed telemetry.
4. A reconciler reads authoritative state and closes the operation only when every required derivative has a receipt or an approved retention exception.

Keep a non-sensitive tombstone so late notifications and old backfills cannot resurrect an item. The tombstone references source identity, disposition reason, source/version boundary, time, hold policy, derivative receipts, and verifier outcome—not deleted text.

## Capacity, cutover, and rollback

Estimate each connector as a pipeline, not calls per second alone:

```text
required_workers = arrival_rate * mean_service_time / target_utilization
backfill_duration = total_objects / sustainable_end_to_end_objects_per_second
revocation_headroom = reserved_capacity_for_restrictions_and_deletes
```

Measure API calls, downloaded bytes, parse seconds, embedded tokens, index mutations, permission lookups, and reconciliation reads per object class. Isolate discovery, content, ACL, parse, projection, and delete queues. Reserve capacity for restrictive changes even during backfill.

Cut over with dual observation and a shadow generation:

- freeze adapter/config/capability versions;
- run old and new adapters against the same fixture and production-safe sample;
- diff object identity, versions, ACLs, tombstones, and derived manifests;
- publish the new generation only after security and retrieval gates pass;
- retain the last compatible adapter and projection during the rollback window;
- if the source has advanced, roll back serving but continue recording raw observations so repair does not lose changes.

## Connector acceptance checklist

- [ ] The capability card records actual edition, scopes, tenant binding, API/protocol version, quotas, and data terms.
- [ ] Stable object, version, hierarchy, permission, and deletion identities are proven with fixtures.
- [ ] Full inventory, incremental change, cursor expiry, duplicates, reordering, and reconciliation converge.
- [ ] Restrictive ACL, group, link, deletion, and connector-deauthorization paths fail closed within an approved objective.
- [ ] Query, citation, cache, aggregate, and long-running-request authorization tests reveal no inaccessible content or existence signal.
- [ ] Source-specific retention, legal hold, restore, and purge states map to explicit derivative behavior.
- [ ] Revocation queues retain capacity during backfill, reindex, provider throttling, and partial outage.
- [ ] Adapter/schema/provider changes trigger a capability probe, shadow comparison, and rollback decision.
- [ ] Source owners and operators can reconcile a named object across source, ledger, indexes, caches, artifacts, and tombstones.
- [ ] The connector can be removed without leaving unknown credentials, content, vectors, graph edges, or background jobs.

## Primary sources

- [Google Drive sharing and permission inheritance](https://developers.google.com/workspace/drive/api/guides/manage-sharing)
- [Google Drive change tracking](https://developers.google.com/workspace/drive/api/guides/about-changes)
- [Microsoft Graph `driveItem: delta`](https://learn.microsoft.com/en-us/graph/api/driveitem-delta?view=graph-rest-1.0)
- [Confluence Cloud content restrictions](https://developer.atlassian.com/cloud/confluence/rest/v1/api-group-content-restrictions/)
- [Confluence Cloud rate limiting](https://developer.atlassian.com/cloud/confluence/rate-limiting/)
- [Slack Events API delivery and retries](https://docs.slack.dev/apis/events-api/)
- [Slack Web API rate limits](https://docs.slack.dev/apis/web-api/rate-limits/)
- [Gmail synchronization and history expiry](https://developers.google.com/workspace/gmail/api/guides/sync)
- [Microsoft Graph message delta](https://learn.microsoft.com/en-us/graph/delta-query-messages)
- [Amazon S3 event notification types and delivery](https://docs.aws.amazon.com/AmazonS3/latest/userguide/notification-how-to-event-types-and-destinations.html)
- [Google Cloud Storage Pub/Sub notification guarantees](https://cloud.google.com/storage/docs/pubsub-notifications)
- [PostgreSQL logical decoding concepts](https://www.postgresql.org/docs/current/logicaldecoding-explanation.html)
- [Elasticsearch document- and field-level security](https://www.elastic.co/guide/en/elasticsearch/reference/current/document-level-security.html)
- [Qdrant multitenancy](https://qdrant.tech/documentation/tutorials/multiple-partitions/)
- [MCP 2026-07-28 specification overview](https://blog.modelcontextprotocol.io/posts/2026-07-28/)
- [MCP authorization](https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization)
