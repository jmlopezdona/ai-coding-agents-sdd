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

#### Reusing criteria from upstream documents (user stories, contracts)

A question that comes up the moment a team starts writing specs seriously: if the user story that motivated the feature **already contains acceptance criteria**, what do I do with them? Reference them? Copy them? Rewrite them?

The short answer is **rewrite with traceability** — not copy verbatim, not reference-only. The longer answer depends on where that upstream document lives.

**Case 1: the user story lives outside the repo (Jira, Linear, Notion).**

The two bad options here are obvious:

- *Reference only* (`"see criteria in JIRA-1234"`) breaks the self-containment chapter 1 defends: the agent and the human reading this in six months have to chase a link into an external system. Worse, the user story can change silently without the code or the spec noticing — it's invisible drift, the worst kind, because not even the chapter 5 sensor can detect it.
- *Copy verbatim* imports the imprecision of product language. A user story says *"the user can easily upload a photo"*; your spec needs *"JPEG or PNG up to 10 MB, fails with 413 and a readable message if exceeded"*. Copying verbatim leaves you with criteria that *look* verifiable because they sound official but aren't.

The right option is to **rewrite the criteria in the spec at the level of precision the validator needs** and **cite the user story as the source of the why**, not as the container of the what. The user story stays canonical for product; the spec becomes canonical for the agent and the code.

