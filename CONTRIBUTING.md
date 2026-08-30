# Contributing Research-Grade Documentation

**Last updated:** 2026-08-30

This repository accepts documentation and navigation only. Do not add executable applications, libraries, demos, generated project scaffolds, or non-Markdown files.

## Definition of done for a major guide

A guide is not ready merely because it is accurate at a high level.

- [ ] Its question and non-goals are explicit.
- [ ] A linked research packet records the search date, source mix, findings, disagreements, and discarded claims.
- [ ] Official documentation and source repositories establish current mechanics.
- [ ] Engineering reports, issues, case studies, or benchmarks expose real failure behavior.
- [ ] Competing approaches are researched on their own terms.
- [ ] Time-sensitive claims are dated and directly sourced.
- [ ] Vendor feature claims are labeled as vendor claims unless independently supported.
- [ ] Stable practice, conditional practice, and experimental technique are separated.
- [ ] The guide covers failure, cost, scaling, security, operations, and change signals where relevant.
- [ ] Diagrams clarify actual control or data flow.
- [ ] Related guides and indexes link both ways.
- [ ] Links and Mermaid blocks pass repository validation.

## Source hierarchy

| Tier | Typical sources | Use |
|---|---|---|
| 1 — Normative/primary | Specifications, official docs, source code, release notes, standards, peer-reviewed papers | Establish current behavior and definitions |
| 2 — Direct production evidence | Engineering postmortems, architecture write-ups, maintainers' design discussions, reproducible benchmarks | Establish operational lessons and trade-offs |
| 3 — Corroborating field evidence | Detailed issues, conference talks, experienced engineering reports, high-quality discussions | Find edge cases and test official claims |
| 4 — Discovery only | Aggregators, unsourced tutorials, marketing comparisons, generic posts | Find leads; do not anchor important claims |

Popularity is not evidence. A GitHub issue is evidence that a reported case exists, not proof of universal behavior. A benchmark score is evidence only for its tested model, scaffold, data, metric, and date.

## Required research packet

Create `docs/research/packets/<topic>.md` before or alongside a major guide. Record:

1. research questions and scope;
2. queries and source families searched;
3. sources actually used, with access/research date;
4. source version or publication date when useful;
5. claims supported and important caveats;
6. disagreements and their likely causes;
7. rejected or superseded ideas;
8. open questions and refresh triggers;
9. guides supported by the packet.

Do not set an arbitrary source count. Stop when new credible sources cease changing the design guidance, not when a quota is reached.

## Writing pattern

Prefer this order when it fits:

1. concept and scope;
2. meaningful visual model;
3. detailed mechanics;
4. choices and trade-offs;
5. failure and recovery;
6. production guidance and checklist;
7. related guides and research notes.

Use canonical guides for shared concepts. Framework pages should link to the canonical guide and explain only the framework-specific implementation, deviation, or trap.

## Maturity and freshness

Every substantial guide should begin with a compact metadata callout:

```markdown
> **Status:** Research-backed draft  
> **Last researched:** YYYY-MM-DD  
> **Scope:** What this guide does and does not establish  
> **Research packet:** Link to the corresponding packet under `docs/research/packets/`
```

Suggested refresh triggers include major version changes, deprecated APIs, changed protocol revisions, new security guidance, benchmark contamination, new runtime guarantees, or a production incident that invalidates a recommendation.

## Prohibited patterns

- Feature-table comparisons that ignore execution semantics.
- “Best framework” conclusions without workload and operational criteria.
- Treating model output validation as authorization.
- Treating checkpoint storage as exactly-once effects.
- Presenting hidden reasoning or chain-of-thought as a required observable interface.
- Large copied passages or lightly rewritten official docs.
- Placeholder guides with generic prose.
- Decorative Mermaid diagrams that do not teach a relationship or flow.
