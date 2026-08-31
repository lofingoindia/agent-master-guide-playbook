# LangGraph Failure Modes, Migrations, and Versioning

**Research date:** 2026-08-31
**Status:** Research-backed lifecycle guide

## Pin the package family, not one package

At this snapshot, the latest checked Python core was `langgraph==1.2.11` (2026-08-11) and Python SDK was `langgraph-sdk==0.4.4` (2026-08-27). Checkpointer, PostgreSQL/SQLite saver, CLI, SDK, server image/API, LangChain integrations, and LangSmith deployment surfaces have separate versions.

Record a release manifest:

```text
application commit and image digest
graph schema/release
langgraph core
checkpoint core and backend
langchain-core and integrations
langgraph SDK/CLI/server image
Python/runtime
model, prompt, tool schema, policy
database schema revision
```

## Latest code applies to resumed threads

LangGraph's compatibility guidance differs from workflow engines that pin a run to the code version on which it started: resumed threads use the latest graph code. A saved checkpoint contains state and next-node information, not a frozen copy of all old application logic.

Consequences:

- old state must deserialize under new code;
- node names referenced by paused threads must still exist;
- reducers and routing must interpret old values;
- tool/effect behavior can change mid-thread unless versioned;
- an approval may resume under different policy or model code.

## Graph and state changes

Current guidance:

| Change | Completed threads | Interrupted/in-flight threads | Safe pattern |
|---|---|---|---|
| Add/remove/reroute edges | Generally supported | Supported if next node still exists | Test old checkpoints |
| Add node | Supported | Supported | No old state assumption |
| Remove/rename node | Completed history unaffected | Can break a thread about to enter it | Keep compatibility shim until drained |
| Add optional state field | Usually safe | Safe with default/tolerant read | `NotRequired`/optional first |
| Remove field | Old checkpoints still contain it | Readers may fail | Deprecate, stop reading, drain, then remove |
| Rename field | Old saved value not automatically moved | Data appears lost to new name | Add new field, dual read/write, migrate |
| Change field type | Compatibility risk | Compatibility risk | Version and transform explicitly |
| Change reducer | Replays/updates can differ | Merge semantics can change | Golden checkpoint/reducer tests |

Do not rely on “schema supports adding/removing keys” as permission to skip application-level semantic migration.

## Interrupt schema migrations

An interrupt can wait indefinitely. Version the proposal and resume envelope. Keep old parsers available for the retention window or run a controlled migration/cancellation process.

On resume:

- load recorded graph/policy/proposal version;
- reauthenticate and authorize now;
- validate the old payload;
- transform to the current internal schema;
- detect changed target/resource version;
- require renewed approval if meaning changed.

## Common failure modes

| Symptom | Likely boundary | First evidence |
|---|---|---|
| `GRAPH_RECURSION_LIMIT` | Route/termination/budget | state, next tasks, step counter |
| Concurrent update error | Missing/unsafe reducer | writers in same superstep |
| Old work repeats after resume | Replay/task/checkpoint boundary | checkpoint/task writes and operation IDs |
| Subgraph starts over | Namespace/nested recovery/version | `checkpoint_ns`, pinned version, nested fixture |
| Approval resumes wrong prompt | Interrupt order/schema changed | interrupt IDs/order and release manifest |
| UI shows complete but run continues | Stream/transport state machine | durable run status and reconnect log |
| Cross-tenant memory | Auth/store namespace | handler decision and backend version |
| Storage/latency grows | Unbounded state/history | per-checkpoint bytes and retention |
| Same thread appears stuck | queued prior run/lease/blocking code | queue age, lease, worker event loop |
| “Cancelled” effect commits | downstream noncooperation | effect ledger/external receipt |

## Upgrade process

1. Read core, checkpoint, SDK, server, integration release notes and advisories.
2. Lock a compatible dependency set and image digests.
3. Run unit, graph, serializer, checkpointer conformance, and security tests.
4. Load and resume the golden checkpoint corpus.
5. Replay nested subgraph and parallel-interrupt fixtures with invocation counters.
6. Run database migration/rollback and restore drills.
7. Canary new threads first.
8. Canary representative old completed and interrupted threads.
9. Monitor recovery, duplicate/unknown effects, auth denials, checkpoint errors, cost, and quality.
10. Roll forward or drain; do not blindly roll code back when the database/state schema already advanced.

## Deployment patterns for long-lived threads

### Compatibility window

Keep old node names and state readers until no supported thread needs them. Simplest for moderate lifetimes.

### Version-routed graph

Persist a graph schema version and route to compatibility nodes. More complex, justified for long waits and regulated workflows.

### Drain and cut over

Stop new work on the old release, finish/cancel/migrate old threads, then remove compatibility code. Strongest simplification when waiting periods are bounded.

### External durable workflow

For processes that must run for months/years with explicit code versioning and activity semantics, place LangGraph calls inside a workflow engine rather than stretching checkpoint compatibility indefinitely.

## Bounded issue evidence

Use issue reports as test leads, not general reliability statistics:

- #6626: parallel interrupt ID collision; regression-test multi-resume.
- #6792: nested task output reuse on interrupt/resume; regression-test subgraph resume.
- #8039: reported `sync` durability write-order race in 1.2.0/1.2.4; crash-test persistence ordering.
- #8458: reported subgraph time-travel regression through 1.2.9; test every nested checkpoint on the pinned version.

Check current status and fixed versions before making a release decision.

## Rollback is a data decision

An application binary rollback may not understand new checkpoint or database data. Before deploy, define:

- forward-only versus reversible DB migration;
- old reader tolerance for new optional fields;
- server API compatibility;
- whether new threads can be quarantined;
- how external effects under the new version are reconciled;
- the maximum safe rollback window.

## Lifecycle checklist

- [ ] Every checkpoint contains or can derive graph schema version.
- [ ] Every trace contains the complete release manifest.
- [ ] Oldest supported completed and interrupted fixtures resume.
- [ ] Renamed nodes/fields use add-then-remove.
- [ ] Approval and tool schemas are versioned independently.
- [ ] Upgrade and rollback include database, queue, state, and effects.
- [ ] Release notes and security advisories are reviewed for every package.
- [ ] A documented owner decides when old threads are drained, migrated, or cancelled.

## Sources

- [Backward compatibility](https://docs.langchain.com/oss/python/langgraph/backward-compatibility)
- [Graph migrations](https://docs.langchain.com/oss/python/langgraph/graph-api)
- [LangGraph releases](https://github.com/langchain-ai/langgraph/releases)
- [LangGraph v1 migration](https://docs.langchain.com/oss/python/migrate/langgraph-v1)
- [Common errors](https://docs.langchain.com/oss/python/common-errors)
- [Security advisories](https://github.com/langchain-ai/langgraph/security/advisories)

Next: [selection and alternatives](selection-and-alternatives.md).
