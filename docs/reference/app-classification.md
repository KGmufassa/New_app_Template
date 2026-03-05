# APPLICATION CLASSIFICATION

## System Type
- AI-native consumer mobile SaaS (iOS-first), freemium/subscription.
- Multimodal knowledge app for saving reels, transcription, semantic search, and AI insights.
- Single-cloud MVP deployment with managed backend services.

## Tenant Model
- Multi-tenant with strict logical isolation.
- Single-user accounts in MVP (no sharing/collaboration roles).

## Scale Profile
- 10k+ profile at 12 months.
- Target: 10,000 MAU.
- Target ingestion: 30,000 reels/month.
- Peak AI concurrency target: 25 global concurrent queries.

## Data Sensitivity
- Moderate to high sensitivity due to stored media, transcripts, and user account data.
- Sensitive categories: health, finance, legal, minors, biometric, sexual content.

## Compliance Tier
- U.S.-only MVP.
- Privacy baseline with account deletion support (immediate access revocation, full purge SLA 30 days).
- Backup deletion must honor immediate deletion semantics.
- Data export required: TXT/Markdown and CSV.
- Audit logging required (internal admin/auth/security events + user-visible login/password/account deletion request history).

## Real-Time Requirements
- Asynchronous processing acceptable.
- p95 targets for all users:
  - Transcription completion <= 60s
  - Full reel analysis <= 90s
  - AI response latency <= 4-5s
  - Vector retrieval <= 200ms

## AI Involvement
- Core product dependency (multimodal retrieval + text Q&A + semantic search).
- Retrieval grounded per folder; cross-folder semantic search enabled.
- World knowledge ON by default, user can disable in settings.
- Single AI provider in MVP; architecture should allow future private inference routing.

## External Integrations
- Instagram API with OAuth required in v1.
- Link parser fallback if API unavailable; block ingestion when legal/compliance confidence is low.
- Transcription provider.
- LLM provider (single vendor for MVP).
- Vector database.
- Authentication: Apple Sign-In, Google Sign-In, email/password.
- Billing: Apple In-App Purchase on iOS; potential web subscription management later.

## Risk Level
- High architectural risk due to external platform constraints (Instagram policy), AI safety handling, and strict deletion/privacy requirements.
