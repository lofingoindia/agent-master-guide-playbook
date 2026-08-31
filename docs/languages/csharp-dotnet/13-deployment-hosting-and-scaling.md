# Deployment, Hosting, and Scaling

> **Last researched:** 2026-08-31
> **Applies to:** .NET 10 LTS, C# 14

Deployment is where ownership assumptions meet real signals, limits, filesystems, identities, and restarts. Test the published image, not just the project.

## Deployment modes

| Mode | Benefit | Cost |
|---|---|---|
| Framework-dependent | Smaller app artifact, normal diagnostics/dynamic behavior | Requires compatible runtime on host |
| Self-contained | Runtime travels with application | Larger artifact and patch responsibility |
| Trimmed | Smaller footprint | Reflection/dynamic feature risk |
| Native AOT | Fast startup and potentially lower memory | No JIT/dynamic loading; strict compatibility and diagnostics trade-offs |

Native AOT is not an automatic production optimization for an agent service. Provider SDKs, JSON serialization, DI activation, proxy generation, plugins, workflow engines, and telemetry exporters may use reflection or dynamic code. Publish and run the exact dependency graph; treat AOT analysis warnings as defects.

Use System.Text.Json source generation and avoid runtime assembly loading. Native AOT does not support dynamic loading such as <code>Assembly.LoadFile</code> and limits runtime code generation. Generic instantiations can also increase binary size.

Framework-dependent deployments receive runtime servicing when the host runtime is patched. Self-contained and container deployments carry their runtime, so the application owner must rebuild and redeploy for .NET servicing/security releases. Record runtime patch, base-image digest, package lock, TFM, RID, architecture, libc, and globalization/time-zone mode in the release manifest.

## Compatibility evidence

Treat each supported artifact as a tested product, not a build flag.

| Dimension | Release gate |
|---|---|
| Runtime/OS | Start and stop on every TFM, RID, architecture, libc, and base-image variant claimed |
| Serialization | Source-generated contexts cover wire/state/tool contracts; trim/AOT warnings are reviewed as defects |
| SDK maturity | Stable package and individual experimental/prerelease APIs are recorded separately |
| Dynamic features | DI activation, proxies, plugins, workflow replay, exporters, and reflection paths execute from the published artifact |
| Tool/process | Exact executable exists; no hidden shell/package-manager assumption; signals and descendant cleanup work on the target OS |
| Network | DNS rotation, proxy, CA/TLS policy, HTTP/2 behavior, identity endpoint, and egress policy are exercised |
| Diagnostics | Required counters/traces/dumps work, or the intentional limitation and incident alternative are documented |

A useful smoke suite starts the real image as its production user, validates configuration without printing secrets, serializes each persisted/schema type, runs a fragmented fake provider stream and constrained process tool, checks readiness/drain, and confirms no writable path or network destination beyond policy.

## Container choices

Official .NET images separate SDK build images from <code>aspnet</code>, <code>runtime</code>, and <code>runtime-deps</code> runtime images. Use multi-stage builds and ship only required runtime content.

Chiseled/distroless variants reduce packages and are non-root by default, but generally omit a shell and package manager. Some variants omit ICU and time-zone data. This improves attack surface while breaking tool implementations that assume <code>/bin/sh</code>, local package installation, globalization, or zone databases.

Do not add a shell to the main agent image solely for model-proposed commands. Use a separate constrained tool-worker image.

## Workload contract

~~~mermaid
flowchart TD
    L[Load balancer] --> Ready[Readiness]
    Ready --> API[Stateless API/admission]
    API --> Q[Durable queue/state]
    Q --> W[Worker pool]
    W --> P[Provider]
    W --> T[Isolated tool pool]
    Stop[Termination signal] --> Ready
    Stop --> Drain[Bounded drain]
    Drain --> Q
~~~

Readiness means the instance can admit new work, not merely that the process is alive. On termination, fail readiness first, stop consumers/admission, drain within the orchestrator grace period, persist/settle, cancel, and terminate children. Keep the host shutdown budget shorter than the platform's hard kill interval.

Liveness should detect an irrecoverable process, not restart a healthy instance merely because a provider is down. Dependency failures belong in readiness or separate health detail only when that dependency is required for admission.

## Security posture

- run as non-root with a read-only root filesystem;
- mount a dedicated bounded writable workspace;
- drop Linux capabilities and restrict syscalls;
- use workload/managed identity rather than long-lived keys where supported;
- restrict egress to approved provider/tool endpoints;
- separate model-facing tool workers from control-plane credentials;
- do not put secrets in images, command lines, logs, or model context;
- protect diagnostic ports and dump storage;
- scan and sign artifacts according to the supply-chain policy.

