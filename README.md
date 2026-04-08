# Lore: A Convention-Over-Configuration Knowledge Framework for AI-Driven Development

## The Core Insight

The next revolutionary dev framework won't be just a dev framework — it will be a **knowledge framework**. The main challenge in AI-assisted development is how we organize project knowledge so the LLM can build the app the way we want. Just as Rails liberated developers from a thousand decisions about where to put things, a curated set of folders and instructions for communicating with an LLM will be the future equivalent.

## The Inversion

Traditional projects treat documentation as a second-class citizen — an afterthought stuffed in a `/docs` folder nobody maintains. Lore inverts the mental model: **knowledge is the primary artifact, code is the implementation**.

The project stops being "a codebase with some docs" and becomes "a body of knowledge with a compiled output." The code is almost a build artifact of the knowledge.

This matters because:

- **Code rots. Knowledge survives.** Implementations get rewritten, frameworks change, languages evolve. But domain knowledge, architectural decisions, and workflow understanding survive across rewrites.
- **Docs stay alive because they're load-bearing.** Today docs rot because they don't *do* anything. When docs are the primary input to AI code generation, an outdated doc means wrong code. That creates natural pressure to keep them accurate.
- **The feedback loop changes.** Write the knowledge → AI generates code → notice the output is wrong → fix the knowledge. Documentation becomes a living spec validated through generation.

## The Scaffold

The knowledge tree lives in a `/lore` folder at the project root, alongside the existing code structure. This works with any project — new or existing — without disrupting CI/CD, deploy scripts, or any tooling that expects standard layouts.

```
my-app/
  lore/
    project.md
    domain/
    decisions/
    conventions/
    workflows/
  app/              ← standard code structure, untouched
  config/
  db/
  ...
```

Four knowledge folders inside `/lore`. Each answers a different question the LLM might have:

| Folder | Question it answers |
|---|---|
| `project.md` | What is this thing? |
| `domain/` | What does the business care about? |
| `decisions/` | Why did we choose X over Y? |
| `conventions/` | How do we do things here? |
| `workflows/` | How do things flow end to end? |

Adoption is trivial for existing projects. Add one folder. No restructuring, no migration.

## Anchors, Not Walls

Each folder has *suggested anchor files* — starting points that work for most projects. But anchors are not walls. If the knowledge wants a different shape, follow the knowledge.

- A domain folder usually has `entities.md` and `rules.md`. It can also have `partial-postings.md` if a concept needs its own narrative space.
- A workflow file usually describes a flow. It can also embed a state machine, reference constants, or include a sub-flow if those are tightly coupled to understanding the flow.
- A conventions folder usually contains files for the six default categories. It can also contain `async-ui.md` or `navigation.md` documenting specific machinery the team built.

Templates are starting shapes. Worked examples and embedded code are first-class — real knowledge is rarely pure prose. The framework's job is to make sure the LLM (and developers) can find what they need, not to enforce a rigid taxonomy.

The litmus test for any file is the same: **would the LLM generate noticeably worse code without it?** If yes, it earns its spot. If the information is already inferrable from the code itself, skip it.

## The Framework as a Skill

The framework itself is an LLM skill called **Lore** that lives outside the project. No CLI to install, no tooling dependency. The skill contains the output templates, conventions, and capture logic — everything needed to scaffold and maintain a project's knowledge tree.

### Skill structure

```
lore/
  SKILL.md
  templates/
    project.md
    domain-entities.md
    domain-rules.md
    feature.md
    workflow.md
    decision.md
  extraction-guides/
    patterns.md
    naming.md
    testing.md
    error-handling.md
    frontend.md
    api.md
```

`SKILL.md` is the brain — it contains the scaffold rules, the capture logic, the extraction logic, and the guidance on what goes where. The templates define the output format for each knowledge file type — they're instructions to the LLM, not blanks for the user to fill in. The extraction guides tell the LLM what to look for in a codebase (and what to ask in a conversation) when populating each convention category.

### Zero framework artifacts in the project

