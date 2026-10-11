# ADR-MOB-008 — Offline stock and billing consistency

Status: **PROPOSED** | Date: 2026-10-11 | Owner: Mobile & Offline

## Muammo
Offline stock and billing consistency bo‘yicha ko‘p tenantli mobil ERP xavfsizligi va ishonchliligini ta’minlash.

## Variantlar
Client posting / provisional / server posting.

## Tanlangan yechim (taklif)
Local stock informational only; server finalizes reservations, finance postings and usage once.

## Sabab
Ledger consistency.

## Xavflar
Late usage period unresolved.

## Boshqa modullarga ta’siri
Inventory, Finance, Billing.

## Qaror chegarasi
APPROVED emas. Mavjud tasdiqlangan biznes qoidalarini o‘zgartirmaydi. Boshqa domenlar bilan kelishish kerak.

[Mobile Architecture](../modules/MOBILE_ARCHITECTURE.md)

## AR-009 reconciliation — OPEN QUESTION (Mobile + Billing)

Offline business evidence can be created in one calendar month and received or accepted in another. Store separate `client_occurred_at` (untrusted), `server_received_at`, `server_accepted_at`, domain-authoritative chargeable event time (once AR-010 is settled), and Billing-assigned `rated_calendar_month`. Use server-side deduplication and audit to prevent replay charges.

**Attribution alternatives, not approved:** (A) trusted business occurrence month with backdated adjustments; (B) server acceptance/chargeable lifecycle month; (C) server receipt month with corrections. Billing must decide timezone, closing, late event, reversal, correction and discount tier effects; Sales/Inventory must confirm chargeable document transition. Mobile does not select a rating month or independently approve business usage charges. Existing approved one-calendar-month actual Seat/Usage negative-balance policy is unchanged.

[Central open questions](../../OPEN_QUESTIONS.md) — AR-009 / AR-010.
