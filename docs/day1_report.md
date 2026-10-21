# Day 1 Report — FreshRetailNet-50K

## 1. Environment

- Operating system: Windows (PowerShell / VS Code).
- Python version: 3.8.20 in the project environment `py3.8_frn`.
- Conda environment: `D:\FreshRetailNet-Project\frn-50k-baseline\py3.8_frn`.
- Dependency verification: NumPy 1.23.5, Pandas 1.5.3, PyTorch 2.0.1+cpu, Hugging Face Datasets 3.1.0.
- CUDA available: False; initial experiments run on CPU.
- Note: core imports succeeded. The output provided confirms these key packages, but does not independently confirm every requirement installed without errors.

## 2. Dataset

- Dataset: `Dingdong-Inc/FreshRetailNet-50K` loaded through Hugging Face `load_dataset`.
- Splits: `train` with 4,500,000 rows and `eval` with 350,000 rows.
- Columns: `city_id`, `store_id`, `management_group_id`, `first_category_id`, `second_category_id`, `third_category_id`, `product_id`, `dt`, `sale_amount`, `hours_sale`, `stock_hour6_22_cnt`, `hours_stock_status`, `discount`, `holiday_flag`, `activity_flag`, `precpt`, `avg_temperature`, `avg_humidity`, `avg_wind_level`.
- Format: Hugging Face dataset loaded from Parquet files.
- Date field: `dt` (string in the dataset schema; parses as dates in the inspected sample).
- Store identifier: `store_id`.
- Product identifier: `product_id`.
- Daily recorded sales field: `sale_amount`.
- Hourly recorded sales: `hours_sale` (24 values per row in the inspected sample).
- Hourly stock-status annotations: `hours_stock_status` (24 values per row in the inspected sample).
- Daily stock-related count: `stock_hour6_22_cnt`.
- Covariates include discount, holiday/activity flags, precipitation, average temperature, humidity, and wind level.
- First record inspected: `dt` = 2024-03-28; `sale_amount` = 0.1. This is an example only.

## 3. Data Quality

Checks below use the first 20,000 training rows unless otherwise stated. These are sample checks, not full-dataset guarantees.

- Missing values: 0 in each of the 19 columns in the sample.
- Duplicate `(store_id, product_id, dt)` keys: 0 in the sample.
- Negative `sale_amount` rows: 0.
- Zero `sale_amount` rows: 600.
- `sale_amount` summary: mean 1.104425, standard deviation 1.353156, minimum 0, median 0.7, 75th percentile 1.2, maximum 21.3.
- Hourly list lengths: all 20,000 sampled rows had 24 values in both hourly list columns; no missing lists.
- Distinct hourly status values observed: 0 and 1.
- Dates parsed successfully for all sample rows.
- Sample date range: 2024-03-28 to 2024-06-25.
- Distinct stores in sample: 3; distinct products: 152; distinct store-product pairs: 223.
- Stock count consistency: sum of `hours_stock_status` at indices 6 through 21 matched `stock_hour6_22_cnt` for all 20,000 sampled rows (0 mismatches).
- No rows were deleted, and no sales values or stock labels were modified during these checks.

### Stock-status interpretation

Repository source inspection found that `latent_demand_recovery/exp/data/generate_data.py` replaces hourly sales with `NaN` wherever `hours_stock_status == 1`. It also selects positions `6:22` (hours 6 through 21) when deriving the stock-related count. This documents how the baseline code treats these labels; it is not independent verification against true demand.

## 4. Baseline Experiment

- Method: 7-day seasonal-naive forecast using daily recorded `sale_amount`.
- Selected series: `store_id = 0`, `product_id = 4`.
- Series length: 90 daily observations.
- Date range: 2024-03-28 to 2024-06-25.
- Time-based split: first 72 observations for training/history and final 18 for testing.
- Test period: 2024-06-08 to 2024-06-25.
- Date continuity: 89 one-day intervals, 0 intervals longer than one day, 0 duplicate dates; 90 unique dates matched 90 expected calendar days.
- MAE: 1.2833.
- Forecast bias (`prediction - actual`): +0.7833.
- Interpretation: on this one series and test period, predictions were higher than recorded sales on average. This does not establish performance across all stores or products.
- Execution: interactively in `notebooks/day1_data_exploration.ipynb`. The code selected a store-product pair from the 20,000-row sample and predicted each test observation using the value seven observations earlier.
- Metric units: the units of `sale_amount` have not yet been independently confirmed.

## 5. Problems Encountered

- Python 3.13 was present in the base environment, while the repository recommends Python 3.8. A separate project environment using Python 3.8.20 was activated successfully.
- Hugging Face emitted a Windows symlink-cache warning and noted `hf_xet` was not installed. Dataset download and loading succeeded using the regular HTTP fallback.
- CUDA was unavailable; PyTorch reports a CPU build. This is acceptable for the small initial baseline.
- No dependency-import or dataset-loading failure was reported in the outputs shared so far.
- Stock-status meanings were not inferred from numeric values alone; baseline preprocessing code was inspected.

## 6. Next Steps

- Review the technical report and summarize the demand-censoring problem, method, and evaluation limitations.
- Continue validating the dataset schema and stockout annotations before using them in experiments.
- Compare forecasts from recorded sales with forecasts from recovered-demand estimates using identical time-based test periods.
- Treat recovered demand as an estimate, not ground truth.
- Design an inventory simulator that reports shortage, leftover/waste, and total cost separately, with explicit cost assumptions.
- Keep additions separate from the baseline repository and prioritize a tested simulator, credible comparison, and useful open-source contribution.

## Limitations and reproducibility notes

- Initial data-quality statistics refer to the first 20,000 training rows only.
- The first forecasting result covers one store-product series and 18 test dates.
- This is a simple baseline, not a claim of state-of-the-art performance.
- Stockout demand is hidden/censored; recorded sales are not automatically true demand.
- Future comparisons must avoid time leakage and use the same test dates, cost settings, and inventory assumptions.
