# Content service

HTTP API (Fiber) for content management of categories, questions, exams, and campaigns.

## Role

- **CRUD** for content resources: categories, questions, exams, and campaigns.
- **REST API** with JWT verification — tokens are signed by the frontend with an RSA private key and verified here with the matching public key (supplied via Docker secret `cnt_pub_key`; see [ADR 0007](../adr/0007-docker-secrets-rsa-jwt.md)).

## Endpoint groups

- [**Category**](category/README.md) — Category CRUD; hierarchical (subject → field → custom).
- [**Question**](question/README.md) — Questions CRUD and rendering.
- [**Exam**](exam/README.md) — Exams CRUD, rendering, and live WS connection.
- [**Campaign**](campaign/README.md) — Campaign CRUD; series of exams and conditions to advance.

## Related

- Uses **identity** (gRPC or JWT) for user/session context and RLS using `(IsPublic | AuthorID) and TenantID`.
