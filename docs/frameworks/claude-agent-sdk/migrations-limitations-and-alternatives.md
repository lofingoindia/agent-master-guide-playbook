# Migrations, Limitations, and Alternatives

Research date: **2026-08-31**  
Maturity: **decision guidance is stable; exact compatibility must be rechecked per release**

## Decide from requirements, not momentum

Migration is justified when the target surface removes a real constraint:

- a custom Messages loop is becoming a Claude Code-like workspace harness;
- `claude -p` automation needs typed streaming, hooks, interruption, or in-process custom tools;
- self-hosted Agent SDK operations no longer fit the desired ownership model and Managed Agents’ beta/retention contract is acceptable;
- Managed Agents needs local control or a stable non-beta contract, favoring the Agent SDK or a custom loop.

Do not migrate merely because one surface is newer.

## Decision matrix

| Requirement | Client SDK loop | Agent SDK | Claude Code | Managed Agents |
|---|---:|---:|---:|---:|
| Any officially supported client language | Strong | Python/TS only | Subprocess | Strong |
| Full loop control | Strong | Medium | Low | Low/medium |
| Built-in workspace tools | Build | Strong | Strong | Strong |
| Self-hosted process/data plane | Strong | Strong | Strong | Partial in self-hosted sandbox |
| Server-managed durable sessions | Build | Build with SessionStore | Local | Strong |
| Stable non-beta API preference | Strong | Pre-1.0 package | Product release cadence | Weak today |
| Minimal infrastructure | Strong for simple loops | Medium/high | Low for human use | Strong if contract fits |
| Strict custom authorization | Strong | Strong with adapter design | Limited product integration | Strong for custom tools; platform policies vary |

## Messages API/client SDK to Agent SDK

### What changes

The application gives up direct ownership of each model request and adopts a child-process harness that supplies:

- tool loop and built-ins;
- sessions and automatic compaction;
- Claude Code prompt/settings behavior;
- permissions and hooks;
- MCP, skills, plugins, and subagents.

### Migration sequence

1. Inventory the existing loop, tools, prompts, retries, storage, and authorization.
2. Keep business tools as narrow adapters; expose them through MCP.
3. Map the application event model to SDK messages.
4. choose a minimal custom prompt or explicit `claude_code` preset.
5. implement terminal-state, interruption, and process supervision.
6. add isolated workspaces only where needed.
7. test compaction, permissions, and tool effects.
8. canary against the existing loop using outcome/cost/latency evaluations.

Do not double-loop. The application should not parse an Agent SDK assistant message, execute the same tool independently, and feed it back as if it still owned the Messages loop.

## Claude Code/`claude -p` to Agent SDK

The migration can preserve much of the harness behavior, but defaults are not identical.

Check:

- SDK minimal system prompt versus CLI full prompt;
- explicit settings sources;
- message type handling instead of stdout text parsing;
- streaming input and interruption;
- hook callbacks versus filesystem hooks;
- session ID capture and storage;
- package-bundled CLI version;
- old `Task` versus current `Agent` tool naming.

Use the `claude_code` preset when parity is desired. Treat the migration as a protocol integration, not a command-line wrapper replacement.

## Old Claude Code SDK naming to Agent SDK

Anthropic renamed the former Claude Code SDK packages and API surface to Claude Agent SDK. Follow the official migration guide for package/import names. Keep the application abstraction named after its purpose, not the vendor package, so future upgrades do not leak naming changes through product code.

## Agent SDK to Managed Agents

This is an operational redesign.

| Agent SDK concept | Managed Agents concept |
|---|---|
| Options/system prompt | Versioned agent definition plus session overrides |
| `cwd` and sandbox image | Environment plus session sandbox |
| SDK message stream | Persisted session/thread event stream |
| Local transcript/SessionStore | Server-side session history |
| Hooks and permission callback | Permission policies, confirmations, custom-tool event handling |
| In-process MCP/custom tool | MCP connector or application-executed custom tool |
| Subagent | Managed Agent child thread |
| App scheduler/process supervisor | Managed session lifecycle, still wrapped by app quotas |
| Workspace artifacts | Session outputs/uploaded files/external store |

Migration steps:

1. approve beta, retention, compliance, and region implications;
2. design agent/environment versioning;
3. convert inputs and outputs to the event model;
4. move files into uploads/mounts and export outputs;
5. adapt custom tools with idempotent event handling;
6. map permission policies and human confirmations;
7. account for `rescheduling` and webhook duplication;
8. design sandbox-state expiry;
9. load-test platform rate and concurrency limits;
10. retain a rollback path until behavioral evaluation passes.

