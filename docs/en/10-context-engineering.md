# 10. Context engineering as alternative

The previous chapter closes with six serious critiques of SDD. It would be unfair to leave you there, in critique without alternative. Fortunately, the same critics — [Isoform](https://isoform.ai/blog/the-limits-of-spec-driven-development) in particular — propose a different idea about where intent and *whys* should live. They call it **context engineering**, and this chapter presents it without sweetening but also without dramatizing the difference with SDD.

> *Capture intent and constraints from discussions, update context iteratively as understanding evolves, and explicitly preserve decision rationales within code itself.*
> — [Isoform](https://isoform.ai/blog/the-limits-of-spec-driven-development), *The Limits of Spec-Driven Development*

That sentence sums up the shift.

## The essential difference from SDD

SDD puts intent in **specs that are documents parallel to the code**. What Isoform proposes is distributing that intent across **artifacts with varying levels of coupling to the code itself**: from inline comments (maximum coupling) to ADRs (low coupling, similar to a spec).

This isn't a binary "inside vs outside the code" dichotomy. It's a **coupling gradient**:

| Artifact | Coupling to code | Desynchronization risk |
|---|---|---|
| Inline comment | Maximum — same file, same line | Very low |
| Commit message | High — tied to the exact diff via `git blame` | None (immutable) |
| AGENTS.md / CLAUDE.md | Medium — in the repo, orientational | Low if kept short |
| ADR | Low — separate document about decisions | Medium (same as a light spec) |
| Spec in repo | Low — separate document about implementation | Medium-high |
| Spec in Confluence/Notion | Minimal — outside the repo | High |

We need to be honest: an ADR at `docs/adr/0005-auth-strategy.md` is a separate document describing decisions about code that lives elsewhere — **it can desynchronize exactly like a spec**. What changes between an ADR and a spec isn't its adherence to the code, but **what it captures** (specific decisions vs complete implementation) and **its ambition** (a concrete why vs a system source of truth).

Isoform's proposal isn't "everything stuck to the code" — it's **preferring artifacts at the top of the gradient** (comments, commits) and using those at the bottom (ADRs) only for decisions, not for describing the entire system. The practical consequences:

1. **Less maintenance surface.** Chapter 9's *maintenance tax* doesn't completely disappear — ADRs can also go stale — but it's reduced because the most coupled artifacts (comments, commits) can't desynchronize by construction.
2. **The most critical whys travel with the code.** An intent comment gets deleted when the line gets deleted. A commit message is immutable. The whys living there aren't lost.
3. **There's no false illusion of completeness** because there's no master document giving a sensation of coverage. Coverage is exactly the code's coverage.

An important observation: **a spec that lives in the repository is also a repo artifact**, versioned with git, reviewable in PRs. The boundary between "spec" and "project artifact" blurs. The real difference isn't where the file lives, but the degree of coupling to the code it describes and the ambition of what it tries to capture.

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

| Dimension | SDD | Code-embedded context |
|---|---|---|
| Where intent lives | Specs (parallel documents) | Distributed across the coupling gradient |
| Synchronized maintenance | Yes, explicit | Reduced but not eliminated (ADRs also age) |
| Captures whys | Sometimes (depends on discipline) | Always, by design |
| Granularity | Per feature/module | Per decision |
| Visible in code review | Not directly | Yes, in every PR |
| Supports spec-as-source | Yes (Tessl) | No |
| Adoption cost | High | Low |
| Practice maturity | Forming | Decades (ADRs are from 2011) |

The two approaches solve the same problem — chapter 1's context collapse — with different philosophies. SDD bets on **explicit persistence in parallel documents**; code-embedded context bets on **distributing intent across artifacts with high coupling to the code**.

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

## An important note on the term "context engineering"

We need to be honest about a term collision. When Isoform talks about "context engineering" in their article, they mean something specific: **embedding intent, decisions and constraints inside the code itself and its adjacent artifacts** (ADRs, commits, intent comments, AGENTS.md). It's a documentation practice tied to the code.

But the term **context engineering** has a much broader and earlier meaning in the AI community. Popularized by figures like Andrej Karpathy and Simon Willison, it refers to the **general discipline of designing everything that reaches the LLM at the moment of action**: system prompts, few-shot examples, RAG results, tool definitions, conversation history, and any other input that conditions the model's response. It's the natural evolution of "prompt engineering" when the input stopped being a single prompt.

The two meanings are not the same:

| | Context engineering (general) | What Isoform proposes |
|---|---|---|
| **Scope** | All LLM input | Repo artifacts only |
| **Includes** | System prompts, RAG, tools, few-shot, specs | ADRs, commits, comments, AGENTS.md |
| **Goal** | Agent acts well | Intent survives in code |
| **Who does it** | Whoever configures the agent | Whoever writes the code |

What Isoform proposes would be more precisely called **code-embedded context**: relevant information beyond the code itself — the whys, the constraints, the decisions — but adhered to the code rather than living in an external document like a spec. It's a subset of general context engineering, not its synonym.

That said, both SDD and code-embedded context are **concrete strategies within general context engineering**. SDD chooses external specs as the main mechanism for feeding the agent's context; code-embedded context chooses the code itself and its adjacent artifacts. The choice depends on the team's context, and as we'll see in the following synthesis, they aren't mutually exclusive.

## What comes next

**Chapter 11** is the shortest in the course and the most operative: a list of **anti-patterns** of SDD you'll see in real teams, with name, symptom and correction. It's the part you'll want to print and put on the team wall.
