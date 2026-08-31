# NuGet, Supply Chain, and Build

> **Last researched:** 2026-08-31
> **Applies to:** .NET 10 LTS, C# 14

Agent applications combine fast-moving provider SDKs, protocol libraries, telemetry exporters, workflow engines, and source generators. Dependency policy is part of runtime safety because packages can alter HTTP policy, serialization, trimming, build behavior, and transitive code.

## Reproducible baseline

Pin:

- SDK in <code>global.json</code> with an intentional roll-forward policy;
- target framework and runtime identifiers;
- package versions centrally in <code>Directory.Packages.props</code>;
- application dependency graph with <code>packages.lock.json</code>;
- container base by supported patch and, where required, digest;
- tool/schema/protocol versions in runtime state.

For deployable applications, run <code>dotnet restore --locked-mode</code> in CI. A library lock file does not control the graph selected by every consuming application.

## Package review

NuGet packages can contribute more than assemblies:

- <code>build</code> and <code>buildTransitive</code> targets;
- analyzers and source generators that execute during build/IDE use;
- native assets and runtime-specific binaries;
- content files and tools;
- transitive packages from additional sources.

Review the package archive and transitive graph for security-critical dependencies. Prefer the official package ID and repository; similarly named SDKs are common in the agent ecosystem. For example, the current official Anthropic C# package is <code>Anthropic</code>, while older community packages use other IDs.

## Sources and trust

Package Source Mapping restricts which configured source may supply packages matching a pattern, including transitive dependencies. Once enabled, every package must match a source. Design non-overlapping patterns where possible and keep internal package IDs in an internal namespace.

Mapping does not prevent all metadata queries to other configured sources, so remove unused feeds and protect credentials. Use trusted signers/package signature validation when the organization has an operational signing policy; signatures complement, not replace, version pinning and review.

## Vulnerability auditing

.NET 10 changes NuGet auditing so transitive packages are included by default when targeting .NET 10. Run audit in CI, set a severity policy, and treat the resulting graph as triage input:

- identify whether the vulnerable code path is reachable;
- upgrade or remove the package;
- document a time-bounded suppression with owner and expiry;
- re-evaluate after every graph change;
- distinguish tooling-only and shipped runtime assets.

Do not silently suppress an advisory because the top-level SDK has not yet released an update. Consider an alternate adapter, isolated worker, or explicit risk acceptance.

## Build quality gates

Recommended gates:

1. restore in locked mode;
2. build with warnings treated according to an explicit policy;
3. run analyzers and nullable checks;
4. test supported target frameworks/runtimes;
5. publish the exact deployment mode;
6. treat trimming and AOT warnings as release blockers for AOT targets;
7. scan packages and container layers;
8. generate an SBOM and provenance/attestation where the delivery system supports it;
9. run smoke tests from the produced artifact.

Deterministic compilation improves reproducibility but does not prove a trustworthy toolchain. Pin and protect CI images, credentials, feeds, signing identities, and artifacts.

## SDK upgrade protocol

Agent packages move quickly and may label individual APIs experimental even in a stable package. For every upgrade:

- read release notes and migration guides;
- diff transitive packages;
- verify retry/timeout defaults;
- regenerate and diff JSON schemas;
- replay provider and MCP contract fixtures;
- test tracing changes and attribute cardinality;
- publish trimmed/AOT artifacts if applicable;
- run a canary with logical-call/network-attempt metrics;
- preserve a rollback artifact and state compatibility.

Do not use floating versions or prerelease packages by default. If a required integration is prerelease, isolate it behind an adapter and pin the exact version.

## Failure patterns

- Package name is trusted without verifying publisher/repository.
- An agent SDK upgrade silently changes retries or wire contracts.
- A lock file is generated but CI does not use locked restore.
- Public and private feeds contain the same package IDs.
- A build-transitive target executes with broad CI credentials.
- Vulnerability warnings are suppressed indefinitely.
- AOT warnings are ignored because the Debug build works.

## Review checklist

- [ ] SDK, packages, feeds, and container bases are pinned.
- [ ] Deployable projects restore in locked mode.
- [ ] Package Source Mapping covers every direct/transitive package.
- [ ] Build scripts, analyzers, generators, native assets, and licenses are reviewed.
- [ ] Transitive vulnerability audit runs in CI.
- [ ] Prerelease/experimental integrations are isolated and tracked.
- [ ] Published artifacts receive smoke and compatibility tests.

## Primary sources

- [PackageReference in project files](https://learn.microsoft.com/en-us/nuget/consume-packages/package-references-in-project-files)
- [Central Package Management](https://learn.microsoft.com/en-us/nuget/consume-packages/central-package-management)
- [Package Source Mapping](https://learn.microsoft.com/en-us/nuget/consume-packages/package-source-mapping)
- [NuGet package auditing](https://learn.microsoft.com/en-us/nuget/concepts/auditing-packages)
- [Install signed NuGet packages](https://learn.microsoft.com/en-us/nuget/consume-packages/installing-signed-packages)
- [.NET 10 transitive NuGet audit change](https://learn.microsoft.com/en-us/dotnet/core/compatibility/sdk/10.0/nugetaudit-transitive-packages)
- [.NET reproducible-build guidance](https://github.com/dotnet/reproducible-builds)
