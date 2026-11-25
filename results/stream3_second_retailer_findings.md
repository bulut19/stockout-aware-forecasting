# Stream 3: Second-Retailer Validation — Rohlik

## Methodology

We used 2,000 product series from the Rohlik Sales Forecasting Challenge V2 across seven warehouses.

Because Rohlik sales are already adjusted for availability, we created semi-synthetic censored sales using:

Synthetic raw sales = Adjusted sales × Availability

We trained Seasonal Naive and LightGBM models on both data versions using matched chronological splits and seven-day forecasting blocks.

Both models were evaluated against Rohlik-adjusted sales as a demand proxy.

## Test Results

| Model | Synthetic Raw WAPE | Corrected WAPE | Improvement |
|---|---:|---:|---:|
| Seasonal Naive | 28.18% | 27.19% | +0.99 pp |
| LightGBM | 28.75% | 28.08% | +0.67 pp |

## Major Findings

1. Corrected sales improved test WAPE for both forecasting models.
2. Aggregate underforecasting decreased for both corrected models.
3. Seasonal Naive outperformed LightGBM on the held-out test week.
4. High-stockout products showed the largest WAPE improvements for both models.
5. Seasonal Naive improved in all seven warehouses; LightGBM improved in six of seven.

## Important Exceptions

- LightGBM worsened by approximately 0.10 percentage points in the medium-stockout group.
- LightGBM worsened by approximately 0.72 percentage points in Prague_2.
- Corrected forecasts did not always produce lower absolute bias within every stockout group.

## Limitations

- The experiment uses semi-synthetic censored sales, not independently observed historical transactions.
- Rohlik's adjusted sales are a demand proxy, not verified true demand.
- We did not independently validate Stream 1's demand-recovery model on Rohlik.
- Product eligibility requires complete validation/test coverage.
- We evaluated one held-out seven-day test period.
- Findings do not establish financial benefits or inventory savings.

## Figures

- stream3_test_wape_comparison.png
- stream3_test_bias_comparison.png
- stream3_stockout_severity.png
- stream3_actual_vs_forecast.png
- stream3_warehouse_improvement.png

Figures are stored in Stockout Project Data/figures/.

## Data Source

Rohlik Sales Forecasting Challenge V2:
https://www.kaggle.com/competitions/rohlik-sales-forecasting-challenge-v2

## Conclusion

Our semi-synthetic second-retailer experiment provides supporting evidence that stockout-related sales censoring can reduce demand forecasting accuracy.

Training on retailer-adjusted sales improved overall WAPE and reduced aggregate underforecasting for both Seasonal Naive and LightGBM.

However, results varied across product groups and warehouses, and the experiment does not directly establish the performance of our original missed-demand recovery method on a second retailer.
