# ADR 0001: Docker, Go, PostgreSQL

## Status

Accepted

## Context

We need a consistent, portable runtime for backend services and databases across development and production.

Also don't want **to pollute the host machine** with unnecessary software needed only for development so we will use Docker to isolate the services and the database and automate the setup.

For the backend we will use Go and Fiber and GORM for the database access, as the Go is one of the most popular languages for backend development and Fiber is a fast and easy to use web framework for Go and GORM is a mature ORM for PostgreSQL.

## Decision

- **Docker** — Containerize all services for isolation, reproducibility, and deployment.
- **Production images** — Go services: multi-stage build, `CGO_ENABLED=0`, `scratch` runtime with CA certs and zoneinfo from Alpine, non-root `USER 10001`. Next.js: `output: 'standalone'`, `oven/bun:alpine` runtime; Drizzle `db:migrate` runs in the container entrypoint before `bun server.js`. Production Compose applies `read_only`, `tmpfs` on `/tmp`, and `cap_drop: [ALL]` on application containers (not PostgreSQL).
- **Go** — Backend language for services (performance, single binary, strong concurrency).
- **PostgreSQL** — Primary relational database for both frontend and backend data.
- **Redpanda** — Kafka-compatible event streaming for async communication between services (see [ADR 0014](0014-redpanda-event-streaming.md); supersedes the earlier RabbitMQ plan).

## Consequences

- Consistent dev/prod parity
- Docker is used to isolate the services and the database and prevent the services from accessing the host machine and the database from accessing the host machine.
- Single-binary Go services simplify deployment
- Release containers are built on CI/CD pipeline and stored in the GitHub Container Registry (GHCR) to reduce the final deployment cost.
- PostgreSQL provides RLS, JSON support, and mature tooling. It's one of the fastest and most popular databases for backend development.
- Redpanda enables decoupled, async workflows (domain events, background processing) without blocking HTTP/gRPC; consumers use the Kafka API and consumer groups.

## Diagram

```mermaid
flowchart TB
    subgraph Host["Host"]
        subgraph Entrypoints["Entrypoints"]
            direction LR
            API["API Entrypoint"]
            NextJS["Next.js Frontend"]
        end

        subgraph Docker["Docker"]
            subgraph Backend["Backend microservices"]
                Content["Content"]
                Identity["Identity"]
                Generator["Generator"]
            end

            PG[(PostgreSQL)]
            RP[Redpanda]
        end
    end

    API --> Content
    NextJS --> Content
    Identity --> Content
    Generator --> Content
    Content --> PG
    Identity --> PG
    Generator --> PG
    Content --> RP
    Identity --> RP
    Generator --> RP

    style Host fill:#b3e5fc,stroke:#5eb8e8,color:#2a7ab5
    style Entrypoints fill:#f8bbd0,stroke:#e891a8,color:#a82d5a
    style Docker fill:#c8e6c9,stroke:#8bc98b,color:#2d6a2d
    style Backend fill:#e1bee7,stroke:#b88fbf,color:#5e3d65
    linkStyle default stroke:#888
```