When you run `/lore init`, the LLM reads the skill and creates a clean `lore/` folder inside the project. No `SKILL.md`, no `templates/`, no `extraction-guides/`, no meta files in the project itself. The framework lives outside; only the knowledge it produces lives inside.

## The Capture Command

Knowledge files are not created from empty templates. They're the output of conversations. You discuss a domain, debug a flow, make a technical choice — and at the end of that session, the LLM captures what was discussed into properly structured files.

### How it works

`/lore capture` reads the current conversation, extracts the relevant knowledge, and writes it into the right files following the output templates. The conversation is the input. The structured files are the output. The skill is the translation layer.

- **`/lore capture domain [name]`** — Extracts entities, properties, states, relationships into `entities.md`. Extracts business rules, constraints, invariants into `rules.md`. Places both in `lore/domain/[name]/`. If the conversation included a substantial concept that needs its own narrative space, also creates a file in `lore/domain/[name]/features/`.
- **`/lore capture workflow [name]`** — Extracts trigger, purpose, steps, failure modes, domains involved into `lore/workflows/[name].md`. Workflows can embed reference material (state machines, constants, sub-flows) when it's tightly coupled to the flow.
- **`/lore capture decision [name]`** — Extracts context, options considered, rationale, and date into `lore/decisions/[name].md`.
- **`/lore capture convention [category]`** — Extracts a convention from the conversation and merges it into `lore/conventions/[category].md`. The complement to `/lore extract` for when conventions emerge from discussion rather than from reading code.
- **`/lore capture feature [domain]/[name]`** — Extracts a feature concept into `lore/domain/[domain]/features/[name].md`. Features come in two flavors: permanent concept docs (worked examples, edge cases, narrative explanation) and work-in-progress scoping docs (requirements, open questions). The same folder holds both.

### Why capture, not generate

Traditional generators create empty files for you to fill in. But knowledge files are never written that way. They emerge from conversation — from debugging sessions, planning discussions, stakeholder meetings, Slack threads. The knowledge already exists in the conversation. The skill's job is the last mile: take everything we just discussed and write it down properly.

This works even when the conversation wasn't explicitly about building that artifact. You might spend an hour debugging receipt matching and discover three new business rules along the way. At the end, `/lore capture domain receipts` captures what you learned.

### Init creates the skeleton; extract and capture fill it

`/lore init` is the one-time scaffolder — it creates the empty `lore/` folder structure and the `project.md`. After that, the knowledge tree grows two ways: extracted from existing code, or captured from conversation. Both produce the same shape of knowledge — the difference is the input.

## The Extract Command

Most teams aren't starting from scratch. They have an existing codebase with years of accumulated conventions that nobody wrote down. `/lore extract` reads that codebase and produces a first draft of the conventions folder.

### How it works

`/lore extract [category]` reads the relevant extraction guide, samples representative files from the directories the guide points to, and produces `lore/conventions/[category].md` as a draft. The default categories are **patterns**, **naming**, **testing**, **error-handling**, **frontend**, and **api** — each backed by an extraction guide that tells the LLM where to look, what's safe to extract, what to flag for human review, and what to output.

The output is structured in three confidence layers:

- **High confidence** — observable facts written as conventions.
- **Medium confidence** — patterns surfaced as "we found this — is it intentional?" for the team to confirm or correct.
- **Discussion needed** — philosophy and principle questions that can't be answered from code alone, listed for the team to capture later.

Inconsistencies are surfaced explicitly. If services use `.call` in 23 files and `.perform` in 4, the draft says so — inconsistency is itself useful information.

### Why extract produces drafts, not finished documents

Code reveals the *what*; humans provide the *why* and the *should*. The codebase shows you that 23 services use `.call` and 4 use `.perform`, but it can't tell you whether the 4 are legacy being migrated, an intentional exception, or just inconsistency nobody noticed. Extract surfaces the observation and stops there. The team fills in the interpretation.

