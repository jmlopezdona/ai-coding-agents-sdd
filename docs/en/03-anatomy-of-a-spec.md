# 3. Anatomy of a good specification

There are many ways to write a spec badly and rather few to write one well. This chapter tries to distill what distinguishes a useful spec for an AI agent from one that's ceremony in disguise. Short version: a good spec captures **intent, constraints, and the *whys*** with enough precision that the agent doesn't have to guess, and with enough brevity that a human will still read it six months from now.

## What needs to be in it

A minimum useful spec has six blocks. Not all six need to be long, but all six need to be there.

### 1. Objective

One or two sentences describing what's being built and *for whom*. Not what API it exposes, not what endpoints it has — that comes later. Just: the observable result that justifies the work.

> *Allow a user to upload a profile photo that will be visible to followers and deletable only by them.*

If the objective can't be written in two sentences, almost always it's because it's mixing two different features. Separate them and write two specs.

### 2. No-goals

This is the section nobody writes and the one that prevents the most drift. **What is NOT being built**, explicitly.

> *Animated GIF support won't be supported in this version. Images won't be auto-compressed. There's no integration with the moderation system; that lives in another spec.*

No-goals are a vaccine against the agent's scope creep. Without them, the agent — following its training — will add compression, content validation, multi-format support and thumbnails, because all of those are "what's normally done". And all of those are work you didn't ask for.

### 3. Technical constraints

