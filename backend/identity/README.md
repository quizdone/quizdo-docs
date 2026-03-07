# Identity service

Handles identity, sessions, and auth context for the rest of the system.

## Role

- **Session and user loading**: Load session and user from the database by token.
- **ID resolution**: Map external UUIDs to internal integer IDs for use in other services (e.g. RLS and authorization).
- **gRPC**: Exposes identity data to other services over gRPC.
- **Optional API auth**: When the API is used outside the Next.js app (external API usage), this service can issue and validate JWT tokens in exchange for API keys.

> **JWT keys**: The frontend (AuthJS) signs JWTs with an RSA private key; backend services that verify JWTs (e.g. Content) use the matching public key from Docker secrets. See [ADR 0007: Docker Secrets and RSA Key Pair for JWT](../adr/0007-docker-secrets-rsa-jwt.md).

> The users coming from the frontend app are authenticated via AuthJS; this service focuses on session/user loading and ID resolution for other services.
