# ADR-MT-002 — Tenant hierarchy va global membership

**Status:** PROPOSED  
**Date:** 2026-10-11  
**Owner:** Multi-Tenancy Architecture

## Muammo
Bitta global user bir nechta tenantda ishlashi, company/branch permission scopes va billing attributionni aniqlash kerak.

## Variantlar
1. User-per-tenant — oddiy authorization, ammo identity takrorlanadi.
2. Global User + TenantMembership — yagona identity, mustaqil tenant permissionlari.
3. Company = Tenant — sodda, lekin ko‘p kompaniyali tenant modelini cheklaydi.

## Tanlov (PROPOSED)
Global User + TenantMembership; Tenant → Company → Branch; tenant/company/branch/resource bo‘yicha server authorization. User bir tenantda aktiv bo‘lishi boshqasiga ruxsat bermaydi.

## Sabab
Identity takrorlanmaydi; har tenant uchun mustaqil permission; yuridik company izolatsiya boundarydan ajratiladi.

## Xavflar
Context leak tenant switchda; sekin membership revoke; branch inheritance; Billing Seat metriği noaniqligi; Company = Tenant bo‘yicha boshqa modul bilan ziddiyat.

## Kamaytirish
Fail-closed context, membershipni qayta tekshirish va revocation, audit, negative isolation tests, company/branch ownership validation; Billing/Finance tasdig‘isiz ownershipni hardcode qilmaslik.

## Ta’siri
Identity/Security membership va scope; Billing Seat hamda billing_account mapping (OPEN); Finance legal company; Sales/Inventory filiallar; Backend/UX tenant selector.

## Tasdiqlash sharti
Master, Finance, Billing va Security bilan hierarchy, ownership, Seat va scope semantikasini tasdiqlash. Ungacha **PROPOSED**.


## 2026-10-11 reconciliation clarification (PROPOSED; AR-001 / AR-013 / MR-01)

This ADR's Tenant→Company→Branch diagram is **a possible organizational mapping**, not approval that Tenant=Company, LegalEntity=Company, or BillingAccount/Wallet cardinality follows hierarchy.

**Proposed relational alternatives:**
- A: Tenant 1:1 Company 1:1 LegalEntity; Tenant 1:1 BillingAccount; BillingAccount 1:1 Wallet.
- B: Tenant 1:N Company, Company 1:1 LegalEntity (assumption), Tenant 1:1 BillingAccount/Wallet; cross-company consolidated payer.
- C: Tenant 1:N Company with effective-dated Company↔LegalEntity mapping and Tenant 1:N BillingAccount; Wallet per-account or multiple per-account OPEN; payer mapping scoped/effective-dated.
- D: Tenant 1:N Company, company 1:1 BillingAccount 1:1 Wallet; shared seat/resource allocation OPEN.

All **OPEN**: legal entity master ownership (Finance candidate), payer and debt owner, cross-company consolidated versus separate billing, branch cross-company transfer, wallet cardinality. Approved hybrid prepaid/business pricing/actual one-calendar-month minus/no estimated limit are independent business invariants and must not be used as evidence of a particular payer model.

**Ownership candidates — two mutually exclusive authority patterns:**
- A (**recommended PROPOSED**): Identity authoritative for credentials/global principal; Multi-Tenant authoritative for membership status/version and tenant-company-branch registry; Security authoritative for role assignments, policy definitions, permission evaluation; business domain authoritative for its resource owner; Billing authoritative for Seat definition, payer, Wallet and ledger.
- B (**alternative OPEN**): Security authoritative for membership lifecycle and permission grants; Multi-Tenant read-only membership projection + authoritative tenant graph. Do not maintain dual writable membership truths. Global `MODULE_REGISTRY.md` assigns membership to Security, so A needs explicit cross-module sign-off.

**Lifecycle proposal:** invited (not authorized) → active → suspended → revoked; revoked rejoin/reactivation semantics OPEN. Auth principal + active membership + valid tenant status + scoped role + resource ownership required; default deny. Cache invalidation/version on status/role/tenant switch. User being member does not automatically equal paid Seat.

**API / event boundaries:** Identity.Authenticate→principal; Membership.Resolve(user,tenant)→membership state/version; Security.Evaluate(principal,membership,company,branch,resource,action)→decision; Tenant.ResolveScope(company,branch)→tenant relationship; Billing.ResolvePayer(tenant,company,branch,event_time)→billing_account or unresolved; UsageFactRecorded.v1 carries tenant/company?/branch?/event_id/idempotency_key/occurred_at/recorded_at/source_document_id?/actor?/membership_id? and never mandates wallet selection by producer. Chargeable document timing, late/offline usage month, refunds and payer switching remain Billing/Sales/Mobile OPEN.

