# Extraction Guide: Patterns

## Question this convention answers

**Where does business logic live and how is it organized?**

Frameworks (Rails, Django, etc.) decide a lot, but they don't decide this.
Two teams using the same framework can have completely different answers,
and the LLM needs to know which answer applies *here* before writing any
non-trivial code.

The sub-questions hiding inside this big one:

- Is there a service layer? What's it called, where does it live?
- What's the interface? `call`, `perform`, `execute`, something else?
- One method per service or many?
- How are services namespaced — by domain, by type, flat?
- What do services return — the model, a result object, a boolean, raise on
  failure?
- How fat are models allowed to get?
- Are concerns used? For sharing behavior, organizing files, both?
- Where do complex queries live — model scopes, query objects, both?
- Are there form objects, presenters, decorators, repositories?
- Where does validation happen — model, form object, both?

The LLM faces these questions every time it writes a piece of business
logic. Without documented answers, it guesses, and the guesses are
inconsistent across prompts.

---

## Where to look

In a Rails project (the principles transfer to other frameworks):

- **`app/services/`** — does it exist? Sample 5-10 files. Look for the
  interface (`def call`, `def perform`, `def run`), the namespacing
  (`Payments::ProcessCharge` vs flat `ProcessChargePayment`), the return
  shape (does it raise, return a model, return a result object?).
- **`app/models/`** — sample a few of the larger models. Count methods.
  Are they fat (50+ methods) or thin? Do they include concerns? What for?
- **`app/controllers/`** — sample 5-10 actions. Are they thin (under 10
  lines) or thick? Where do they delegate to?
- **`app/queries/`, `app/operations/`, `app/interactors/`, `app/forms/`,
  `app/decorators/`, `app/presenters/`** — does any of these exist?
  Sample a few files from each that does. Identify the convention.
- **`app/concerns/` or concerns inside `app/models/concerns/` and
  `app/controllers/concerns/`** — what's in them? Behavior sharing?
  Domain logic extraction? Mixin-style organization?
- **`lib/`** — anything in here that's actually business logic, or is it
  truly library code?
- **A `Service` or similar concern** — search for `include Service`,
  `include Operation`, etc. If a base concern exists, read it. The base
  concern often defines the team's whole pattern.

Don't try to read everything. A large app has thousands of files. Read
enough to identify the dominant pattern and any major exceptions.

---

## High-confidence extractable

These are observable facts. Write them as conventions directly:

- **Which directories exist** under `app/` beyond the Rails defaults.
- **The interface convention** for services (or whatever the team calls
  them). Method name, arguments, return type.
- **The namespacing convention.** Flat vs nested, by domain vs by type.
- **The base concern or class** that services include or inherit from,
  and what it provides (validations, params, result wrapping).
- **The return shape.** Does the team use result objects? Raise exceptions?
  Return the modified record?
- **Model size.** Average method count, presence of concerns, fat-vs-thin
  tendency.
- **Whether query objects exist** and roughly when they're used (search
  pages, complex filtering, reports).
- **Counts and outliers.** "47 services follow `Module::VerbNoun.call`,
  3 don't. The outliers are in `app/services/legacy/`." Inconsistency is
  itself useful information — surface it.

---

## Medium-confidence (surface for confirmation)

These look like patterns but the LLM can't be certain they're intentional.
Write them as "we found this — is this your convention?":

- **The rule for when to use a service vs put logic on a model.** You can
  see that some logic is in services and some is on models. You can't tell
  from code whether the split was deliberate or accidental.
- **The rule for when to extract a query object vs add a scope.** Same
  problem.
- **Implicit conventions in concern usage.** Concerns might be doing real
  organizational work, or they might be a junk drawer. Hard to tell from
  outside.
- **Whether the codebase is mid-migration.** If you find two patterns
  coexisting, ask whether one is replacing the other.

---

## What needs human input

These don't live in the code at all:

- **The principle behind the structure.** "Services coordinate multiple
  models, single-model logic stays on the model" is a rule that lives in
  the team's heads. You can't infer it from observation.
- **The aspirational direction.** Where the team is heading, even if the
  current code doesn't reflect it yet. "We're moving away from service
  objects toward..."
- **Why specific abstractions exist.** "We built a query object layer
  because Ransack wasn't expressive enough for our search pages" — that
  why isn't in the code.

When extraction surfaces these gaps, list them in the output as
"Discussion needed" so the team can fill them in via `/lore capture`.

---

## Questions for capture mode

When there's no codebase to read, or when extract surfaces gaps, ask:

- "Where does business logic live in this app?"
- "Walk me through how you'd add a new feature that touches multiple
  models. Where does the orchestration go?"
- "When do you reach for a service object vs just adding a method to a
  model?"
- "Do you have a result/response wrapper that services return? What does
  it expose?"
- "How do you handle complex queries — scopes, query objects, both?"
- "Are there any structural conventions you wish were stricter or looser?"
- "Is anything in the codebase a pattern you're moving away from?"

---

## Output structure

Write to `lore/conventions/patterns.md`. Suggested shape:

```markdown
# Patterns

## Service layer

{What exists, where it lives, what the interface looks like, what it returns.
Use a small code example pulled from the actual codebase.}

### When to use a service

{The rule, if known. Mark "Discussion needed" if unknown.}

## Models

{Fat or thin, concern usage, what kinds of methods belong on models.}

## Queries

{Scopes vs query objects, when each applies.}

## Other layers

{Form objects, decorators, presenters, etc. — only if they exist in this
codebase.}

## Inconsistencies / outliers

{Things that don't match the dominant pattern. Explicit, not hidden. The
team can decide whether they're legacy, intentional, or worth cleaning up.}

## Discussion needed

{Things extract couldn't determine from code alone. Each one is a prompt for
a future capture session.}
```

The output should look like a *draft for discussion*, not a finished
document. Be honest about confidence levels. The team turns it into a
finished document through review and capture.
