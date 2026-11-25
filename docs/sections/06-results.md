## Result 1: correction helps every model, every week

![Correction lowers error for every model](figures/fig_fair_scores.png)

Over five test weeks, scored on every day, training on corrected demand beats training on raw sales **in all 5 weeks for all 4 models**, on both the lower and the upper error bound (one-sided sign test p = 0.031 for each model).

| Model (5-week average) | Observed-hours WAPE | Lower bound | Upper bound | Bias |
|---|---|---|---|---|
| Chronos-2, raw | 0.369 | 0.303 | 0.411 | -16% |
| **Chronos-2, corrected** | **0.339** | **0.262** | **0.383** | **-3%** |
| LightGBM, raw | 0.395 | 0.327 | 0.434 | -19% |
| LightGBM, corrected | 0.356 | 0.278 | 0.397 | -4% |
| GRU, raw | 0.414 | 0.347 | 0.452 | -22% |
| GRU, corrected | 0.371 | 0.297 | 0.412 | -10% |
| Seasonal naive, raw | 0.415 | 0.344 | 0.452 | -12% |
| Seasonal naive, corrected | 0.394 | 0.310 | 0.432 | +1% |

Every model trained on raw sales under-forecasts real demand by **12 to 22%**. Correction cuts that to a few percent. Against LightGBM on raw sales, our final setup (corrected demand into Chronos-2) is **14% better** on observed-hours error (0.395 to 0.339) and **20% better** on the lower bound (0.327 to 0.262).

## Result 2: a zero-shot foundation model beats every trained model

On the official test week, with the original in-stock scoring:

| Model | Raw | Corrected |
|---|---|---|
| Chronos-2 (zero-shot) | 0.332 | **0.316** |
| TimesFM-3 (zero-shot) | 0.331 | 0.317 |
| TFT | 0.377 | 0.344 |
| LightGBM | 0.389 | 0.350 |
| Seasonal naive | 0.385 | 0.384 |

Without seeing a single FreshRetailNet example in training, both foundation models beat every trained model, and on every-day scoring Chronos-2 on corrected demand is the best model **in every one of the five weeks**. Correction still helps it: in the test week it wins in 100% of 2,000 random resamples of the products. Rescored on every day, TFT improves from 0.406 to 0.368 observed-hours WAPE with correction, but corrected Chronos-2 (0.328) stays 11% ahead.

**How does this compare to the benchmark paper?** Our numbers are not directly comparable: we forecast every day exactly 7 days ahead, which is harder (TFT on raw sales scores 0.377 in our setup vs. 31.75% in the paper). Under identical conditions, Chronos-2 on corrected demand outperforms TFT, the paper's best forecaster, by **8%** on in-stock scoring and **11%** on observed-hours scoring.

## Result 3: correction beats learning around stockouts

![Learning around stockouts closes most of the gap](figures/fig_censored.png)

The censored GRU recovers about **three quarters** of correction's benefit (observed-hours 0.414 raw, 0.391 censored, 0.371 corrected). Two details stood out:

- **More censored data makes a plain model worse.** Trained on raw sales of all 50,000 products, the GRU's error rises to 0.444 and its under-forecasting deepens to 26%: it learns the stockout dips more confidently.
- **The censored loss is what makes scale useful.** With the censored loss, 50,000 products beat 2,000 in 4 of 5 weeks (0.385), but it still trails the corrected GRU (0.371), corrected LightGBM (0.356) and corrected Chronos-2 (0.339).

On the official test week alone, the censored GRU had looked better than the corrected one (0.335 vs. 0.344). Across five weeks it wins only 1 of 5. That reversal is the reason we stopped trusting single-week results.

## Result 4: it holds on a second retailer

![Second retailer: same ranking, same direction](figures/fig_rohlik.png)

Rohlik publishes availability-adjusted sales, so Stream 3 built censored sales by multiplying them by each day's availability share, then forecast both versions with the same products, weeks and seven-day blocks. We added Chronos-2 with exactly the same setup.

| Rohlik, WAPE | Test raw | Test corrected | 3-week average raw | 3-week average corrected |
|---|---|---|---|---|
| **Chronos-2 (zero-shot)** | 0.232 | **0.222** | 0.213 | **0.203** |
| LightGBM | 0.288 | 0.281 | 0.279 | 0.274 |
| Seasonal naive | 0.282 | 0.272 | 0.285 | 0.278 |

Correction helps all three models in **3 of 3 weeks**, most on days with incomplete availability (Chronos-2: 0.288 to 0.230 on those days), and Chronos-2 corrected beats the best trained model by about **18%**. Here the correction is Rohlik's own adjustment rather than our filler, so this checks the forecasting result, not our missed-demand method.

![Rohlik: benefit across warehouses](figures/stream3_warehouse_improvement.png)

Seasonal naive improves in all seven warehouses and LightGBM in six of seven.
