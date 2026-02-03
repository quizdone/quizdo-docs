# Quizdo backend (API)

- HTTP server with Gin
- Database (MariaDB) managed by [GORM](https://gorm.io/) (migration, CRUD ops),
- Errors gathered with Sentry.io
- Provides commands (in `./cmd/`) for
    1. DB migration (create tables)
    1. DB seed basic data (root user, categories)
    1. Generating Typescript types for frontend

## Endpoint groups

- [`/Cat/`](./category/README.md) - Category CRUD operations

- [`/Question/`](./question/README.md) - Questions CRUD operations and rendering

- [`/Exam/`](./exam/README.md) - Exams CRUD operations, rendering and live WS connection

## Model (data structure)
>
> Models (data structures) and other backend docs are located on the backend [Go App submodule](https://gitlab.com/pisemkomat/goapp)

- Backend models are made to hold data and define DB structure for questions, categorization, quizes and translation of these.
- Backend serves for basic **CRUD** operations on these resource

### Protobuf and protovalidate integration

Protobuf makes it possible to share the same data structures between frontend and backend services.

- Protobuf and protovalidate integration is located in the `projectroot/proto/` directory
- To generate the code, run `make proto-generate` from the `projectroot/` directory
- To clean the generated code, run `make proto-clean` from the root of the repository
