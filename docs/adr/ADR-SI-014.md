# ADR-SI-014 — Round 2 Domain Ownership & Party Identity

**Status: PROPOSED** | 2026-10-12 | AR-002, AR-008, MR-07

## Muammo
Catalog/Product/SKU, Pricing, Sales, Inventory, Purchasing va CRM assortmentni bitta Sales authorityga birlashtirish product master driftga olib keladi. Customer, Outlet, Payer va legal Debtor o‘rtasidagi semantik farqlar saqlanishi shart.

## Variantlar
A. Sales barcha product/order/stockga egalik qiladi — sodda ko‘ringani bilan xavfli coupling.
B. Catalog, Pricing, Sales, Inventory, Purchasing va CRM bounded contextlari alohida yozuvchi — tavsiya PROPOSED.
C. Mustaqil deployable microservices — erta murakkablik.
Catalog namespace: tenant-only; shared global+mapping; shared core+company overlays — OPEN.

## Tanlangan taklif va sabab
B: Catalog Product/SKU/UoM master; Pricing PriceList; Sales SalesOrder; Inventory StockMovement/Balance/Reservation; Purchasing PO/receipt intent; CRM customer relationship/outlet/assortment. Finance receivable obligation va posted invoice debtor snapshotining yagona yozuvchisi. CRM Party canonical master yoki shared Party context tanlovi OPEN. Payer remitter va Outlet debtor bilan aynan bir emas. Shu chegaralar audit va source authorityni saqlaydi.

## Xavflar
Tenant/global SKU migration, external mapping, Party merge, filiallararo access; versioned contract, immutable scoped ID, alias lineage, auditable reconciliation talab etiladi.

## Boshqa modullarga ta'siri
Backend ownership matrix; CRM assortment va Party; Finance AR/debtor; Billing SaaS payer; Multi-Tenant scope; Integration ID mapping; UX-T02/T08.

## OPEN / Approval gate
AR-002/AR-008 uchun 04 Backend, 05 Sales/Inventory, 06 Finance, 07 CRM va foydalanuvchi katalog scope va canonical Party writerini tanlaydi. Hozircha APPROVED emas. [Round 2 details](../modules/SALES_INVENTORY.md#12-round-2-correction--2026-10-12-ar-002005008010-mr-0708).
