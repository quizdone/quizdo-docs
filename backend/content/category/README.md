# Category (Content API)

> [**Category**](../../../general/components/Category.md)

- CRUD category management (implementation).
- Category is the core of the Quizdo app, allowing users to categorize their questions and use it in their quizzes.
- **Hierarchical levels** (using *ParentID*):
    1. Subject (Math, Physics, ...)
    2. Field (Adding, Quadratic equations, Thermodynamics, ...)
    3. Custom user defined categories

## Endpoints

- `GET /Cat/` - Get categories by parameters (tree view or detail view data structure)
- `GET /Cat/:idOrSlug` - Get category by numeric ID or locale slug/path segment
- `POST /Cat/` - Create or update category
- `DELETE /cat/:id` - Delete category (numeric id; requires **MANAGE**)
- `POST /Cat/user/<user_id>/` - Assign/Reassign category to user - mark usage as subject or interest to simplify the future questions selection for the user

### Authorization

See [ADR 0008](../../../ADR/0008-per-resource-permission-tables.md).

| Route | Middleware / check |
|-------|-------------------|
| `POST /cat/` | `RequirePermission` → **EDIT** (`AllowEmptyURIParam` on create) |
| `DELETE /cat/:id` | `RequirePermission` → **MANAGE** |
| `GET /cat/`, `GET /cat/:idOrSlug` | Listing/detail scopes + `permission` on response; `unconfirmed` hidden from non-authors in lists |

`CategoryPermissionBit`: VIEW (1), EXTEND (2), EDIT (4), MANAGE (8). Authors receive full mask in `list_permission`; `public` adds VIEW for non-granted viewers.

### Slug contract

- Slug is locale-based and URI-safe (`^[a-z0-9]+(?:-[a-z0-9]+)*$`).
- Uniqueness is enforced per `tenant + locale + slug`.
- Existing numeric ID references remain supported.
- Backfill/migration guidance: `SLUG-MIGRATION.md`.
