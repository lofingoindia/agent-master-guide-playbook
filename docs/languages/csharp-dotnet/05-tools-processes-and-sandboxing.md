# Tools, Processes, and Sandboxing

> **Last researched:** 2026-08-31
> **Applies to:** .NET 10 LTS, C# 14

A model-proposed tool call is an untrusted request to cross a capability boundary. JSON schema validation improves shape; it does not grant permission, prove intent, or make execution safe. The dispatcher must authorize the caller, tool, target, and effect immediately before execution.

## Capability pipeline

~~~mermaid
flowchart LR
    Model[Model tool request] --> Parse[Strict parse]
    Parse --> Resolve[Resolve exact tool]
    Resolve --> Authorize[Identity and policy]
    Authorize --> Budget[Reserve time bytes effects]
    Budget --> Execute[In-process remote or process]
    Execute --> Receipt[Durable effect receipt]
    Receipt --> Redact[Bound and redact result]
    Redact --> Model
~~~

Do not expose a generic shell, filesystem root, unrestricted HTTP fetcher, or arbitrary database query when narrow capability tools can express the requirement.

## Tool contract

Each tool definition should declare:

- stable name and version;
- input and result schemas;
- caller and tenant authorization rule;
- effect class: read-only, idempotent write, or non-idempotent write;
- resource limits and deadline;
- allowed network destinations and filesystem roots;
- concurrency and idempotency behavior;
- redaction rules;
- approval requirement;
- compensation or reconciliation procedure.

Tool discovery is not authorization. The fact that an MCP client can list a tool or an Agent Framework catalog contains it must not be used as an access-control decision.

## In-process versus isolated execution

| Execution mode | Strength | Limitation |
|---|---|---|
| In-process delegate | Lowest latency, typed dependencies | Fully trusted; crash or memory corruption affects host |
| Child process | Separate lifetime and standard streams | Same identity is usually not a security boundary |
| Container / job | Resource and filesystem controls | Kernel and control-plane policy still matter |
| Remote service | Strong organizational boundary | Network ambiguity, latency, and distributed auth |
| VM / microVM | Stronger isolation | Highest operational cost |

<code>Process</code> provides lifecycle control, not a sandbox. Real isolation uses OS identities, access-control lists, container namespaces, cgroups or job objects, syscall policy, network policy, and a minimal mounted filesystem.

## Starting processes safely

Use an exact executable path and <code>ProcessStartInfo.ArgumentList</code>. Avoid a shell and string-concatenated commands.

~~~csharp
var start = new ProcessStartInfo
{
    FileName = executablePath,
    UseShellExecute = false,
    RedirectStandardInput = true,
    RedirectStandardOutput = true,
    RedirectStandardError = true,
    WorkingDirectory = isolatedWorkingDirectory,
    CreateNoWindow = true
};
start.ArgumentList.Add("--input");
start.ArgumentList.Add(validatedInputPath);
start.Environment.Clear();
start.Environment["PATH"] = minimalPath;
~~~

On modern .NET, <code>UseShellExecute</code> defaults to false, but set it explicitly at a security boundary. Never place secrets on the command line; process listings and crash data can expose them. Pass the minimum environment and use a dedicated secret transport where possible.

## Preventing pipe deadlocks

When stdout and stderr are redirected, drain both concurrently. Waiting for process exit before reading, or reading one stream to completion before the other, can deadlock when an OS pipe fills.

Bound retained output while continuing to drain excess bytes. If the cap is exceeded:

1. mark the result truncated;
2. stop accepting more retained content;
3. keep draining or terminate the process;
4. record total observed bytes;
5. return a structured error, not silently valid-looking partial JSON.

Decode incrementally and set a maximum line/frame length. A tool can emit a single line large enough to exhaust memory.

## Termination

<code>WaitForExitAsync</code> cancellation stops the wait, not necessarily the process. A tool owner should:

1. close stdin or send a supported graceful signal;
2. wait for a small reserved interval;
3. call <code>Kill(entireProcessTree: true)</code> when escalation is authorized;
4. await exit and finish draining;
5. dispose the process;
6. report the possibility of surviving descendants where the platform cannot prove tree termination.

