# ADR 0006: JWT Verification and User Profile Middleware

## Status

Accepted

## Context

Authenticate HTTP requests with JWTs and expose a verified user identity and profile to handlers. Verification must happen once; downstream code trusts the token in Locals.

## Decision

- **JWT middleware** (`shared/jwt`) — Validates Bearer token with RSA public key (RS256), stores parsed token in `c.Locals(JWTContextKey)`. Invalid/missing token → 401.
- **No decryption** — JWTs are signed; the public key verifies authenticity. The IDs are UUID so its safe to use it in JWT payload.
- **User profile middleware** (`shared/middleware.LoadUserProfile`) — Runs after JWT. Reads token from Locals, extracts `sub` and `tenant_id`, calls `UserProfileLoader`, sets profile and RLS keys in context.
- **UserProfileLoader** — `LoadProfileFromClaims` (from JWT claims only) or `LoadProfileFromIdentity` (gRPC Identity service).

## Consequences

Single verification point; clear split between auth and profile loading; profile from claims or Identity by swapping the loader.

## Workflow

```mermaid
flowchart LR
    B[jwt.Middleware] --> C{Valid?}
    C -->|No| D[401 Unauthorized]
    C -->|Yes| E[Store token in Fiber.Locals]
    E --> F[LoadUserProfile]
    F --> H[UserProfileLoader]
    H --> I[Set profile and RLS in Locals and context]
    I --> J[Handler]
```
