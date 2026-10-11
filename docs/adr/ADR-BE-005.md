# ADR-BE-005 — Worker & queue architecture

- **Holat:** PROPOSED
- **Loyiha:** Sales doc copy — ERP SaaS
- **Yo‘nalish:** Backend Architecture
- **Manba:** [BACKEND_ARCHITECTURE.md](../modules/BACKEND_ARCHITECTURE.md)

- **Muammo:** reliable deferred processing, burst control.
- **Variantlar:** synchronous-only; DB-backed queue; Redis Streams; RabbitMQ; Kafka.
- **Tanlov (PROPOSED):** initial DB-backed durable jobs + independent workers; load bo'yicha upgrade.
- **Sabab:** boshida kam infratuzilma, DB bilan transaction yaqinligi.
- **Xavf:** job polling DB bosimi, throughput chegarasi.
- **Ta'sir:** Billing, Notifications, Integrations, Infrastructure.
