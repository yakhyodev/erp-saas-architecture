# ADR-BILL-012 — MVP Seat Lifecycle, kunlik proratsiya va charge identity

- **Holat:** `PROPOSED` — ADR-BILL-011 dagi user-approved Seat scope **APPROVED_BY_USER**, ushbu hujjatdagi iqtisodiy tafsilotlar `PROPOSED/OPEN`
- **Sana:** 2026-10-12
- **Owner:** 01 Monetization & Billing
- **Cross-module:** 02 Multi-Tenant, 04 Backend, 06 Finance (SaaS/ERP ledger separation), 08 UX, 09 Mobile, 10 Security, 00 Master
- **Audit:** MR-03, MR-05; AR-001, AR-013
- **Mavjud hujjatlar:** [ADR-BILL-010](ADR-BILL-010.md), [ADR-BILL-011](ADR-BILL-011.md), [Billing Architecture](../modules/BILLING_ARCHITECTURE.md)

## 1. Muammo va tasdiqlangan chegaralar

**APPROVED_BY_USER (ADR-BILL-011):** MVP da har Company alohida LegalEntity, BillingAccount va Wallet'ga ega. Seat iqtisodiy identifikatori **`(tenant_id, company_id, canonical_user_id)`**: shu user bir Company ichidagi bir nechta Branch, Role yoki Device bo'yicha **qo'shimcha Seat emas**; boshqa Company'da esa alohida Seat hisoblanadi. Seat narxining arzon bo'lishi va haqiqiy Seat/Usage xarajatlarining Hybrid Prepaid tizimida hisoblanishi oldindan APPROVED. Uchinchi tomon funder SaaS qarzdorni o'zgartirmaydi. **Seat status, daily proration, tariff, suspension va refund bu tor scope tasdig'i ichiga kirmaydi.**

## 2. Variantlar va tavsiya

| Mavzu | Variantlar | MVP tavsiya (`PROPOSED`) | Ochiq iqtisodiy nuqta |
|---|---|---|---|
| Boshlanish | Invitation / acceptance / activation | `ACTIVE` effective membership'dan keyin billable; invitation/pending = 0 | Activation'ni kim va qachon rasmiy qiladi |
| Period | Full-month / daily proration / exact-second | Calendar-day proratsiya, month length bo'yicha; real Seat interval | Oyning bosh-oxir timezone, boshlangan/to'xtagan kun siyosati |
| Pause | Suspend hamon paid / pause / end-cycle | Security suspend va commercial licensed Seat'ni alohida ko'rish; user-visible paid state aniq bo'lsin | `SUSPENDED` avtomatik bepul bo'ladimi? **OPEN** |
| Reactivation | New seat / same seat interval | Original user×Company seat, yangi active interval; idempotent, no reset | Qayta faollashgan kun hisoblanishi |
| Revoke/remove | Instant billing end / end-cycle | `REMOVED` dan keyin kelgusi accrual yo'q; posted tarix tuzatma orqali | Revocation day va pro-rata qaytim **OPEN** |
| Company change | Origin to destination / pooled | Company A/B o'z active intervaliga ko'ra alohida, old liability snapshot saqlanadi | Same-day transfer iqtisodiy attribution **OPEN** |
| Month-end posting | Daily / monthly / hybrid | Kunlik deterministic `seat_day` read snapshot + oylik summary/ledger accrual candidate | Charge cadence, statement, late changes, reversal **OPEN** |

**Taklif asoslari:** charge faqat haqiqiy faollik daliliga bog'lansin; davr va narx keyinchalik qayta hisoblanadigan darajada tushunarli; qimmat heartbeat yoki device-based Seat metering kerak bo'lmasin. Kanonik user bir nechta membership/branchga ega bo'lsa, dedup *membership ID bilan emas*, user×Company iqtisodiy identity bilan qilinadi.

## 3. State machine va writer/reader boundary (`PROPOSED`)

| Membership fact | Auth/UX | Billable Seat candidate | Izoh |
|---|---|---|---|
| INVITED/PENDING | No auth to protected use-cases | 0 | Payment on invitation yo'q deb tavsiya |
| ACTIVE/ACTIVATED | Scope-granted access after server guard | 1 per eligible user×Company | Effective versioned event + active commercial status |
| SUSPENDED | Backend deny relevant action | `OPEN`: pause yoki reserved paid | Auth revoke `!=` commercial billing change |
| REACTIVATED | New authorized interval | Continue one user×Company Seat | Replays duplicate count bermaydi |
| REMOVED/REVOKED | Access denied as soon as server authoritative | Future accrual stops candidate | Billing effective cutoff and refund OPEN |
| COMPANY_SCOPE_ADDED | New Company permission separately validated | Additional Company Seat, if active | User across A+B = independent approved Seat scope |
| COMPANY_SCOPE_REMOVED | Old Company access removed | Old Company's future accrual stops candidate | Prior Wallet/debt immutable |
| BRANCH/ROLE/DEVICE_CHANGED | Permission reassess | **No additional Seat** | APPROVED scope invariant |

**Single writer candidate (AR-013 still OPEN):** Identity owns global principal/credentials; Multi-Tenant owns effective `TenantMembership` status/company/branch scopes, versioned immutable events; Security owns role/grant and server authorization; Billing *only* owns billable-seat snapshots, charge policy and SaaS ledger; Backend rejects unauthenticated, revoked, stale or out-of-scope operations. Billing never modifies membership and a stale/offline device token is never valid charge evidence.

## 4. Versioned economic identity, proration va time policy (`PROPOSED`)

