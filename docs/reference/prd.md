# PRODUCT REQUIREMENTS DOCUMENT (TECHNICAL BUILD VERSION)

## 1. Document Control
- Product Name: Sage Reel AI (working name)
- Version: v1.0 (MVP)
- Status: Draft - Ready for implementation planning
- Author: OpenCode (AI assistant)
- Engineering Lead: TBD
- Last Updated: 2026-03-05
- Target Release: TBD (MVP)

## 2. System Overview
### 2.1 Product Summary
Sage Reel AI is an iOS-first consumer app that lets users save public Instagram reels into custom folders (one subfolder level), transcribe and analyze saved content, and query a multimodal knowledge base with semantic search and AI Q&A.

### 2.2 Problem Definition
Creators, students, and marketers save short-form video content but cannot reliably retrieve insights across large collections. Existing bookmark flows are weak for structured learning and synthesis.

### 2.3 System Scope
#### In Scope (MVP)
- Consumer single-user accounts with Apple Sign-In, Google Sign-In, and email/password.
- Instagram OAuth ingestion for public reels; fallback link parsing when API is unavailable.
- Folder + single-level subfolder organization.
- Video storage, transcript generation, vector indexing, cross-folder semantic search.
- AI Q&A with folder-grounded retrieval; world knowledge enabled by default and user-toggleable.
- On-demand generation of bullet notes, outlines, and flashcards.
- Freemium + Apple IAP subscriptions with enforced quotas.
- Data export (TXT/Markdown, CSV), account deletion workflow, audit logs.

#### Out of Scope
- Team collaboration, shared folders, and role-based end-user permissions.
- Offline mode.
- Private/self-hosted inference in MVP.
- Regional deployments outside U.S.
- Detailed citation UI in cross-folder search results.

## 3. Functional Requirements
Use numbered format:
FR-1: The system shall require authenticated sign-in before first save and support Apple, Google, and email/password providers.
FR-2: The system shall enforce one subscription per app account with no cross-account subscription sharing.
FR-3: The system shall support creation of main folders and one level of subfolders only.
FR-4: The system shall ingest public Instagram reels via Instagram OAuth integration.
FR-5: If Instagram API ingestion is unavailable, the system shall attempt link parsing fallback.
FR-6: If legal/compliance confidence is low during ingestion, the system shall block ingestion and return a fully generic error message.
FR-7: The system shall store downloaded reel media in private object storage until user deletion or account deletion.
FR-8: The system shall transcribe each ingested reel and persist transcript text and metadata.
FR-9: The system shall generate multimodal analysis artifacts (audio/frame-derived features) for retrieval.
FR-10: The system shall index content for semantic retrieval across all user folders and subfolders.
FR-11: Cross-folder search ranking shall use semantic relevance as primary signal, with recency and user activity as secondary signals.
FR-12: The system shall provide AI Q&A over user content with retrieval grounding on selected folder scope.
FR-13: The system shall allow AI answers to use world knowledge by default, with a user setting to disable world knowledge.
FR-14: The system shall generate bullet notes, outlines, and flashcards on demand only.
FR-15: The system shall apply freemium limits: 0.5 GB storage, 60 transcription minutes/month, 75 AI queries/month.
FR-16: The paid tier shall provide 5x freemium limits: 2.5 GB storage, 300 transcription minutes/month, 375 AI queries/month.
FR-17: The system shall enforce a hard reel duration cap of 120 seconds for all users.
FR-18: Deleting a reel shall remove media immediately but retain transcript + metadata for search/AI until account deletion.
FR-19: In MVP, users shall not be able to manually delete retained transcript/metadata independently of account deletion.
FR-20: Account deletion shall revoke access immediately and complete backend purge within 30 days.
FR-21: Backup handling shall honor immediate deletion semantics for deleted user data.
FR-22: The system shall provide user export of transcripts/notes/metadata in TXT/Markdown and CSV.
FR-23: The system shall record internal audit logs for admin actions, auth events, and security alerts.
FR-24: The system shall provide user-visible activity logs for login history, password changes, and account deletion requests.
FR-25: Sensitive-category detection (health, finance, legal, minors, biometric, sexual content) shall allow third-party AI fallback with a generic warning in MVP.

Include:
- Authentication & Identity: FR-1, FR-2
- Core Domain Features: FR-3 through FR-14, FR-17 through FR-19
- Data Operations: FR-7 through FR-11, FR-18 through FR-24
- External Integrations: FR-4, FR-5, FR-6, FR-16, FR-25

