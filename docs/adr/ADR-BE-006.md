# ADR-BE-006 — Event contracts and outbox

- **Holat:** PROPOSED
- **Loyiha:** Sales doc copy — ERP SaaS
- **Yo‘nalish:** Backend Architecture
- **Manba:** [BACKEND_ARCHITECTURE.md](../modules/BACKEND_ARCHITECTURE.md)

- **Muammo:** event publish va DB commit orasida data loss.
- **Variantlar:** direct publish; polling; outbox + relay.
- **Tanlov (PROPOSED):** transactional outbox + versioned events + idempotent consumer.
- **Sabab:** publish failurelarga chidamlilik.
- **Xavf:** duplicate, ordering, schema governance va replay murakkabligi.
- **Ta'sir:** barcha event-producing modullar.
