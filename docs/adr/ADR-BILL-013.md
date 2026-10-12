# ADR-BILL-013 — Negative Wallet: anniversary calendar month, cap-free actual charge ledger

- **Status:** `PROPOSED` — `DESIGN_SELECTED_UNDER_DELEGATION`; original one-calendar-month actual-minus/no-estimated-limit rules `APPROVED_BY_USER`.
- **Owner:** Billing (Finance/Multi-Tenant/Security/Mobile/Backend deps); **Date:** 2026-10-12
- **Source:** [ADR-BILL-006](ADR-BILL-006.md), [ADR-BILL-011](ADR-BILL-011.md), [ADR-BILL-012](ADR-BILL-012.md); Round 3 audit. User's 2026-10-12 request delegates selection of best remaining architecture defaults, **not** fictional legal/financial pricing approvals.

## Muammo va tanlov
One-calendar-month grace might mean a named calendar month or anniversary-based calendar arithmetic. 30 consecutive days is **not** a calendar month. Choosing a named month can yield only a few hours of grace at month-end; anniversary gives more predictable duration. Choose **anniversary calendar month, end-of-month clamp** as MVP `DESIGN_SELECTED_UNDER_DELEGATION`; alternative named month retained in [BILL-006](ADR-BILL-006.md), not deleted.

## Specific deterministic candidate policy

1. Wallet negative begins **at first successfully committed actual Seat/Business Usage/Resource Usage debit that changes its durable balance from >=0 to <0**; no estimates, forecast limit or speculative monthly reservation. Exactly zero is not negative. Immutable `grace_start_event_id`, `first_negative_posted_at_utc`, `billing_timezone_iana`, `policy_version`.
2. Define half-open interval `[first_negative_at, anniversary_end_at)`; convert instant to assigned legal Company's persisted IANA `billing_timezone`, add exactly one calendar month keeping local wall-clock and **clamp to last valid day**. If time falls into DST gap/ambiguity, use documented deterministic resolution (next valid instant for gap, earlier offset for overlap) and save resolved UTC. Pilot may default `Asia/Tashkent` when user/company explicitly accepts; do not assume all tenants use it.
3. Examples **illustrative**: October 28 16:00 → November 28 16:00; January 31 10:00 → February 28 10:00 (Feb 29 in leap year); December 31 23:00 → January 31 23:00. Do not roll Jan 31 to March 3 or use 30×24h.
4. If verified funding restores Wallet to >=0, mark `recovered_at` and close current grace window. New negative episodes **within the same open billing cycle** do not automatically grant unlimited renewed grace: use the prior window's original expiry, with `episode_id` for accounting. Reset eligibility at the next company-level billing cycle only after a confirmed nonnegative close. Cycle calendar/statement cutoff is independently configurable and needs economic sign-off.
5. At end of allowed window: **no further new speculative negative allowance**. Continue read-only access/history/export and reconciliation; new *billable* economic operations enter `PAYMENT_REQUIRED / PENDING_REVIEW` rather than silently generating unauthorized unbounded deficit. Nonbillable operations and already-authorized reversal/reconciliation remain allowed. This commercial service restriction is a **design-selected proposal** requiring business/legal sign-off before implementation.
6. Verified top-up posts once with payment reconciliation, never on client attempt. Company-specific Wallet/account; third-party payer may fund target Wallet but debt remains Company's LegalEntity. No automatic cross-company netting or asset transfer.
7. Delayed offline events: client timestamps never authoritative; server rejects currently revoked commands. Accepted source has approved `domain_effective_at`. If former usage month is closed, post immutable linked adjustment in current open posting period with original usage period; do not rewrite grace start or closed invoice without authorized dispute/mapping procedure. Interactions with previously expired grace and backdated corrections remain **OPEN for legal/accounting sign-off**.
8. Distinguish `grace_start`, `grace_end`, `usage_period`, `posting_period`, `invoice_due_date` and `grace_eligibility`. Never confuse SaaS Wallet negative with ERP customer AR.
9. Unique posting key `(billing_account_id, business_occurrence_id, charge_component_version)`; retries/inbox duplicates at-most-once **economic** posting; same key with different payload -> CONFLICT quarantine. Amounts fixed precision, credit conversion 1 credit=10 UZS.

## State matrix — PROPOSED
| State | Condition | New actual charges | UI/operations |
|---|---|---|---|
| `POSITIVE` | balance >= 0 | normal approved | normal |
| `GRACE_ACTIVE` | balance < 0, before grace end | actual-only authorized debit; no cap forecast | show exact posted debt+countdown |
| `RECOVERED` | verified top-up restores >=0 | normal | link preceding episode |
| `GRACE_EXPIRED` | balance < 0, at/after end | no new over-grace debit absent policy; reconciliation/reversal still permitted | payment-required, read/export preserved |
| `REVIEW` | late-event adjustment or disputed liability | policy-guided, no guessed Wallet | operator resolution, audit |

## Risks / alternatives / tests (NOT EXECUTED)
Calendar gaps/leap years, DST overlap, repeated small top-ups to evade grace, late offline corrections, concurrent topup+debit serializability, business timezone changes and enterprise contract exceptions. Compare named-month vs anniversary alternative; choose anniversary for equitable grace. Gate tests: 28/29/30/31, leap year, DST, timezone change after start, zero vs negative crossing, concurrent posting, duplicate topup, repeated recovery, expiry, historical adjustments, no estimated limit, never cross Wallet balances. No numeric pricing, credit ceiling or achieved tests claimed.

## OPEN
Full business sign-off for renewal gate and post-expiry status, customer communication and terms of service; legal invoice due date/tax, billing cycle and timezone default, closed-month correction policy. No Codex product implementation authorized by this ADR alone.
