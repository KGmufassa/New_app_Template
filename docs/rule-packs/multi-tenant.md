# RULE PACK – MULTI-TENANT

Apply when:
- Tenant Model = Multi

Requirements:

- Tenant ID must exist on all tenant-owned tables.
- Tenant scoping middleware required.
- No cross-tenant queries allowed.
- Index tenant_id for performance.
- Tenant isolation strategy must be documented.
