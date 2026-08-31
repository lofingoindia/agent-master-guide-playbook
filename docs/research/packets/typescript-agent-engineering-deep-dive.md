# Research Packet: TypeScript Agent Engineering Deep Dive

> **Research date:** 2026-08-31  
> **Status:** Pass-2 usefulness and technical-depth refinement complete for the language-specific guide set  
> **Output:** [`docs/languages/typescript/`](../../languages/typescript/README.md)  
> **Boundary:** Node.js event-loop, stream, worker, cancellation, process, and lifecycle engineering remains in the existing [TypeScript and Node.js runtime guide](../../languages/typescript-node-agent-runtimes.md).

## Research objective

Determine the TypeScript-specific production practices required to build agent systems whose compiler configuration, erased-type boundaries, tool and event contracts, provider adapters, packages, artifacts, and upgrades remain safe under real runtime and dependency change.

The work intentionally does not duplicate generic agent architecture or Node runtime operations. It focuses on the gap between “the program type-checks” and “the deployed, versioned, provider-integrated system enforces the intended contract.”

## Method

Research used primary sources as the foundation:

1. current TypeScript announcements, handbook, TSConfig reference, release notes, repository/wiki;
2. official runtime/package documentation for Node.js, Deno, Bun, and Cloudflare Workers;
3. official schema/validator documentation for Zod, Ajv, JSON Schema, Standard Schema, and TypeBox;
4. official provider and protocol SDK/documentation for OpenAI, Anthropic, Google Gen AI, and MCP;
5. primary build, package, API-surface, testing, linting, and npm supply-chain documentation.

Claims were cross-checked across compiler, host, and tool documentation where responsibilities overlap. Vendor performance claims are labeled as vendor results rather than independently benchmarked facts. Community commentary was not used as authority where a primary source existed.

The second pass rechecked the time-sensitive baseline and then stress-tested whether a reader could turn each recommendation into release evidence. It added maintainer issue evidence only where it demonstrates a concrete failure class, and labels those issue states as refreshable snapshots rather than universal product claims.

## Current baseline established

| Area | Verified baseline on 2026-08-31 | Engineering consequence |
|---|---|---|
| TypeScript stable | 7.0.2 native compiler line | New default/removed-option migration must be handled now, not as hypothetical future work |
| Compatibility compiler | TypeScript 6.0.3 via the current `@typescript/typescript6@6.0.2` wrapper | Tools requiring the programmatic compiler API still need a TS 6 path; record wrapper and resolved compiler separately |
| TS 7 programmatic API | Not shipped in the stable 7.0 release | Do not migrate compiler-API tools by changing only the CLI dependency |
| TS 7 implementation | Native Go compiler/language server | Official 8×–12× build claims are promising but workload-specific verification remains required |
| TS 7 defaults | Stricter and more explicit baseline, including strict mode and changed source/type defaults | Shared configs need an intentional compatibility pass rather than inheriting defaults accidentally |
| TS 7 removals | Legacy/deprecated compiler and module-resolution choices become hard errors/removals | Become deprecation-clean on TS 6 before switching the release gate |
| MCP TypeScript SDK | v2 stable against 2026-07-28 protocol documentation | A concrete example of package, schema-library, handler, and public-type churn |
| Zod JSON Schema | Zod 4 provides native conversion with documented limitations | Generated JSON Schema is a projection, not proof of semantic identity |
| Node native TS | Type stripping is documented, with a constrained erasable-syntax configuration | It is a target-specific execution profile, not a universal TypeScript build replacement |

The exact release numbers above are time-sensitive. The guide set includes refresh triggers rather than presenting them as permanent architecture.

### Pass-2 verification notes

