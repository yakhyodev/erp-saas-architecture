# ADR-SI-011: Canonical Product/SKU/Catalog/Pricing/Sales/Inventory/Purchasing ownership

- **Status:** PROPOSED — not an approval
- **Date:** 2026-10-11
- **Harmonization:** [SALES_INVENTORY.md §11](../modules/SALES_INVENTORY.md)

## Muammo
AR-002 / MR-07: CRM ADR Sales products owner vs Backend/Sales independent Catalog boundary.

## Ko‘rib chiqilgan variantlar
A: Sales owns all product fields; B: separate Catalog bounded context within 05; C: standalone Catalog module; catalog namespace options global, tenant or shared+overlays.

## Tavsiya etilgan yechim
B as logical ownership proposal only: Catalog alone mutates category/product/SKU/UoM; Pricing owns prices; Sales orders; Inventory stock; Purchasing POs; CRM customer assortment/agent/outlet relationships; Finance GL; Billing SaaS rating. Physical packaging and deployment undecided.

## Sababi
Exclusive write authority avoids double mutation, SKU identity drift and private-table dependencies.

## Xavflar va mitigatsiya
Global vs tenant catalog; duplicate mappings, ownership of customer party still AR-008; cross-company visibility OPEN.

## Boshqa modullarga ta’siri
Backend, CRM, Inventory, Purchase, Pricing, Finance, Multi-Tenant, Integrations.

## Open / approval gate
Approve global/tenant catalog and CRM relationship with 05+07+04; AR-002 remains OPEN.

**Existing APPROVED business invariants untouched. Go/PostgreSQL/Next.js/Flutter/modular monolith remain PROPOSED. No code.**
