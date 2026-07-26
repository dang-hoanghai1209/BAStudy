# Personas

**Status:** Draft — hypothesised from the product vision. **Not validated by
interviews.** Per [`../ba/00-PROBLEM-STATEMENT.md`](../ba/00-PROBLEM-STATEMENT.md),
at least three sellers or e-commerce operations staff should be interviewed
before any statement here is treated as confirmed. Until then, every "pain"
below is an *assumed* pain.

These personas expand [`../ba/02-STAKEHOLDER-MAP.md`](../ba/02-STAKEHOLDER-MAP.md)
into the detail needed to design screens. The role names map directly onto
the system roles in [`../security/AUTHORIZATION-POLICY.md`](../security/AUTHORIZATION-POLICY.md).

---

## P1 — Linh, Shop Owner

**System role:** Owner

**Context.** Runs a small multi-channel retail business: three shops across
two or three marketplaces, a few hundred SKUs, a few thousand orders a
month. Not technical. Lives in the marketplace seller centres and a
spreadsheet.

**What they are trying to decide.** Which products to keep selling, which
channel is actually worth the effort, whether last month was profitable,
and whether a discount campaign made money or quietly lost it.

**Assumed pains.**
- Revenue is easy to see; profit is not. High revenue months have felt bad
  in the bank account and they cannot explain why.
- Marketplace fee lines are numerous and unexplained.
- Money received rarely matches what was expected, and finding out why
  takes a day of spreadsheet work — so usually it is not done.

**What "good" looks like.** Opens the dashboard, sees profit per shop for
last month, trusts the number enough to act on it, and can drill down when
it looks wrong instead of losing faith in the whole figure.

**Design implications.**
- The dashboard must be readable in under a minute, without training.
- Every headline number must be one click from its explanation.
- Never show a confident number built on incomplete data — show the gap.
- Empty and partial states matter more than for a technical user: an
  unexplained zero reads as "this product is broken".

**Anti-goals.** Does not want to configure a fee engine, learn accounting
vocabulary, or read documentation.

---

## P2 — Minh, Operations Analyst

**System role:** Analyst

**Context.** Does the daily work: downloads exports, uploads them, chases
mismatches, maintains cost data. Comfortable with spreadsheets, not a
developer. This is the persona who touches the system most.

**What they are trying to do.** Get this week's data in cleanly, fix
whatever the import complains about, keep costs current, and answer
"why is this SKU showing a loss?" when the Owner asks.

**Assumed pains.**
- Marketplace exports change format without warning.
- Reconciling a statement line by line is tedious and error-prone.
- Nobody knows which cost figure was used for an order three months ago.

**What "good" looks like.** Import tells them exactly which rows failed and
why, in their language; they fix the cause once and reprocess; the
exception queue empties; nothing needs re-doing next week.

**Design implications.**
- Error messages must name the row, the column, and the business problem —
  not a stack trace or a database constraint name.
- Bulk actions matter: 200 unmapped SKUs cannot be a 200-click task.
- Re-doing work is the primary frustration. Column-mapping profiles,
  reprocessing and idempotent import exist for this persona.
- Volume is real. Tables need filtering, sorting and pagination from day
  one, not as a later polish item.

**Anti-goals.** Does not want to be blocked waiting for the Owner to approve
routine work. Does not want the system to auto-fix things silently — they
are accountable for the numbers.

---

## P3 — Thu, Accountant / Bookkeeper

**System role:** Viewer (possibly a distinct role — **Open**, see
[`../security/AUTHORIZATION-POLICY.md`](../security/AUTHORIZATION-POLICY.md) §2)

**Context.** External or part-time. Consumes SellerMargin output as an input
to their own process. Interacts rarely — monthly, around period close.

**What they are trying to do.** Get a reliable statement of what was earned
and what was actually received, in a form their existing process accepts.

**Assumed pains.**
- Receives inconsistent, hand-built spreadsheets.
- Cannot tell whether a figure has changed since they last saw it.

**What "good" looks like.** Exports a period report, and the numbers do not
move afterwards — or if they do, there is a visible reason.

**Design implications.**
- Exports must match on-screen figures **exactly**, including rounding.
- Period locking exists largely for this persona.
- The export format is a requirement, not a preference — ask before
  designing.

**Anti-goals.** Does not want to log in to explore. Wants a file.

---

## P4 — Portfolio reviewer / interviewer

**System role:** none — reads the repository and a demo

**Context.** A hiring manager, senior BA, or engineer evaluating this
project as evidence of capability. This is the project's stated secondary
goal and it legitimately shapes some decisions.

**What they are looking for.** Whether the business analysis is real or
decorative; whether the numbers are traceable; whether hard cases (refunds,
cost history, reconciliation) were handled or avoided; whether the
documentation matches the code.

**Design implications.**
- Traceability must be *demonstrable in the UI*, not just claimed in a
  document.
- One end-to-end business scenario must be walkable in a demo.
- Documentation that contradicts the repository is worse than no
  documentation — this is why `CURRENT-STATE.md` is kept separate from
  `TARGET-ARCHITECTURE.md`.

**Anti-goals.** Not a user. Never let this persona's needs distort the
product into a demo that a real seller could not use.

---

## Priority when personas conflict

1. **Minh (Analyst)** — highest interaction volume; if the daily work is
   painful, nothing else gets used.
2. **Linh (Owner)** — the decision-maker and the reason the product exists.
3. **Thu (Accountant)** — periodic but unforgiving about accuracy.
4. **Reviewer** — influences what is *documented and demonstrable*, never
   what is *built*.

## Validation plan

Before VS-006, conduct at least three interviews and record findings in
`../ba/00-PROBLEM-STATEMENT.md`, updating these personas. Specifically test:
whether cost data is actually maintained anywhere today, whether
reconciliation is done at all or simply skipped, and which marketplaces
matter (which also unblocks D-07).
