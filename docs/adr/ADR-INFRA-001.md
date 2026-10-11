# ADR-INFRA-001: Bosqichma-bosqich deployment

- **Status:** PROPOSED
- **Date:** 2026-10-11
- **Owner:** Infrastructure & Scaling
- **Related module:** [INFRA_ARCHITECTURE.md](../modules/INFRA_ARCHITECTURE.md)

## Muammo
Small initial budget versus future horizontal scaling.

## Ko‘rib chiqilgan variantlar
VM plus containers; Kubernetes from day one; serverless; dedicated bare metal.

## Tanlangan yechim — PROPOSED
Start with simple container host/minimal VMs; move to load balancer and N stateless app instances when measured.

## Sabab
Less operational overhead and portable releases.

## Xavflar va cheklovlar
First stage has single point of failure and manual operational burden. Numeric SLO/RPO/RTO and vendor selection are not approved.

## Boshqa modullarga ta’siri
Backend, Security, Frontend, Operations.

## Tasdiqlash sharti
Cross-module review and explicit user approval; until then not an approved infrastructure decision.
