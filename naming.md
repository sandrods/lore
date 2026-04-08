# Extraction Guide: Naming

## Question this convention answers

**What do we call things, beyond what the framework dictates?**

The framework gives you some naming for free — Rails decides that models are
singular and tables are plural, that controllers are pluralized and end with
`Controller`, that mailers end with `Mailer`. Those defaults don't need
documenting.

What needs documenting is everything the framework leaves open: how services
get named, how jobs get named, how scopes are phrased, what database columns
are called for domain concepts, how booleans are formed, how money is stored,
what events are called. These are the places teams diverge and the LLM
guesses inconsistently.

This is the highest-confidence category for extraction. Most of it is
observable. Humans only need to confirm or correct edge cases.

---

## Where to look

- **`app/services/`** (or wherever services live) — list all the file names.
  Look for the pattern: `Module::VerbNoun`, `VerbNoun`, `NounVerb`, etc.
- **`app/jobs/`** — same. Are jobs named after the service they wrap?
  After what they do?
- **`app/models/`** — model class names. Are they business-language or
  technical? `OwnerStatement` (business) or `Report` (generic)?
- **`db/schema.rb`** — the goldmine for column conventions. Look for:
  - Boolean column patterns (`is_active`, `active`, `has_x`, `can_x`)
  - Timestamp patterns (`created_at`, `processed_at`, `closed_at`)
  - Money columns (`amount`, `amount_cents`, `price_in_cents`)
  - Foreign key patterns (Rails default `<table>_id` or something custom)
  - Enum columns (`status`, `state`, `type`, etc.)
  - Polymorphic patterns
- **Model files** — scopes. How are they named? After the result (`active`,
  `recent`) or the mechanism (`with_active_status`, `created_within_30_days`)?
- **`config/routes.rb`** — route naming. Standard REST, or custom action
  names?
- **`app/components/`, `app/views/`** — component class naming. Suffix
  conventions (`Component`, `View`, none).
- **Event names** — search for `broadcast`, `publish`, `emit`, `notify`. If
  the project has events, see how they're named (past-tense domain
  occurrences? imperative? something else?).

---

## High-confidence extractable

Almost everything in this category. Specifically:

- **Service naming pattern.** Module structure, verb form, single vs
  multi-word.
- **Job naming pattern.** Suffix (`Job`, `Worker`), relationship to services.
- **Boolean column conventions.** Prefix patterns or no prefix.
- **Timestamp column conventions.** `_at` suffix or other.
- **Money storage.** Cents-as-integer, decimal, money column type.
- **Foreign key naming.** Standard Rails or customized.
- **Scope naming style.** Result-describing vs mechanism-describing. Pick the
  dominant style and note exceptions.
- **Predicate methods.** `?` ending, presence of `is_`/`has_`/`can_` prefixes
  on methods.
- **Component suffix.** What components are called.
- **Event naming style.** If events exist.
- **Routes.** Standard REST + any non-standard patterns.

For each of these, the convention is observable. Write it as a rule plus an
example from the actual codebase.

---

## Medium-confidence (surface for confirmation)

- **Domain language.** Models named after business concepts vs technical
  ones. You can see the names but you can't always tell if they match the
  business's vocabulary. Worth confirming with the team — "your code calls
  these `Statement`, is that what the business calls them too?"
- **Inconsistencies.** "Most boolean columns use no prefix (`active`,
  `closed`), but a few use `is_` (`is_admin`, `is_default`). Is the prefix
  intentional in those cases?"

---

## What needs human input

- **The why behind the choices.** Some naming conventions encode invisible
  decisions. "We store money as cents because we had a rounding bug in 2022"
  is the kind of context that only the team knows.
- **Aspirational naming.** If the team is mid-rename (e.g., migrating from
  `User` to `Member`), the code will show the old name and the new name in
  different places. You can detect the inconsistency but not the direction.

---

## Questions for capture mode

- "How do you store money?"
- "What's the convention for boolean columns and methods?"
- "How do you name scopes?"
- "Are there any words that mean something specific in this codebase that an
  outsider might miss?" (Glossary territory — could go in domain instead.)
- "Are there any naming conventions you're trying to migrate toward?"

---

## Output structure

Write to `lore/conventions/naming.md`. Suggested shape:

```markdown
# Naming

## Services and jobs

{Pattern, with examples.}

## Models and database

{Class naming, column naming, including booleans, timestamps, money,
foreign keys, enums.}

## Methods

{Predicate ending, dangerous methods, boolean variables.}

## Scopes

{Style — result-describing or mechanism-describing — with examples.}

## Routes

{Anything beyond standard REST.}

## Components / views

{If applicable.}

## Events / broadcasts

{If applicable.}

## Inconsistencies / outliers

{Found patterns that don't match the dominant style. Worth confirming.}

## Discussion needed

{The few things extract couldn't determine. Should be short for this category.}
```

Naming is the category where extraction can produce the closest-to-final
draft. The team's review pass should mostly be confirming and adding the
rare aspirational note.
