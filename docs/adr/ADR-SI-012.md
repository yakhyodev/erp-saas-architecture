# ADR-SI-012: Typed document transition to physical stock and Finance GL fact boundaries

- **Status:** PROPOSED — not an approval
- **Date:** 2026-10-11
- **Harmonization:** [SALES_INVENTORY.md §11](../modules/SALES_INVENTORY.md)

## Muammo
AR-005 / MR-08: generic SalesDocumentPosted may double-count stock or Finance GL at confirmation, dispatch or invoicing.

## Ko‘rib chiqilgan variantlar
A: one generic posting triggers stock+GL; B: typed transitions with Inventory stock authority and Finance GL authority; C: all effects in one cross-context DB transaction.

## Tavsiya etilgan yechim
B as proposed semantic model: Draft no effects; confirm no physical stock; reserve only available; dispatch Inventory debit and consume reservation; goods receipt Inventory credit; invoice Finance-led GL only, no second stock; returns/reversals compensating facts. Cross-context atomicity (single DB versus outbox) remains OPEN.

## Sababi
Distinct typed facts and idempotency boundaries prevent double posting. Inventory local movement+balance+reservation+outbox transaction proposed; Finance owns journal idempotent processing.

## Xavflar va mitigatsiya
Financial recognition timing and valuation OPEN, downstream event delay, retry and reconciliation risk.

## Boshqa modullarga ta’siri
Database, Backend, Finance, Sales, Purchasing, Billing, Offline.

## Open / approval gate
05+06+04+03 approve GL policy and effect timing. AR-005 remains OPEN.

**Existing APPROVED business invariants untouched. Go/PostgreSQL/Next.js/Flutter/modular monolith remain PROPOSED. No code.**
