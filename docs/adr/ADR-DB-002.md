# ADR-DB-002 — Shared schema default / hybrid future

> Status: **PROPOSED**  
> Scope: Database Architecture  
> Source: [DATABASE_ARCHITECTURE.md](../modules/DATABASE_ARCHITECTURE.md)  
> Tasdiqlangan biznes talablari bilan chalkashtirilmasin. Ushbu texnik qaror hali tasdiqlanmagan.

- **Status:** PROPOSED.
- **Muammo:** 1000+ tenantni boshqarish narxi va isolation o‘rtasidagi balans.
- **Variantlar:** shared schema; schema-per-tenant; DB-per-tenant; hybrid.
- **Tanlov:** shared schema + tenant_id; enterprise uchun dedicated DB escape hatch.
- **Sabab:** migration/pooling/monitoring soddaligi va tejamkor multi-tenant density.
- **Xavf:** isolation bug, noisy neighbor, restore complexity.
- **Ta’sir:** Multi-Tenant, Security, Infrastructure, Billing.