Technical design decisions *specific to this feature* that the agent needs to know to stay on track. These are not general project conventions (those live in the agent's global configuration — CLAUDE.md, rules, skills —, not in each spec). They're the architectural decisions that bound the solution space for *this* particular change.

> *Images are served from the existing S3 bucket; no new bucket is introduced. Authentication goes through the `requireAuth` middleware; no other is invented. No new table is created; the avatar is stored as a field on the existing `User` model.*

Constraints are what makes the feature fit in the rest of the system instead of landing as a foreign body. They're the antidote to the context collapse from chapter 1.

### 4. Verifiable acceptance criteria

How you know it's done. The keyword is **verifiable**: an outsider, reading only the criterion, should be able to decide whether the code meets it or not.

> *An authenticated user can upload a JPEG or PNG image up to 10 MB. Upload fails with a clear message if the file is another type or exceeds the size. Only the file's owner can delete it. An integration test covers all three cases.*

"Works well" isn't an acceptance criterion. "Returns 413 with a readable message if the file exceeds 10 MB" is.

When acceptance criteria come from upstream documents (user stories, contracts, ADRs), the rule is **rewrite with traceability**, not copy or reference blindly. The detail of how to do this — including the special case of documents living in the same repo — is developed in [section 4](04-spec-in-context.md#the-spec-and-its-external-sources-consume-produce-modify).

### 5. The *whys*

This is where ordinary specs die and good ones separate from the rest. Almost all specs explain the *what*. Almost none explain the *why*. And the whys are exactly what the agent needs to make smart decisions in the gaps the spec doesn't cover.

> *We don't auto-compress the image because the design team wants to preserve quality for verified-account avatars. (September 2025 decision, owner: @maria.)*
>
> *Only the owner can delete because adding moderation would change the permissions model and we want to keep this iteration's spec minimal.*

Whys are the ingredient [Isoform's critique](https://isoform.ai/blog/the-limits-of-spec-driven-development) identifies as what bad SDD loses, and the main reason specs become useless months later. A spec without whys ages at the speed of the decisions around it. A spec with whys outlives those decisions because it explains them.

### 6. Boundaries — the three-tier system

This is [Addy Osmani's](https://addyosmani.com/blog/good-spec/) most useful contribution to thinking about specs for agents. Instead of a flat list of "do" and "don't", divide constraints into **three tiers**:

- **✅ Always do** — things the agent should do without asking permission.
  *Example: run the test suite before declaring any task done.*
- **⚠️ Ask first** — things the agent should **ask confirmation** before doing.
  *Example: modify the database schema, add a new dependency, touch deployment config files.*
- **🚫 Never do** — things the agent must never do.
  *Example: commit secrets or keys, modify `vendor/`, disable tests so they pass, force push.*

The crucial distinction is the middle one. Without "ask first", the agent has only two modes: act or freeze. "Ask first" introduces a third — "consult before proceeding" — which is where productive human–agent conversation actually fits. It's the piece most teams forget, and the one that turns a restrictive spec into a collaborative one.

#### The six blocks at a glance

| Block | Purpose | Typical trap |
|---|---|---|
| **Objective** | Observable outcome and for whom, in 1-2 sentences | Mixing two features into one |
| **Non-goals** | Vaccine against the agent's scope creep | Not writing them |
| **Technical constraints** | Design decisions that bound the solution for this feature | Mixing in global project conventions |
| **Acceptance criteria** | How you know it's done, verifiable | "Works well" as a criterion |
| **Whys** | Reasons with owner and date; what ages well | Documenting *what* and omitting *why* |
| **Boundaries** | Always / Ask first / Never for the agent | Forgetting "Ask first" |

## A minimal template

```markdown
# Spec: [feature name]

## Objective
One or two sentences. Observable result. For whom.

## No-goals
- Thing NOT done
- Other thing NOT done

## Technical constraints
- Design decisions specific to this feature
- Existing components to use (and which ones not to touch)
- (Global project conventions live in CLAUDE.md / rules, not here)

## Acceptance criteria
- [ ] Verifiable criterion 1
- [ ] Verifiable criterion 2

## Whys
- Why this decision and not another (with owner and date)
- Accepted trade-offs

## Boundaries
- ✅ Always: ...
- ⚠️ Ask first: ...
- 🚫 Never: ...
```

This template fits in less than a screen and covers 80% of the value a spec can deliver. The other 20% are domain-specific details — state diagrams, API contracts, concrete input/output examples — that you add when the domain demands them, not by default.

It's normal for a spec to touch more than one component. Information about those components fits in the six blocks themselves: components consumed without modification go in **technical constraints**, guarantees of what's produced or modified go in **acceptance criteria**, and the reasoning behind each decision goes in **whys**. If you struggle to fit that information into the six blocks, the spec is probably too large and should be split. [Section 4](04-spec-in-context.md#the-spec-and-its-external-sources-consume-produce-modify) develops how to think about the spec's relationship with its external sources.

## The curse of instructions

**More information in a single spec isn't better — it's worse.** A global 500-line spec is a bad plan; five modular 100-line specs where today's task references just two is a good plan. This is the most important practical rule in the whole chapter, and it cuts against the intuition of almost everyone who starts with SDD.

The reason has a name. [Addy Osmani](https://addyosmani.com/blog/good-spec/), citing academic studies, calls it **the curse of instructions**: the more you put in a prompt, the worse the model follows *each* of the things. It's not linear — it's a qualitative collapse. A spec with five constraints gets followed reasonably well. A spec with fifty gets followed reasonably badly across all of them, and often the agent acts as if none existed.

The winning strategy, then, is to break large specs into small modules and pass the agent only the module relevant to the current task.

This is also one of the reasons tools like [Spec-kit and Kiro](07-native-sdd-tools.md) generate **many small files** instead of one big one, and why that proliferation, mismanaged, becomes its own problem (we'll see this in chapter 10).

## What if a generator agent writes the spec?

**The thesis of this section, up front:** the agent works as a *draft generator of the what* — objective, candidate criteria, tentative non-goals, boundaries derived from the repo — but **not** of the *how*. And the **whys** it can't generate well no matter how much prompting you throw at it, because they aren't in the input. If you keep one rule: *if your generated spec mentions a function signature or a class name, delete it*.

The rest of the section explains why the optimistic version — *"the agent writes the whole spec"* — fails by default almost every time, not because of bad prompting but for a structural reason worth understanding before you spend weeks discovering it.

It's a natural idea: if the agent can write code, it should also be able to write specs. You hand it a user story, a feature description or an architectural component, and it returns the template above filled in. Startup cost drops, the entry barrier disappears, and apparently you've solved SDD's most expensive problem — writing specs. But there's a structural problem underneath.

### The default mode: pseudocode dressed up as a spec

When you ask an LLM "generate a spec for this feature", the model optimizes for the metric it knows best how to measure: *apparent coverage*. And the easiest way to look exhaustive is to drop down to code-level detail, because that's where the model is comfortable — its training corpus is code. Climbing up to *intent*, *constraints* and *whys* requires abstraction, which is exactly what the model does worst.

The result is predictable and almost universal: the generated "spec" is actually **commented pseudocode** dressed up as a design doc. It has classes, methods, function signatures, API payloads, inline business rules, sequence diagrams. It looks thorough. And it's exactly the *opposite* of what a good spec should be, according to everything we've seen in this chapter.

A good spec operates at a **different abstraction level** from the code. If it mentions classes and methods, it's no longer a spec: it's an implementation sketch. The spec's question isn't "which class calls which method?", it's "what observable guarantees must this satisfy, under what constraints, and why?". When the agent drops to the first level, the spec has lost its reason for existing, because it stops being **comparable against the code from a different vantage point**. If the spec says the same thing as the code but in markdown, it's just noise.

### The two serious consequences

When you end up with specs like this — and, if you leave the agent in its default mode, you will — you pay two costs that reinforce each other.

**You implement twice.** You've already paid the cost of thinking through the implementation at the level of classes and methods in the spec, and you're going to pay the same cost again when you code. Worse: anything you discover while implementing (and implementing always reveals something) forces you to go back and edit the spec, or accept drift on day one. You're at the worst point on the chapter 10 curve: maintenance tax from day one, before you've written a single line of code.

**Human review becomes as expensive as code review.** This is exactly what [Fowler reports about Spec-kit](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html) in chapter 7 — that the generated files were *heavier to review than the code itself*. And it's worse than reviewing code directly, because at least code runs and tests verify it; pseudocode in markdown has no safety net. You're reviewing something at code-level detail but **without code's guarantees**. The tired reviewer skims it — and the "spec" passes the filter as if it had been reviewed.

The combination of the two is poisonous: you double the work *and* review it worse. It's almost always worse than not having written a spec at all.

### Why it isn't just a template problem

It's tempting to think this gets fixed with better prompting or a stricter template. *"I'll tell the agent not to include signatures, not to mention classes, to express everything as externally observable invariants."* And yes, that reduces the problem. But it **doesn't eliminate it**, for two reasons worth being clear about.

First, the model's bias toward dropping to code level is very strong because that's where it "feels productive". When you restrict everything it does well and ask only for what it does badly — abstracting intent, capturing whys that aren't in the input — the resulting spec is short, thin, and the team senses "something is missing". The next step is almost always asking for more detail, and you're back to pseudocode. It's an unstable equilibrium.

Second, there are things the agent **can't** generate well no matter what template you give it. The **whys** are almost never in the input — they live in the PM's head, in a Slack conversation, in a trade-off decided two years ago. What the agent doesn't know, it invents or omits. A spec generated with invented "whys" is worse than a spec without "whys", because it lies with the appearance of authority.

### Legitimate use: the agent as a draft generator of the *what*, not the *how*

This doesn't mean agent-generated specs are useless. They're useful, but in a narrower role than most people imagine:

- **The agent can start well**: the **objective** block, the **verifiable acceptance criteria** (carefully), a first list of **candidate non-goals** that the human then refines, and a proposal for **boundaries** based on patterns it already sees in the repo.
- **The human has to put in, no exceptions**: the **whys**, the **real non-goals** (especially political or scope ones that aren't in any input), the team's **tacit technical constraints**, and the decision of *how much* spec the task deserves (what chapter 9 will call *modulation*: matching the weight of the spec to the actual complexity and risk of the feature). And, above all, the human has to **delete** anything the agent injected as pseudocode: classes, methods, signatures, payloads. If after deleting all that the spec is empty, you didn't need a spec; you needed well-written code (chapter 11, *context engineering*).

The most useful practical rule: **if your generated spec mentions a function signature or a class name, delete it**. The spec talks about what guarantees the system meets, not how it's built.

And the honest warning: using the agent as a draft generator **doesn't reduce the total cost** of doing SDD well. What it reduces is the cost of starting the first draft, which for many teams is the psychological barrier that weighs the most. The real cost — thinking intent precisely, capturing whys, keeping the spec alive — is still there, with no shortcut, and the agent doesn't pay it for you. If your SDD adoption depends on automatic generation eliminating that cost, what you'll have isn't SDD — you'll have [anti-pattern #12](12-anti-patterns.md#12-generated-spec-thats-pseudocode-in-disguise) from chapter 12.

## What comes next

Up to here we've seen what to put inside a spec and the two most expensive traps when writing one: specs that are too large and specs generated by agents without supervision. But a spec's anatomy doesn't end with its blocks — **the right level of detail depends on context**: the spectrum level you're operating at, the technical components you touch, and the kind of validation you'll apply.

In **[section 4](04-spec-in-context.md)** we tackle exactly that: how to calibrate the spec so that detail is neither waste nor blindness.
