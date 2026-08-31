# Packages, Migration, Limitations, and Alternatives

> Research date: **2026-08-31** | Source snapshot: Vercel AI repository at commit `e1bfe50427d09e65404cffea9f71a60a66af0f3e` (2026-08-30).

AI SDK is a coordinated package family. Upgrade it as a tested compatibility set, not as unrelated packages whose major versions happen to differ.

## Researched package snapshot

| Package | Version | Role |
| --- | ---: | --- |
| `ai` | 7.0.85 | Core APIs, messages, tools, agents, UI server utilities |
| `@ai-sdk/provider` | 4.0.9 | Provider specification (`LanguageModelV4`) |
| `@ai-sdk/provider-utils` | 5.0.34 | Adapter HTTP, schemas, retries, streaming utilities |
| `@ai-sdk/react` | 4.0.88 | React UI hooks |
| `@ai-sdk/otel` | 1.0.85 | OpenTelemetry integration |
| `@ai-sdk/devtools` | 1.0.14 | Local generation inspection |
| `@ai-sdk/workflow` | 2.0.15 | `WorkflowAgent` and transport integration |
| `workflow` peer | `^5.0.0-beta.42` | Workflow runtime; still beta at snapshot |
| `@ai-sdk/openai` | 4.0.52 | Direct OpenAI adapter |
| `@ai-sdk/anthropic` | 4.0.46 | Direct Anthropic adapter |
| `@ai-sdk/google` | 4.0.58 | Direct Google adapter |
| `@ai-sdk/gateway` | 4.0.69 | AI Gateway adapter |

These are evidence, not evergreen recommendations. Read the lockfile, package manifests, changelogs, and migration guide at upgrade time.

## AI SDK 7 baseline

Version 7 requires Node.js 22+, is ESM-only, and is tested by the project on Node 22, 24, and 26. Use a currently maintained Node line. The core provider interchange is v4; indexed documentation or examples mentioning `LanguageModelV3` or `MockLanguageModelV3` are stale for this snapshot.

Run the official codemod in a clean branch, then review every change:

```bash
npx @ai-sdk/codemod v7
```

Material v7 changes include:

- `system` to `instructions` in affected APIs;
- lifecycle `onFinish` to `onEnd` and step/tool callback renames;
- `experimental_telemetry` to `telemetry` and global `registerTelemetry` integration;
- stable `prepareStep`, `activeTools`, context, image/speech/transcription names;
- structured output through `Output.*` on `generateText`/`streamText`;
- deprecated `generateObject`/`streamObject`;
- Core approval through `toolApproval`, while WorkflowAgent still uses `needsApproval`;
- `fullStream` renamed to `stream` and usage detail restructuring.

Codemods cannot decide runtime compatibility, persisted message migrations, callback semantics, policy behavior, or provider capability. Test those manually.

## Upgrade process

1. Inventory every `ai`, `@ai-sdk/*`, `workflow`, adapter, and framework UI package plus experimental symbols.
2. Snapshot provider contract fixtures, UI stream fixtures, persisted-message fixtures, eval scores, and cost/latency baselines.
3. Upgrade the package family and Node/ESM build in one controlled branch.
4. Run codemods, type checking, tests, and lint; inspect callback and telemetry diffs manually.
5. Test direct provider and Gateway paths, tools/approvals, structured output, abort, retry, and errors after stream start.
6. Run persistence migration and rolling client/server compatibility tests.
7. For Workflow, test suspended old runs against the deployment strategy.
8. Canary with provider/model/attempt telemetry and a rollback plan.

Pin exact versions for experimental APIs and `workflow@beta`. Even on stable packages, use a lockfile and deliberate update automation rather than unconstrained production resolution.

## Limitations

- Provider neutrality does not guarantee capability or behavior parity.
- In-process agents are not durable and durable agents do not make external effects exactly once.
- AI SDK UI offers state/transport primitives, not authorization or a persistence database.
- Telemetry and DevTools can capture sensitive content unless configured carefully.
- The SDK has testing primitives but not a complete application evaluation/governance platform.
- Gateway routing, budgets, Vercel cancellation, Fluid concurrency, and Workflow execution are platform guarantees, not Core guarantees.
- Experimental and beta APIs can change outside normal stable expectations.

## Alternatives and complements

| Requirement | Consider |
| --- | --- |
| One provider, maximum native feature access | Provider's official SDK directly |
| Explicit deterministic orchestration | Plain application code, state machine, or established workflow engine |
| Durable jobs without agent-specific abstraction | Queue/workflow platform with Core calls inside steps |
| Python-first agent stack | A Python-native SDK/framework after equivalent production review |
| Strong UI portability with provider choice | AI SDK UI/Core remains a strong fit |
| Managed routing and spend visibility | AI Gateway or another gateway, evaluated independently |

The simplest reliable alternative is often direct provider SDK plus explicit application code. Choose AI SDK when its normalized provider, stream, tool, message, and UI contracts remove more complexity than they add.

## Refresh triggers

Refresh this area when `ai` changes major, provider specification changes, Workflow 5 leaves beta, approval or Output APIs stabilize differently, Vercel runtime/cancellation limits change, or a security advisory affects an installed package.

## Sources

- [AI SDK repository package manifests](https://github.com/vercel/ai)
- [AI SDK 7 migration guide](https://ai-sdk.dev/docs/migration-guides/migration-guide-7-0)
- [AI SDK versioning](https://ai-sdk.dev/docs/migration-guides/versioning)
- [AI SDK changelogs](https://github.com/vercel/ai/tree/main/packages)
- [`@ai-sdk/workflow` package](https://github.com/vercel/ai/tree/main/packages/workflow)
- [Node.js release schedule](https://nodejs.org/en/about/previous-releases)

