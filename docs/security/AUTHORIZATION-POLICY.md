# Authorization Policy

**Status:** Decided (model and principles) / Draft (full permission matrix —
finalised in VS-016 as endpoints are added).

This document is the source of truth for **who may do what**. When it
conflicts with an implementation, the implementation is wrong.

## 1. Two independent axes

Per [`../architecture/adr/ADR-0006-AUTHORIZATION-MODEL.md`](../architecture/adr/ADR-0006-AUTHORIZATION-MODEL.md),
authorization has two axes that never merge:

| Axis | Mechanism | Scope | Grants |
|---|---|---|---|
| **Platform administration** | `user.is_superuser` (inherited from the template) | The installation, outside any organization | User account administration, platform diagnostics |
| **Organization role** | `membership` (user × organization × role) | One organization | All business data access |

Non-negotiable consequences:

- **`is_superuser` grants no access to any organization's business data.**
  A superuser without a membership gets 404 on organization-scoped
  endpoints, exactly like any other stranger. This is tested.
- An organization role grants no platform privilege.
- A business endpoint must never contain `if current_user.is_superuser`.
  If it does, the two axes have been mixed and the endpoint is wrong.
- A person may hold both independently.

## 2. Organization roles

| Role | Intended holder | Summary |
|---|---|---|
| **Owner** | Shop owner / business decision-maker | Everything within the organization, including members, roles, fee rules, reconciliation confirmation and period unlocking |
| **Analyst** | Day-to-day operator | Import, map, enter costs, investigate, resolve exceptions. Cannot change organization structure, roles, or confirm reconciliations |
| **Viewer** | Read-only stakeholder | Read dashboards, reports and drill-downs. No writes |

A user may hold different roles in different organizations. Roles are
per-organization; per-shop role scoping is available on `membership` but is
not used before VS-016.

**Open:** whether "Accountant" becomes a distinct fourth role or is
"Viewer + export" is not decided. Tracked in
[`../ba/02-STAKEHOLDER-MAP.md`](../ba/02-STAKEHOLDER-MAP.md) and resolved in
VS-016 or VS-018, whichever comes first.

## 3. Enforcement rules

1. **Deny by default.** A capability not explicitly granted is denied. New
   endpoints are denied to everyone until added to the matrix below.
2. **Server-side only.** Hiding a button is an affordance. The server
   check is the control. Every restricted action has a test that calls the
   API directly with an insufficient role.
3. **One shared dependency.** Organization scoping and permission checks
   come from a shared dependency in `app/api/deps.py`, not from per-handler
   code. A handler that filters by organization itself is a review failure.
4. **404, not 403, across organizations.** A resource in another
   organization must be indistinguishable from a resource that does not
   exist. Within the caller's own organization, an insufficient role
   correctly returns 403.
5. **Checked on every request.** No caching of permission decisions across
   requests. Removing a role takes effect on the caller's next request.
6. **Attributed.** Every permission-relevant action records the acting
   user; sensitive actions also emit an audit event
   ([`AUDIT-POLICY.md`](AUDIT-POLICY.md)).

## 4. Permission matrix

Legend: ✅ allowed · ❌ denied · — not applicable.
This matrix grows with each slice; the slice that adds a capability adds
its row in the same pull request.

### Organization and structure (VS-001, VS-016)

| Capability | Owner | Analyst | Viewer | Superuser (no membership) |
|---|---|---|---|---|
| View organization | ✅ | ✅ | ✅ | ❌ |
| Update organization settings | ✅ | ❌ | ❌ | ❌ |
| Create / update / deactivate shop | ✅ | ❌ | ❌ | ❌ |
| View shops | ✅ | ✅ | ✅ | ❌ |
| Invite member, assign role | ✅ | ❌ | ❌ | ❌ |
| Remove member | ✅ | ❌ | ❌ | ❌ |
| View members | ✅ | ✅ | ❌ | ❌ |

### Catalog and mapping (VS-002, VS-003)