.NET 10 adds Windows process-group support through <code>ProcessStartInfo.CreateNewProcessGroup</code>. It improves group signaling but does not make behavior identical across Windows, Linux, containers, and init systems.

## MCP-specific boundaries

- Stdio servers must write protocol bytes only to stdout; logs go to stderr.
- Streamable HTTP is the current recommended network transport; legacy SSE exists for compatibility.
- Modern stateless HTTP simplifies scaling, but "stateless" describes the protocol session, not the tool's business state. Use explicit handles and external stores for application state; stateful legacy clients require affinity or externalized lifecycle state.
- HTTP requests can use a <code>ClaimsPrincipal</code>; stdio has no built-in remote authentication boundary.
- In the C# SDK, <code>[Authorize]</code> attributes are enforced for MCP tools, prompts, and resources only after authorization filters are registered with <code>AddAuthorizationFilters()</code>. Test both discovery filtering and direct invocation denial.
- Structured tool output advertises an output schema; it does not validate the caller's permission or make returned content safe for a model.
- The SDK's in-memory MCP task store is for development and tests. Production task execution needs a durable, tenant-scoped implementation and cooperative cancellation.
- Treat resource contents, prompts, tool metadata, and tool results as untrusted and size-bound.
- Allowlist servers and capabilities. Remote MCP discovery must not become automatic tool installation.

## Secret containment

Tool workers should receive capabilities, not the host's ambient credential set. Clear inherited environment variables, mount only required secret material, and prefer short-lived workload identity or a narrowly scoped brokered credential. Never expose provider keys, database credentials, managed-identity tokens, or control-plane sockets merely because a tool may need network access.

Test containment with a deliberately secret-shaped canary: it must not appear in process arguments, inherited environment, stdout/stderr, model-visible results, traces, dumps uploaded by the normal workflow, or durable history. Redaction is a final guard, not the permission boundary.

## Failure patterns

- Valid schema bypasses authorization.
- A URL fetch tool permits loopback, metadata-service, or private-network access.
- Shell quoting is assumed portable across operating systems.
- The process is killed but grandchildren retain files or ports.
- Output is capped by stopping reads, causing the child to block forever.
- The container image is chiseled/distroless but a tool assumes a shell or package manager.
- A tool result includes secrets that are then sent to the model and telemetry.

## Review checklist

- [ ] Tool identity, authorization, and effect class are explicit.
- [ ] Executable and arguments are separated; no shell by default.
- [ ] stdin, stdout, stderr, runtime, bytes, files, and network are bounded.
- [ ] Both output streams are drained concurrently.
- [ ] Cancellation escalates to process termination when required.
- [ ] Child-process isolation is not described as sandboxing.
- [ ] Tool results are bounded, classified, and redacted before reuse.

## Primary sources

- [ProcessStartInfo.UseShellExecute](https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.processstartinfo.useshellexecute?view=net-10.0)
- [Process.StandardOutput and deadlock guidance](https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.process.standardoutput?view=net-10.0)
- [Process.Kill](https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.process.kill?view=net-10.0)
- [.NET 10 process changes](https://learn.microsoft.com/en-us/dotnet/core/whats-new/dotnet-10/libraries)
- [MCP C# transport concepts](https://github.com/modelcontextprotocol/csharp-sdk/blob/main/docs/concepts/transports/transports.md)
- [MCP C# identity concepts](https://github.com/modelcontextprotocol/csharp-sdk/blob/main/docs/concepts/identity/identity.md)
- [MCP C# authorization filters](https://github.com/modelcontextprotocol/csharp-sdk/blob/main/docs/concepts/filters.md)
- [MCP C# tool schemas](https://github.com/modelcontextprotocol/csharp-sdk/blob/main/docs/concepts/tools/tools.md)
- [MCP C# tasks](https://github.com/modelcontextprotocol/csharp-sdk/blob/main/docs/concepts/tasks/tasks.md)
- [MCP authorization specification](https://modelcontextprotocol.io/specification/2025-06-18/basic/authorization)
