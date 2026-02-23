# Category (Content API)

- Basic endpoints for CRUD category management.
- Category is the core of the Quizdo app, allowing users to categorize their questions and use it in their quizzes.
- **Hierarchical levels** (using *ParentID*):
    1. Subject (Math, Physics, ...)
    2. Field (Adding, Quadratic equations, Thermodynamics, ...)
    3. Custom user defined categories

## Endpoints

- `GET /Cat/` - Get categories by parameters (tree view or detail view data structure)
- `POST /Cat/` - Create or update category
- `DELETE /Cat/` - Delete category
- `POST /Cat/user/<user_id>/` - Assign/Reassign category to user - mark usage as subject or interest to simplify the future questions selection for the user
