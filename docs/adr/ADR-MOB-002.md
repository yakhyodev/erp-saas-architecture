# ADR-MOB-002 — Scoped encrypted local storage

Status: **PROPOSED** | Date: 2026-10-11 | Owner: Mobile & Offline

## Muammo
Scoped encrypted local storage bo‘yicha ko‘p tenantli mobil ERP xavfsizligi va ishonchliligini ta’minlash.

## Variantlar
Plain cache / encrypted relational store / full replication.

## Tanlangan yechim (taklif)
Encrypted local relational store with assignment-scoped datasets and OS-protected keys.

## Sabab
Fast local reads, least privilege.

## Xavflar
Device loss, stale access.

## Boshqa modullarga ta’siri
Security, Multi-Tenant, Database.

## Qaror chegarasi
APPROVED emas. Mavjud tasdiqlangan biznes qoidalarini o‘zgartirmaydi. Boshqa domenlar bilan kelishish kerak.

[Mobile Architecture](../modules/MOBILE_ARCHITECTURE.md)