Self-hosted sandboxes do not preserve Agent SDK process semantics. They move tool execution to your worker while the Managed Agents control plane remains authoritative.

## Managed Agents to Agent SDK

Teams may move back for stronger data-plane control, non-beta stability requirements, custom process isolation, or portability. They must then build:

- durable job/session APIs;
- event persistence and reconnect;
- process and workspace lifecycle;
- approval durability;
- autoscaling and affinity;
- artifact retention;
- multi-agent scheduling;
- operational viewer/debugging.

Export session evidence and artifacts before deleting managed resources. There is no assumption that a Managed Agents event stream can be imported as an Agent SDK transcript.

## Important Agent SDK limitations

- Python and TypeScript only; other languages use the CLI subprocess or a custom loop.
- Pre-1.0 packages and a bundled runtime whose behavior changes with upgrades.
- One active session generally means one child process tree.
- No complete top-level wall-clock timeout.
- Session resume does not restore arbitrary workspace or business state.
- Automatic compaction is lossy.
- `canUseTool` approval is tied to a live process.
- SessionStore mirroring can drop a batch after final failure.
- File checkpointing covers selected file tools, not Bash or full filesystem state, and conflicts with SessionStore.
- No per-subagent total deadline; background stall detection is not a deadline.
- Permissions are not sandboxing.
- Terminal cost is estimated and can be incomplete after crashes.
- Python and TypeScript feature/event/default parity is not perfect.

## Important Managed Agents limitations

- Beta API and separate beta headers.
- Stateful data retained until deletion; no ZDR or HIPAA BAA eligibility today.
- Cloud sandbox state expires 30 days from creation regardless of activity.
- Environments are not versioned.
- Self-hosted mode still sends model/tool inputs and outputs through Anthropic.
- Interrupt stop reason is not uniquely distinguishable from normal end by stop reason alone.
- Budget checks can overshoot slightly due to in-flight work.
- Platform retries/rescheduling require idempotent application event submission.
- Cloud sandbox/network/tool quotas and platform rate limits are service constraints.

## Alternatives

### Small explicit tool loop

Use a client SDK plus a state machine when there are few typed tools and strict control matters. This is often the best support/operations architecture.

### Deterministic workflow plus model steps

Let a workflow engine own sequence, retries, approvals, and compensation; call Claude only for bounded reasoning. This avoids asking a general agent loop to be a transaction coordinator.

### Claude Code directly

For a person-in-the-loop terminal/IDE workflow or a simple CI command, the product may already supply enough interface and lifecycle.

### Other agent frameworks

Consider an alternative when it has a required language, durable workflow engine, provider portability, graph/state semantics, or ecosystem integration. Compare concrete runtime and failure contracts, not feature-count marketing.

### No agent

Use ordinary code when the decision can be expressed as rules, queries, parsers, or a fixed workflow. It will be cheaper, faster, and more predictable.

## Migration acceptance gates

- [ ] Current and target ownership boundaries are documented.
- [ ] Prompt, tool, event, session, and effect semantics are mapped.
- [ ] Retention/compliance and tenant isolation are approved.
- [ ] Golden and adversarial workflows pass repeated evaluations.
- [ ] Cost, latency, resource, and failure rates meet SLOs.
- [ ] Resume, cancellation, retry, and approval recovery are tested.
- [ ] Artifacts and audit history have a migration/retention plan.
- [ ] Rollback does not depend on importing incompatible transcripts.

## Refresh triggers

Revisit these decisions on:

- Agent SDK 1.0 or a stable hosted Managed Agents release;
- a new official Agent SDK language;
- a change in Managed Agents ZDR/BAA eligibility;
- SessionStore transactional/durability changes;
- durable hosted approval or new Agent SDK workflow primitives;
- changes to sandbox retention or self-hosted data flow;
- provider/model capability changes that affect the selected surface.

## Sources

- [Agent SDK migration guide](https://code.claude.com/docs/en/agent-sdk/migration-guide)
- [Agent SDK overview](https://code.claude.com/docs/en/agent-sdk/overview)
- [Managed Agents migration guide](https://platform.claude.com/docs/en/managed-agents/migration)
- [Managed Agents overview](https://platform.claude.com/docs/en/managed-agents/overview)
- [CLI, SDKs, and libraries](https://platform.claude.com/docs/en/cli-sdks-libraries/overview)
- [Hosting the Agent SDK](https://code.claude.com/docs/en/agent-sdk/hosting)
- [API and data retention](https://platform.claude.com/docs/en/manage-claude/api-and-data-retention)
