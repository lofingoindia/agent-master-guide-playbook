# Rust Agent Engineering Deep-Dive Research Packet

> **Research date:** 2026-08-31
> **Status:** Primary-source synthesis supporting the deep Rust language area
> **Scope:** Rust/Tokio production architecture, transport, schemas, effects, durability, ecosystem, observability, resource control, sandboxing, and supply chain
> **Runtime snapshot:** Rust 1.98.0; Tokio 1.53.1; reqwest 0.13.4; Tower 0.5.3; tower-http 0.7.0; SQLx 0.9.0; OpenTelemetry Rust signals Beta

## Research questions

This pass asked:

1. What does Rust's ownership model change for an agent runtime, and what does it not solve?
2. What are Tokio's exact task, cancellation, timeout, blocking, and shutdown semantics?
3. Which I/O operations are cancellation-safe inside repeated `select!` loops?
4. How should Rust services bound model streams, HTTP bodies, queues, tasks, processes, and memory?
5. How do Serde and Schemars behave at hostile model/tool/schema boundaries?
6. How should typed Rust errors drive retry, idempotency, panic, and ambiguity policy?
7. Which persistence and durable-execution options have real Rust support, at what maturity?
8. Which model/provider/agent/MCP surfaces are official vendor SDKs versus community projects?
9. What observability and concurrency-verification tools are available, and what are their limitations?
10. What supply-chain and build-time execution risks are current as of the research date?

## Method and evidence rules

- Prefer official Rust, Tokio, Cargo, Serde, reqwest/Tower, SQLx, OpenTelemetry, provider, MCP, durable-runtime, and Wasmtime documentation or repositories.
- Treat docs.rs for a crate's own rustdoc/source as primary package documentation.
- Record current release numbers only where the fetched primary source established them.
- Separate stable package release, vendor ownership, protocol tier, and product feature maturity.
- Treat project roadmaps and “users” lists as maintainer claims, not independent production proof.
- Use GitHub issues only to identify a limitation or roadmap item, not to estimate prevalence.
- Exclude benchmark leaderboards, generic Rust advocacy, SEO comparisons, and unsupported “zero-cost” claims.

## Finding 1: the stable baseline is current, but MSRV is a separate contract

Rust 1.98.0 was released on 2026-08-20 and is the current stable toolchain in this snapshot. Cargo's `rust-version` field records the minimum supported Rust version and affects diagnostics and some dependency selection. Cargo documentation notes that raising MSRV is generally treated as a minor incompatibility and that the Rust project only provides fixes for the latest Rust release.

Implications:

- pin the build toolchain separately from declaring a library/package MSRV;
- choose and publish an MSRV update policy;
- test the minimum and production toolchains;
- do not assume `Cargo.lock` alone pins the compiler, linker, native dependencies, or code generators;
- prioritize toolchain security releases even if the application source did not change.

Tokio 1.53.1 is the checked current release. Its 2026 changelog includes scheduler/time/sync regressions and fixes across closely spaced patch releases. This supports pinning plus representative cancellation/load tests rather than assuming a minor-line upgrade is operationally invisible.

## Finding 2: Rust ownership helps only when task ownership is architectural

Rust can encode:

- separate run/tenant/effect identifiers;
- transitions from raw to validated to authorized commands;
- permits whose `Drop` releases capacity;
- state enums with exhaustive transition handling;
- narrow capabilities rather than global services;
- borrowed local data and owned task-bound data.

It cannot by itself:

- join a detached Tokio task;
- stop a blocking call;
- fence two workers in a database;
- cap queue element bytes;
- make a provider/tool effect idempotent;
- persist run state through a crash;
- authorize a model-proposed action;
- turn in-process code into a sandbox.

Tokio's `JoinHandle` documentation says dropping the handle detaches the task. A task can therefore outlive request scope and retain everything moved into it. An owner must retain/join the handle, use a `JoinSet`, or register work with a supervisor/tracker.

`TaskTracker` provides a useful graceful-shutdown primitive: close it, then wait until closed and empty. Its documentation contrasts it with `JoinSet`: completed task memory can be freed immediately rather than retaining all completed outcomes, and dropping the tracker does not abort tasks. The application still needs a channel or result policy to observe errors.