| Capability | Owner | Analyst | Viewer |
|---|---|---|---|
| View products / SKUs | ✅ | ✅ | ✅ |
| Create / update product, variant, internal SKU | ✅ | ✅ | ❌ |
| Deactivate internal SKU | ✅ | ✅ | ❌ |
| View SKU mappings | ✅ | ✅ | ✅ |
| Confirm a SKU mapping | ✅ | ✅ | ❌ |
| Remap a confirmed mapping | ✅ | ✅ | ❌ |

### Import (VS-004, VS-005)

| Capability | Owner | Analyst | Viewer |
|---|---|---|---|
| Upload a file / start an import | ✅ | ✅ | ❌ |
| View import batches and errors | ✅ | ✅ | ✅ |
| Download the original uploaded file | ✅ | ✅ | ❌ |
| Reprocess a batch | ✅ | ✅ | ❌ |
| Soft-delete a batch | ✅ | ❌ | ❌ |
| Save / edit a column-mapping profile | ✅ | ✅ | ❌ |

### Costs and fees (VS-006, VS-008)

| Capability | Owner | Analyst | Viewer |
|---|---|---|---|
| View cost history | ✅ | ✅ | ✅ |
| Add a cost record | ✅ | ✅ | ❌ |
| Correct (supersede) a past cost record | ✅ | ❌ | ❌ |
| View fee rules | ✅ | ✅ | ✅ |
| **Create / edit a fee rule** | ✅ | ❌ | ❌ |

The fee-rule restriction is the project's working answer to mandatory
question #15, pending the owner's confirmation. It is recorded here as
Draft, not Decided.

### Profit, settlement and reconciliation (VS-009 – VS-012)

| Capability | Owner | Analyst | Viewer |
|---|---|---|---|
| View profit figures and explanations | ✅ | ✅ | ✅ |
| Trigger a recalculation | ✅ | ✅ | ❌ |
| Trigger a recalculation affecting a reported period | ✅ | ❌ | ❌ |
| Import a settlement / statement | ✅ | ✅ | ❌ |
| Run reconciliation | ✅ | ✅ | ❌ |
| Manually match / unmatch a line | ✅ | ✅ | ❌ |
| **Confirm / close a reconciliation** | ✅ | ❌ | ❌ |
| **Lock a period** | ✅ | ❌ | ❌ |
| **Unlock a period** | ✅ | ❌ | ❌ |

### Exceptions, audit, reports (VS-013, VS-017, VS-018)

| Capability | Owner | Analyst | Viewer |
|---|---|---|---|
| View exception queue | ✅ | ✅ | ✅ |
| Assign / comment on an exception | ✅ | ✅ | ❌ |
| Resolve an exception | ✅ | ✅ | ❌ |
| View audit log | ✅ | ❌ | ❌ |
| Export audit log | ✅ | ❌ | ❌ |
| View reports | ✅ | ✅ | ✅ |
| Export a report | ✅ | ✅ | Open — depends on the Accountant-role decision |
| Export raw data containing personal data | ✅ | Open | ❌ |

## 5. Sensitive actions

These require the highest role, emit an audit event with a mandatory
reason, and each has a dedicated negative test:

- editing or superseding a fee rule
- correcting a past cost record
- confirming or reopening a reconciliation
- locking or unlocking a period
- triggering a recalculation that changes a previously reported figure
- soft-deleting an import batch
- changing a member's role or removing a member
- exporting data containing buyer personal data

## 6. Testing obligation

Every slice adding an endpoint must add:

1. a row to the matrix above, in the same pull request;
2. a positive test for each allowed role;
3. a negative test for each denied role, calling the API directly;
4. a cross-organization test asserting **404**;
5. for organization-scoped endpoints, a test that a superuser **without a
   membership** also receives 404.

A slice whose matrix rows have no corresponding tests is not done.

## 7. Explicitly out of scope

SSO / SAML / OIDC, attribute-based access control, field-level permissions,
delegated cross-organization administration, time-bounded support access,
and customer-defined custom roles. Each would be a new ADR.
