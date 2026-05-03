# ADR 0011: Slug-Identified Lookup

## Status

Accepted

## Context

ADR 0010 we want to use human readable localized slugs as first-class unique selectors to be able to use `nice URIs` for the frontend and backend.

We will only use it for `READ` and filtering operations. No `CREATE` or `UPDATE` operations will be allowed with slugs.

## Decision

We will use slug-capable selectors for **read/detail**:
  - `GET /<resource>/:idOrSlug` — handler (e.g. `GetCategory`, `GetTagGroup`, `GetTag`) parses the segment: numeric → service `GetByID`, otherwise → `GetBySlug`.
  - Where **collection/list** endpoints exist (`GET /<resource>` or resource-specific list routes), support optional query filtering (partial matching) for **slug** (add `slug` to the filter params and to fulltext index). Exact query parameter names are defined per handler but the capability is shared.

## Consequences

- Read routes remain human-friendly and locale-oriented
- Slugs won't contain special characters or spaces (non ASCII).
  - `READ` APIs are less error-prone.
- Mutation routes stay explicit and safer (no mixed ID/slug delete semantics).
- Slug is added to both `UNIQUE` and `FULLTEXT` indexes.
