# Tools, MCP, and Human Approval

Tools are Mastra's effect boundary. The model may propose a call; the
application must decide whether the caller is allowed to perform it, whether the
arguments are valid, whether it is safe to repeat, and whether a human must see
the exact effect first.

## A production tool contract

Each tool should have:

- a stable ID and unambiguous description;
- a narrow input schema;
- a small, explicit output contract;
- authorization against a trusted actor and tenant;
- timeout and cancellation support;
- an idempotency key for effects;
- a receipt or durable outcome record;
- classified retry behavior;
- redacted logs and trace attributes;
- an approval rule based on risk, not prompt wording.

Mastra validates the input schema before execution and passes execution context
such as request context, tracing information, and an abort signal. Standard
JSON-schema-compatible libraries such as Zod, Valibot, and ArkType can define
schemas. Schema validation still cannot prove business authorization.

A useful implementation shape keeps the Mastra-specific wrapper thin and puts
authorization plus deduplication in an application service:

~~~ts
export const refundTool = createTool({
  id: "refund-payment",
  description: "Refund one captured payment after explicit approval.",
  inputSchema: z.object({
    paymentId: z.string(),
    amountCents: z.number().int().positive(),
    operationId: z.string(),
  }),
  outputSchema: refundReceiptSchema,
  requireApproval: true,
  execute: async (input, { requestContext, abortSignal }) => {
    const actor = requestContext.get("verifiedActor");
    if (!actor) throw new Error("missing verified actor");

    return refunds.authorizeAndExecuteOnce({
      actor,
      input,
      idempotencyKey: input.operationId,
      signal: abortSignal,
    });
  },
});
~~~

The example intentionally does not trust an actor or tenant in tool arguments.
<code>authorizeAndExecuteOnce</code> must tenant-filter the payment, recheck the
approved amount, persist a receipt, and classify timeout-after-send as
ambiguous. The model-facing result should be smaller than the internal receipt.

## Effect lifecycle

~~~mermaid
sequenceDiagram
    participant L as Model
    participant M as Mastra hook
    participant A as Authorization
    participant H as Human approval
    participant T as Tool
    participant R as Receipt store
    L->>M: Propose tool and arguments
    M->>A: Trusted actor, tenant, exact proposal
    A-->>M: Allow or deny
    opt Approval required
        M->>H: Display exact effect and expiry
        H-->>M: Signed/recorded decision
    end
    M->>T: Execute with idempotency key
    T->>R: Commit or find existing receipt
    T-->>L: Bounded, untrusted result
~~~

Authorization must run before display and again at commit if approval can wait.
Permissions, balances, or target ownership may have changed.

## Hooks and transforms

Agent-level <code>beforeToolCall</code> and <code>afterToolCall</code> hooks
apply to local tools, runtime toolsets, MCP tools, workflows, and subagent
wrappers. A before hook can skip execution and return an output. Use hooks for
consistent telemetry, coarse policy, and argument normalization; keep the final
authorization inside the tool because tools can be invoked through more than
one path.

Two transformation surfaces serve different audiences:

- <code>toModelOutput</code> limits or reshapes what the model sees;
- tool transforms shape display/transcript input, output, errors, approvals, and
  suspend payloads.

A display transform can hide sensitive fields while the application retains a
raw result. Test transform errors: current behavior should not be treated as a
guaranteed raw-output fallback.

## Approval and suspension are different

| Mechanism | Question | Example |
|---|---|---|
| Approval | “May this exact proposed effect execute?” | Charge 500 cents to account A |
| Suspension | “What information is missing to continue?” | Ask the user to select a shipping address |

Tools can require approval unconditionally or conditionally, and tools can
suspend with a schema. An agent can also require tool approval broadly. Registering
an approval- or suspension-capable tool may force tool execution to a
concurrency of one so multiple unresolved calls do not race. This favors
correctness but can surprise performance estimates.

Never implement approval as “the user clicked yes” without binding the decision
to:

- user and tenant;
- tool and normalized arguments;
- target resource/version;
- proposal or payload hash;
- run and tool-call ID;
- issue and expiry time;
- policy version;
- one-time consumption state.

After approval, execute through the same authorization and idempotency path as a
direct request.

## Auto-resume policy

Mastra can automatically resume some suspended tool flows when memory and the
same thread are available. Restrict this convenience to low-risk information
gathering. For money, deletion, publishing, permission changes, or outbound
communication, preserve an explicit product-level pending-action record and
resume only after verified consent.

## MCP client choices

