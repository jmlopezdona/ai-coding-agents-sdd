# 8. Patterns of application: features, refactors and bug fixes

One of the easiest traps to fall into with SDD is **applying the same process to everything**. It's the exact trap Fowler points out in Kiro in chapter 6: treating a three-line bug fix as a multi-story feature. The process doesn't scale uniformly because problems aren't uniform. This chapter walks through the three most common work patterns — new feature, refactor and bug fix — and describes **how to modulate the SDD cycle** for each.

The general rule to keep in front of you: **the spec's weight should be proportional to the cost of the change if it goes wrong**. A critical, broadly-scoped feature deserves a full spec. A trivial bug fix deserves, at most, a note.

## Pattern 1 — New feature

This is the case SDD was designed for and where it pays the most. There's intent to capture, constraints to write, acceptance criteria to enumerate, edge cases to dig out before touching code.

### The applied cycle

1. **Specify.** Full chapter 3 template: objective, no-goals, constraints, criteria, whys, boundaries. Write this *before* talking to the agent about code.
2. **Clarify.** Pass the spec to the agent and ask it to ask you questions. This phase is rarely skipped for free: the questions the agent asks you are exactly the ambiguities that would have broken the code later.
3. **Plan.** The agent proposes which files it touches, in what order, what tests it writes. You review. If the plan is wrong, you fix it *here*, not later.
4. **Implement task by task.** Each task is atomic with respect to the spec. Each task brings its tests.
5. **Verify.** Compare the original spec with the final code. Acceptance criteria become a checked checklist.
6. **Bidirectional update.** Any decision made during implementation that wasn't in the original spec gets promoted to the spec.

### Typical errors of this pattern

- **Skipping step 2** and barreling ahead with the first spec you write. Elicitation looks like cost; it's actually savings.
- **Confusing the plan with the code**. The plan is debatable and cheap; the code is expensive to revert. Argue in the plan, not in the diff.
- **Forgetting step 6** and leaving the spec frozen in its initial version. This is the difference between spec-first and spec-anchored from chapter 2.

### When this pattern **isn't** for you

- Experimental features where the intent is exactly "see what comes out". If you don't have clear intent before starting, there's nothing to specify yet.
- Research spikes. The spec comes *after* the spike, as conclusion, not before.

## Pattern 2 — Refactor

The refactor is the pattern most teams confuse how to apply SDD to. The temptation is to treat it like a new feature — "let's specify how we want the module to look" — and that almost always leads to an inflated spec nobody respects. The refactor deserves a different process.

### The right starting question

> *What external invariants must remain true after the refactor?*

This is the thesis question of SDD applied to refactors. Not "how does the code look inside" — that's the agent's call — but "what externally observable guarantees can't be broken". Those invariants are your spec. Everything else is implementation freedom.

### The applied cycle

1. **Capture invariants.** A short list of things that must remain true: the public API doesn't change, existing tests still pass, the error contract is preserved, performance doesn't degrade more than X%.
2. **Characterize the current state.** If existing tests don't cover the invariants well, write the missing tests *before* touching anything. A refactor without a test net is vibe coding in disguise.
3. **Plan the refactor.** Small steps, each revertible, no step breaks invariants while running.
4. **Implement step by step, with green tests at each step.**
5. **Verify against the invariants.** Same tests, same APIs, same external observability.

### What you don't write

- You don't write "new design" because new design is the output, not the input.
- You don't write behavior acceptance criteria because behavior *must not change*. The invariants are your criteria.
- You don't write no-goals like in a feature; in a refactor the principal no-goal is implicit ("introduce no functional changes") and you only spell out exceptions.

### Typical errors

- **Doing the refactor and "while we're at it" fixing two bugs and adding a small improvement.** No. A refactor that changes behavior is no longer a refactor — it's a feature, and gets specified as one.
- **Skipping step 2.** If existing tests don't cover the invariants, refactoring is Russian roulette. Characterization is the most underestimated phase of a refactor.

## Pattern 3 — Bug fix

The bug fix is the pattern where full SDD is counterproductive. A three-line bug fix with full template, iterative clarification and formal plan is exactly the bureaucracy chapter 9 critiques. Don't do it.

But "don't do it" doesn't mean "do vibe coding". It means the bug fix has its own very light version of the cycle, and it's worth naming it.

### The applied cycle (minimum version)

1. **Reproduce the bug** with a failing test. This is your minimum viable spec: the passing test is the acceptance criterion, all in one.
2. **Diagnose root cause.** If the agent finds it, perfect. If not, do it yourself.
3. **Apply the fix** with the red test in front.
4. **Check the test passes and the others too.**
5. **A short note in the relevant spec** (if there is one) explaining the bug and the fix. One line, not a paragraph.

### When the bug fix turns into pattern 1 or 2

Sometimes, while diagnosing a bug, you discover it's not local — it's a symptom of an architectural problem. At that point you **escalate**: stop the bug fix and open a feature or refactor spec. The signal this is happening is that the fix asks you to touch more than two unrelated files, or there are already three different bugs pointing to the same place.

This "bug → refactor" transition is one of the most valuable moves in well-applied SDD and almost never explicitly taught. The heuristic: **if the fix is mechanical, it's a bug fix; if the fix forces you to understand five things you didn't understand yesterday, it has stopped being a bug fix**.

## A matrix to pick the right pattern

| Question | Yes → pattern |
|---|---|
| Is there new intent to capture? | Feature |
| Does observable behavior change? | Feature |
| Should behavior stay the same and only structure change? | Refactor |
| Is there a concrete failing test to fix? | Bug fix |
| Does the fix touch five unrelated files? | Refactor (or feature) |
| Can you say in one sentence what has to change? | Bug fix |

If you hesitate between two patterns, choose the lighter one and escalate if you discover it's too small. Late escalation hurts less than early over-specification.

## The meta-pattern: read size before choosing process

If you keep one idea from this chapter, let it be this: **before applying SDD, read the problem's size**. Not its apparent size — its real size, almost always discovered by asking two questions:

1. *How many distinct places in the system do I need to touch for this to work?*
2. *How much does it hurt if this goes wrong and nobody notices for a week?*

The answers to those two questions, multiplied, tell you how much SDD you need. Lots × lots = pattern 1 with everything. Little × little = pattern 3 minimal. Intermediate combinations fall in pattern 2 variants.

The chapter 6 tools (Kiro especially) tend to apply pattern 1 to everything, and that's why they sometimes feel heavy. The modulation criterion you're learning here is what no tool offers today: it's contributed by the engineer, and it's exactly the kind of judgment that distinguishes a team using SDD from one suffering SDD.

## What comes next

We arrive at the most uncomfortable chapter of the course. **Chapter 9** is the honest critique of SDD: where it fails, what it costs, why some critiques are fair, and what to do about it. If you've been reading the course convinced SDD is the answer, that chapter is designed to unsettle you a little. It's deliberate and necessary.
