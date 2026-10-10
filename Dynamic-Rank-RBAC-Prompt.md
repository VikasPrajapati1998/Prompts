# Prompt — Dynamic Rank-Based Access Control (RBAC) for any existing application

Give this file to Claude Code (or any coding agent) inside the target repository. It is technology-neutral: it states the model, the rules, the invariants and the acceptance tests, and leaves the stack-specific wiring to you. Read the target codebase first. Follow that project's own conventions and its CLAUDE.md. Do not run tests, builds or servers unless the owner asks; hand back for them to validate.

---

## 1. Goal

Add **dynamic, rank-based role management** to an application that already has users, roles and permissions (or add a minimal version of them). Administrators create and edit roles at runtime. Every role has a **rank**. A manager can see and manage only roles and users **at or below their own rank**, and within those only the permissions and data they hold themselves.

## 2. Core model

### 2.1 Rank

- Each role has one integer `rank`. **A smaller number is a more powerful role.**
- Reserved: the single top role (Superadmin/Owner) has rank **0**, is unrestricted, immutable and visible only to itself. **No other role may have rank 0. Ranks are never negative.**
- Typical seeds: Superadmin 0, Admin 1, User 2; custom roles default to 1 or the creator's own rank, whichever is larger (adapt to the app).
- Rank is stored data, editable at runtime. It is never inferred from role names.
- Equal rank counts as "at or below": peers see each other.

### 2.2 Visibility (rank gate)

An actor sees a role, and the users holding it, only if `role.rank >= actor.rank` and the role is not the top role (unless the actor is the top role). A role the actor may not see behaves as **nonexistent**: lists omit it, dropdowns omit it, direct access returns **404** (not 403), so its existence is not revealed.

### 2.3 Content is a left join

Of a role the actor can see, the actor sees only the part they **also hold**:

- **Permissions:** `shown = role.permissions ∩ actor.permissions`.
- **Scoped data access** (if the app has it: database → table → column → row-value filters): `shown = role.grant ∩ actor.grant`.

On save, the actor submits only their slice. The system computes:

- `new permissions = submitted ∪ (existing − actor.permissions)`.
- `new data grant = submitted ∪ (existing outside actor.grant)`.

So whatever lies outside the actor's access is **preserved untouched**, never wiped or revealed. Invariant to test: *view a role, save it unchanged, get the identical role back.*

### 2.4 Grant ceiling

Pickers, catalogs and option lists show only what the actor holds. A create/edit request naming any permission or data item the actor does not hold is **rejected (403) and logged** (actor, target, offending items). Unknown items count as "not held" so probing cannot distinguish them.

### 2.5 Rank edits

- Anyone may set a rank only **>= their own rank** (and >= 1).
- A role's rank cannot be changed on the actor's **own role** or on the top role.
- Editing rank requires its own permission (e.g. `roles.rank.update`), separate from generic role editing. When introducing it, grant it to every role that could already edit roles, so nobody loses ability on upgrade.
- Creating a role: default rank per 2.1; the creator cannot give it a rank more powerful than their own.

### 2.6 Effects of changes

- Changing a role's **rank or permissions** invalidates the sessions/tokens of every user holding it (force re-login), so the change applies immediately.
- Deleting a role reassigns its users to the default role in the same transaction.
- Assigning a role to a user requires: the new role is within the actor's rank, **and** the target user's current role is within the actor's rank, the target is not the actor, and not the top role. The same target-rank check applies to block and delete of users. User lists show only users the actor outranks or equals.

## 3. Data model (adapt names)

```
role(id, name unique, description, is_system bool, display_name, rank int NOT NULL)
permission(id, code unique, label, description, group)
role_permission(role_id, permission_id)
user_role(user_id PK, role_id)            -- one role per user (adapt if multi-role)
-- optional scoped data access, cascading default "no narrowing rows = everything at that level":
role_db_access / role_table_access / role_column_access / role_value_access
```

Migration must be **idempotent and self-applying**: add `rank` nullable, backfill only NULL rows (system roles from seeds, custom roles to a default), then set default and NOT NULL. Never reset an edited rank on restart. Seeding built-in roles must not overwrite rank on conflict.

If the app supports multiple roles per user, define the actor's scope as the union of their roles' permissions/grants and the actor's rank as the **minimum** (most powerful) of their roles' ranks.

## 4. Architecture rules