- The TypeScript 7.0.2 / 6.0.3 dual-compiler baseline remains correct. The official compatibility wrapper is separately versioned at `@typescript/typescript6@6.0.2`; the guides no longer conflate those versions. The compatibility guidance is narrower than a major-version slogan: TS 6 should use stable type ordering and no ignored deprecations before parity is expected.
- MCP TypeScript SDK v2 is the stable release line for the 2026-07-28 revision. The usefulness pass expanded adoption evidence beyond compilation to package/runtime, schema/list-call behavior, handler context, transport/error mapping, and protocol conformance.
- npm install-script controls are version-sensitive. The guides now require a pinned package-manager version and an exercised policy fixture rather than assuming an allowlist field is enforced identically across npm lines.
- OpenTelemetry GenAI conventions have moved to a dedicated repository and continue to evolve. Telemetry is therefore treated as another versioned projection, not an application domain contract or replay authority.
- A current issue in the official Google Gen AI SDK repository shows CommonJS JavaScript executing while TypeScript declarations fail for the same consumer mode. It supports separate value/type packed-consumer gates; it is not used to claim that all SDK versions or module modes are broken.

## Core findings

### 1. TypeScript is intentionally not a runtime proof system

The handbook describes structural compatibility and intentional unsoundness. Type annotations are erased; assertions, generic parameters, and interfaces do not validate network, model, queue, checkpoint, environment, or SDK data.

**Decision:** Every foreign or durable value enters as `unknown`, crosses an executable validator, and only then becomes a trusted domain type. Both tool input and tool output are validated. Serialization has a separate JSON-safe contract.

**Rejected approach:** Casting `JSON.parse`, fetch responses, or model tool arguments to an interface. It shortens code by removing the only enforcement point.

### 2. Module resolution must model the actual host

TypeScript module checking is host-sensitive. `nodenext`, `bundler`, Deno conventions, Bun behavior, and worker/bundler environments do not resolve or expose identical APIs. `paths` informs TypeScript but does not make a runtime or published consumer implement the mapping.

**Decision:** Use a small shared strict base plus per-target module, `lib`, `types`, and entry-point configs. Validate the built artifact on the actual minimum host. Treat export maps as public package boundaries.

**Rejected approach:** One config containing Node and DOM/worker globals. It describes a fictional runtime and allows unavailable APIs to type-check.

### 3. Strictness is a set of domain questions

`strict` is the baseline, but important production gaps remain. `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`, `useUnknownInCatchVariables`, `noImplicitOverride`, and `noPropertyAccessFromIndexSignature` expose different categories of ambiguity.

**Decision:** Enable them deliberately, fix models through narrowing and unions, and keep target/environment types explicit. In particular, decide whether absent, `undefined`, and `null` have different wire meanings.

**Rejected approach:** Enabling flags and adding assertions/non-null operators until compilation returns to green.

### 4. Agent state benefits from discriminated unions and staged trust types

Optional-property bags admit contradictory states. A call should not be executable merely because it has a `name` and `arguments` field.

**Decision:** Use small app-owned unions for run states, commands, events, failures, and provider-normalized stream events. Represent tool calls as proposed → validated → authorized stages, with construction only through validation/policy functions.

**Caveat:** Exhaustiveness is appropriate for app-owned closed unions. Foreign enums and independently evolving wire input need explicit unknown handling.

### 5. Runtime schema authority must be explicit

Schema-first, validator-first, and type-first workflows each have trade-offs. Zod documents constructs that cannot be represented in JSON Schema, including runtime-only types and transformations. Input and output types can differ. JSON Schema providers support subsets and dialect details.

**Decision:** Choose one canonical executable boundary representation and document it. Generate provider schemas as versioned, tested projections with explicit loss reports. Always revalidate locally.

**Rejected approach:** Silent conversion fallback to `{}`/any or assuming provider acceptance equals complete validation.

### 6. Provider SDK types belong at adapters

SDKs reflect vendor release cadence and protocol representation. OpenAI's official SDK notes that static type changes may appear in minor versions. MCP v1→v2 demonstrates package, schema, import, and API change across a protocol generation.

**Decision:** Provider adapters accept app-owned requests and emit app-owned event/failure unions. Preserve original unknown values for safe observability, assemble bounded stream fragments, and expose a tested capability descriptor. Compile/fixture-test SDK upgrades.

