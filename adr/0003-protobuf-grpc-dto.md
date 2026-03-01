# ADR 0003: Protobuf for gRPC and Unified DTOs

## Status

Accepted

## Context

We need a consistent way to define service contracts and data structures across Go backend services and the Next.js frontend.

Inter-service communication uses gRPC; HTTP APIs and frontend consume the same data shapes.

## Decision

- **Protobuf** — Define all service contracts and DTOs in `.proto` files under `proto/`.
- **gRPC** — Use Protobuf for gRPC inter-service communication.
- **Unified DTOs** — Same Proto messages used for gRPC, REST (JSON), and frontend TypeScript types via `buf generate`.

## Consequences

- Single source of truth for service contracts and data structures
- Type-safe contracts across Go, gRPC, and TypeScript
- `buf` for linting, breaking-change detection, and code generation
- Proto models shared between backend and frontend via generated code

## Implementation

- Protobuf files are located in the `proto/` directory.
- The `buf.gen.yaml` file is used to generate the code for the backend and the frontend.
- Every DB entity should have corresponding "toDto()" method converting model to DTO. Every data transformation should be prepared in the entity