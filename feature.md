# Template: feature

This template defines the *suggested* output format for files in
`lore/domain/<domain>/features/`.

## What features are for

A feature file is for a concept within a domain that needs its own narrative
space — when bullet points in `entities.md` or `rules.md` would lose what
matters.

The test is **"does this need its own narrative space?"** — not "is this
temporary?" Features come in two legitimate flavors, and the same folder
holds both:

**Permanent concept docs.** A piece of the business that's substantial
enough to warrant a dedicated explanation. Worked examples with real numbers,
edge case walkthroughs, an argument for why a distinction matters. These
don't dissolve over time — the narrative *is* the knowledge. Example:
"Partial postings explained, with worked examples for underpayment and
overpayment scenarios."

**Work-in-progress scoping docs.** A scoped piece of work being planned or
built — requirements, edge cases, open questions. These often dissolve over
time as their content gets absorbed into entities, rules, or workflows. By
the time the work is done, what's left is either reference material or
nothing at all. Example: "PDF generation for owner statements: scope,
requirements, open questions."

The framework doesn't try to distinguish them at the file level. Both live
in `features/`. The shape each takes depends on what kind of knowledge it
holds.

## Suggested shapes

### For permanent concept docs

```markdown
# {Concept Name}

## Problem statement / What this is

{Plain-language explanation of what this concept is and why it exists.
Sometimes the "why does this exist" framing is the whole point — the
concept only makes sense once you understand the problem it solves.}

## Scenarios / Worked examples

### {Scenario name}

{Concrete example with real numbers or values. Walk through what happens
step by step. The examples are doing the teaching here — they're not
illustrations of an abstract rule, they're the primary content.}

### {Another scenario}

{Repeat as needed.}

## Edge cases / Interactions

{When this concept interacts with other concepts in surprising ways, or
when there are subtleties that don't fit the main scenarios, document them
here.}

## Related concepts

- `{path/to/related-concept.md}` — {how they relate}
```

### For work-in-progress scoping docs

```markdown
# {Feature Name}

## Scope

{What this feature does and doesn't do. Be explicit about what's
deliberately excluded.}

## Requirements

- {Concrete, testable requirement.}
- {Another.}

## Edge cases

- {Edge case and how it should be handled.}

## Open questions

- {Anything unresolved. Remove items as they get answered — move the
  answers into requirements, edge cases, or back into the domain's
  entities/rules.}

## Related domains

- `{domain/}` — {how this feature interacts with that domain}
```

## Guidelines

- Feature files live inside domain folders: `lore/domain/<domain>/features/<n>.md`.
- The two shapes above are starting points. If your concept needs a
  different shape — a state machine, a glossary, a decision walkthrough —
  follow the knowledge.
- **Worked examples are first-class.** Across both flavors, the most
  valuable feature docs are the ones that show concrete examples instead
  of stating abstract rules. If you can use real numbers, real names, real
  data shapes, do.
- **Embedded code is welcome.** Snippets, configuration, sample outputs —
  whatever makes the concept concrete.
- For scoping docs, the lifecycle handles itself: as the work progresses,
  open questions get answered, requirements get refined, and content
  migrates into more permanent homes (entities, rules, workflows). What's
  left at the end is either reference material or nothing.
- For concept docs, there's no lifecycle to manage. They stay until the
  underlying business reality changes.
- A feature doc that seems to be growing its own sub-concepts is a signal
  that it might want to become its own domain. Don't force it.
