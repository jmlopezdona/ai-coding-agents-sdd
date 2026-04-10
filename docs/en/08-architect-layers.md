# 8. Architect layers over agents: Traycer and the wrapper pattern

The previous chapter's tools share a feature: they're **standalone** SDD tools. You live inside them, their workflow is the workflow. There's another category growing in parallel that's worth understanding separately: tools that don't ask you to switch agent but **sit on top of the agent you already use**. We call them *architect layers*, and the most visible piece of this category today is **Traycer**.

## The question this category answers

When a team already uses Cursor, Claude Code, Codex or similar and knows them well, the friction of adopting Kiro or Spec-kit isn't only learning a new tool. It's **abandoning the tool where the team has hours, shortcuts, configurations and muscle memory**. For many teams, that cost is prohibitive even if the new tool is objectively better in some dimensions.

Architect layers solve that friction by not asking for it. Your agent stays Cursor or Claude Code or whatever you use. The layer inserts itself in the workflow at two specific points: **before** the agent (planning) and **after** the agent (verifying). In the middle, the agent works as always.

## What Traycer does concretely

According to its material and community discussion, Traycer adds three things a bare agent doesn't:

### 1. Elicitation

Before letting you talk to your main agent, Traycer runs a round of **questions** to surface requirements you'd probably have forgotten to mention. It's the chapter 5 "iterative clarification" phase incorporated as automatic mechanism instead of mental reminder.

The idea is direct: most prompts people give their agent are poorly specified, and most agent failures come from that poverty. If a tool automates the "wait, before we start tell me what should happen if...", the quality of everything after goes up.

### 2. Planning

Once the requirement is clarified, Traycer generates a **detailed implementation plan** — which files it touches, what dependencies between tasks, what order to follow — *before* the main agent writes code. The plan is reviewable. If it's wrong, you fix it. If it's right, you hand it down to the main agent as rich context, not as a loose prompt.

This directly addresses the chapter 1 *context drift* problem: the plan forces explicit blast radius ahead of time, instead of discovering it after the agent has already broken things it didn't look at.

### 3. Automatic verification

When the agent finishes, Traycer **compares what the agent did against the original spec/plan** and flags divergences. It's the chapter 5 validation phase, automated and mandatory, instead of "if I remember, I'll review it".

This post-code verification is exactly the gap chapter 7 identifies as unsolved by native SDD tools. Traycer closes it.

## The general pattern, beyond Traycer

Even if Traycer is the most visible piece today, the pattern is worth understanding abstracted from the concrete tool:

> **An architect layer wraps a coding agent with three things: requirement clarification before the prompt, explicit planning before the code, and automatic verification after the code.**

The pattern is independent of the tool and will appear in other forms. There will be more tools occupying this space. The architectural question for your team isn't "do I use Traycer?" — it's "is my flow missing any of these three hooks?". If the answer is yes, then it's worth looking for a layer that adds them, whether this one or others.

## A note on source bias

The most complete community discussion of Traycer comes from a r/vibecoding post that is **clearly promotional**. Its author declares it as "my best option" and the post structure is marketing's, not neutral analysis's. This doesn't mean the claims are false — the category exists and the pattern logic is solid — but it means it's worth tempering the original material's enthusiasm.

The honest way to evaluate Traycer (or any tool of the pattern) is: spend a week of real work with it, measure how much rework you save and how many additional prompts you need to fix what the tool didn't anticipate. If the difference is notable, continue. If not, back out at no cost.

## Layers vs. native tools: how to choose

| Question | If your answer is… | Look at… |
|---|---|---|
| Does your team already have years on Cursor/Claude Code? | yes | a layer (ch. 8) |
| Does your workflow suffer more from lack of plan than lack of process? | yes | a layer (ch. 8) |
| Want to adopt SDD from scratch with no preferred agent? | yes | native tool (ch. 7) |
| Want a global constitution and checkpoint discipline? | yes | Spec-kit (ch. 7) |
| Is your real problem organizing big features into steps? | yes | Kiro (ch. 7) |
| Curious about pushing the spec-as-source frontier? | yes | Tessl (ch. 7) |
| Is your bottleneck coordination between distinct roles? | yes | BMAD (ch. 7) |

The two categories aren't exclusive. A mature team can perfectly use Spec-kit for global constitution and boundaries discipline, and Traycer (or equivalent) over its daily agent for tactical plans and verifications. They're different layers of the same building.

## What architect layers **don't** solve

There's a real limit worth naming: architect layers **don't solve** the problem of keeping specs alive over months (chapter 6). They solve the tactical cycle of a concrete feature — clarify, plan, execute, verify — but not the sustained discipline of maintaining a spec updated as the code evolves and the team changes.

For that, layers are insufficient by design. Their work unit is the session, not the module's lifecycle. Maintenance discipline remains team work, and that work has a real cost (chapter 10).

## The hook to the harness

There's a sentence worth carving down because it connects this chapter to the trilogy's next course. The "architect layer over agent" pattern is technically **a piece of harness applied to SDD**. What Traycer does — intercepting inputs, planning, verifying outputs — is exactly the kind of wrapper the harness course develops for many other dimensions (tests, sandboxes, sensors, hooks).

Seen this way, Traycer isn't only an SDD tool: it's a concrete example of how a good harness improves SDD. And Kiro's hooks mentioned in chapter 7 are another example of the same principle: harness + SDD reinforce each other. Chapter 13 returns to this.

## What comes next

Up to here we've seen the cycle (ch. 5), living specs (ch. 6), and two categories of tools (ch. 7 and 8). In **Chapter 9** we go down to practice: how all this applies to three different kinds of work — new features, refactors, and bug fixes — because the optimal process isn't the same in all three cases.
