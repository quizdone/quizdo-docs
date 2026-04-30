# ADR 0007: Docker Secrets and RSA Key Pair for JWT

## Status

Accepted

## Context

Auth and JWT important keys must not live in `.env` or in images. The frontend signs JWTs; backend services verify them. Keys must be shared consistently and rotatable without rebuilding.

## Decision

- **Secrets in repo root** — A `secrets/` directory holds generated secret files; it is gitignored except for keeping the directory.
- **Generation** — `make secrets` generates new RSA key pairs and writes them to `secrets/`, dont forget to mention the new key in the `compose.yaml` secrets section.

## Consequences

- Secrets stay off repo and out of images; rotation does not require image rebuilds. But advised to eg. reload the keys on the services.
- Clear split: frontend signs, backends verify with their copy of the public key.

## Related

- [ADR 0006: JWT Verification and User Profile Middleware](0006-jwt-verification-and-user-profile-middleware.md)
- [ADR 0002: Next.js + Strong TypeScript](0002-nextjs-strong-ts.md)