## Finding 3: Tokio cancellation is future drop at a yield boundary

Tokio tasks can be aborted through a join/abort handle. Tokio documents:

- abort schedules shutdown the next time the task yields;
- an idle task can be shut down promptly;
- local variables are destroyed through their destructors;
- abort returns before cancellation completes;
- awaiting the join handle confirms the cancellation/termination result;
- a task may complete normally in a race with abort;
- runtime shutdown drops outstanding async tasks without guaranteeing completion.

`tokio::time::timeout` cancels the wrapped future on expiry. It polls the future before checking the timeout, so a future that does not yield can complete after the nominal deadline without a timeout error.

This creates the key audit question: what is safe if this future is dropped at each `.await`?

## Finding 4: Tokio publishes a concrete cancellation-safety matrix

The current `select!` documentation calls these cancellation-safe:

- bounded and unbounded mpsc `recv`;
- broadcast `recv`;
- watch `changed`;
- TCP/Unix listener `accept`;
- Unix signal `recv`;
- async `read`/`read_buf`;
- async `write`/`write_buf`;
- stream `next`.

It calls these not cancellation-safe:

- `read_exact`;
- `read_to_end`;
- `read_to_string`;
- `write_all`.

It also notes that cancelling fair queue operations loses queue position:

- async mutex/RwLock acquisition;
- semaphore acquisition;
- `Notify::notified`.

This matters for agent stream parsers, subprocess pipe drains, and retry loops. An owned child task can poll a multi-step operation to completion while its owner selects over the child's join handle and cancellation, but that child still needs a bounded stop/reconciliation design.

Tokio `select!` randomly selects the first branch to check by default. `biased;` removes that randomness but makes fairness the application’s responsibility; a constantly ready data branch can starve shutdown if ordered poorly.

## Finding 5: blocking work is a distinct, potentially unkillable pool

Tokio documents that:

- core async tasks switch only at yield points;
- `spawn_blocking` uses a separate pool with a very large upper limit by default;
- after the upper limit, submissions queue;
- CPU-heavy blocking work should be separately limited;
- started `spawn_blocking` closures cannot be aborted;
- runtime shutdown may wait indefinitely for them;
- `shutdown_timeout` stops waiting but does not cancel the closure; running work/threads continue until return;
- short terminating blocking work fits `spawn_blocking`, while persistent loops should use dedicated threads.

Therefore no untrusted, potentially hanging, or unbounded library call belongs in `spawn_blocking`. Use a killable process/container/VM boundary. For CPU work, use an explicit limit or a dedicated compute executor.

## Finding 6: queues and semaphores need byte and workload-class design

`tokio::sync::mpsc::channel` is bounded by element count and blocks sends after capacity, providing backpressure. `unbounded_channel` buffers arbitrarily until system memory; Tokio warns this can cause process abort.

Neither primitive limits the byte size of an element. An event may own a large `Vec`, JSON tree, artifact, or prompt. Production designs need a byte budget or artifact handles.

Tokio's semaphore is fair. Weighted `acquire_many` at the queue head can block smaller acquisitions even when smaller requests could proceed. One shared weighted pool can therefore create head-of-line blocking between small and large model/tool work. Separate workload classes or a dedicated admission scheduler may be simpler and fairer.

## Finding 7: HTTP protection is phase-specific

reqwest 0.13.4 documents that `Client` contains a connection pool, is already internally reference-counted, and should be created once and reused. Creating clients per request defeats connection reuse. `Client::new` can panic if TLS/resolver initialization fails; a startup `ClientBuilder` lets the service fail explicitly.

Tower 0.5.3 and tower-http 0.7.0 provide:

- per-service or shared global concurrency limits;
- request-body length limits, including early `Content-Length` rejection;
- request future timeouts;
- request/response body idle timeouts;
- absolute request/response body deadlines.

The tower-http docs clarify that body streaming happens outside the original service future. A request timeout therefore does not replace body idle and absolute deadline layers. The body limiter also notes that without `Content-Length`, enforcement occurs as the body is consumed; an unread excess does not automatically produce an application error.

