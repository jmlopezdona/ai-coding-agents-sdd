# 3. Anatomy of a good specification

There are many ways to write a spec badly and rather few to write one well. This chapter tries to distill what distinguishes a useful spec for an AI agent from one that's ceremony in disguise. Short version: a good spec captures **intent, constraints, and the *whys*** with enough precision that the agent doesn't have to guess, and with enough brevity that a human will still read it six months from now.

## What needs to be in it

A minimum useful spec has six blocks. Not all six need to be long, but all six need to be there.

### 1. Objective

One or two sentences describing what's being built and *for whom*. Not what API it exposes, not what endpoints it has — that comes later. Just: the observable result that justifies the work.

> *Allow a user to upload a profile photo that will be visible to followers and deletable only by them.*

If the objective can't be written in two sentences, almost always it's because it's mixing two different features. Separate them and write two specs.

### 2. No-goals

This is the section nobody writes and the one that prevents the most drift. **What is NOT being built**, explicitly.

> *Animated GIF support won't be supported in this version. Images won't be auto-compressed. There's no integration with the moderation system; that lives in another spec.*

No-goals are a vaccine against the agent's scope creep. Without them, the agent — following its training — will add compression, content validation, multi-format support and thumbnails, because all of those are "what's normally done". And all of those are work you didn't ask for.

### 3. Technical constraints

What the agent *cannot* touch, and what it *must* respect. These are the system's invariants.

> *Images are served from the existing S3 bucket; no new bucket is introduced. Authentication goes through the `requireAuth` middleware; no other is invented. Error handling follows the `Result<T, AppError>` pattern used in `src/api/`.*

Constraints are what makes the feature fit in the rest of the system instead of landing as a foreign body. They're the antidote to the context collapse from chapter 1.

### 4. Verifiable acceptance criteria

How you know it's done. The keyword is **verifiable**: an outsider, reading only the criterion, should be able to decide whether the code meets it or not.

> *An authenticated user can upload a JPEG or PNG image up to 10 MB. Upload fails with a clear message if the file is another type or exceeds the size. Only the file's owner can delete it. An integration test covers all three cases.*

"Works well" isn't an acceptance criterion. "Returns 413 with a readable message if the file exceeds 10 MB" is.

### 5. The *whys*

This is where ordinary specs die and good ones separate from the rest. Almost all specs explain the *what*. Almost none explain the *why*. And the whys are exactly what the agent needs to make smart decisions in the gaps the spec doesn't cover.

> *We don't auto-compress the image because the design team wants to preserve quality for verified-account avatars. (September 2025 decision, owner: @maria.)*
>
> *Only the owner can delete because adding moderation would change the permissions model and we want to keep this iteration's spec minimal.*

Whys are the ingredient Isoform's critique identifies as what bad SDD loses, and the main reason specs become useless months later. A spec without whys ages at the speed of the decisions around it. A spec with whys outlives those decisions because it explains them.

### 6. Boundaries — the three-tier system

This is Addy Osmani's most useful contribution to thinking about specs for agents. Instead of a flat list of "do" and "don't", divide constraints into **three tiers**:

- **✅ Always do** — things the agent should do without asking permission.
  *Example: run the test suite before declaring any task done.*
- **⚠️ Ask first** — things the agent should **ask confirmation** before doing.
  *Example: modify the database schema, add a new dependency, touch deployment config files.*
- **🚫 Never do** — things the agent must never do.
  *Example: commit secrets or keys, modify `vendor/`, disable tests so they pass, force push.*

The crucial distinction is the middle one. Without "ask first", the agent has only two modes: act or freeze. "Ask first" introduces a third — "consult before proceeding" — which is where productive human–agent conversation actually fits. It's the piece most teams forget, and the one that turns a restrictive spec into a collaborative one.

## A minimal template

```markdown
# Spec: [feature name]

## Objective
One or two sentences. Observable result. For whom.

## No-goals
- Thing NOT done
- Other thing NOT done

## Technical constraints
- Invariants respected
- Project patterns followed
- What can't be touched

## Acceptance criteria
- [ ] Verifiable criterion 1
- [ ] Verifiable criterion 2

## Whys
- Why this decision and not another (with owner and date)
- Accepted trade-offs

## Boundaries
- ✅ Always: ...
- ⚠️ Ask first: ...
- 🚫 Never: ...
```

This template fits in less than a screen and covers 80% of the value a spec can deliver. The other 20% are domain-specific details — state diagrams, API contracts, concrete input/output examples — that you add when the domain demands them, not by default.

## The curse of instructions

There's a concrete trap to avoid. Addy Osmani, citing academic studies, identifies what he calls **the curse of instructions**: the more you put in a prompt, the worse the model follows *each* of the things. It's not linear — it's a qualitative collapse. A spec with five constraints gets followed reasonably well. A spec with fifty gets followed reasonably badly across all of them, and often the agent acts as if none existed.

The operating consequence is counterintuitive: **more information in a single spec isn't better**. The winning strategy is to break large specs into small modules and pass the agent only the module relevant to the current task. A global 500-line spec is a bad plan; five modular 100-line specs where today's task references just two is a good plan.

This is also one of the reasons tools like Spec-kit and Kiro generate **many small files** instead of one big one, and why that proliferation, mismanaged, becomes its own problem (we'll see this in chapter 9).

## What distinguishes a good spec from a mediocre one

If you have to remember three things from this chapter, let them be these:

1. **No-goals save you more drift than any other section**. Always write them.
2. **Whys are what age well**. Without them, the spec lasts weeks. With them, it lasts years.
3. **"Ask first" is the missing category in almost every spec**. It's where real collaboration with the agent lives.

Bad specs enumerate technical trivia and say nothing about intent. Mediocre specs say a lot about *what* and nothing about *why*. Good specs are short, say *what*, *why* and *what not*, and can be read in five minutes without losing anything important.

## What comes next

Up to here we've seen what's *inside* a spec. In **chapter 4** we'll see the cycle a spec lives in: the four phases of the SDD process (with their two variants in the literature), how to chain them with an agent, and where the verification that prevents code from drifting away from the spec fits.
