# ADR 0012: Question render HTTP via Generator gRPC

## Status

Accepted

## Context

The pisemkomat **Generator** runs as a separate process exposing `QuestionRendererService` over gRPC (`RenderQuestion`, `RenderQuestionCollection`). The **Content** service owns persisted questions and JWT-protected HTTP APIs.

Clients need one HTTP entry point to obtain a **rendered** question (template expanded) without speaking gRPC to the Generator directly.

## Decision

- Expose persisted-question render as **`GET /question/:idOrSlug?preview=…`** with any **non-empty** `preview` query value (same path as `GetByIDOrSlug`; same auth as other question reads). Content **loads** the question from the DB (respecting the same visibility rules as `GetByID`), builds a `QuestionRenderPayload` from the localized row + core flags, and calls **Generator** `RenderQuestion` over gRPC.
- **Draft preview:** **`POST /question/?preview=…`** with any **non-empty** `preview` query value (e.g. `preview=1`, `preview=true`) and the same JSON body as **`POST /question/`** (create/update). Content maps the body to an in-memory `question.Question` entity (no `Save`), then renders via Generator. Shared helper: **`shared/questionrenderer`** (`NewRenderer` + `RenderLocalized` / `RenderForLocale`, and **`RenderPayload`** for ad-hoc payloads).
- **idOrSlug** (ADR 0011):
  - Numeric segment → question id.
  - Otherwise → lookup by `questions_localized.meta_strings` JSON key **`slug`** (constant `MetaKeySlug` in code).
- Generator dial settings: **`GENERATOR_GRPC_HOST`** (optional; empty = render endpoint returns 503) and **`GENERATOR_GRPC_PORT`** (default `50091`).

## Consequences

- Single HTTP contract for apps; Generator stays an internal dependency.
- Content must run with Generator reachable when render is used; misconfiguration surfaces as 503, not a silent failure.
- Slug-based render requires `meta_strings.slug` to be set on the relevant locale row (until a dedicated slug column exists).
