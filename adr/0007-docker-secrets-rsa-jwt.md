# ADR 0007: Docker Secrets and RSA Key Pair for JWT

## Status

Accepted

## Context

Auth and JWT important keys must not live in `.env` or in images. The frontend signs JWTs; backend services verify them. Keys must be shared consistently and rotatable without rebuilding.

## Decision

- **Secrets in repo root** — A `secrets/` directory holds generated secret files; it is gitignored except for keeping the directory.
- **One RSA key pair** — One private key (frontend signs JWTs); each backend that verifies gets the matching public key. Identity does not use a key.
- **Docker secrets at runtime** — Keys are not in images; they are mounted from the host when containers run. Frontend receives the private key; each service (e.g. Content) receives its public key.
- **Generation** — A single make target generates/overwrites all secret files so keys can be rotated easily.

## Consequences

- Secrets stay off repo and out of images; rotation does not require image rebuilds. But advised to eg. reload the keys on the services.
- Clear split: frontend signs, backends verify with their copy of the public key.

## Related

- [ADR 0006: JWT Verification and User Profile Middleware](0006-jwt-verification-and-user-profile-middleware.md)
- [ADR 0002: Next.js + Strong TypeScript](0002-nextjs-strong-ts.md)
