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

A spec is the unit of work that **one agent receives and resolves autonomously in its inner loop**. The agent reads the spec, internally decomposes the work into tasks, implements, and validates — all within a single execution. The principle is always the same: **each task is atomic with respect to the spec** — either it's fully done or it's reverted.

What varies is how the agent manages those tasks internally. Today's tools give it two options:

### Direct execution — The agent solves everything in its context

The agent executes tasks one at a time within its own session. For each task: read the spec, read the task, write the tests, write the code, run the tests, verify. If tests don't pass, iterate. If they pass, move on to the next.

Interfaces between tasks are implicit — the agent remembers them because it works in the same context. It's the simplest and most predictable option, and works well when tasks have **strong dependencies between them** or when the spec fits comfortably in the agent's context window.

### Delegation to sub-agents — The agent coordinates internally

Today's tools (Claude Code with sub-agents, Cursor with background agents) allow the agent, **within its same execution**, to delegate parts of the work to specialized or context-scoped sub-agents.

Imagine a spec that touches front-end, middleware, and back-end. The main agent can decide — for better context management or to use more specialized prompts — to launch internal sub-agents for each layer. Each sub-agent receives its slice of the spec and the interface contracts with the others, implements and validates its part. The main agent integrates the results and verifies the parts fit together.

It's important to understand that **this is still a single spec executed by a single agent**. The decision to use sub-agents is internal to the execution — it's an implementation strategy of the agent, not a different workflow. From the outside, the result is the same: the agent received a spec and delivered code that fulfills it.

That delegation can happen in two ways:

