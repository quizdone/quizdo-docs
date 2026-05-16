# Category component

## Overview

Categories are the **hierarchical** library structure in Quizdo: subjects, fields, and user-defined nodes. Users attach **questions** and browse **exams** under categories via the **category finder**.

## Purpose

- **Content organization**: Place questions and exams in a tree (`parent_id`, `depth`).
- **Multi-language**: Localized title, shortcut, description per locale (`CategoryLocalized` in **proto** `models.v1`).
- **Readable routing**: Localized `slug` per locale for URI-safe routes and API lookups.
- **Visibility**: `public` categories are readable by anyone with access to the tenant context; non-public rows need an explicit grant or authorship.
- **Ownership**: Author and optional tenant-specific confirmation flow (`unconfirmed` — only the author can see and use unconfirmed categories in listings until confirmed).
- **Collaboration**: Per-user bitmask grants in `category_permissions` (VIEW, EXTEND, EDIT, MANAGE). The API returns the caller’s effective mask as `permission` on `Category` / `CategoryMinified`; the app uses it for edit, delete, and “add child” actions. See [ADR 0008](../../ADR/0008-per-resource-permission-tables.md).

## Hierarchical structure

1. **Depth 0** — Subjects (e.g. Mathematics, Physics).
2. **Depth 1** — Fields within a subject.
3. **Depth 2+** — Custom subcategories.

## In the current app

- **Contracts**: [`proto/models/v1/category.proto`](../../../proto/models/v1/category.proto) — `Category`, `CategoryLocalized`, `CategoryMinified`, `CategoryPermissionBit`, filters, create/update requests; TS/Go generated under `frontend` and `backend`.
- **Backend**: [Content service](../../backend/content/README.md) — Fiber handlers for category CRUD; Identity **gRPC** for profile after JWT verification.
- **Frontend**: **Categories** and other content pages use **`CategoriesFinder`** (tree, lazy children, URL `path` sync). URI `path` segments are slug-first (fallback to legacy shortcut for compatibility).
