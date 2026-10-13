# Business AI Suite: Shared Data Contracts

The tools in this suite process business data independently. This document defines minimum interchange expectations so they can later integrate with XYLO without silently changing field meanings.

## General rules

- Preserve the original input file and write transformed data to a separate output.
- Reject missing required columns with an actionable message.
- Do not silently convert malformed numeric values to zero.
- Report input, accepted, and rejected row counts when a tool filters records.
- Record the input period and currency wherever financial totals are reported.
- Keep raw values available for audit and troubleshooting.

## KPI transaction CSV

Required columns:

| Column | Expected value | Validation |
|---|---|---|
| `date` | Date such as `2026-01-31` | Parseable, non-empty date |
| `amount` | Numeric transaction amount | Finite number; invalid values are reported |
| `type` | `revenue` or `expense` | Trim and normalize case before aggregation |

Use one consistent currency and sign convention per file. Classify transactions by `type`, rather than inferring revenue or expense from a sign alone.

## Inventory history CSV

Required columns:

| Column | Expected value | Validation |
|---|---|---|
| `day` | Numeric time index | Finite; sort before forecasting |
| `stock_left` | Inventory count | Finite and non-negative |

Forecasts are estimates, not purchase orders. Combine predicted stock with lead time, minimum order quantity, and a human review step before placing an order.

## Model output metadata

For model-backed tools, include or log:

- Model or algorithm name and version
- Forecast horizon or scoring timestamp
- Input time window
- Units and currency, where applicable
- Validation warnings and assumptions

## Forecast evaluation safeguards

- Keep validation rows later in time than training rows; do not randomly shuffle time-series observations.
- Report the forecast horizon and the baseline used for comparison.
- Include at least one scale-aware error metric, such as MAE or RMSE, and state its units.
- Flag negative predicted inventory as a projected stockout, not physically possible stock.
- Record when history is too short or contains gaps that make a forecast unreliable.

## Compatibility principle

When a schema changes, update the producing tool, its tests, and this contract together. Downstream tools should not have to guess units, periods, class labels, or missing-value behavior.
