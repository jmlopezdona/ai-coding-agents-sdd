# 1. From vibe coding to SDD: context collapse and context drift

Before talking about specs we have to understand exactly what fails when there are none. The public conversation about the limitations of coding agents tends to stay in comfortable places — "they hallucinate", "they don't understand context", "the model isn't good enough" — and almost never diagnoses the two specific failures SDD exists to solve. Let's name them.

## Vibe coding, the default practice

We call *vibe coding* the way most people use coding agents today: you open a session, you write in natural language what you want ("add an endpoint to upload photos"), you look at what comes out, you iterate a couple of times with additional prompts, and you either accept it or throw it away. It's agile, it's fun, and for prototypes it works reasonably well.

And it breaks systematically as soon as the project stops being a prototype. The reason isn't that the agent is bad. It's that you're operating in a regime where two specific failures become inevitable.

## Failure #1: context collapse

**Context collapse** is a failure *between* sessions. Every new conversation with the agent starts from zero. The knowledge the agent "had" yesterday isn't there today. The project's naming conventions, the error-handling patterns the team decided two sprints ago, the unwritten rule that the `legacy/` module isn't to be touched because it's being deprecated — all of that is invisible to a fresh agent.

In a five-file project it doesn't matter. In a two-hundred-file one, it matters enormously. When it can't find explicit information, the agent does what any model trained on the internet does: it extrapolates from general training. And that extrapolation almost never matches your project, because your project has history, decisions, and commitments that aren't in training data.

The typical symptom: the agent produces syntactically correct, generally idiomatic code that is **stylistically foreign to the rest of the repo**. Names that don't fit. Error-handling patterns different from the existing ones. A new layer where your team would never have put one. Things a human who knows the project would catch in five seconds and that the agent, with no context, can't even suspect.

## Failure #2: context drift

**Context drift** is different and worth not confusing with the previous one. It happens *within* a single session, when the agent fixes something in one file and breaks three others it didn't look at. It's not that it "forgot" the context — its attention window doesn't cover the real blast radius of the change.

You ask it: "rename this function". The agent renames it in the file where it found it. But the function is imported from nine other places, and the agent didn't look there because its search closed when it found the first match. Result: nine breakages that will appear when you run tests — if you have tests — or in production if you don't.

Context drift is operational: it happens even if you gave the agent every bit of context that fit in the window. It's a coverage failure, not a memory failure.

## Why both failures together are lethal

Each failure is manageable on its own. Together they produce a very concrete pattern that is the signature of vibe coding in production:

1. You ask the agent for a feature.
2. The agent implements it reasonably, without seeing conventions (collapse) or lateral dependencies (drift).
3. You approve the change because "it looks good".
4. A week later, another change breaks this. Another agent fixes it. More context lost.
5. Repeat twenty times.
6. The repo is now a mosaic of styles, with a pile of subtle bugs nobody knows how got introduced.

That's vibe coding at scale. There's no moment where something breaks spectacularly. There's continuous erosion that you only notice when you try to make a big change and discover the repo has become fragile in a way that's hard to explain.

## Other symptoms worth learning to name

Beyond the two central failures, vibe coding produces a handful of symptoms the literature has learned to label:

- **Silent pattern regression**: every new feature is slightly different from previous ones because the agent didn't see the previous ones.
- **Wasted tokens**: endless back-and-forth prompts to fix errors a spec would have prevented. It's the *economic* cost of vibe coding and almost nobody measures it.
- **Loss of traceability**: a year from now, no one knows why this logic exists. The answer lives in a chat that no longer exists.
- **Happy path bias**: with no spec enumerating edge cases, the agent optimizes for the happy path. Timeouts, race conditions, malformed inputs don't appear in generated code unless you explicitly ask for them.

Each of these symptoms is individually fixable. What SDD proposes is to attack all of them at once with a single structural move: **shift the source of truth from prompt to persistent artifact**.

## The inflection point

The moment it's worth leaving vibe coding behind is very concrete and many teams cross it without noticing: when you start spending more time *fixing* what the agent produces than you would have spent writing it by hand. At that point, the agent has stopped being a multiplier and become a rework generator.

The metric isn't perfect — a bad day doesn't count — but as a sustained heuristic over time, it's the most reliable signal that you need structure. And the structure isn't "more prompts" or "more expensive model". It's a spec.

## What a spec does, exactly

A spec, in the most minimal possible sense, is **a persistent artifact that captures intent and constraints, and that the agent reads before acting**. That's it. It doesn't have to be long. It doesn't have to be exhaustive. It has to exist, live in the repo, and be read.

What this resolves, row by row:

| Failure | How the spec resolves it |
|---|---|
| Context collapse | The spec persists across sessions; context stops being ephemeral. |
| Context drift | The spec defines invariants that post-code verification can compare against. |
| Pattern regression | The spec references project patterns; the agent reads them, doesn't invent them. |
| Wasted tokens | Fewer iterations because the first one has better context. |
| Loss of traceability | The spec outlives the chat. A year later it's still there. |
| Happy path bias | Edge cases are explicitly enumerated, not assumed. |

It's not magic. It's simply moving what was implicit in your head to a place the agent can read. The price you pay is writing the spec. The price you avoid is everything above.

## What comes next

If you accept that vibe coding has a ceiling and that moving the source of truth to the artifact resolves most symptoms, the next question is *what kind of spec* and *with how much commitment*. That's the question of **chapter 2**, where we introduce the specification spectrum: three levels of SDD adoption, with very different tradeoffs. Most teams believe they're at one level and are actually at another, and understanding that is the most important decision you'll make in this course.
