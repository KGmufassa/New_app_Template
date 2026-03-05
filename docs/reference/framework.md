# CUSTOM DEVELOPMENT FRAMEWORK

## Folder Structure
- `apps/ios/` - iOS client (SwiftUI, local cache, auth/session, search UI, AI chat UI).
- `services/api/` - versioned backend API (`/api/v1`), presentation layer only.
- `services/worker/` - async jobs (ingestion, transcription, embedding, analysis, retry/backoff).
- `features/` - feature modules, each with `service`, `repository`, `model`, `validation`, `types`, `tests`:
  - `features/auth/`
  - `features/instagram_ingestion/`
  - `features/folders/`
  - `features/reels/`
  - `features/transcripts/`
  - `features/search/`
  - `features/ai_insights/`
  - `features/billing/`
  - `features/account_lifecycle/` (deletion, export)
  - `features/audit/`
- `core/` - shared utilities (errors, logging, telemetry, pagination, retry policies, shared DTOs).
- `config/` - typed config only (env mapping, provider config, per-environment settings).
- `infrastructure/` - cloud integrations (db, object storage, queues, vector store, provider clients), no business logic.
- `migrations/` - schema migrations + rollback artifacts.
- `docs/reference/` - architecture reference docs and operating constraints.

## Required Modules
- `Authentication` - Apple, Google, email/password; account binding rules; session management.
- `Tenant Isolation` - strict logical isolation and user-scoped filtering across all reads/writes.
- `Instagram Ingestion` - OAuth-based ingestion; fallback link parsing; compliance gate that blocks low-confidence cases.
- `Media Storage` - private object storage for videos, signed URL access, retention-by-user deletion.
- `Transcription & Analysis Pipeline` - async orchestration with retries, idempotency, and status tracking.
- `Multimodal Retrieval` - embeddings, vector retrieval, cross-folder semantic ranking with secondary recency/activity signals.
- `AI Answering` - folder-grounded retrieval, optional world knowledge toggle, sensitive-content warning path.
- `Learning Outputs` - on-demand bullet notes, outlines, flashcards.
- `Billing` - freemium quota enforcement and Apple IAP subscriptions.
- `Account Lifecycle` - immediate access revocation + 30-day purge orchestration + export generation (CSV, TXT/Markdown).
- `Audit Logging` - internal security/admin logs and user-visible activity logs.

## Infrastructure Requirements
- Single-cloud MVP with managed services preferred.
- Managed relational database (UUID PKs, FK indexes, `created_at`/`updated_at`, migration + rollback discipline).
- Private object storage for media with signed URL retrieval.
- Queue + worker system for asynchronous ingestion/transcription/analysis.
- Vector store for semantic retrieval (p95 retrieval target <= 200 ms).
- Single AI provider for MVP (LLM + transcription path), provider abstraction at infrastructure boundary.
- Backup system with encrypted backups and delete semantics aligned with immediate deletion requirements.
- API gateway/rate limiting layer with `/api/v1` versioning.

## Security Requirements
- TLS 1.2+ for all transport.
- AES-256 encryption at rest for database, objects, and backups.
- Role-scoped service accounts and least-privilege access policies.
- Signed URLs for controlled media access.
- No direct env access outside `config/`; typed config for all secrets and endpoints.
- Sensitive categories handled with warning path when third-party fallback is used: health, finance, legal, minors, biometric, sexual content.
- Immediate account lockout on deletion request; full purge completion within 30 days.
- Audit coverage:
  - Internal: admin actions, authentication events, security alerts.
  - User-visible: login history, password changes, deletion requests.

## Scaling Strategy
- Design baseline: 10,000 MAU, 30,000 ingestions/month, 25 global concurrent AI queries.
- Capacity planning centered on AI/worker bottlenecks, not controller throughput.
- Enforce quotas at API edge and service layer:
  - Free: 0.5 GB storage, 60 transcription minutes/month, 75 AI queries/month.
  - Paid: 5x free limits.
- Hard reel duration cap: 120 seconds for all users.
- Asynchronous first for ingestion and analysis; retries 2-3 attempts with backoff.
- SLO targets (p95, all users):
  - Transcription <= 60s
  - Full analysis <= 90s
  - AI response <= 4-5s
  - Vector retrieval <= 200ms
- Reliability targets: 99.5% uptime, RPO <= 5m, RTO <= 1h.

## Observability Standards
- Structured JSON logs only.
- Correlation ID propagated across API, worker, AI provider, and storage operations.
- Centralized error handling middleware; explicit client-safe error responses only.
- Health, readiness, and liveness probes mandatory.
- Metrics required:
  - Ingestion success/failure and compliance-block rates.
  - Queue depth, retry counts, and processing latency by stage.
  - AI response latency, retrieval latency, token/usage and quota burn.
  - Deletion pipeline SLA tracking and export job latency.
- Alerts required for SLO breach trends, auth anomaly spikes, and sustained queue backlog.

## Governance Rules
- Enforce layered architecture: Presentation -> Service -> Repository -> Infrastructure.
- No business logic in controllers/routes or infrastructure adapters.
- Services can call repositories; repositories cannot call services.
- No cross-feature direct imports; shared utilities live in `core/`.
- All feature modules must be independently testable.
- Input validation before service invocation.
- All schema changes via migrations with documented rollback.
- Service-layer unit tests and integration tests for critical flows are mandatory.
- Architecture revalidation required after major feature additions.

## Prohibited Patterns
- Business logic in controllers/route handlers.
- Direct database access from presentation layer.
- Global mutable state for tenant/user context.
- Silent error swallowing or non-structured errors.
- Tight coupling between feature modules.
- Hardcoded secrets or untyped ad hoc configuration.
- Bypassing tenant/user scoping filters in any query path.
- Processing or ingesting content when compliance confidence is low.
