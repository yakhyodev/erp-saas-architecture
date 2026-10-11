# ADR-UX-005 — Server-driven data views

**Status:** PROPOSED
**Source:** [UX_ARCHITECTURE.md](../modules/UX_ARCHITECTURE.md)

- **Status:** PROPOSED
- **Muammo:** katta qatorlar brauzer xotirasi va latencyga zarar qiladi.
- **Variantlar:** full client table; infinite scroll; server query + paginated views.
- **Tanlov:** server-driven filtering/sorting/pagination; virtualization optional.
- **Sabab:** predictable resurs, tez ochilish, tenant-permission nazorati.
- **Xavf:** qiyin aggregate, filtr semantikasi, eksport consistency.
- **Ta'sir:** Backend, Database, Reporting, Security.