There's also an aspirational gap. Code shows you the present; conventions are forward-looking instructions for what to generate next. A team mid-migration has *current* conventions in their code that contradict where they're heading. Extraction can't see the heading.

The honest framing: extract gets you a 60% draft with the questions made obvious. A 30-minute team conversation closes the gap, and `/lore capture convention` records what was decided.

## Folder Ownership: Framework vs Project

The framework ships **opinions about which questions to ask**, not opinions about what the answers should be. Everything inside `lore/` is project-owned. The framework lives outside the project as a skill, and contributes to the knowledge tree through templates and extraction guides, not through pre-filled content.

### What the framework ships

- **Templates** — suggested output formats for each kind of knowledge file. They're starting shapes, not constraints.
- **Extraction guides** — for each of the six default convention categories, instructions on what to look for in a codebase, what's safe to extract, what to flag for review, and what needs human input.
- **The scaffold structure itself** — the five folders, the conventional anchor files, the philosophy.

### What the team owns

Everything inside the project's `lore/` folder. Domain knowledge is yours because only you know your business. Decisions are yours because only you made them. Workflows are yours because only you know how things flow end-to-end. **Conventions are also yours**, because every team builds differently — even within the same framework — and shipped conventions either get thrown away or quietly ignored.

### Why no shipped Rails opinions

An earlier version of this concept imagined the framework shipping "flavors" — Rails conventions, Next.js conventions, Phoenix conventions — that teams would inherit on day one. The intuition was that strong defaults make new projects productive immediately, the way Rails does.

It didn't survive contact with reality. Two Rails teams can use ViewComponents or Phlex; RSpec or Minitest; service objects or interactors or fat models; JSON:API or plain REST. Shipping any of those as "the default" means most teams throw it away on day one. That's not convention-over-configuration — that's configuration disguised as convention. Worse, when the LLM follows pre-filled conventions that don't match how the team actually builds, it generates code that looks right but doesn't fit, which is harder to detect and fix than no convention at all.

So the framework's opinion shifted: be opinionated about *which conventions are worth documenting*, not about *what the conventions should be*. The six extraction guides are that opinion.

### Upgrades stay clean

Because the framework doesn't ship content into the project, framework upgrades touch nothing. New version of the skill? The project doesn't change. You re-run extract or capture if you want to adopt new patterns the framework now supports, but the existing knowledge tree is untouched.

## How the Knowledge Tree Fills Up

When you run `/lore init`, you get an empty scaffold — five folders and a `project.md`. Nothing pre-filled. From there, the tree grows two ways:

1. **Extracted from existing code** — `/lore extract` reads the codebase and produces draft conventions for whichever categories apply. This is how most existing projects bootstrap, since they already have years of accumulated patterns waiting to be documented.
2. **Captured from conversation** — `/lore capture` distills knowledge from ongoing discussions: domain explorations, debugging sessions, decision moments, design reviews. This is how the tree keeps growing after the initial bootstrap.

The two are complementary. Extract gets you a starting draft fast; capture refines and extends it as you actually use the project. Neither replaces the other.

## Deep Dive: `domain/`

The heart of the project. Each domain concept gets its own folder containing the business reality — not database schemas or class definitions.

### Structure

```
lore/domain/
  payments/
    entities.md
    rules.md
    features/
      recurring-billing.md
  receipts/
    entities.md
    rules.md
    features/
      email-parsing.md
  cards/
    entities.md
    rules.md
```

Each domain folder has:

- **`entities.md`** — What exists in this domain and how it's shaped. The business entities, their properties, their states, their relationships. Encodes the ubiquitous language (DDD). When the LLM works on a payment feature, this is the source of truth for what's correct.
- **`rules.md`** — What must always be true. Business rules that govern this domain. "A payment cannot be processed if the card is expired." "A receipt can only be matched to a transaction within a 3-day window." These are the things scattered across validations, conditionals, and service objects in a typical codebase — the stuff that's hardest to infer from code and most dangerous to get wrong.
- **`features/`** — Concepts within the domain that need their own narrative space. The test is **"does this need its own narrative space?"** — not "is this temporary?" Features come in two legitimate flavors that share the same folder: **permanent concept docs** (worked examples, edge case walkthroughs, argumentative explanation — "Partial Postings explained, with worked examples for underpayment and overpayment"), and **work-in-progress scoping docs** (requirements, open questions, edges of a feature being built). Permanent concept docs stay forever because the narrative is the knowledge. WIP scoping docs naturally dissolve over time as their content gets absorbed into entities, rules, or workflows. The framework doesn't try to distinguish them at the file level — the lifecycle handles itself.

