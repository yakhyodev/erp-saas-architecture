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

## Round 2 AR-009 / MR-12 — delayed offline economic occurrence (OPEN)

Distinguish `client_occurred_at` (untrusted device observation), `server_received_at` (ingress), `server_accepted_at` (authoritative command acceptance), domain `occurred_at` (business occurrence after owner validation), `billed_at` (Billing posting) and `rated_calendar_month` (Billing's chosen accounting/rating attribution). Store timezone, event/occurrence identity and correction/reversal linkage. `billed_at` is not automatically the rated month.

**Billing options, all OPEN:** A trusted occurrence month with late adjustments; B accepted/chargeable lifecycle month; C receipt month with corrections. Compare month close, timezone, 30+ day delay, progressive discounts, duplicate/replay, refund/reversal and BIZ-006 actual-cost negative calendar month anchor. No estimated negative credit ceiling. AR-010 document chargeable transition and authoritative source event remain OPEN.

**No duplicate economic effect:** Mobile produces local order/delivery/visit/evidence intents only. Sales confirmation does not automatically decrement stock; Inventory dispatch posts physical movement once under its own occurrence key; Finance posting separate; Billing charges only its approved chargeable server occurrence with unique economic key. GPS/media retransmission never creates a new billable document or dispatch. Owner-specific atomic transaction + outbox and idempotent consumer are proposed cross-module contract; final transaction boundary follows Database/Backend owner approval.