Agent-specific conclusion: separately bound admission wait, connect/TLS, first byte, inter-frame idle, absolute duration, compressed bytes, decompressed bytes, partial frame, parsed events/tokens, and downstream backlog.

## Finding 8: Serde defaults are not a validation policy

Serde ignores unknown fields by default for self-describing formats. `#[serde(deny_unknown_fields)]` rejects them, but Serde explicitly says it is unsupported with `flatten` on the outer or flattened field.

Serde's untagged enums can have weak diagnostics and a costly matching approach. Explicit tags better support stable tool/result protocols.

`serde_json::Deserializer::disable_recursion_limit` is available only with an `unbounded_depth` feature and explicitly warns about stack overflow in parsing and later recursive operations including `Display`, `Debug`, and `Drop`. Keep depth protection and add byte/frame limits.

Serde can perform zero-copy borrowed deserialization, but streaming input generally requires owned data. For most run state, converting promptly into owned validated domain types avoids coupling lifetimes to a network buffer.

## Finding 9: generated JSON Schema is versioned evidence, not truth

Schemars 1.2.x currently defaults to JSON Schema 2020-12 and exposes settings for dialect and serialization/deserialization contract. Its documentation warns:

- the default schema version may change in a future release;
- generated schema structure may change between versions without a SemVer-breaking release;
- MSRV may rise in a minor release;
- schemas generated from example values are less precise, especially for enums.

Providers implement their own supported subsets. The safe pipeline is:

1. derive from a narrow wire type;
2. pin Schemars and settings;
3. normalize to the target provider/protocol subset;
4. keep a golden schema and digest;
5. conformance-test it against the exact provider;
6. locally deserialize and validate every returned value.

Rust type, JSON wire type, provider schema, and durable payload are four related but independent contracts.

## Finding 10: process APIs avoid shells, but cancellation and descendants remain application work

Rust's `Command::arg` passes an argument literally, not through a shell. The standard library warns that `cmd.exe` and batch files use non-standard parsing and malicious arguments can execute commands. Preserve direct executable/argument invocation, resolve allowlisted absolute executables, clear the environment, and never feed untrusted values to a shell.

Tokio process `kill_on_drop` defaults to false. Dropping a `Child` can leave the process running. On Unix, exited processes need reaping; Tokio makes only a best-effort attempt for dropped children and recommends fully awaiting them for strict cleanup.

Immediate-child termination is not a complete process-tree contract. Production isolation must use process groups/job objects/container supervision and test each target OS. stdout and stderr must be drained concurrently and bounded to avoid pipe deadlock or memory exhaustion.

## Finding 11: capability APIs and Wasmtime are useful defense-in-depth boundaries

`cap-std` represents external authority as values. A `Dir` supports paths relative to an opened directory, reducing reliance on ambient filesystem namespaces. Calls requiring `AmbientAuthority` make the authority transition visible. In-process unsafe/ambient APIs can still bypass this model; it is not isolation from hostile Rust code.

Wasmtime 48.0.1's `Store` supports:

- synchronous/async resource limiters;
- fuel;
- epoch deadlines;
- async yielding intervals.

Its docs say a store is intended to be short-lived and does not reclaim created instances until the store drops. Unbounded instance creation in one store retains memory. Fuel can provide deterministic instruction limits; epoch interruption is coarse. Neither bounds a guest stuck inside an unbounded host call. Host imports need their own capabilities, deadlines, and byte/resource limits.

## Finding 12: error types should carry operational policy

Rust's standard guidance distinguishes recoverable `Result` errors from unrecoverable panics. Production boundaries need variants such as:

- invalid input;
- denied;
- conflict/stale version;
- rate limited with retry-after;
- transient dependency;
- deadline;
- cancelled;
- ambiguous external outcome;
- invariant violation/bug.

The application can map those to fail, refresh/replan, backoff, reconcile, or isolate. Erasing them into strings before the retry boundary makes safe policy impossible.

Tokio returns task panics through `JoinError` when the handle is awaited. Detached/ignored handles hide them from the supervisor. `catch_unwind` is not a general reliability boundary; panic abort strategy and unsafe/FFI behavior limit it, and violated invariants may not be safe to continue.

