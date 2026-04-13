# 5. Possible lifecycle approaches

If you read three articles about Spec-Driven Development, you'll find three different versions of the "SDD lifecycle". That's not a flaw of the discipline; it's a sign the discipline is still forming. But it leaves the reader with a concrete problem: *what phases are there, in what order, and why do two serious sources say different things?*

This chapter disambiguates. There are **two main approaches** of the lifecycle in the literature, they aren't interchangeable, and the choice between them says something about what kind of project you're in.

## Approach A

[The arXiv SDD paper](https://arxiv.org/html/2602.00180) proposes a four-phase cycle:

1. **Specify** — define what the software has to do through behavior descriptions, acceptance criteria and requirements. Without prescribing implementation.
2. **Plan** — decide how it's built: technologies, architecture, data models, interfaces.
3. **Implement** — produce code that realizes the spec according to the plan, in small validated increments.
4. **Validate** — verify that the code actually meets the spec via automated tests, BDD scenarios, stakeholder acceptance.

What characterizes this approach is that **validation is an explicit, separate phase**. Verification isn't something that happens "at the end if there's time"; it's one of the four corners of the cycle, with the same weight as implementation.

## Approach B

[The most practical article on day-to-day SDD use](https://dev.to/pockit_tools/specification-driven-development-how-to-stop-vibe-coding-and-actually-ship-production-ready-5788) proposes a different cycle, also four phases, but with different names and boundaries:

1. **Requirements** — what's being built, including user stories, edge cases, security requirements and explicit *no-goals*.
2. **Design** — translate requirements into technical decisions: database schema, endpoints, architecture, error handling.
3. **Tasks** — break the design into discrete steps, dependency-ordered, small enough for a focused agent session, each with tests.
4. **Implementation** — the agent writes code task by task, guided by the full context of previous phases, with TDD verifying each step.

The important difference from approach A is that here **validation is embedded inside the task phase** (each task brings its tests) and doesn't appear as a standalone phase. In exchange, there's an explicit *Tasks* phase that doesn't exist as such in approach A and lives hidden inside "Plan".

## Which is "correct"?

Neither. They're two different views of the same process, optimized for different situations:

- **Approach A (Specify → Plan → Implement → Validate) is better when the system has strong invariants needing explicit and formal validation.** Public APIs, regulated systems, code crossing team boundaries. If verification is a social and technical event deserving its own phase, this is your approach.

- **Approach B (Requirements → Design → Tasks → Implementation) is better when the bottleneck is decomposing into chunks digestible by the agent.** Big features a human could do but an unaided agent can't, because the blast radius of changes exceeds the attention window. If your problem is "the agent gets lost in my 200-file repo", this is your approach.

In practice, mature teams end up combining: they use approach B's names in daily flow and break out a separate validation phase (approach A) when the feature touches something critical. That's not contradictory. It's choosing the right tool for the change's risk.

## The three phases of the lifecycle

Both approach A and B — and every framework in [chapter 7](07-native-sdd-tools.md) — do the same three things, even if they name them differently and distribute them in different ways. These three phases are the general model of the SDD lifecycle:

| Phase | What happens | Approach A | Approach B |
|---|---|---|---|
| **1. Design the spec** | Everything before code: specify intent, plan, decompose into tasks | Specify + Plan | Requirements + Design + Tasks |
| **2. Implement the spec** | Write code guided by what phase 1 produced | Implement | Implementation |
| **3. Validate the spec** | Verify the code does what the spec asked — and nothing more | Validate | *(embedded in each task)* |

The native tools implement these same three phases in different ways:

| Framework | Phase 1: Design | Phase 2: Implement | Phase 3: Validate |
|---|---|---|---|
| [**Kiro**](07-native-sdd-tools.md#kiro) | 3 documents: Requirements → Design → Tasks | Agent executes tasks | On-save hooks per task |
| [**Spec-kit**](07-native-sdd-tools.md#spec-kit-github) | Constitution + feature specs with checkpoints | Agent implements | Aspires to anchoring, but is spec-first in practice |
| [**Tessl**](07-native-sdd-tools.md#tessl) | Single formal spec (spec-as-source) | Generates code from spec | Regeneration as reconciliation |
| [**BMAD**](07-native-sdd-tools.md#bmad) | PM (requirements) + Architect (design) + QA (test plan) | Developer agent | QA agent verifies |

What this table reveals matters:

- **Phase 1 is where the complexity lives.** It can be a single act (writing a brief spec) or three documents with three different reviewers. The granularity depends on context, not method.
- **Phase 2 is the most stable.** Everyone does the same thing: the agent writes code guided by what Phase 1 produced. There's no significant variation here.
- **Phase 3 is where frameworks diverge most** — and where they fail most. [Chapter 7](07-native-sdd-tools.md#what-none-of-these-tools-solve) identifies post-code verification as the gap no native tool fully solves. Some make it explicit (approach A), some embed it in each task (approach B, Kiro), some barely implement it (Spec-kit), and some delegate it to a specialized agent (BMAD). Whether validation exists as a phase with real weight — not as something optional done "if there's time" — is what separates SDD from vibe coding with documentation.

## Phase 1 — Design the spec

This phase covers **everything that happens before writing code**. It's the phase with the most variable granularity: in a simple project it can be a single step; in an enterprise project with stakeholders and prior documentation it can unfold into several sub-steps with their own artifacts and reviews.

!!! warning "What «spec» means in this lifecycle"

    When this chapter says *"the spec"*, it doesn't necessarily mean a single document. It means **everything the agent needs for completeness and rigor**: what to implement, what not to implement, why, why not, and how to validate it.

    For a PoC, a prototype, or a small greenfield project, that information fits in a single file — the chapter 3 spec is enough. But for an enterprise, brownfield, or sufficiently complex project, the spec **is the sum** of the SDD document plus the upstream functional and technical documentation that feeds it: client-validated requirements, architectural decisions, API contracts, documented business rules.

    If that upstream documentation doesn't exist or lacks the necessary rigor, the first step isn't writing the spec — it's **getting that upstream in place**. Skipping this step and trusting the agent to derive requirements from an informal conversation is exactly how you fall into the lack of rigor SDD is meant to prevent. Chapter 4 details [how to consume, produce and reference those artifacts](04-spec-in-context.md#the-spec-and-its-external-sources-consume-produce-modify).

### One document or many: two strategies

The sub-steps of this phase produce artifacts — but **do they live in a single file or in several?** The answer depends on the project's scale and whether upstream documentation already exists.

**Integrated document** — The six blocks from chapter 3 plus the plan and tasks coexist in a single file. This is the natural option for PoCs, prototypes, greenfield projects, and small to medium changes. Its main advantage: the [bidirectional loop from chapter 6](06-living-specs.md) updates a single location, with no risk of drift between artifacts.

**Separate artifacts** — Each sub-step produces its own document: requirements, technical design, task list. This is the model [Kiro](07-native-sdd-tools.md#kiro) implements natively with its **Requirements → Design → Tasks** flow, and the one that fits enterprise teams where different people review different layers (product validates requirements, architecture validates design, the team validates tasks). The chapter 3 spec acts as the **master document** that references the others without duplicating them — exactly the [reference without reimplementing](04-spec-in-context.md#three-relationships-three-rules) rule from chapter 4.

The cost of separate artifacts isn't trivial: each additional document is a drift surface. [Spec-kit](07-native-sdd-tools.md#spec-kit-github) suffered exactly this — Fowler observed that generated files were *"heavier to review than the code itself"*. If you choose this strategy, you need discipline or tooling to keep artifacts synchronized, and today [none of the native tools fully solve this](07-native-sdd-tools.md#what-none-of-these-tools-solve).

In both cases, the **conceptual "spec" is always one** — it's the sum of everything the agent needs to implement with rigor. What changes is how it's physically distributed. And the deciding factor is usually the chapter 4 question: **does validated upstream documentation exist?** If yes, the spec consumes and references it (separate artifacts makes sense). If not, the spec must contain everything (integrated document is more natural).

| Strategy | When it fits | Reference framework | Main risk |
|---|---|---|---|
| **Integrated document** | PoC, greenfield, small teams, medium changes | Manual with ch. 3 template | Can grow too large for complex projects |
| **Separate artifacts** | Enterprise, brownfield, layered stakeholders, validated upstream | [Kiro](07-native-sdd-tools.md#kiro), [BMAD](07-native-sdd-tools.md#bmad) | Drift between documents; maintenance tax |

### Sub-step 1.1 — Specify intent

You open a session with the agent. **You don't ask it to write code**. You ask it to help draft the spec using the chapter 3 template, **feeding it with whatever upstream documentation** you already have — functional requirements, acceptance criteria, prior technical decisions. You give the high-level objective and let it ask questions. If the agent doesn't ask questions, ask them yourself: "what parts of the system does this touch", "what edge cases am I forgetting", "what aren't we building that we should explicitly say".

The output of this sub-step is a file in `specs/` or wherever your project hosts them. Committed, reviewable, versioned.

!!! tip "Iterative clarification"

    The [*Code Generation with LLM-based Agents*](https://arxiv.org/html/2508.00083v1) survey documents that systems like **ClarifyGPT** and **TiCoder** introduce an iterative questioning phase **before** the spec is finalized: instead of taking the first prompt as truth, the agent surfaces ambiguities and gaps before they materialize as wrong code.

    In practice, this means specifying **isn't linear**. It's a mini-loop: prompt → agent questions → answers → draft spec → more questions → final spec. Teams that treat this sub-step as ping-pong instead of dictation get substantially better specs.

    This clarification connects directly with chapter 3's **boundaries** — particularly the *"ask first"* category: the questions the agent should ask before acting. The difference is that here the questions happen before the spec exists, not after.

### Sub-step 1.2 — Plan implementation

With the spec in hand, you ask the agent for an **implementation plan**: which files it touches, in what order, what dependencies between tasks, what tests it intends to write. The plan isn't code; it's an intermediate document. You read it, discuss it, correct it if the agent misunderstood any constraint.

This sub-step is where Traycer-style tools add the most value ([chapter 8](08-architect-layers.md)), because the plan's quality determines the quality of everything that comes after.

### Sub-step 1.3 — Decompose into tasks

You take the plan and cut it into **small, sequential tasks, each with a "done" criterion**. A typical task should fit in a single focused agent session, with no ambiguity. If a task needs more than one session, it's badly cut.

Heuristic rule: if you can't describe the "after the task" state in one sentence, the task is too big.

!!! note "Not every project needs all three sub-steps"

    For a small change or a PoC, sub-steps 1.2 and 1.3 can be implicit or nonexistent — the spec itself is the plan and the tasks are obvious. The entire Phase 1 can take five minutes. For an enterprise project with multiple stakeholders, each sub-step can have its own review and approval cycle. **The weight of Phase 1 should be proportional to the change's risk** — exactly the rule from [chapter 9](09-patterns-of-application.md).

## Phase 2 — Implement the spec

The agent executes tasks one at a time. For each task: read the spec, read the task, write the tests the task asks for, write the code, run the tests, verify. If tests don't pass, iterate. If they pass, mark done and move on.

What matters here is that **each task is atomic with respect to the spec**: either it's fully done or it's reverted. There are no half-done tasks "to be fixed in another".

This is the most stable phase of the cycle — every framework implements it similarly. What varies is the agent's level of autonomy (from a general agent with the spec as context, to [BMAD](07-native-sdd-tools.md#bmad) with a specialized Developer agent) and whether validation happens within each task (approach B, [Kiro](07-native-sdd-tools.md#kiro)) or is deferred to Phase 3.

## Phase 3 — Validate the spec

When all tasks are done, someone — the agent, you, or a second reviewer agent — takes the original spec and verifies that **everything the spec asked is done, and that nothing the spec didn't ask for has been done**. This phase is the easiest to skip and the most important: it's where acceptance criteria stop being text and become a checked-or-unchecked checklist.

It's also the phase where **frameworks diverge most** — and where they fail most:

- **Explicit and separate validation** (approach A): verification is its own event with weight in the process. The most rigorous option, appropriate when there are strong invariants, public APIs, or code crossing team boundaries.
- **Validation embedded in each task** (approach B, [Kiro](07-native-sdd-tools.md#kiro)): each task brings its tests and self-validates on completion. More agile, but without a final verification you lose the global view — each task passing its tests doesn't guarantee the whole fulfills the spec.
- **Aspirational validation** ([Spec-kit](07-native-sdd-tools.md#spec-kit-github)): aspires to spec-anchored but in practice has no automatic mechanism comparing spec against code. It's spec-first in disguise.
- **Validation by regeneration** ([Tessl](07-native-sdd-tools.md#tessl)): if you regenerate code from the spec and get the same result, it "validates". But LLM non-determinism conflicts directly with this promise.
- **Validation delegated to a role** ([BMAD](07-native-sdd-tools.md#bmad)): a specialized QA agent verifies. The specialization helps with focus, but coordination failures between agents are subtle.

!!! warning "Phase 3 is what separates SDD from vibe coding with documentation"

    If Phase 3 doesn't exist or is reduced to "trust that the tests pass", you're not doing SDD — you're doing spec-first with good intentions. Verifying that the code fulfills the spec **as a whole** (not just task by task) is the mechanism that closes the cycle. Without it, the spec ages from the moment it's written.

    This is exactly the gap [chapter 7 identifies as a common problem](07-native-sdd-tools.md#what-none-of-these-tools-solve) across native tools, and where the architect layers from [chapter 8](08-architect-layers.md) have positioned themselves.

## The lifecycle isn't linear

A note almost always omitted in canonical descriptions: **the cycle is rarely linear in practice**. What really happens is in Phase 2 you discover something that invalidates a Phase 1 assumption, and you go back. Phase 3 teaches you that a criterion wasn't verifiable and you have to rewrite it. Mid-task, you realize the spec has a hole.

This is **normal and desirable**. The difference between a healthy cycle and a pathological one isn't that the first has no backtracks; it's that the healthy one **updates the spec when it discovers it was wrong** instead of ignoring it and continuing to code. That feedback from Phase 3 back to Phase 1 is exactly the idea of the next chapter: **living specifications**.

## What comes next

Chapter 6 is about the difference between static specs (the ones that age badly) and living specs (the ones that stay useful because implementation feeds back into the spec). It's the piece that turns the lifecycle described in this chapter into a sustainable process instead of a startup ritual.
