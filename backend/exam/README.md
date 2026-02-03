# Exams

Exams are list of questions and properties to generate result exam.

## Queries

Describes the logic of how to get questions into the exam based on the query params.

- `Type` - Type of query (questions, category, tags)
- `ObjectIDs` - IDs of objects to include in the exam
- `Count` - Number of questions from this query to include in the exam
- `Filters` - Filters to apply to additionaly filter the questions (public, author, difficulty, etc.)

## Endpoints

- `GET /Exam/` - Get exams by parameters
- `POST /Exam/` - Create or update exam
- `DELETE /Exam/` - Delete exam
- `GET /Exam/render` - Render exam with live WS connection