## Finding 13: local persistence and durable execution solve different problems

SQLx 0.9.0 provides:

- a cloneable bounded pool with fair acquisition and acquire timeouts;
- explicit asynchronous pool close;
- transactions whose drop schedules rollback;
- compile-time checked queries with offline metadata;
- migration locking, enabled by default.

SQLx warns that dropping pools may not gracefully close client/server database connections because Rust lacks async drop; use `close().await` during shutdown. It also documents that cancellation during pool acquisition can drop a connection in some paths rather than return an uncertain connection to the pool.

A database state machine plus outbox/inbox is appropriate for bounded step graphs. The application owns leases, retries, timers, visibility, and repair. Durable engines add recorded replay, timers, signals, and workflow ownership, but require deterministic workflow code and versioning.

## Finding 14: Temporal and Restate have Rust SDKs, but neither should inherit another language's maturity label

The official Temporal Rust repository now calls `temporalio-sdk` a **Public Preview** Rust SDK built on its Rust Core. Core powers TypeScript, Python, .NET, and Ruby SDKs. The fact that Core is widely used does not make the public Rust workflow API GA.

The official Restate Rust SDK repository provides Rust service/workflow support and server compatibility guidance. It says the SDK is in active development and may break across releases. The generic Restate workflow docs cover durable operations, signals, timers, cancellation, parallelism, retry, and sagas, but some pages/examples list other languages and may not demonstrate Rust parity.

Adoption requirements:

- pin server and SDK together;
- validate the exact Rust feature;
- replay/upgrade test;
- keep model and external effects in recorded activities/steps;
- persist application-owned receipts;
- define worker/deployment versioning and repair.

## Finding 15: official provider support is uneven

OpenAI's current official SDK page lists official client libraries for JavaScript/TypeScript, Python, .NET, Java, Go, and Ruby. It places `async-openai` under a separate community-library section and says community projects are not verified for correctness or security. The same page lists the official Agents SDK only for TypeScript and Python.

Therefore:

- Rust has no OpenAI-maintained client or Agents SDK in the checked official list;
- direct REST remains supported from any HTTP environment;
- `async-openai` is a community option requiring application ownership;
- do not call a community crate the “official Rust SDK.”

AWS's official SDK for Rust includes Bedrock Runtime service APIs and examples. This is an official provider client, not a general agent loop.

## Finding 16: Rig is substantial but explicitly evolving

Rig's official repository describes a multi-provider Rust framework for completions, embeddings, tools, vector stores, streaming, and agent workflows, with many provider integrations. At the research snapshot:

- repository examples show current version lines around 0.4x;
- the repository contains an explicit warning that future updates will include breaking changes;
- release 0.41 split portable `rig-core` contracts and a `rig-agent` runtime behind a facade;
- the coding-agent roadmap still tracks lifecycle, cancellation, resume, nested execution, hosted tools, and code mode;
- the project publishes cassette/provider integration testing guidance.

Rig warrants serious evaluation and can reduce provider abstraction work. It remains a community framework with an evolving contract. Persist application-owned run state, pin exact feature/provider crates, and test streaming/tool/cancellation behavior for the exact release.

## Finding 17: MCP Rust package stability and tier maturity changed during the research window

Primary sources conflict in time:

1. The MCP 2026-07-28 GA announcement listed TypeScript, Python, Go, and C# as Tier 1 and described the Rust SDK's new-spec support as beta.
2. The official Rust SDK later published stable `rmcp` 3.0.0/3.0.1 releases.
3. Its current roadmap says all SEP-1730 Tier 1 requirements are met, reports complete dated conformance, and explains a tier-tool issue with workspace tag prefixes.

This does not justify silently rewriting history. The research-safe statement is:

- stable `rmcp` 3.0.x exists;
- the repository reports full required conformance and a completed Tier 1 self-assessment;
- the earlier public GA announcement called Rust beta;
- verify the current SDK tier registry/approval before a contractual Tier 1 claim.

