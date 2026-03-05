# APPLICATION BUILD PLAN

## Infrastructure Phase
- Establish monorepo structure per framework: `apps/ios`, `services/api`, `services/worker`, `features/*`, `core`, `config`, `infrastructure`, `migrations`.
- Provision managed single-cloud MVP stack: PostgreSQL, private object storage, queue/worker runtime, vector store, observability suite, secrets manager.
- Enforce environment isolation (`dev`, `staging`, `prod`) with typed config in `config/` and no direct env access outside config loaders.
- Implement shared `core/` primitives first: structured errors, correlation ID middleware, logging, telemetry, pagination, retry policy, quota primitives, tenant scope guards.
- Stand up API gateway and `/api/v1` routing conventions, auth middleware, and per-user rate-limit middleware.
- Prepare migration baseline with UUID PKs, `created_at`/`updated_at`, FK indexes, partial indexes for user-scoped retrieval, and rollback artifacts.

## Core Module Setup
- `features/auth`: provider auth, account identity binding, sessions, login activity.
- `features/folders`: folder + single-level subfolder hierarchy rules.
- `features/instagram_ingestion`: OAuth ingestion, fallback link parsing, compliance gate.
- `features/reels`: reel metadata lifecycle, media deletion semantics, ingestion status.
- `features/transcripts`: transcript persistence, chunking metadata, quality markers.
- `features/search`: embedding index metadata, semantic search ranking orchestration.
- `features/ai_insights`: grounded Q&A, world-knowledge toggle, sensitive-content warning path, learning artifacts.
- `features/billing`: freemium/paid plans, quota accounting, Apple IAP integration.
- `features/account_lifecycle`: exports, deletion request workflow, purge orchestration.
- `features/audit`: internal audit events and user-visible activity events.

## Feature-by-Feature Breakdown

### 1) Authentication & Identity (FR-1, FR-2)
- Module scope: Apple/Google/email-password authentication, account linking constraints, single subscription ownership per app account, session issuance/revocation.
- Required schema: `users`, `auth_identities`, `sessions`, `password_credentials`, `user_profile`, `auth_events`.
- Service layer logic: provider callback exchange, identity dedupe/linking, minimum-age gate check (13+), session mint/refresh/revoke, login anomaly triggers.
- Repository structure: `users.repository`, `authIdentities.repository`, `sessions.repository`, `authEvents.repository` with strict `user_id` scope.
- Validation rules: provider callback payload validation, email/password complexity, nonce/state verification, duplicate identity prevention.
- API endpoints: `POST /api/v1/auth/{provider}/callback`, `POST /api/v1/auth/email/register`, `POST /api/v1/auth/email/login`, `POST /api/v1/auth/logout`, `GET /api/v1/activity` (auth slice).
- Test requirements: unit tests for identity binding and session rules; integration tests for each provider callback; negative tests for replay, invalid state, underage rejection.
- Security checks: OAuth state/nonce, hashed credentials, TLS-only session transport, least-privilege provider credentials, structured generic auth errors.
- Observability hooks: auth success/failure counters, anomaly rates, callback latency histograms, correlation IDs across auth provider calls.

### 2) Folder Hierarchy (FR-3)
- Module scope: create/update/archive top-level folders and one-level subfolders; prohibit deeper nesting.
- Required schema: `folders` (`id`, `user_id`, `parent_folder_id`, `name`, timestamps) with constraint disallowing depth > 1.
- Service layer logic: enforce ownership, parent existence, depth cap, name uniqueness per sibling scope, soft archive behavior if needed.
- Repository structure: `folders.repository` with helpers `listRoot`, `listChildren`, `createRoot`, `createChild`.
- Validation rules: folder name length/charset, parent ownership, disallow parent that itself has parent.
- API endpoints: `POST /api/v1/folders`, `POST /api/v1/folders/{id}/subfolders`, `GET /api/v1/folders`, `PATCH /api/v1/folders/{id}`.
- Test requirements: unit tests for depth enforcement; integration tests for hierarchy reads/writes and tenant isolation.
- Security checks: mandatory user scoping in every query; no cross-user parent references.
- Observability hooks: folder create/update counters, hierarchy validation failure metrics.

