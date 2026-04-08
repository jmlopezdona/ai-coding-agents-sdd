# 2. The specification spectrum

One of the most useful things the arXiv paper on Spec-Driven Development contributes is that it refuses to present SDD as one thing. It presents it as a **spectrum of three commitment levels**, and understanding which of the three you operate at — and which one you *think* you operate at — is probably the most consequential decision you'll make in this course.

> *The spec declares intent. The code realizes it.*
> — Spec-Driven Development paper, arXiv

This is the paper's thesis sentence, and it's worth keeping in front of you as we go through the three levels.

## Level 1 — Spec-First

The lightest level. You write a specification **before** starting to code, hand it to the agent, the agent generates the code, and from then on the spec is **abandoned**. The code takes on a life of its own. Any future change is made directly on the code and the spec stays fossilized in its initial version.

It's how most teams believe they're doing SDD. "Yes, we write a design doc before starting the feature." Fine, but that doc is just *spec-first*. Nothing more. It serves one specific thing — starting a feature with clear intent — and nothing else.

**When it works:**
- New features with defined scope and a short horizon.
- Prototypes where code is disposable and the spec doesn't need to keep living.
- Small teams where knowledge of the change fits in the head of whoever makes it.

**When it breaks:**
- As soon as the code enters maintenance.
- As soon as someone (or some agent) has to touch it six months later and the spec no longer describes reality.
- As soon as the drift between spec and code is such that the spec misleads instead of informs.

The most common error with spec-first is **believing it's spec-anchored**. A team writes specs at the start of every feature and believes it's doing SDD. It isn't. It has a startup ritual.

## Level 2 — Spec-Anchored

The middle level, and where most serious teams that adopt SDD well end up living. Spec and code **evolve together** as equal partners, and there's some automatic mechanism that detects when they desynchronize. Tests verifying behaviors described in the spec, BDD frameworks like Cucumber, validators comparing declared signatures in the spec with actual ones in the code, or recurring agents that detect drift between the two.

What distinguishes spec-anchored from spec-first isn't the quality of the initial spec. It's that **there's a tangible cost to letting the spec go out of date**. If a test fails, someone has to decide: do I update the code to satisfy the spec, or do I update the spec to reflect what the code now does? That decision, made a hundred times, is what keeps the spec alive.

The paper says something important about this level:

> *BDD frameworks like Cucumber exemplify this approach, making it the practical choice for most production systems.*

It's an honest hint. Spec-anchored isn't a new SDD invention. It's BDD done well, lifted to architectural intent rather than user-facing behavior. If your team already practices BDD with discipline, you're closer to SDD than you think.

**When it works:**
- Production systems with long lifespans.
- Teams where code passes through multiple hands (including artificial ones).
- Domains where invariants are important and non-negotiable.

**When it breaks:**
- When the cost of keeping spec and code synchronized exceeds the benefit. This is real and chapter 9 develops it honestly.
- When automated tests don't cover enough to fulfill their "anchoring" role.

## Level 3 — Spec-as-Source

The most radical level and, today, the least viable in most contexts. The spec **is** the source code. Humans only edit the spec. Code is generated from it, always, and never edited directly. If something is wrong in the code, you fix the spec and regenerate.

Tessl is the most visible example of this philosophy today: it generates code with a `// GENERATED FROM SPEC - DO NOT EDIT` comment at the top of each file. The promise is you have a single source of truth again, like classical compilation.

The paper is clear about where this actually works:

> *Currently viable mainly in mature domains like automotive embedded systems using tools such as Simulink.*

Translated: in domains where code generation from models has worked for decades because the models have a very narrow, very well-understood semantics. For general-purpose software — web APIs, frontends, internal scripts, everything most teams do 95% of the time — spec-as-source is still more promise than reality. The fundamental reason is the same one that killed Model-Driven Development in the 2000s, and chapter 9 returns to this.

**When it works:**
- Mature domains with formal semantics (embedded, control, hardware/software co-design).
- Thin glue code layers where the spec→code mapping is trivial.

**When it breaks:**
- In most real software. The spec→code translation isn't deterministic when the spec is in natural language and the generator is an LLM, and that non-determinism is exactly what breaks the level's central promise.

## How to know what level you're really at

A simple diagnostic question: **what happens when someone changes the code directly without touching the spec?**

- If the answer is "nothing, the spec stays as it was" → you're in spec-first, even if you call it SDD.
- If the answer is "CI fails, or a linter warns me, or a BDD test breaks" → you're in spec-anchored.
- If the answer is "you can't, the code is marked as generated" → you're in spec-as-source.

If you can't answer with confidence, the operative answer is the first one. You're in spec-first. It's not a tragedy, but don't tell yourself a different story.

## The "we want to be in spec-anchored" trap

The most expensive trap we see in teams adopting SDD is this: they aspire to spec-anchored, write very detailed specs as if they were there, but **don't invest in the anchoring mechanism**. There are no tests comparing spec and code. No validator. No recurring agent. Just a `specs/` folder with markdowns slowly going out of date.

That's spec-first dressed as spec-anchored, and it's worse than pure spec-first because it generates the false sensation of having a source of truth when what you have is documented fiction. Chapter 9 returns to this under the name "illusion of control".

The operating rule: **don't declare you do spec-anchored until you have the automatic anchoring mechanism working**. Without that mechanism, what you have is spec-first with more paperwork.

## What level should you choose

There's no universal answer, but there is an honest recommendation:

- **Start with spec-first, without disguising it**. Write specs before important features. Don't force yourself to maintain them.
- **Promote to spec-anchored only what has a long life**. The parts of the system that will live for years, where the cost of anchoring amortizes. Don't apply it to code you know will be replaced in six months.
- **Don't go to spec-as-source** unless you're in one of the mature domains where it works, or you're deliberately experimenting with Tessl on an isolated surface.

This progression mimics what the industry has done with tests: we started with no tests, then with tests where it hurt, then with TDD where the code is critical, and never with universal TDD except in projects where rigor justifies it. SDD follows the same curve, and understanding that saves you two years of overcommitting to a level you don't need.

## What comes next

Once you know which level you're operating at, the next question is very concrete: *how do you write a spec that's actually useful?* That's the content of **chapter 3**. We'll talk about what parts a good spec has, which ones really move the needle, and the *whys* — the ingredient most specs forget and which is exactly what makes them useful months later.
