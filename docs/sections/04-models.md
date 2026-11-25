## Forecasting setup

Every model forecasts **daily totals** and every target day is forecast **exactly 7 days ahead**: to forecast June 30, a model may use data up to June 23 and nothing later. This matches how a store orders a week out, and it is stricter than the benchmark paper, which forecasts days 1 to 7 ahead from a single starting point (the near days are easier).

Each model is run twice: once on **raw** recorded sales and once on **corrected** demand (recorded sales plus the filled hours). Everything else stays identical, so any difference comes from the correction.

## Models and why we chose them

| Model | Type | Why it is here |
|---|---|---|
| Seasonal naive | Same weekday one week earlier | The floor any model has to beat |
| LightGBM | Gradient-boosted trees, Tweedie loss | Strong standard for retail tabular forecasting |
| TFT | Temporal Fusion Transformer (attention + LSTM) | Best model in the benchmark paper |
| GRU | Global recurrent network, category embedding | Our own deep model; base for the censored variant |
| Chronos-2 | Pretrained transformer (Amazon), zero-shot | Tests whether foundation models help |
| TimesFM-3 | Pretrained transformer (Google), zero-shot | Second foundation model, suggested by the professor |

**LightGBM** uses the features listed in the Data tab, 1,500 trees at most with learning rate 0.05 and 63 leaves, and **early stopping on the validation weeks** (it stops adding trees once validation error stops improving) to avoid overfitting. A second set of LightGBM models with quantile loss gives 10th, 50th and 90th percentile forecasts for the ordering simulation.

**TFT** reads 2 to 4 weeks of history and uses two attention heads, dropout 0.1, quantile loss, gradient clipping and early stopping.

**GRU.** One network for all products: a GRU reads the last 14 to 28 days of history (ending 7 days before the target), a learned category embedding and the day's calendar and promotion features feed a small dense head with dropout. Inputs are scaled per product; we stop on validation error and average three random seeds.

**Chronos-2 and TimesFM-3** need no training. Each product's history goes in, and a forecast with quantiles comes out. Neither model saw FreshRetailNet during pretraining: the Chronos-2 paper lists its 23 real training datasets and none is retail. Google does not publish TimesFM-3's training list, which we note as a caveat; Chronos-2 is our final model.

### Things we tried that did not earn their place

| Idea | Lecture topic | Result |
|---|---|---|
| LoRA fine-tuning of Chronos-2 (1,000 steps, GPU) | Transfer learning, LoRA | 0.316 to 0.314 test WAPE; beat zero-shot in only 88% of product resamples |
| Weekly covariates (discount, holiday, weather) | Model inputs | Better on test (0.309), worse on validation (0.346): not consistent |
| Cross-learning across products in Chronos-2 | Attention across series | No consistent gain |
| Chronos-2 + LightGBM ensemble | Ensembles, stacking | 0.313 to 0.315, within noise |
| Chronos-2 vs. TimesFM-3 | Foundation models | A tie: forecasts correlate at 0.992 |

Zero-shot Chronos-2 on corrected demand stayed our main setup: simplest, and nothing beat it reliably.

## Learning around stockouts instead of correcting them

Instead of filling the gaps first, could a network learn to ignore stockouts on its own? We gave the GRU two tools: it sees which past days had empty-shelf hours, and it uses a **censored loss**. On a stockout day, recorded sales are only a lower bound on demand, so the loss penalizes a forecast only if it is *below* recorded sales:

```python
def censored_loss(pred, y, censored):
    err = y - pred
    one_sided = torch.clamp(err, min=0)      # stockout day: only forecasts below sales count
    return torch.where(censored > 0, one_sided, err.abs()).mean()
```

We trained it on our 2,000 products and again on **all 50,000 products** (scored on the same 2,000). Results are in the Results tab.

## Rolling backtest: five test weeks, not one

A single test week can mislead. We repeated every forecast for **four earlier weeks** (May 15 to Jun 11) plus the official test week. Each week uses only data up to its own cutoff; trained models (LightGBM, GRU) are retrained for every week, with LightGBM's product means recomputed before each cutoff so no future data leaks in. We report the five-week average and how many weeks correction wins, with a one-sided sign test (5 of 5 wins gives p = 0.031). TFT was too expensive to retrain five times, so it appears in the test-week comparison only.
