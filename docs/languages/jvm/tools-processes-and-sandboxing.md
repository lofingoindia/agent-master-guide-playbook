# Tools, Processes, and Sandboxing

## A tool is an untrusted effect

A model chooses a tool and proposes arguments; trusted code authorizes and executes it. Keep those phases separate:

~~~mermaid
flowchart LR
    M[Model proposal] --> V[Schema validation]
    V --> P[Policy and approval]
    P --> R[Effect record]
    R --> X[Isolated execution]
    X --> N[Normalize and redact]
    N --> S[Persist outcome]
~~~

The model never chooses executable paths, credentials, network policy, environment variables, working directories, or raw shell syntax.

## Java subprocess rules

Use <code>ProcessBuilder</code> with an argument list. Avoid a shell unless shell grammar is the actual tool, then run it only inside a hardened worker with strict allowlists.

Always drain stdout and stderr concurrently or redirect them. OS pipe buffers are finite; an unconsumed stream can block the child and deadlock the parent. Apply byte limits while draining—reading into an unbounded string merely moves the denial of service into heap.

Termination sequence:

1. close child stdin;
2. request normal termination with <code>destroy()</code>;
3. wait for a short bounded period;
4. request <code>destroyForcibly()</code>;
5. wait again and record survivors.

<code>destroyForcibly()</code> may return before the process exits. Cancelling <code>onExit()</code> does not terminate the process. <code>ProcessHandle.descendants()</code> is a racing snapshot and PIDs can be reused, so portable “kill the entire tree” is not a reliable sandbox primitive.

A runner should receive a server-resolved executable and argument list, never model-authored shell text, and should own the drainers for exactly the child lifetime:

~~~java
Process process = new ProcessBuilder(approvedCommand)
    .directory(approvedScratch.toFile())
    .redirectInput(ProcessBuilder.Redirect.PIPE)
    .start();
process.getOutputStream().close();

Future<BoundedBytes> stdout = ioExecutor.submit(
    () -> drain(process.getInputStream(), STDOUT_LIMIT));
Future<BoundedBytes> stderr = ioExecutor.submit(
    () -> drain(process.getErrorStream(), STDERR_LIMIT));

if (!process.waitFor(run.remaining().toMillis(), TimeUnit.MILLISECONDS)) {
    process.destroy();
    if (!process.waitFor(TERM_GRACE.toMillis(), TimeUnit.MILLISECONDS)) {
        process.destroyForcibly();
        process.waitFor(KILL_GRACE.toMillis(), TimeUnit.MILLISECONDS);
    }
}
ToolResult result = normalize(process, stdout, stderr);
~~~

`drain` stops and triggers termination when the byte cap is exceeded; `normalize` uses bounded waits and reports surviving process state rather than blocking forever. In production, run the command in the selected container/microVM boundary, and let that boundary own process-tree/cgroup cleanup. The JVM parent alone cannot make this snippet a sandbox.

## The JVM is not the sandbox

The Java Security Manager is permanently disabled starting with JDK 24 and cannot be enabled on JDK 25. Class loaders, reflection filters, a subprocess, or a custom <code>SecurityManager</code> do not create a security boundary.

Run risky tools in a separate OS identity, container, microVM, or dedicated worker pool with:

- read-only root filesystem and explicit writable scratch directory;
- CPU, memory, process, file-size, and wall-time limits;
- network deny-by-default with destination allowlists;
- minimal filesystem mounts;
- no host Docker socket or cloud instance credentials;
- per-effect short-lived credentials;
- syscall/capability restrictions appropriate to the platform;
- output and artifact scanning before re-entry.

Container isolation is weaker than a VM boundary and depends on runtime/kernel configuration. Select isolation by consequence, not convenience.

## Policy and approval

Tool policy consumes normalized data: tenant, principal, run, tool version, canonical argument digest, requested destinations/resources, and risk classification. Store the policy version and decision with the planned effect.

Approvals are capabilities, not chat text. Bind them to the exact effect ID and digest, approver, expiry, and allowed action. A model rewrite or argument change requires a new approval.

## Tool result contract

Return a typed envelope:

| Field | Purpose |
|---|---|
| effect ID and attempt | correlation and replay safety |
| status | success, domain rejection, transient failure, unknown |
| structured value | schema-validated bounded output |
| human summary | safe prompt-facing view |
| artifacts | content-addressed references, not arbitrary host paths |
| diagnostics | redacted, access-controlled |
| timing/usage | budgets and operations |

Separate prompt-facing output from raw diagnostics. Tool output can contain prompt injection; label it as untrusted evidence and never splice it into system instructions.

## In-process tools

Pure computations can run in-process if they are bounded and have no ambient authority. Still apply input validation, CPU/size limits, cancellation checks, and telemetry. Do not load third-party model-selected classes or serialized objects.

## Checklist

- [ ] Tool name resolves through a server-side registry.
- [ ] Arguments are schema-validated and canonicalized.
- [ ] Authorization occurs immediately before execution.
- [ ] Effect is persisted before execution.
- [ ] Subprocess streams are drained concurrently with byte caps.
- [ ] Process termination is awaited and survivors are reported.
- [ ] Risky work uses OS/container/VM isolation.
- [ ] Credentials and network access are least privilege.
- [ ] Results are normalized, bounded, and prompt-injection aware.

## Sources

- [Java Process API](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/Process.html)
- [Java ProcessHandle API](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/ProcessHandle.html)
- [JDK 25: Security Manager is permanently disabled](https://docs.oracle.com/en/java/javase/25/security/security-manager-is-permanently-disabled.html)
- [OWASP OS command injection defense](https://cheatsheetseries.owasp.org/cheatsheets/OS_Command_Injection_Defense_Cheat_Sheet.html)
