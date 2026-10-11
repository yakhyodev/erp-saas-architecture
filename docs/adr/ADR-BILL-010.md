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
