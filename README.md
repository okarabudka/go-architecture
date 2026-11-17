# go-architecture

A minimal Go web service that serves as a template for a layered project structure (`cmd` / `internal` / `configs`) with Docker support.

## Context

Built in May 2024 as a learning project while picking up Go alongside day-to-day PHP backend work. The goal was to try out a conventional Go project layout and a clean split between application wiring, HTTP endpoints, middleware and business logic, not to ship a product. The business logic is intentionally trivial.

## Features

- HTTP server on port 8080 built with the Echo framework.
- `GET /date` returns the number of days left until the next New Year, for example `Days until New 2027 Year: 45`.
- Route-level middleware (`DateCheck`) that logs the current date before the handler runs.
- Multi-stage Docker build producing a small Alpine-based image.
- `docker-compose.yml` with the app and a PostgreSQL 14 container (prepared for later use; the code does not connect to it yet).

## Tech stack

- Go 1.22
- [Echo v4](https://echo.labstack.com/) web framework
- Docker (multi-stage build: `golang:1.22-alpine` builder, `alpine` runner)
- Docker Compose, PostgreSQL 14 (`postgres:14-alpine`)

## Architecture

```
cmd/app/main.go                     entry point: creates and starts the app
internal/app/app.go                 wiring: service -> endpoint -> Echo routes
internal/app/endpoint/endpoint.go   HTTP handlers, depend on a Service interface
internal/app/service/service.go     business logic (days until New Year)
internal/app/middleware/            Echo middleware
configs/config.yaml                 configuration draft (HTTP, gRPC, PostgreSQL, CORS)
```

Key decisions:

- Everything except the entry point lives under `internal/`, so it cannot be imported by other modules.
- Handlers depend on a small `Service` interface instead of the concrete type, which keeps the layers decoupled and testable.
- `configs/config.yaml` is copied into the Docker image, but the application does not read it yet; the port is a constant in `internal/app/app.go`.

## Getting started

Run locally (requires Go 1.22):

```bash
go run ./cmd/app
curl http://localhost:8080/date
```

Run with Docker:

```bash
docker build -t go-architecture:local .
docker run -p 8080:8080 go-architecture:local
```

`docker-compose.yml` uses the same `go-architecture:local` image and reads `HTTP_PORT` / `HTTPS_PORT` from `.env` (see `.env.example`). Note that it maps container ports 80/443, while the app currently listens on 8080.

## Testing

There are no automated tests.

## Status

Learning project, developed in May 2024 and not actively maintained. The configuration file, PostgreSQL service and Compose port mapping are prepared but not yet wired into the code.

## License

MIT, see [LICENSE](LICENSE).
