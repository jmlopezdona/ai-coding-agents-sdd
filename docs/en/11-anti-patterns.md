# 11. SDD anti-patterns

A short list of the most expensive errors seen in teams adopting Spec-Driven Development. They're not here to scare you but so when you recognize them in your own process — and you will — you know how to stop them before they become habit.

Each anti-pattern follows the same format: **name**, observable symptom, why it happens, and the correction.

---

## 1. Spec-as-Theatre

**Symptom.** The team writes full template specs, commits them to the repo, and never reads them again. Neither the agent nor humans. Specs exist to "show we do SDD" in process reviews, not to be used.

**Why it happens.** Top-down adoption without conviction. Someone decided "we're going to do SDD" and the team's response was to comply with the minimum ritual. Specs are tribute to authority, not tools.

**Correction.** Either specs are used or they're eliminated. There's no middle ground. The signal that the spec is used is that when there's drift between code and spec **someone discusses it in a PR**. If that discussion never appears, specs are theater and it's better to acknowledge it.

---

## 2. Big Spec Up Front

**Symptom.** Before writing a single line of code, two weeks are spent on an exhaustive 800-line spec. By the time the team starts implementing, half the assumptions have already changed.

**Why it happens.** Confusion between SDD and waterfall. People coming from heavy processes apply heavy reflexes to SDD and produce giant specs reminiscent of design docs of yore.

**Correction.** Light, living specs. The chapter 3 template fits on one screen. If your spec needs more than that, almost always you're describing two features that should be separated, or specifying things the agent can already infer from the code's context.

---

## 3. Spec without whys

**Symptom.** The spec describes the *what* in detail — schemas, criteria, contracts — but doesn't say why decisions were chosen. Six months later, no one knows whether a spec decision is still valid or is legacy from a constraint that no longer exists.

**Why it happens.** Whys are the easiest section to cut when the team is in a hurry, and short-term it doesn't show. Medium-term, it's what kills the spec's value.

**Correction.** Whys are mandatory section, not optional. In PR reviews, a spec without whys is treated as incomplete. And if a decision has no clear why, that itself should block the merge — because it means it hasn't been thought through.

---

## 4. Theater spec with fake anchoring

**Symptom.** The team declares spec-anchored (chapter 2) but has no automatic mechanism detecting drift between spec and code. The two live in parallel, both get modified, and nobody knows when they stopped matching.

**Why it happens.** Ambition without infrastructure. Aspiring to spec-anchored is easy; implementing the anchoring (tests, validators, recurring agents) requires real investment the team didn't make.

**Correction.** Honesty about the real level. If you have no automatic anchoring, you're in spec-first, not spec-anchored. Call it by its name. And if you want to level up, first invest in the mechanism, then declare the change.

---

## 5. Single pattern for any task

**Symptom.** Every change — feature, refactor, three-line bug fix — goes through the same complete process: template spec, formal plan, task decomposition, validation. The team complains SDD "is too slow" and is right.

**Why it happens.** Lack of modulation. Adoption was done without teaching the difference between chapter 8 patterns.

**Correction.** Process proportional to risk. Trivial bug fixes don't deserve a spec; big features do. Chapter 8's rule — *spec weight proportional to the cost of the change if it goes wrong* — isn't optional, it's what makes the process sustainable.

---

## 6. The spec that devours code in review

**Symptom.** PRs include so many markdown changes (specs, plans, tasks, checklists) that reviewing the code itself gets buried. Reviewers read markdown for 20 minutes and look at the code diff for 3.

**Why it happens.** Fowler identifies this as one of Spec-kit's specific problems: the process generates more artifacts to review than real code. If unbounded, "review" turns into "review markdown".

**Correction.** Measure markdown/code ratio in PRs over a sprint. If markdown systematically wins, simplify templates or automate generation of repetitive artifacts. Human review should focus where judgment adds value — code and new decisions — not where a linter could pass.

---

## 7. The eternal spec

**Symptom.** A spec is written at the start of the feature and never updated again. Code evolves, the spec stays. Six months later, reading the spec actively misleads: it says things the code no longer does.

**Why it happens.** Absence of bidirectional loop (chapter 5). The definition of "done" doesn't include updating the spec, so the incentive is not to.

**Correction.** "Done" = "code merged **and** spec updated". Without the second, the task isn't done. This sounds harsh and it is, but it's the only way the bidirectional loop survives a team under pressure.

---

## 8. Trusting the spec more than the code

**Symptom.** When there's disagreement between what the spec says and what the code does, the team assumes the spec is right and "the code is wrong". This reflex is exactly the opposite of correct.

**Why it happens.** A dense, well-written spec inspires confidence, and confidence becomes authority. But the authority is unjustified: the spec might be wrong, out of date, or written with assumptions that no longer apply.

