## Data

**FreshRetailNet-50K** (Dingdong, released on Hugging Face under CC BY 4.0) covers **50,000 store-product series** from **898 stores in 18 cities** and **865 perishable products**. Each series has 97 days (March 28 to July 2, 2024) of:

- **hourly sales** (24 values a day, scaled quantities rather than units or dollars),
- **hourly stock status** (in stock or out of stock),
- daily discount, holiday and activity flags, precipitation, temperature and product category.

We downloaded the parquet files once into a shared Drive folder and wrote one loader (`src/load_data.py`) so every stream reads the same tables. Loading all hourly arrays at once (116 million rows) crashed a 7 GB machine, so the loader reads only the columns and series a stream needs.

| Split | Dates | Days |
|---|---|---|
| Train | Mar 28 to Jun 11 | 76 |
| Validation | Jun 12 to Jun 25 | 14 |
| Test | Jun 26 to Jul 2 | 7 |

Heavy models run on a **2,000-product sample** stratified by category and stockout rate. It matches the full data on stockout rate (44.6% vs. 44.4%) and covers 28 of 32 categories. One model (the censored network) was also trained on all 50,000 products.

**Second retailer.** For Stream 3 we used the **Rohlik Sales Forecasting Challenge V2** (Kaggle): daily sales for an online grocer's warehouses in Czechia, Germany and Hungary, with a daily **availability** share and prices. We took 2,000 products across seven warehouses.

## Data checks

- Every series has all 97 days, with no duplicates or missing values, and hourly sales add up exactly to daily sales.
- Selling hours are **6:00 to 22:00**. Overnight "out of stock" flags are evening stockouts that last until the morning restock, so all hourly work uses selling hours only.
- A discount of 0 appears on 17,339 rows (0.36%) with high sales, which is not a real price. We treat it as missing and add a `discount_missing` flag.

## What the stockouts look like

![Most stockouts are partial-day](figures/eda_partial_vs_zero.png)

**Daily data hides most stockouts.** 44% of product-days have at least one empty-shelf hour, but only 4% are empty all day and only 4.4% show zero sales. A rule that flags stockouts from zero-sales days sees a small fraction of the problem.

![Shelves keep emptying through the afternoon](figures/eda_by_hour.png)

**Afternoon and evening demand is lost most often.** The share of products out of stock climbs from about 6% in the morning to 41% by 9 pm. The median stockout day has 6 empty selling hours, and 15% of stockout days start with the shelf already empty at opening.

![Shelves empty when demand runs high](figures/eda_before_stockout.png)

**Shelves empty on busy days.** In the two hours before a stockout, sales run **1.79 times** normal for the same product and hour. This one fact drives several later results: a plain historical average under-estimates missed demand, and stockout-free days are quieter than average.

**Demand drivers.** On in-stock days, deeper discounts raise sales (+34% vs. the product's average at more than 30% off), and activity days and holidays add about 13 to 14%. Warmer, rainier days show somewhat higher sales, but temperature also rises with the season, so we do not read much into it.

## Features

The trained forecasters use only information available 7 days before the target day: sales (or corrected demand) from exactly 7, 14 and 21 days earlier, their same-weekday average, 7- and 14-day rolling averages starting 7 days back, the product's training-period mean, day of week, discount (with the missing flag), holiday and activity flags, weather (rain, temperature, humidity, wind) and category. Stockout hours are deliberately left out: a store cannot know in advance whether next Tuesday will sell out. Weather and promotions are treated as known in advance, as in the benchmark paper.
