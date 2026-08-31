# Dependencies, Build, and Supply Chain

## Pin the execution environment

Agent services combine fast-moving provider, protocol, framework, telemetry, serialization, and workflow libraries. A reproducible graph is a reliability and security control.

Pin:

- JDK vendor, major, and patch for build and runtime;
- Gradle/Maven wrapper version and distribution checksum;
- Kotlin compiler/plugin version;
- direct and transitive library versions through a reviewed BOM/lock;
- container base image by digest;
- OpenTelemetry Java agent by version and checksum.

Compile <code>--release</code> and runtime testing are separate. A class file that targets an older release can still fail because a dependency or runtime behavior differs.

## Gradle

Use JVM toolchains so developer <code>JAVA_HOME</code> does not silently select the compiler. Dependency locking records resolved transitive versions and should be committed; strict mode catches missing lock state. Do not use changing/SNAPSHOT artifacts in a release graph.

Dependency verification checks artifact checksums/signatures. Review generated verification metadata before committing it—blindly accepting a compromised graph only pins the compromise. Centralize allowed repositories and plugin repositories; repository order can change which bytes satisfy the same coordinate.

Verify the Gradle wrapper distribution and wrapper JAR. Keep build logic small: plugins execute code during the build and are part of the threat model.

## Maven

Use the Maven Wrapper with <code>wrapperSha256Sum</code> and <code>distributionSha256Sum</code>. Enforce Maven/JDK versions and dependency convergence. Import vendor/framework BOMs deliberately and inspect the resolved tree; nearest-wins resolution can otherwise conceal an incompatible version.

For reproducibility:

- avoid version ranges and snapshots;
- pin plugin versions;
- set <code>project.build.outputTimestamp</code>;
- run <code>artifact:check-buildplan</code>;
- compare clean-build artifacts when publishing;
- remember OS newlines and JDK major can affect output.

## Dependency conflict hotspots

| Area | Typical conflict |
|---|---|
| JSON | Jackson 2 versus 3 package ecosystem; Kotlin serialization adapters |
| HTTP | provider SDK transport versions and framework clients |
| logging | multiple SLF4J providers or bridge loops |
| telemetry | OTel API/SDK/semantic conventions versus auto-agent |
| MCP | Jackson 2/3 modules and Reactor versions |
| coroutines | Kotlin compiler/plugin/library alignment |
| cloud/auth | Netty/gRPC/credential library convergence |

Prefer the framework or SDK BOM that owns an integration, then test other adapters against it. Avoid excluding transitive dependencies until the effect is understood and verified.

## Supply-chain controls

~~~mermaid
flowchart LR
    S[Source and lockfiles] --> B[Hermetic-ish CI build]
    B --> T[Tests and scans]
    T --> M[SBOM and provenance]
    M --> I[Signed image by digest]
    I --> D[Policy-controlled deploy]
~~~

- allow only required repositories through a trusted mirror;
- isolate release credentials from pull-request builds;
- generate an SBOM for application, agent JAR, native libraries, and image;
- scan continuously because vulnerabilities appear after release;
- sign/provide provenance for artifacts and images;
- run builds with least privilege and no production credentials;
- review provider/MCP/telemetry updates for behavioral, not only binary, compatibility.

Auto-instrumentation agents transform bytecode and have broad visibility. Treat their update as an application dependency change. The OpenTelemetry Java instrumentation project has shipped security fixes; pin a current patched release rather than an old copied coordinate.

### Release evidence bundle

Attach or retain for each deployable image:

- source revision and clean/dirty build status;
- JDK vendor/full build string and base-image digest;
- Gradle/Maven wrapper and resolved dependency/plug-in lock evidence;
- SBOM for JARs, native libraries, Java agents, and container packages;
- image digest, signature/provenance attestation, and vulnerability-scan time;
- provider/MCP/schema fixture versions exercised by CI;
- preview flags, JVM options, and telemetry-agent checksum;
- migration/rollback compatibility window for event, queue, and workflow state.

An SBOM is inventory, not proof of safety. Repository allowlists, checksum/signature verification, least-privilege builds, advisory monitoring, and a tested rebuild/rollback path remain necessary. Generated provider clients and schema/code-generation plugins are executable supply-chain inputs too.

## Java modules and runtime images

<code>jdeps</code> can reveal JDK module dependencies; <code>jlink</code> assembles a custom runtime from modules. A smaller image can reduce surface and startup transfer, but reflection-heavy frameworks, service loading, dynamic agents, charsets, TLS providers, and attach/JFR tooling need explicit validation. Do not optimize away incident-response capabilities without an alternative.

## Upgrade workflow

1. read official release notes and security advisories;
2. update lock/BOM in an isolated change;
3. inspect dependency diff and duplicate providers;
4. run Java/Kotlin compilation, schema goldens, stream fixtures, and cancellation tests;
5. exercise MCP/provider feature probes;
6. run load and shutdown tests for runtime/JDK changes;
7. canary with comparable JFR/OTel evidence;
8. retain a rollback image and schema compatibility.

## Checklist

- [ ] Toolchain and wrapper are pinned and verified.
- [ ] Complete transitive graph is locked/convergent.
- [ ] Repositories are centralized and allowlisted.
- [ ] Plugins and Java agents are treated as executable dependencies.
- [ ] Releases avoid snapshots and version ranges.
- [ ] SBOM, provenance/signing, scanning, and digest deployment exist.
- [ ] Jackson/Kotlin/OTel/MCP convergence is tested.
- [ ] Runtime-image minimization preserves required diagnostics.

## Sources

- [Gradle toolchains](https://docs.gradle.org/current/userguide/toolchains.html)
- [Gradle dependency locking](https://docs.gradle.org/current/userguide/dependency_locking.html)
- [Gradle build security](https://docs.gradle.org/current/userguide/security.html)
- [Maven Wrapper checksum verification](https://maven.apache.org/tools/wrapper/index.html)
- [Maven dependency convergence](https://maven.apache.org/enforcer/enforcer-rules/dependencyConvergence.html)
- [Maven reproducible builds](https://maven.apache.org/guides/mini/guide-reproducible-builds.html)
- [Oracle jlink](https://docs.oracle.com/en/java/javase/25/docs/specs/man/jlink.html)
