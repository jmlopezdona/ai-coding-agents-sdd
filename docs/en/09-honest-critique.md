# 9. The honest critique: maintenance tax, MDD and the illusion of control

So far the course has been constructive: why SDD is needed, how to do it, with what tools. This chapter isn't. This chapter is the counterweight, and it exists because any guide that only teaches you the positive patterns leaves you defenseless against the moment — and it will come — when your own process becomes the bottleneck.

The critiques we'll collect aren't marginal objections. They come from two sources worth reading whole: **[Isoform](https://isoform.ai/blog/the-limits-of-spec-driven-development)** (*The Limits of Spec-Driven Development*) and **[Martin Fowler / Thoughtworks](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html)** (*Understanding Spec-Driven Development: Kiro, spec-kit, and Tessl*). Both are written by people who tried SDD seriously and decided to publish what they saw.

## Critique 1 — The maintenance tax

Isoform's most devastating critique is that detailed specs create **documentation debt disguised as engineering discipline**.

> *Comprehensive specifications create documentation debt disguised as engineering discipline.*

The logic is direct. When the system has a living spec (chapter 6), every change in the code asks for a change in the spec. When requirements change — and they always change — you need coordinated changes in both places. Cost rises. And unlike the maintenance cost of code, which is visible and discussed in sprint reviews, the spec maintenance cost is invisible: it doesn't appear in any burndown, nobody counts it, and it accumulates until one day the team realizes it's spending an uncomfortable proportion of its time maintaining documentation of a system instead of evolving it.

Isoform puts it this way: SDD doesn't reduce overhead, it *displaces* it. What used to be undocumented code is now documented code, yes, but the cost doesn't disappear — it just changes location. And sometimes that new location is worse, because the spec has become a second system to maintain, with its own entropy, its own bugs (internally contradictory specs) and its own cognitive load.

**How to defend yourself:** measure the cost. Honestly, during a sprint, log how many hours the team spent updating specs and how many evolving code. If the ratio rises above 20% and the spec is still out of date, **something is wrong with your process**. Either specs are too detailed, or the chapter 6 loop isn't calibrated, or you're applying full spec to places that didn't need it (chapter 9). Any of the three, stop and redesign.

## Critique 2 — The loss of the *whys*

This is the critique most worth taking seriously because it strikes at the heart of SDD.

> *Specs describe what systems should do, but cannot capture why decisions were made. The missing context is where the real problems show up.*

Isoform argues typical specs focus too much on the **hows** — schemas, signatures, contracts, criteria — and fall short on the **whys**: assumptions, the real-world constraints that led to a decision, the trade-offs chosen, and especially the ones rejected and why.

And whys are exactly what an agent (or a new human) needs six months later to make smart decisions in the gaps the spec doesn't cover. A spec without whys ages at the speed of the decisions around it, because it loses the ability to explain itself.

Chapter 3 already tries to shield against this by making whys a mandatory section of the template. But the shield is only process; in practice, whys are the section most easily skipped when the team is in a hurry, and the one worst updated by the chapter 6 bidirectional loop. If you have to cut spec maintenance time, almost everyone cuts whys first — and that's exactly the worst thing you can cut.

**How to defend yourself:** treat whys as tier 0 of the spec. If a spec has no whys, it's an incomplete spec. If the chapter 6 bidirectional loop doesn't update whys, it's not really bidirectional — it's half-loop.

## Critique 3 — The false illusion of completeness

> *Detailed specifications create misleading confidence.*

This is subtle and very real. A detailed spec looks exhaustive. When you read an 800-line spec with diagrams, criteria and templates, your brain assumes the system is well thought out. The spec works as **internal social proof**: "someone sat down to think this through, so it must be right".

The trap is that detail ≠ correctness. A spec can be detailed and wrong. A spec can be internally consistent and not match reality. A spec can cover the 47 cases you thought of and not cover the 48th that only shows up in production.

And worse: a detailed spec **inhibits creative iteration**. When there are 800 lines everyone accepted, changing course becomes a social move, not a technical one. The spec becomes a cultural brake. Isoform cites this as "reinforcing waterfall-like thinking", and the coincidence with waterfall isn't accidental: the more complete the spec before starting, the more the process resembles waterfall, with all the problems waterfall had.

**How to defend yourself:** suspect your own confidence in a dense spec. If nobody has questioned a spec decision in two sprints, it's not a sign of quality — it's a sign the spec has become an obstacle to questioning it. Periodically ask yourself: *if I rewrote this spec today, from scratch, would it come out the same?*

## Critique 4 — The MDD parallel

This is Fowler's critique, and the one most worth understanding by historical context. In the 2000s, there was a wave of **Model-Driven Development** (MDD) that promised exactly what SDD promises today: you model, tools generate, single source of truth, magical maintainability. The models were XML with OCL, but the promise was the same.

MDD failed. It did so for two reasons worth remembering:

1. **The abstraction level was awkward.** Models were too abstract for low-level details and too concrete for architecture. You ended up writing models almost as verbose as the code they generated, without the flexibility of code.
2. **Generation was inflexible.** When generated code didn't fit a specific situation, modifying it "broke" the model. You had to choose between living with suboptimal code or abandoning generation.

Fowler sees a direct echo in SDD:

> *SDD sits at an awkward abstraction level and just creates too much overhead and constraints. While LLMs reduce some MDD constraints, they introduce non-determinism — potentially combining MDD's inflexibility with LLM unpredictability.*

That last sentence deserves to stay on the wall. **SDD potentially combines MDD's inflexibility with LLM unpredictability**. The worst of both worlds: model rigidity plus generative model surprise. If MDD failed from just one of those problems, SDD could fail from the sum.

**How to defend yourself:** read the historical warning. When you're building very detailed specs, ask whether what you're writing is more maintainable than the code it would generate, and how much certainty you have that the agent will generate consistent code between runs. If both answers aren't clear, you're at the place where MDD died.

## Critique 5 — The review burden

A minor but practical Fowler critique: in his evaluation, spec-kit generated verbose, repetitive markdown files that **were heavier to review than the code itself** would have been. It's a specific observation about a specific tool, but it points to a general pattern: if your SDD process forces you to review more artifacts than the code it replaces, you've inverted the sign of the benefit.

**How to defend yourself:** measure real review time. A week of honest tracking tells you whether the spec saves review or adds to it. If it adds, simplify it.

## Critique 6 — The control that isn't control

Fowler also observes that, even with elaborate specs and structured workflows, **agents still ignore instructions, over-interpret others, create duplicates and produce unexpected things**. The spec gives a sensation of control that doesn't match real agent behavior. Process elaboration has gone up, output predictability hasn't.

It's the chapter's harshest observation: **a longer spec doesn't produce a more obedient agent**. It produces a more confident human and an equally unpredictable agent. And the gap between those two facts is exactly where problems sneak in.

**How to defend yourself:** measure adherence, not completeness. The question isn't "how detailed is the spec?", it's "what percentage of the spec's constraints get respected without re-prompting?". If the spec grows and adherence doesn't improve, you've added weight without traction.

## When SDD clearly *isn't* for you

After reading these six critiques, there are concrete signals SDD isn't the answer for your situation:

- You're in discovery phase where intent changes week to week.
- Every time you write a spec, the relevant code has changed before you finished.

If you recognize your situation in either of the two, **the cost of SDD probably exceeds its benefit for you**. It's not a failing of the discipline. It's that the tool doesn't fit your problem, and forcing it is wasting team time.

## What comes next

The critiques leave the reader with a legitimate question: *if bad SDD is this, what do I do then?* chapter 10 develops the alternative Isoform proposes: **context engineering**. It's not "vibe coding but worse"; it's a deliberately different proposal about where intent should live.