The 2026-07-28 protocol also changed substantially: sessionless transport, standard routing headers, multi-round-trip requests, cache hints, Tasks as an extension, auth hardening, and deprecations. Protocol conformance does not prove tool authorization, sandboxing, tenant isolation, or resource control.

## Finding 18: OpenTelemetry Rust remains Beta across all three signals

The official OpenTelemetry Rust page currently labels traces, metrics, and logs **Beta**. Application teams should:

- pin the OpenTelemetry crate family;
- use `tracing` as the Rust-native instrumentation facade where appropriate;
- integration-test exporter shutdown/backpressure and semantic convention changes;
- keep the collector/export path out of the critical request path;
- avoid high-cardinality prompt/tool content.

`tracing::Span` documentation warns that holding an enter guard across `.await` produces incorrect traces because another task can run on the same thread while the span remains entered. Use `Instrument`, async `#[instrument]`, or `in_scope` for synchronous sections.

Tokio Console provides task/resource views, poll duration, busy time, wakes, and warnings for tasks that do not yield or self-wake. It complements, rather than replaces, distributed traces and OS/allocator profiles.

## Finding 19: concurrency testing needs complementary tools

Tokio paused time supports deterministic timer/retry/deadline tests on the current-thread runtime. It affects Tokio `Instant`, not standard time.

Loom permutes modeled concurrent executions under a C11-style model. Its own README documents limitations:

- some `SeqCst` behavior is modeled as weaker `AcqRel`, allowing false alarms;
- some load-buffering executions are not explored, so a passing model is not sound proof.

Use Loom for small critical synchronization structures, not a whole agent service. Use ordinary tests, property/state-machine tests, Miri/sanitizers for exercised memory behavior, cargo-fuzz for parsers/validators, and real load/fault/soak tests for pools, processes, and downstream interactions.

## Finding 20: the 2026 supply-chain evidence changes the threat model

Cargo build scripts are compiled and executed before the crate. Procedural macros also execute during compilation. Native toolchains and FFI extend the build/runtime trust boundary.

The Rust Security Response Team reported a 2026-08-20 supply-chain attack in which malicious crates used a build script to download a payload and a popular crate was republished with a malicious dependency. A lockfile freezes the chosen malicious version; it does not make it safe.

2026 Cargo advisories also affected alternate registries:

- CVE-2026-5223: symlink extraction in third-party registry crate archives, fixed in Rust 1.96;
- CVE-2026-5222: credential handling for sparse registry URL normalization, fixed in Rust 1.96;
- an earlier 2026 Cargo/tar advisory affected directory permissions during extraction, especially relevant to alternate registries.

Use multiple controls:

- pinned current toolchain and `Cargo.lock`;
- `--locked` release builds;
- vendoring/offline builds where appropriate;
- RustSec/`cargo audit`;
- license/source/duplicate policy such as cargo-deny;
- review attestations such as cargo-vet;
- SBOM/auditable artifacts and signed provenance;
- build isolation with no production secrets for untrusted changes;
- explicit review of build scripts, proc macros, unsafe/FFI, and native dependencies.

## Decision synthesis

Choose Rust for an agent component when:

- the owning team can debug and operate async Rust;
- the component is a gateway, tool host, MCP client/server, parser, local daemon, sandbox host, or bounded worker;
- explicit ownership, native deployment, memory safety, and low runtime overhead materially help;
- required provider/framework features exist in an acceptable official/community layer;
- the team accepts compile-time and ecosystem trade-offs.

Do not choose Rust merely because:

- “no GC” is assumed to mean bounded memory;
- static types are assumed to validate model data;
- Tokio tasks are assumed to be structured;
- Wasm is assumed to be safe without host limits;
- a stable crate is assumed to mean a GA product surface;
- a smaller binary is assumed to dominate prompt/context memory;
- a framework logo is assumed to provide provider parity.

A mixed architecture is justified when Rust provides a meaningful security, performance, protocol, deployment, or native-integration boundary. It is not justified when it only duplicates a Python/TypeScript agent controller behind a new service hop.

## Contradictions and caveats requiring future resolution

