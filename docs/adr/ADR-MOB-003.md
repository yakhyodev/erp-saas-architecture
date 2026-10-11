# ADR-MOB-003 — Delta synchronization

Status: **PROPOSED** | Date: 2026-10-11 | Owner: Mobile & Offline

## Muammo
Delta synchronization bo‘yicha ko‘p tenantli mobil ERP xavfsizligi va ishonchliligini ta’minlash.

## Variantlar
Full reload / timestamps / scoped cursor delta.

## Tanlangan yechim (taklif)
Durable transactional outbox, opaque scoped delta cursor, tombstones, atomic checkpoints.

## Sabab
Bandwidth and crash recovery.

## Xavflar
Cursor expiry and resync.

## Boshqa modullarga ta’siri
Backend, Database.

## Qaror chegarasi
APPROVED emas. Mavjud tasdiqlangan biznes qoidalarini o‘zgartirmaydi. Boshqa domenlar bilan kelishish kerak.

[Mobile Architecture](../modules/MOBILE_ARCHITECTURE.md)
