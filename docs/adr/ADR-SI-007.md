# ADR-SI-007: Posted hujjat immutability

- **Status:** PROPOSED (tasdiqlanmagan)
- **Date:** 2026-10-11
- **Domain:** Sales & Inventory
- **Source:** [SALES_INVENTORY.md](../modules/SALES_INVENTORY.md)

## Muammo
Postingdan keyin tahrir/del ikki marta stock effektiga sabab bo‘lishi mumkin.

## Variantlar
Mutable posted docs; compensation/reversal docs.

## Taklif etilgan qaror
Posted document immutable; korreksiya kompensatsion hujjat bilan.

## Sababi
Audit trail saqlanadi va billing charge mapping aniqlashtiriladi.

## Xavflar va mitigation
Reversal UX va bog‘liq accounting correction murakkab.

## Boshqa modullarga ta’siri
Finance, Billing, Audit, UX.

## Tasdiqlash
Bosh arxitektura, tegishli domen egalari bilan reviewdan keyin APPROVED qilinadi. Hozircha implementation mandate emas.

## 2026-10-11 effect and reversal clarification (PROPOSED)
Posted stock document reversal creates linked compensating physical movement via Inventory only if original physical leg exists. Finance reverses its own journal only if actually posted; Billing determines independently whether a charge adjustment/refund is valid. Neither GL nor usage charge is automatically implied by stock reversal; no second stock debit on invoice. Canonical source fact/leg and original_fact_id+reversal_id make retries idempotent. See [ADR-SI-012](ADR-SI-012.md), [ADR-SI-013](ADR-SI-013.md). AR-005/010 remain OPEN.
