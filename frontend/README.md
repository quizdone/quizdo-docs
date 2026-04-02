# Frontend

> [**General components**](../general/components/README.md)

The frontend is the **GUI** of the application: a `Next.js` app used to manage content and setup live testing and result collection. It consumes the backend **Content CRUD API** for exams and campaigns.

> It's responsible for the main auth and user management, session management, API key management, preferences management, etc.
>
> Backend only consumes the data from the frontend database for the user and session management.

## Main app (GUI)

- [**GUI**](gui.md) — `Next.js` interface for creating, editing, sharing, and managing user content (categories, questions, exams, campaigns). Also used for viewing results of live testing and evaluating the results.

## Consumed APIs

- **Content API** — CRUD on content (categories, questions, exams, campaigns). See [Content service](../backend/content/README.md).
- **Evaluator API** — API for manual/semi-automatic/automatic evaluating the results of live testing. See [Evaluator service](../backend/evaluator/README.md).

## Tech

- Next.js, React, TypeScript, Tailwind
- AuthJS (with Drizzle adapter) for authentication
- Drizzle ORM for frontend DB (users, sessions, API keys, preferences)
- Server actions and API calls to the backend

## Usage

The main app is available at [Quizdo.eu](https://quizdo.eu).
