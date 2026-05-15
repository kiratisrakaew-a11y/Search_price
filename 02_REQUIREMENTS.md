# 01 Project Overview

## Purpose

This project builds a Phase 1 BOQ / construction / service / material price reference search system using Google Sheets, Google Apps Script, and a simple WebApp.

The system normalizes multiple source sheets with different structures into a single searchable `MASTER_PRICE_DATABASE`. Users can search price items, select a result manually, enter their own price/unit/quantity, and compare it against the reference price.

## Main Goals

1. Build a reliable master price database from multiple sources.
2. Preserve raw source data and prevent unsafe overwrites.
3. Support practical search across item names, categories, notes, keywords, aliases, and normalized text.
4. Allow users to choose search results manually.
5. Compare user-provided price against the selected database reference.
6. Support unit conversion using rule-based logic first, then Gemini only for complex unit interpretation.
7. Maintain auditability through refresh and search logs.
8. Keep Codex implementation strictly within Phase 1 scope.

## Confirmed Phase 1 Scope

Phase 1 includes:

- Source refresh/process actions.
- Normalization into `STAGING_NORMALIZED`.
- Validation before master updates.
- Source-specific replacement into `MASTER_PRICE_DATABASE`.
- `REFRESH_LOG`.
- Search from `MASTER_PRICE_DATABASE` only.
- `SEARCH_LOG`.
- `ALIAS_DICTIONARY` enrichment.
- Simple WebApp for search + compare.
- User manual selection of search result.
- Unit matching and unit conversion.
- Gemini-assisted unit conversion only when rule-based conversion is insufficient.
- Price comparison display.
- Google Sheets custom menu for admin/manual actions.
- Testing/check routines.

Phase 1 excludes:

- Login.
- Role-based permission.
- Approval workflow.
- Full admin panel.
- Dashboard.
- Export report.
- `COMPARISON_LOG`.
- Auto-approve price.
- Auto-reject price.
- Auto-update master price from WebApp.
- Auto-learn alias directly into master.
- AI search engine.
- WebApp editing of master/source/staging sheets.

## Data Flow

```text
RAW SOURCE SHEETS
→ STAGING_NORMALIZED
→ MASTER_PRICE_DATABASE
→ WEBAPP SEARCH
→ USER SELECTS RESULT
→ USER INPUTS PRICE / UNIT / QUANTITY / PRICE TYPE
→ UNIT MATCHING / CONVERSION
→ PRICE COMPARISON RESULT
```

## Operational Source Sheets

1. `laborcost_cgd`
2. `laborcost_obec`
3. `materialcost_obec`
4. `materialcost_tpso`

## Processing / Control Sheets

1. `STAGING_NORMALIZED`
2. `MASTER_PRICE_DATABASE`
3. `ALIAS_DICTIONARY`
4. `REFRESH_LOG`
5. `SEARCH_LOG`
6. `ALIAS_SUGGESTIONS` if alias suggestion process is implemented

## Documentation Sheet

- `CHECKLIST_2_SCHEMA`

## External Services

1. TPSO API, used only to update `materialcost_tpso`.
2. Gemini, used only for complex unit conversion support.

## Implementation Principle

The master database must be treated like a warehouse ledger, not a scratchpad. Nothing should overwrite it unless new data has passed staging and validation.