### 3) Instagram Ingestion (FR-4, FR-5, FR-6, FR-17)
- Module scope: ingest public reels using OAuth context, fallback link parse path, compliance confidence gate, duration cap enforcement.
- Required schema: `reels` (source URL/provider IDs/status/duration), `ingestion_jobs`, `compliance_checks`, `provider_events`.
- Service layer logic: validate source, compute idempotency key (user + source hash), compliance evaluation, route to queue, fallback parser on provider unavailability, block low-confidence with generic error.
- Repository structure: `reels.repository`, `ingestionJobs.repository`, `compliance.repository`, `providerEvents.repository`.
- Validation rules: URL format + host allowlist, public availability, max duration <= 120s, duplicate ingestion dedupe.
- API endpoints: `POST /api/v1/reels/ingest`, `GET /api/v1/reels/{id}` (status), optional `GET /api/v1/reels` listing.
- Test requirements: unit tests for compliance gating and dedupe; integration tests for OAuth success, fallback activation, blocked-compliance path, duration rejection.
- Security checks: provider token protection, generic compliance block messaging, anti-abuse rate limits on ingest endpoint.
- Observability hooks: ingestion requested/queued/succeeded/failed/block rates, fallback usage rate, duration-cap rejection rate, queue latency by stage.

### 4) Media Storage & Reel Lifecycle (FR-7, FR-18, FR-19)
- Module scope: private media persistence, signed URL access, delete media immediately while retaining transcript/metadata until account deletion.
- Required schema: `reels` storage refs, `media_objects`, `deletion_markers`.
- Service layer logic: upload/store metadata, signed URL issuance, media delete transaction with retention flags, prevent manual transcript-only deletion in MVP.
- Repository structure: `reels.repository`, `mediaObjects.repository`, `retention.repository`.
- Validation rules: media type/size checks, storage reference integrity, deletion eligibility checks.
- API endpoints: `DELETE /api/v1/reels/{id}`, `GET /api/v1/reels/{id}/media-url` (signed).
- Test requirements: integration tests for immediate media deletion + retained transcript discoverability; policy tests blocking transcript-only delete.
- Security checks: private buckets only, short-lived signed URLs, ownership checks before URL issuance/deletion.
- Observability hooks: media storage bytes per user, signed URL generation latency/errors, delete lifecycle events.

### 5) Transcription & Multimodal Analysis Pipeline (FR-8, FR-9)
- Module scope: async transcription, frame/audio feature extraction, status tracking with retries and DLQ.
- Required schema: `transcripts`, `analysis_artifacts`, `pipeline_jobs`, `job_attempts`.
- Service layer logic: enqueue jobs, idempotent worker execution, stage transitions (`processing -> transcribed -> indexed -> ready`), retry/backoff max 3, DLQ routing.
- Repository structure: `transcripts.repository`, `analysis.repository`, `pipelineJobs.repository`.
- Validation rules: transcript language whitelist, payload schema for worker stages, state transition guards.
- API endpoints: internal worker callbacks/events; external `GET /api/v1/reels/{id}` includes processing status.
- Test requirements: worker unit tests for idempotency and transition rules; integration tests for retry exhaustion and DLQ behavior.
- Security checks: encrypted artifacts at rest, scoped worker credentials, no transcript leakage in errors/logs.
- Observability hooks: transcription p95, analysis p95, retry counts, DLQ depth, stage-level failure rates.

