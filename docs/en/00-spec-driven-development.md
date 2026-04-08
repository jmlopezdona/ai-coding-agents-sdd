# Spec-Driven Development: building software with intent

There's a moment, almost always around the fifth or sixth serious session with a coding agent, when the trick stops working. Up until then everything was reasonable: the agent understood what you asked, generated something that compiled, you iterated two or three times and walked away with code that looked healthy. And then something breaks. The feature works but breaks three others the agent never looked at. The refactor is structurally correct but ignores an unwritten convention of the project. The next session starts from zero as if the previous one never existed. The agent "understands" perfectly and still produces something subtly wrong.

The usual reaction is to blame the model, wait for the next one, or convince yourself you need better prompts. All three are evasions. The problem isn't the model and it's not fixed by better prompts. The problem is that **you're trying to make an agent execute an intent you never specified clearly**, and you're doing it in a medium that doesn't forgive ambiguity. The community has named this practice *vibe coding* — coding by intuition, prompt by prompt, with no structure — and the discipline that tries to correct it, *Spec-Driven Development*.

## What SDD is exactly (and what it isn't)

SDD isn't writing lots of documentation. It isn't waterfall in disguise. It isn't project-management ceremony. SDD is one very concrete idea:

> **The specification of intent — what is being built, why, with what constraints and under what acceptance criteria — is the primary artifact of the work. Code is the realization of that specification.**

Two things change. First, **order**: you specify before asking the agent to code, not after. Second, **persistence**: the spec lives in the repository, versioned, and is updated as the code evolves. It's the difference between "I drop a ticket on the agent and see what comes out" and "I give it a contract I can compare its output against".

What SDD isn't: a promise that the agent will get it right the first time, a guarantee that generated code will be correct, a substitute for reviewing what the agent produces, or a process that scales uniformly to any task — a three-line bug fix and a database migration don't ask for the same amount of spec, and forcing the opposite is one of the fastest ways to kill a team (chapter 9).

## Why we need a discipline, not just better prompts

The cleanest argument for SDD comes from accepting two facts about how agents work today.

The first is **context collapse**: every session begins with zero project knowledge. What the agent knew yesterday it doesn't know today. The naming conventions, the error-handling patterns, the architectural decisions the team discussed in Slack two months ago — all of that is invisible to the agent unless it's explicitly written somewhere the agent can read. In a 200-file project, implicit knowledge is enormous, and the agent guesses from its general training — which almost never matches yours.

The second is **context drift**, the operational cousin of context collapse but happening *within* a single session: the agent fixes a bug in one file and breaks three others it didn't look at. It's not that it forgot — its attention window doesn't cover the real blast radius of the change. The bigger the repo, the worse it gets.

A spec resolves both at once. It resolves collapse because it's persistent: the next session starts with the spec read. And it resolves drift because it turns the change into a *comparable contract*: if the spec says "these invariants don't change", post-code verification has a reference point.

That's why "better prompt" isn't a solution. A better prompt is an ephemeral correction. A spec is a correction that persists and any future agent can use.

## The role shift: from programmer to intent architect

If you accept SDD, your job changes less than it looks and more than you expect. You're still an engineer, but the medium is different:

- Before, you spent most of your time writing code that executed an intent you had clearly in your head.
- Now, you spend most of your time **making that intent explicit** — constraints, no-goals, acceptance criteria, the *whys* — so an agent can execute it.

It's less artisanal and more architectural work. The hard part stops being typing fast and starts being thinking precisely: *what* exactly do I want, *why* do I want it this way, *what* don't I want, *how* do I know it's right. When a team says "the agent doesn't understand what I'm asking", the honest translation is almost always "I didn't know exactly what I was asking". The spec is the tool that forces that precision before the agent gets a chance to guess.

## The specification spectrum, in one idea

One useful contribution of the arXiv SDD paper is that it doesn't present SDD as one thing. It presents it as a spectrum of three commitment levels:

- **Spec-First**: you write a spec before coding; useful to start; then the code diverges freely.
- **Spec-Anchored**: spec and code evolve together; automatic tests guarantee alignment; this is where serious teams live.
- **Spec-as-Source**: the spec *is* the source code; code is regenerated; humans only edit the spec. Today only viable in mature domains (Simulink, automotive embedded).

Most teams confuse what they're doing: they say "we're doing SDD" when they're actually doing aspirational *spec-first*, which decays to *vibe coding* once the code is six months old. Chapter 2 develops this spectrum because understanding what level you're operating at is the most important decision in this course.

## The critique that's also part of the course

An honest guide on SDD has to accept that SDD can also go wrong. Chapter 9 develops the strongest critiques — Isoform's *maintenance tax*, Martin Fowler's parallel with the old Model-Driven Development, the loss of the *whys*, the false illusion of control — and chapter 10 presents Isoform's alternative: *context engineering*, where intent and *whys* are preserved **inside the code** rather than in external specs.

They aren't there for noise. They're there because misapplied SDD is indistinguishable from bureaucracy, and if the course only teaches you the positive patterns it leaves you defenseless against the moment — and it will come — when your own SDD process becomes the bottleneck.

## How this fits in the trilogy

This guide is the middle piece between two sister courses:

- **Fundamentals** taught you what an agent is, how to talk to it, what tools it has. It's the *what*.
- **SDD** (this course) teaches you to structure work so the agent can execute it well. It's the *how to work with it*.
- **Harness Engineering** teaches you to engineer the system around the agent — sandboxes, sensors, hooks, observability, the whole repo as substrate. It's the *what's built around it*.

SDD exists because between knowing how to use an agent and knowing how to engineer a full harness around it, there's an intermediate discipline worth learning on its own. Without SDD, the harness becomes infrastructure without purpose. Without harness, specs become bureaucracy without traction. The three courses form a progression, and each one builds on the previous.

## Where to go next

If you want to understand the concrete pain SDD tries to solve, jump to **chapter 1**. If you already get it and want the conceptual model, start with **chapter 2** (the spectrum). If you came for tools, **chapters 6 and 7** are your shortcut. And if you came looking for judgment so you don't swallow the hype, read **chapter 9** first and come back to the start with new eyes.

You don't have to read it cover to cover. You do have to not stop at the comfortable chapters.

---

*This text synthesizes ideas from the arXiv SDD paper, Addy Osmani and Augment Code on living specs and boundaries, the critical analyses of Isoform and Martin Fowler (Thoughtworks) on the limits of SDD, and the community discussion around tools like Kiro, Spec-kit, Tessl, BMAD, and Traycer. URLs in the [README sources section](../../README.md#sources).*
