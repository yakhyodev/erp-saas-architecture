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

## 21. ROUND 2 CORRECTION — AR-013 P0 / MR-11 / AR-006 (2026-10-12)
**Status: PROPOSED. Membership writer decision OPEN. No production verification, benchmark or security test has been executed.** This section extends §§17–20, not a new APPROVED status.

### 21.1 Authoritative writer vs enforcement matrix (single writer per entity; candidate A)
| Domain entity | Sole authoritative writer (PROPOSED A) | Readers / enforcement | Mandatory invariant |
|---|---|---|---|
| IdentityAuth / UserCredentials | Identity/Auth bounded context (Security identity subdomain candidate) | Backend session guards; Multi-Tenant references user_id | Credentials, MFA factors, credential_version, device/session revocation only writable here |
| TenantMembership | Multi-Tenant | Identity links user; Security evaluates; Backend guards; Billing consumes changes | (tenant_id, membership_id), user_id, lifecycle state, membership_version; Security is **not** second membership writer |
| RoleAssignment / scoped grants | Security/Permissions | Multi-Tenant validates org scope; business modules enforce | (tenant_id, membership_id, grant_id), role_id, scope, grant_version, effective/revoked times; no scope escalation |
| PolicyDefinition | Security/Permissions | Backend/domain guard calls evaluator | policy_id, schema_version, policy_version, action vocabulary, deny precedence; domain owns resource attributes/invariants |
| PermissionEvaluation | Security/Permissions **policy evaluator** (not source-of-truth writer of decision rows) | Backend/domain/API/job/file/search guard performs enforcement | deterministic allow/deny + reason and evaluated versions; no persistent independent writable authorization truth |

**Alternative B (OPEN):** Security/IAM owns *all membership state* and publishes a read-only projection to Multi-Tenant; identity credentials remain Identity owner, Tenant graph stays Multi-Tenant and grants stay Security. A is preferred for alignment with ADR-MT-002; B avoids separate membership admin API but couples tenant lifecycle to IAM. **Rejected anti-pattern:** both Security and Multi-Tenant modifying membership status or two independently writable role-grant stores. Modular monolith deployment placement does not change domain ownership. Backend provides transport/orchestration and guards, **not** a second credential/membership/grant writer.

### 21.2 Source conflicts and resolution path (not silently resolved)
- `MODULE_REGISTRY.md` Security row says “Identity, membership, RBAC/ABAC”; later Round 2 candidate table describes Multi-Tenant authoritative membership writer, still OPEN. The generic row is conflicting/ambiguous, **not user approval**.
- Security §17 proposes Identity→Multi-Tenant membership→Security grants. Aligns with ADR-MT-002 candidate A; remains OPEN.
- `ADR-MT-002` original text says “Identity/Security membership and scope” but Round 2 addendum compares A Multi-Tenant writer and B Security writer. Treat initial phrase as historical ambiguity, not confirmed ownership.
- `ADR-BILL-010` original § “IAM authoritative membership transitions” and “Identity/Security owns memberships” conflict with candidate A; its Round 2 correction supersedes that prior **proposal text**, and keeps Billing as Seat qualification, effective-date snapshot, rating and ledger writer. **Membership activation ≠ automatically billable Seat**; day-level proration selected for discussion is pending approval.
- Global registry and other module files may only be patched by their owners; 00-chat patch is below. No previous APPROVED decision was altered.

### 21.3 Contract: request, tenant switch, revocation and cache
**Guard input (PROPOSED):** principal_id, session_id, device_id?, credential/session_version, tenant_id, membership_id, membership_version, tenant_state, company_id?, branch_id?, resource_type/id, authoritative resource tenant/company/branch/status, action, grant_version, policy_version, MFA assurance, operation_id?, correlation_id.
**Server sequence:** validate current authenticated principal/device → resolve current membership **from its authoritative owner** → resolve company/branch ownership in Multi-Tenant → Security Evaluate(policy/grants/resource attributes) → domain-specific guard checks → recheck authorization at critical commit/side-effect boundary → per-tenant DB transaction/outbox. A browser/mobile UI hide/show or route protection is informational only. Apply to reads, writes, streams, websocket subscriptions, batch, worker, export, search metadata/counts/facets/autocomplete, file/signed URLs and background activities. Wrong-scope IDs should not expose existence via errors/timing/content.
**Switch:** new verified tenant context tied to session/user/device and selected active membership/version; previous tenant context cannot authorize the new tenant; invalidate selected company/branch, cached scopes, cursors and subscriptions. **Revocation:** writer atomically changes state/version and outbox event; Security updates grant/policy versions independently; event subscribers evict affected caches and revoke relevant sessions/subscriptions. Events are *invalidation hints*, not authorization proof. For sensitive mutation, if authoritative current versions are unavailable → fail closed (or retryable unavailable without side effect). Same tenant + stale role/membership version must not remain allowed. Event delivery may be duplicated/out of order; compare monotonic versions. No fixed permission TTL, MFA matrix or revocation latency is approved.
**Cache key:** principal/session + tenant + membership/version + grant/policy versions + company/branch + resource scope/action + assurance level; deny cache cross-tenant reuse. Authorization freshness gate for read-heavy operations vs strong check for writes is OPEN; never authorize using only long-lived claims.

