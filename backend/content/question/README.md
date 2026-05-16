# Question (Content API)

> [**Question**](../../../general/components/Question.md)

- Questions contain raw question data that can be used by the generator to produce results. Task **content** is **MDX** with an app-defined component set only (no author-controlled `import` / `export`); see [ADR 0013](../../../ADR/0013-question-content-mdx-fixed-components.md).
- Each question belongs to one category
- Questions can be tagged with multiple tags
- **Usage**: in exams and single rendered results

## Endpoints

- `GET /question/` - List questions. If the request omits `author` query parameters, results are restricted to the authenticated user’s identity serial (same as passing `author=<self>`). Optional `category` repeats are numeric category ids (question↔category junction). With **`category_subtree`** set (any truthy query value), each `category` id is expanded to include **all descendant** category ids before filtering, so parent categories aggregate questions from the whole subtree.
- `GET /question/:idOrSlug` - Load one question (numeric id or localized meta slug). With query **`preview`** set (any non-empty value), returns a **rendered** variant via Generator instead of the stored template.
- `POST /question/` - Create or update question (one locale slice). With query **`preview`** set (any non-empty value, e.g. `preview=1`), the **same JSON body** is validated and **rendered** via Generator; **nothing is written** to the database (draft preview).
- `DELETE /question/:id` - Delete question (requires **MANAGE**)

### Authorization

See [ADR 0008](../../../ADR/0008-per-resource-permission-tables.md).

| Route | Middleware / check |
|-------|-------------------|
| `POST /question/`, `POST /question/validate` | **EDIT** (`AllowEmptyURIParam` where no `:id`) |
| `DELETE /question/:id` | **MANAGE** |
| `GET /question/:idOrSlug` | Non-`public` rows require author or **VIEW** via `UserCan`; listings include `permission` |

`QuestionPermissionBit`: VIEW (1), EDIT (2), MANAGE (4).
