# Architecture and workflow

High-level architecture, deployment, and how the main workflows fit together.

## Overview

- **Frontend** — Next.js app (GUI). Auth, user/session/API-key management, content UI. Talks to backend via API.
- **Backend** — Microservices (Identity, Content, Generator, Collector, Evaluator). REST + gRPC, per-service DB or file-based logs.
- **Databases** — Frontend DB (users, sessions, preferences); backend DBs (content, identity, etc.). Live testing results are file-based (text logs).

See [Backend](../backend/README.md) and [Frontend](../frontend/README.md) for service-level detail.

## Containers and deployment

Containers are used for development and production. Requests are proxied by **Nginx** to the **Frontend** or **Backend** containers. Docker adds isolation and part of the security model.

- **Compose files**: `docker-compose.yml` (dev), `docker-compose.prod.yml` (production). Deploy builds: `docker-compose.deploy.yml`.

### 1. Frontend

- **Type**: Single Page Application (SPA).
- **Tech**: React (Next.js), TypeScript, Tailwind, AuthJS with Drizzle adapter, Drizzle ORM.
- **Responsibilities**:
  - Public website
  - User authentication (AuthJS with DrizzleAdapter)
  - Database access using Drizzle ORM (frontend DB)
  - UI for managing and displaying categories, questions, exams, campaigns (including progressive/campaign mode)
  - Interactive student testing and collection of responses; teacher access to results and evaluation
- **Communication**: With backend via API (NextJS server actions and HTTP calls).

### 2. Backend

- **Style**: Microservice architecture.
- **APIs**: REST (Fiber) + gRPC.
- **Tech**: Go, Fiber, GORM.
- **Responsibilities**:
  - Provide data to the frontend (Content API, Evaluator API)
  - Generate and render questions/exams (Generator, Content)
  - Run live testing and persist results (file-based test logging)
  - Evaluate results and generate reports (Evaluator)
  - Identity and JWT validation for other services
- **Data**: Per-service databases; live testing writes text files consumed by Evaluator.

### 3. Database

- **Type**: Relational (RDBMS).
- **Tech**: PostgreSQL (or MariaDB where still in use).
- **Split**:
  - **Frontend DB**: Users, authentication, sessions, API keys, subscriptions, payments, user preferences. Owned by the frontend; backend may read for user/session resolution.
  - **Backend DBs**: Categories, questions, exams, campaigns, business logic, identity data. One schema per microservice where applicable.
  - **Live testing results**: Stored as **text files** (file-based test logging), not in DB; Evaluator reads them for listing and reports.

## Request flow

1. **User** → Nginx → **Frontend** (static/SSR/API routes) or **Backend** (API).
2. **Frontend** (authenticated via AuthJS) → calls **Backend** Content API and Evaluator API (and Identity when needed).
3. **Backend** services use their own DBs and, for live testing, read/write **file-based logs**; Evaluator consumes these logs.
4. **NextJS App** can trigger `Evaluator` live testing for certain content (question/exam), then `Collector` communicates with `Evaluator` and stores results to files; After session is finished `Evaluator` reads files and serves list/evaluate/report to the frontend.
