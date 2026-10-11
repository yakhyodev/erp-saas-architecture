# ADR-UX-003 — Universal search + commands

**Status:** PROPOSED
**Source:** [UX_ARCHITECTURE.md](../modules/UX_ARCHITECTURE.md)

- **Status:** PROPOSED
- **Muammo:** ma'lumot turiga qarab turli sahifalarda izlash kerak.
- **Variantlar:** har modulda alohida qidiruv; yagona search natijalari; search + command palette.
- **Tanlov:** ruxsatga asoslangan global search + alohida belgilangan commands.
- **Sabab:** ma'lumot yoki amalni kirish nuqtasidan mustaqil topish.
- **Xavf:** indeks kechikishi, permission leak, qidiruv xarajati.
- **Ta'sir:** Backend, Database, Security, audit, Resource Usage (pricing doirasini billing moduli belgilaydi).
