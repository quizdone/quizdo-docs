# Backend (microservices)

> [**General components**](../general/components/README.md)

The backend is split into **microservices**, each with its own entrypoint, data, and communication style. Shared code (DB, JWT, config) lives in `backend/shared/`. Data structures are shared with the frontend via **Protobuf** in `proto/`.

## Architecture

- Each service has its own **database** for data isolation.
- Services use **gRPC** or **HTTP server (Fiber)** for APIs.
- **GORM** with **RLS (Row Level Security)** for DB access.
- Auth is handled by **AuthJS** in the app (frontend signs JWTs with an RSA private key)
    - backend services that protect APIs (e.g. Content) verify JWTs with an RSA public key.
    - See [ADR 0007: Docker Secrets and RSA Key Pair for JWT](../adr/0007-docker-secrets-rsa-jwt.md).
- **Live testing**: **[Content](content/README.md)** calls the **[Collector](collector/README.md)** **livetest** **HTTP** API to plan a session and get the **link** shared with participants; Collector then runs the session (e.g. WebSocket) and writes logs for **Evaluator**.

## Services

| Service | Type | Description |
|:--------|:-----|:------------|
| [**Identity**](identity/README.md) | gRPC | JWT/session validation, user resolution, UUID→internal ID; exposes gRPC for other services. |
| [**Content**](content/README.md) | REST | Content API: categories, questions, exams, campaigns, tags (domain); HTTP (Fiber). |
| [**Generator**](generator/README.md) | gRPC | Parametric question/exam generation; used by Content (internal gRPC). |
| [**Collector**](collector/README.md) | REST, WebSocket | Plans and runs live testing sessions. |
| [**Evaluator**](evaluator/README.md) | REST | List results from Collector logs, grading, reports. |

## Backend directory layout

- `backend/cmd/<service>/` — entrypoint and config per service (e.g. `main.go`, `.env`).
- `backend/services/<service>/` — service logic (handlers, services, repos, gRPC).
- `backend/shared/` — shared code (DB, JWT, config, logging).
- `proto/` — Protobuf definitions; run `make proto` from project root to generate code. It should build automatically when using `make` to run the service.

## Running

- Run a service from the repo root, e.g. `make be` (see Makefile), or from `backend/cmd/<service>` with the right `.env`.
- Migrations and seeds are service-specific; see each service’s README.

## Related

- [**Generator**](generator/README.md) — External Go tool (separate repo) used by the content service to generate questions from parametric data.
