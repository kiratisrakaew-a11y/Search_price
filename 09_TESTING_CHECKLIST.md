# 09 Testing Checklist

All tests in this document are required for Phase 1.

## 1. Sheet Structure Tests

Test that required sheets exist:

- `laborcost_cgd`
- `laborcost_obec`
- `materialcost_obec`
- `materialcost_tpso`
- `STAGING_NORMALIZED`
- `MASTER_PRICE_DATABASE`
- `ALIAS_DICTIONARY`
- `REFRESH_LOG`
- `SEARCH_LOG`
- `CHECKLIST_2_SCHEMA`
- `ALIAS_SUGGESTIONS` if implemented

Test:

- Data sheets have one header row.
- Master and staging headers are row 1.
- No merged cells in script-read sheets.
- `CHECKLIST_2_SCHEMA` is not used as data source.

## 2. Schema Tests

Validate `MASTER_PRICE_DATABASE` columns match `03_DATA_SCHEMA.md`.

Validate `STAGING_NORMALIZED` columns match `03_DATA_SCHEMA.md`.

Validate `REFRESH_LOG`, `SEARCH_LOG`, and `ALIAS_DICTIONARY` schemas.

## 3. Source Mapping Tests

Test price mapping for all sources:

- `laborcost_cgd`
- `laborcost_obec`
- `materialcost_obec`
- `materialcost_tpso`

Ensure `material_cost`, `labor_cost`, `total_cost`, `price`, and `price_basis` are correct.

## 4. TPSO API Tests

Test:

- Successful API fetch.
- Write to `materialcost_tpso` first.
- No direct write to master.
- Header detection.
- API failure does not change master.
- API 0 rows does not change master.
- Missing required header does not change master.
- `REFRESH_LOG` written on failure.

## 5. Normalize / Search Field Generation Tests

Test generation of:

- `item_name_clean`
- `search_keywords`
- `alias_terms`
- `normalized_text`

Check:

- `item_name_clean` does not change meaning.
- Specs are preserved, such as `2x2.5`, `20mm`, `1/2"`, `VAF`, `THW`.
- Alias enrichment uses only active `ALIAS_DICTIONARY` rows.
- No auto-add to `ALIAS_DICTIONARY`.

## 6. Validation Tests

Critical validation tests:

- Row count = 0.
- Required header missing.
- Header row cannot be detected.
- `item_name_original` blank.
- `unit` blank.
- Both `material_cost` and `labor_cost` blank.
- Negative material cost.
- Negative labor cost.
- `source_name` blank.
- `source_type` blank.
- `price` blank.
- `total_cost` blank.
- API response empty.
- API fail.

Warning validation tests:

- Data pattern looks unusual.
- Row count drops more than 30%.

Expected result:

- Critical fail blocks master update.
- Warning over threshold blocks entire source update.

## 7. Master Replace Tests

Test:

- Old master data is not deleted before validation passes.
- Only updated source rows are replaced.
- Other sources are unaffected.
- Validation failure keeps existing data.
- Warning threshold breach blocks source.
- `data_status` updated.
- `last_refresh_at` updated.

## 8. REFRESH_LOG Tests

Test logging for:

- `success`
- `failed`
- `blocked_by_validation`
- `completed_with_warning`

Ensure row counts, validation counts, action taken, error message, and triggered by are recorded.

## 9. Search Tests

Test:

- Search reads from `MASTER_PRICE_DATABASE` only.
- Exact match.
- Partial match.
- Token match.
- Alias match.
- Simple fuzzy match.
- `match_score` calculation.
- Sort high to low.
- Limit top 10.
- No display of `source_name`, `source_type`, `match_reason`.
- `SEARCH_LOG` writing.

Example test cases:

- Search `ปูนซีเมนต์` returns cement items.
- Search `ปลั๊กไฟ` returns outlet/เต้ารับ aliases.
- Search `สาย VAF 2x2.5` preserves spec.
- Typo search returns fuzzy result if possible.
- No result search returns suggestions/nearby items.

## 10. WebApp UI Tests

Test:

- Main search page loads.
- Search input supports Thai, English, and specs.
- Loading state.
- Error state.
- Result card fields.
- Result card hides source and match reason.
- User selects result manually.
- No auto-select.
- Detail section is read-only.
- User can choose another result.

## 11. Price Comparison Tests

Test:

- Required `user_price`.
- Required `user_unit`.
- Required `user_quantity`.
- Required `user_price_type`.
- Optional `user_note`.
- Dropdown default = `unknown`.
- `material` compares to `material_cost`.
- `labor` compares to `labor_cost`.
- `total` compares to `total_cost`.
- `unknown` defaults to `total_cost`.
- If `total_cost` blank, fallback to `price`.
- If reference field blank, return `cannot_compare`.
- Calculate `variance_amount`.
- Calculate `variance_percent`.
- Apply ±10% threshold.

## 12. Unit Conversion Tests

Rule-based conversions:

- kg ↔ ton
- g ↔ kg
- m ↔ cm
- m2 ↔ cm2
- m3 ↔ liter
- piece ↔ dozen
- hour ↔ day with assumption
- day ↔ month with assumption

Complex conversions:

- roll → meter
- bag → kilogram
- job → point
- set → item components
- trip → distance/volume

Expected:

- Rule-based runs first.
- Gemini used only when needed.
- If cannot convert, result is `cannot_compare`.

## 13. Gemini Tests

Test:

- Gemini used only for complex unit conversion.
- Structured output.
- `status` exists.
- `assumption_used` exists.
- `cannot_compare_reason` exists when applicable.
- Gemini does not guess prices.
- Gemini does not modify master/source/staging.
- Gemini is not core search engine.
- Gemini fail fallback works.
- Prompt sends only necessary data.
- Prompt does not send master database.
- Prompt does not send credentials.

## 14. Safety / Negative Tests

Test that the system does not:

- Let WebApp edit raw source sheets.
- Let WebApp edit staging.
- Let WebApp edit master.
- Let comparison flow edit master.
- Let Gemini edit master.
- Auto-approve prices.
- Auto-reject prices.
- Auto-learn aliases into master.
- Create/use `COMPARISON_LOG` in Phase 1.
- Add login/dashboard/admin/export in Phase 1.
