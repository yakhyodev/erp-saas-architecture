# ADR-BILL-006 — Negative balance lifecycle without forecast limit

- **Status:** PROPOSED
- **Date:** 2026-10-11
- **Scope:** Billing Architecture
- **Business-rule authority:** APPROVED biznes invariantlari [Billing module](../modules/BILLING_ARCHITECTURE.md) da; bu ADR texnik usulni tasdiqlamaydi.

## Muammo
Balans tugagach tasdiqlangan bir kalendar oy haqiqiy xarajatlarni qanday boshqarish kerak?

## Ko'rilgan variantlar
Hard prepaid stop; forecast credit threshold; real cost negative period with policy state.

## Tanlangan yechim (taklif)
PROPOSED: actual Seat/Usage posting throughout approved calendar month; no estimated cap; configurable status machine only after OPEN edge cases approved.

## Sabab
APPROVED biznes qoidalariga mos; taxminiy limit qo'shmaydi.

## Xavflar va ochiq masalalar
Calendar-month start, timezone, renewal, after-expiry blocks va partial top-up hammasi OPEN.

## Boshqa modullarga ta'siri
IAM, Tenant, Notifications, Legal, Support.

## Qaror tartibi
Foydalanuvchi alohida tasdiqlamaguncha PROPOSED. O'zgartirilsa eski ADR SUPERSEDED sifatida tarixda qoladi.

## AR-009 / MR-02 negative-balance clarification (PROPOSED)
See [Billing §19.3–19.4](../modules/BILLING_ARCHITECTURE.md#194-mr-02--one-calendar-month-negative-balance-timeline). APPROVED: actual Seat/Usage charges for **one calendar month** in minus, no estimated ceiling. Calendar-month meaning, timezone, first-negative trigger, end, partial refill, service expiry and repeat crossing remain OPEN. Suggested half-open start/end and versioned timezone/period policy only after approval. Offline late event must preserve event effective period and immutable posted_at separately. Do not equate calendar month to 30 rolling days, do not forecast/debt-cap or silently suspend at month end.

## Round 2 clarification — MR-02 / AR-009 (2026-10-12, PROPOSED)
Compare *named month* (Oct 28 negative ends Nov 1 00:00 if current named October; Jan 31 ends Feb 1) vs *anniversary calendar month* (Oct 28→Nov 28; Jan 31→Feb 28 if clamp rule); named **next** month and month-end rollforward are distinct OPEN options. Rolling 30 days is not equivalent. See [Billing §20.2](../modules/BILLING_ARCHITECTURE.md). Store IANA timezone, immutable first_negative_event and first_negative_posted_at, policy version, inclusive start / exclusive UTC end, verified top-up, recovery, repeat and expiry events. Closed-month adjustments are linked current-period postings, not overwritten entries. **APPROVED:** only actual Seat and Usage costs can go negative for one calendar month; **NO estimated credit ceiling**. First crossing, partial/full refill behavior, repeated negative windows, post-expiry access and late offline attribution all OPEN. Anniversary clamp only a review candidate, **not APPROVED**.

## User-selected candidate follow-up (2026-10-12, PROPOSED)
Anniversary-based **calendar month** chosen for discussion only: Oct 28 16:00→Nov 28 16:00; Jan 31 10:00→Feb 28 10:00 only if last-day **clamp** approved (roll-forward remains alternative). Named-month current/next alternatives remain documented in [Billing §20.2/§21](../modules/BILLING_ARCHITECTURE.md). OPEN: IANA timezone owner, exact first-negative crossing, top-up reset, full/partial recovery, repeat deficit, late/closed period, expiry and post-expiry access. **APPROVED nonnegotiable**: only actual Seat and Usage debits for one calendar month; NEVER an estimated credit ceiling or forecast-based blocking.