- Two distinct IDs: `membership_id` identifies source membership history; `seat_subject=(tenant_id,company_id,user_id)` identifies approved economic scope. At most one concurrent active economic Seat per subject, even with duplicate memberships or multiple branches/devices; multiple Companies are independent.
- Example proposed data: `tenant_id, company_id, legal_entity_id, billing_account_id, wallet_id, user_id, membership_id, membership_version, company_scope_version, status, effective_at, committed_at, accepted_at, source_transition_id, billing_timezone_iana, billing_month_key, accrual_day_local, pricing_version_id, seat_policy_version, charge_occurrence_id, reversal_of_charge_id?, correlation_id`. Wallet and legal liability are server-resolved as of effective policy; never client authority.
- **Monthly list-price proration proposal:** `period_charge = monthly_seat_price × billable_local_dates / local_dates_in_month`, fixed-precision rational calculation followed by policy-defined deterministic rounding. If price or Company liability changes mid-period, compute non-overlapping versioned segments; don't silently mutate old posted ledger.
- **Day-cutoff decision remains OPEN:** one candidate is charging a local calendar date with any positive active interval, once (including activation date); alternative next-day effective or precise-hour proration. If activated and revoked on same local day, candidate any-active-interval charges one day; exact price effect *must not become binding without user-approved policy*. Dates of DST changes use timezone-aware boundaries, not 24h = one calendar day.
- Monthly Seat billing calendar window and **one-calendar-month negative-balance allowance** are **different clocks**. Anniversary vs named negative-month, grace expiry, auto suspension and timezone are MR-02, remain OPEN. Never calculate estimated credit limit or pre-charge unapproved speculative Seat usage.
- For a late correction to an already posted month: append `charge_adjustment` / reversal referencing original source occurrence, policy version, company and reason; do not UPDATE/DELETE old Wallet entries. AR-009 delayed-offline period policy remains OPEN.

## 5. Idempotency, ordering, transaction va audit (`PROPOSED`)

1. Source membership event in owner-local ACID + outbox: `event_id` for delivery, `source_transition_id` and effective `membership_version` for semantic identity; same transition with changed payload = CONFLICT.
2. Billing inbox dedup source transitions; sort/reconcile by authoritative version/effective time, not arrival time. Replays, reconnect, branch/device duplicates create no new Seat.
3. Canonical charge uniqueness candidate: `(billing_account_id, tenant_id, company_id, user_id, accrual_day_local, seat_policy_version, charge_component_version)`; if daily snapshot and monthly posting mixed, use *one canonical economic occurrence* per day and a summary of those events, **never charge both**.
4. Owner-local ledger post atomic with unique `charge_occurrence_id`; retries at-least-once delivery, at-most-once economic debit per canonical occurrence (where durable dedup enforced); mismatch -> quarantine/reconciliation.
5. Calendar day cutoff or late membership correction after close creates explicit current open posting-period adjustment linked to original effective period only if approved AR-009 policy; never silently backdate.
6. SaaS ledger charges *actual* approved billable intervals; no guessed payer, cross-company wallet deduction, speculative usage, or extra negative overdraft limit.

## 6. UX va acceptance test specification — NOT EXECUTED

| Scenario | Proposed acceptance oracle |
|---|---|
| Invite → accept → ACTIVE on month day 11 | No pending-invite charge; day policy version explains first chargeable day |
| User active Company A Branch 1+2 and 3 devices | Exactly one economic user×Company Seat; independent of role/device/branch |
| Same user active Company A+B | Exactly two distinct scoped Seats; each Company's own Wallet/legal debtor |
| Concurrent membership changes, duplicated/out-of-order events | Deterministic intervals, source-version conflict or retry; no double charge |
| Suspend → reactivate → remove | Security deny independent from commercial license; no charge outside approved eligible intervals |
| Company A→B transfer, multiple month/timezone boundary | No accidental old future charges; old historical liability preserved |
| Midday activation/deletion, Feb 28/29, month 30/31, DST | Explicit approved local-day policy; fixed-point amounts and rounding reproducible |
| Negative Wallet month + top-up + late correction | Only actual effects; one-calendar-month rule and no estimated ceiling preserved |
| Revoked session/device retries an offline mutation | Server rejection/UNKNOWN recovery; no client charge event |
| Rate change + tenant finance report | Immutable exact pricing snapshot, no silent re-rating, clear Seat count / payable UI |

All tests require implementation and fixtures; this ADR claims neither executed tests nor performance success.

## 7. Xavflar, boshqa modullarga ta'siri, status

Risks: counting duplicate user under multiple memberships, incorrectly counting invitation/suspend, same-day billing disputes, double posting across daily/monthly summary, cross-company Wallet debt contamination, out-of-order late events and locale-dependent dates. 01 Billing coordinates with 02 Multi-Tenant (facts), 10 Security (revoke), 04 Backend (events/outbox), 06 Finance (LegalEntity and ledger separation), 08 UX (charge explanation), 09 Mobile (late offline), 00 Master governance.

**Status:** ADR `PROPOSED`; user-approved Seat scope only as linked from ADR-BILL-011. Existing ADR-BILL-010 remains (earlier alternatives and provenance retained); this document is a refined candidate, not silent blanket approval.

**OPEN:** activated/daily proration economic authorization, month/cutoff, suspend paid-state, same-day transition, refund/posting cadence, exact prices, rounding, enterprise tiers, company change time, legal/tax details, AR-013 membership owner technical sign-off and runtime acceptance. No application code should be started on these unresolved billing rules.