### Context loading note

Features live inside domain folders because that's the natural mental model for developers — open `lore/domain/owner-statements/` and see everything about owner statements. However, when loading context for the LLM, `entities.md` and `rules.md` are the core files you always want. Feature files should only be loaded when the work touches that specific concept (for permanent concept docs) or when the work is the feature being scoped (for WIP scoping docs). The `project.md` can guide LLMs on this: "when working on a domain, load entities and rules; only load feature files when the work touches that specific concept."

### Growth pattern

Start flat and simple. Day one a domain might just have `entities.md` with a few lines. Rules emerge as you build. Features get created when you start scoped work. Let nesting emerge from need, not from structure.

Both `entities.md` and `rules.md` start sparse and grow as you implement. Every edge case discovered, every business decision made, adds a line or two. The habit is the same as workflows: **update the domain docs before you commit the code.**

### How bugs feed into domain

Not every bug produces knowledge. Most are just bugs — fix them and move on. But some bugs reveal business truths you didn't know. "We discovered that some banks report transactions with a 48-hour delay, so matching by date needs a wider window." That's a rule. It goes in `rules.md` for that domain. The bug ticket gets closed, but the knowledge survives.

The filter is the same litmus test: **would the LLM generate worse code without this knowledge?** A business rule about bank transaction delays — yes. A CSS alignment fix — no.

## Deep Dive: `decisions/`

The project's institutional memory. An append-mostly log of technical and business choices — why things are the way they are.

Critical for the LLM because without it, the AI has no memory of paths already explored and rejected. It might suggest pulling notifications into a microservice — something you spent a week evaluating and decided against six months ago. The decision log says "we already thought about this, here's why we said no."

Each decision is short and follows a simple format:

```markdown
# Use Sidekiq over Solid Queue

## Context
We needed background job processing for async receipt parsing.

## Options considered
- Solid Queue (Rails default, simpler)
- Sidekiq (mature, battle-tested, better monitoring)

## Decision
Sidekiq. We need the monitoring dashboard and
the retry semantics are more predictable for our use case.

## Date
2025-03-15
```

Decisions are append-mostly. You almost never edit an existing one. You add new ones, and occasionally supersede an old one with a new decision that references it. It's a log, not a document.

This is the easiest folder to maintain. Made a technical choice? Two minutes. Never touch it again. But it saves hours of going in circles when the LLM (or a new team member) encounters that same decision space later.

### How bugs feed into decisions

Some bugs reveal technical constraints or limitations. "We found that Sidekiq silently drops jobs over 50MB, so we added a size check before enqueuing." That's a technical decision made in response to a real problem. It goes in `decisions/` so the LLM doesn't generate code that enqueues oversized payloads in the future.

## Deep Dive: `conventions/`

How this team builds. Project-owned and built up over time through `extract` and `capture`.

The framework provides extraction guides for six default categories. Each guide answers a question the LLM will keep facing as it generates code:

| Category | Question it answers |
|---|---|
| `patterns.md` | Where does business logic live and how is it organized? |
| `naming.md` | What do we call things, beyond what the framework dictates? |
| `testing.md` | How do we verify code works? Framework, philosophy, what gets tested. |
| `error-handling.md` | How do errors flow through the system? |
| `frontend.md` | How is the UI built? Components, styling, interactivity. |
| `api.md` | How does the system talk to the outside world? |

