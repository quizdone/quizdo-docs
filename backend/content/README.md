# Content service

> [**General components**](../../general/components/README.md)

HTTP API (Fiber) for content management of categories, questions, exams, and campaigns.

## Role

- **CRUD** for content resources: categories, questions, exams, and campaigns.
- **REST API** with JWT verification — JWTs are signed by Next.js with `jwt_private_key` and verified here with `jwt_public_key` (see [ADR 0007](../ADR/0007-docker-secrets-rsa-jwt.md)).
- **Loading profile from Identity service** — **when JWT is verified by middleware**, the service calls **Identity gRPC** `GetUserProfile` to load user data and internal profile data for using in the app.
- **Authorization** — Per-resource permission tables and bitmask checks ([ADR 0008](../../ADR/0008-per-resource-permission-tables.md)): mutating routes use `RequirePermission` middleware; list/detail responses include `permission` for the current viewer (`list_permission` SQL scope). Identity does not evaluate resource ACLs.
- **Live testing** — To **plan** a live test and obtain the **participant link**, this service calls the **Collector** **livetest** **HTTP** API (exam/campaign flows). The client still talks to Content; Content orchestrates the call to Collector and returns the link.

## Areas

- [**Category**](category/README.md) — Category CRUD; hierarchical (subject → field → custom).
- [**Question**](question/README.md) — Questions CRUD and rendering.
- [**Tags**](tags/README.md) — Tags and tag groups; group-level permission rows.
- [**Exam**](exam/README.md) — Exams CRUD, rendering, and live WS connection.
- [**Campaign**](campaign/README.md) — Campaign CRUD; series of exams and conditions to advance.

## Related

- Uses **identity** (gRPC) for user/session context; **RLS** on tenant-scoped rows; **resource ACLs** via `*_permissions` tables and [ADR 0008](../../ADR/0008-per-resource-permission-tables.md).