- **Implicit** — The spec says nothing about how to decompose the work. The agent decides on its own whether to delegate and how, based on its assessment of complexity. Simpler to write, but you depend on the agent making good decomposition decisions — and those decisions aren't documented anywhere.
- **Explicit** — The spec defines the delegable parts, their boundaries, and the contracts between them. *"The front-end consumes this contract; the back-end produces it; they're independently implementable."* More work in Phase 1, but it gives the agent the information to delegate with rigor. Moreover, those boundaries serve as documentation of the change's structure even if the agent doesn't use sub-agents — they connect directly with chapter 3's **boundaries** and with [chapter 4's produced and consumed artifacts](04-spec-in-context.md#three-relationships-three-rules).

| | Direct execution | Delegation to sub-agents |
|---|---|---|
| **Context** | One session, one context | Main agent + sub-agents with scoped context |
| **Parallelism** | No | Possible, by context or layer |
| **Interfaces between tasks** | Implicit (same context) | **Explicit in the spec** (each sub-agent only sees its part) |
| **Main risk** | Context window limits | Integration failures between parts |
| **When it fits** | Scoped specs, tasks with strong dependencies | Specs crossing layers with well-defined interfaces |

!!! tip "Impact on Phase 1"

    If the spec has the scale or structure that makes it likely the agent will delegate to sub-agents, **interfaces between parts become load-bearing**: they can't be implicit. *"The endpoint accepts X and returns Y; the front-end consumes Y with this contract"* has to be in the spec, because each sub-agent will only see its context. This connects directly with [chapter 4's produced and consumed artifacts](04-spec-in-context.md#three-relationships-three-rules) — each sub-agent *consumes* the contract another sub-agent *produces*.

!!! note "Connection with BMAD and Traycer"

    Delegation to sub-agents isn't conceptually new — [BMAD](07-native-sdd-tools.md#bmad) already uses multiple agents, but with **fixed roles** (PM, Architect, QA, Developer). What today's tools enable is more flexible delegation: sub-agents by **spec context** (front, back, infra), not by process role. And architect layers like [Traycer](08-architect-layers.md) fit naturally as the coordination logic — their planning and verification functions are exactly what the main agent needs to orchestrate sub-agents.

Neither option is universally better. Direct execution is simpler; delegation to sub-agents manages context better for large specs but requires explicit interfaces in Phase 1 and integration validation in Phase 3. The decision is made by the agent (or its configuration) based on the spec's nature.

## Phase 3 — Validate the spec

Validation is what closes the cycle — where acceptance criteria stop being text and become a checked-or-unchecked checklist. But it's not a single act: it happens at **two levels** and with **two types of mechanism**.

### Two levels: inner loop and outer loop

**Inner loop validation** — The agent itself, before declaring "done", verifies that all acceptance criteria in the spec are met. Each task already has its individual validation (tests), but here it's about a **global check against the spec as a whole**: was everything the spec asked for done? Was anything done that the spec didn't ask for? If the agent delegated to sub-agents, this includes verifying the parts integrate correctly. This validation closes the agent's execution.

**Outer loop validation** — After the agent delivers, a human or an external process (CI, an independent reviewer agent, a PR review) validates that the spec was fulfilled. This is where things the agent can't verify alone come in: does the result make business sense? Does the integration with the rest of the system work? Does the stakeholder agree? Were constraints that aren't code-verifiable respected (regulatory, UX, performance under real load)?

Without inner loop, the agent delivers half-done work and the human bears all verification burden. Without outer loop, you blindly trust the agent evaluated its own work correctly.

### Two mechanisms: deterministic and stochastic

Within each level, there's an equally important distinction — **how** validation happens:

**Deterministic validation** — Unit tests, integration tests, linters, type checkers, OpenAPI contracts against code, CI checks. The result is always the same: pass or fail. No ambiguity. It's the most reliable validation and **should be the foundation** of both levels.

**Stochastic validation (by agent)** — An LLM reviews whether the code fulfills the spec, whether acceptance criteria are satisfied, whether nothing was done out of scope. It's useful for what deterministic tools can't capture — does the code respect the intent? Were the non-goals respected? Is the implementation coherent with the whys? — but it's inherently non-deterministic: two runs of the same validation prompt can give different results.

| | Deterministic | Stochastic (agent) |
|---|---|---|
| **Inner loop** | Tests, linters, type checks — the agent runs them as part of its cycle | The agent self-evaluates against the spec before declaring "done" |
| **Outer loop** | CI, contract checks, regression suite | Independent reviewer agent, human assisted by agent |

The healthy combination is **deterministic as foundation, stochastic as complement** — not the other way around. If your only validation is asking an agent to review another agent's work, you have stochastic validating stochastic — which is exactly the reliability problem [Tessl](07-native-sdd-tools.md#tessl) suffers with regeneration.

### How the frameworks handle it

Validation is where **frameworks diverge most** — and where they fail most:

- **Explicit and separate validation** (approach A): verification is its own event with weight in the process. The most rigorous option, appropriate when there are strong invariants, public APIs, or code crossing team boundaries.
- **Validation embedded in each task** (approach B, [Kiro](07-native-sdd-tools.md#kiro)): each task brings its tests and self-validates on completion. More agile, but without a final verification you lose the global view — each task passing its tests doesn't guarantee the whole fulfills the spec.
- **Aspirational validation** ([Spec-kit](07-native-sdd-tools.md#spec-kit-github)): aspires to spec-anchored but in practice has no automatic mechanism comparing spec against code. It's spec-first in disguise.
- **Validation by regeneration** ([Tessl](07-native-sdd-tools.md#tessl)): if you regenerate code from the spec and get the same result, it "validates". But LLM non-determinism conflicts directly with this promise — it's pure stochastic validation with no deterministic foundation.
- **Validation delegated to a role** ([BMAD](07-native-sdd-tools.md#bmad)): a specialized QA agent verifies. The specialization helps with focus, but it's still stochastic validating stochastic unless the QA agent runs deterministic tests.

!!! warning "Phase 3 is what separates SDD from vibe coding with documentation"

    If Phase 3 doesn't exist or is reduced to "trust that the tests pass", you're not doing SDD — you're doing spec-first with good intentions. Verifying that the code fulfills the spec **as a whole** (not just task by task), at **both levels** (inner and outer loop), and with a **deterministic foundation** (not just stochastic) is the mechanism that closes the cycle. Without it, the spec ages from the moment it's written.

    This is exactly the gap [chapter 7 identifies as a common problem](07-native-sdd-tools.md#what-none-of-these-tools-solve) across native tools, and where the architect layers from [chapter 8](08-architect-layers.md) have positioned themselves.

## The lifecycle isn't linear

A note almost always omitted in canonical descriptions: **the cycle is rarely linear in practice**. What really happens is in Phase 2 you discover something that invalidates a Phase 1 assumption, and you go back. Phase 3 teaches you that a criterion wasn't verifiable and you have to rewrite it. Mid-task, you realize the spec has a hole.

This is **normal and desirable**. The difference between a healthy cycle and a pathological one isn't that the first has no backtracks; it's that the healthy one **updates the spec when it discovers it was wrong** instead of ignoring it and continuing to code. That feedback from Phase 3 back to Phase 1 is exactly the idea of the next chapter: **living specifications**.

## What comes next

Chapter 6 is about the difference between static specs (the ones that age badly) and living specs (the ones that stay useful because implementation feeds back into the spec). It's the piece that turns the lifecycle described in this chapter into a sustainable process instead of a startup ritual.
