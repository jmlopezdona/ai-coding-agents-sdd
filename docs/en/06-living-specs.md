# 6. Living specs: the bidirectional loop

The problem that kills most Spec-Driven Development initiatives isn't writing the specs. It's what happens **after**. Specs age, code leaves them behind, and three months later you have a folder of markdowns describing a system that no longer exists. That's exactly the pattern [Augment Code](https://www.augmentcode.com/guides/living-specs-for-ai-agent-development) calls the **spec gap**, and this whole chapter is about how to avoid it.

## The unidirectional flow problem

A static spec works like this:

1. The human writes the spec.
2. The agent reads it and produces code.
3. Code evolves, gets modified, gets refactored.
4. The spec stays where it was.

Information flows in one direction: spec → code. And that means any decision made during implementation — and many always are: trade-offs, discoveries, edge cases that show up while typing — **doesn't go back to the spec**. The spec describes initial intent; the code describes current reality; between them grows a crack with every commit.

[Augment](https://www.augmentcode.com/guides/living-specs-for-ai-agent-development) puts it with uncomfortable precision:

> *Gaps in the specification widen with direct code changes and keep resurfacing because AI generation is non-deterministic.*

That is: **the problem isn't only that the spec ages, it's that LLM generation is non-deterministic**, so every time you regenerate code from an out-of-date spec, you introduce new inconsistencies. The gap doesn't just grow — each regeneration cycle amplifies it.

## What a living spec is

A living spec is a spec that's updated as the code changes. That's the short definition. The operative definition is more demanding:

> A living spec has a **bidirectional feedback loop**: decisions made during implementation are written back into the spec, so the spec always describes the system's current state, not just its initial desired state.

The difference from a static spec isn't the document's first version. The two can start identical. The difference is what happens on day 30, day 90 and day 365.

## The four phases of the living loop

Augment's framework breaks the bidirectional loop into four phases:

### Phase 1 — Specify initial intent

Identical to phase 1 of the chapter 5 lifecycle. A minimum viable spec: objective, constraints, criteria, no-goals, whys. Nothing new.

### Phase 2 — Implement against the spec

The agent executes tasks guided by the spec. It makes tactical decisions the spec didn't anticipate: which library to use for a utility, how to name a variable, what pattern for an edge case. The spec didn't dictate them, so the agent invents them.

### Phase 3 — Bidirectional update (the key phase)

This is the phase that defines a living spec and the one almost every team forgets. When a task ends, **someone — agent or human — writes into the spec what was actually decided during implementation**. Not "what the spec said" but "what the code ended up doing and why". Tactical decisions get promoted and recorded.

Augment says exactly this:

> *Agents or developers update the spec to reflect what was actually built.*

The practical consequence: the spec stops being a plan and starts being a **description**. It still captures intent, but it also captures the decisions made while realizing that intent. The crack between intent and reality closes.

### Phase 4 — Production feedback

This is the phase Augment adds compared to almost any other description of the SDD process, and it's brilliant. Once the code is in production, **metrics, incidents and operational learnings also get written back into the spec**. If a spec assumption turns out to be false in production ("we thought images would average less than 5 MB, in reality they're 12"), that goes back to the spec as a finding. If an incident reveals an edge case the spec didn't contemplate, that case is added.

The spec, in this model, is a **document that learns**. It starts describing intent, passes through implementation decisions, and ends containing the scars of what reality has taught the system.

## Why phase 4 matters more than it seems

One of the strongest critiques of SDD (chapter 10, [Isoform](https://isoform.ai/blog/the-limits-of-spec-driven-development)) is that specs **lose real context** because they only capture "what we were going to do", not "what we learned doing it". Phase 4 of living specs is the direct answer to that critique. If your spec records production feedback, real context **isn't lost** — it accumulates.

It's the difference between a design doc and an operations manual. And for an agent arriving six months later to make a change, reading the spec with production feedback is radically more useful than reading the original spec.

## Mechanisms so the loop doesn't die

The reason static specs win isn't ideological — it's entropic. Maintaining a living spec requires discipline, and discipline without mechanism evaporates. Here are the mechanisms that actually work:

- **Make the update part of the definition of "done"**. A task isn't finished until the spec reflects what was done. This goes in the PR checklist, not in informal "would be nice if...".
- **Use recurring agents that detect drift**. An agent that nightly compares specs and code and opens issues when there are divergences. That's harness work, not strictly SDD, which is why this point gets developed in the next course.
- **Treat implementation decisions as mini-PRs to the spec**. Every non-trivial commit asks: does this change anything the spec assumes? If yes, the commit includes a spec entry.
- **Make the spec the first place the agent looks**. If your workflow takes the agent to the code before the spec, the agent will never learn to update the spec. If the spec is always the first document loaded, the loop closes.

None of these mechanisms is free, and that's where SDD's real cost lives. Chapter 10 develops this cost honestly — the famous *maintenance tax* — and why it's not negligible.

## What distinguishes a healthy living spec from a dead one

Quick test. Look at the last significant modification of your spec. Compare it to the last modification of the code the spec describes. If the difference is hours or days, your spec is alive. If it's weeks or months, your spec is dead and nobody has told you yet.

Another test: when a new dev joins the team, do you send them to read the specs or to read the code? If you send them to the code and warn "the specs are out of date, don't trust them", the answer is clear — you've officially accepted the specs are no longer useful. And from that moment on they're worse than useless, because they generate noise without signal.

## The right granularity for updates

A practical question that comes up early: do you have to update the spec with every change? The honest answer is no. There are two kinds of code changes:

- **Changes the spec already covered** (internal refactor, optimization, typo fix). They don't touch the spec.
- **Changes the spec didn't cover or that change a spec assumption** (new decision, edge case discovered, trade-off chosen). They do touch the spec.

The distinction isn't always sharp, and that's where the engineer's judgment matters. But the practical rule is: **if six months from now someone reading the spec would be surprised at what the code does, the spec needs that update now**. If not, no.

## What comes next

Up to here we've talked about process. In **Chapter 7** we land on concrete tools: the four native SDD tools defining the state of the art today — **Kiro, Spec-kit, Tessl and BMAD** — what they do well, what they do badly, and what kind of team each fits. And in chapter 8, a different category: tools that sit **on top** of your current agent, like Traycer.
