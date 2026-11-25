## Recovering missed demand

The core idea is simple: for every hour the shelf was empty, estimate what would have sold if it had been stocked, and add it to recorded sales. The hard part is that we never observe the answer for those hours. So we built a test we can grade.

### A masking test we can grade

We took **fully stocked days**, where every hour's sales are real demand, and **hid 2 to 6 hour blocks** of them, starting at the hours when real stockouts usually start. Each method then filled the hidden hours, and we compared its estimate to the true sales of the block (WAPE on the block total, plus bias). Training and test blocks come from different days.

```python
# hide a block that starts when real stockouts tend to start
first_out = hourly[hourly.out_of_stock == 1].groupby(['series_id', 'dt']).hour.min()
start_hours = first_out[first_out >= 8].values
test_days['start'] = rng.choice(start_hours, len(test_days))
test_days['end'] = np.minimum(test_days.start + rng.integers(2, 7, len(test_days)), 22)
```

| Method | Block WAPE | Bias |
|---|---|---|
| Recorded sales (assume zero) | 100.0% | -100.0% |
| Historical average (product, hour, weekday) | 67.8% | +13.5% |
| Day-adjusted average | 65.4% | +4.8% |
| LightGBM (Tweedie loss) | 62.2% | -0.2% |
| Bidirectional LSTM | 62.5% | -0.2% |
| **Average of LightGBM + LSTM** | **62.0%** | **-0.2%** |

- **LightGBM had to train on practice blocks** (hours hidden on other days). Trained on ordinary in-stock hours, it barely beat the averages, because it never saw the task it is graded on.
- **The bidirectional LSTM** reads the day's 16 selling hours forwards and backwards, so it can use both the morning before the gap and any visible hours after it. It is trained with fresh random gaps every epoch and a loss counted only on hidden hours, a form of self-supervised masking.
- **The historical average over-estimates on fully stocked days**, which are quieter than average. This is the mirror image of the 1.79x finding before stockouts.
- We kept the **average of LightGBM and LSTM**: lowest error and almost no bias.

62% may sound high, but hourly grocery sales are very noisy: most hours sell nothing or almost nothing, so even a perfect model of the average hour has a large error on individual blocks. What matters for forecasting is that the daily totals are unbiased.

### What it tells us

Filling the real empty-shelf hours of our 2,000 products gives three headline numbers:

- **Stockouts hide about 21% of demand.** True demand is 20.7% higher than recorded sales overall.
- **On stockout days, true demand is about 45% above recorded sales.**
- **About 81% of missed demand happens on days that still show some sales**, so a daily zero-sales rule would never flag it. Stream 1 checked this headline with a different method (a day-adjusted average) and it changed by only 0.1 points.

A second, much simpler method agrees: scaling each stockout day's sales up by the share of the day that was in stock gives demand **42%** above recorded sales on the test week's stockout days, against **37%** for our filler.

### Which loss should a filler use?

Stream 1 compared LSTM, transformer and dilated 1-D CNN fillers under three losses (absolute error, squared error and Poisson), with last week's same-hour sales as an extra input. The lesson was about the loss, not the architecture:

- With **absolute error (MAE)**, the networks learned to predict **zero** for most hidden hours (80 to 100% of predictions near zero). Most hours sell nothing, so zero minimizes absolute error, and the filler under-estimates by 60% or more. It even "wins" on hour-level WAPE, which is why hour-level WAPE is the wrong way to choose a filler.
- With **squared error or Poisson loss**, the collapse disappears, because those losses aim at the average, and averages are what add up to daily totals.
- Selecting on validation by bias within 10% and lowest block-total error picks a **dilated 1-D CNN with Poisson loss** (block-total WAPE 53.7%, bias -10%). Its masking protocol differs from ours, so the numbers are not directly comparable, and our original filler stays the shared file for forecasting.

### How good is "good enough"?

In the semi-synthetic test described in the Evaluation tab, where we create stockouts ourselves and know the real demand, our filler recovers only about **60% of the hidden demand**. So our headline numbers are probably **conservative**. Yet forecasts built on our corrected demand score the same as forecasts built on the perfect truth (0.308 vs. 0.308 WAPE): for forecasting, the filler is good enough.
