# Prompt — API Contract Change

Use when a slice adds or changes endpoints.

---

```
Design and implement the API change for <VS-0xx — Name>.

Read first:
- docs/architecture/API-CONTRACT-POLICY.md
- docs/architecture/GENERATED-FILES.md
- docs/security/AUTHORIZATION-POLICY.md
- docs/architecture/CURRENT-STATE.md §5 (existing API surface)

Capabilities needed: <list what the frontend must be able to do>

Constraints:
- All routes mount under /api/v1
- EVERY router must declare tags=[...]. The generated client's service
  class name comes from the first tag (operation IDs are "{tag}-{name}"),
  so a missing tag breaks client generation and a changed tag is a
  BREAKING change
- Every business endpoint goes through the organization-scoping dependency.
  A business endpoint must never contain "if current_user.is_superuser"
- Cross-organization access returns 404, not 403
- Money is serialized as a JSON string, not a number
- Prefer additive changes. A breaking change to a shipped endpoint needs a
  note in the slice's integration report explaining why it is necessary

Steps:
1. Propose the endpoints: method, path, tag, request schema, response
   schema, status codes, error cases, and the required permission for each.
   WAIT for my approval.
2. Implement the backend to match exactly.
3. Add the permission rows to docs/security/AUTHORIZATION-POLICY.md §4.
4. Regenerate the client:
     bash scripts/generate-client.sh
   Never hand-edit frontend/src/client/**. If the client is wrong, the
   backend schema is wrong.
5. Show me the diff in frontend/src/client/ and explain any change that is
   not simply additive — that diff is how a breaking change becomes visible.
6. Write integration tests including: each role allowed, each role denied,
   cross-organization returning 404, and a superuser WITHOUT a membership
   also getting 404.

Note: openapi.json is git-ignored — it is a build artifact, not a committed
contract file. The reviewable artifact is the frontend/src/client/ diff.
```

---

## Notes

- The tag rule is the most common way client generation breaks. It is worth
  keeping in the prompt even though it is documented.
- Step 5 turns "did we break the contract?" into something visible in
  review rather than something discovered by the frontend later.
