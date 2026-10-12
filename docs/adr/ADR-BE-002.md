# ADR-BE-002 — Modular monolith boundaries

- **Holat:** PROPOSED
- **Loyiha:** Sales doc copy — ERP SaaS
- **Yo‘nalish:** Backend Architecture
- **Manba:** [BACKEND_ARCHITECTURE.md](../modules/BACKEND_ARCHITECTURE.md)

- **Muammo:** monolit kompleksligi va microservice operatsion xarajati.
- **Variantlar:** qatlamli monolit; modular monolith; microservices first.
- **Tanlov (PROPOSED):** bounded contextli modular monolith.
- **Sabab:** transactionlar sodda va deploy operatsiyasi kam, kelajakda extraction mumkin.
- **Xavf:** boundary erosion, tightly coupled queries.
- **Ta'sir:** barcha biznes modullar, Database, Security.

## Round 2 addendum — AR-002 / MR-07 (2026-10-12; PROPOSED)
Canonical writer candidates are **distinct**: Catalog Product/SKU/UoM (tenant vs company scope OPEN), Pricing price rule/quote, Sales commercial order, Inventory reservation and physical movement, Purchasing purchase intent, Finance ERP GL, Billing SaaS charge. CRM owns Customer/Outlet relationships; canonical Party writer is **OPEN** between CRM-owned registry and shared Party context. Old broad "Sales owns products/inventory" wording is an alternative/disputed historic proposal, not authority to merge writers. Consumers reference versioned IDs/read ports and immutable price/UoM snapshots, never mutate another owner's store. Alternatives: one Sales super-module (simpler at first, boundary erosion), separated logical boundaries (recommended), early microservices (high operations cost). Risks: cross-owner latency, late projection, duplicate aliases. Effects: Sales, CRM, Finance, Inventory, Integration, Database. User approval of Catalog/Party scope remains OPEN. Details: [Backend §18.2](../modules/BACKEND_ARCHITECTURE.md).
