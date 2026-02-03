# Category

- Basic endpoints for CRUD category management.
- Category is the core of the Quizdo app, allowing users to categorize their questions and use it in their quizzes.
- **Hierarchical levels** (using *ParentID*):
    1. Subject (Math, Physics, ...)
    2. Field (Adding, Quadratic equations, Thermodynamics, ...)
    3. Custom user defined categories
- [category.model.go](https://gitlab.com/pisemkomat/goapp/-/blob/main/model/category.model.go), [category.go](https://gitlab.com/pisemkomat/goapp/-/blob/main/endpoints/eapp/category.go)

## Endpoints

- `GET /Cat/` - Get categories by parameters
- `POST /Cat/` - Create or update category
- `DELETE /Cat/` - Delete category
