# ADR-INT-001 — Anti-Corruption Adapter Layer

- Date: 2026-10-11
- Status: **PROPOSED** (NOT APPROVED)
- Owner: Integration Architecture

## Muammo
External provider semantics, IDs and transport models must not leak into ERP domains.

## Taklif etilgan contract
Provider adapter owns:
- authentication envelope;
- transport errors and rate limits;
- provider contract version;
- evidence level (DOCUMENTED/LIVE_VERIFIED/INFERRED/UNVERIFIED);
- normalized output for domain services.

Domain modules own business validation and posting decisions.

## Variantlar
Direct module connectors; source API mirror; anti-corruption adapter.

## Tanlangan PROPOSED yechim
Provider-neutral adapter with versioned canonical import/command contracts.

## Xavflar
Mapping complexity and central orchestration growth.

## Ta'sir
Backend, CRM, Sales, Inventory, Finance, Security.

## Status
No implementation authorization. Requires domain review.
