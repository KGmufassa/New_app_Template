# ARCHITECTURE GOVERNANCE – BASE RULES

These rules apply to ALL applications regardless of system type.

They cannot be overridden by rule packs.

---

# 1. Layered Architecture Enforcement

Applications must follow strict layering:

Presentation Layer
→ Service Layer
→ Repository Layer
→ Infrastructure Layer

Rules:

- No business logic in controllers or route handlers.
- Services may call repositories.
- Repositories may not call services.
- Infrastructure may not contain business logic.
- No circular dependencies between modules.

---

# 2. Feature-Based Modularity

All domain features must be isolated.

Each feature must contain:

- service
- repository
- schema/model
- validation
- types

Rules:

- No cross-feature direct imports.
- Shared logic must live in /core.
- Features must be independently testable.

---

# 3. Configuration Management

- No direct access to environment variables outside /config.
- Config must be strongly typed.
- Secrets must never be committed.
- Separate configs for dev, staging, prod.

---

# 4. Error Handling Standards

- Centralized error middleware required.
- Structured error format enforced.
- No raw stack traces exposed to clients.
- All errors must be logged.

---

# 5. Logging & Observability

- Structured JSON logging required.
- Correlation ID per request.
- Health check endpoint required.
- Application must expose readiness & liveness probes.

---

# 6. Database Standards

- UUID primary keys.
- created_at / updated_at required.
- Foreign keys indexed.
- Migrations required for schema changes.
- Rollback strategy must be documented.

---

# 7. API Standards

- Versioned routes (/api/v1).
- Input validation required before service layer.
- Explicit error responses.
- Rate limiting defined.

---

# 8. Testing Discipline

- Service layer unit tests required.
- Critical flows integration tested.
- No feature considered complete without validation.

---

# 9. Prohibited Patterns

- Business logic in controllers.
- Global mutable state.
- Silent error swallowing.
- Tight coupling between modules.
- Hardcoded secrets.
- Direct DB access from presentation layer.

---

# 10. Architectural Drift Prevention

- Architecture must be revalidated after major feature additions.
- Any violation must trigger governance update.
