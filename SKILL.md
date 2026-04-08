---
name: lore
description: >
  Knowledge framework for AI-driven development. A convention-over-configuration
  scaffold where project knowledge (domain, decisions, conventions, workflows)
  sits above code as the primary source of truth. Use this skill whenever the
  user mentions "lore", uses /lore commands, asks to initialize a knowledge-first
  project, wants to extract conventions from an existing codebase, wants to
  capture domain knowledge or decisions from a conversation, or discusses
  structuring project knowledge for LLM-assisted development. Also trigger when
  the user asks about organizing project context, creating decision logs,
  documenting business rules, or setting up a knowledge tree for AI code
  generation.
---

# Lore

A convention-over-configuration scaffold for AI-driven development. The project
becomes a **body of knowledge with a compiled output**, not a codebase with docs
bolted on. Code is the implementation. Knowledge is the source of truth.

## Why this matters

- **Code rots. Knowledge survives.** Implementations get rewritten, frameworks
  change. Domain knowledge, architectural decisions, and workflow understanding
  persist across rewrites.
- **Docs stay alive because they're load-bearing.** When docs are the primary
  input to AI code generation, an outdated doc means wrong code. That creates
  natural pressure to keep them accurate.
- **The feedback loop changes.** Write the knowledge → AI generates code →
  notice the output is wrong → fix the knowledge.

---

## The Scaffold

Lore lives in a single `lore/` folder at the project root, alongside the
existing code structure. It works with any project — new or existing — without
disrupting CI/CD, deploy scripts, or any tooling that expects a standard layout.

```
my-app/
  lore/
    project.md          # What is this thing?
    domain/             # What does the business care about?
    decisions/          # Why did we choose X over Y?
    conventions/        # How do we do things here?
    workflows/          # How do things flow end to end?
  app/                  ← standard code structure, untouched
  config/
  db/
  ...
```

Adoption is trivial for existing projects. Add one folder. No restructuring,
no migration, no rename of existing directories.

---

## Philosophy: Anchors, Not Walls

Each folder has *suggested anchor files* — starting points the framework
recommends because they're useful in most projects. But anchors are not walls.
If the knowledge wants a different shape, follow the knowledge.

- A domain folder usually has `entities.md` and `rules.md`. It can also have
  `partial-postings.md` if a concept needs its own narrative space.
- A workflow file usually describes a flow. It can also embed a state machine,
  reference constants, or include a sub-flow if those are tightly coupled to
  the flow being described.
- A conventions folder usually contains the six categories below. It can also
  contain `async-ui.md` or `navigation.md` documenting specific machinery the
  team built.

Templates are starting shapes. Worked examples and embedded code are welcome —
real knowledge is rarely pure prose. The framework's job is to make sure the
LLM (and developers) can find what they need, not to enforce a rigid taxonomy.

The litmus test for any file: **Would the LLM generate noticeably worse code
without it?** If yes, it earns its spot. If the information is already
inferrable from the code itself, skip it.

---

## Commands

### `/lore init`

Creates the `lore/` folder structure inside an existing or new project.

**What it does:**

1. Ask the user for the project name and a short description if not already
   clear from context.
2. Create the folder skeleton:

```
lore/
  project.md          ← generated from templates/project.md
  domain/             ← empty
  decisions/          ← empty
  conventions/        ← empty (filled later via extract or capture)
  workflows/          ← empty
```

3. Fill `project.md` using `templates/project.md` as the output format.
4. Tell the user what comes next: run `/lore extract` if there's an existing
   codebase to learn from, or start having conversations and use `/lore capture`
   to distill knowledge as it emerges.

**No conventions are pre-filled.** The framework does not ship opinions about
how to build a Rails (or any other) app. Conventions are how *your* team builds,
and they get populated through `extract` and `capture` — not from a template.

---

### `/lore extract`

Reads an existing codebase and produces a first draft of the conventions
folder. The codebase already encodes how the team builds — extract surfaces
that into documentation.

**Usage:** `/lore extract` or `/lore extract <category>`

**Categories:** `patterns`, `naming`, `testing`, `error-handling`, `frontend`,
`api`. Each has an extraction guide in `extraction-guides/<category>.md` that
tells the LLM where to look in the code, what to extract with high confidence,
what to flag for human review, and what to output.

**What it does for each category:**

1. Read the relevant extraction guide.
2. Sample representative files from the directories the guide points to. Don't
   try to read everything — large codebases are too big. Read enough to identify
   the dominant patterns.
3. Produce `lore/conventions/<category>.md` with three layers of confidence:
   - **High confidence** — observable facts written as conventions.
   - **Medium confidence** — patterns surfaced as "we found this — is it
     intentional?" for the team to confirm or correct.
   - **Discussion needed** — philosophy and principle questions that can't be
     answered from code alone, listed for the team to capture later.
4. Note any inconsistencies. If services use `.call` in 23 files and `.perform`
   in 4, surface that — inconsistency is itself useful information.

**Important:** Extract produces drafts, not finished documents. The output is a
seed for a 30-minute team conversation, not a substitute for one. Be honest
about what was observed and what was guessed.

---

### `/lore capture`

Distills knowledge from the current conversation into properly structured
files. The conversation is the input. The structured files are the output.
The skill is the translation layer.

Knowledge files are never written from empty templates. They emerge from
conversation — debugging sessions, planning discussions, design reviews, slack
threads. The knowledge already exists in the conversation. Capture is the
last mile.

**Subcommands:**

