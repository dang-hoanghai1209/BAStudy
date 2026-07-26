# ADR-0007 — Adopt Vitest + Testing Library for frontend component tests

**Status:** Accepted (decision) / Not yet implemented (deliberately deferred to its own commit)
**Date:** 2026-07-26
**Deciders:** Project owner (human), recorded at VS-000 approval

## Context

The project's own test strategy requires, for every slice, a "frontend test
for new UI states — at minimum loading, empty, and error states, not just
the happy path". `CLAUDE.md` §7 repeats the requirement, and
`DEFINITION-OF-READY-DONE.md` makes it a condition of done.

The VS-000 audit found that **the repository has no frontend component-test
runner at all**:

- `frontend/package.json` contains no Vitest, no Jest, no
  `@testing-library/*` package
- `bun run test` resolves to `bunx playwright test`
- the only frontend tests are 6 Playwright end-to-end specs in
  `frontend/tests/`

So the requirement was unsatisfiable as written. Every slice from VS-001
onward would have failed its own Definition of Done on the first day. The
two honest options were to add a runner or to amend the strategy.

## Decision

**Add Vitest with React Testing Library.**

Specifically:

- **Vitest** as the runner — it reuses the existing Vite config, resolves
  the `@/` alias and the SWC React plugin without a second build pipeline,
  and needs no separate transform configuration. Jest would require
  duplicating build config that Vite already owns.
- **@testing-library/react** + **@testing-library/jest-dom** +
  **@testing-library/user-event** for component rendering and assertions.
- **jsdom** as the environment.
- Test files live beside the component as `*.test.tsx`, keeping
  `frontend/tests/` unambiguously Playwright's.
- Scripts: `test:unit` (Vitest) added; **`test` continues to mean
  Playwright** so no CI workflow or documented command silently changes
  meaning.
- A new CI job runs `test:unit` on pull requests.

### This is a dependency change, and it gets its own commit

Adding these packages modifies `frontend/package.json` and `bun.lock`.
That is explicitly outside VS-000's documentation-only scope, and it is
also the kind of change `guard-dependencies.yml` polices.

Therefore:

- **VS-000 does not add the dependency.** This ADR records the decision
  only.
- The dependency is added as its own small, reviewable commit —
  **VS-000a** — before VS-001 begins implementation, containing: the
  package additions, the Vitest config, one example component test proving
  the harness works, the CI job, and the `TEST-STRATEGY.md` update.
- Until VS-000a lands, no slice may claim the "frontend test" line of the
  Definition of Done.

## Alternatives considered

- **Amend the test strategy so Playwright satisfies the UI-state
  requirement** — rejected. E2E can reach a loading or error state only by
  manipulating the backend or intercepting network calls; it is slow,
  flaky for this purpose, and cannot cover a component's empty state in
  isolation. It would also weaken a genuine quality claim to avoid a small
  dependency addition.
- **Jest + jsdom** — rejected. It duplicates transform and alias config
  that Vite already owns, for no benefit in a Vite project.
- **No component tests at all** — rejected. The states most likely to be
  wrong in a data-heavy product are exactly loading, empty and error, and
  they are the ones a happy-path E2E never exercises.
- **Bun's built-in test runner** — rejected. It lacks the React
  ecosystem integration (jsdom environment, Testing Library support) that
  makes component testing straightforward, and would need more custom
  setup than Vitest.

## Consequences

- Positive: the Definition of Done becomes satisfiable as written.
- Positive: loading/empty/error states get cheap, fast coverage, leaving
  Playwright for genuine end-to-end journeys.
- Trade-off: four to five new devDependencies and a CI job.
- Trade-off: two test commands to remember (`test:unit` vs `test`).
  Documented in `TEST-STRATEGY.md` and `CURRENT-STATE.md`.
- Obligation: `docs/delivery/TEST-STRATEGY.md` currently states the gap
  honestly; it must be updated in the VS-000a commit, not before.
- Obligation: VS-000a must land before VS-001 implementation starts.

## Related

[`../../delivery/TEST-STRATEGY.md`](../../delivery/TEST-STRATEGY.md),
[`../../delivery/DEFINITION-OF-READY-DONE.md`](../../delivery/DEFINITION-OF-READY-DONE.md),
[`../CURRENT-STATE.md`](../CURRENT-STATE.md) §9,
[`../../delivery/RISK-REGISTER.md`](../../delivery/RISK-REGISTER.md) B-3.
