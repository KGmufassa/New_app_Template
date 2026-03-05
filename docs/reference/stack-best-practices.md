# STACK BEST PRACTICES

## Stack Overview

- Frontend: SwiftUI (iOS native) + XCTest
- Backend: FastAPI (Python)
- Database: PostgreSQL 16 + pgvector extension
- ORM: SQLAlchemy 2.0 + Alembic
- Deployment Target: AWS (ECS Fargate, RDS PostgreSQL, S3, SQS)

---

## Recommended Folder Structure

- Keep client and backend separate: `apps/ios/`, `services/api/`, `services/worker/`.
- In FastAPI, organize by feature + layers: `api/routes`, `services`, `repositories`, `schemas`, `models`, `infrastructure`, `config`, `tests`.
- Keep migration assets isolated under `migrations/` (Alembic) with versioned scripts.
- Keep shared cross-feature helpers in `core/`; avoid cross-feature direct imports.

---

## Service & Layering Patterns

- Use FastAPI dependency injection (`Depends`) for service wiring and request-scoped dependencies.
- Keep route handlers thin; business logic belongs in service layer.
- In SQLAlchemy 2.0, manage transaction boundaries at outer service boundary (`with session.begin():`).
- Use repository interfaces for data access; no direct DB calls from controllers.
- Prefer eager loading (`selectinload`) for known relationship access to avoid N+1 query patterns.

---

## Configuration Management

- Use typed settings via `pydantic-settings` `BaseSettings`.
- Cache settings object (`@lru_cache`) and inject via FastAPI dependency.
- Centralize environment variable loading in one config module.
- Separate environment configs for dev/staging/prod; do not hardcode secrets.

---

## Database Best Practices

- Use PostgreSQL relational modeling with explicit constraints and indexed foreign keys.
- Design partitioning only when access patterns justify it; partition by columns heavily used in `WHERE` clauses.
- Ensure partition pruning remains enabled and validate with `EXPLAIN` in performance tests.
- Use SQLAlchemy 2.0 session lifecycle consistently (session per request/job, explicit commit/rollback).
- Manage schema evolution through Alembic migrations with tested rollback paths.
- Use parameterized queries/ORM-bound parameters exclusively; never build SQL with string concatenation.

---

## API Patterns

- Version APIs under `/api/v1`.
- Validate input with Pydantic request models before service execution.
- Standardize structured error responses (no raw stack traces to clients).
- Use dependency overrides in tests for deterministic integration scenarios.
- Keep idempotent endpoints for ingestion and background job submission.

---

## Testing Strategy

- iOS: XCTest for unit/UI tests; cover auth flow, folder hierarchy, query UX, quota states.
- Backend: pytest + FastAPI `TestClient` for API tests.
- Use dependency overrides for settings/provider stubs in FastAPI tests.
- Add service-layer unit tests plus integration tests for critical flows: ingest -> transcribe -> index -> query.
- Add migration tests and representative DB query plan checks for performance-sensitive paths.

---

## Deployment & DevOps

- Use containerized API/worker deployments on ECS Fargate.
- Use managed PostgreSQL (RDS), private object storage (S3), queue workers (SQS).
- Run one CI pipeline for lint, tests, migration checks, and image build.
- Require readiness/liveness endpoints and rolling/blue-green deployment safeguards.
- Keep infrastructure as code and separate environments with strict secret management.

---

## Performance Considerations

- Prioritize async/background processing for ingestion, transcription, and embedding.
- Keep DB hot paths indexed for user-scoped retrieval and recency ranking.
- Use vector retrieval latency budgets and monitor p95 query timings continuously.
- Batch heavy operations in workers; avoid blocking API handlers on long-running tasks.
- Use efficient relationship loading (`selectinload`) and avoid unnecessary lazy-load round trips.

---

## Security Considerations

- Enforce TLS in transit and AES-256 encryption at rest.
- Use least-privilege IAM roles for API, worker, DB, and storage access.
- Serve media through signed URLs from private buckets only.
- Centralize authN/authZ checks and user-scope filtering in service/repository layers.
- Log auth events, admin actions, and security alerts with correlation IDs.
- Keep sensitive category handling explicit in policy and expose user warning for third-party fallback.
