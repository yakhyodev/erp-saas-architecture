# SECURITY_ARCHITECTURE.md

> ERP SaaS · v0.1 · 2026-10-11 · **Status: PROPOSED**. No code.

## Status distinctions
**APPROVED business rules (pre-existing):** Hybrid Prepaid SaaS; inexpensive Seat + document-priced Business Usage with volume credits + Resource Usage; 1 credit = 10 UZS; progressive volume discount and enterprise contracts; after balance depletion, actual Seat/Usage charges accumulate negative for one calendar month, without an estimated credit ceiling; unified platform for companies/branches; uz/en/ru with hidden future Uzbek alphabet.

**PROPOSED security controls below:** no technical security decision is approved yet. Go, PostgreSQL, Next.js, Flutter, modular monolith and shared database are provisional.

## 1. Trust boundaries and identity
Separate global identity, tenant membership, active tenant, branch scope, service principal and user/device session. Platform support does not automatically have tenant data access. Enforce boundaries across client/API, application/database, queues, integrations, storage, search, analytics, reports and backups.

## 2. Authentication — [ADR-SEC-001](../adr/ADR-SEC-001.md)
Central identity and validated membership. Verified identifier, salted memory-hard passwords (e.g. Argon2id), anti-enumeration and throttling. Privileged roles use MFA, with passkey/WebAuthn preferred and TOTP fallback (proposed scope). Web sessions use secure HttpOnly SameSite cookies with CSRF protection and revocation; mobile uses OS secure storage, short-lived credential/refresh rotation. Tenant switch rechecks membership. Sensitive actions require step-up MFA. Offline credentials are time-limited and sync rechecks authorization.

## 3. Authorization / RBAC+ABAC — [ADR-SEC-002](../adr/ADR-SEC-002.md)
Deny by default, enforce at backend for actions, records, lists, reports, exports, downloads and async jobs. Roles grant coarse permissions such as `module.resource.action`; context checks tenant, branch, ownership, state, SoD, approval limits and authentication strength. Proposed role examples: owner, admin, branch manager, salesperson, warehouse operator, accountant, read-only auditor, integration service. No self-escalation. UI visibility is not security enforcement. Cache is tenant/membership/version scoped and invalidated after revocation.

## 4. Tenant and branch isolation — [ADR-SEC-003](../adr/ADR-SEC-003.md)
Trusted auth membership provides tenant context, never trust client tenant ID alone. Tenant ownership applies to business objects, events, billing usage, files, cache keys, queues, search indexes and audit. Cross-tenant FKs/object lookups are invalid. Branch rights cannot exceed tenant boundary; missing branch value does not imply global access. Proposed initial option: application checks + shared Postgres RLS defense in depth; enterprise dedicated data plane/sharding later only after cross-module decision.

## 5. PostgreSQL RLS — [ADR-SEC-004](../adr/ADR-SEC-004.md)
Conditional on PostgreSQL approval: ENABLE/FORCE RLS for tenant-owned tables; runtime non-BYPASSRLS and non-owner DB roles; transaction-local trusted context and pool isolation; default deny. Privileged backups/migrations, SECURITY DEFINER/views/replicas require dedicated audited review. RLS does not replace branch or business-domain authorization.

## 6. Audit trail — [ADR-SEC-005](../adr/ADR-SEC-005.md)
Append-only/tamper-evident critical events with original actor, acting-as, tenant, branch, session/correlation, resource/action, time, outcome and reason. Login/MFA, membership/roles, support, exports, billing/finance, sensitive configurations and secret changes are auditable. Never log tokens or credentials. Access, retention and legal hold requirements need compliance review.

## 7. Support impersonation — [ADR-SEC-006](../adr/ADR-SEC-006.md)
Explicit tenant permission or documented break-glass; short-lived purpose-scoped elevation, MFA, reason/ticket, clear acting-as UI and complete audit. No unrestricted support-superuser, and restrict sensitive billing/permission administration while impersonating.

## 8. Encryption — [ADR-SEC-007](../adr/ADR-SEC-007.md)
TLS for transport, encrypted database, object storage and backups at rest, managed KMS/envelope for sensitive data, access-limited keys with rotation and audit, mobile encrypted offline storage. Store password hashes, not reversible plaintext passwords.

## 9. Secrets — [ADR-SEC-008](../adr/ADR-SEC-008.md)
Managed vault and scoped workload credentials, environment and tenant separation for integrations, automatic rotation/revocation, secret scanning and emergency incident response. Never commit credentials or expose them in frontend, logs, or ordinary config.

## 10. API security — [ADR-SEC-009](../adr/ADR-SEC-009.md)
Authenticate and authorize object level (BOLA/IDOR). Input/schema validation, bounded pagination/payload/files, anti-replay and idempotency for critical writes, quotas/throttling, CORS allowlist and CSRF for cookie flows, signed/timestamped webhooks, safe error responses. Usage events from authoritative source only.

## 11. Threat model, negative tests — [ADR-SEC-010](../adr/ADR-SEC-010.md)
Test credential stuffing, session theft, horizontal/vertical privilege escalation, tenant A querying tenant B through detail/list/export/report/search/file/CDN, branch bypass, pooled RLS context reuse, async queue/cache/job mixups, support abuse, revoked session/offline sync, secret exposure and audit tampering. Automated isolation regression is a proposed release gate.

## 12. Compliance questions
Identify actual served jurisdictions and data residency rules. Counsel to verify GDPR applicability, controller/processor roles, DPA, lawful basis, rights requests, retention/deletion, backup deletion, transfers and breach notifications; assess Uzbekistan personal data and fiscal retention/localization laws. No compliance certification is claimed.

## 13. Other module impacts
Master/Requirements: zero trust and nonfunctional controls. Multi-Tenant/Database: tenant IDs, composite FK, RLS, sharding. Backend: central enforcement, trusted context. Finance/Billing: immutable usage evidence, SoD, audit; do not change approved negative balance rules. Sales/Inventory/CRM: branch scope and exports. Mobile/Offline: encryption and re-auth on sync. Integrations: tenant-scoped tokens, webhook signature. Infrastructure: secret vault, KMS, backup and incident response. UI/UX/Localization: consent, MFA and access indications without UI-only authorization.

## 14. ADR registry — all PROPOSED
ADR-SEC-001 Authentication; 002 Authorization; 003 Tenant isolation; 004 PostgreSQL RLS; 005 Audit; 006 Impersonation; 007 Encryption; 008 Secrets; 009 API security; 010 Verification. See [ADR folder](../adr/).

## 15. Proposed common-document sync — NOT applied
DECISIONS.md: add ADR links as PROPOSED. MASTER_ARCHITECTURE.md: trust boundaries/security invariants. REQUIREMENTS.md: security NFR and testing. MODULE_REGISTRY.md: Security dependencies. OPEN_QUESTIONS.md: MFA/SSO, isolation, impersonation, retention, GDPR/local law. CHANGELOG.md: proposal creation event. These files have not been modified here.

## 16. Codex security specification — architecture only
Identity+membership+session contracts, MFA and step-up, revocation; server-side policy evaluation for action/object/tenant/branch; trusted tenant context throughout app/DB/jobs/files; conditional PostgreSQL RLS; immutable sensitive audit; TLS/KMS/vault; IDOR-safe APIs with validation/webhooks/rate limits and idempotency; automated cross-tenant negative tests. Do not write code until necessary ADR approvals and cross-module review.
