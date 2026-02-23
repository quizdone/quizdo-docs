# Exam (Content API)

Exams are list of questions and properties to generate result exam.

## Queries

Describes the logic of how to get questions into the exam based on the query params.

- `Type` - Type of query (questions, category, tags)
- `ObjectIDs` - IDs of objects to include in the exam
- `Count` - Number of questions from this query to include in the exam
- `Filters` - Structured filters to apply to additionally filter the questions (public, author, difficulty, etc.)

## Endpoints

- `GET /Exam/` - Get exams by parameters
- `POST /Exam/` - Create or update exam
- `DELETE /Exam/` - Delete exam
- `GET /Exam/preview` - Render exam preview for the user
- `POST /Exam/livetest` - Enable live testing and result collection for the exam and produce a link to the exam for the user
