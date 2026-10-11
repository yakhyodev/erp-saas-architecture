# ADR-BILL-003 — Usage event idempotency

- **Status:** PROPOSED
- **Date:** 2026-10-11
- **Scope:** Billing Architecture
- **Business-rule authority:** APPROVED biznes invariantlari [Billing module](../modules/BILLING_ARCHITECTURE.md) da; bu ADR texnik usulni tasdiqlamaydi.

## Muammo
At-least-once delivery, offline replay va retried event ikki marta haq chiqarishi mumkin.

## Ko'rilgan variantlar
Synchronous billing call; exactly-once transportga ishonish; inbox/outbox + unique economic posting.

## Tanlangan yechim (taklif)
PROPOSED: transactional outbox/inbox, tenant+producer+source event+charge-type dedup, unique constraints, durable retry.

## Sabab
Tarmoq va worker xatosida iqtisodiy exactly-once effect.

## Xavflar va ochiq masalalar
Source ID o'zgarishi, document amendment va out-of-order event; compensation policy OPEN.

## Boshqa modullarga ta'siri
Sales, Inventory, Mobile, Backend, Database.

## Qaror tartibi
Foydalanuvchi alohida tasdiqlamaguncha PROPOSED. O'zgartirilsa eski ADR SUPERSEDED sifatida tarixda qoladi.

## AR-009 / AR-010 clarification (PROPOSED)
See [Billing §19.2–19.3](../modules/BILLING_ARCHITECTURE.md#192-ar-010--mr-04--document-type--transition--charge-policy). Distinguish source business transition ID, source revision, semantic billable occurrence ID and transport event ID. Suggested unique economic key combines tenant, producer, billable occurrence and charge component/version; retransmission of identical payload is replay-safe, mismatched payload is conflict. Offline device does not post charges; only accepted backend transition can qualify. Document create/confirm/post/finalize/return charge triggers and late usage period policy remain OPEN. Compensation event references original charge. Required tests: repeated import, duplicate device sync, amended document, two workers, out-of-order reversal.
