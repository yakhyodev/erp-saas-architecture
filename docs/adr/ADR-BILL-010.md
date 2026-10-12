# ADR-BILL-010 — Seat activation, proration and charge identity

- **Status:** PROPOSED
- **Date:** 2026-10-11
- **Owner:** Billing with Identity & Security, Multi-Tenant and Business
- **Issue:** MR-03, MR-05, AR-001
- **Reference:** [BILLING_ARCHITECTURE §19.5](../modules/BILLING_ARCHITECTURE.md#195-mr-03--seat-activation-prorating-and-identity)

## Muammo
Low-cost Seat billing **APPROVED**, but billable user membership states, charge period, scope, activation, suspension, prorating and seat discount treatment are OPEN. Parallel branch/device memberships risk double charging.

## Variantlar
1. Invited seat billable vs only accepted/activated seat billable.
2. Full period vs calendar-day proration vs precise-interval proration.
3. Reserved seat remains billable when suspended vs suspension pauses accrual.
4. Per company, tenant membership or assigned seat pool; branch/device duplication semantics.
5. Enterprise negotiated seat net tariff vs standard progressive tier stacking.

## Tanlangan texnik taklif
**PROPOSED:** IAM authoritative effective-dated membership transitions; Billing records versioned billable seat snapshots and computes immutable period/interval charges only under a business-approved lifecycle rule. No option above is APPROVED by this ADR.

## Contract
`tenant_id, company_id?, membership_id, user_id, source_transition_id, source_version, status, effective_at, received_at, billable_seat_scope?, billing_account_assignment_version, seat_pricing_version, seat_period_key`. Dedicated semantic dedup keys for state transition and periodic charge; no client-side seat charges. Rerun must not duplicate same period; membership correction appends debit/credit adjustment linked to original.

## Proposed tests
Invite→accept→activate→suspend→reactivate→remove; join/leave at month boundary; concurrent same user across branches; replay and out-of-order status update; rate change; enterprise tier; rounding and leap month; tenant/company migration.

## Sabab
Preserves authoritative user scope and prevents repeated charges while keeping pricing transparent.

## Xavflar
Wrong billable membership source, charging pending invitations, duplicate seats across branches, timezone/proration disputes, discount stacking conflicts, negative-balance timing.

## Boshqa modullarga ta'siri
Identity/Security owns memberships and authorization, Multi-Tenant owns company/branch mapping; Billing owns seat rating and ledger; enterprise pricing and negative-period windows reference approved period policy.

## Approval gate
MR-03/MR-05 remain OPEN: approved states, scope, charge cycle, effective date source, proration, discounts, rounding and price values required before code. No status promotion.

## Round 2 correction — AR-013 / MR-03 (2026-10-12, PROPOSED ownership; OPEN approval)
**Supersedes this ADR's earlier prose that “Identity/Security owns memberships” as a *proposal*, not as an approved decision.** Candidate authoritative writer chain: **Identity** owns global UserCredentials/authentication; **Multi-Tenant** writes TenantMembership, statuses and company/branch affiliations; **Security** writes RoleAssignment/Grant and evaluates permissions; **Billing** consumes versioned authoritative membership events and owns seat charge snapshots. [Security §17](../modules/SECURITY_ARCHITECTURE.md), [Multi-Tenant](../modules/MULTI_TENANT_ARCHITECTURE.md) align; root MODULE_REGISTRY still says Security owns membership, an explicit unresolved cross-module documentation conflict. Do NOT write membership from Billing or Security under this candidate; AR-013 still OPEN.

| Transition | Billable candidate | Approval |
|---|---|---|
| invited | not billable vs seat reservation | OPEN |
| activated | start immediate prorated vs full next cycle | OPEN |
| suspended | pause vs reserved paid seat | OPEN |
| reactivated | resume interval vs fresh interval | OPEN |
| removed/revoked | end immediately vs end of interval | OPEN |
| company transfer | change BillingAccount at effective_at without double count | OPEN |
| multi-branch/multi-device | one membership/license vs scoped purchased seats | OPEN |

Event `tenant_id, membership_id, user_id, membership_version, transition_id, effective_at, company_scope_before/after, grant_version?, correlation_id`. TenantMembership writer publishes immutable change fact. Security rechecks revocation at sync/server commit; Billing doesn't trust client membership or device session as billable evidence. Transition dedup `tenant_id+membership_id+transition_id`; economic seat interval dedup `billing_account_id+membership_id+seat_license_scope+period+charge_component_version`, corrections link original charge. Proration day/instant/full cycle, price, discounts, suspension and transfer liability unresolved. See [Billing §20.3](../modules/BILLING_ARCHITECTURE.md). No changes to overall APPROVED business rules.