| Topic | Evidence tension | Current documentation rule |
|---|---|---|
| MCP Rust maturity | 2026-07-28 GA announcement called it beta; later stable 3.0.x roadmap says Tier 1 requirements met | State stable release and reported conformance; verify public approved tier |
| Temporal Rust | Earlier ecosystem summaries often said no public Rust SDK; official repository now says Public Preview | Use Public Preview, not “none” and not GA |
| Restate Rust parity | SDK exists and evolves; generic advanced docs do not always list Rust examples | Claim only exact verified Rust features; pin server/SDK |
| Rig maturity | Broad feature/adoption claims; repository warns breaking evolution and tracks major coding-agent gaps | Community evolving framework; test exact release |
| OpenTelemetry | OTel signal specs may be stable; Rust implementation signals are Beta | Use Rust implementation maturity |
| Stable Rust crates | Stable SemVer release can sit above preview/beta protocol/product behavior | Report package and product maturity separately |

## Guides supported

- [Rust Agent Engineering index](../../languages/rust/README.md)
- [Architecture and ownership](../../languages/rust/architecture-and-ownership.md)
- [Tokio runtime, cancellation, and shutdown](../../languages/rust/tokio-runtime-cancellation-and-shutdown.md)
- [HTTP, streaming, and backpressure](../../languages/rust/http-streaming-and-backpressure.md)
- [Tools, processes, and sandboxing](../../languages/rust/tools-processes-and-sandboxing.md)
- [Schemas, Serde, and structured output](../../languages/rust/schemas-serde-and-structured-output.md)
- [Errors, retries, idempotency, and panics](../../languages/rust/errors-retries-idempotency-and-panics.md)
- [State, persistence, and durable workers](../../languages/rust/state-persistence-and-durable-workers.md)
- [Providers, frameworks, and MCP](../../languages/rust/providers-frameworks-and-mcp.md)
- [Observability, profiling, and testing](../../languages/rust/observability-profiling-and-testing.md)
- [Memory, resources, and admission control](../../languages/rust/memory-resources-and-admission-control.md)
- [Supply chain, build, and deployment](../../languages/rust/supply-chain-build-and-deployment.md)
- [Failure modes and production checklist](../../languages/rust/failure-modes-and-production-checklist.md)

## Refresh triggers

- New Rust stable or Cargo security advisory.
- Tokio task/cancellation/process/runtime major behavior or release line changes.
- reqwest, Tower, Axum, Serde, Schemars, SQLx, Wasmtime, or tracing major release.
- OpenTelemetry Rust signal maturity changes.
- Official OpenAI or other vendor Rust client/Agents SDK release.
- Rig stable compatibility line or major runtime redesign.
- MCP protocol revision or Rust SDK approved-tier change.
- Temporal Rust exits public preview or changes support status.
- Restate Rust stabilizes/breaks its SDK compatibility model.
- crates.io, RustSec, build-script, proc-macro, registry, or native dependency incident.

## Selected primary sources

### Rust, Cargo, and supply chain

