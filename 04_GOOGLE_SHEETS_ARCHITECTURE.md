# 04 Google Sheets Architecture

## Workbook Structure

Operational sheets:

1. `laborcost_cgd`
2. `laborcost_obec`
3. `materialcost_obec`
4. `materialcost_tpso`
5. `STAGING_NORMALIZED`
6. `MASTER_PRICE_DATABASE`
7. `ALIAS_DICTIONARY`
8. `REFRESH_LOG`
9. `SEARCH_LOG`
10. `ALIAS_SUGGESTIONS` if alias suggestion process is implemented

Documentation sheet:

11. `CHECKLIST_2_SCHEMA`

## Sheet Roles

### `laborcost_cgd`

- Raw source sheet.
- Manual update.
- Yearly update.
- Input only.
- WebApp must not read directly.

### `laborcost_obec`

- Raw source sheet.
- Manual update.
- Yearly update.
- Input only.
- WebApp must not read directly.

### `materialcost_obec`

- Raw source sheet.
- Manual update.
- Yearly update.
- Input only.
- WebApp must not read directly.

### `materialcost_tpso`

- Raw source sheet.
- API monthly update.
- Input only.
- WebApp must not read directly.
- Header may not be on row 1. Must use special header detection.

### `STAGING_NORMALIZED`

- Processing sheet.
- Used for normalization and validation before master update.
- WebApp must not read directly.

### `MASTER_PRICE_DATABASE`

- Searchable master database.
- WebApp reads from this sheet only.
- Comparison flow reads from this sheet only.
- User should not edit manually.

### `ALIAS_DICTIONARY`

- Dictionary for user terms, canonical terms, and related search terms.
- Used to enrich `alias_terms`.
- Can be edited through controlled manual process.

### `REFRESH_LOG`

- Stores refresh/process logs.
- Used for audit.
- WebApp does not read directly.

### `SEARCH_LOG`

- Stores user search behavior.
- Used to improve alias/search quality later.
- WebApp may write to this sheet.

### `CHECKLIST_2_SCHEMA`

- Documentation only.
- Not a data source.
- WebApp must not read.

## Header Rules

For all database/processing/log sheets:

- Header must be one row only.
- Header should be row 1.
- No field description row in row 2.
- No merged cells.
- No blank row before header.

Exception:

- `materialcost_tpso` may contain metadata before the actual header.
- Script must find real header row by fields such as `Column ID`, `type`, `typeName`.
- If header row cannot be found, stop processing and do not update master.

## Sheet Protection

Manual edit allowed:

- `laborcost_cgd`
- `laborcost_obec`
- `materialcost_obec`
- `ALIAS_DICTIONARY`

System/API update only:

- `materialcost_tpso`
- `STAGING_NORMALIZED`
- `MASTER_PRICE_DATABASE`
- `REFRESH_LOG`
- `SEARCH_LOG`
- `ALIAS_SUGGESTIONS` if implemented

Documentation only:

- `CHECKLIST_2_SCHEMA`

## WebApp Scope

WebApp read:

- `MASTER_PRICE_DATABASE`
- `ALIAS_DICTIONARY` only if necessary for search

WebApp write:

- `SEARCH_LOG` only

WebApp must not write:

- Raw source sheets
- `STAGING_NORMALIZED`
- `MASTER_PRICE_DATABASE`
- `REFRESH_LOG`
- `ALIAS_DICTIONARY`

Phase 1 comparison flow does not write `COMPARISON_LOG`.
