## Scoring forecasts when demand is hidden

On a stockout day, the true demand is unknown, so how do you grade a forecast? The benchmark paper and our original plan used the common answer: **score only days with no stockouts**, where sales equal demand. Our five-week backtest showed that this answer is biased.

### Stockout-free days are quieter than average

Stockouts happen on busy days (sales run 1.79 times normal just before one). So the days that survive the "no stockout" filter are the quieter ones. A forecast that aims at typical demand looks too high on those days, and a forecast that runs low looks good. In-stock scoring therefore **rewards under-forecasting and penalizes correction**.

![Stockout-free days are always quieter](figures/fig_quiet_days.png)

In every week, stockout-free days are **7 to 9% quieter** than the rest of their week, the test week included. The test week looked different for another reason: its demand jumped **13% above** the four weeks before it, which offset the quiet-day effect for forecasts built on history. That is why, on in-stock scoring, correction looked clearly better on the test week but won only 3 of 5 weeks overall (Chronos-2: 0.352 vs. 0.356 average).

### Two scores that use every day

**Observed-hours WAPE.** Even on a stockout day, the hours before the shelf emptied are real demand. We split each daily forecast across hours using the product's normal hourly pattern (learned from in-stock hours before May 8) and compare it only with the in-stock hours of each day, including the busy hours right before a stockout.

```python
# share of the day's demand that falls in the hours the shelf was stocked
obs = in_stock_hours.groupby(['series_id', 'dt']).agg(obs_sales=('sales', 'sum'),
                                                      obs_share=('share', 'sum'))
f['fc_obs'] = f.forecast * f.obs_share
observed_hours_wape = (f.obs_sales - f.fc_obs).abs().sum() / f.obs_sales.sum()
```

**Error bounds.** On a stockout day we know one thing for sure: demand was at least what sold. That gives a range for the true error:

- **Lower bound:** on stockout days, only forecasts *below* recorded sales count as errors (demand is at least sales). On its own this score could be gamed by forecasting very high, so we never use it alone.
- **Upper bound:** on stockout days, demand is taken as sales scaled up by the share of the day that was in stock. This assumes the empty hours would have sold at the usual rate.

A forecast that wins on **both** bounds wins whatever the true demand was within that range.

### Checking the scores where we know the answer

Reasoning is not enough, so we built a **semi-synthetic test**. We treated our corrected demand as the "true" demand of a simulated world, gave every product-day a stock level (the 60th percentile of its last 28 days, with noise) and stopped sales once stock ran out. That world looks like the real one: about 41% stockout days, true demand 42% above recorded sales on stockout days, and stockout-free days 20% quieter than average. We then ran the **full pipeline** (our filler, then Chronos-2) on the censored sales and graded every score against the known truth.

![With known demand, in-stock scoring ranks the forecasts backwards](figures/fig_semisynthetic.png)

| Chronos-2, 5-week average | True error | In-stock | Observed-hours | Lower bound | Upper bound |
|---|---|---|---|---|---|
| Raw sales | 0.336 | **0.289** | 0.313 | 0.235 | 0.337 |
| Corrected (our filler) | **0.308** | 0.309 | **0.301** | **0.219** | **0.314** |
| Perfect correction | 0.308 | 0.328 | 0.308 | 0.224 | 0.316 |

- The **true error** says correction wins in **5 of 5 weeks**. **In-stock scoring says raw wins in all 5**, and it gives perfect correction its worst score.
- The **bounds** pick correction in 5 of 5 weeks and observed-hours in 4 of 5. Their agreement with the true ranking of all six forecasts (rank correlation): in-stock 0.58, observed-hours 0.79, lower bound 0.90, upper bound 0.93.
- One caveat: for seasonal naive, observed-hours picked correction in only 1 of 5 weeks and the lower bound in 3 of 5, while the truth says 5 of 5. The bounds are the most reliable scores, so we lead with them.