These six are defaults, not a fixed taxonomy. A backend-only service has no `frontend.md`. A project may need others (auth, observability, data) that emerge from its specific work. The framework is opinionated about which questions are usually worth asking, not about which answers are correct.

### Why these six

These are the gaps the framework leaves open. Rails decides MVC, ActiveRecord, routing, mailers, jobs — but it doesn't tell you where business logic lives, how to name your services, how to test, how to handle errors, how to build the UI, or how to shape your API. Two Rails teams can answer any of those completely differently and both have clean codebases. The LLM needs to know which answer applies *here*.

### Conventions also contains specific machinery

In addition to the six category files, real conventions folders contain documentation for specific abstractions the team built — a homegrown concern, a component catalog, a custom DSL. These aren't subcategories of patterns or frontend; they're their own files alongside them. `conventions/async-ui.md`, `conventions/navigation.md`, `conventions/stateful-params.md`. The framework gives permission for this; the team decides what's worth documenting.

The relationship between the category files and the machinery files is that the categories establish the principles ("we have a service layer that uses the Service concern, services return result objects with success?/failure?") and the machinery files document the specific instances those principles refer to ("here's exactly how AsyncUi works, here's how to wire it up").

### Conventions are functional, not decorative

These aren't static documentation. They directly influence what the LLM generates. When the conventions say "services return result objects with success? and failure?", that's an instruction the AI follows on every service it writes. Conventions have teeth in a way they never did before — which is also why they have to be accurate. An out-of-date convention generates wrong code on every prompt that touches it.

## Deep Dive: `workflows/`

The connective tissue between domain concepts. Fills a gap nothing else covers: **what actually happens when a user does something?**

### Why workflows matter

- Domain files describe things in isolation. Workflows describe how they compose for real user goals.
- This is exactly where LLMs fail most — generating locally correct but globally broken code because the orchestration knowledge is spread across controllers, services, callbacks, and jobs.
- Every developer already explains these flows daily — in Slack, PR descriptions, onboarding sessions, debugging. That knowledge just evaporates. Workflows give it a home.

### The key principle: workflows evolve with the code

Workflows are NOT written as complete specs before implementation. They start sparse and grow as you build. Day one might be:

```markdown
# Receipt Processing

## Purpose
Match incoming receipts to card transactions
so users don't have to do it manually.

## Trigger
Email received at receipts@...

## Expected outcome
Receipt is matched to a transaction, user is notified.

## Open questions
- What formats will we support?
- What happens when there's no match?
```

As you implement, the workflow grows. You discover edge cases, make decisions, add them. The habit is simple: **update the workflow before you commit the code.**

This works because:

- It's lightweight — a few lines per decision, ten seconds of work.
- It's self-interest, not discipline — if you don't update it, the AI works from stale knowledge next time.
- The LLM can do it for you — "implement this and update the workflow" in one prompt.
- Git history tells the full story — `git log lore/workflows/receipt-processing.md` shows every decision and when it was made.

### The selling concept

**Workflows don't create new work — they capture work you're already doing in a place where it's actually useful.** Every PR description, every Slack explanation of a flow, every debugging session where you trace through async code — that knowledge already exists. It just disappears. Workflows are where it compounds instead of evaporating.

### Scope

Workflows represent meaningful business processes that cross domain boundaries. "User clicks save" is not a workflow. "User onboards onto the platform" is. A mid-sized app might have 10-20 workflows.

### Workflows can embed reference material

A workflow file isn't restricted to a flat list of steps. When the flow can't be understood without supporting material, that material belongs *in* the workflow file rather than being scattered across other folders. This includes:

- **State machines** for the entity the workflow drives, when the workflow is what causes the transitions.
- **Constants and thresholds** that govern the flow's behavior (`MAX_RETRIES = 5`, `TIMEOUT_SECONDS = 30`).
- **Sub-flows** like a failure-escalation path with its own internal logic.
- **Components overview** showing which code participates, when the flow crosses many files.
- **Diagrams** — Mermaid is preferred for state machines and sequence flows; ASCII art when Mermaid can't express the shape.

