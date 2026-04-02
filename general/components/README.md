# App components

This folder is the **source of truth** for **what each domain component means in Quizdo** in a **user-readable** way: teacher/student workflows, purpose, and how concepts fit together (the “user story” view).

- **Use these docs** when explaining the product, onboarding, or aligning features.
- **[Backend](../../backend/README.md)** and **[frontend](../../frontend/README.md)** docs describe **implementation** (services, routes, stack). They **link here** for domain meaning and must not be the only place where “what is a tag / exam / campaign?” is defined.

For architecture and deployment decisions, see [ADRs](../adr/README.md).

---

- [Categories](Category.md) — Hierarchical library (subjects, fields, custom); **proto** `models.v1.Category`; Content API + category finder in the app.
- [Tags](Tags.md) — Non-hierarchical labels and tag groups (alongside categories); UI copy present; persistence **planned**.
- [Questions](Question.md) — Parametric questions, generator, localization; category finder + question lists (**stub** until Content APIs land).
- [Quizzes](Quiz.md) — Exams built from questions; backend **Exam** resource; exams page mirrors questions flow.
- [Campaigns](Campaign.md) — Sequences of exams with advance conditions.
- [Collector](Collector.md) — Live session results; file-based logs for Evaluator.
- [Evaluator](Evaluator.md) — Manual, automatic, or AI-assisted grading; reports.
- [Generator](Generator.md) — External generator library; **gRPC** service + hash/cache (**planned**); see [backend generator](../../backend/generator/README.md).
