# ADR 0009: Dedicated migration service (separate from API binaries)

## Status

Accepted

## Context

Schema changes are implemented with GORM `AutoMigrate` and related hooks (including RLS) via `shared/migration`.

Running migrations inside services individually in their `main` `init()` functions would lead to increased startup coupling to the database, and makes it harder to run migrations **once** before deploy without starting the full HTTP/gRPC stack.

We want clear **dev** ergonomics (rebuild on entity changes like other Go services), and a **prod** path that runs migrations only **one-time**.

## Decision

- **Dedicated migration service** — **`services/migration`** runs config load, parallel **`Step.Migrate`** per DB, then optional seeds by **string key** using **`map[string]func() error`** only; it does **not** import domain entity packages. **`cmd/migration`** registers concrete **`Migrate`** functions and seed closures (domain imports stay in **`cmd`**).

Domains expose migration entrypoints in **`services/<servicename>/entities/migrator.go`** (`package entities`, `func Migrate`) and will add an call to it in the migrator command;

They **do not** import the whole migration orchestrator (no circular dependency). `TableMigrationConfig`, hooks, and custom steps live in that file in the top level of the entity package.

- **`backend/services/migration`** — Loads config (shared host/port/user/pass), then for **each** supplied **`Step`** opens that step’s database name from env (e.g. **`CNT_DB_NAME`**) and calls **`Step.Migrate`**. **`cmd/migration`** builds the **`[]Step`** list and seed name→runner map.
- We will use the Postgres superuser (or equivalent) from env so it can migrate every service database in one process.
- **Development** — In development migrations are run as parallell service that will watch for changes in the entities and run the migrations.  
    - Docker Compose service **`be-migration`** uses **`migration-watch.sh`** (`inotifywait`) to rebuild and run the migration binary on file changes; **Compose develop.watch** syncs sources from the host. Other Go services still use **Air**; migration intentionally does not and only runs on file changes, then exits and reruns when the sources change.
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
- **Identity DB** (or other services) will need to add `services/<servicename>/entities/migrator.go` and register an extra **`Step`** in **`cmd/migration`** (e.g. `migrationSteps()`), not in `services/migration`.
