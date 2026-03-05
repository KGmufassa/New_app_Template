# APPLICATION ARCHITECTURE DOCUMENT

This document defines how the application MUST be structured, organized, and implemented.

It serves as the engineering contract for build consistency and long-term maintainability.

---

# 1. Architectural Philosophy

## 1.1 Design Principles

- Separation of Concerns
- Single Responsibility
- Stateless Services (where possible)
- Explicit Dependency Boundaries
- Configuration over Hardcoding
- Observability by Default
- Security by Design
- Testability First

Implementation stance for this product:
- API and workers are stateless; state lives in PostgreSQL, S3, and queue payloads.
- Business logic is isolated in service layer and tested independently.
- Tenant/user scope is mandatory in every repository query path.

---

# 2. Project Structure (Enforced Layout)

## 2.1 Root Directory Layout


/src
/app -> Route definitions / entry layer
/modules -> Feature-based domain modules
/core -> Shared domain logic
/infrastructure -> External systems (DB, cache, APIs)
/interfaces -> Controllers, API handlers, DTOs
/config -> Environment config
/lib -> Utilities (pure functions only)
/types -> Global types/interfaces
/tests -> Unit & integration tests


No business logic allowed in `/app`.

Concrete repository layout:
- `apps/ios/` - SwiftUI iOS client.
- `services/api/src/` - FastAPI HTTP service.
- `services/worker/src/` - background workers for ingestion/transcription/indexing/purge/export.
- `services/api/alembic/` - migrations.
- `docs/reference/` - governing artifacts (classification, framework, PRD, architecture).

---

## 2.2 Module Structure (Feature-Based)

Each domain module must follow:


/modules/{feature}/
index.ts
service.ts
repository.ts
schema.ts
types.ts
validator.ts


### Rules:
- `service.ts` = business logic only
- `repository.ts` = database access only
- `schema.ts` = DB models
- `validator.ts` = input validation
- `types.ts` = module-specific types
- No cross-module imports except through public exports

Mapped to Python/FastAPI naming:
- `service.py`, `repository.py`, `models.py`, `schemas.py`, `types.py`, `validator.py`, `router.py`.

Required feature modules:
- `auth`
- `folders`
- `reels_ingestion`
- `transcripts`
- `semantic_search`
- `ai_query`
- `learning_artifacts`
- `billing`
- `account_lifecycle`
- `audit`

---

# 3. Layered Architecture

## 3.1 Layer Responsibilities

### Presentation Layer
- Handles HTTP / UI input
- No business logic
- Calls service layer only

Concrete scope:
- FastAPI routers (`/api/v1/*`) in API service.
- SwiftUI views/view models in iOS app.

### Service Layer
- Contains core business rules
- Orchestrates repositories
- No framework-specific code

Concrete responsibilities:
- Quota enforcement (free vs paid limits).
- Ingestion policy checks (public-only, compliance confidence gate).
- Reel delete semantics (media delete, transcript/metadata retain).
- AI query orchestration (retrieval-grounding + world knowledge toggle).

### Repository Layer
- Handles persistence only
- No business rules

Concrete responsibilities:
- PostgreSQL CRUD/query logic via SQLAlchemy 2.0.
- User-scoped query filters and indexes usage.
- Vector retrieval calls through infrastructure adapters.

### Infrastructure Layer
- DB connection
- Cache
- Message queues
- Third-party APIs

Concrete adapters:
- Instagram OAuth/API client.
- Fallback link parsing adapter.
- Transcription/LLM provider adapter.
- S3 object storage + signed URL adapter.
- SQS queue adapter.
- Billing adapter for Apple IAP verification.

---

# 4. Frontend Architecture

## 4.1 Structure


/src
/app
/components
/features
/hooks
/state
/services
/styles


### Rules:
- No API calls inside components
- All API calls in `/services`
- All global state in `/state`
- Components must be dumb/presentational when possible

iOS mapping:
- `apps/ios/SageReelAI/App/`
- `apps/ios/SageReelAI/Components/`
- `apps/ios/SageReelAI/Features/`
- `apps/ios/SageReelAI/State/`
- `apps/ios/SageReelAI/Services/`

Frontend constraints:
- Require auth before save actions.
- Folder tree supports one subfolder depth only.
- Cross-folder semantic search enabled.
- AI world-knowledge setting ON by default and user-toggleable.
- Blocked ingestion shows generic error only.

---

# 5. API Design Conventions

## 5.1 REST Standards

- `/api/v1/`
- Noun-based endpoints
- No verbs in routes
- Proper HTTP status codes

Required endpoints:
- `POST /api/v1/reels/ingest`
- `DELETE /api/v1/reels/{id}`
- `GET /api/v1/search`
- `POST /api/v1/ai/query`
- `POST /api/v1/learning-artifacts`
- `POST /api/v1/account/delete`
- `POST /api/v1/exports`
- `GET /api/v1/activity`

## 5.2 Validation

- All requests validated before reaching service layer
- Schema validation required

Concrete validation gates:
- Reel duration hard cap: 120 seconds.
- Plan quotas: storage, transcription minutes, AI query counts.
- Public-content only ingestion policy.

---

# 6. Database Design Standards

## 6.1 Conventions

- Snake_case tables
- UUID primary keys
- Timestamps required:
  - created_at
  - updated_at

Additional mandatory columns:
- `tenant_id` (logical isolation), `user_id` where applicable.
- Soft-delete not used for reel media deletion path; physical deletion of media object required.

Core tables:
- `users`, `subscriptions`, `folders`, `reels`, `transcripts`, `embedding_chunks`, `ai_queries`, `learning_artifacts`, `audit_events`, `deletion_requests`, `export_jobs`.

## 6.2 Indexing Strategy