**Rejected approach:** Re-exporting vendor request/response types or persisting SDK event objects.

### 7. Provider schema subsets require deliberate projection

Structured output and tool definitions are not one universally portable JSON Schema surface. Provider/model capability can change independently of the client compiler.

**Decision:** Project by provider, feature mode, and tested capability; fail closed when a required canonical constraint cannot be represented. Cache against canonical schema digest and projector version. Keep canonical local enforcement.

**Rejected approach:** A provider-neutral schema generator that silently strips every unsupported keyword.

### 8. Durable contracts need their own version lifecycle

Package SemVer, interface names, and deployment timestamps do not identify stored JSON semantics. Events, checkpoints, and effects also have different invariants.

**Decision:** Use stable schema names and explicit integer versions, language-neutral wire values, deterministic upcasters, and read-old/write-new rollout. Carry event ID, aggregate sequence, correlation, causation, and effect request identity where the runtime relies on them.

**Rejected approach:** Inferring version from the TypeScript package or rewriting all old data before tolerant readers deploy.

### 9. TypeScript cannot type promise rejection

`Promise<T>` types fulfillment only; JavaScript can reject with any value. TypeScript intentionally has no checked-exception model. Callback rejection reasons also require lint support to avoid `any`.

**Decision:** Put expected failures in a `Result`-style discriminated union, catch unexpected failures as `unknown`, and serialize only an allowlisted versioned failure. Use typed lint to catch floating promises and unsafe rejection handling.

**Rejected approach:** Catch annotations/casts that claim an SDK error subclass and raw `Error` objects in durable events.

### 10. Async types expose ownership but do not enforce lifecycle

An `AbortSignal` property, `AsyncIterable`, or `AsyncDisposable` type does not prove that cancellation is observed, early iteration closes a connection, or shutdown waits for work.

**Decision:** Make signal/deadline context required on blocking APIs, document stream terminal and early-exit semantics, supervise every started promise, and test cleanup. Keep the runtime mechanics in the Node/runtime guides.

**Rejected approach:** Optional cancellation on critical operations and `void` as a general fire-and-forget policy.

### 11. Public declarations and JavaScript are separate compatibility surfaces

Package export conditions, module modes, and declaration paths must agree. Workspace source links and `paths` can hide broken tarballs. `isolatedDeclarations` improves declaration emit constraints but does not define the public API.

**Decision:** Explicit exports, named app-owned public types, API/declaration reports, packed consumer fixtures, and resolution checks. Use ESM-only by default unless real CommonJS consumers justify the additional surface.

**Rejected approach:** Recursive barrels, public deep imports, and `skipLibCheck` in the publisher release gate.

### 12. Workspace, project-reference, and runtime graphs differ

Project references build declaration boundaries; workspaces install/link packages; runtime exports determine consumer loading. They should align but solve different problems. TypeScript 5.6 build mode continues past errors by default unless configured to stop; TS 7 adds parallel build/check capabilities.

**Decision:** Adopt references only for stable, useful build boundaries; keep graphs acyclic and ban cross-package source imports. Fail closed for releases and resource-bound parallel jobs in CI.

### 13. A transformer is not a type checker

esbuild documents that it removes TypeScript syntax without type checking and does not emit declarations or decorator metadata. TypeScript 7's native CLI and API gap can require more than one compiler/tool role.

**Decision:** Separate type gate, transformation, declaration generation, schema generation, and artifact tests. Record every exact tool/version in a build manifest.

**Rejected approach:** Treating a successful fast bundle or Node native type stripping as type validation.

### 14. Cross-runtime support is artifact-specific

Node, Node native stripping, Bun, Deno, and Workers offer overlapping but non-identical TypeScript, module, global, and Node-compatibility behavior. Cloudflare documents that some compatibility imports can exist as throwing stubs.

