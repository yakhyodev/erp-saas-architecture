# ADR-MT-002 — Tenant hierarchy va global membership

**Status:** PROPOSED  
**Date:** 2026-10-11  
**Owner:** Multi-Tenancy Architecture

## Muammo
Bitta global user bir nechta tenantda ishlashi, company/branch permission scopes va billing attributionni aniqlash kerak.

## Variantlar
1. User-per-tenant — oddiy authorization, ammo identity takrorlanadi.
2. Global User + TenantMembership — yagona identity, mustaqil tenant permissionlari.
3. Company = Tenant — sodda, lekin ko‘p kompaniyali tenant modelini cheklaydi.

## Tanlov (PROPOSED)
Global User + TenantMembership; Tenant → Company → Branch; tenant/company/branch/resource bo‘yicha server authorization. User bir tenantda aktiv bo‘lishi boshqasiga ruxsat bermaydi.

## Sabab
Identity takrorlanmaydi; har tenant uchun mustaqil permission; yuridik company izolatsiya boundarydan ajratiladi.

## Xavflar
Context leak tenant switchda; sekin membership revoke; branch inheritance; Billing Seat metriği noaniqligi; Company = Tenant bo‘yicha boshqa modul bilan ziddiyat.

## Kamaytirish
Fail-closed context, membershipni qayta tekshirish va revocation, audit, negative isolation tests, company/branch ownership validation; Billing/Finance tasdig‘isiz ownershipni hardcode qilmaslik.

## Ta’siri
Identity/Security membership va scope; Billing Seat hamda billing_account mapping (OPEN); Finance legal company; Sales/Inventory filiallar; Backend/UX tenant selector.

## Tasdiqlash sharti
Master, Finance, Billing va Security bilan hierarchy, ownership, Seat va scope semantikasini tasdiqlash. Ungacha **PROPOSED**.
