# ADR-BE-004 — API-first versioned REST

- **Holat:** PROPOSED
- **Loyiha:** Sales doc copy — ERP SaaS
- **Yo‘nalish:** Backend Architecture
- **Manba:** [BACKEND_ARCHITECTURE.md](../modules/BACKEND_ARCHITECTURE.md)

- **Muammo:** web/mobile/integration evolyutsiyasi.
- **Variantlar:** REST/OpenAPI; GraphQL; gRPC public.
- **Tanlov (PROPOSED):** REST/OpenAPI; gRPC internal extraction payti qayta baholanadi.
- **Sabab:** contract clarity, client/tooling compatibility.
- **Xavf:** chatti endpoints, version compatibility burden.
- **Ta'sir:** Web, Mobile, Integrations, Security.

## Round 2 addendum — MR-10 API compatibility (2026-10-12; PROPOSED)
Public REST/OpenAPI v1 candidate; versioned commands/events independently, contract-first schemas, additive optional fields and unknown-safe enum handling. Major breaking change requires v2 and parallel migration, not in-place reinterpretation. Mutation Idempotency-Key + normalized fingerprint; If-Match/expected revision, missing precondition 428/stale 412; same-key-different-payload 409. Cursor/keyset pagination: bounded limit, stable sort, tenant/filter-bound opaque cursor; invalid 400. Predictable errors: code/message_key/request_id/correlation_id/retryable/field_errors/current_revision, no private data leakage. Deprecation/sunset and mobile offline compatibility windows OPEN. Authorized global search and lightweight read-model staleness markers required, neither projection nor UI a source of authority. Risks: compatibility cost, stale read models. Details: [Backend §18.6](../modules/BACKEND_ARCHITECTURE.md).