The principle: facts have a primary home, but tightly coupled facts can be co-located where they're most useful. A state machine that only makes sense in the context of a specific workflow doesn't need to be split into `domain/entities.md` for purity's sake.

## Two Flavors of Features

Files in `lore/domain/*/features/` come in two legitimate flavors, distinguished by their lifecycle, not their location:

**Permanent concept docs.** A piece of the business that's substantial enough to warrant a dedicated explanation — worked examples with real numbers, edge case walkthroughs, an argument for why a distinction matters. The narrative *is* the knowledge. These don't dissolve over time; they stay until the underlying business reality changes. "Partial Postings explained, with worked examples for underpayment and overpayment" is one of these.

**Work-in-progress scoping docs.** A piece of work being planned or built — requirements, edge cases, open questions. These often dissolve over time as their content gets absorbed into entities, rules, or workflows. By the time the work is done, what's left is either reference material or nothing. "PDF generation for owner statements: scope, requirements, open questions" is one of these.

The framework doesn't try to distinguish them at the file level. Both live in `features/`. The shape each takes depends on what kind of knowledge it holds.

### Git branching handles WIP maturity

For the work-in-progress flavor, the maturity gradient is real — a feature starts as a rough draft and matures through implementation into reliable knowledge. Git branching handles this naturally. A feature being developed lives in a branch. By the time the PR merges, the doc has matured through the development process itself. No need for separate draft/active folders or status metadata — git's existing branching model solves the maturity problem.

For the permanent concept flavor, there's no lifecycle to manage. The doc is written once and edited as the business reality changes — the same as any other piece of long-lived knowledge.

## What about contracts?

Contracts (API boundaries, integration interfaces, event schemas) are NOT a default folder. Strong conventions reduce the need for per-instance documentation. If conventions say "always wrap third-party services, never call them directly," then the Stripe wrapper is just code that follows the convention.

Contracts become an optional add-on when a project reaches certain complexity (multiple services that need to stay in sync, for example). Generated on demand, not part of the default scaffold.

## What about bugs and tech debt?

Bugs and tech debt are **transient** — they belong in project management tools (GitHub Issues, Linear, Jira), not in the knowledge tree.

However, some bugs produce genuine knowledge:

- **Business discoveries** go into `lore/domain/*/rules.md`. "We discovered that banks report transactions with a 48-hour delay" is a rule, not a bug.
- **Technical constraints** go into `lore/decisions/`. "Sidekiq silently drops jobs over 50MB" is a decision to document, not just a ticket to close.

Most bugs produce no knowledge at all. Fix them and move on. The filter is always the same: **would the LLM generate worse code without this knowledge?**

### Litmus test for any file

**Would the LLM generate noticeably worse code without it?** If yes, it earns its spot. If the information is already inferrable from conventions plus source code, skip it.

## How This Differs from Existing Approaches

### vs Spec Kit (GitHub)

Spec Kit is a **process tool** — a sequence of steps (specify → plan → tasks → implement). Specs are per-feature and organized by feature, not by domain. No persistent conventions layer. Stack-agnostic by design.

Lore is a **knowledge tool** — persistent, cumulative knowledge that grows with the project. Domain knowledge doesn't belong to a feature; it grows across every feature that touches that domain. Conventions are project-owned and built up through extraction and capture, not prescribed by the framework.

### vs BMAD Method

BMAD is a **methodology framework** — traditional agile mediated by AI agents (product briefs, PRDs, epics, sprint planning). Very comprehensive, very structured.

Lore is lighter. It doesn't prescribe a development process. It gives you a knowledge scaffold, an extraction tool to bootstrap from existing code, and a capture command to grow the knowledge tree as you work — then gets out of the way.

### The gap

Both existing approaches are process-first. Neither has a persistent, evolving knowledge layer that's part of the project forever. Neither separates *which questions to ask* from *what the answers should be* — Lore is opinionated about the former and silent about the latter, which is what makes the knowledge tree teams produce actually fit how they build. That's the space Lore occupies.