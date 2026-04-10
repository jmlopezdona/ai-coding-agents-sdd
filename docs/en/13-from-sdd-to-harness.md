# 13. From SDD to the harness: how the pieces fit

If you've made it this far, you have SDD as a discipline: you know how to specify intent, maintain the bidirectional loop, modulate the process by task type, and you have judgment to not confuse it with bureaucracy. What's missing is understanding **where this connects** with the trilogy's next layer: the **harness**, the internal infrastructure system that surrounds the agent so the discipline scales to a whole team and to months at a time.

This chapter is the bridge. It doesn't repeat the harness course content — that's a whole other repository — but it points precisely at the five exact points where SDD couples with harness, and why the combination produces something better than the two separately.

## The bridge thesis

> SDD tells you **what** to specify and how. The harness tells you **where** that specification lives, **who** reads it, **when** it gets validated, and **how** it stays alive without sustained human discipline.

SDD without harness is ceremony: it depends on every team member applying perfect discipline, and perfect discipline doesn't scale. Harness without SDD is purposeless infrastructure: you have sandboxes and sensors and loops, but the agent doesn't know what to build because intent was never captured. Both together form a system; both apart are loose pieces.

## Coupling point 1 — Specs as persistent context

The harness's first pillar, per the corresponding course, is **context as system of record**: the idea that everything the agent should know has to be materialized in the repo, not in Slack or in heads. SDD specs are **a specialization of that principle**.

A well-written spec is persistent context with a specific shape: *intent + constraints + whys + verifiable criteria*. The harness contributes the general discipline ("context lives in the repo"); SDD contributes the concrete shape of one of the artifacts that lives in it.

The operating consequence: when a team builds a harness, specs aren't just another folder — they're one of the central pieces of context. And when that same team applies SDD, the harness ensures specs reach the agent in the right form, at the right moment, with the right hooks.

## Coupling point 2 — Sensors that validate specs

The harness's second pillar is the duality **guides and sensors**: artifacts that orient the agent *before* acting (guides) and verifiers that detect when the agent has drifted *after* (sensors). The chapter 5 SDD cycle's phase 5 — verification — is exactly a kind of sensor.

Without harness, that phase 5 is manual or, at best, an ad-hoc script someone runs. With harness, it becomes infrastructure: tests comparing code against spec, recurring agents detecting drift, linters validating that spec constraints are still respected.

This is the technical way to close the chapter 6 bidirectional loop. Without harness, the loop depends on human discipline ("remember to update the spec"). With harness, the loop is automatic ("the sensor detected code and spec diverged and opened an issue to reconcile them").

## Coupling point 3 — Hooks that keep specs alive

The chapter 7 Kiro detail — the **hooks that fire on file save** — isn't a detail. It's the prefiguration of one of the central harness mechanisms: **repo events that trigger agent actions** (regenerate docs, update indexes, validate invariants, refresh the living-specs loop).

When a Kiro-style tool has those hooks, it's implementing a micro-piece of harness inside its own product. When you build your full harness, you have hooks for any event, and the spec maintenance axes that require human discipline in pure SDD become machine-triggered in SDD + harness.

The operating rule: **anything that in pure SDD requires "remember to do X when Y happens", in SDD + harness should become an automatic hook**. If something stays in "remember", it'll be forgotten.

## Coupling point 4 — Architect layers as harness prototype

The chapter 8 tools — Traycer and similar — are technically **harness applied to SDD**. What they do — intercepting inputs, planning, verifying outputs — is exactly the harness guides-and-sensors dynamic, applied to the concrete cycle of a session with an agent.

If the full harness seems too much to start with, an architect layer over your agent is the miniature version of the idea: try what it feels like to have guides and sensors wrapping the agent, before committing to building your own harness. And when you're ready for the full harness, you'll discover Traycer's logic is a particular case of the general pattern — because it is.

## Coupling point 5 — Anti-patterns that only the harness solves

Several chapter 12 anti-patterns are specifically hard to avoid without harness:

- **Spec-as-Theatre** dies when there's an automatic sensor measuring adherence: if the spec isn't respected, the harness says so.
- **The eternal spec** dies when there's a recurring agent detecting drift and opening automatic issues.
- **Trusting the spec more than the code** becomes rare when there are automatic validators that on disagreement force you to investigate before accepting the merge.
- **The review burden** gets reduced when hooks generate the repetitive markdowns automatically and human attention is freed for what contributes judgment.

In other words: **a significant part of what kills bad SDD is exactly what the harness is designed to solve**. The two courses aren't sequential by whim — they're sequential because the problems you discover doing the discipline well are the problems harness addresses.

## When to take the next step

Not every team adopting SDD needs to jump to the harness immediately. There are specific signals you're ready for the transition:

- **The team applies SDD with discipline but sustainability depends on individuals.** If Alice goes on vacation for a week and specs go out of date, you need mechanism, not more discipline.
- **The chapter 6 bidirectional loop works when someone takes care of it and breaks when nobody looks.** That's a sign it needs to go from human process to infrastructure.
- **Chapter 12 anti-patterns reappear periodically** even though the team knows them. Individual awareness doesn't scale; infrastructure does.
- **Chapter 7 tools fall short because they're single-dimension** and your system needs multiple layers (sensors + hooks + sandboxes + observability). That's a full harness, not one more tool.

If you recognize two or more, the trilogy's next course is your natural next step.

## What the harness will teach you (in short preview)

To make the bridge concrete, here's a very condensed idea of what the harness course develops and how it connects to what you've learned here:

- **Guides and sensors as two pillars.** Harness ch. 3 develops this duality. Specs are guides; validators and recurring agents are sensors.
- **The loop as primitive.** Harness ch. 4 presents "iteration as primitive". This course's ch. 5 cycle is a particular case.
- **Isolated, reproducible environments.** Harness ch. 5 talks about disposable sandboxes. Here you improve the reliability of SDD's implementation phase.
- **Context as system of record.** Harness ch. 6 is where your specs fit as one of the central artifacts.
- **Architecture for agents.** Harness ch. 7 teaches you to design the repo so the agent can navigate it. Specs only work when the agent can find them.

The progression is natural and worth doing in order: first this guide for the discipline, then the harness for the infrastructure.

## A goodbye without promises

An honest guide doesn't end with "and now your life will change". It ends with two simple truths:

First, **well-done SDD improves the agent's output and the project's sustainability**. This is documented in the practice of teams that have applied it for a while and shows in concrete metrics: less rework, less drift, better onboarding. It's not magic, but it's real.

Second, **bad SDD is indistinguishable from bureaucracy and sometimes worse**. This is also documented and is why chapter 10 exists. Applying it without judgment turns the discipline into its own bottleneck, and within weeks the team goes back to vibe coding with an additional layer of cynicism.

The difference between the two isn't the tool. It's the team's judgment to modulate the formality level by problem, keep whys alive, read honest critiques as part of the course, and understand that SDD is **one** strategy within context engineering, not the only one.

If you take that, you take the best this guide can offer.

---

*This chapter closes the course. If you want to take the next step, read the [harness guide](https://jmlopezdona.github.io/ai-coding-agents-harness/). And if you're missing the foundations on what an agent is and how to drive one, the [fundamentals guide](https://jmlopezdona.github.io/ai-coding-agents-fundamentals/) is the natural starting point of the trilogy. All chapters here are self-contained: jump to whichever one you need when the problem it solves shows up.*
