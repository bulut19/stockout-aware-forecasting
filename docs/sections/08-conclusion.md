## Conclusion

Recorded sales understate grocery demand, and they do it most on the busy days that matter most for ordering. In FreshRetailNet-50K, stockouts hide about 21% of demand, and 81% of that loss sits on days that still show sales, invisible to any daily rule. Filling the empty-shelf hours with a LightGBM plus bidirectional LSTM estimate and forecasting on the corrected history improves every model we tried in every one of five test weeks, removes most of the under-forecasting, and cuts the cost of lost sales plus spoilage by about 14% in an ordering simulation. A zero-shot foundation model, Chronos-2, was the best forecaster throughout, and the result held on a second retailer.

## Lessons learned

- **How you score decides what you conclude.** Scoring only stockout-free days, the standard practice, quietly rewards under-forecasting. On the official test week it happened to agree with us; on earlier weeks it did not. Only scores that use every day, checked against a semi-synthetic world with known demand, gave a consistent answer.
- **One week is not a result.** Two of our early "wins" (a large test-week gain for corrected Chronos-2 on in-stock scoring, and a censored network beating explicit correction) shrank or reversed over five weeks. Resampling products within one week could not reveal this; resampling weeks did.
- **The loss function is a modeling decision.** Absolute-error loss made our gap-filling networks predict zero, and choosing a filler by hour-level WAPE would have picked the worst one for daily totals.
- **More data can make a biased model worse.** Training on 25 times more censored sales made a plain network forecast lower, not better. The loss has to know which observations are censored.
- **Pretrained models are a strong default.** Chronos-2 beat everything we trained, and fine-tuning, covariates and ensembles added little on top.

## Future work

- **Retrain the filler at each backtest cutoff.** Our missed-demand models were trained once on all training days, so in the earliest backtest weeks the corrected history uses a filler that saw slightly later data. The effect touches only filled hours, but removing it would make the backtest fully clean.
- **Make the filler less conservative.** Practice gaps on busy days at the end of the day, like real stockouts, and a Poisson-trained network may recover more of the hidden demand.
- **Carry the filler's uncertainty forward.** Sampling several plausible corrected histories and forecasting each would give honest forecast ranges for ordering.
- **Use real prices and margins**, per category, instead of one assumed margin.
- **Test our filler on another retailer** with hourly stock data, rather than relying on that retailer's own adjustment.
