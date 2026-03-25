# ADR 0009: Dedicated migration service (separate from API binaries)

## Status

Accepted

## Context

Schema changes are implemented with GORM `AutoMigrate` and related hooks (including RLS) via `shared/migration`.

Running migrations inside services individually in their `main` `init()` functions would lead to increased startup coupling to the database, and makes it harder to run migrations **once** before deploy without starting the full HTTP/gRPC stack.

We want clear **dev** ergonomics (rebuild on entity changes like other Go services), and a **prod** path that runs migrations only **one-time**.

## Decision

- **Dedicated migration service** — Only **`cmd/migration`** **imports** domain packages and calls their **`Migrate` methods**.

Domains expose migration entrypoints in **`services/<servicename>/entities/migrator.go`** (`package entities`, `func Migrate`) and will add an call to it in the migrator command;

They **do not** import the whole migration orchestrator (no circular dependency). `TableMigrationConfig`, hooks, and custom steps live in that file in the top level of the entity package.

- **`backend/cmd/migration`** — Loads config (shared host/port/user/pass), then for **each** registered service opens a connection to that service’s database name from env (e.g. **`CNT_DB_NAME`**, **`IDE_DB_NAME`**) and calls the matching `entities.Migrate`. The effective `DB_NAME` changes **per step** during the run; a placeholder `DB_NAME` in env may exist only to satisfy config parsing.
- We will use the Postgres superuser (or equivalent) from env so it can migrate every service database in one process.
- **Development** — In development migrations are run as parallell service that will watch for changes in the entities and run the migrations.  
    - Docker Compose service **`be-migration`** uses **`migration-watch.sh`** (`inotifywait`) to rebuild and run `cmd/migration` on file changes; **Compose develop.watch** syncs sources from the host. Other Go services still use **Air**; migration intentionally does not and only runs on file changes, then exits and reruns when the sources change.
- **Production** — Build a **`migration`** binary in the prod image (`Dockerfile_prod` target `quizdo-migration`). **One-shot** container: **`cmd/migration` exits 0** on success (no healthcheck). **`be-content`** depends on **`be-migration`** with **`condition: service_completed_successfully`**. Push the **`be-migration`** image with the rest of the stack (`compose.deploy.yaml`). **After a successful run**, **record the migration completion datetime** (audit / operations) where desired.
- **Runtime services** (e.g. `cmd/content`) **do not** run migrations on startup; they assume schema is already applied (enforced in dev by Compose ordering).
- The migration service could be extended to run seeding data after the migrations based on given flags.
- Makefile should contain targets to run the migrations for all services or for a specific service.

## Consequences

### Benefits

- **Smaller conceptual surface for API binaries**: content/identity servers focus on serving traffic, not schema migration.
- **Separate migration lifecycle**: operators and CI can run migrations independently of scaling API replicas.
- **Dev parity**: file changes to entities or migration wiring trigger the same watch/rebuild pattern as other backend services.
- **Ordering**: **dev** — healthcheck marker + `service_healthy` (watch script keeps the container alive after each run). **prod** — **`service_completed_successfully`** on exit code 0.
- **Clear imports**: the migration binary depends on domains; domains do not depend on a migration orchestration package.

### Tradeoffs

- **Separate build and deploy** for API and migration services. Two artifacts to build and deploy (microservices + migration image).
- **Migration must run** (or have already run) before new code that depends on new schema; release discipline is required.
- **Identity DB** (or other services) will need to add `services/<servicename>/entities/migrator.go` and an extra `entities.Migrate` call in `cmd/migration/main.go`.