**Decision:** Use target entry points and small host-capability ports; type-check and execute the artifact on every supported target. Test operations, not just imports.

**Rejected approach:** Runtime-name checks spread through core code or assuming Node compatibility means semantic identity.

### 15. Source maps and generated schemas are production artifacts

Source maps can expose source text/paths; mismatched maps harm incident diagnosis. Generated schemas and declarations can drift semantically or refer to absent package files.

**Decision:** Coordinate JavaScript, declarations, schemas, maps, licenses, and manifest under one source/artifact digest. Inspect file contents, test deliberate stack mapping, and validate schema semantics.

### 16. Build dependencies are production supply-chain inputs

Compilers, bundlers, validators, and schema generators determine shipped bytes even as dev dependencies. Lifecycle scripts execute dependency code. npm provenance/signatures improve origin/integrity evidence but do not prove safety.

**Decision:** Frozen clean install, explicit lifecycle-script policy, reviewed lockfile updates, generated-artifact diff plus semantic tests, packed file inventory, provenance where available, and immutable artifact promotion.

**Rejected approach:** Assuming dev dependencies are non-production or automatically merging SDK/toolchain updates after ordinary unit tests.

### 17. Production readiness is an evidence ladder, not a strictness preset

Compiler truth, runtime validation, domain-state modeling, provider isolation, and release/artifact proof are separate adoption levels. Combining them into one rewrite makes failures hard to attribute and rollback unsafe.

**Decision:** Advance through inventory → host-accurate compiler → validated boundaries → state/event/effect contracts → provider isolation → independently tested artifact. Each level has a concrete exit test.

### 18. Projection loss must be executable and reviewable

A golden JSON Schema diff can change cosmetically while behavior stays stable, or stay small while a constraint disappears. Provider acceptance also does not cover refusals, incomplete responses, or later capability changes.

**Decision:** Projection functions return schema, canonical digest, projector version, and exact loss records. Fixtures compare canonical and projected accept/reject behavior; a widened loss fails closed. Live registration is a separate bounded smoke, and successful output is revalidated locally.

### 19. Observability libraries are third-party adapters

Auto-instrumentation can double spans, change attribute conventions, record sensitive model content, or make exporter behavior affect availability. High-cardinality provider IDs are useful trace evidence but dangerous metric labels.

**Decision:** Project a small app-owned observation into a pinned telemetry convention. Test cardinality, redaction, correlation, exporter-disabled/failure behavior, exact instrumentation versions, and convention migrations. Domain events—not spans—remain replay authority.

### 20. Package type and runtime surfaces require separate consumers

A real provider SDK issue demonstrates that a CommonJS value entry can run while the shared declaration file is interpreted as ESM and rejected by TypeScript. Workspace tests and runtime-only smokes miss this class.

**Decision:** Install the tarball in empty fixtures, import one value and one public type per entry point, check supported resolvers/compilers, run each advertised module mode, and assert private/unsupported paths fail deliberately.

## TypeScript 7 transition decision record

### Context

The stable TypeScript 7 native CLI and language server are available, but the programmatic compiler API is not. Existing tools in the ecosystem may import `typescript` for program creation, AST traversal, transforms, declaration work, or plugins.

### Decision

- Make the TypeScript 6 configuration deprecation-clean first.
- Pin the native TS 7 CLI for the intended application check/build lane.
- Pin `@typescript/typescript6` or an explicit alias for compiler-API consumers.
- Invoke each role explicitly; do not depend on incidental dependency resolution.
- Compare diagnostic codes/locations, type tests, declarations, artifacts, and tool integrations.
- Retire TS 6 only after the required native API exists and each consumer has output-equivalence evidence.

### Consequences

The temporary toolchain is more explicit and slightly more complex, but it avoids pretending that CLI compatibility implies API compatibility. The build manifest must identify both compilers.

### Refresh trigger

Re-evaluate immediately when Microsoft publishes a stable TypeScript 7 programmatic API or changes the official TS 6 compatibility guidance.

## Schema strategy decision record

