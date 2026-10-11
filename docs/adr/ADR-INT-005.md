# ADR-INT-005 — Contragent/SalePoint Normalization

- Date: 2026-10-11
- Status: **PROPOSED** (NOT APPROVED)
- Owner: Integration Architecture

## Muammo
Ordinary vs contragent server changes which party owns payments and debt.

## Variantlar
Flatten clients; mirror source server-mode flag; separate payer and sale point.

## Taklif etilgan yechim
Represent payer and sale point separately on affected transactions, subject to Finance and CRM approval.

## Sabab
Avoid source-specific accounting assumptions, preserve both models.

## Xavflar
Duplicates, orphan payer and debt mismatch.

## Boshqa modullarga ta'siri
CRM, Finance, Sales, Multi-Tenant.

## Qaror sharti
Owning domain review required; no implementation authorization. See [main architecture](../modules/INTEGRATION_ARCHITECTURE.md).
