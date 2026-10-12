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

## Round 2 addendum — UX/read performance and proof (2026-10-12; PROPOSED)
Read-only bounded tenant-filtered summary projections, indexed/keyset queries and explicitly reported as_of/projection_lag; owner endpoint for read-your-write. Authorized global search filters **per result** using current membership, grant, company/branch and resource scope; revoked rights must not leak through autocomplete, counts or cache. Correlation/causation trace request→outbox→effect; metrics for latency, outbox age, retries, duplicates, DLQ, auth rejects and projection staleness with privacy/cardinality controls. Extraction justified by measured bottlenecks and independent ownership, not immediate microservices. SLO/latency numeric targets and tests OPEN/UNVERIFIED. Details: [Backend §18.6–18.7](../modules/BACKEND_ARCHITECTURE.md).