### 21.4 MR-11 negative-test acceptance matrix — DESIGN ONLY / NOT EXECUTED
| Test ID | Scenario | Expected |
|---|---|---|
| SEC-R2-01 | Tenant A principal guessing B IDs: detail/list/mutate | no B content or mutation, default deny |
| SEC-R2-02 | Branch A→B, wrong company/branch FK | deny before side effect |
| SEC-R2-03 | revoked membership/role; stale cache; tenant switch race | no stale permit; session/subscription recalculated |
| SEC-R2-04 | missing/spoofed transaction tenant context; RLS USING/WITH CHECK | deny SELECT and INSERT/UPDATE/DELETE |
| SEC-R2-05 | same pooled DB connection A→B, rollback, retry, prepared statements, transaction pooling | B cannot inherit A scope; missing context denied |
| SEC-R2-06 | runtime role table-owner/BYPASSRLS/superuser attempt, SECURITY DEFINER/view | ordinary role cannot bypass; privileged routes separately audited |
| SEC-R2-07 | tenant-scoped worker, outbox/inbox, scheduled job, replay | no data/event/charge crossing tenant |
| SEC-R2-08 | search hit, counts, facets, autocomplete, analytics, cache keys | no cross-tenant result, cardinality or metadata leak |
| SEC-R2-09 | export, object-store path, presigned URL, CDN and revocation during download | tenant/resource authorization; expiry and refresh protection |
| SEC-R2-10 | offline queue on device revoked / stolen token / expired refresh | reject with audited permission/device cause, no business commit |
| SEC-R2-11 | impersonation without consent/expiry/step-up; break-glass | deny or scoped audited exceptional path |
| SEC-R2-12 | multiple retries/timeouts with stable operation_id | at-most-once business application, unknown ≠ accepted |
| SEC-R2-13 | tenant placement_epoch stale after move/restore | fail closed, no cross-placement access |

**Evidence required before release:** test dataset tenant A/B and branches, actors/roles and resource ownership map, case IDs, test runner build/commit, environment+DB/runtime role and pool mode, actual SQL policy configuration where applicable, command/response artifacts with PII redacted, observed pass/fail, penetration review and exception remediation links. Target is zero successful forbidden operations, **not a measured result**. PostgreSQL/shared schema/RLS, dedicated placement and tenant migration remain PROPOSED and MR-11 OPEN.

### 21.5 Mobile auth and support (AR-006; PROPOSED)
- Mobile ADR-MOB-004/009 **exist**; their auth, device revocation and conflict contracts are PROPOSED, not missing. Register device with session family; refresh rotation and server-maintained revoke status. Device clock is not authority. Reauthenticate identity + check device + current membership/grants/scope at every queued command acceptance.
- Statuses: accepted / rejected_permission / rejected_device / rejected_validation / conflict_version / conflict_business / retryable / unknown. Preserve stable operation_id on timeout, do not double-submit finance/stock/billing effects. Offline drafts may be quarantined pending retention and user recovery policy. **Remote wipe while disconnected is not guaranteed**.
- Stolen access token: short lifetime proposed, server revocation/session/version checks; protect refresh in OS secure store; theft detection and rotation; unknown risk blocks sensitive operations where assurance insufficient. MFA: step-up for role elevation, impersonation, billing/financial sensitive actions; exact user cohort and factor/recovery UX OPEN.
- Support impersonation: no default tenant access; tenant approval or documented break-glass, MFA, reason/ticket, least privilege, bounded expiry, dual actor identities in audit; no untracked bypass of RLS, and deny restricted financial/permission actions until separately approved.

### 21.6 00-chat registry patch proposal — NOT APPLIED
1. `MODULE_REGISTRY.md` row 10 replace generic "Identity, membership, RBAC/ABAC" with "IdentityAuth/Credentials and RoleAssignment/PolicyDefinition/PermissionEvaluation (PROPOSED); TenantMembership authoritative owner: Multi-Tenant candidate A vs Security candidate B OPEN (AR-013); enforcement Backend/domains".
2. `DECISIONS.md` register candidate A and B as **PROPOSED**, explicitly mark AR-013 OPEN; never mark a new membership owner APPROVED.
3. `OPEN_QUESTIONS.md` preserve AR-013 OPEN; add PolicyDefinition owner, delegation, reassignment/rejoin, cache/revoke SLA, B alternative, and MR-11 runtime role/pooling/placement evidence questions.
4. `MASTER_ARCHITECTURE.md` refer to owner matrix + single-writer invariant; no premature hardcoded entity schema.
5. `REQUIREMENTS.md` security acceptance/test evidence and negative test suite as proposed release gates; `CHANGELOG.md` note Round 2 PROPOSED alignment.
**Open sign-off:** Security+Multi-Tenant+Backend+Billing+Database+Mobile+00 Master; no code or security tests until decisions are approved.
