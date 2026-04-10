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

## The lifecycle, step by step, with a real agent

What follows is a **practical synthesis of both approaches** — a five-step flow that takes the best of each. It's neither in its pure form, but rather how the process looks when you actually do it.

!!! warning "What «spec» means in this lifecycle"

    When this chapter says *"the spec"*, it doesn't necessarily mean a single document. It means **everything the agent needs for completeness and rigor**: what to implement, what not to implement, why, why not, and how to validate it.

    For a PoC, a prototype, or a small greenfield project, that information fits in a single file — the chapter 3 spec is enough. But for an enterprise, brownfield, or sufficiently complex project, the spec **is the sum** of the SDD document plus the upstream functional and technical documentation that feeds it: client-validated requirements, architectural decisions, API contracts, documented business rules.

    If that upstream documentation doesn't exist or lacks the necessary rigor, the first step isn't writing the spec — it's **getting that upstream in place**. Skipping this step and trusting the agent to derive requirements from an informal conversation is exactly how you fall into the lack of rigor SDD is meant to prevent. Chapter 4 details [how to consume, produce and reference those artifacts](04-spec-in-context.md#the-spec-and-its-external-sources-consume-produce-modify).

### Step 1 — Specify intent

You open a session with the agent. **You don't ask it to write code**. You ask it to help draft the spec using the chapter 3 template, **feeding it with whatever upstream documentation** you already have — functional requirements, acceptance criteria, prior technical decisions. You give the high-level objective and let it ask questions. If the agent doesn't ask questions, ask them yourself: "what parts of the system does this touch", "what edge cases am I forgetting", "what aren't we building that we should explicitly say".

The output of this step is a file in `specs/` or wherever your project hosts them. Committed, reviewable, versioned.

### Step 2 — Plan implementation

With the spec in hand, you ask the agent for an **implementation plan**: which files it touches, in what order, what dependencies between tasks, what tests it intends to write. The plan isn't code; it's an intermediate document. You read it, discuss it, correct it if the agent misunderstood any constraint.

This step is where Traycer-style tools add the most value (chapter 8), because the plan's quality determines the quality of everything that comes after.

### Step 3 — Decompose into tasks

You take the plan and cut it into **small, sequential tasks, each with a "done" criterion**. A typical task should fit in a single focused agent session, with no ambiguity. If a task needs more than one session, it's badly cut.

Heuristic rule: if you can't describe the "after the task" state in one sentence, the task is too big.

### Step 4 — Implement task by task

The agent executes tasks one at a time. For each task: read the spec, read the task, write the tests the task asks for, write the code, run the tests, verify. If tests don't pass, iterate. If they pass, mark done and move on.

What matters here is that **each task is atomic with respect to the spec**: either it's fully done or it's reverted. There are no half-done tasks "to be fixed in another".

### Step 5 — Validate against the original spec

When all tasks are done, the agent (or you, or a second reviewer agent) takes the original spec and verifies that everything the spec asked is done, and that nothing the spec didn't ask for has been done. This phase is the easiest to skip and the most important: it's where acceptance criteria stop being text and become a checked-or-unchecked checklist.

## Iterative clarification: what the arXiv survey adds

There's a detail the [*Code Generation with LLM-based Agents*](https://arxiv.org/html/2508.00083v1) survey documents that's worth being in this cycle even though it doesn't appear explicitly in either of the two approaches above. Systems like **ClarifyGPT** and **TiCoder** introduce a phase **prior** to the spec: instead of taking the first prompt as truth, the agent asks iterative questions to surface ambiguities and gaps before they materialize as wrong code.

In practice, this means your phase 1 of the cycle **isn't linear**. It's a mini-loop within the loop: prompt → agent questions → answers → draft spec → more questions → final spec. Teams that treat phase 1 as ping-pong instead of dictation get substantially better specs, and this is independent of which tool you use.

This clarification phase connects directly with chapter 3's **boundaries** — particularly the *"ask first"* category: the questions the agent should ask before acting. The difference is that here the questions happen before the spec exists, not after.

## The lifecycle isn't linear

A final note almost always omitted in canonical descriptions: **the cycle is rarely linear in practice**. What really happens is in step 4 you discover something that invalidates a step-1 assumption, and you go back. Step 5 teaches you that the criterion you wrote wasn't verifiable and you have to rewrite it. Mid-task, you realize the spec has a hole.

This is **normal and desirable**. The difference between a healthy cycle and a pathological one isn't that the first has no backtracks; it's that the healthy one **updates the spec when it discovers it was wrong** instead of ignoring it and continuing to code. That's exactly the idea of the next chapter: **living specifications**.

## What comes next

Chapter 6 is about the difference between static specs (the ones that age badly) and living specs (the ones that stay useful because implementation feeds back into the spec). It's the piece that turns the lifecycle described in this chapter into a sustainable process instead of a startup ritual.
