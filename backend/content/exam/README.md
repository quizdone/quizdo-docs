# Exam (Content API)

> [**Quiz**](../../../general/components/Quiz.md)

Exams are list of questions and properties to generate result exam.

It also holds options for the exam, like **Number of questions** or if it should be **randomly shuffled** etc.

## Queries

Describes the logic of how to get questions into the exam based on the query params.

- `Type` - Type of query
    - `questions` - Questions by IDs
    - `category` - Questions by category
    - `tags` - Questions that contain any of the specified tags
    - `tagGroup` - Questions by tag group
    - `tagGroupValue` - Questions by tag group value (with optional operator like `exact`, `min`, `max`, `between`)
- `QueryData` - Allowed data for the specified type of query
    - `questions` - Array of question IDs
    - `category` - Category IDs
    - `tags` - Array of tag IDs
    - `tagGroup` - Tag group ID, operator and values
    - `tagGroupValue` - Tag group value (with optional operator like `exact`, `min`, `max` to specify range of values)
- `Count` - Number of questions from this query to include in the exam
- Multiple conditions can be combined to specify the query conditions. Eg. we want to select from category 1 if it has tag `university` and `difficulty` is `easy` or `medium`.
- Multiple queries are combined to create a single exam. The queries are stored to JSON field in DB.

## Endpoints

- `GET /Exam/` - Get exams by parameters
- `POST /Exam/` - Create or update exam
- `DELETE /Exam/` - Delete exam
- `GET /Exam/preview` - Render exam preview for the user