### Context

Agent contracts must serve local validation, provider tool/structured output, durable JSON, other languages, and developer inference. No single TypeScript type construct maps perfectly across those surfaces.

### Decision

- Canonical authority is an executable, code-reviewed boundary schema or a code-reviewed JSON Schema compiled into a validator.
- The application derives input/output types when supported but never treats them as validation.
- Provider schemas are explicit projections with loss reports.
- Durable contracts carry `$id`/schema name and application version.
- Runtime accept/reject parity tests supplement schema text diffs.
- Untrusted schemas are not compiled/executed in the privileged agent process without separate complexity limits and isolation.

### Consequences

Some domain types remain separate from wire types. Dates, big integers, maps/sets, transformations, branded values, and recursive objects require explicit encoding decisions.

## Guide-set map

| Guide | Research applied |
|---|---|
| [Compiler Configuration and Module Resolution](../../languages/typescript/compiler-configuration-and-module-resolution.md) | TS 7/6 transition, strictness, module/host alignment, decorators |
| [Runtime Validation and JSON Schema](../../languages/typescript/runtime-validation-and-json-schema.md) | Erasure, schema authority, Zod/Ajv/JSON Schema/provider projection |
| [Domain Modeling and State Machines](../../languages/typescript/domain-modeling-and-state-machines.md) | Unions, narrowing, trust stages, brands, expected failures |
| [Tools, Events, and Versioned Contracts](../../languages/typescript/tools-events-and-versioned-contracts.md) | Tool registry, schemas, event envelopes, upcasters, replay invariants |
| [Provider Adapters and SDK Churn](../../languages/typescript/provider-adapters-and-sdk-churn.md) | App-owned ports, capabilities, stream normalization, SDK upgrade gates |
| [Async APIs, Errors, and Resource Lifecycles](../../languages/typescript/async-apis-errors-and-resource-lifecycles.md) | Unknown rejection, result unions, cancellation/lifecycle type contracts |
| [Packages, Public APIs, and Monorepos](../../languages/typescript/packages-public-apis-and-monorepos.md) | Exports/declarations, project references, consumer testing |
| [Build Artifacts, Source Maps, and Cross-Runtime Delivery](../../languages/typescript/build-artifacts-source-maps-and-cross-runtime.md) | Build roles, maps, Node/Deno/Bun/Workers, artifact manifest |
| [Testing Types and Compatibility](../../languages/typescript/testing-types-and-compatibility.md) | Type/schema/adapter/package/runtime evidence matrix |
| [Supply Chain and Release Engineering](../../languages/typescript/supply-chain-and-release-engineering.md) | Frozen installs, scripts, provenance, artifact promotion |
| [Migration Playbook and Anti-Patterns](../../languages/typescript/migration-playbook-and-anti-patterns.md) | Phased modernization, TS 7 migration, decorators, rollback |

## Primary source register

### TypeScript language, compiler, and packages

