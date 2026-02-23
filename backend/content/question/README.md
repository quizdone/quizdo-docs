# Question (Content API)

- Questions contain raw question data that can be used by the generator to produce results. (See [Question](../../../general/components/Question.md) for more details.)
- Each question belongs to one category
- Questions can be tagged with multiple tags
- **Usage**: in exams and single rendered results

## Endpoints

- `GET /Question/` - Get questions by parameters (category tree or detail view)
- `GET /Question/preview` - Render question with validation
- `POST /Question/` - Create or update question
- `DELETE /Question/` - Delete questions
