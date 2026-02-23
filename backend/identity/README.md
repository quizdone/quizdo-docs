# Identity service

Handles identity, sessions, and auth context for the rest of the system.

## Role

- **JWT validation**: Read and validate JWTs (signed with the shared secret).
- **Session and user loading**: Load session and user from the database by token.
- **ID resolution**: Map external UUIDs to internal integer IDs for use in other services (e.g. RLS and authorization).
- **gRPC**: Exposes identity data to other services over gRPC.
- **Optional API auth**: When the API is used outside the Next.js app (external API usage), this service can issue and validate JWT tokens in exchange for API keys.

> This is the only service that holds the **secret key** used to sign and issue JWTs for API authentication.

> The users coming from frontend app will be authenticated via AuthJS, this service will only validate the JWT tokens and load the session and user data.
