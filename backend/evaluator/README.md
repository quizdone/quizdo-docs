# Evaluator service

> [**Evaluator**](../../general/components/Evaluator.md)

HTTP API for **manual**, **semi-automatic**, and **automatic** evaluation of live testing results. It provides a **list of results from users who launched live testings** — a collection of test results and watched behaviors — and exposes endpoints for listing, grading, and reporting.

## Data source

The Evaluator gets data from **Collector text files** (file-based test logging using JSON lines format) and parses it to enable evaluation of the answers.

- The Collector service writes test results and watched behaviors to these files.
- The Evaluator reads them to build the list of results and support evaluation and reports.
- Each evaluated session data and results is written to the database with the evaluation type and the evaluation result.

## Role

- **List results from live testing** — List live testing runs: collection of test results and watched behaviors from users who launched live testings (exams, campaigns).
- **Show answer details** — Show question, correct answer, and user answer for each response; show behavior data when available and let the user evaluate the answers manually or assist with AI.
- **Evaluate** — Manual grading, semi-automatic (e.g. by rules), or automatic evaluation (e.g. using question result data or AI when enabled). Evaluation type for each question is logged in the results database.
    - Manual grading is done by the user seeing the results and grading the answers manually.
    - Semi-automatic grading is done using predefined answers and rules.
    - Automatic grading is done using question predefined answers and AI (when enabled).
- **Reports** — Generate reports summarizing performance, scores, and outcomes.

## API

Evaluator is an HTTP service. Typical endpoints (to be defined in this repo):

- `GET /results/` — List evaluation results / sessions by parameters (from Collector log files or from DB for  in progress evaluations).
- `GET /results/:id` — Get answer details and watched behaviors for a result or session.
- `POST /results/:id/evaluate` — Submit or update evaluation (manual or trigger automatic).
- `GET /results/report` — Generate or download a report (e.g. by session, exam, campaign).

## Related

- Consumes **Collector** output: text files (file-based test logging) produced by the [Collector service](../collector/README.md)
- If the evaluation is in progress, it uses the DB to store the evaluation data and results.
- Used by the **frontend** GUI for viewing and evaluating results and generating reports.
