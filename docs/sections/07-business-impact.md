## From forecasts to orders

A forecast is only worth what it saves. Stream 5 turned every forecast into daily orders and counted the cost over the same five test weeks for the 2,000 products.

**Cost assumptions** (the dataset has no prices, so these are stated assumptions):

- each unit sells for $1 with a **30% margin** (also tested at 20% and 40%), so a missed sale loses $0.30;
- unsold perishables spoil the next day and lose their **$0.70** cost.

**Ordering rule.** With those costs the classic newsvendor answer is to order the forecast's **30th percentile**: 0.30 / (0.30 + 0.70). Ordering more risks spoilage that costs more than the margin it protects. So the simulator orders each model's 30th-percentile forecast (Chronos-2 gives these directly; LightGBM uses quantile models), and true demand comes from our corrected demand.

![Ordering from corrected forecasts cuts total cost](figures/fig_cost.png)

| 5 weeks, 2,000 products | Lost sales | Spoilage | Total cost | Fill rate |
|---|---|---|---|---|
| Chronos-2, raw | $10,242 | $1,993 | $12,236 | 58.7% |
| **Chronos-2, corrected** | **$7,031** | $3,487 | **$10,518** | **71.7%** |
| LightGBM, raw | $11,091 | $1,758 | $12,849 | 55.3% |
| LightGBM, corrected | $7,682 | $3,394 | $11,076 | 69.0% |
| Seasonal naive, raw | $9,909 | $4,439 | $14,348 | 60.1% |
| Seasonal naive, corrected | $7,602 | $6,212 | $13,814 | 69.4% |

- **Corrected forecasts cut total cost by about 14%** for both Chronos-2 ($1,718 saved) and LightGBM ($1,773), and win in **5 of 5 weeks** (p = 0.031). They spoil a little more but lose far fewer sales, and the store serves about 13 points more of its customers.
- **Chronos-2 on corrected demand is the cheapest setup in every week.**

![Cost of ordering each percentile](figures/stream5_cost_by_percentile.png)

**The order level matters as much as the forecast.** The corrected forecast is cheapest exactly where theory says, at the 30th percentile. Raw forecasts run so low that their cheapest point drifts to the 40th percentile, and even there they cost about 11% more than the corrected forecast at the 30th. Ordering the median instead makes corrected forecasts look *worse* than raw ones, because the median of a corrected forecast over-orders perishables.

### How sure are we?

The dollar result depends on what true demand was on stockout days, which nobody observes. So we ran the simulation under three versions of the truth:

| 30% margin, gain from correction | Chronos-2 | LightGBM |
|---|---|---|
| Truth = recorded sales (assumes stockouts hid no demand) | -4.8% | -1.4% |
| **Truth = our corrected demand** | **+14.0%** | **+13.8%** |
| Truth = sales scaled up for empty hours | +4.2% | +5.8% |

The first row is a deliberate worst case: if stockouts hid no demand at all, there is nothing to correct and correction can only add spoilage. Under any truth that admits hidden demand, correction pays. The gain under our main truth is stable across margins (12 to 15% at 20%, 30% and 40%).

### What a store would do with this

1. **Stop training on raw sales.** Fill empty-shelf hours first; the stock-status data needed is already collected by most grocery systems.
2. **Order the right quantile, not the forecast.** With 30% margins and next-day spoilage, order the 30th percentile of the corrected forecast.
3. **Watch for sell-out patterns.** Products that sell out often gain the most from correction (for LightGBM, high-stockout products improved from 0.393 to 0.319 WAPE on the test week).
4. **Start with a foundation model.** Zero-shot Chronos-2 needs no training pipeline and beat every model we trained.