- Index foreign keys
- Index frequently filtered columns
- Avoid premature over-indexing

Required indexes:
- `(user_id, created_at)` on `reels`, `transcripts`, `ai_queries`.
- `(folder_id, created_at)` on `reels`.
- Vector index for embedding retrieval.
- Partial indexes for active subscriptions and pending deletion/export jobs.

---

# 7. Configuration Management

- No environment variables accessed directly in business logic
- All config centralized in `/config`
- Strict typing of config values

FastAPI implementation:
- Use `pydantic-settings` `BaseSettings`.
- Cache settings object via `@lru_cache` dependency provider.
- Separate config profiles for dev/staging/prod.

---

# 8. Error Handling

- Centralized error handler
- No raw error leakage to client
- Structured error responses

Mandatory error taxonomy:
- `validation_error`
- `quota_exceeded`
- `ingestion_blocked`
- `provider_failure`
- `resource_not_found`
- `unauthorized`
- `forbidden`

Compliance block behavior:
- Return generic user-facing message without detailed legal reason.

---

# 9. Logging & Observability

- Structured JSON logs
- Correlation IDs
- Request tracing
- Error logging with stack traces
- Health check endpoint required

Required telemetry:
- p95 latencies: transcription, full analysis, AI response, vector retrieval.
- Queue depth, retry counts, dead-letter counts.
- Ingestion success/block/failure rates.
- Deletion SLA progress (30-day completion guarantee).

---

# 10. Testing Strategy

## 10.1 Unit Tests
- Service layer tested independently
- Repository mocked

Required backend unit coverage:
- Quota enforcement logic.
- Reel delete retention logic.
- Sensitive-category warning path.
- World-knowledge toggle behavior.

## 10.2 Integration Tests
- DB integration tests
- API endpoint tests

Required integration flows:
- Ingest -> transcribe -> index -> search -> AI query.
- Account deletion request -> immediate revocation -> purge completion.
- Export generation in CSV and TXT/Markdown.

## 10.3 Test Placement


/tests/unit
/tests/integration


Tooling:
- Backend: `pytest`, FastAPI `TestClient`.
- iOS: `XCTest`.

---

# 11. DevOps & Deployment

- Separate environments: dev, staging, production
- CI must:
  - Lint
  - Type-check
  - Run tests
- No direct deploys to production
- Feature flags for risky changes

MVP deployment target:
- AWS ECS Fargate for API and workers.
- AWS RDS PostgreSQL 16 (+ pgvector extension).
- AWS S3 private buckets for media.
- AWS SQS for job queue.

Release controls:
- Blue/green or rolling deploy with health gate.
- Migration gate required before app rollout.

---

# 12. Security Standards

- Rate limiting
- Input sanitization
- JWT validation
- Secure cookies
- CSRF protection
- Secrets never committed

Product-specific controls:
- TLS 1.2+ in transit.
- AES-256 at rest (DB, object storage, backups).
- Signed URLs for media access.
- Role-scoped service accounts and least privilege IAM.
- Immediate access revocation on account deletion request.
- Backup pipeline must honor immediate deletion semantics.

---

# 13. Performance Strategy

- Caching layer defined
- Lazy loading where possible
- Avoid N+1 queries
- Background job processing for heavy tasks

Explicit targets (p95, all users):
- Transcription completion <= 60s.
- Full reel analysis <= 90s.
- AI response <= 5s.
- Vector retrieval <= 200ms.

Capacity targets:
- 10,000 MAU (12 months).
- 30,000 reel ingestions/month.
- 25 global concurrent AI queries.

---

# 14. Scaling Plan

## 14.1 Horizontal Scaling

- Stateless API servers
- External session store

Execution plan:
- Scale API service by CPU/request latency.
- Scale workers by queue depth and stage-specific SLA.
- Keep per-user throttles and global concurrency guards for AI calls.

## 14.2 Database Scaling

- Read replicas
- Query optimization
- Partitioning if needed

Execution plan:
- Start with single writer + optional read replica.
- Add partitioning only when table size and pruneable predicates justify it.
- Review top query plans monthly and tune indexes with measured regressions.

---

# 15. Anti-Patterns (Prohibited)

- Business logic in controllers
- Cross-module circular imports
- Hardcoded secrets
- Global mutable state
- Direct DB queries inside controllers
- Silent error swallowing

Additional prohibited patterns for this product:
- Returning compliance-specific block reasons to end users.
- Bypassing user/tenant filters in repository queries.
- Synchronous inline transcription/analysis in API request path.
- Unbounded retries without dead-letter handling.

---

# 16. Architectural Trade-offs

Document:
- Why specific technologies were chosen
- What constraints influenced decisions
- What future refactors may be required

Trade-offs:
- FastAPI chosen for rapid API development and typed validation; trade-off is Python runtime overhead versus lower-level alternatives.
- Single AI provider reduces integration complexity; trade-off is vendor concentration risk.
- Retaining transcript/metadata after reel deletion preserves knowledge utility; trade-off is higher privacy/governance sensitivity and lifecycle complexity.
- Single-cloud AWS MVP improves delivery speed; trade-off is migration effort if multi-cloud is later required.
- One-level subfolders simplify UX/data model; trade-off is reduced information hierarchy flexibility.

---

# 17. Future Evolution

- Multi-tenant expansion
- Event-driven migration
- Microservice extraction plan
- AI/LLM integration plan (if applicable)

Planned evolutions:
- Add provider abstraction for private inference routing of sensitive categories.
- Introduce collaboration/shared folders after single-user MVP stabilization.
- Evaluate service extraction (`ingestion`, `retrieval`, `billing`) after load or team-size triggers.
- Add configurable retention/deletion controls for retained transcript/metadata.