Operationally: paste the criteria as an initial draft (with all the warnings from anti-pattern #12), rewrite them one by one, and in the "whys" section add something like *"criteria derived from JIRA-1234, refined in spec session on Mar 12, owner @maria"*. The chapter 5 validation runs against the spec, not against the user story.

**Case 2: the user story lives in the same Git repo as the spec.**

Here some objections to the "reference" option go away. Self-containment is preserved (everything is in the repo), drift stops being invisible (there's a commit), traceability becomes exact (you can cite a file and a commit), and a new possibility appears: an automatic sensor that detects when the user story changes without the spec being updated — a git hook or a nightly recurring agent. It's exactly the kind of two-floor bidirectional loop chapter 12 anticipates when it talks about the harness.

But **what doesn't change**:

- They're still two artifacts with two distinct purposes. The user story answers *what the user wants and why it matters to the business*. The spec answers *what observable guarantees the system must meet*. The right level of detail doesn't align by sharing a repo.
- Each one changes for different reasons. The user story is refined for product reasons; the spec is updated for technical reasons. Having a single file for both forces you to edit it for cross-cutting reasons that don't apply.
- The "rewrite, don't copy" rule still holds. The only thing that changes is that the complementary reference is now legitimate: you can point to `docs/product/avatar-upload.md` and the agent reads it as additional context, without that replacing your precise version in the spec.

The unified rule: **the spec contains its own precise, verifiable version of the acceptance criteria. The upstream document is cited as the source of the why. If both live in the same repo, you can additionally enable a sensor that watches for divergence between them.**

**The anti-pattern to avoid in both cases**: fusing the user story and the spec into a single hybrid file because "they say the same thing". They don't. And the resulting file is neither a good user story (too technical for the PM) nor a good spec (too vague for the agent). If you find yourself editing "the file" to reflect both a product copy change and a technical implementation discovery, split it back. We develop this pattern as anti-pattern #13 in chapter 11.

**The special case where reference without rewriting is appropriate.** When the upstream document really is the authoritative source and lives under its own validation discipline: an API contract maintained in OpenAPI/Protobuf with its own CI, a corporate security policy, an external standard (RFC, protocol spec). In those cases the spec **summarizes the implications** (*"this endpoint complies with the contract defined in `api-contracts/avatar.yaml@v3`"*) but doesn't copy the content. The critical distinction: that upstream document has its *own* validation operating on it. A Jira user story almost never has that property.

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

## Affected surfaces (optional, use when the spec touches >1 component)
### [Component] — [produced | modified | consumed]
- **Functional:** ...
- **Non-functional:** ...
- **Technical:** ...
- **More detail:** [reference to deeper doc if it exists]
```

This template fits in less than a screen and covers 80% of the value a spec can deliver. The other 20% are domain-specific details — state diagrams, API contracts, concrete input/output examples — that you add when the domain demands them, not by default.

## The curse of instructions

There's a concrete trap to avoid. Addy Osmani, citing academic studies, identifies what he calls **the curse of instructions**: the more you put in a prompt, the worse the model follows *each* of the things. It's not linear — it's a qualitative collapse. A spec with five constraints gets followed reasonably well. A spec with fifty gets followed reasonably badly across all of them, and often the agent acts as if none existed.

The operating consequence is counterintuitive: **more information in a single spec isn't better**. The winning strategy is to break large specs into small modules and pass the agent only the module relevant to the current task. A global 500-line spec is a bad plan; five modular 100-line specs where today's task references just two is a good plan.

This is also one of the reasons tools like Spec-kit and Kiro generate **many small files** instead of one big one, and why that proliferation, mismanaged, becomes its own problem (we'll see this in chapter 9).

## What if a generator agent writes the spec?

It's a natural idea: if the agent can write code, it should also be able to write specs. You hand it a user story, a feature description or an architectural component, and it returns the template above filled in. Startup cost drops, the entry barrier disappears, and apparently you've solved SDD's most expensive problem — writing specs.

It's an idea worth trying, and one that **fails by default almost every time**, not because of bad prompting but for a structural reason worth understanding before you spend weeks discovering it.

### The default mode: pseudocode dressed up as a spec

When you ask an LLM "generate a spec for this feature", the model optimizes for the metric it knows best how to measure: *apparent coverage*. And the easiest way to look exhaustive is to drop down to code-level detail, because that's where the model is comfortable — its training corpus is code. Climbing up to *intent*, *constraints* and *whys* requires abstraction, which is exactly what the model does worst.

The result is predictable and almost universal: the generated "spec" is actually **commented pseudocode** dressed up as a design doc. It has classes, methods, function signatures, API payloads, inline business rules, sequence diagrams. It looks thorough. And it's exactly the *opposite* of what a good spec should be, according to everything we've seen in this chapter.

A good spec operates at a **different abstraction level** from the code. If it mentions classes and methods, it's no longer a spec: it's an implementation sketch. The spec's question isn't "which class calls which method?", it's "what observable guarantees must this satisfy, under what constraints, and why?". When the agent drops to the first level, the spec has lost its reason for existing, because it stops being **comparable against the code from a different vantage point**. If the spec says the same thing as the code but in markdown, it's just noise.

### The two serious consequences

When you end up with specs like this — and, if you leave the agent in its default mode, you will — you pay two costs that reinforce each other.

**You implement twice.** You've already paid the cost of thinking through the implementation at the level of classes and methods in the spec, and you're going to pay the same cost again when you code. Worse: anything you discover while implementing (and implementing always reveals something) forces you to go back and edit the spec, or accept drift on day one. You're at the worst point on the chapter 9 curve: maintenance tax from day one, before you've written a single line of code.

**Human review becomes as expensive as code review.** This is exactly what Fowler reports about Spec-kit in chapter 6 — that the generated files were *heavier to review than the code itself*. And it's worse than reviewing code directly, because at least code runs and tests verify it; pseudocode in markdown has no safety net. You're reviewing something at code-level detail but **without code's guarantees**. The tired reviewer skims it — and the "spec" passes the filter as if it had been reviewed.

The combination of the two is poisonous: you double the work *and* review it worse. It's almost always worse than not having written a spec at all.

### Why it isn't just a template problem

It's tempting to think this gets fixed with better prompting or a stricter template. *"I'll tell the agent not to include signatures, not to mention classes, to express everything as externally observable invariants."* And yes, that reduces the problem. But it **doesn't eliminate it**, for two reasons worth being clear about.

First, the model's bias toward dropping to code level is very strong because that's where it "feels productive". When you restrict everything it does well and ask only for what it does badly — abstracting intent, capturing whys that aren't in the input — the resulting spec is short, thin, and the team senses "something is missing". The next step is almost always asking for more detail, and you're back to pseudocode. It's an unstable equilibrium.

Second, there are things the agent **can't** generate well no matter what template you give it. The **whys** are almost never in the input — they live in the PM's head, in a Slack conversation, in a trade-off decided two years ago. What the agent doesn't know, it invents or omits. A spec generated with invented "whys" is worse than a spec without "whys", because it lies with the appearance of authority.

### Legitimate use: the agent as a draft generator of the *what*, not the *how*

This doesn't mean agent-generated specs are useless. They're useful, but in a narrower role than most people imagine:

- **The agent can start well**: the **objective** block, the **verifiable acceptance criteria** (carefully), a first list of **candidate non-goals** that the human then refines, and a proposal for **boundaries** based on patterns it already sees in the repo.
- **The human has to put in, no exceptions**: the **whys**, the **real non-goals** (especially political or scope ones that aren't in any input), the team's **tacit technical constraints**, and the decision of *how much* spec the task deserves (the modulation of chapter 8). And, above all, the human has to **delete** anything the agent injected as pseudocode: classes, methods, signatures, payloads. If after deleting all that the spec is empty, you didn't need a spec; you needed well-written code (chapter 10, *context engineering*).

The most useful practical rule: **if your generated spec mentions a function signature or a class name, delete it**. The spec talks about what guarantees the system meets, not how it's built.

And the honest warning: using the agent as a draft generator **doesn't reduce the total cost** of doing SDD well. What it reduces is the cost of starting the first draft, which for many teams is the psychological barrier that weighs the most. The real cost — thinking intent precisely, capturing whys, keeping the spec alive — is still there, with no shortcut, and the agent doesn't pay it for you. If your SDD adoption depends on automatic generation eliminating that cost, what you'll have isn't SDD — you'll have anti-pattern #12 from chapter 11.

## The right level of detail depends on the spectrum level

So far we've talked about a spec's anatomy as if it were a single thing. It isn't. **The appropriate level of detail depends on which level of the chapter 2 spectrum you're operating at**, and writing a spec at the wrong level of detail for your spectrum level is one of the most common — and most expensive — ways to suffer bad SDD.

### Spec-first → light, intentional, non-exhaustive detail

In spec-first the spec is read once, when the feature kicks off, and from then on the code drifts freely. Its only function is to **align team and agent at the start**. Nothing is going to read it again, so every extra line you write is work no one will ever recover.

The right amount of detail: objective, non-goals, acceptance criteria, the critical whys, and little else. If your spec-first runs longer than a screen, it's almost always because you're writing *aspirational* spec-anchored (anti-pattern #4 in chapter 11) or because you've fallen into pseudocode (anti-pattern #12). The optimal spec-first is the minimum viable to start with clear intent.

### Spec-anchored → medium detail, bounded by what the anchoring can verify

The logic shifts here. The spec *is* going to be read again — by validators, by tests, by recurring agents that detect drift. Detail is no longer optional: it has to be **enough for the anchoring mechanism to compare against**.

But there's a counterintuitive upper bound: **detail should not go beyond what your anchoring knows how to verify**. If your validator checks API contracts and your spec describes UI rules, the UI part isn't anchored — it's spec-first disguised as spec-anchored. And because that piece isn't verified, it drifts freely.

The operating rule: the detail of a spec-anchored spec is measured against the **scope of the anchoring**, not against an abstract notion of "completeness". If what you write can't be verified automatically, writing it doesn't give you more anchoring — it just gives you more maintenance tax.

### Spec-as-source → exhaustive detail, but of a different kind

This is where the conversation gets interesting. Spec-as-source does need maximum detail, because the code is generated from the spec. But that detail is of **a different nature** from the other two levels.

It's **formal or semi-formal** detail: type signatures, contracts, invariants, grammars, transformation rules. It's **generator-friendly** detail: designed so a generator (LLM or otherwise) can produce deterministic code from it. And it's legitimate for signatures, schemas and concrete structures to appear — because at this level **the spec is the source code, just in a different notation**.

Here's the uncomfortable connection with the previous section: an agent-generated "spec" that is pseudocode in disguise *looks* superficially like a spec-as-source. It has classes, methods, payloads, rules. But there's a critical difference: **a legitimate spec-as-source comes with a deterministic generator that produces the code**. Without that generator, what you have is the worst possible combination: spec-as-source's exhaustive detail with spec-first's non-determinism. It's the conceptual equivalent of writing XML+OCL in the 2000s without an MDA compiler behind it. Fowler would know exactly what to call it.

### The unifying rule

If you have to distill all of the above into a single sentence:

> **The right level of detail for a spec is the one your validation mechanism — human, automatic, or generative — knows how to consume. More than that is waste; less than that is blindness.**

In spec-first the "validator" is the team in a single initial reading, so the right detail is what fits in that reading. In spec-anchored the validator is the anchoring mechanism, so the right detail is what the anchoring knows how to compare. In spec-as-source the validator is the generator, so the right detail is what the generator needs to produce unambiguous code.

Almost every pathology we'll see in chapter 11 comes from **misaligning these three things**: writing spec-anchored without anchoring (#4), writing spec-as-source without a generator (#9 + #12), or writing spec-first with spec-as-source-level detail (#2 + #12). Anatomy isn't absolute — it's relative to what kind of validation will be applied to what you write.

## The spec and technical components: consume, produce, modify

So far we've talked about a spec's anatomy as if it applied to an isolated piece. In practice, almost every real spec **touches multiple components at once** — one is built new, another is modified, and several are consumed without being touched. Each of those relationships has different rules, and mixing them in a single language is one of the fastest ways to inflate a spec without gaining precision.

This section extends the earlier subsection on *"Reusing criteria from upstream documents"* — where we covered the user-story case — to the more general question: what does a spec say about the technical components it relates to?

### Three relationships, three rules

A spec can have three distinct relationships with each component it touches:

**1. Consumed (it exists; the spec uses it).** The component is already built, has its own contract (API, module, library), and the spec leans on it without modifying it. The rule is **reference the contract, don't reimplement it**. The spec documents *how* the component is used — which endpoints it calls, which errors it expects to handle — but it doesn't document what the component does on its own. That lives in the component's contract, which is its own source of truth.

If the component is an ADR, an API contract, a security policy or an external standard, the rule is the same with one nuance: **cite the specific invariant this spec depends on, not the entire document**. *"This spec respects ADR-007 (single Postgres instance) and ADR-012 (no synchronous calls between services)"* is useful. *"Related: ADR-007"* is noise.

**2. Produced (it doesn't exist; the spec creates it).** This is the typical case for a new feature: *"this spec creates a new endpoint / a new service / a new module"*. Here the trap is obvious and dangerous: because the component doesn't exist yet, you feel you have to "define it" in the spec, and you almost always end up describing it with classes, methods, signatures and payloads. That's exactly anti-pattern #12 (pseudocode dressed up as spec).

The right form: the spec describes the **observable contract** the new component must offer, not its internal structure. *"After this spec, there must be a capability to accept authenticated avatar uploads (JPEG/PNG, ≤10 MB) and return a retrievable URL"* is observable contract. *"Create `AvatarUploadService` with method `upload(file, user_id) -> AvatarMetadata`"* is pseudocode.

There's a conceptually important nuance: a spec that produces a new component is **the transient source of truth that hands off**. While the component doesn't exist, the spec is the only thing describing it. Once the component exists, **its own documentation** (its README, its OpenAPI contract, its tests, its own code) becomes the operational source, and the original spec becomes the *"why it was built this way"* — historical intent, not living contract. If you confuse the two roles, you end up with two artifacts competing to be the source of truth for the same component, and you fall back into the silent fusion of anti-pattern #13.

**3. Modified (it exists; the spec adds, changes or removes capabilities).** Here the rule is **describe the observable delta, not the component's full state**. The spec doesn't have to re-document how `User` works; it has to say *"add the `avatar_url` field (optional, current avatar URL or absent). Updated when an avatar is uploaded; cleared when deleted. No other model behavior changes."*

The **"no other behavior changes"** part is what saves the spec from inflating. Without it, the team tends to rewrite the entire component "to make the spec complete". The delta is the spec; the rest of the component lives in its own documentation.

### Capturing the load-bearing essentials in three dimensions

For each component the spec produces or modifies, the immediate question is *"how much detail do I include?"*. The vague answer leads to pseudocode. The useful answer is to decompose into **three explicit dimensions** and, for each, capture only what's load-bearing — and reference deeper docs when they exist.

The three dimensions, applied to the component's **delta**:

- **Functional** — what changes in observable behavior. *"The `User` model can now have an avatar; the field is optional; updated on upload, cleared on delete."*
- **Non-functional** — which cross-cutting constraints the change must respect. Performance, security, compatibility, observability, data. *"The migration must not require downtime; the field must not break existing model serialization."*
- **Technical** — which contract/integration decisions the change explicitly closes. *"The field is indexed by `user_id`; the avatar URL goes through the existing CDN, not directly through the bucket."*

For each dimension, an **optional reference to the deeper document** if it exists: *"Migration detail in `docs/db/migrations/0042-add-avatar.md`. CDN policy in `infra/cdn/policy.md`."*

Three important clarifications about this pattern:

1. **If a dimension doesn't add anything, omit it.** Don't force three lines per component for symmetry. Absence is informative: it says that dimension doesn't change.
2. **What goes in the spec is what's load-bearing**: what the chapter 5 validator can compare against the code, and what the team wants to be contractual. Finer detail lives in the component's documentation, which has its own validation mechanism (tests, type checks, versioned contracts).
3. **The term "non-functional" is classical but contested** (Fowler argues everything is functional, just along different dimensions). For the purposes of this chapter we use it as an operational category — three different questions to ask about each component — not as ontology. If your team prefers calling them *behavior*, *qualities*, *contracts*, it makes no difference. What matters is that they are **three different questions**, not one.

### The "Affected surfaces" block

When a spec touches multiple components, it's worth having an explicit block at the start that **enumerates the affected surfaces with their relationship type**. It's not work breakdown (it doesn't enumerate tasks, doesn't mention files, doesn't assign order — that lives in the planning phase of chapter 4). It's **surface mapping**, and it does three things no other part of the spec does well:

1. It gives the agent and the chapter 5 validator an **enumerable list** of the change's blast radius, which in chapter 1 was invisible.
2. It gives the human a **signal of the spec's size**: if the list has 12 entries, the spec is almost certainly *two features* badly stitched together, or a refactor masquerading as a feature.
3. **It isn't work breakdown**, which keeps it on the "what" side rather than the "how".

An example of the block enriched with the three dimensions:

```markdown
## Affected surfaces

### `User` model — modified

- **Functional:** adds `avatar_url` (optional, current avatar URL or absent).
  Updated on upload; cleared on delete. Rest of the model invariant.
- **Non-functional:** the migration must not require downtime; existing
  serialization can't break (test `test_user_serialization_back_compat`).
- **Technical:** field indexed by `user_id`. Permission schema unchanged.
- **More detail:** `docs/db/migrations/0042-add-avatar.md` (once created).

### `POST /api/avatar` endpoint — produced

- **Functional:** accepts authenticated uploads (JPEG/PNG, ≤10 MB), returns URL.
  Errors: 401 unauthenticated, 413 if oversize, 415 if invalid type.
- **Non-functional:** existing API gateway rate limit applies unchanged.
- **Technical:** consumes `services/avatar-storage` (current contract, unchanged)
  and `auth-middleware` (current contract, unchanged).
- **More detail:** this block is the transient source; after implementation,
  operational doc lives in the new service's README.

### `services/avatar-storage` — consumed

- **Functional:** calls `POST /upload`; handles 413 and 415 explicitly,
  propagates 5xx to the caller.
- Rest: see `services/avatar-storage/contract.openapi.yaml@v2`.
```

Notice how the *consumed* component has a much shorter block — it only describes *how this feature uses it*, not what the component does, because that lives in its own contract. And notice how the *modified* component declares explicitly what does **not** change. Those two habits are what distinguish a useful spec from an inflated one.

### Size heuristic

A practical rule that works: if a spec produces or modifies more than **3-4 surfaces**, you probably have one of three problems, and it's worth diagnosing before continuing:

- **(a)** A feature too big that should be split into multiple specs (chapter 8 modulation).
- **(b)** An architectural change disguised as a feature, which deserves its own ADR that this spec then hangs from.
- **(c)** A refactor camouflaged as a feature (also chapter 8 modulation): what you're changing is the *shape* of the system, not adding business capability.

The three have different treatments in the rest of the course, and the spec alone isn't the right artifact for any of them.

### Consolidated summary table

Pulling this section together with the earlier subsection on upstream documents, this is the unified rule by relationship type:

| Relationship | Main rule | Spec section | Typical trap |
|---|---|---|---|
| **Consumed component** | Reference the contract; document the use | Technical constraints | Re-documenting the component |
| **Produced component** | Define observable contract, not structure | Affected surfaces + Criteria | Pseudocode (#12); failing to mark the handoff to the component's doc |
| **Modified component** | Describe the delta, declare the rest invariant | Affected surfaces + Criteria | Rewriting the entire component |
| **ADR / architecture** | Cite specific invariants | Technical constraints | Copying invariants (drift guaranteed) |
| **API contract / external standard** | Reference + summary of implications | Technical constraints | Re-documenting the contract |
| **Feature brief / product doc** | Extract the cardinal whys | Whys | Importing vague product language |
| **User story** | Rewrite criteria with traceability | Criteria + Whys | Verbatim copy or opaque reference |

And a new anti-pattern that appears when references accumulate without discipline: **reference soup** — a spec where its own content gets buried under a long list of citations to ADRs, contracts, briefs and components, all without prioritization. The agent and the human don't know which are load-bearing and which are tangential, and they end up ignoring all of them. It's the inverse of the *curse of instructions*, and we develop it as anti-pattern #14 in chapter 11. The preventive rule: **reference only what the agent or the human needs for this task, and annotate the why of each reference**.

## What distinguishes a good spec from a mediocre one

If you have to remember three things from this chapter, let them be these:

1. **No-goals save you more drift than any other section**. Always write them.
2. **Whys are what age well**. Without them, the spec lasts weeks. With them, it lasts years.
3. **"Ask first" is the missing category in almost every spec**. It's where real collaboration with the agent lives.

Bad specs enumerate technical trivia and say nothing about intent. Mediocre specs say a lot about *what* and nothing about *why*. Good specs are short, say *what*, *why* and *what not*, and can be read in five minutes without losing anything important.

## What comes next

Up to here we've seen what's *inside* a spec. In **chapter 4** we'll see the cycle a spec lives in: the four phases of the SDD process (with their two variants in the literature), how to chain them with an agent, and where the verification that prevents code from drifting away from the spec fits.
