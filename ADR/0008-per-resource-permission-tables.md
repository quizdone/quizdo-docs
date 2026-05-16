# ADR 0008: Per-resource permission tables

## Status

Accepted

## Context

The platform shares educational content (categories, questions, tags, tag groups, and similar resources) between users and teams. We need a durable, query-efficient way to represent **who may do what** on each resource without overloading the main entity rows or using models that break down as the number of users, collaborators, and permission combinations grows.

Alternatives considered:

- A single global polymorphic permission table for all resources in every microservice (rejected: weaker FK constraints, harder RLS, mixed lifecycles).

## Decision

### Storage

Store access control as **Unix-style bitmasks** in **dedicated relational tables per resource type**. Each row is keyed by `(user_id, resource_id)` where `user_id` is the Identity **`userSerialId`** (`id_serial`, `int32`).

| Resource   | Permission table            | Go type                         | Proto enum                 |
|------------|-----------------------------|---------------------------------|----------------------------|
| Category   | `category_permissions`      | `category.Permission` (`int8`)  | `CategoryPermissionBit`    |
| Question   | `questions_permissions`     | `question.Permission` (`int8`)  | `QuestionPermissionBit`    |
| Tag        | `tags_permissions`          | `tag.Permission` (`int8`)       | `TagPermissionBit`         |
| Tag group  | `tag_groups_permissions`    | `taggroup.Permission` (`int8`)  | `TagGroupPermissionBit`    |

`perm_bits` is persisted as `smallint`; API responses expose the effective mask as **`uint32 permission`** on `Category`, `CategoryMinified`, `Question`, `QuestionMinified`, `Tag`, `TagMinified`, `TagGroup`, and `TagGroupMinified` (declared as the **last field** in each proto message; field numbers unchanged).

### Permission bits (content service)

Bits are powers of two. **Test** a capability with `(mask & bit) == bit` (all required bits must be set). **Combine** grants with bitwise OR.

| Bit    | Category | Question | Tag / tag group |
|--------|:--------:|:--------:|:---------------:|
| VIEW   | 1        | 1        | 1               |
| EXTEND | 2        | —        | —               |
| EDIT   | 4        | 2        | 2               |
| MANAGE | 8        | 4        | 4               |

- **EXTEND** (categories only): create child categories under a node the user does not own (finder “add child”).
- **MANAGE**: destructive actions (e.g. `DELETE` routes).

Tag **ACL rows are on the tag group**, not per-tag: granting edit on a group governs tags in that group (`POST /tag-group/:groupId/tags` checks group permission).

### Two enforcement layers

1. **Mutations (HTTP middleware and `UserCan`)** — Before create/update/delete handlers run, `middleware.RequirePermission` / `RequirePermissionURIParam` calls `repo.UserCan(ctx, resourceID, userID, perm)`. Allowed when the user is the **author** (`author_id == userSerialId`) **or** `(perm_bits & required) == required`. **`public` does not grant edit/manage** here; it only affects read/list masks below.
2. **Reads and listings (`list_permission`)** — List/detail queries apply GORM scope `WithListPermissionScope` / `WithListPermissionScopeFromContext` so each row gets a computed `list_permission` column in the same round-trip:

   ```text
   list_permission =
     (author_id = viewer ? FULL_MASK : 0)
     | COALESCE(perm_bits, 0)
     | (public ? VIEW : 0)
   ```

   `FULL_MASK` is all bits for that resource type (e.g. category: VIEW|EXTEND|EDIT|MANAGE). Mapped to proto `permission` on the way out. If there is no profile in context (`viewerSerial == 0`), the scope is a no-op (no join).

Services may apply additional **visibility** rules (e.g. categories: hide `unconfirmed` rows from non-authors in listings) independent of the permission table.

### Repository contract

Each resource repository embeds `iface.WithPermission[P]` (`backend/shared/lib/iface/with_permission.go`):

```go
type WithPermission[P ResourcePermission] interface {
	UserCan(ctx context.Context, resourceID int32, userID int32, perm P) (bool, error)
	UpdatePermissions(ctx context.Context, resourceID int32, userID int32, perm P) error
}
```

`UserCan` results are cached per repository instance (otter); cache is cleared when `UpdatePermissions` succeeds.

### HTTP middleware (Content service)

`backend/shared/middleware/permission.go` — requires JWT profile (`GetProfileFromContext`). Resource id from path param (default `id`; use `RequirePermissionURIParam` for names like `groupId`). Options: `AllowEmptyURIParam: true` for routes without an id (e.g. `POST /cat/` create).

Example (Content router):

```go
cat.Post("/", middleware.RequirePermission(categoryRepo, category.PermEdit, middleware.RequirePermissionOptions{AllowEmptyURIParam: true}), handler.CreateOrUpdateCategory)
cat.Delete("/:id", middleware.RequirePermission(categoryRepo, category.PermManage, middleware.RequirePermissionOptions{}), handler.DeleteCategory)
tagGroups.Post("/:groupId/tags", middleware.RequirePermissionURIParam(tagGroupRepo, taggroup.PermEdit, "groupId", middleware.RequirePermissionOptions{}), tagHandler.CreateOrUpdateTag)
```

GET routes rely on service/repository visibility and `list_permission` (and optional `UserCan` for non-public single-resource reads, e.g. questions).

### Frontend

Use generated `*PermissionBit` enum values with `hasPermission(mask, bit)` (`frontend/src/lib/permission/hasPermission.ts`): `(permissionMask & wantPermission) === wantPermission`. Drive UI affordances (edit, delete, add child) from `permission` on minified/full DTOs; do not re-derive author/public rules in the client.

### Identity service

Identity provides **authentication context** (`GetUserProfile`, serial ids). It does **not** evaluate per-resource permission tables; that stays in the Content service (and future domain services with their own `*_permissions` tables).

## Consequences

### Benefits

- **Predictable lookups**: Composite keys and indexes on permission tables match “who can access this resource?” queries.
- **Sparse grants**: Row-per-grant storage scales with collaborators, not parent row width.
- **Clear separation**: `public` + author implicit full access for **read UI**; explicit rows for collaborators; middleware for **writes**.
- **Consistent pattern**: Same repository interface, middleware, proto `permission` field, and TS helper across resource types.
- **Request blocked before handler work** on mutating routes.

### Tradeoffs

- **More tables and joins** on list/detail queries (`LEFT JOIN` + computed column).
- **Per-resource migrations and enums** (mitigated by shared iface and middleware).
- **`UserCan` vs `list_permission`**: `public` grants VIEW only in listings, not in `UserCan` — callers must use the right layer for read vs write.
- **Tag group vs tag**: Group-level ACL must be documented for API consumers.

## References

- Interface: `backend/shared/lib/iface/with_permission.go`
- Middleware: `backend/shared/middleware/permission.go`, `profile.go` (`ViewerSerialFromContext`)
- Example entities: `CategoryPermission`, `QuestionPermission`, `TagPermission`, `TagGroupPermission` under `backend/services/content/entities/`
- List scope example: `category.Category.WithListPermissionScope` in `category.go`
- Proto: `proto/models/v1/category.proto`, `question.proto`, `tag.proto`, `tag_group.proto`
- Auth chain: [ADR 0006](0006-jwt-verification-and-user-profile-middleware.md), [ADR 0005](0005-rls-microservice-architecture.md)
