# 10. Context engineering as alternative

The previous chapter closes with six serious critiques of SDD. It would be unfair to leave you there, in critique without alternative. Fortunately, the same critics — [Isoform](https://isoform.ai/blog/the-limits-of-spec-driven-development) in particular — propose a different idea about where intent and *whys* should live. They call it **context engineering**, and this chapter presents it without sweetening but also without dramatizing the difference with SDD.

> *Capture intent and constraints from discussions, update context iteratively as understanding evolves, and explicitly preserve decision rationales within code itself.*
> — [Isoform](https://isoform.ai/blog/the-limits-of-spec-driven-development), *The Limits of Spec-Driven Development*

That sentence sums up the shift.

## The essential difference from SDD

SDD puts intent **outside the code**, in specs the agent reads. Context engineering puts intent **inside the code and the directly readable artifacts** of the project: structured comments, ADRs (Architecture Decision Records), rich commit messages, README notes and, yes, occasional instructions in `AGENTS.md`-style files that live in the repo but **without pretending to be a parallel source of truth**.

The difference looks small but has three deep consequences:

1. **There's no second system to maintain.** Decisions live where the code lives, so you can't desynchronize them because they're the same thing. Chapter 9's *maintenance tax* disappears by construction.
2. **Whys aren't lost** because they're stuck to the code that implements them. Remove the logical unit and the why goes with it.
3. **There's no false illusion of completeness** because there's no master document giving a sensation of coverage. Coverage is exactly the code's coverage.

## What context engineering does

Context engineering isn't "do nothing". It's a set of concrete practices, all oriented to **leaving readable traces inside the system itself**:

### 1. ADRs (Architecture Decision Records)

Each important architectural decision is captured as a short ADR: context, decision, alternatives considered, consequences. Lives in `docs/adr/` or equivalent, versioned as code. It doesn't describe the whole system — it describes **decisions**, one per document, with date and owner.

The difference with a spec: a spec says "the system does this"; an ADR says "we chose to do it this way for these reasons". ADRs are bare whys, without the whats. And when an agent reads an ADR before touching the corresponding module, it gets exactly what Isoform's critique said specs didn't provide.

### 2. Commit messages that explain

A rich commit message — not "fix" or "update" but three sentences saying *what* changes, *why* and *what side effect* — is context engineering in its cheapest form. `git blame` becomes an intent retrieval system. Any future agent asking "why this line?" has the answer one command away.

### 3. Comments about intent, not mechanics

Traditional comments explain *what* the code does. Context engineering asks for comments that explain *why* the code does what it does, especially when the decision has a non-obvious reason.

```python
# We don't use exponential retry here because the upstream
# client has its own backoff and the two concatenated
# generate duplicate timeouts.
# (Post-incident decision 2025-11-12.)
```

That comment is pure context engineering: it lives with the code, explains a why, and no future agent will "clean up" the retry by mistake because the reason is right there.

### 4. AGENTS.md (or CLAUDE.md) as map, not as spec

A short `AGENTS.md` file at the repo root, orienting the agent about **where things are** and **what conventions are followed**, is context engineering done well. It's not a system spec; it's an orientation map for someone arriving from outside. It ideally fits on one screen.

The line between "AGENTS.md as map" and "AGENTS.md as giant spec" is exactly the line between context engineering and bad SDD. When AGENTS.md grows beyond orientation and starts describing the system, you're sliding into spec discipline by the back door, with all its costs.

## Where it resembles SDD and where it doesn't

To avoid making them seem like irreconcilable enemies — they aren't — it's worth tabulating where they coincide and where they differ.

| Dimension | SDD | Context engineering |
|---|---|---|
| Where intent lives | External specs | Inside code and repo |
| Synchronized maintenance | Yes, explicit | Not required by construction |
| Captures whys | Sometimes (depends on discipline) | Always, by design |
| Granularity | Per feature/module | Per decision |
| Visible in code review | Not directly | Yes, in every PR |
| Supports spec-as-source | Yes (Tessl) | No |
| Adoption cost | High | Low |
| Practice maturity | Forming | Decades (ADRs are from 2011) |

The two approaches solve the same problem — chapter 1's context collapse — with different philosophies. SDD bets on **explicit persistence**; context engineering bets on **substrate adherence**.

## When to choose context engineering over SDD

There are situations where context engineering is objectively the better option:

- **Small teams** where the cost of maintaining external specs is disproportionate.
- **Discovery-phase projects** where intent changes faster than a spec can be written.
- **Mature codebases with good conventions** where most context already lives in code and only the whys need capturing.
- **Teams that already practice disciplined ADRs** and would prefer not to add a second documentary system on top.

And situations where context engineering falls short:

- **Regulated systems** where the separate spec is an audit requirement, not a stylistic choice.
- **Coordination across multiple teams** where you need a shared artifact not buried in commits.
- **Large features with broad scope** where the formal upfront plan adds more than scattered ADRs.

## The honest synthesis: use both

One thing many teams discover after reading both positions is that **they aren't mutually exclusive**. The mature team ends up doing a mix:

- **ADRs and rich commit messages as base layer**, always, for everything. This is pure context engineering and is free (or very cheap).
- **Light specs in chapter 3 style** only for big features with broad scope and long life, following chapter 8's pattern 1.
- **Living specs with bidirectional loop** only for modules where the maintenance cost amortizes through criticality.
- **None of that** for minor refactors and trivial bug fixes.

This may sound like "do whatever you want", and it's exactly the opposite. It's **deliberate modulation** of the formality level according to the change's cost and risk. It's the skill that distinguishes a team applying SDD well from one suffering it universally.

## A note on the term "context engineering"

The term is gaining traction precisely because it captures something SDD doesn't: the real battle isn't writing specs, it's **managing the context** that reaches the agent at the moment of action. And context can come from many places: specs, ADRs, commits, comments, AGENTS.md, codebase searches, RAG over the repo, and retrieval tools. Calling all of that "context engineering" is honest about what's really going on.

SDD is a **concrete strategy** within context engineering: the strategy that picks external specs as the main mechanism. And like any strategy, it has contexts where it shines and contexts where it gets in the way. Seen this way, the chapter 9 and this chapter's positions aren't enemies: they're two points on the same spectrum of solutions to the same problem, and the choice depends on the team's context.

## What comes next

**Chapter 11** is the shortest in the course and the most operative: a list of **anti-patterns** of SDD you'll see in real teams, with name, symptom and correction. It's the part you'll want to print and put on the team wall.
