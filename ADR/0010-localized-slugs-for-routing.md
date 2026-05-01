# ADR 0010: Localized Slugs for Routing and Resource Lookup

## Status

Accepted

## Context

Current routing is inconsistent: some screens and APIs use numeric IDs, while category finder path sync uses ad-hoc shortcut values. Shortcuts are not guaranteed URI-safe, are not locale-first, and are difficult to standardize across resources.

We need one strategy for readable URLs, locale-aware routing, and backward compatibility with existing ID references.

## Decision

- Introduce first-class localized `slug` fields for categories, tags, and tag groups in proto/API contracts.
- Standardize slug validation with `buf.validate`:
  - allowed format: `^[a-z0-9]+(?:-[a-z0-9]+)*$`
  - reserved words blocked for categories and URI-critical endpoints.
- Keep existing numeric ID compatibility:
  - **List/Detail** endpoints accept `GET /<resource>/:idOrSlug` (numeric segment → ID lookup, otherwise slug lookup). Optional list filtering by slug and fulltext is specified in [ADR 0011](./0011-slug-identified-lookup.md).
  - **Delete** uses numeric ID in the path only: `DELETE /<resource>/:id`. Any earlier assumption here that delete accepted `/:idOrSlug` is **superseded by [ADR 0011](./0011-slug-identified-lookup.md)**.
  - **Create/update** accepts numeric ID only in body (`id`) when updating.
- Enforce uniqueness at persistence layer with composite key: `tenant_id + locale + slug`.
- Use deterministic collision handling at write time (`slug`, `slug-2`, `slug-3`, ...).
- Category hierarchical routing uses slug path segments in the UI (`path=a|b|c`) with fallback resolution for legacy shortcut-based links during rollout.

## Consequences

- URLs become human-readable and locale-scoped across resources.
- Existing ID links remain valid, reducing rollout risk.
- Backfill migration is required for existing rows with missing slugs.
- `List` endpoints support optional query filtering by the exact slug. `Slug` is also included in the fulltext search index.
- `Detail` loader handler need dual parser logic for the `:idOrSlug` path segment and check for reserved words and NON-ASCII characters.
- `Delete` or `Create/Update` handlers stay ID-only (no slug parsing on delete or create/update).

## Rollout Notes

1. Add nullable `slug` columns and uniqueness indexes with `WHERE slug IS NOT NULL`.
2. Backfill existing localized rows with deterministic slug generation and collision suffixing.
3. Make `slug` required in write contracts and DB constraints.
4. Switch frontend URL construction to slug-first.
5. Keep shortcut/ID fallback until old links are no longer needed.
