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
- `DELETE /Cat/:idOrSlug` - Delete category by ID or slug
- `POST /Cat/user/<user_id>/` - Assign/Reassign category to user - mark usage as subject or interest to simplify the future questions selection for the user

### Slug contract

- Slug is locale-based and URI-safe (`^[a-z0-9]+(?:-[a-z0-9]+)*$`).
- Uniqueness is enforced per `tenant + locale + slug`.
- Existing numeric ID references remain supported.
- Backfill/migration guidance: `SLUG-MIGRATION.md`.
