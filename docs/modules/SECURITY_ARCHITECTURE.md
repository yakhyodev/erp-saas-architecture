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

## 17. Architecture alignment — AR-013 (PROPOSED ownership contract; OPEN approval)
**Authority matrix (one writable source of truth per entity; candidate, not approval):**
| Entity / capability | Candidate authoritative owner | Writes / authoritative state | Consumer boundary |
|---|---|---|---|
| Identity / UserCredentials | Identity/Auth (Security identity subdomain; Backend hosts implementation only) | global user_id, login identifiers, credential verifier, MFA, sessions, device trust, principal status | Tenant uses stable user_id only, never password/MFA copies |
| TenantMembership | Multi-Tenant | membership_id, tenant_id, user_id, invited/active/suspended/revoked, version, effective scope association | Security consumes active membership evidence; does not modify membership state directly |
| RoleAssignment / Grant | Security/Permissions | tenant_id + membership_id + role_id + explicit company/branch/resource scope, effective interval, assignment_version, grant/delegation provenance | Tenant validates org hierarchy; domain owner validates actual resource attributes |
| PermissionEvaluation | Security/Permissions, as shared policy contract | policy catalog/version and deterministic allow/deny decision; no persistent cross-domain privilege materialization | Backend/domain server-side guards enforce evaluated decision for each use-case and each object |
| Tenant/company/branch registry | Multi-Tenant | lifecycle, tenant graph, company/branch ownership, optional placement_epoch | Security must not create companies/branches |
| Seat billable status | Billing | billable seat criteria, time/period and charge attribution | Membership active != billable Seat; no assumed seat price |

**Proposed request sequence:** Identity.Authenticate → resolve active session/device/principal → MultiTenant.ResolveMembership(user_id, target_tenant_id) → MultiTenant.ResolveOrgScope(company_id?, branch_id?) → Security.Evaluate(principal, membership_id/version, role_grants/version, policy_version, action, authoritative resource attributes, device/MFA strength) → Backend/domain guard fail-closed → DB transaction/tenant-scoped worker context. **Never** authorize solely via UI, token claim, submitted tenant_id, header or stale cache. Fetches, lists, count/autocomplete, exports, jobs, websockets and files use the same invariant.
**Switch and revoke:** tenant switch re-resolves membership, creates distinct signed/server-verified active context bound to user/session/device, tenant, membership/version and scoped grants; previous tenant context must not authorize new operations. Session/device logout or revoke invalidates refresh; access credentials subject to server revocation/version check. Membership suspend/revoke and grant/role changes increment authoritative versions and issue idempotent change notification via outbox; invalidate scoped caches and active subscriptions. **Events are hints not sole safety mechanism:** critical mutations and all sync must check current authoritative state before commit/acceptance. Failure to obtain authoritative status -> deny or retryable unavailable; no privilege fallback. In-flight/retry operations re-evaluate before side-effect. Audit actor, tenant, changed grant, reason and revoker.
**OPEN choices:** candidate A (Multi-Tenant membership owner, Security role/grants owner) vs candidate B (Security owns membership; Tenant read model); rejoin of revoked identities, role delegation policies, service principal model, invitation identity merge, security policy caching maximum staleness, exact session/token expiry and MFA scope. AR-013 remains OPEN until joint Security+Multi-Tenant+Backend review and explicit approval. Registry descriptions are non-authoritative candidates, not permission to duplicate writes.

## 18. AR-006 — mobile authentication, device revocation and queued sync (PROPOSED)
Reference Mobile-owned [ADR-MOB-004](../adr/ADR-MOB-004.md) and [ADR-MOB-009](../adr/ADR-MOB-009.md); Security owns auth/device-status verification policy, Mobile owns local UX, outbox and conflict flow, Backend owns authoritative command acceptance.
- DeviceRegistration links global user, device_id (unpredictable, not sufficient authenticator), session family and optional active tenant scope. Lost/stolen/revoked devices cannot refresh; server checks current device/session state and membership/permission before **every queued-command processing attempt**. A queued offline draft is never proof of current authority.
- Command contract (candidate): operation_id stable across retries, device_id, session identity proof, target tenant/branch/resource, base_version, payload schema and correlation_id. Backend independently resolves trust context and idempotency; never trusts submitted tenant claims, device time, or cached grants.
- Response classes distinguish accepted, rejected_permission/device_revoked, rejected_validation, conflict_version, conflict_business, retryable/unavailable. Timeout = UNKNOWN; retry same operation_id, never double-post. Revoked queued data retained only under bounded local quarantine/deletion policy pending approval, never secretly uploaded or posted.
- Offline remote wipe cannot be guaranteed; online next contact locks/wipes scoped local data according to approved retention and legal hold. Device binding/attestation, local TTL, expiry of cached entitlement, recovery/export of unsubmitted drafts, and remote wipe policy remain **OPEN**. Server-side revocation effectiveness only on connectivity; do not claim immediate remote purge.

## 19. MR-11 — conditional RLS and pooled tenant isolation test contract (PROPOSED, not executed)
PostgreSQL/shared schema/RLS selection remains OPEN with Multi-Tenant, Database and Infrastructure. If selected, table-owned tenant rows use trusted tenant_id, tenant-composite uniqueness/FKs, RLS USING+WITH CHECK where needed, FORCE RLS threat review, runtime role with no superuser/BYPASSRLS or owning table authority. Transaction-local tenant context must be mandatory, initialized after trusted auth and cleared at transaction end, never session-global. Pooled connection reused by a different tenant must fail closed when no new context is supplied. No privileged bypass in ordinary HTTP/worker processes.
**Design acceptance matrix (tests NOT RUN):** (1) A vs B read/write/list/count/export/search/file/CDN, (2) branch ownership and resource status, (3) missing/malformed tenant context, (4) sequential A→B reuse of the same pooled connection, rollback/retry, prepared statements, long-running cursor, job/worker and batch, (5) runtime role cannot bypass policies; SECURITY DEFINER, owner roles, views, migrations/backup explicitly reviewed, (6) stale placement_epoch, dedicated migration, restore and worker placement fencing, (7) cache/index/job/attachment URL tenant scoping and negative leakage tests, (8) revoked membership, stale permission snapshot, concurrent tenant switch and offline replay. Verify both unauthorized read and unauthorized mutation; no fake passing results.
**OPEN:** final data-placement topology, privileged operator break-glass, RLS bypass recovery mechanics, isolation test fixtures, RPO/RTO, regional legal retention, connection pooling mode and tenant mobility cutover are DB/Multi-Tenant/Infra joint decisions. MR-11 remains OPEN. Security does not set DB schema or deployment topology unilaterally.

## 20. Codex handoff gate — proposed, NOT execution approval
Contract docs only: identity/membership/grants ownership; per-operation authorization guard; opaque scoped context binding; revoke/version invalidation and fail-closed rules; offline sync rejection taxonomy; negative tenant/RLS/pool test specification. No implementation, migration, test execution or security certification is claimed. Review gate AR-013+AR-006+MR-11 stays BLOCKED pending owning ADR approval and joint sign-off.
