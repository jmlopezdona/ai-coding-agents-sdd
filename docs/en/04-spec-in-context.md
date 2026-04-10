# 4. The spec in context: detail, components and calibration

In the [previous section](03-anatomy-of-a-spec.md) we saw what to put inside a spec — the six blocks, the template, the curse of instructions, and the traps of agent generation. Now comes the follow-up question: **how much detail**, **how to incorporate information from upstream documents**, and **how does the spec relate to the technical components** of the system it touches.

## The right level of detail depends on the spectrum level

So far we've talked about a spec's anatomy as if it were a single thing. It isn't. **The appropriate level of detail depends on which level of the chapter 2 spectrum you're operating at**, and writing a spec at the wrong level of detail for your spectrum level is one of the most common — and most expensive — ways to suffer bad SDD.

### Spec-first → light, intentional, non-exhaustive detail

In spec-first the spec is read once, when the feature kicks off, and from then on the code drifts freely. Its only function is to **align team and agent at the start**. Nothing is going to read it again, so every extra line you write is work no one will ever recover.

The right amount of detail: objective, non-goals, acceptance criteria, the critical whys, and little else. If your spec-first runs longer than a screen, it's almost always because you're writing *aspirational* spec-anchored (anti-pattern #4 in chapter 12) or because you've fallen into pseudocode (anti-pattern #12). The optimal spec-first is the minimum viable to start with clear intent.

### Spec-anchored → medium detail, bounded by what the anchoring can verify

The logic shifts here. The spec *is* going to be read again — by validators, by tests, by recurring agents that detect drift. Detail is no longer optional: it has to be **enough for the anchoring mechanism to compare against**.

But there's a counterintuitive upper bound: **detail should not go beyond what your anchoring knows how to verify**. If your validator checks API contracts and your spec describes UI rules, the UI part isn't anchored — it's spec-first disguised as spec-anchored. And because that piece isn't verified, it drifts freely.

The operating rule: the detail of a spec-anchored spec is measured against the **scope of the anchoring**, not against an abstract notion of "completeness". If what you write can't be verified automatically, writing it doesn't give you more anchoring — it just gives you more maintenance tax.

### Spec-as-source → exhaustive detail, but of a different kind

This is where the conversation gets interesting. Spec-as-source does need maximum detail, because the code is generated from the spec. But that detail is of **a different nature** from the other two levels.

It's **formal or semi-formal** detail: type signatures, contracts, invariants, grammars, transformation rules. It's **generator-friendly** detail: designed so a generator (LLM or otherwise) can produce deterministic code from it. And it's legitimate for signatures, schemas and concrete structures to appear — because at this level **the spec is the source code, just in a different notation**.

Here's the uncomfortable connection with the [previous section](03-anatomy-of-a-spec.md#what-if-a-generator-agent-writes-the-spec): an agent-generated "spec" that is pseudocode in disguise *looks* superficially like a spec-as-source. It has classes, methods, payloads, rules. But there's a critical difference: **a legitimate spec-as-source comes with a deterministic generator that produces the code**. Without that generator, what you have is the worst possible combination: spec-as-source's exhaustive detail with spec-first's non-determinism. It's the conceptual equivalent of writing XML+OCL in the 2000s without an MDA compiler behind it. Fowler would know exactly what to call it.

### The unifying rule

If you have to distill all of the above into a single sentence:

> **The right level of detail for a spec is the one your validation mechanism — human, automatic, or generative — knows how to consume. More than that is waste; less than that is blindness.**

In spec-first the "validator" is the team in a single initial reading, so the right detail is what fits in that reading. In spec-anchored the validator is the anchoring mechanism, so the right detail is what the anchoring knows how to compare. In spec-as-source the validator is the generator, so the right detail is what the generator needs to produce unambiguous code.

Almost every pathology we'll see in chapter 12 comes from **misaligning these three things**: writing spec-anchored without anchoring (#4), writing spec-as-source without a generator (#9 + #12), or writing spec-first with spec-as-source-level detail (#2 + #12). Anatomy isn't absolute — it's relative to what kind of validation will be applied to what you write.

## Reusing criteria from upstream documents (user stories, contracts)

An unavoidable question: if the upstream user story **already contains acceptance criteria**, what do I do with them? The short answer is **rewrite with traceability** — neither copy verbatim (it imports the imprecision of product language), nor reference-only (`"see JIRA-1234"` breaks the self-containment of chapter 1 and opens invisible drift). You rewrite the criteria in the spec at the level of precision the chapter 6 validator needs, and cite the user story in the "whys" section as the source of *what motivated the decision*, not as the container of *what to do*.

The unified rule: **the spec contains its own precise, verifiable version of the criteria. The upstream document is cited as the source of the why.** The anti-pattern to avoid — *fusing user story and spec into a single hybrid file because "they say the same thing"* — we develop as anti-pattern #13 in chapter 12.

**When the user story lives in the same repo as the spec**, the most serious objections go away: self-containment is preserved (everything is in git), drift stops being invisible (there's a commit), and a new possibility appears — an automatic sensor (hook or recurring agent) that detects when the user story changes without the spec being updated. But the "rewrite with traceability" rule still holds: they remain two artifacts with different purposes (the user story answers *what the user wants*; the spec, *what guarantees the system must meet*), and each changes for different reasons.

A concrete risk in this scenario: the temptation to edit both documents frequently without each change being fully justified. Every cross-edit is an opportunity for misalignment — and the more there are, the harder it is to know which of the two reflects the current truth. The trace (the explicit reference connecting a spec criterion to its origin in the user story) serves a dual purpose: it acts as a **checkpoint at the time of change** (the human or agent editing the spec can verify that the origin is still valid) and as **audit material** for a doc-gathering agent that scans for discrepancies between both documents on a recurring basis.

**One legitimate exception**: when the upstream document is genuinely authoritative and lives under its own validation discipline (an OpenAPI/Protobuf contract with its own CI, a corporate security policy, an RFC), the spec **summarizes the implications** without copying the content. The critical distinction: that upstream document has its *own* validation operating on it. A Jira user story almost never has that property.

## The spec and technical components: consume, produce, modify

Anatomy doesn't live in a vacuum: every spec lands on code that already exists. And in practice, almost every real spec **touches multiple components at once** — one is built new, another is modified, and several are consumed without being touched. Each of those relationships has different rules, and mixing them in a single language is one of the fastest ways to inflate a spec without gaining precision.

This section extends the previous one on upstream documents — where we covered the user-story case — to the more general question: what does a spec say about the technical components it relates to?

### Three relationships, three rules

A spec can have three distinct relationships with each component it touches:

**1. Consumed (it exists; the spec uses it).** The component is already built, has its own contract (API, module, library), and the spec leans on it without modifying it. The rule is **reference the contract, don't reimplement it**. The spec documents *how* the component is used — which endpoints it calls, which errors it expects to handle — but it doesn't document what the component does on its own. That lives in the component's contract, which is its own source of truth.

If the component is an ADR, an API contract, a security policy or an external standard, the rule is the same with one nuance: **cite the specific invariant this spec depends on, not the entire document**. *"This spec respects ADR-007 (single Postgres instance) and ADR-012 (no synchronous calls between services)"* is useful. *"Related: ADR-007"* is noise.

**2. Produced (it doesn't exist; the spec creates it).** This is the typical case for a new feature: *"this spec creates a new endpoint / a new service / a new module"*. Here the trap is obvious and dangerous: because the component doesn't exist yet, you feel you have to "define it" in the spec, and you almost always end up describing it with classes, methods, signatures and payloads. That's exactly anti-pattern #12 (pseudocode dressed up as spec).

The right form: the spec describes the **observable contract** the new component must offer, not its internal structure. *"After this spec, there must be a capability to accept authenticated avatar uploads (JPEG/PNG, ≤10 MB) and return a retrievable URL"* is observable contract. *"Create `AvatarUploadService` with method `upload(file, user_id) -> AvatarMetadata`"* is pseudocode.

There's a conceptually important nuance: a spec that produces a new component is **the transient source of truth that hands off**. While the component doesn't exist, the spec is the only thing describing it. Once the component exists, **its own documentation** (its README, its OpenAPI contract, its tests, its own code) becomes the operational source, and the original spec becomes the *"why it was built this way"* — historical intent, not living contract. If you confuse the two roles, you end up with two artifacts competing to be the source of truth for the same component, and you fall back into the silent fusion of anti-pattern #13.

**3. Modified (it exists; the spec adds, changes or removes capabilities).** Here the rule is **describe the observable delta, not the component's full state**. The spec doesn't have to re-document how `User` works; it has to say *"add the `avatar_url` field (optional, current avatar URL or absent). Updated when an avatar is uploaded; cleared when deleted. No other model behavior changes."*

The **"no other behavior changes"** part is what saves the spec from inflating. Without it, the team tends to rewrite the entire component "to make the spec complete". The delta is the spec; the rest of the component lives in its own documentation.

> **The three relationships in one line**: *consumed* = reference the contract and describe only the use; *produced* = define the observable contract and prepare the handoff to the component's doc; *modified* = describe the delta and declare the rest invariant.

### Capturing the load-bearing essentials in three dimensions

For each component the spec produces or modifies, the immediate question is *"how much detail do I include?"*. The vague answer leads to pseudocode. The useful answer is to decompose into **three explicit dimensions** and, for each, capture only what's load-bearing — and reference deeper docs when they exist.

The three dimensions, applied to the component's **delta**:

- **Functional** — what changes in observable behavior. *"The `User` model can now have an avatar; the field is optional; updated on upload, cleared on delete."*
- **Non-functional** — which cross-cutting constraints the change must respect. Performance, security, compatibility, observability, data. *"The migration must not require downtime; the field must not break existing model serialization."*
- **Technical** — which contract/integration decisions the change explicitly closes. *"The field is indexed by `user_id`; the avatar URL goes through the existing CDN, not directly through the bucket."*

For each dimension, an **optional reference to the deeper document** if it exists: *"Migration detail in `docs/db/migrations/0042-add-avatar.md`. CDN policy in `infra/cdn/policy.md`."*

Three important clarifications about this pattern:

1. **If a dimension doesn't add anything, omit it.** Don't force three lines per component for symmetry. Absence is informative: it says that dimension doesn't change.
2. **What goes in the spec is what's load-bearing**: what the chapter 6 validator can compare against the code, and what the team wants to be contractual. Finer detail lives in the component's documentation, which has its own validation mechanism (tests, type checks, versioned contracts).
3. **The term "non-functional" is classical but contested** (Fowler argues everything is functional, just along different dimensions). For the purposes of this chapter we use it as an operational category — three different questions to ask about each component — not as ontology. If your team prefers calling them *behavior*, *qualities*, *contracts*, it makes no difference. What matters is that they are **three different questions**, not one.

> **The three dimensions in one line**: *functional* describes observable behavior; *non-functional* the cross-cutting constraints; *technical* the contract and integration decisions. Only the load-bearing parts go in the spec; the rest gets referenced.

### Where this information goes in the six blocks

The consume/produce/modify taxonomy and the three dimensions don't create a new section in the spec — they help you write the sections you already have better:

- **Consumed components** → go in **technical constraints**: "auth goes through `requireAuth`; `avatar-storage` is used via `POST /upload`, handling 413 and 415 explicitly".
- **Produced or modified components** → their observable guarantees go in **acceptance criteria**: "accepts JPEG/PNG uploads ≤10 MB; returns URL; fails with 413/415 with readable message". Their design bounds go in **technical constraints**: "no new table; the avatar is a field on `User`".
- **The three dimensions** (functional, non-functional, technical) are the three questions you ask yourself when writing each criterion or constraint, not three subsections within the spec.
- **The whys** behind each decision go, as always, in **whys**.

When a component is consumed, the spec is shorter for it — it only describes *how this feature uses it*, not what the component does (that lives in its own contract). When a component is modified, the spec explicitly declares what does **not** change — that's what saves the spec from inflating. Those two habits are what distinguish a useful spec from an inflated one.

### Size heuristic: when the spec needs splitting

A practical rule that works: if while writing constraints and criteria you discover the spec produces or modifies more than **3-4 components**, you probably have one of three problems, and it's worth diagnosing before continuing:

1. A feature too big that should be split into multiple specs (chapter 9 modulation).
2. An architectural change disguised as a feature, which deserves its own ADR that this spec then hangs from.
3. A refactor camouflaged as a feature (also chapter 9 modulation): what you're changing is the *shape* of the system, not adding business capability.

The three have different treatments in the rest of the course, and the spec alone isn't the right artifact for any of them.

### Consolidated summary table

Pulling this section together with the earlier subsection on upstream documents, this is the unified rule by relationship type:

| Relationship | Main rule | Spec section | Typical trap |
|---|---|---|---|
| **Consumed component** | Reference the contract; document the use | Technical constraints | Re-documenting the component |
| **Produced component** | Define observable contract, not structure | Criteria + Constraints | Pseudocode (#12); failing to mark the handoff to the component's doc |
| **Modified component** | Describe the delta, declare the rest invariant | Criteria + Constraints | Rewriting the entire component |
| **ADR / architecture** | Cite specific invariants | Technical constraints | Copying invariants (drift guaranteed) |
| **API contract / external standard** | Reference + summary of implications | Technical constraints | Re-documenting the contract |
| **Feature brief / product doc** | Extract the cardinal whys | Whys | Importing vague product language |
| **User story** | Rewrite criteria with traceability | Criteria + Whys | Verbatim copy or opaque reference |

And a new anti-pattern that appears when references accumulate without discipline: **reference soup** — a spec where its own content gets buried under a long list of citations to ADRs, contracts, briefs and components, all without prioritization. The agent and the human don't know which are load-bearing and which are tangential, and they end up ignoring all of them. It's the inverse of the *curse of instructions* — a spec drowned by its own citations — and we develop it as anti-pattern #14 in chapter 12. The preventive rule: **reference only what the agent or the human needs for this task, and annotate the why of each reference**.

## In summary: what distinguishes a good spec from a mediocre one

If you have to remember three things from these two sections, let them be these:

1. **Non-goals save you more drift than any other section**. Always write them.
2. **Whys are what age well**. Without them, the spec lasts weeks. With them, it lasts years.
3. **"Ask first" is the missing category in almost every spec**. It's where real collaboration with the agent lives.

### Mediocre vs good, side by side

The difference is best seen in contrast. Same feature, two specs:

**Mediocre** (looks complete, isn't):

> *Spec: Avatar upload. The system will allow users to upload an avatar. A `POST /api/avatar` endpoint will be created that receives the file and stores it. The file will be validated to be an image. The URL of the saved avatar will be returned. An `avatar_url` field will be added to the `User` model. Tests will be implemented.*

It talks about the *what* in implementation terms, mentions no *why*, has no non-goals, no boundaries, the criteria aren't verifiable ("the file will be validated to be an image" — which types? which size? which error?), and it leaks code-level decisions (`POST /api/avatar`, `avatar_url`) that are implementation, not contract.

**Good** (short, intentional, ages well):

> **Spec: Avatar upload**
>
> **Objective:** allow an authenticated user to upload a profile photo visible to followers and deletable only by them.
>
> **Non-goals:** no animated GIF support; no automatic compression; content moderation lives in another spec.
>
> **Constraints:** served from the existing S3 bucket; auth goes through `requireAuth`; no new table, the avatar is a field on `User`.
>
> **Criteria:** an authenticated user can upload JPEG or PNG ≤10 MB; fails with 413 if oversize and 415 if invalid type, both with a readable message; only the owner can delete; an integration test covers all three cases.
>
> **Whys:** we don't compress because design wants to preserve quality for verified accounts (Sept-2025, @maria); only the owner deletes because adding moderation would change the permission model and we want this iteration minimal.
>
> **Boundaries:** ✅ run tests before declaring done · ⚠️ ask before touching the `User` schema · 🚫 never disable existing tests or commit secrets.

The good one fits in roughly the same space as the mediocre one, but it captures intent, constraints, verifiable criteria, and the whys that will let it survive the decisions surrounding it.

**The final rule**: bad specs enumerate technical trivia and say nothing about intent. Mediocre specs say a lot about *what* and nothing about *why*. Good specs are short, say *what*, *why* and *what not*, and can be read in five minutes without losing anything important.

## What comes next

Up to here we've seen what's *inside* a spec and how to calibrate its detail for context. The unifying rule: **write only the detail your validation mechanism knows how to consume**. If anatomy answers the *what*, what comes next is the *when* and the *how it's validated*: the moment a spec stops being a document and starts behaving like a contract.

In **Chapter 5** we'll see the cycle a spec lives in: the four phases of the SDD process (with their two variants in the literature), how to chain them with an agent, and where the verification that prevents code from drifting away from the spec fits.
