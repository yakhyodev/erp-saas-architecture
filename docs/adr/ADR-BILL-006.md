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
