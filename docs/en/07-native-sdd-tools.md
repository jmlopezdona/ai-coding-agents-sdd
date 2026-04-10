# 7. Native SDD tools: Kiro, Spec-kit, Tessl and BMAD

There are a handful of tools explicitly proposing themselves as infrastructure for doing Spec-Driven Development. We call them **native** because the spec is the center of their workflow, not an add-on. In this chapter we walk through the four most visible today — Kiro, Spec-kit, Tessl and BMAD — situating each in the chapter 2 spectrum and being honest about where each fits and where each breaks.

> **Important note:** this chapter will age fast. The SDD ecosystem is in full ferment and what's described here is the state of the art in early 2026. Treat it as a map, not as a buyer's guide.

## Fowler's warning

Before going tool by tool, it's worth keeping in mind the conclusion [Martin Fowler](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html) reaches after evaluating the three main ones:

> *Neither of them is suitable for the majority of real life coding problems.*

It's not reverse marketing. It's the honest observation that current tools all suffer from the same problem — *workflow mismatch*: they apply a one-size-fits-all process to problems of radically different sizes. A three-line bug fix treated as a multi-story feature is bureaucracy. A critical migration treated as a regular commit is negligence. Today's tools don't distinguish well between the two extremes, and that's their common limitation.

With that in front, we go one by one.

## Kiro

Kiro is AWS's agentic IDE. Its workflow is probably the simplest of the four: three markdown documents — **Requirements → Design → Tasks** — generated in order to guide the agent through the cycle. It maps almost exactly to chapter 5's "variant B".

**What it does well:**

- The flow is legible at first glance. An outsider understands the process in five minutes.
- It has **hooks that fire on file save** to run background tasks (update docs, regenerate tests, validate against spec). This is relevant because it connects directly to one of the central themes of the harness course.
- It's pragmatic. It doesn't try to force you into Spec-as-Source. It lives comfortably in spec-anchored.

**What Fowler flags as problematic:**

- **Excessive for small bug fixes.** It treats every change as a multi-story feature, which for fixing a typo is absurd. If your day-to-day is many small changes, Kiro will frustrate you.
- The rigid structure doesn't adapt to problem size.

**Who it fits:**

- Teams with medium-large features where the three-document ritual amortizes its cost.
- Teams wanting a soft onramp to SDD without rewriting their tooling.

## Spec-kit (GitHub)

Spec-kit is GitHub's official toolkit for Spec-Driven Development. It's a CLI and a set of templates introducing *checkpoints* at each stage of the process. Its distinctive feature is the **constitution**: a document of fundamental project rules living above individual specs that the agent reads before any task.

**What it does well:**

- The constitution idea is good: it separates project invariants (chapter 3 "boundaries") from feature-specific things.
- It uses explicit checklists forcing the agent not to skip steps.
- It's open source and integrates naturally with GitHub flows (PRs, issues, Actions).

**What Fowler flags as problematic:**

- **Aspires to spec-anchored but in practice is spec-first.** It generates many interconnected files, but the automatic mechanism for keeping spec and code synchronized isn't really implemented. It's spec-first in disguise.
- **Generates verbose, repetitive markdown.** Fowler reports that in his evaluation, the generated files were *heavier to review than the code itself* would have been. This is exactly what critics predict from bad SDD: review tax goes up instead of down.

**Who it fits:**

- Teams deeply integrated in GitHub wanting the ergonomics and not afraid of the markdown review cost.
- Teams wanting to try the "constitution" idea as a global boundaries layer.

## Tessl

Tessl is the most radical proposal of the four. It aims directly at the **Spec-as-Source** level of the spectrum: the spec is the only source humans edit, code is generated, generated files carry a `// GENERATED FROM SPEC - DO NOT EDIT` mark. If the code is wrong, you fix the spec and regenerate.

**What it does well:**

- It's the only tool of the four really pursuing spec-as-source. If that's the strategic direction you believe in, you don't have many alternatives.
- It forces a clarity in the spec the others don't demand, because the spec **is** the system, not a description of it.

**What Fowler flags as problematic:**

- **It's in private beta**, with all the limitations that implies (access, support, risk of disruptive changes, risk of disappearing).
- **LLM non-determinism conflicts directly with the "regenerate and you get the same thing" promise**. If two generations of the same spec produce different code, the "single source of truth" promise deflates. This is almost exactly the problem that killed Model-Driven Development in the 2000s and that chapter 10 develops.

**Who it fits:**

- Deliberate experimentation on isolated, small surfaces. Not a tool to bet your main repo on in 2026.
- Teams curious about where SDD might go long term.

## BMAD

BMAD is the most distinct case of the four, which is why I left it for last. Instead of focusing on the spec as artifact, BMAD deploys a **team of specialized role agents** — Product Manager, Architect, QA, Developer — that manage the full agile cycle while keeping consistent context between them. It's Spec-Driven Development implemented as a multi-agent system.

**What it does well:**

- Role specialization helps with chapter 3's *curse of instructions*. Each agent only sees the instructions relevant to its role, instead of a megaprompt mixing everything.
- It fits what the academic arXiv survey ([*A Survey on Code Generation with LLM-based Agents*](https://arxiv.org/html/2508.00083v1)) describes as the family of **multi-agent role-playing systems** (alongside ChatDev and MetaGPT on the more academic side).
- For teams already thinking in agile-role terms, the mental model translation is trivial.

**What to look at carefully:**

- Operational complexity is high. Coordinating multiple role agents is a miniature harness, and coordination failures between agents are subtle and hard to debug.
- The multi-agent promise is over-represented in marketing and under-represented in empirical evidence outside of demos.

**Who it fits:**

- Teams with experimental appetite and mature agile processes that can absorb the complexity.
- Cases where role separation of concerns adds more than the simplicity of a single well-directed agent.

## How to choose (or not choose)

An honest recommendation, knowing it'll age:

1. **If you've never done SDD**, don't pick a tool yet. Start with pure spec-first, written by hand using the chapter 3 template and a general-purpose agent. Learn what parts of the process hurt *before* shopping for a tool that solves them.

2. **If you've done specs by hand and know what's missing**, evaluate Kiro or Spec-kit depending on whether rigidity (Kiro) or verbosity (Spec-kit) bothers you more.

3. **If you're interested in pushing the frontier**, try Tessl on an isolated surface or build a proof-of-concept with BMAD. Don't put them on the critical path.

4. **If your real problem is the *blast radius* of changes and not process structure**, no tool in this chapter will solve it. What you need lives in the next chapter: an **architect layer** over your current agent.

## What none of these tools solve

There's a pattern worth naming that none of the four tools solves on its own: **post-code verification**. The four help you structure spec and plan; none verifies with sufficient rigor that the generated code **meets** the original spec. That verification, when done, is typically another round of manual prompting.

That's exactly the gap where Traycer (chapter 8) and other tools in the "architect layer" pattern have positioned themselves.

## What comes next

Chapter 8 covers the **other category** of SDD tools: the ones that don't replace your agent but wrap it. It's an important distinction because it answers a different question. The tools in this chapter answer "which tool do I use?". The next chapter's answer "how do I improve the tool I'm already using?".