- [Rust 1.98.0 announcement](https://blog.rust-lang.org/releases/latest/)
- [Cargo `rust-version`](https://doc.rust-lang.org/stable/cargo/reference/rust-version.html)
- [Cargo dependency resolution](https://doc.rust-lang.org/cargo/reference/resolver.html)
- [Cargo features](https://doc.rust-lang.org/cargo/reference/features.html)
- [Cargo build scripts](https://doc.rust-lang.org/cargo/reference/build-scripts.html)
- [Cargo vendor](https://doc.rust-lang.org/cargo/commands/cargo-vendor.html)
- [Cargo profiles](https://doc.rust-lang.org/cargo/reference/profiles.html)
- [RustSec](https://rustsec.org/)
- [cargo-vet](https://github.com/mozilla/cargo-vet)
- [2026 arrayref supply-chain incident](https://blog.rust-lang.org/2026/08/20/supply-chain-attack-on-arrayref/)
- [Cargo CVE-2026-5223](https://blog.rust-lang.org/2026/05/25/cve-2026-5223/)
- [Cargo CVE-2026-5222](https://blog.rust-lang.org/2026/05/25/cve-2026-5222/)

### Tokio and transport

- [Tokio 1.53.1 changelog](https://docs.rs/crate/tokio/latest/source/CHANGELOG.md)
- [Tokio tasks and cancellation](https://docs.rs/tokio/latest/tokio/task/)
- [Tokio `select!`](https://docs.rs/tokio/latest/tokio/macro.select.html)
- [Tokio timeout](https://docs.rs/tokio/latest/tokio/time/fn.timeout.html)
- [Tokio graceful shutdown](https://tokio.rs/tokio/topics/shutdown)
- [Tokio `spawn_blocking`](https://docs.rs/tokio/latest/tokio/task/fn.spawn_blocking.html)
- [Tokio runtime shutdown](https://docs.rs/tokio/latest/tokio/runtime/struct.Runtime.html)
- [Tokio bounded mpsc](https://docs.rs/tokio/latest/tokio/sync/mpsc/fn.channel.html)
- [Tokio unbounded mpsc](https://docs.rs/tokio/latest/tokio/sync/mpsc/fn.unbounded_channel.html)
- [Tokio semaphore](https://docs.rs/tokio/latest/tokio/sync/struct.Semaphore.html)
- [Tokio process `Command`](https://docs.rs/tokio/latest/tokio/process/struct.Command.html)
- [reqwest client](https://docs.rs/reqwest/latest/reqwest/struct.Client.html)
- [tower concurrency limit](https://docs.rs/tower/latest/tower/limit/concurrency/)
- [tower-http body limits](https://docs.rs/tower-http/latest/tower_http/limit/)
- [tower-http timeouts](https://docs.rs/tower-http/latest/tower_http/timeout/)

### Schemas, state, isolation, and verification

- [Serde container attributes](https://serde.rs/container-attrs.html)
- [Serde lifetimes](https://serde.rs/lifetimes.html)
- [serde_json deserializer](https://docs.rs/serde_json/latest/serde_json/struct.Deserializer.html)
- [Schemars](https://docs.rs/schemars/latest/schemars/)
- [SQLx pool](https://docs.rs/sqlx/latest/sqlx/struct.Pool.html)
- [SQLx transaction](https://docs.rs/sqlx/latest/sqlx/struct.Transaction.html)
- [SQLx checked queries](https://docs.rs/sqlx/latest/sqlx/macro.query.html)
- [Rust process `Command`](https://doc.rust-lang.org/std/process/struct.Command.html)
- [cap-std](https://docs.rs/cap-std/latest/cap_std/)
- [Wasmtime `Store`](https://docs.rs/wasmtime/latest/wasmtime/struct.Store.html)
- [tracing async span guidance](https://docs.rs/tracing/latest/tracing/struct.Span.html)
- [Tokio paused time](https://docs.rs/tokio/latest/tokio/time/fn.pause.html)
- [Loom](https://github.com/tokio-rs/loom)
- [Rust Fuzz Book](https://rust-fuzz.github.io/book/cargo-fuzz/tutorial.html)

### Provider, framework, protocol, telemetry, and durability

- [Official OpenAI library list](https://developers.openai.com/api/docs/libraries)
- [AWS SDK for Rust](https://docs.aws.amazon.com/sdk-for-rust/latest/dg/welcome.html)
- [Rig](https://github.com/0xplaygrounds/rig)
- [Rig 0.41 release discussion](https://github.com/0xPlaygrounds/rig/discussions/2225)
- [Rig coding-agent roadmap](https://github.com/0xPlaygrounds/rig/issues/2118)
- [MCP 2026-07-28 release](https://blog.modelcontextprotocol.io/posts/2026-07-28/)
- [MCP SDK tiers](https://modelcontextprotocol.io/community/sdk-tiers)
- [MCP Rust SDK releases](https://github.com/modelcontextprotocol/rust-sdk/releases)
- [MCP Rust roadmap](https://github.com/modelcontextprotocol/rust-sdk/blob/main/ROADMAP.md)
- [OpenTelemetry Rust](https://opentelemetry.io/docs/languages/rust/)
- [Temporal Rust SDK](https://github.com/temporalio/sdk-rust)
- [Restate Rust SDK](https://github.com/restatedev/sdk-rust)
- [Restate workflows](https://docs.restate.dev/tour/workflows)
