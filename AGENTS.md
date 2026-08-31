# AGENT.md

## Working Rules

This repository is a **Markdown-only AI Agent Engineering knowledge base**.

Follow these rules for every task.

### 1. Research Before Writing

- Never write substantial documentation from assumptions or model memory alone.
- Perform **deep internet research first**.
- Research as broadly and deeply as the topic reasonably requires.
- Prefer:
  1. Official documentation
  2. Official repositories and source code
  3. Release notes and changelogs
  4. GitHub issues/discussions
  5. Engineering blogs and production case studies
  6. Research papers
  7. High-quality real-world engineering discussions
- Cross-check important claims across multiple reliable sources.
- Check current versions and dates. Do not present outdated behavior as current.
- Keep researching until additional research stops producing materially useful information.

### 2. Markdown Only

- Repository content must remain primarily `.md` documentation.
- Do not build applications, libraries, demos, scripts, or unrelated code.
- Small pseudocode or snippets are allowed only when they materially improve explanation.

### 3. Keep Files and Folders Clean

- Inspect the existing repository structure before creating anything.
- Place every document in the most appropriate existing section.
- Create a new folder only when the topic has enough depth to justify one.
- Large topics should be split into focused files instead of one giant Markdown dump.
- Do not create useless micro-files with only a small amount of information.
- Avoid duplicate documentation. Link to an existing canonical guide instead.
- Keep file and folder names clear, predictable, and consistent.
- Update relevant README/index/navigation files whenever structure changes.

### 4. Write Premium Documentation

Every document should feel like a **high-quality engineering reference**, not generic AI-generated content.

Documentation must be:

- technically accurate;
- deeply researched;
- production-focused;
- practical;
- concise where possible but detailed where necessary;
- free from filler and repeated explanations;
- clear about trade-offs, limitations, and failure modes.

Explain not only **what** something is, but also:

- how it works;
- why it exists;
- when to use it;
- when not to use it;
- production best practices;
- common mistakes;
- failure modes;
- security implications;
- performance/cost implications;
- alternatives and trade-offs.

### 5. Make Everything Easy to Understand

Use simple, clear technical English.

Do not unnecessarily complicate explanations with academic or vague language.

For complex topics, prefer this flow:

**Concept → Visual → How It Works → Production Guidance → Trade-offs → Failures → Checklist**

A new reader should understand the concept, while an experienced engineer should still gain useful implementation and architectural insight.

### 6. Use Visual Documentation

Use Markdown visually, not as walls of text.

Use when useful:

- Mermaid diagrams
- Architecture diagrams
- Flowcharts
- Sequence diagrams
- State diagrams
- Decision trees
- Tables
- Comparison matrices
- Checklists
- Failure matrices
- Short callouts

Use Mermaid whenever a flow, lifecycle, architecture, relationship, decision, or state transition is easier to understand visually.

Diagrams must explain something useful — never add decorative diagrams.

### 7. Be Technology-Specific

When documenting an SDK, framework, harness, provider, or programming language:

- research that technology specifically;
- read its own official documentation and ecosystem;
- document its real behavior and limitations;
- do not copy generic agent advice and rename the heading.

If a technology is large enough, create a dedicated folder and multiple focused guides.

Do not artificially limit important technologies to one overview file.

### 8. Production Reality Over Hype

Prefer proven engineering practices over fashionable architecture.

Never assume:

- more agents are better;
- more memory is better;
- more frameworks are better;
- more autonomy is better;
- a vendor claim is automatically true.

Always consider simplicity, reliability, maintainability, scalability, security, latency, and cost.

Clearly label experimental or emerging techniques as such.

### 9. Sources Matter

Every deeply researched guide should keep useful references to the strongest sources used.

Prefer primary sources.

Do not copy documentation verbatim. **Research, understand, validate, and synthesize.**

### 10. Quality Check Before Finishing

Before completing any documentation task, verify:

- Was enough research performed?
- Is the information current?
- Are important claims supported?
- Is anything duplicated?
- Is the folder/file placement correct?
- Are important edge cases covered?
- Are production failures covered?
- Are trade-offs clear?
- Could a useful Mermaid diagram improve understanding?
- Is the document easy to scan and understand?
- Are related guides linked?
- Were indexes updated if needed?
- Does this document genuinely add value?

If the answer to an important item is no, improve the work before considering the task complete.

## Core Principle

**Research deeply → understand correctly → organize cleanly → explain simply → visualize clearly → document production reality.**

Quality is more important than speed or file count.


For every major topic, perform **deep and broad internet research before writing**. Do not rely only on your existing knowledge or the first few search results. Search extensively across official documentation, SDK/framework repositories, GitHub discussions and issues, engineering blogs, architecture guides, research papers, production case studies, conference talks, benchmarks, security guidance, experienced engineer write-ups, and high-quality community discussions when they provide real practical insight. Cross-check important claims across multiple sources and always prefer primary/official sources where available.

The goal is not to summarize individual websites; the goal is to **extract the strongest useful knowledge available on the internet and synthesize it into a better guide than any single source provides**. Look for hidden implementation details, production lessons, edge cases, failure modes, trade-offs, scaling concerns, performance behavior, cost implications, security risks, known limitations, common mistakes, and patterns used by serious production systems. Research competing approaches as well, so the playbook explains not only the recommended pattern but also why alternatives may be weaker or better in specific situations.

Do as much high-quality research as the topic reasonably requires. Do not stop because you already found enough information to produce a basic article. Continue until additional research stops revealing materially useful insights. For large topics, research them from multiple angles and multiple search queries. Treat official documentation as the foundation, but supplement it with real-world engineering experience because documentation often explains how features work without fully explaining how they fail at scale or in production.

Be highly selective while synthesizing. The internet contains outdated, duplicated, incorrect, hype-driven, and low-quality agent advice; do not blindly include it. Validate ideas, compare publication/update dates, identify deprecated approaches, distinguish experimental techniques from established production practices, and discard weak information. Where experts or frameworks disagree, document the disagreement and explain the trade-off instead of pretending there is one universal answer.

Whenever possible, research the **latest available version and current best practices at the time the document is written**, and record useful sources and research dates so future agents can refresh stale guides. If a topic has evolved significantly, explain what older approaches have been replaced and why.

Think of this repository as a **curated compression of the best agent-engineering knowledge available on the internet**: thousands of pages of documentation, discussions, papers, repositories, and production lessons distilled into structured, accurate, practical Markdown guides. Research depth should be limited by usefulness, not by an arbitrary number of searches, sources, or documents.
