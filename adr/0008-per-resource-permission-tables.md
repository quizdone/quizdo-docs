# ADR 0008: Per-resource permission tables

## Status

Accepted

## Context

The platform shares educational content (categories, questions, exams, and similar resources) between users and teams. We need a durable, query-efficient way to represent **who may do what** on each resource without overloading the main entity rows or using models that break down as the number of users, collaborators, and permission combinations grows.

Alternative thoughts:
- creating a single global polymorphic permission table for all resources in every microservice

## Decision

We will store access control bitmask permissions in **dedicated relational tables per resource type**.  
> E.g. `category_permissions` with rows keyed by user and category, each row representing a bitmask (unix like) permissions for a single user.

## Custom interface

The repository layer uses an embeddable contract so each resource repository exposes permission checks and updates in a consistent shape.

Implementations must satisfy `iface.WithPermission[P]` (`backend/shared/lib/iface/with_permission.go`). `UserCan` / `UpdatePermissions` are intended for use from handlers, services, or HTTP middleware when deciding whether to allow an action.

```go
// P is each resource’s own permission type (e.g. category.Permission); constrained by ResourcePermission.
type ResourcePermission interface {
	~int8 | ~int16 | ~int32 | ~uint8 | ~uint16 | ~uint32
}

type WithPermission[P ResourcePermission] interface {
	UserCan(ctx context.Context, resourceID int32, userID int32, perm P) (bool, error)
	UpdatePermissions(ctx context.Context, resourceID int32, userID int32, perm P) error
}

//... repository implementation ...
type CategoryRepository interface {
	iface.WithPermission[category.Permission]
	// List returns categories matching the provided scopes.
	List(ctx context.Context, scopes ...func(*gorm.DB) *gorm.DB) ([]category.Category, error)
	// GetByID returns a single category by its ID or nil when not found.
	GetByID(ctx context.Context, id uint32) (*category.Category, error)
	// Save creates or updates the given category (and its localized data) in a transaction.
	Save(ctx context.Context, cat *category.Category) error
	// Delete soft-deletes all categories with the given IDs. Related localized rows are soft-deleted as well.
	Delete(ctx context.Context, ids []uint32) error
}

func NewCategoryRepository(db *gorm.DB) CategoryRepository {
	return &categoryRepository{
		db: db,
	}
}
```

## Middleware (pattern)

HTTP middleware (or equivalent) can run before mutating handlers and call `UserCan` (or service logic built on the repository) so unauthorized requests return **403 Forbidden** without touching domain logic.

The middleware takes (1) a repository implementing `iface.WithPermission[P]` for that resource, and (2) the required permission bitmask `P` (the resource’s own enum, e.g. `category.PermEdit`). The resource id is taken from the route (default path param `id`). Example:

```go
cat.Delete("/:id", middleware.RequirePermission(categoryRepo, category.PermEdit), handler.DeleteCategory)
// non-default path param name:
cat.Delete("/:catId", middleware.RequirePermission(categoryRepo, category.PermEdit, "catId"), handler.DeleteCategory)
```


## Consequences

### Benefits

- **Predictable lookups**: Composite indexes on `(resource_id, user_id)` or `(user_id, resource_id)` match common queries (“who can access this category?”, “what can this user access?”) with index-backed plans instead of scanning wide parent rows.
- **Better scaling with sparse grants**: Memory and I/O stay proportional to **actual grants**. Embedding large or growing permission structures on every resource row wastes space when most resources have few collaborators; row-per-grant tables stay sparse.
- **Reduced overflow and evolution risk**: Fixed-width bitmasks on the parent row tie permission vocabulary to a small integer width; separate rows avoid “running out of bits” when adding capabilities, and avoid awkward migrations that rewrite every parent row.
- **Less contention on hot rows**: Permission changes update narrow rows in the permission table instead of rewriting large resource rows that are also updated for content.
- **Clearer security and auditing**: Optional columns (`granted_at`, `granted_by`, source) and constraints (foreign keys, uniqueness per principal/resource) are straightforward; row-level security policies can reference a single, well-defined join pattern per resource.
- **Consistent cross-cutting pattern**: The same shape (principal, resource id, encoded rights) can be repeated per domain entity, keeping services and migrations predictable.
- **Less code duplication**: The middleware can be used for any resource type and can be changed in single place.
- **Request can be blocked before any action is taken**: This will save resources and time and help with auditing.

### Tradeoffs

- **More tables and joins**: Authorization checks and listings require joins (or subqueries) to permission tables; indexes must be maintained.
- **Per-resource-type definitions**: Unlike one polymorphic table, each resource may need its own migration and repository helpers (mitigated by shared naming and small shared helpers).
- **Cardinality**: Very large collaborator sets still mean many rows; that is expected and remains cheaper than duplicating fat structures on every parent row when indexed appropriately.

## References

- Example entity: `CategoryPermission` in the content service maps users to categories with permission bits in a dedicated table.
