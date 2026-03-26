# App components

The app logic is composed of the following parts. For architecture and deployment see [ADRs](../adr/README.md).

- [Categories](Category.md) - Categories (subjects, fields, custom categories) for categorizing Questions, Quizzes, Campaigns
- [Questions](Question.md) - Questions parametrically defined questions for generating different versions (values, languages)
- [Quizzes](Quiz.md) - Quizzes (exams) generated from Questions
- [Campaigns](Campaign.md) - Campaigns (series of Quizzes and conditions to advance to the next Quiz) generated from Quizzes
- [Collector](Collector.md) - Collector for collecting results from Live testing of Quizzes, Campaigns and Questions
- [Evaluator](Evaluator.md) - Show the test answers, allow to evaluate the test results and generate a report
- [Generator](Generator.md) - Generator for generating different versions (values, languages) of Questions, Quizzes, Campaigns