### 6) Semantic Search & Retrieval (FR-10, FR-11)
- Module scope: embedding chunk indexing and cross-folder semantic retrieval with secondary recency/activity weighting.
- Required schema: `embedding_chunks`, `search_events`, `retrieval_features`.
- Service layer logic: chunk transcript/artifacts, vector upsert, retrieval top-k, rerank with recency/user activity secondary signals.
- Repository structure: `embeddingChunks.repository`, `searchEvents.repository`, `retrieval.repository` (vector adapter).
- Validation rules: query length bounds, folder scope ownership, pagination limits.
- API endpoints: `GET /api/v1/search`.
- Test requirements: relevance/ranking unit tests, integration tests for user-scoped cross-folder queries, latency tests to protect p95 <= 200ms retrieval target.
- Security checks: strict tenant filtering in candidate selection and post-filter, prompt-injection-safe query handling.
- Observability hooks: retrieval latency histogram, hit rate, zero-result rate, rerank feature distributions.

### 7) AI Q&A & Learning Outputs (FR-12, FR-13, FR-14, FR-25)
- Module scope: grounded AI Q&A, world-knowledge toggle (default on), sensitive-category fallback warning, on-demand notes/outlines/flashcards.
- Required schema: `ai_queries`, `learning_artifacts`, `safety_events`, `prompt_contexts`.
- Service layer logic: validate scope, retrieve context, assemble prompt, call model, apply safety checks and sensitive warning path, persist response metadata and latency.
- Repository structure: `aiQueries.repository`, `learningArtifacts.repository`, `safetyEvents.repository`.
- Validation rules: artifact type enum, query token/length limits, toggle boolean defaults, scope ownership.
- API endpoints: `POST /api/v1/ai/query`, `POST /api/v1/learning-artifacts`.
- Test requirements: unit tests for world-knowledge toggle and artifact generation modes; integration tests for grounded responses, sensitive-category warning behavior, quota interactions.
- Security checks: output filtering, sensitive-category fallback disclosure, no training-on-user-data control flags in provider config.
- Observability hooks: AI latency p95, token usage, answer failure rate, sensitive-category trigger rate, artifact generation latency.

### 8) Billing & Quotas (FR-15, FR-16)
- Module scope: freemium and paid tier limits, Apple IAP validation, hard enforcement on storage/transcription/AI quotas.
- Required schema: `subscriptions`, `subscription_events`, `usage_counters`, `quota_resets`, `iap_receipts`.
- Service layer logic: entitlement resolution, receipt verification, rolling monthly quota accounting, preflight checks at API edge + service layer.
- Repository structure: `subscriptions.repository`, `usageCounters.repository`, `receipts.repository`.
- Validation rules: plan enum, non-negative counters, receipt signature/transaction validation, single active subscription/account.
- API endpoints: `GET /api/v1/usage`, `POST /api/v1/billing/iap/validate`.
- Test requirements: unit tests for multiplier math (5x paid), quota reset windows, rejection on overages; integration tests for receipt lifecycle.
- Security checks: signed receipt verification, idempotent receipt processing, anti-fraud audit trails.
- Observability hooks: quota burn metrics, overage rejection rates, upgrade conversion funnel, receipt failure rates.

### 9) Account Lifecycle: Export + Deletion (FR-20, FR-21, FR-22)
- Module scope: user export jobs, deletion initiation with immediate access revocation, 30-day purge orchestration including backups semantics.
- Required schema: `deletion_requests`, `export_jobs`, `purge_tasks`, `backup_tombstones`.
- Service layer logic: create deletion request, revoke sessions instantly, schedule purge pipeline, verify completion SLA; generate TXT/Markdown + CSV export bundles.
- Repository structure: `deletionRequests.repository`, `exportJobs.repository`, `purgeTasks.repository`.
- Validation rules: single active deletion request/user, export format enum, state-machine transition guards.
- API endpoints: `POST /api/v1/account/delete`, `POST /api/v1/exports`, `GET /api/v1/exports/{id}`.
- Test requirements: integration tests for immediate lockout, purge completion deadlines, export format correctness and authorization.
- Security checks: irreversible action confirmation requirements, purge worker least privilege, backup deletion/tombstone enforcement.
- Observability hooks: deletion SLA countdown metrics, purge backlog, export success/failure latency, lockout event telemetry.

