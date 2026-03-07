# Backend (microservices)

The backend is split into **microservices**, each with its own entrypoint, data, and communication style. Shared code (DB, JWT, config) lives in `backend/shared/`. Data structures are shared with the frontend via **Protobuf** in `proto/`.

## Architecture

- Each service has its own **database schema** for data isolation.
- Services use **gRPC** or **REST (Fiber)** for APIs.
- **GORM** with RLS (Row Level Security) for DB access.
- Auth is handled by **AuthJS** in the app (frontend signs JWTs with an RSA private key); backend services that protect APIs (e.g. Content) verify JWTs with an RSA public key supplied via Docker secrets. See [ADR 0007: Docker Secrets and RSA Key Pair for JWT](../adr/0007-docker-secrets-rsa-jwt.md).

## Services

| Service   | Description |
|----------|-------------|
| [**Identity**](identity/README.md) | JWT/session validation, user resolution, UUID→internal ID; gRPC for other services. |
| [**Content**](content/README.md)   | Content API: categories, questions, exams, answers; HTTP (Fiber). |
| [**Collector**](collector/README.md) | Runs live sessions; file-based test logging (text files using JSON lines format) for results and behaviors; consumed by Evaluator. |
| [**Evaluator**](evaluator/README.md) | Evaluation API: list results from Collector (file-based logs), grading, reports. |

## Repo layout

- `backend/cmd/<service>/` — entrypoint and config per service (e.g. `main.go`, `.env`, Dockerfile).
- `backend/services/<service>/` — service logic (handlers, services, repos, gRPC).
- `backend/shared/` — shared code (DB, JWT, config, logging).
- `proto/` — Protobuf definitions; run `make proto` from project root to generate code.

## Running

- Run a service from the repo root, e.g. `make be` (see Makefile), or from `backend/cmd/<service>` with the right `.env`.
- Migrations and seeds are service-specific; see each service’s README.

## Related

- [**Generator**](generator/README.md) — External Go tool (separate repo) used by the content service to generate and validate questions from parametric data.
- [**Collector**](collector/README.md) — Runs live sessions and writes file-based test logs (text files) consumed by the Evaluator.
