# Template: workflow

This template defines a *suggested* output format for `lore/workflows/<n>.md`
files. Real workflows often need more than a flat list of steps — they pull in
state machines, constants, sub-flows, and small architectural sketches when
those are tightly coupled to understanding the flow. That's allowed and
encouraged. Anchors are not walls.

## Suggested output format

```markdown
# {Workflow Name}

## Purpose

{One or two sentences. What does this workflow accomplish for the user or the
business?}

## Trigger

{What kicks this off? A user action, a scheduled job, an incoming event.}

## Domains involved

{List the domain areas this workflow touches. This is why workflows exist —
they're the connective tissue between domains.}

## Steps

1. **{Step name}** — {What happens. Be specific about which system/service
   handles it. Note async boundaries explicitly.}
2. **{Step name}** — {Next step.}
3. **{Step name}** — {Continue.}

## Failure modes

- **{Failure scenario}** — {What goes wrong and how the system responds.}
- **{Failure scenario}** — {Another failure case.}

## Expected outcome

{What does "success" look like? What state is the system in when this workflow
completes normally?}

## Open questions

- {Anything unresolved. Remove items as they get answered.}
```

## Optional sections (use when relevant)

Real workflows often benefit from these. Add them when the flow can't be
understood without them.

- **Components overview** — A small sketch of the code that participates in
  the flow. Useful when the flow crosses many files and the LLM needs to know
  which pieces wire together.
- **State transitions** — A state machine for the entity the workflow drives.
  Belongs here (not in `domain/entities.md`) when the workflow is what causes
  the transitions and the state machine only makes sense in that context.
- **Constants and thresholds** — Magic numbers that govern the flow's behavior
  (`MAX_RETRIES = 5`, `TIMEOUT_SECONDS = 30`). The LLM needs to know these
  exist when generating any code that touches the flow.
- **Sub-flows** — A nested flow inside the main one. The MFA "failure
  escalation" path is a good example: failures compound through retries and
  resends in a way that's its own mini-flow.
- **Method selection / branching logic** — When the workflow picks between
  multiple paths based on a rule, document the rule explicitly (often as a
  small code snippet or table).

## Diagrams

Diagrams earn their keep in workflows more than anywhere else in the knowledge
tree. State machines, decision trees, and sequence flows are dramatically
clearer with a picture than with prose.

- **Mermaid is preferred** — it's compact, renders in most markdown viewers,
  and doesn't break in narrow terminals.
- ASCII art is acceptable when Mermaid can't express the shape (or when the
  team doesn't render markdown).
- Don't add diagrams just because you can. Add them when prose alone would
  force the reader to mentally reconstruct the picture anyway.

## Guidelines

- Workflows represent meaningful business processes that cross domain
  boundaries. "User clicks save" is not a workflow. "User onboards onto the
  platform" is. A mid-sized app might have 10-20 workflows.
- Workflows start sparse and grow. Day one might just have Purpose, Trigger,
  and Open questions. That's fine. Steps fill in as you implement.
- This is exactly where LLMs fail most — generating locally correct but
  globally broken code because the orchestration knowledge is spread across
  controllers, services, callbacks, and jobs. Workflows prevent that.
- Note async boundaries explicitly. "Step 3 enqueues a background job" changes
  how the LLM generates the code.
- Failure modes are where the real value is. The happy path is usually
  obvious. What happens when the email is malformed? When the gateway times
  out? When the user cancels mid-flow?
- Update the workflow before you commit the code. If you implemented a new
  step or discovered a failure mode, capture it.
- **Embedded code is welcome.** A snippet showing the actual selection logic
  or the constants that govern the flow is often clearer than prose.