1. **One shared helper module** owns all of this logic (e.g. `roles/scope.py`). Services and routes call it; nothing re-derives it. Suggested API:
   - `AccessScope(rank, permissions, grant)`; `actor_scope(actor)` (top role short-circuits to unrestricted with no DB reads).
   - `rank_visible(scope, role_name, role_rank)`, `filter_visible_roles`, `require_visible_role` (404), `assert_rank_allowed`.
   - `visible_permissions`, `merge_visible_permissions`, `assert_permissions_within`.
   - If scoped data exists: `grant_violations`, `intersect_grant`, `merge_hidden_grant`, `grant_to_tree`, `clamp_to_scope` (a picker's "all" means "all the actor can see").
2. **Enforce on the server, always.** The UI hides things for usability only; every rule above is re-checked in the API.
3. **Order of checks per endpoint:** authenticate → route permission → resolve actor scope → target visibility (404) → rank/ownership guards → payload ceiling (403) → validation → write → invalidate sessions → audit.
4. **Atomicity:** all checks run before the first write; a rejected request leaves no partial role behind.
5. **Errors:** use the project's exception hierarchy with chaining (`403` ceiling/rank, `404` hidden, `409` duplicate name, `4xx` validation). Never swallow exceptions; log rejections with actor, target and reason through the central logger. No `print`.
6. **Audit** role create/update/delete, rank changes, and user role changes (who, what, before/after).
7. **No role-name logic for authority.** Only rank, permissions and grants decide.
8. Keep dependencies minimal; reuse existing helpers; remove superseded code (old per-role "can assign" guards) rather than leaving both.

## 5. Frontend (if the app has one)

- Roles list: show a Rank column, only visible roles, each with only the permissions the actor holds.
- Permission and data pickers are built **only from what the API returns**; rows the actor cannot grant are not rendered.
- Role create/edit form: a whole-number rank input with `min = max(actor rank, 1)`; blank means default; reject negative, non-integer and below-minimum values client-side. Disable it for the actor's own role, the top role, and when the actor lacks the rank permission.
- User screens: role dropdowns list only assignable roles (no greyed-out higher roles); users the actor cannot manage are not listed or are locked.
- Add the rank permission as one picker row (one code per row).

## 6. Tests to write

Pure-logic unit tests (no DB) plus service tests with the repository mocked:

- Rank visibility: equal and larger visible, smaller hidden, top role only to itself.
- Rank assignment: below own rank rejected, own rank and larger allowed, top role may set any rank >= 1; rank 0 and negatives rejected at the schema.
- Permission left join with a worked example: role holds p2–p6, actor holds p1,p2,p3,p6,p7 → actor sees p2,p3,p6; saving p2,p3 leaves the role with p2,p3,p4,p5.
- Ceiling: any permission/data item beyond the actor → 403 and a log entry; hidden role → 404.
- Data grant intersection and merge, including column and value-level hiding, and the **round-trip invariant**.
- Assignment guards: cannot assign a higher role, cannot touch a higher user, cannot act on self or the top role; bulk assignment is all-or-nothing.
- Rank/permission change signs out the role's users; an unchanged rank does not.
- Migration idempotency: second run changes nothing; edited ranks survive restart.

## 7. Acceptance checklist (for the owner to run)

1. Schema migrates cleanly; seed ranks are correct; a restart changes no edited rank.
2. As a mid-level admin: the top role and every higher role/user are absent from lists, dropdowns and pickers, and direct access returns 404.
3. As a restricted admin: pickers show only held permissions/data; a crafted over-grant request returns 403 and is logged.
4. Editing a lower role keeps its permissions/data the admin cannot see.
5. A new role cannot be rank 0 or negative; rank cannot be set more powerful than the actor's own; own-role and top-role ranks are locked; without the rank permission the field is read-only and the API refuses.
6. Changing a role's rank or permissions forces its users to sign in again.
7. The admin can assign any role at or below their rank to any user at or below their rank, and nothing above.

## 8. Rollout notes and known gaps to disclose

- Existing custom roles receive the default rank; tell the operator to review ranks once.
- Duplicate-name errors can reveal that a hidden role name exists; decide whether to mask them.
- Any other action that targets a user (activity views, token limits, etc.) needs the same target-rank check; list the ones you did not cover.
- Rank is a total order: if the business needs incomparable branches (departments), rank alone is not enough.

## 9. Deliverables

Implement in small, reviewable steps: schema + seeds, shared helper + unit tests, services/routes, user-side guards, frontend, docs. Update the project's architecture/setup docs if files or env change. Finish with: what changed, files touched, what is verified vs unverified, and the acceptance checklist above.
