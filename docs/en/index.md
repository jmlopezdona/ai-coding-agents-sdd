# Spec-Driven Development

A guide for technical teams that already know how to drive AI coding agents (Claude Code, Codex, Cursor, Copilot…) and have hit the wall: results are inconsistent, agents drift mid-session, refactors break files the agent never saw, and "write a better prompt" stopped working a long time ago.

This is the **second guide in a trilogy**: it comes after the [fundamentals guide](https://jmlopezdona.github.io/ai-coding-agents-fundamentals/) (what an agent is, how to use one) and before the [harness guide](https://jmlopezdona.github.io/ai-coding-agents-harness/) (how to engineer the system around the agent). SDD is the discipline in between: **how to structure work so the agent can execute it well**.

## Premises

- You can already drive an agent and have seen its limits in real projects.
- You're not looking for a magic product. You're looking for a discipline.
- You're willing to pay the cost of writing specs if it avoids the larger cost of rework, drift and invisible debt.
- You want judgment, not dogma: you care about when SDD helps and when it's bureaucracy.

## Central thesis

> The bottleneck is no longer writing code. It is **specifying intent** clearly enough for an agent to execute it — and keeping that specification alive as the code evolves.

## How to read this guide

Chapters are ordered **problem → foundations → cycle → tooling → practice → critique → bridge**, but each one stands on its own. If you already have the "why", jump to whichever chapter solves a concrete problem. If you're starting fresh, read in order.

Two chapters are **not optional** even though they look negative: chapter 10 (the honest critique) and chapter 11 (context engineering as alternative). Without them, this course is marketing. With them, it gives judgment.

## Index

### Chapter 0 — Entry point

- [Spec-Driven Development: building software with intent](00-spec-driven-development.md)

### Part 1 — The problem

- [1. From vibe coding to SDD: context collapse and context drift](01-from-vibe-coding-to-sdd.md)

### Part 2 — Foundations

- [2. The specification spectrum](02-specification-spectrum.md)
- [3. Anatomy of a good specification](03-anatomy-of-a-spec.md)
- [4. The spec in context](04-spec-in-context.md)

### Part 3 — Lifecycle

- [5. The SDD lifecycles: two approaches and when to use each](05-sdd-lifecycles.md)
- [6. Living specs: the bidirectional loop](06-living-specs.md)

### Part 4 — Tooling

- [7. Native SDD tools: Kiro, Spec-kit, Tessl, BMAD](07-native-sdd-tools.md)
- [8. Architect layers over agents: Traycer and the wrapper pattern](08-architect-layers.md)

### Part 5 — Practice

- [9. Patterns of application: features, refactors, bug fixes](09-patterns-of-application.md)

### Part 6 — Critique

- [10. The honest critique: maintenance tax, MDD and the illusion of control](10-honest-critique.md)
- [11. Context engineering as alternative](11-context-engineering.md)

### Part 7 — Common mistakes

- [12. SDD anti-patterns](12-anti-patterns.md)

### Part 8 — Industrialization

- [13. From SDD to the harness: how the pieces fit](13-from-sdd-to-harness.md)

## What you *won't* find in this version

- Exhaustive tool comparisons. The ecosystem turns over every quarter; chapter 7 is a map, not a buyer's guide.
- Production-ready spec templates. There are skeletons in chapters 3 and 9, but useful specs are written against your domain, not against a generic template.
- Productivity promises. If someone promises you 10×, they probably haven't read chapter 10.

## Sources

- [*Spec-Driven Development: From Code to Contract in the Age of AI Coding Assistants*](https://arxiv.org/html/2602.00180) — arXiv
- [*A Survey on Code Generation with LLM-based Agents*](https://arxiv.org/html/2508.00083v1) — arXiv 2508.00083
- Addy Osmani — [*How to write a good spec for AI agents*](https://addyosmani.com/blog/good-spec/)
- Augment Code — [*How to Write Living Specs for AI Agent Development*](https://www.augmentcode.com/guides/living-specs-for-ai-agent-development)
- Pockit Tools — [*Specification-Driven Development: How to Stop Vibe Coding*](https://dev.to/pockit_tools/specification-driven-development-how-to-stop-vibe-coding-and-actually-ship-production-ready-5788)
- r/vibecoding — [*Everything one should know about Spec-Driven Development*](https://www.reddit.com/r/vibecoding/comments/1qs80k4/everything_one_should_know_about_specdriven/)
- Isoform — [*The Limits of Spec-Driven Development*](https://isoform.ai/blog/the-limits-of-spec-driven-development)
- Martin Fowler / Thoughtworks — [*Understanding Spec-Driven Development: Kiro, spec-kit, and Tessl*](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html)
