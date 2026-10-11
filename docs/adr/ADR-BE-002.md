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
