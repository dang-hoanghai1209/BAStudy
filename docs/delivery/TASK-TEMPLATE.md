# Task Template

Use this for a single task within a vertical slice. A slice
(`VERTICAL-SLICE-SPEC-TEMPLATE.md`) typically breaks down into several
of these.

---

**Task ID:** VS-0xx-T## (e.g. VS-003-T01)
**Slice:** <link to the slice spec>
**Title:** <short, action-oriented>

## Business context

What user story or business rule does this task serve? Link to
`docs/ba/08-BUSINESS-RULES.md` and/or the relevant user story.

## Scope of this task

What layer does this touch — BA doc, data model, API, backend logic,
frontend, tests? A task should usually stay within one or two adjacent
layers of the vertical-slice pipeline, not the whole thing at once.

## Acceptance criteria

- [ ] <criterion 1>
- [ ] <criterion 2>

## Definition of done for this task

- [ ] Code/doc changes made
- [ ] Relevant tests added/updated (per [`TEST-STRATEGY.md`](TEST-STRATEGY.md))
- [ ] No unverified assumption this task depended on was left unresolved —
      if one was discovered, it is recorded as an open decision in
      [`../ba/08-BUSINESS-RULES.md`](../ba/08-BUSINESS-RULES.md) §2, not
      left implicit
- [ ] Linked BA/architecture docs updated if this task changed a rule,
      entity, or contract
- [ ] If the task touched the backend contract, the client was regenerated
      (never hand-edited)

## Notes / open questions

<anything discovered during the task that needs a decision>
