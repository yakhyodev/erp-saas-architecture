# ADR-MOB-007 — Media upload lifecycle

Status: **PROPOSED** | Date: 2026-10-11 | Owner: Mobile & Offline

## Muammo
Media upload lifecycle bo‘yicha ko‘p tenantli mobil ERP xavfsizligi va ishonchliligini ta’minlash.

## Variantlar
Inline upload / queue / resumable.

## Tanlangan yechim (taklif)
Encrypted media queue, checksums, retry, resumable large uploads, server acknowledgement.

## Sabab
Resilience.

## Xavflar
Orphan media, storage limits.

## Boshqa modullarga ta’siri
Backend, Infrastructure.

## Qaror chegarasi
APPROVED emas. Mavjud tasdiqlangan biznes qoidalarini o‘zgartirmaydi. Boshqa domenlar bilan kelishish kerak.

[Mobile Architecture](../modules/MOBILE_ARCHITECTURE.md)
