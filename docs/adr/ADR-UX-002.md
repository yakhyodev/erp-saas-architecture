# ADR-UX-002 — Role-aware UI, server-authorized actions

**Status:** PROPOSED
**Source:** [UX_ARCHITECTURE.md](../modules/UX_ARCHITECTURE.md)

- **Status:** PROPOSED
- **Muammo:** hamma rolga bir xil murakkab ekran berish va UI permission bilan authorizationni adashtirish.
- **Variantlar:** statik ekran; frontend-only role hide; permission-aware server-authorized presentation.
- **Tanlov:** permission-aware personalizatsiya, barcha huquq serverda tekshiriladi.
- **Sabab:** moslik va himoya.
- **Xavf:** permission tizimiga kuchli bog'liqlik, cache eskirishi.
- **Ta'sir:** Auth/RBAC, tenant scope, search, offline.

## AR-012 active-context acceptance — PROPOSED

Role is presentation, not permission. Tenant/company/branch switch reauthorizes and invalidates cached cards, counts, suggestions, recents, files and in-flight views before repaint. Offline pending commands never migrate scope. Denied deep links, revoked membership and unauthorized All-branches must reveal no foreign record. User/owner decisions AR-001 and AR-013 remain OPEN. See [UX §16](../modules/UX_ARCHITECTURE.md), [Security](../modules/SECURITY_ARCHITECTURE.md), [Mobile](../modules/MOBILE_ARCHITECTURE.md).