## 4. Non-Functional Requirements
### Performance
- p95 transcription completion <= 60 seconds.
- p95 full reel analysis <= 90 seconds.
- p95 AI response latency <= 5 seconds.
- p95 vector retrieval <= 200 ms.

### Scalability
- Support 10,000 MAU within first 12 months.
- Support 30,000 reel ingestions per month.
- Support 25 global concurrent AI queries.

### Reliability
- Service availability target: 99.5% uptime.
- RPO <= 5 minutes.
- RTO <= 1 hour.
- Background retry policy: 2-3 attempts with backoff.

### Security
- TLS 1.2+ for all network traffic.
- AES-256 encryption at rest for DB, object storage, and backups.
- Private object storage for media and signed URLs for access.
- Role-scoped service accounts, least-privilege IAM.
- User-scoped data filtering enforced at service and repository layers.

### Compliance
- U.S.-only deployment boundary for MVP.
- Minimum user age: 13+.
- Account deletion with immediate access revocation and 30-day purge SLA.
- No model training on user data in MVP.

## 5. System Architecture
### High-Level Architecture
- Layered architecture enforced: Presentation -> Service -> Repository -> Infrastructure.
- iOS app -> API service -> domain services -> repositories -> DB/object/vector stores.
- Async worker pipeline handles ingestion, transcription, embedding, analysis, and retries.
- No business logic in controllers or infrastructure adapters.

### Technology Stack
- Client: iOS (Swift/SwiftUI).
- API: versioned REST (`/api/v1`) in a typed backend framework.
- DB: PostgreSQL-compatible relational database with migrations.
- Storage: private object storage with signed URL access.
- Search: vector database/store for semantic retrieval.
- AI: single provider for LLM + transcription in MVP.
- Queue/Workers: managed queue + worker runtime.
- Auth: Apple, Google, email/password.
- Billing: Apple In-App Purchases.

### Data Model Overview
- Core data objects: User, Subscription, Folder, Reel, Transcript, EmbeddingChunk, AIQuery, LearningArtifact, AuditEvent, DeletionRequest, ExportJob.
- UUID primary keys, `created_at`/`updated_at` on all mutable entities.

### API Design
- REST endpoints under `/api/v1`.
- Request validation before service layer execution.
- Structured error responses only; no stack traces to clients.

### Deployment Strategy
- Single-cloud MVP using managed DB, object storage, queue, and observability stack.
- Separate environments: dev, staging, prod.
- Infrastructure must support future provider abstraction for private inference.

## 6. Data Model Design
### Core Entities
- `users`: identity, auth provider links, age gate, status.
- `subscriptions`: plan, quota counters, renewal state.
- `folders`: top-level folders and one-level subfolders.
- `reels`: source URL, provider IDs, media metadata, ingestion status.
- `transcripts`: transcript text, timestamps, language, quality markers.
- `embedding_chunks`: chunk text/vector refs and retrieval metadata.
- `ai_queries`: prompt, scope, settings (world knowledge on/off), latency, outcome.
- `learning_artifacts`: on-demand bullet notes/outlines/flashcards.
- `audit_events`: internal and user-visible activity log entries.
- `deletion_requests`: lifecycle state, revocation timestamp, completion timestamp.
- `export_jobs`: format, status, completion metadata.

### Relationships
- User 1:N Folder, Reel, AIQuery, LearningArtifact, AuditEvent, ExportJob.
- Folder 1:N Reel; Folder 1:N Subfolder (single depth).
- Reel 1:1 Transcript; Transcript 1:N EmbeddingChunk.
- User 1:N SubscriptionHistory (or 1:1 active subscription + history table).
- User 1:N DeletionRequest (latest active request unique constraint).

### Migration Plan
- All schema changes via forward-only migrations with rollback scripts.
- Enforce FK indexes, UUID defaults, and audit timestamps in initial baseline migration.
- Add partial indexes for user-scoped retrieval and quota enforcement queries.

## 7. API Design
### API Paradigm
- Resource-oriented REST with JSON payloads and bearer-token auth.

### Endpoint Specifications
- `POST /api/v1/auth/{provider}/callback` - authenticate user.
- `POST /api/v1/folders` / `POST /api/v1/folders/{id}/subfolders` - manage hierarchy.
- `POST /api/v1/reels/ingest` - ingest by OAuth context or fallback link.
- `DELETE /api/v1/reels/{id}` - delete media, retain transcript/metadata.
- `GET /api/v1/search` - cross-folder semantic search.
- `POST /api/v1/ai/query` - grounded Q&A with optional world-knowledge flag.
- `POST /api/v1/learning-artifacts` - on-demand notes/outline/flashcards generation.
- `GET /api/v1/usage` - quota status.
- `POST /api/v1/account/delete` - initiate deletion workflow.
- `POST /api/v1/exports` / `GET /api/v1/exports/{id}` - export operations.
- `GET /api/v1/activity` - user-visible activity log.

