# Day 3 Report — Forecast Comparison

## 1. Objective

Compare forecasts generated from recorded sales history and a simple recovered-demand history.

## 2. Experimental Setup

- Store ID: 0
- Product ID: 4
- Training observations: 72 days
- Testing observations: 18 days
- Training period: 2024-03-28 to 2024-06-07
- Testing period: 2024-06-08 to 2024-06-25
- Forecast method: 7-day seasonal-naive baseline

The last seven training observations were repeated to generate predictions for the 18 test days. Both approaches used the same test dates.

## 3. Results

| Forecast approach | MAE | Bias |
|---|---:|---:|
| Recorded-sales forecast | 2.1611 | +1.7833 |
| Recovered-demand forecast | 1.8511 | +1.5850 |

MAE is the mean absolute error; lower is better. Bias is the average of prediction minus actual sales. Positive bias indicates overprediction.

The recovered-demand forecast reduced MAE by approximately 14.3% relative to the recorded-sales forecast on this test period.

## 4. Interpretation

The recovered-demand forecast performed better on both reported metrics for this particular series and test period. Both approaches still overpredicted sales on average.

These results are preliminary and do not establish that recovered-demand forecasting will perform better across other store-product series.

## 5. Limitations

- Only one store-product series was evaluated.
- The test period contains 18 days and only two stockout-annotated days.
- Recovered demand was estimated using a simple training-period mean on stockout-annotated days.
- The estimate is not ground-truth demand.
- Evaluation against recorded sales may be misleading on stockout days because actual demand may exceed observed sales.
- The 7-day seasonal-naive method is a simple baseline, not an advanced forecasting model.

## 6. Saved Outputs

- `day3_forecast_results.csv` — metrics table.
- `day3_forecast_comparison.png` — forecast comparison chart.

Both files are saved in the repository root.

## 7. Next Steps

On Day 4, build an inventory-ordering simulator to estimate shortages, leftover stock and total cost under documented assumptions.