### 10) Audit Logging & User Activity (FR-23, FR-24)
- Module scope: internal audit (admin/auth/security) and user-visible account activity timelines.
- Required schema: `audit_events`, `user_activity_events`, immutable append-only constraints.
- Service layer logic: centralized event emitters from auth, billing, security, deletion, admin actions; user activity projection filters.
- Repository structure: `auditEvents.repository`, `activityEvents.repository`.
- Validation rules: event type enums, actor metadata requirements, redaction of sensitive payload fields.
- API endpoints: `GET /api/v1/activity`.
- Test requirements: unit tests for event taxonomy mapping and redaction; integration tests for event completeness across critical flows.
- Security checks: append-only access policies, restricted internal audit read paths, no PII overexposure in user-visible feed.
- Observability hooks: event ingestion throughput, missing-event detector alerts, auth anomaly and security-alert counters.

## Cross-Cutting Concerns
- Architecture governance: enforce Presentation -> Service -> Repository -> Infrastructure; block direct DB access in controllers and cross-feature direct imports.
- Tenant isolation: every repository method accepts `user_id` context and applies scoped filters by default; add lint/static checks for unscoped queries.
- Error handling: structured JSON errors only, generic client-safe messages for compliance and sensitive failure paths.
- Idempotency and retries: ingestion/transcription/analysis/deletion jobs use deterministic dedupe keys and bounded exponential backoff.
- Config hygiene: all secrets/endpoints typed in `config/`; no hardcoded credentials.

## Performance Strategy
- SLOs: transcription p95 <= 60s, analysis p95 <= 90s, AI response p95 <= 5s, retrieval p95 <= 200ms.
- Optimize worker throughput first: queue partitioning by stage, concurrency tuning, hot-path profiling around provider calls.
- Retrieval latency plan: precomputed embeddings, vector index tuning, lightweight rerank features cached by user/activity.
- API latency plan: auth/session caches, quota prechecks, request validation short-circuiting before service invocation.
- Capacity plan aligned to 10k MAU, 30k ingestions/month, 25 concurrent AI queries with periodic load tests.

## Security Enforcement
- Transport/storage controls: TLS 1.2+, AES-256 at rest for DB/object/backups, private object storage with short TTL signed URLs.
- Identity and access: least-privilege service accounts, role-scoped tokens, strict separation of internal/admin and user-facing APIs.
- Compliance protections: block low-confidence ingestion, generic block message, sensitive-category warning path when third-party fallback is used.
- Data lifecycle controls: immediate access revocation on deletion request, purge completion within 30 days, backups honoring deletion semantics.
- Security telemetry: auth anomalies, unusual ingestion patterns, quota abuse, and purge SLA risk alerts.

## Validation Checklist
- Architecture checks: no business logic in presentation/infrastructure; repositories never call services; no cross-feature imports.
- Schema checks: all changes via forward migrations + rollback scripts; UUID PKs, indexed FKs, user-scope indexes present.
- API checks: endpoint validation runs before service invocation; structured errors only; per-user rate limiting enabled.
- Test gates: mandatory service-layer unit tests and integration tests for auth, ingestion, search, AI, billing, deletion, exports, audit.
- Reliability checks: retries bounded (max 3), DLQ configured, health/readiness/liveness probes passing.
- Observability checks: correlation IDs end-to-end, required metrics/dashboard coverage, SLO and anomaly alerts active.

## Deployment Strategy
- Release sequence:
  1. Foundation deploy: auth, folders, shared core, DB/storage/queue/vector infrastructure, baseline observability.
  2. Ingestion pipeline deploy: OAuth ingestion, fallback parser, compliance gate, async transcription/analysis.
  3. Retrieval + AI deploy: embeddings, search endpoint, grounded Q&A, learning artifacts.
  4. Productization deploy: billing/quotas, exports/deletion, activity logs, hardening.
- CI/CD requirements: lint, unit/integration tests, migration dry-runs, security scans, environment promotion gates.
- Runtime requirements: blue/green or canary for API, staged worker concurrency ramp, automatic rollback on SLO breach trends.
- Operational readiness: backup restore drills, incident runbooks for queue backlog/provider outage, App Store release checklist.
