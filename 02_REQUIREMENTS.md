# 02 Requirements

## 1. Source Registry

| Source Sheet | Use in System | Source Type | Update Frequency | Update Method | Price Mapping |
|---|---|---|---|---|---|
| `laborcost_cgd` | Yes | labor | yearly / manual | replace old data | `labor_cost_thb` → `labor_cost` |
| `laborcost_obec` | Yes | labor | yearly / manual | replace old data | `labor_cost_thb` → `labor_cost` |
| `materialcost_obec` | Yes | material + labor | yearly / manual | replace old data | `material_cost_thb` → `material_cost`, `labor_cost_thb` → `labor_cost` |
| `materialcost_tpso` | Yes | material | monthly / API | replace old data | `priceCur` → `material_cost` |

## 2. Master Database Requirements

The central searchable sheet is `MASTER_PRICE_DATABASE`.

It must contain separate price columns:

- `price`
- `material_cost`
- `labor_cost`
- `total_cost`
- `price_basis`

Price mapping:

### `laborcost_cgd`

- `material_cost` = blank
- `labor_cost` = `labor_cost_thb`
- `total_cost` = `labor_cost_thb`
- `price` = `labor_cost_thb`
- `price_basis` = `labor_only`

### `laborcost_obec`

- `material_cost` = blank
- `labor_cost` = `labor_cost_thb`
- `total_cost` = `labor_cost_thb`
- `price` = `labor_cost_thb`
- `price_basis` = `labor_only`

### `materialcost_obec`

- `material_cost` = `material_cost_thb`
- `labor_cost` = `labor_cost_thb`
- `total_cost` = `material_cost_thb + labor_cost_thb`
- `price` = `total_cost`
- `price_basis` = `material_plus_labor`

The WebApp must display `material_cost` and `labor_cost` separately and may also display `total_cost`.

### `materialcost_tpso`

- `material_cost` = `priceCur`
- `labor_cost` = blank
- `total_cost` = `priceCur`
- `price` = `priceCur`
- `price_basis` = `material_only`

## 3. Notes Requirement

- Notes must be preserved from all source sheets.
- Notes must not be dropped during cleaning, normalization, or master update.

## 4. Search Requirements

Search must read from `MASTER_PRICE_DATABASE` only.

Search must use multiple fields:

- `item_name_original`
- `item_name_clean`
- `category_level_1`
- `category_level_2`
- `category_level_3`
- `unit`
- `note`
- `search_keywords`
- `alias_terms`
- `normalized_text`

Search must support:

- Exact match.
- Partial match.
- Token match.
- Alias match.
- Simple fuzzy match.

Search results must be ranked by `match_score`, high to low, and limited to top 10 in Phase 1.

## 5. Search Display Requirements

Search result cards must display:

- `item_name`
- `unit`
- `material_cost`
- `labor_cost`
- `total_cost`
- `price_basis`
- `note`
- `match_score`
- `needs_review` if applicable

Search result cards must not display:

- `source_name`
- `source_type`
- `match_reason`

## 6. User Selection / Comparison Requirements

The WebApp must not auto-select a result.

User must manually select a search result before comparison.

After selection, user must input:

- `user_price`
- `user_unit`
- `user_quantity`
- `user_price_type`
- `user_note`

`user_note` is optional. Other fields are required.

`user_price_type` values:

- `material`
- `labor`
- `total`
- `unknown`

Default is `unknown`.

## 7. Unit Conversion Requirements

The system must check unit matching before comparison.

If units match, compare directly.

If units differ, use rule-based conversion first.

Use Gemini only when the unit is not straightforward and requires interpretation or additional assumptions.

If conversion is not possible, return `cannot_compare` and do not conclude whether the user price is high or low.

## 8. Comparison Requirements

Use threshold ±10%:

- Within ±10% = `close_to_reference`
- More than +10% = `higher_than_reference`
- Less than -10% = `lower_than_reference`
- Cannot convert = `cannot_compare`

Formula:

```text
variance_amount = user_comparable_price - database_reference_price
variance_percent = variance_amount / database_reference_price * 100
```

Phase 1 must not write `COMPARISON_LOG`.

## 9. Logging Requirements

The system must use:

- `REFRESH_LOG` for refresh/process outcomes.
- `SEARCH_LOG` for search behavior.

Phase 1 does not use `COMPARISON_LOG`.
