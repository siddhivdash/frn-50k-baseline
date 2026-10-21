# Day 2 Report — Comparison Dataset Preparation

## 1. Objective

Prepare a small dataset to compare forecasting using recorded sales against forecasting using a simple recovered-demand estimate.

## 2. Dataset and Series

- Dataset: FreshRetailNet-50K
- Store ID: 0
- Product ID: 4
- Total observations: 90 daily rows
- Date range: 2024-03-28 to 2024-06-25

This is a small experimental subset, not the complete dataset.

## 3. Time-Based Split

| Split | Observations | Date range |
|---|---:|---|
| Training | 72 | 2024-03-28 to 2024-06-07 |
| Testing | 18 | 2024-06-08 to 2024-06-25 |

The data was sorted by date before splitting. The training and testing periods do not overlap.

## 4. Demand Histories

### Recorded sales

The original `sale_amount` values were preserved in a separate `recorded_sales` column.

### Simple recovered-demand estimate

The mean `sale_amount` on training days where `stock_hour6_22_cnt == 0` was approximately 2.8028.

For days where `stock_hour6_22_cnt > 0`, `recovered_demand` was set to this training-period mean. Other days retained their recorded sales.

This is a simple baseline estimate, not ground-truth demand.

## 5. Validation Results

- Total rows: 90
- Columns in saved CSV: 15
- Duplicate dates: 0
- Recorded sales match original sales: True
- Non-stockout estimates remain unchanged: True
- Stockout-day estimates use the training mean: True
- Training observations: 72
- Testing observations: 18

## 6. Limitations

- The experiment uses only one store-product series.
- The test period contains only two stockout-annotated days.
- The mean-based estimate does not account for weekday, discount, holiday or weather differences.
- Stockout annotations indicate potentially censored observations; they do not reveal the exact amount of unobserved demand.
- The recovered-demand values must be treated as estimates, not actual demand.

## 7. Output

File: `day2_comparison_series.csv`

The CSV contains the date, store and product identifiers, split label, original sales, recorded-sales history, recovered-demand estimate, stockout count and selected covariates.

## 8. Next Steps

On Day 3, compare forecasting from recorded-sales history against forecasting from recovered-demand history. Use the same testing dates and evaluate both approaches with MAE and forecast bias.