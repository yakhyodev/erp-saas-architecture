# ADR-BE-008 — Observability and controlled extraction

- **Holat:** PROPOSED
- **Loyiha:** Sales doc copy — ERP SaaS
- **Yo‘nalish:** Backend Architecture
- **Manba:** [BACKEND_ARCHITECTURE.md](../modules/BACKEND_ARCHITECTURE.md)

- **Muammo:** ko'p tenantda production muammo tashxisi va scale.
- **Variantlar:** ad hoc logs; structured observability + SLO; microservices immediately.
- **Tanlov (PROPOSED):** logs+metrics+traces+audit, migration mezonlari bilan bosqichma-bosqich extraction.
- **Sabab:** qarorlarni real dalillarga tayash.
- **Xavf:** telemetry xarajatlari, PII leakage, metric cardinality.
- **Ta'sir:** Infrastructure, Security, Master Architecture.