**Correction.** Code is the operational source of truth. When there's disagreement, **investigate** instead of assuming. Sometimes the code is wrong and the spec is right. Sometimes — more often than the team's ego will accept — it's the spec that's wrong.

---

## 9. Promoting prematurely to spec-as-source

**Symptom.** The team, excited by Tessl or by the generation idea, declares "now we edit the spec and regenerate the code". Two weeks later there are manual changes in files marked `// GENERATED — DO NOT EDIT`, and the promise has deflated.

**Why it happens.** Spec-as-source today is still viable only in narrow domains (chapter 2). Applying it to general-purpose software clashes with LLM non-determinism and edge cases the spec didn't anticipate.

**Correction.** Spec-as-source is experimentation, not production, except in domains where it's worked for decades. If you want to explore it, do it on an isolated, reversible surface. Don't bet the critical path on it.

---

## 10. SDD as excuse not to talk

**Symptom.** The team stops discussing decisions because "it's in the spec". The spec is used as substitute for the human conversation about trade-offs. Questions die with "read the spec".

**Why it happens.** Writing is more comfortable than discussing, especially for distributed teams. An elaborate spec starts working as a decree instead of as a proposal.

**Correction.** The spec is a starting point for conversation, not its end. If an important spec decision hasn't been questioned in six months, it's not a quality signal — it's a sign the spec has killed the conversation that should be alive.

---

## 11. SDD applied to problems that aren't SDD problems

**Symptom.** The team adopts SDD expecting it to solve problems that are actually about something else: poorly defined product, poorly thought architecture, inexperienced team, broken deploy infrastructure. The discipline doesn't move the needle because it wasn't the right lever.

**Why it happens.** Methodology reification. "If SDD is good, my problem must be solvable with SDD." Not always.

**Correction.** Diagnose before treating. Ask what's failing exactly before deciding what methodology to apply. If the problem is product or infrastructure, no spec will fix it.

---

## 12. Generated spec that's pseudocode in disguise

**Symptom.** The team uses an agent to generate specs from user stories, features, or architectural components. The resulting specs are full of classes, methods, function signatures, API payloads, and inline business rules. They look thorough. When implementation arrives, the team discovers that (a) it's implementing twice, because the decisions were already made in the "spec", and (b) human review of those specs is as expensive as reviewing code that doesn't exist yet — but without the safety net of tests or actual execution.

**Why it happens.** When you ask an LLM to "generate a spec", the model optimizes for *apparent coverage* and drops to the level of detail where it's comfortable: code. That's where its training corpus gives it the most confidence. Climbing to the right level for a spec — intent, constraints, whys — requires abstraction, which is exactly what the model does worst. The result is predictable: pseudocode in markdown, dressed up as a design doc.

On top of this comes the team's own bias: a generated spec with 600 lines, classes, methods and diagrams *looks* rigorous. The tired reviewer assumes that something so detailed must be well thought out. And the "spec" passes the filter without anyone really checking whether it captures intent or just describes one of many possible implementations. Chapter 3 develops this anti-pattern in its section *"What if a generator agent writes the spec?"*; here we just give the operational corrections.

**Correction.** Three concrete rules:

1. **If your spec mentions function signatures, class names or payload structures, delete them.** The spec talks about what guarantees the system must meet, not how it's built. If after deleting them the spec is empty, you didn't need a spec — you needed well-written code (ch. 10).
2. **Use the agent as a draft generator only for the *what*, not the *how*.** The agent can start the objective, acceptance criteria and a candidate list of non-goals. The human has to put in, no exceptions, the whys, the real non-goals and the tacit constraints. What the agent doesn't know, it invents or omits — and invented whys are worse than missing whys.
3. **Measure the implementation-detail / intent-detail ratio of your specs.** If more than 30% of the spec describes code structures, you're writing pseudocode, not a spec. It's a sign that the generator agent (or you) has dropped a level of abstraction that breaks the document's purpose.

The simple test: a good spec survives **two different implementations** by the same team and is still valid for both. A spec generated as pseudocode doesn't — it describes *one* specific way of implementing, and if the code drifts even slightly it's already in drift. If your spec can't pass that test, you don't have a spec; you have an implementation sketch you call a spec.

---

## How to use this list

Once a quarter, in a retro, read this list aloud and ask the team: *do we recognize any of these in ourselves?* People, contrary to what ego promises, almost always recognize one or two. And recognizing them is half of stopping them.

If you recognize three or more, **the problem isn't one of the anti-patterns; it's the whole adoption of SDD**. Go back to chapter 9 and 10 and rethink whether SDD is the right tool for your situation.

## What comes next

**Chapter 12** closes the course connecting SDD with the next step: the harness. The exact points where the discipline couples with harness engineering, and why the two together produce much more than the two apart.
