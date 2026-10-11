# ADR-MOB-001 — Offline-first model

Status: **PROPOSED** | Date: 2026-10-11 | Owner: Mobile & Offline

## Muammo
Offline-first model bo‘yicha ko‘p tenantli mobil ERP xavfsizligi va ishonchliligini ta’minlash.

## Variantlar
Online-only / full replica / scoped offline-first.

## Tanlangan yechim (taklif)
Scoped offline-first, server-authoritative posting.

## Sabab
Works without connectivity; avoids global replicas.

## Xavflar
Stale data, delayed sync.

## Boshqa modullarga ta’siri
Backend, Sales, Finance, Billing.

## Qaror chegarasi
APPROVED emas. Mavjud tasdiqlangan biznes qoidalarini o‘zgartirmaydi. Boshqa domenlar bilan kelishish kerak.

[Mobile Architecture](../modules/MOBILE_ARCHITECTURE.md)
