# 4. The spec in context: detail, components and calibration

In the [previous section](03-anatomy-of-a-spec.md) we saw what to put inside a spec — the six blocks, the template, the curse of instructions, and the traps of agent generation. Now comes the follow-up question: **how much detail**, and **how does the spec relate to its external sources** — product documents, architectural decisions, code components.

## The right level of detail depends on the spectrum level

So far we've talked about a spec's anatomy as if it were a single thing. It isn't. **The appropriate level of detail depends on which level of the chapter 2 spectrum you're operating at**, and writing a spec at the wrong level of detail for your spectrum level is one of the most common — and most expensive — ways to suffer bad SDD.

### Spec-first → light, intentional, non-exhaustive detail

In spec-first the spec is read once, when the feature kicks off, and from then on the code drifts freely. Its only function is to **align team and agent at the start**. No one is going to read it again, so every extra line you write is work no one will ever recover.

The right amount of detail: objective, non-goals, acceptance criteria, the critical whys, and little else. If your spec-first runs longer than a screen, it's almost always because you're writing *aspirational* spec-anchored ([anti-pattern #4](12-anti-patterns.md#4-theater-spec-with-fake-anchoring) in chapter 12) or because you've fallen into pseudocode ([anti-pattern #12](12-anti-patterns.md#12-generated-spec-thats-pseudocode-in-disguise)). The optimal spec-first is the minimum viable to start with clear intent.

### Spec-anchored → medium detail, bounded by what the anchoring can verify

The logic shifts here. The spec *is* going to be read again — by validators, by tests, by recurring agents that detect drift. Detail is no longer optional: it has to be **enough for the anchoring mechanism to compare against**.

But there's a counterintuitive upper bound: **detail should not go beyond what your anchoring knows how to verify**. If your validator checks API contracts and your spec describes UI rules, the UI part isn't anchored — it's spec-first disguised as spec-anchored. And because that piece isn't verified, it drifts freely.

The operating rule: the detail of a spec-anchored spec is measured against the **scope of the anchoring**, not against an abstract notion of "completeness". If what you write can't be verified automatically, writing it doesn't give you more anchoring — it just gives you more maintenance tax.

### Spec-as-source → exhaustive detail, but of a different kind

This is where the conversation gets interesting. Spec-as-source does need maximum detail, because the code is generated from the spec. But that detail is of **a different nature** from the other two levels.

It's **formal or semi-formal** detail: type signatures, contracts, invariants, grammars, transformation rules. It's **generator-friendly** detail: designed so a generator (LLM or otherwise) can produce deterministic code from it. And it's legitimate for signatures, schemas and concrete structures to appear — because at this level **the spec is the source code, just in a different notation**.

Here's the uncomfortable connection with the [previous section](03-anatomy-of-a-spec.md#what-if-a-generator-agent-writes-the-spec): an agent-generated "spec" that is pseudocode in disguise *looks* superficially like a spec-as-source. It has classes, methods, payloads, rules. But there's a critical difference: **a legitimate spec-as-source comes with a deterministic generator that produces the code**. Without that generator, what you have is the worst possible combination: spec-as-source's exhaustive detail with spec-first's non-determinism. It's the equivalent of writing exhaustive formal specifications without a generator to execute them — exactly what [Fowler](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html) criticized about the original Model-Driven Development: formalism without execution to back it up.

### The unifying rule

If you have to distill all of the above into a single sentence:

> **The right level of detail for a spec is the one your validation mechanism — human, deterministic, or generative — knows how to consume. More than that is waste; less than that is blindness.**

In spec-first the "validator" is the team in a single initial reading, so the right detail is what fits in that reading. In spec-anchored the validator is the anchoring mechanism, so the right detail is what the anchoring knows how to compare. In spec-as-source the validator is the generator, so the right detail is what the generator needs to produce unambiguous code.

Almost every pathology we'll see in chapter 12 comes from **misaligning these three things**: writing spec-anchored without anchoring ([#4](12-anti-patterns.md#4-theater-spec-with-fake-anchoring)), writing spec-as-source without a generator ([#9](12-anti-patterns.md#9-promoting-prematurely-to-spec-as-source) + [#12](12-anti-patterns.md#12-generated-spec-thats-pseudocode-in-disguise)), or writing spec-first with spec-as-source-level detail ([#2](12-anti-patterns.md#2-big-spec-up-front)). Anatomy isn't absolute — it's relative to what kind of validation will be applied to what you write.

## The spec and its external sources: consume, produce, modify

Every spec relates to artifacts that already exist — product documents, architectural decisions, code components. The taxonomy is always the same: **do you consume it, produce it, or modify it?** Understanding which relationship you have with each external artifact determines what goes in the spec, how much detail you include, and where it falls within the six blocks.

### Three relationships, three rules

**1. Consumed (it exists; the spec uses it without modifying it).** Applies to both code components (an existing service, a library) and documents (a user story, an ADR, an API contract). The rule is **reference the contract or artifact, don't reimplement it**. The spec documents *how* this feature uses it, not what the artifact does on its own — that lives in its own source of truth.

An important nuance: **cite the specific invariant this spec depends on, not the entire document**. *"This spec respects ADR-007 (single Postgres instance) and ADR-012 (no synchronous calls between services)"* is useful. *"Related: ADR-007"* is noise.

**2. Produced (it doesn't exist; the spec creates it).** The typical case for a new feature: *"this spec creates a new endpoint / a new service / a new module"*. The trap: because it doesn't exist yet, you feel you have to "define it" in the spec, and you almost always end up describing it with classes, methods, signatures and payloads. That's exactly [anti-pattern #12](12-anti-patterns.md#12-generated-spec-thats-pseudocode-in-disguise) (pseudocode dressed up as spec).

The right form: the spec describes the **observable contract** the new artifact must offer, not its internal structure. *"There must be a capability to accept authenticated avatar uploads (JPEG/PNG, ≤10 MB) and return a retrievable URL"* is observable contract. *"Create `AvatarUploadService` with method `upload(file, user_id) -> AvatarMetadata`"* is pseudocode.

**When a prior design document already exists** (a tech design, an interface document, an API contract designed before implementation), the component at the code level is *produced*, but at the design level it's *consumed*. In that case, the spec references the design document and describes only the implications for this feature — exactly as with any consumed artifact. The "define the observable contract" rule applies only when no prior design already does so.

A conceptually important nuance: a spec that produces a new artifact is **the transient source of truth that hands off**. Once the artifact exists, **its own documentation** becomes the operational source, and the spec becomes the *"why it was built this way"* — historical intent, not living contract. Confusing the two roles leads to the silent fusion of [anti-pattern #13](12-anti-patterns.md#13-fusing-user-story-and-spec-into-a-single-file).

**3. Modified (it exists; the spec adds, changes or removes capabilities).** The rule is **describe the observable delta, not the full state**. The spec doesn't have to re-document how `User` works; it has to say *"add the `avatar_url` field (optional). Updated when an avatar is uploaded; cleared when deleted. No other model behavior changes."*

The **"no other behavior changes"** part is what saves the spec from inflating. The delta is the spec; the rest lives in its own documentation.

> **In summary**: *consumed* = reference and describe only the use; *produced* = define the observable contract and prepare the handoff; *modified* = describe the delta and declare the rest invariant.

### Special case: product documents (user stories, briefs)

Product documents are *consumed* — the spec feeds on them but doesn't modify them. However, they have a particularity: **they speak a different language from the spec**. A user story says *"the user can easily upload a photo"*; the spec needs *"JPEG/PNG ≤10 MB, fails with 413/415 with readable message"*. Consuming a product document isn't copying — it's **deriving engineering criteria** from product criteria.

How to do it depends on whether the agent can access the original document:

**Without access** (Jira without MCP, Notion without API, a standalone document): **derive with traceability**. Write your own precise criteria in the spec and cite the user story in the "whys" section as the source of *what motivated the decision*. A bare reference (`"see JIRA-1234"`) breaks the self-containment of chapter 1 — the agent can't read that link, and the user story can change without anyone noticing.

**With access** (same Git repo or MCP to Jira/Linear/Notion): you don't need to copy — but you still need to **derive your own criteria at the spec's level of precision** and reference the origin. In this scenario drift stops being invisible (there's a commit or a timestamp), and an **automatic sensor** can detect when the user story changes without the spec being updated.

A concrete risk in both cases: the temptation to edit both documents frequently without each change being fully justified. Every cross-edit is an opportunity for misalignment. The trace serves a dual purpose: **checkpoint at the time of change** and **audit material** for a doc-gathering agent that scans for discrepancies on a recurring basis.

The rule: **the spec always contains its own criteria, derived at the level of precision validation needs. The upstream document is cited as the source of the why, not as the container of the what.** The anti-pattern to avoid — *fusing user story and spec into a hybrid* — we develop as [anti-pattern #13](12-anti-patterns.md#13-fusing-user-story-and-spec-into-a-single-file) in chapter 12.

### Special case: documents with automatic validation against code

When the consumed artifact is automatically validated against code on an ongoing basis (an OpenAPI contract with CI, a Protobuf schema with compatibility checks), it's enough to reference it and summarize the implications for your feature. A Jira user story has no automatic validation against code — its consistency depends on occasional human discipline, which makes it vulnerable to silent drift.

### How much detail: three dimensions

For each artifact the spec produces or modifies, the question is *"how much detail?"*. The useful answer is to ask **three explicit questions** and capture only what's load-bearing:

- **Functional** — what changes in observable behavior. *"The `User` model can now have an avatar; the field is optional; updated on upload, cleared on delete."*
- **Non-functional** — which cross-cutting constraints the change must respect. *"The migration must not require downtime; the field must not break existing serialization."*
- **Technical** — which contract/integration decisions the change explicitly closes. *"The field is indexed by `user_id`; the URL goes through the existing CDN."*

For each dimension, an **optional reference to the deeper document** if it exists. If a dimension doesn't add anything, omit it — absence is informative.

Two clarifications:

1. **What goes in the spec is what's load-bearing**: what the chapter 6 validator can compare against the code. Finer detail lives in the artifact's documentation.
2. **The term "non-functional" is classical but contested** ([Fowler](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html) argues everything is functional, just along different dimensions). We use it as an operational category — three different questions — not as ontology.

### Where this information goes in the six blocks

The consume/produce/modify taxonomy and the three dimensions don't create a new section in the spec — they help you write the sections you already have better:

- **Consumed artifacts** (components, ADRs, contracts, user stories) → go in **technical constraints** (the bounds) and in **whys** (the motivation derived from upstream).
- **Produced or modified artifacts** → their observable guarantees go in **acceptance criteria**; their design bounds go in **technical constraints**.
- **The three dimensions** are the questions you ask when writing each criterion or constraint, not subsections within the spec.

### Size heuristic: when the spec needs splitting

If while writing constraints and criteria you discover the spec produces or modifies more than **3-4 artifacts**, you probably have one of three problems:

1. A feature too big that should be split into multiple specs (chapter 9 modulation).
2. An architectural change disguised as a feature, which deserves its own ADR.
3. A refactor camouflaged as a feature (also chapter 9 modulation): what you're changing is the *shape* of the system, not adding business capability.

### Summary table by relationship type

| Relationship | Main rule | Spec section | Typical trap |
|---|---|---|---|
| **Consumed component** | Reference the contract; document the use | Technical constraints | Re-documenting the component |
| **Produced component** | Define observable contract, not structure | Criteria + Constraints | Pseudocode (#12); not marking the handoff |
| **Modified component** | Describe the delta, declare the rest invariant | Criteria + Constraints | Rewriting the entire component |
| **ADR / architecture** | Cite specific invariants | Technical constraints | Copying invariants (drift guaranteed) |
| **API contract / standard** | Reference + summary of implications | Technical constraints | Re-documenting the contract |
| **Product doc / brief** | Extract the cardinal whys | Whys | Importing vague product language |
| **User story** | Derive criteria with traceability | Criteria + Whys | Verbatim copy or opaque reference |

And an anti-pattern that appears when references accumulate without discipline: **reference soup** — a spec drowned by its own citations, where the agent and the human don't know which are load-bearing and end up ignoring them all. We develop it as [anti-pattern #14](12-anti-patterns.md#14-reference-soup) in chapter 12. The preventive rule: **reference only what the agent or the human needs for this task, and annotate the why of each reference**.

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