**Approval gates:** AR-001 / MR-01 Billing+Finance+Master user decision; AR-013 Security+Identity+Multi-Tenant ownership and scope sign-off. Until then **PROPOSED**.

## Round 2 addendum — Cardinality, membership writer and liability (2026-10-12)

**Status stays PROPOSED; AR-001/AR-013/MR-01 OPEN.** Tenant is data isolation boundary, not automatically Company, LegalEntity, BillingAccount, Wallet, legal payer or debtor. Finance candidate owns LegalEntity and ERP GL; Billing owns BillingAccount, payer assignment, SaaS Wallet, debt and credit ledger; Multi-Tenant candidate owns Tenant/Company/Branch; Security owns RoleAssignment/Policy/PermissionEvaluation; Identity owns global user credentials.

**Compared models** (see [module §19.2](../modules/MULTI_TENANT_ARCHITECTURE.md)): single tenant/company/branch; one company with multiple branches; multi-company shared payer; per-company BillingAccount and wallet (**Round 2 recommended MVP candidate, not approved**); enterprise effective-dated consolidated billing across companies. Wallet 1:1 vs 1:N, legal debtor and Seat splitting stay OPEN. In Billing §3 company wallet appears prescriptive, but Billing §19 explicitly reclassifies as only one option. No tenant ID→payer or wallet derivation.

**Writer ownership alternatives:** A (recommended PROPOSED): Identity global principal/sessions, Multi-Tenant sole TenantMembership status/version writer, Security sole RoleAssignment/Grant and policy writer, Backend guards, Billing Seat qualification. B (OPEN): Security/IAM sole membership writer, Multi-Tenant read-only member projection. **Contradiction remains:** `MODULE_REGISTRY.md` Security lists membership; Billing §2/ADR-BILL-010 calls IAM membership, while Security §17 and Backend §17 favor Multi-Tenant. Never dual-write membership status. Revoke/version invalidation plus server-authoritative recheck on switch/job/sync required.

**Event contract proposal:** membership lifecycle state/version and usage evidence carry immutable tenant_id, company_id?, branch_id?, user_id?, membership_id?, occurred_at, recorded_at, idempotency key and source ID; Billing does effective-dated payer mapping, rating and liability snapshot, unresolved payer→pending. Multi-Tenant never rates Seat/Usage or writes Wallet.

**Approval gate:** user + Billing/Finance decide cardinality/payer/debtor/allocation; Security/Backend/Master decide one membership writer, delegation/invalidation policy; no code until approved. Negative tests [MT §19.5](../modules/MULTI_TENANT_ARCHITECTURE.md) are specifications, not verified tests.


## Round 2 current-source delta — Billing §20 / registry v0.2 (2026-10-12)

**Status: PROPOSED, OPEN** — no new ownership approval. The newer Billing §20 explicitly treats company-level wallet as a **preferred MVP option** and distinguishes *paying actor* (`payer_party_id`) from *legal debtor* (`legal_debtor_entity_id`) and payment authorization; neither is inferred from Tenant, Company or Branch IDs. Tenant/Company/LegalEntity/BillingAccount/Wallet cardinalities remain open for user, Billing and Finance. Source-activity attribution may have company/branch but SaaS charge liability is assigned only by effective-dated Billing mapping and immutable mapping version. Global/shared Seat and Resource Usage must not guess a company.

The refreshed `MODULE_REGISTRY.md` Round 2 v0.2 records Multi-Tenant as the **candidate** membership writer while keeping Security-owned alternative OPEN; its earlier high-level Security membership assignment is still a documented conflict, not settled. Latest Backend §18 similarly proposes Identity global credentials → Multi-Tenant Membership → Security RoleAssignment and PermissionEvaluation → Backend/domain enforcement. **No dual writer** for membership or grants. Reader projections and admin UX do not become owners.

Transport `event_id` and canonical `business_occurrence_id` must be distinct where source business occurrence exists; identical occurrence with a new envelope must not duplicate wallet economics. Billing alone rates/post charges; membership transitions are billable-seat inputs, not debit commands. Extra test cases: third-party payer vs legal debtor, company transfer across effective mapping, same occurrence across placement cutover. No test executed.

See [module §20](../modules/MULTI_TENANT_ARCHITECTURE.md). Shared DB/RLS, enterprise dedicated placement and owner selection remain **PROPOSED/OPEN**; no code release approved.
