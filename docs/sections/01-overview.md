![When the shelf empties, sales stop but demand does not](figures/hero_one_day.png)

*One real product-day from our data. The shelf empties at 2 pm; recorded sales (black) stop, but our estimate of demand (orange) keeps going.*

## Abstract

When a grocery shelf runs empty, the sales record stops counting demand. Models trained on recorded sales learn those empty-shelf dips as if customers simply wanted less, forecast too low, and the store orders too little: a loop that keeps shelves empty. Using **FreshRetailNet-50K**, a public dataset of 50,000 product-store series with hourly sales and hourly stock status, we estimate the demand hidden by stockouts, retrain forecasting models on corrected demand, and measure what the correction is worth in dollars.

Stockouts hide about **21% of demand** in our sample, and **81%** of it falls on days that still show sales, so a daily view would never flag it. Correcting for it improves every model we tested, from seasonal naive and LightGBM to a zero-shot time series foundation model (**Chronos-2**), in all five test weeks. Our best setup, corrected demand fed to Chronos-2 with no training, beats LightGBM on raw sales by **14 to 20%** and nearly removes under-forecasting. We also show that the usual way of scoring these models, on stockout-free days only, is biased against correction, and we confirm it with a semi-synthetic test where true demand is known. In an ordering simulation, corrected forecasts cut total cost (lost sales plus spoilage) by about **14%**, and the forecasting result holds on a second retailer.

## The problem: sales are not demand

Fresh groceries spoil within days, so stores order daily and try to keep just enough on the shelf. When an item sells out at 4 pm, every customer who wanted it after 4 pm leaves without it, and none of them show up in the data. Recorded sales are a **censored** version of demand: equal to demand while the shelf is stocked, below it once the shelf is empty.

This matters because forecasting systems learn from recorded sales. A model trained on censored sales learns that demand is lower than it is, especially for products that sell out often. It forecasts too low, the store orders too little, the shelf empties again, and the next round of data is censored even more. In our data, **44% of product-days have at least one empty-shelf hour**, so this is the normal case, not an edge case.

## Related work

FreshRetailNet-50K was released by Dingdong with a benchmark paper that frames the task as latent demand recovery followed by forecasting. Its best forecaster, the Temporal Fusion Transformer (TFT), reached 31.75% WAPE on raw sales and 29.02% after demand recovery, scored on the stockout-free days of one test week. Censored demand is a long-standing problem in operations research: the classic newsvendor ordering model assumes demand is observed, which stockouts break. More recently, pretrained **time series foundation models** such as Amazon's Chronos-2 and Google's TimesFM-3 forecast new series with no training at all. Our professor suggested testing whether they help here.

## Our approach

![Project pipeline](figures/pipeline.svg)

We split the work into five streams that share one data loader, one set of metric functions and frozen date splits:

1. **Missed demand:** fill each empty-shelf hour with an estimate of what would have sold, and measure how much a daily view misses.
2. **Trained forecasters:** seasonal naive, LightGBM and TFT, once on raw sales and once on corrected demand.
3. **Second retailer:** repeat the comparison on Rohlik, a European online grocer.
4. **Foundation models:** Chronos-2 and TimesFM-3, fine-tuning, and a network that learns from censored sales directly, over five rolling test weeks with fair scoring.
5. **Ordering simulation:** turn forecasts into orders and count lost sales and spoilage in dollars.

## Contributions

- A full pipeline from hourly stock data to dollars, evaluated over **five rolling test weeks** instead of one.
- Evidence that **zero-shot foundation models beat trained models** on this data, and that demand correction still adds to their accuracy.
- A finding about **evaluation itself**: scoring only stockout-free days rewards forecasts that run too low, because those days are quieter than average. We propose scores that use every day, give each a lower and an upper bound, and validate them with a **semi-synthetic test** where the true demand is known.
- A head-to-head of **explicit correction vs. end-to-end learning** from censored sales (correction wins).
- A **second-retailer check** and an **ordering simulation** that puts the gain in dollars.
