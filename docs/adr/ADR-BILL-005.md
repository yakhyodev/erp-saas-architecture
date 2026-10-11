# ADR-BILL-005 — Verified settlement before wallet credit

- **Status:** PROPOSED
- **Date:** 2026-10-11
- **Scope:** Billing Architecture
- **Business-rule authority:** APPROVED biznes invariantlari [Billing module](../modules/BILLING_ARCHITECTURE.md) da; bu ADR texnik usulni tasdiqlamaydi.

## Muammo
Provider callback, retry va fake payment walletni noto'g'ri oshirishi mumkin.

## Ko'rilgan variantlar
Immediate initiation credit; webhook-only credit; verified settlement + reconciliation.

## Tanlangan yechim (taklif)
PROPOSED: settlement verification, unique provider reference, immutable credit posting, exception queue.

## Sabab
Duplicate/replayed paymentni bloklash va bank mosligini tekshirish.

## Xavflar va ochiq masalalar
Settlement kechikishi, chargeback/refund va provider spesifik farqlar.

## Boshqa modullarga ta'siri
Payments, Security, Finance, Operations.

## Qaror tartibi
Foydalanuvchi alohida tasdiqlamaguncha PROPOSED. O'zgartirilsa eski ADR SUPERSEDED sifatida tarixda qoladi.
