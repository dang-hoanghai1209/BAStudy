# Prompt — Frontend Slice

Use once the backend contract is stable and the client has been
regenerated.

---

```
Build the frontend for <VS-0xx — Name>.

Read first:
- docs/product/UX-PRINCIPLES.md
- docs/product/SCREEN-INVENTORY.md
- docs/product/PERSONAS.md
- docs/architecture/CURRENT-STATE.md §7 (existing frontend stack)

Screens: <list, with routes from SCREEN-INVENTORY.md>

Use what is already there — do not introduce a second library for anything:
- TanStack Router (file-based routes in src/routes/)
- TanStack Query for all server state
- components/ui/* (shadcn/Radix), Common/DataTable, Pending/* skeletons
- react-hook-form + zod for forms
- sonner for toasts
- Tailwind 4; Biome for lint/format
- The generated client from @/client is the ONLY way to reach the API

Every screen must implement all four states:
- Loading — skeleton matching the final layout, no layout shift
- Empty — explain WHY it is empty and give the next action, never a bare
  "No data"
- Error — what failed, whether it is retryable, and a retry control
- Happy — the data

Rules:
- No business rule is reimplemented here. If the UI needs a derived
  number, the API returns it
- The frontend never does money arithmetic. Money arrives as a string;
  format it, do not compute with it
- Actions the current role cannot perform are visible but disabled with a
  reason — not hidden. Server-side enforcement is the real control
- Lists get filtering, sorting, pagination and a visible total count
- Anything that changes reported history states WHAT will change before
  the user confirms
- Do not edit src/routeTree.gen.ts — the Vite plugin regenerates it
- Do not edit src/client/** — regenerate from the backend

Note: there is no component-test runner until VS-000a lands
(ADR-0007). Until then, cover the four states with Playwright where
practical and say plainly in the integration report which states are not
covered — do not claim coverage that does not exist.

Finish by listing the screens built, their states, and anything from
SCREEN-INVENTORY.md you did NOT build and why.
```

---

## Notes

- The "visible but disabled with a reason" rule is a deliberate departure
  from hiding controls; hiding teaches users a feature does not exist.
- The honesty instruction about test coverage matters more than it looks:
  the alternative is a slice that claims a Definition of Done item it did
  not meet.
