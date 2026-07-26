# API Contract Policy

**Status:** Verified (pipeline and paths) / Decided (conventions).

## 1. Why a contract checkpoint matters with a single agent

Even when one agent writes both sides, deciding the API shape deliberately
— and recording it — before UI code exists prevents backend and frontend
from co-evolving into something nobody designed. The contract step is not
inter-agent ceremony; it is the moment the interface gets thought about.

## 2. The contract in this repository

- The backend is authoritative. FastAPI generates the OpenAPI schema from
  the route signatures and response models.
- **`openapi.json` is git-ignored** (`frontend/.gitignore`). It is a build
  artifact produced between generation steps, not a committed contract
  file. Do not design a review process around diffing it.
- **The reviewable artifact is the `frontend/src/client/` diff.** Those ten
  files are committed, and a breaking API change shows up there — which is
  exactly what makes it catchable in review.

Full pipeline and configuration:
[`GENERATED-FILES.md`](GENERATED-FILES.md).

## 3. Workflow for a contract change

1. Business rules and acceptance criteria for the slice exist
   ([`../../CLAUDE.md`](../../CLAUDE.md) §5).
2. The data model change is drafted in [`DATA-MODEL.md`](DATA-MODEL.md).
3. Endpoints are designed — method, path, tag, request and response
   schemas, status codes, error cases, **and the required permission** —
   and the permission rows are added to
   [`../security/AUTHORIZATION-POLICY.md`](../security/AUTHORIZATION-POLICY.md) §4.
4. The backend is implemented to match.
5. The client is regenerated: `bash scripts/generate-client.sh`.
6. Frontend work begins only against the regenerated client.
7. A mismatch found later — the frontend needs a field the schema lacks —
   goes back to step 3. Never a manual patch in `src/client/`.

The `generate-frontend-sdk` pre-commit hook runs step 5 automatically on
any `backend/**` change, so this cannot be forgotten silently.

## 4. Conventions

| Convention | Rule |
|---|---|
| Prefix | All routes under `/api/v1` (`settings.API_V1_STR`) |
| **Tags** | **Every router must declare `tags=[...]`.** Operation IDs are `{tag}-{name}`; the first tag becomes the generated service class name. Missing tag → generation fails. Changed tag → breaking change |
| Naming | Plural resource paths (`/orders`, `/import-batches`) |
| Schemas | `*Base`, `*Create`, `*Update`, `*Public`, `*sPublic` (data + count), following the template |
| Money | Serialized as a JSON **string**, not a number, to avoid float round-tripping in JavaScript ([ADR-0003](adr/ADR-0003-MONEY-AND-CURRENCY.md)) |
| Timestamps | ISO 8601 with offset, UTC |
| Pagination | `skip` / `limit` with a `count`, matching the existing template endpoints |
| Errors | `404` for a resource outside the caller's organization — never `403`, which confirms existence. `403` only for insufficient role **within** the caller's own organization |
| Auth | Every business endpoint goes through the organization-scoping dependency. **A business endpoint must never contain `if current_user.is_superuser`** ([ADR-0006](adr/ADR-0006-AUTHORIZATION-MODEL.md)) |

## 5. Compatibility

- Prefer additive changes: new optional field, new endpoint.
- A breaking change to a shipped endpoint requires a note in the slice's
  integration report explaining what breaks and why it is necessary.
- Breaking changes include: removing or renaming a field, making an
  optional field required, changing a type, changing a status code,
  changing a route path, **and changing a router tag**.
- Because the frontend and backend ship together, versioned endpoints are
  not needed. If the API ever gets an external consumer, that becomes a new
  ADR.

## 6. Review checklist for an API change

- [ ] Every new router declares a tag
- [ ] The regenerated client diff was read, and any non-additive change is
      explained
- [ ] Permission rows added to the authorization policy matrix
- [ ] Cross-organization access returns 404, with a test
- [ ] Money fields are strings, not numbers
- [ ] Error responses carry actionable messages, not raw database errors
      and no personal-data values
- [ ] The client was regenerated, not hand-edited