### Rate Limiting
- Per-user limits on ingestion, search, and AI query endpoints.
- Hard monthly quota enforcement for transcription minutes and AI queries.
- Distinct paid-tier multipliers applied server-side.

## 8. State Management & Event Flow
- Ingestion flow states: `requested -> validating -> queued -> processing -> transcribed -> indexed -> ready`.
- Failure states: `blocked_compliance`, `provider_error`, `retry_exhausted`.
- AI query flow: request validation -> retrieval -> prompt assembly -> model response -> post-processing -> persist.
- Account deletion flow: `requested -> access_revoked -> purge_in_progress -> purge_completed`.

## 9. Background Processing
- Queue workers execute ingestion, transcription, embedding, and analysis jobs asynchronously.
- Jobs must be idempotent (dedupe key: user + reel source ID/hash).
- Retry strategy: exponential backoff, max 3 attempts.
- Dead-letter queue required for failed jobs with operator visibility.
- Purge and export jobs run in background with progress status API.

## 10. Frontend Requirements
- iOS app must require authentication before save actions.
- UX optimized first for creators; supports students and marketers.
- Folder UI supports top-level + one subfolder depth only.
- Search UI supports cross-folder semantic queries.
- AI UI includes toggle for world knowledge usage.
- Learning artifact UI supports on-demand bullet notes, outlines, flashcards.
- Billing UI surfaces free vs paid quotas and Apple IAP upgrade flow.
- Compliance block messaging must be generic only.

## 11. DevOps & Deployment
- CI/CD pipeline with lint, tests, migration checks, and environment promotion gates.
- Environment isolation: dev/staging/prod with typed config per environment.
- Secrets in managed secret store only; never committed.
- Health endpoints: liveness/readiness required for services and workers.
- Backups encrypted and verified; restore drills required to validate RTO/RPO targets.

## 12. Observability
- Structured JSON logs with correlation IDs across request and job lifecycles.
- Metrics: ingestion success/failure, queue depth, job latency, AI latency, retrieval latency, quota burn.
- Tracing across API -> worker -> provider calls.
- Alerting for SLO breaches, queue backlog, auth anomalies, and purge SLA breach risk.

## 13. Analytics & Instrumentation
- Track activation funnel: signup -> first ingest -> first AI query -> first artifact generation.
- Track persona usage patterns (creator/student/marketer self-selection if collected).
- Track retention events: weekly active users, repeat search, repeat AI query.
- Track monetization: upgrade conversion, quota exhaustion triggers, churn signals.
- Privacy: analytics must avoid storing raw sensitive transcript fragments unless required and approved.

## 14. Risk Assessment
- Platform dependency risk: Instagram API policy/access constraints.
- Legal/compliance risk: public-content ingestion boundaries and copyright concerns.
- AI quality risk: hallucinations when world knowledge is enabled.
- Cost risk: AI/transcription spend under heavy usage.
- Data lifecycle risk: retained transcript/metadata after reel deletion may create user expectation mismatch.

## 15. Milestones & Implementation Phases
- Phase 1: Foundation (auth, folder model, base API, DB, storage, config, observability).
- Phase 2: Ingestion pipeline (OAuth ingestion, fallback parsing, compliance gate, queue workers).
- Phase 3: AI retrieval (transcription, embeddings, cross-folder semantic search, Q&A).
- Phase 4: Productization (learning artifacts, quotas, billing, export/deletion, activity logs).
- Phase 5: Hardening (SLO tuning, security review, load tests, App Store readiness).

## 16. Assumptions
- Cloud provider and exact managed services are not yet selected.
- Single AI vendor can satisfy both transcription and LLM latency/quality targets.
- Instagram OAuth access approval is attainable for MVP timeline.
- U.S.-only boundary is sufficient for initial legal/compliance posture.
- Engineering lead and release date are pending organizational assignment.

## 17. Open Questions
- Which concrete cloud stack (compute, DB, queue, object, vector) is preferred for implementation?
- Should future paid tiers introduce higher reel duration caps beyond 120 seconds?
- Should retained transcript/metadata deletion controls be added before GA to reduce privacy risk?