On Azure, use Microsoft Entra ID and managed identity for Azure OpenAI/Foundry where feasible. Assign the narrow data-plane role; <code>DefaultAzureCredential</code> convenience does not replace explicit production identity configuration.

### Secret lifecycle

ASP.NET Core Secret Manager is development-only: its values are not encrypted and it is not a trusted production store. Environment variables are also readable in process/container environments and often leak through diagnostics. In production, prefer workload identity/federation where the dependency supports it; otherwise retrieve a narrowly scoped secret from a controlled secret manager such as Azure Key Vault.

Validate required secret names and access at startup/readiness without logging values. Define rotation behavior: bounded cache lifetime, overlapping credentials when supported, retry only after a confirmed credential refresh, and revocation drills that do not require image rebuilds. Keep secrets out of model context, tool schemas/results, command arguments, child environments, exception data, health responses, telemetry, and durable events. Test redaction and scan the published image/configuration for known canaries.

## Horizontal scaling

Keep API/front-end instances stateless. Externalize sessions, run state, effect receipts, leases, and large content. A stateful MCP or provider session needs affinity or an external lifecycle store; prefer stateless protocols where semantics permit.

Workers require:

- broker prefetch/concurrency tuned to resource admission;
- fencing tokens for ownership;
- idempotent activities and effects;
- per-tenant/provider partitions;
- autoscaling on queue age plus saturation, not queue length alone;
- scale-down drain and lease handoff;
- separate pools for trusted in-process work and untrusted/heavy tools.

Autoscaling cannot repair provider rate limits. More replicas can worsen throttling unless global/partitioned admission coordinates them.

## State and storage

Large prompts, attachments, stream logs, and tool outputs belong in object storage with content hash, size, media type, encryption metadata, and tenant ownership. Databases store bounded references and state. Define retention and deletion for both.

Use an outbox or atomic state/message strategy when a state commit must trigger future work. A crash between database commit and broker send otherwise strands the run.

## Failure patterns

- Native AOT is selected before verifying SDK compatibility.
- Runtime image lacks ICU/time-zone data and changes behavior.
- Tool code assumes a shell in a chiseled image.
- Kubernetes kills the pod before the .NET host drain completes.
- Liveness depends on a provider and causes a restart storm.
- Every replica enforces only a local provider rate limit.
- Scale-down abandons processes or loses in-memory work.

## Review checklist

- [ ] Exact published artifact runs integration and shutdown tests.
- [ ] Base image, runtime mode, globalization, and time-zone needs are explicit.
- [ ] Readiness stops before drain; platform grace exceeds host budget.
- [ ] Identity, filesystem, egress, diagnostics, and tool isolation are restricted.
- [ ] Runtime/base-image patch ownership and rebuild cadence are explicit.
- [ ] Secrets use workload identity or a controlled store, rotate without leakage, and never flow to model/tool context by default.
- [ ] State/effects are externalized for horizontal scaling.
- [ ] Admission works across replicas, not only locally.
- [ ] Autoscaling uses queue age and resource saturation.

## Primary sources

- [.NET Native AOT deployment](https://learn.microsoft.com/en-us/dotnet/core/deploying/native-aot/)
- [ASP.NET Core Native AOT](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/native-aot?view=aspnetcore-10.0)
- [.NET container images](https://learn.microsoft.com/en-us/dotnet/core/docker/container-images)
- [Official .NET container repository](https://github.com/dotnet/dotnet-docker)
- [.NET image variants](https://github.com/dotnet/dotnet-docker/blob/main/documentation/image-variants.md)
- [Ubuntu chiseled images](https://github.com/dotnet/dotnet-docker/blob/main/documentation/ubuntu-chiseled.md)
- [Authenticate to Azure OpenAI with .NET](https://learn.microsoft.com/en-us/dotnet/ai/azure-ai-services-authentication)
- [Safe storage of app secrets in development](https://learn.microsoft.com/en-us/aspnet/core/security/app-secrets?view=aspnetcore-10.0)
- [Azure Key Vault security guidance](https://learn.microsoft.com/en-us/azure/key-vault/general/secure-key-vault)
- [System.Text.Json source generation](https://learn.microsoft.com/en-us/dotnet/standard/serialization/system-text-json/source-generation)
- [Generic Host shutdown](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/host/generic-host?view=aspnetcore-10.0)
