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

## Round 2 UX correction — AR-012 / MR-16/17/20 (PROPOSED)

Round 2 MR-17/20: Large-table acceptance uses candidate 20k orders/100k products, two tenants, three branches; authoritative filter/sort/pagination and permission-safe totals/facets/export. Loading skeleton must never show previous tenant rows; 403/409/timeout preserve safe state. Client virtualization optional; exact dataset/latency targets and largest-tenant budgets remain OPEN.

Details: [UX architecture §17](../modules/UX_ARCHITECTURE.md). No tests performed; no approved new technical decision.