- [Announcing TypeScript 7.0](https://devblogs.microsoft.com/typescript/announcing-typescript-7-0/)
- [Announcing TypeScript 6.0](https://devblogs.microsoft.com/typescript/announcing-typescript-6-0/)
- [TypeScript repository and releases](https://github.com/microsoft/TypeScript)
- [TypeScript TSConfig reference](https://www.typescriptlang.org/tsconfig/)
- [Modules theory](https://www.typescriptlang.org/docs/handbook/modules/theory.html)
- [Modules reference](https://www.typescriptlang.org/docs/handbook/modules/reference.html)
- [Choosing compiler options](https://www.typescriptlang.org/docs/handbook/modules/guides/choosing-compiler-options.html)
- [Project references](https://www.typescriptlang.org/docs/handbook/project-references)
- [Narrowing and discriminated unions](https://www.typescriptlang.org/docs/handbook/2/narrowing.html)
- [Type compatibility and soundness trade-offs](https://www.typescriptlang.org/docs/handbook/type-compatibility.html)
- [`exactOptionalPropertyTypes`](https://www.typescriptlang.org/tsconfig/exactOptionalPropertyTypes.html)
- [`noUncheckedIndexedAccess`](https://www.typescriptlang.org/tsconfig/noUncheckedIndexedAccess.html)
- [`useUnknownInCatchVariables`](https://www.typescriptlang.org/tsconfig/useUnknownInCatchVariables.html)
- [`noImplicitOverride`](https://www.typescriptlang.org/tsconfig/noImplicitOverride.html)
- [`noPropertyAccessFromIndexSignature`](https://www.typescriptlang.org/tsconfig/noPropertyAccessFromIndexSignature.html)
- [TypeScript 4.9 `satisfies`](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-9.html)
- [TypeScript 3.4 const assertions](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-4.html)
- [TypeScript 3.9 `@ts-expect-error`](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-9.html)
- [TypeScript 5.0 decorators](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-0.html)
- [Legacy decorators](https://www.typescriptlang.org/docs/handbook/decorators)
- [TypeScript 5.2 explicit resource management](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-2.html)
- [TypeScript 5.5 `isolatedDeclarations`](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-5.html)
- [TypeScript 5.6 build behavior](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-6.html)
- [Declaration-file publishing](https://www.typescriptlang.org/docs/handbook/declaration-files/publishing.html)
- [TypeScript FAQ, including checked exceptions](https://github.com/microsoft/TypeScript/wiki/faq)

### Runtime validation and protocols

- [Zod basics](https://zod.dev/basics)
- [Zod JSON Schema conversion](https://zod.dev/json-schema)
- [Ajv strict mode](https://ajv.js.org/strict-mode.html)
- [Ajv security](https://ajv.js.org/security.html)
- [Ajv standalone validation code](https://ajv.js.org/standalone.html)
- [JSON Schema Draft 2020-12 validation specification](https://json-schema.org/draft/2020-12/json-schema-validation)
- [Standard Schema](https://standardschema.dev/)
- [TypeBox](https://github.com/sinclairzx81/typebox)
- [MCP TypeScript SDK v2](https://ts.sdk.modelcontextprotocol.io/v2/)
- [MCP v2 schema libraries](https://ts.sdk.modelcontextprotocol.io/v2/advanced/schema-libraries)
- [MCP TypeScript v2 migration](https://ts.sdk.modelcontextprotocol.io/v2/migration/upgrade-to-v2)
- [MCP TypeScript SDK roadmap and stable-line policy](https://github.com/modelcontextprotocol/typescript-sdk/blob/main/ROADMAP.md)
- [OpenAI Structured Outputs](https://platform.openai.com/docs/guides/structured-outputs)
- [OpenAI Responses API TypeScript reference](https://developers.openai.com/api/reference/typescript/resources/responses/methods/create)
- [Gemini structured output](https://ai.google.dev/gemini-api/docs/structured-output)

### Provider SDKs

- [OpenAI Node SDK](https://github.com/openai/openai-node)
- [Anthropic TypeScript SDK](https://github.com/anthropics/anthropic-sdk-typescript)
- [Google Gen AI JavaScript/TypeScript SDK](https://github.com/googleapis/js-genai)
- [Google Gen AI SDK declaration/module mismatch issue](https://github.com/googleapis/js-genai/issues/1811)

### Build, package, runtime, and tests

- [Node.js packages](https://nodejs.org/api/packages.html)
- [Node.js native TypeScript support](https://nodejs.org/api/typescript.html)
- [Deno TypeScript](https://docs.deno.com/runtime/fundamentals/typescript/)
- [Deno Node/npm compatibility](https://docs.deno.com/runtime/fundamentals/node/)
- [Bun TypeScript](https://bun.sh/docs/typescript)
- [Cloudflare Workers TypeScript](https://developers.cloudflare.com/workers/languages/typescript/)
- [Cloudflare Workers Node compatibility](https://developers.cloudflare.com/workers/runtime-apis/nodejs/)
- [esbuild TypeScript](https://esbuild.github.io/content-types/#typescript)
- [esbuild source maps](https://esbuild.github.io/api/#sourcemap)
- [API Extractor](https://api-extractor.com/pages/overview/intro/)
- [AreTheTypesWrong](https://github.com/arethetypeswrong/arethetypeswrong.github.io)
- [tsd](https://github.com/tsdjs/tsd)
- [Vitest type testing](https://vitest.dev/guide/testing-types)
- [typescript-eslint typed linting](https://typescript-eslint.io/getting-started/typed-linting/)
- [`no-floating-promises`](https://typescript-eslint.io/rules/no-floating-promises/)
- [`use-unknown-in-catch-callback-variable`](https://typescript-eslint.io/rules/use-unknown-in-catch-callback-variable/)
- [OpenTelemetry GenAI semantic conventions repository](https://github.com/open-telemetry/semantic-conventions-genai)
- [OpenTelemetry GenAI metrics (development status)](https://github.com/open-telemetry/semantic-conventions-genai/blob/main/docs/gen-ai/gen-ai-metrics.md)

### npm supply chain

- [npm clean installs](https://docs.npmjs.com/cli/commands/npm-ci/)
- [npm audit signatures](https://docs.npmjs.com/cli/v11/commands/npm-audit/)
- [npm package provenance](https://docs.npmjs.com/viewing-package-provenance/)
- [npm install-script allowlist and strict policy](https://docs.npmjs.com/cli/install/)
- [npm install-script approval management](https://docs.npmjs.com/cli/v11/commands/npm-install-scripts/)

## Claims deliberately bounded or excluded

- Official TypeScript 7 speedups are not presented as guaranteed for every monorepo; local measurement is required.
- This packet does not claim any validator or schema library is universally best. Authority choice depends on cross-language, provider, inference, and runtime requirements.
- Cross-runtime support is not inferred from TypeScript compilation or a runtime's “Node compatible” label.
- Provider schema support is treated as a changing capability; the guides do not freeze a universal keyword list.
- TypeScript types are not described as a security boundary.
- Multi-agent architecture, prompt design, model selection, Node event-loop behavior, and general deployment topology were excluded because they are owned by other repository areas.
- No claim is made that provenance, registry signatures, `npm audit`, or SemVer proves package safety.
- The pass-2 audit did not call paid/live provider APIs or publish packages. Live schema registration, model capability, remote MCP interoperability, and staged SDK smoke tests remain project-specific release gates.
- The Google Gen AI declaration mismatch is an open maintainer-repository issue observed on the cited versions. Consumers must retest the exact version they adopt; the packet does not generalize it to all versions.
- OpenTelemetry GenAI convention names/stability and npm install-script policy behavior are explicitly refreshable. Pin the convention/package-manager line used by a deployment rather than treating the current text as permanent.

## Refresh triggers

Review this packet and the linked guides when any of the following occurs:

- TypeScript ships or materially revises the native programmatic compiler API;
- TypeScript 7 changes stable defaults, compatibility packages, or removed options;
- a supported runtime changes native TypeScript/module behavior;
- Zod/Ajv/Standard Schema or the chosen authority changes JSON Schema conversion semantics;
- a provider changes structured-output schema support or streaming event contracts;
- MCP publishes a new stable protocol/SDK major;
- a provider SDK changes its SemVer/type compatibility policy;
- npm changes lifecycle-script, signature, provenance, or lockfile behavior;
- the repository adds a new deployment host, module format, or independently deployed contract consumer.

## Resulting production principle

The reliable TypeScript agent architecture is not “types everywhere.” It is:

> host-accurate compiler configuration + `unknown` at foreign boundaries + executable/versioned contracts + app-owned state/provider types + independently verified declarations, schemas, and runtime artifacts.

That combination uses TypeScript for its strongest value—making legal domain transitions and public relationships visible—while explicitly enforcing everything its erased and intentionally pragmatic type system cannot prove.
