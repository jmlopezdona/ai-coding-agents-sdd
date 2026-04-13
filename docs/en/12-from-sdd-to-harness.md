# 12. From discipline to the software factory

If you've made it this far, you have SDD as a discipline: you know how to specify intent, maintain the bidirectional loop, modulate the process by task type, and you have judgment to not confuse it with bureaucracy. What's missing is understanding **where all this leads** when you stop depending on individual discipline and start building a system that works on its own.

## The autonomy progression

The natural path isn't "SDD and then something else". It's a continuous progression where the system takes on increasing responsibility and the human shifts from the center to the periphery:

### Level 1 — Human in the loop (pure SDD)

This is where you are now. The human writes specs, reviews the agent's output, maintains the bidirectional loop manually, and detects drift with their own eyes. It works, but it has a ceiling: **it depends on every team member applying perfect discipline, and perfect discipline doesn't scale**. If Alice goes on vacation for a week, specs go out of date. If nobody checks the loop, it breaks.

### Level 2 — Human on the loop (SDD + automation)

The human no longer executes each step — **they supervise a system that executes for them**. Automatic hooks fire validations when code changes. Sensors detect drift between spec and code and open issues without human intervention. Recurring agents keep specs alive. The human defines intent, reviews exceptions, and makes decisions the system can't make alone.

The operational difference is profound: at level 1, forgetting to update the spec is a discipline failure. At level 2, it's an infrastructure bug — and infrastructure bugs get fixed once.

### Level 3 — Human over the loop (software factory)

The human defines intent at a high level and the system handles the rest: decomposes intent into specs, implements, verifies, detects problems, and self-corrects. The human intervenes for exceptions and strategic decisions, not for the normal flow. Each iteration of the system makes it more autonomous and more reliable.

This isn't science fiction — it's the direction tools like Traycer ([chapter 7](07-native-sdd-tools.md#traycer)) are already pointing toward with their elicitation → planning → verification cycle. The difference is that at level 3, that cycle doesn't need a human supervising each session.

## The mechanisms that enable each jump

The progression isn't abstract. Each level is enabled by concrete mechanisms that connect directly to what you've learned in this course:

### Specs as persistent context

Everything the agent should know has to be materialized in the repo, not in Slack or in heads. SDD specs are a specialization of that principle: persistent context with a specific shape — *intent + constraints + whys + verifiable criteria*.

At level 1, specs are written and maintained by a human. At level 2, the system ensures specs reach the agent in the right form, at the right moment, with the right hooks. At level 3, the system generates and updates specs as part of its own cycle.

### Sensors that validate specs

Chapter 5's verification phase is a manual sensor: someone compares what the agent did with what the spec asked for. At level 2, that becomes infrastructure: tests comparing code against spec, recurring agents detecting drift, linters validating that constraints are still respected.

This is the technical way to close the chapter 6 bidirectional loop. Without automation, the loop depends on human discipline ("remember to update the spec"). With automation, the loop is automatic ("the sensor detected code and spec diverged and opened an issue to reconcile them").

### Hooks that eliminate "remember to..."

The hooks that fire on file save — like Kiro's in chapter 7 — are the prefiguration of a more general principle: **repo events that trigger automatic actions** (regenerate docs, update indexes, validate invariants, refresh the living-specs loop).

The operating rule: **anything that in pure SDD requires "remember to do X when Y happens" should become an automatic hook**. If something stays in "remember", it'll be forgotten. Each hook you add is a step from level 1 to level 2.

### SDD tools as factory prototypes

Tools like Traycer are technically **level 2 prototypes**: they intercept inputs, plan, verify outputs. It's the guides-and-sensors dynamic applied to the concrete cycle of a session with an agent.

If the full factory seems too much to start with, an SDD tool over your agent is the miniature version of the idea: try what it feels like to have guides and sensors wrapping the agent, before committing to building your own infrastructure.

### Anti-patterns that only automation solves

Several [chapter 11 anti-patterns](11-anti-patterns.md) are specifically hard to avoid with human discipline alone:

- **Spec-as-Theatre** dies when there's an automatic sensor measuring adherence: if the spec isn't respected, the system says so.
- **The eternal spec** dies when there's a recurring agent detecting drift and opening automatic issues.
- **Trusting the spec more than the code** becomes rare when there are automatic validators that on disagreement force you to investigate before accepting the merge.
- **The review burden** gets reduced when hooks generate the repetitive markdowns automatically and human attention is freed for what contributes judgment.

In other words: **a significant part of what kills bad SDD is exactly what automation is designed to solve**.

## Signals it's time to level up

Not every team adopting SDD needs to jump to level 2 immediately. There are specific signals:

- **Sustainability depends on individuals.** If one person goes away for a week and specs go out of date, you need mechanism, not more discipline.
- **The bidirectional loop works when someone takes care of it and breaks when nobody looks.** That's a sign it needs to go from human process to infrastructure.
- **[Chapter 11 anti-patterns](11-anti-patterns.md) reappear periodically** even though the team knows them. Individual awareness doesn't scale; infrastructure does.
- **Chapter 7 tools fall short** because your system needs multiple layers (sensors + hooks + sandboxes + observability) that no single tool covers.

If you recognize two or more, it's time to start building infrastructure. The [harness engineering guide](https://jmlopezdona.github.io/ai-coding-agents-harness/) develops the concrete mechanisms to take that step.

## A goodbye without promises

An honest guide doesn't end with "and now your life will change". It ends with two simple truths:

First, **well-done SDD improves the agent's output and the project's sustainability**. This is documented in the practice of teams that have applied it for a while and shows in concrete metrics: less rework, less drift, better onboarding. It's not magic, but it's real.

Second, **bad SDD is indistinguishable from bureaucracy and sometimes worse**. This is also documented and is why chapter 9 exists. Applying it without judgment turns the discipline into its own bottleneck, and within weeks the team goes back to vibe coding with an additional layer of cynicism.

The difference between the two isn't the tool. It's the team's judgment to modulate the formality level by problem, keep whys alive, and understand that manual discipline is only the first step — the goal is to build a system where quality is a consequence of infrastructure, not individual heroism.

If you take that, you take the best this guide can offer.

---

*This chapter closes the course. If you want to take the next step toward automation, read the [harness engineering guide](https://jmlopezdona.github.io/ai-coding-agents-harness/). And if you're missing the foundations on what an agent is and how to drive one, the [fundamentals guide](https://jmlopezdona.github.io/ai-coding-agents-fundamentals/) is the natural starting point of the trilogy. All chapters here are self-contained: jump to whichever one you need when the problem it solves shows up.*
