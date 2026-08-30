# Evaluation and Observability

> **Status:** Research-backed core available; workload-specific and online-evaluation deep dives remain queued.  
> **Last researched:** 2026-08-30  
> **Evidence:** [Security, evaluation, context, and memory research packet](../research/packets/security-evaluation-context-memory.md)

## Read in this order

1. [Evaluation-driven development](evaluation-driven-development.md) — task contracts, grader hierarchy, datasets, release gates, and the failure flywheel.
2. [Trajectory and reliability evaluation](trajectory-and-reliability-evaluation.md) — state versus path, invariants, repeated trials, `pass@k` versus `pass^k`, and adversarial evaluation.
3. [Observability and tracing](observability-and-tracing.md) — application event schema, OTel mapping, privacy, metrics, effect evidence, and debugging.

## Evaluation layers

| Layer | Questions |
|---|---|
| Outcome | Did the real task reach the correct, policy-compliant state? |
| Trajectory | Were tools, arguments, order, and recovery decisions appropriate? |
| Reliability | Does it pass repeatedly, under perturbation and partial failure? |
| Efficiency | What tokens, cost, latency, tool calls, and retries were required? |
| Safety | Were forbidden or unnecessarily risky actions attempted? |
| Operations | Can a trace explain the failure and support replay or incident response? |

## Stable baseline

- Grade observable environment state and effects before the final narrative.
- Express path requirements as required/forbidden events, partial orders, dataflow constraints, and budgets unless one exact sequence is truly required.
- Use deterministic graders first; calibrate model judges against reviewed human labels.
- Repeat stochastic tasks, show uncertainty and slices, and distinguish `pass@k` from `pass^k`.
- Version the task, environment, simulator, model, prompt, tools, policy, harness, and grader.
- Keep a durable effect/audit ledger separate from sampled operational traces.
- Own a stable event envelope; map it to OpenTelemetry, whose agent conventions remain **Development** as of 2026-08-30.

## Remaining research queue

- Workload-specific suites and graders for coding, browsing, research, support, infrastructure, and financial actions.
- Statistically efficient rare-event safety evaluation and adaptive red-team methodology.
- Production user/environment simulator validation and counterfactual replay.
- Online evaluator drift, privacy-preserving sampling, clustering, and human-review operations.
- Cross-platform trace export and migration once OpenTelemetry agent conventions stabilize.