The <code>@mastra/mcp</code> package provides clients for consuming MCP servers
and a server for exposing Mastra tools, agents, workflows, prompts, and
resources.

Use:

- <code>listTools()</code> for a process-level connection with shared
  credentials;
- <code>listToolsets()</code> for request-specific credentials and connection
  configuration.

Close request-scoped clients. A leaked MCP connection can retain credentials,
sockets, and server state beyond the request.

## MCP trust boundary

An MCP server is remote code and content from the agent's perspective. Its tool
descriptions, annotations, results, resources, prompts, and redirects are
untrusted.

Security controls:

1. allowlist servers and tool IDs;
2. authenticate the MCP transport;
3. propagate a narrow user/tenant identity rather than a global platform key;
4. validate tool schemas and bound results;
5. require approval for sensitive remote tools;
6. apply timeouts, quotas, and circuit breakers per server;
7. sanitize remote text before returning it to the model;
8. trace server identity and version without recording secrets.

For stdio servers, Mastra passes a curated default environment rather than the
whole process environment. Set <code>inheritDefaultEnv: false</code> and supply
an explicit environment for high-risk tools. For HTTP servers, configure allowed
hosts and redirect policy. If a custom fetch implementation follows redirects,
it must enforce the same host policy itself.

## Hosting an MCP server

When exposing Mastra capabilities through <code>MCPServer</code>:

- authenticate every client;
- authorize each tool/resource operation;
- enforce tenant filters server-side;
- use separate credentials per environment;
- protect discovery metadata when capability names are sensitive;
- rate-limit expensive agent/workflow calls;
- avoid returning raw internal errors;
- version schemas compatibly.

Mastra MCP Apps render interactive UI in a sandboxed iframe with host RPC. The
sandbox reduces access to parent DOM, cookies, and storage, but it does not make
the remote server trustworthy. Review the app and tool server as one supply
chain and data-egress boundary.

## Delegated approval caveat

GitHub issue
[#20934](https://github.com/mastra-ai/mastra/issues/20934) reported that
<code>@mastra/core@1.57.0</code> streaming agent-as-tool delegation surfaced the
outer delegate call instead of the sensitive nested tool and arguments. The
issue was closed by
[#20948](https://github.com/mastra-ai/mastra/pull/20948), but it is a valuable
regression test: a production approval UI must display the exact innermost
effect and resume the correct nested run on both approve and decline.

Do not infer safety from issue closure. Test direct, subagent, streaming, and
durable paths against the pinned version.

## Idempotency pattern

For an effecting tool:

1. derive or receive a stable operation ID;
2. start a transaction or atomic compare-and-set;
3. return the previous receipt if the operation already completed;
4. record an “executing” claim with bounded ownership;
5. call the downstream system with its idempotency key when supported;
6. store the canonical response;
7. classify timeout-after-send as ambiguous, not failed;
8. reconcile ambiguous operations from the downstream source of truth.

Framework retry configuration cannot determine whether a network timeout
happened before or after the external effect.

## Failure matrix

| Failure | Unsafe assumption | Correct response |
|---|---|---|
| Tool times out after send | “No response means no effect” | Mark ambiguous and reconcile |
| Model repeats call | “The loop will remember” | Deduplicate with operation ID |
| Approval waits for hours | “Old authorization remains valid” | Reauthorize and check expiry |
| MCP redirects | “Allowed first host covers redirects” | Validate every redirect target |
| Tool output contains instructions | “Tool data is trusted context” | Treat as untrusted and constrain |
| Nested approval shows delegate | “Any approval card is enough” | Bind and display the actual effect |

## Checklist

- [ ] Every effecting tool has authorization, idempotency, and reconciliation.
- [ ] Approval displays and binds the exact normalized effect.
- [ ] Suspend is not used as a substitute for consent.
- [ ] MCP servers and tools are allowlisted and authenticated.
- [ ] Stdio environment and HTTP redirects are restricted.
- [ ] Tool results are bounded before entering model context.
- [ ] Delegated approval paths have version-specific regression tests.
- [ ] Request-scoped MCP clients are closed.

## Primary sources

- [Tools documentation source](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/agents/tools.mdx)
- [Human-in-the-loop documentation source](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/agents/human-in-the-loop.mdx)
- [MCP documentation source](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/connections/mcp.mdx)
- [MCP package source](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/packages/mcp)
- [Nested approval regression report](https://github.com/mastra-ai/mastra/issues/20934)
- [Canonical idempotency and side-effects guide](../../reliability/idempotency-and-side-effects.md)