#### `/lore capture domain <name>`

Extract domain knowledge and write it into `lore/domain/<name>/`.

1. Read `templates/domain-entities.md` and `templates/domain-rules.md`.
2. Extract entities, properties, states, relationships → `entities.md`.
3. Extract business rules, constraints, invariants → `rules.md`.
4. If the conversation included a substantial concept that needs its own
   narrative space (worked examples, edge case walkthroughs, argumentative
   explanation), create a `features/<concept>.md` file for it instead of
   trying to compress it into bullet points. Read `templates/feature.md`
   for guidance on this.
5. If files already exist, merge new knowledge — never overwrite. Append,
   refine, or update specific sections.

#### `/lore capture workflow <name>`

Extract a workflow and write it into `lore/workflows/<name>.md`.

1. Read `templates/workflow.md`.
2. Extract trigger, purpose, steps, failure modes, domains involved.
3. Workflows can embed reference material that's tightly coupled to the flow:
   state machines, constants, sub-flows, small architectural sketches. Don't
   force this material into other files if it's clearly part of understanding
   the workflow.
4. Workflows start sparse and grow. Capture what's known, mark what's unknown
   as open questions.

#### `/lore capture decision <name>`

Extract a decision and write it into `lore/decisions/<name>.md`.

1. Read `templates/decision.md`.
2. Extract context, options considered, rationale, date.
3. Decisions are append-mostly. Almost never edit an existing one. Add new
   decisions that supersede old ones with a reference.

#### `/lore capture convention <category>`

Extract a convention from the conversation and add it to
`lore/conventions/<category>.md`. Use this when a convention emerges from
discussion rather than from reading code (the complement to `/lore extract`).

1. Read `extraction-guides/<category>.md` to understand what the category covers.
2. Extract the convention from the conversation in the same shape that
   extraction would produce.
3. Merge into the existing file if one exists, or create a new one.

#### `/lore capture feature <domain>/<name>`

Extract a feature concept and write it into
`lore/domain/<domain>/features/<name>.md`. See `templates/feature.md` for what
shape the file takes — features come in two flavors (work-in-progress scoping
docs and permanent concept docs) and the template covers both.

---

## Folder Reference

### `domain/`

The heart of the project. Each domain concept gets its own folder. The
conventional anchors are `entities.md` (what exists) and `rules.md` (what must
always be true), plus a `features/` subfolder for concepts that need their own
narrative space. Domain folders are permissive — additional files are welcome
when they make the knowledge clearer.

Domain knowledge is described in business language, not implementation
language. "A Payment represents a transfer of funds from a tenant to a property
owner" — not "payments table with foreign key to users."

### `decisions/`

The project's institutional memory. An append-mostly log of technical and
business choices — why things are the way they are. Without this, the LLM has
no memory of paths already explored and rejected, and will keep suggesting
options the team has already evaluated and turned down.

Each decision is short (rarely more than 30 lines) and follows the format in
`templates/decision.md`.

### `conventions/`

How this team builds. Project-owned and built up over time through `extract`
and `capture`. The framework provides extraction guides for six default
categories — patterns, naming, testing, error handling, frontend, API — but
these are starting points, not a fixed taxonomy. Real conventions folders also
contain files documenting specific machinery the team built (a homegrown
concern, a component catalog, a custom DSL). All of it lives here.

### `workflows/`

The connective tissue between domain concepts. What actually happens when a
user does something? This is exactly where LLMs fail most — generating locally
correct but globally broken code because the orchestration knowledge is spread
across controllers, services, callbacks, and jobs. Workflows give that
knowledge a home.

Workflows can embed reference material that's tightly coupled to the flow:
state machines, constants, sub-flows. They start sparse and grow as you build.
The habit: update the workflow before you commit the code.

---

## Templates

Output format guidance lives in `templates/`. Read the relevant template before
writing any knowledge file. Templates are instructions to the LLM, not blanks
for the user to fill in. They provide *suggested* shapes — if the knowledge
needs a different shape, follow the knowledge.

| Template | Used by |
|---|---|
| `templates/project.md` | `/lore init` |
| `templates/domain-entities.md` | `/lore capture domain` |
| `templates/domain-rules.md` | `/lore capture domain` |
| `templates/workflow.md` | `/lore capture workflow` |
| `templates/decision.md` | `/lore capture decision` |
| `templates/feature.md` | `/lore capture feature` and concept docs in domain folders |

---

## Extraction Guides

Instructions for what to look for in a codebase (and what to ask in a
conversation) when populating each convention category. Read the relevant
guide before running `/lore extract <category>` or `/lore capture convention
<category>`.

| Guide | Question it answers |
|---|---|
| `extraction-guides/patterns.md` | Where does business logic live and how is it organized? |
| `extraction-guides/naming.md` | What do we call things, beyond what the framework dictates? |
| `extraction-guides/testing.md` | How do we verify code works? Framework, philosophy, what gets tested. |
| `extraction-guides/error-handling.md` | How do errors flow through the system? Exceptions, results, logging. |
| `extraction-guides/frontend.md` | How is the UI built? Components, styling, interactivity. |
| `extraction-guides/api.md` | How does the system talk to the outside world? (If it has an API.) |

These six are defaults, not a fixed taxonomy. A project may not need all of
them (a backend-only service has no `frontend.md`), and a project may need
others (auth, data, observability) that emerge from the team's specific work.
The framework is opinionated about *which questions are usually worth asking*,
not about which answers are correct.
