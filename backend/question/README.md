# Questions

- Questions contain raw question data that can be used by the generator to produce results
- Each question belongs to one category
- Questions can be tagged with multiple tags
- **Usage**: in exams and single rendered results
- [question.model.go](https://gitlab.com/pisemkomat/goapp/-/blob/main/model/question.model.go), [question.go](https://gitlab.com/pisemkomat/goapp/-/blob/main/endpoints/eapp/question.go)

## Endpoints

- `GET /Question/` - Get questions by parameters
- `GET /Question/render` - Render question with validation
- `POST /Question/` - Create or update question
- `DELETE /Question/` - Delete questions
