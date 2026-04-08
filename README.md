# Spec-Driven Development — a guide between fundamentals and harness

A guide, in 13 chapters, on how technical teams can adopt **Spec-Driven Development (SDD)** as an intermediate discipline between learning to use AI coding agents and engineering a full harness around them.

> The agent doesn't need a better prompt. It needs a clearer specification of intent — and a process that keeps that specification alive.

This is the second course in a trilogy:

1. **[Fundamentals](https://github.com/jmlopezdona/ai-coding-agents-fundamentals)** — what an agent is and how to use it.
2. **Spec-Driven Development** *(this guide)* — how to structure work so the agent can execute it reliably.
3. **[Harness Engineering](https://github.com/jmlopezdona/ai-coding-agents-harness)** — how to engineer the system around the agent at scale.

## 📖 Read the web version

**[https://jmlopezdona.github.io/ai-coding-agents-sdd/](https://jmlopezdona.github.io/ai-coding-agents-sdd/)**

Available in English and Spanish, built with MkDocs Material.

## 📁 Source

The 13 chapters live in [`docs/`](docs/), organized by language: [`docs/en/`](docs/en/) for English and [`docs/es/`](docs/es/) for Spanish.

- [Chapter 0 — Spec-Driven Development: building software with intent](docs/en/00-spec-driven-development.md)
- [Full guide index](docs/en/index.md)

## About the guide

Written for teams that already know how to drive an agent and have hit the wall: results are inconsistent, agents drift mid-session, refactors break things the agent never saw, and "just write a better prompt" stopped working a long time ago.

The guide synthesizes the emerging consensus around SDD — from the arXiv literature, Addy Osmani's spec writing patterns, the *living specs* framing from Augment, the GitHub Spec Kit / Kiro / Tessl / BMAD / Traycer toolchain, and the critical readings from Isoform and Martin Fowler — and presents it as a discipline, not a product. It also takes the critiques seriously: chapters 9 and 10 are deliberately uncomfortable.

## Sources

- [*Spec-Driven Development: From Code to Contract in the Age of AI Coding Assistants*](https://arxiv.org/html/2602.00180) — arXiv
- [*A Survey on Code Generation with LLM-based Agents*](https://arxiv.org/html/2508.00083v1) — arXiv 2508.00083
- Addy Osmani — [*How to write a good spec for AI agents*](https://addyosmani.com/blog/good-spec/)
- Augment Code — [*How to Write Living Specs for AI Agent Development*](https://www.augmentcode.com/guides/living-specs-for-ai-agent-development)
- Pockit Tools — [*Specification-Driven Development: How to Stop Vibe Coding*](https://dev.to/pockit_tools/specification-driven-development-how-to-stop-vibe-coding-and-actually-ship-production-ready-5788)
- r/vibecoding — [*Everything one should know about Spec-Driven Development*](https://www.reddit.com/r/vibecoding/comments/1qs80k4/everything_one_should_know_about_specdriven/)
- Isoform — [*The Limits of Spec-Driven Development*](https://isoform.ai/blog/the-limits-of-spec-driven-development)
- Martin Fowler / Thoughtworks — [*Understanding Spec-Driven Development: Kiro, spec-kit, and Tessl*](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html)

## License

[CC-BY-4.0](LICENSE). You can share and adapt the content freely, even commercially, as long as you keep attribution.
