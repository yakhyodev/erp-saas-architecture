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

## Round 2 UX correction — AR-012 / MR-16/17/20 (PROPOSED)

Round 2 AR-012: Persona dashboard is server-capability scoped, not fixed-role security. Switch state machine clears prior tenant/company/branch data BEFORE new scope authorization. Old queries, counts, previews, cache, notifications and delayed responses may not render in new context. Unauthorized deep links use privacy-safe denial. Test revoke-mid-session and race. Membership/role authority AR-013 remains OPEN.

Details: [UX architecture §17](../modules/UX_ARCHITECTURE.md). No tests performed; no approved new technical decision.
