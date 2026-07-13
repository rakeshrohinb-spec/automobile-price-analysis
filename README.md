# Automobile Price Analysis

I wanted to figure out what actually drives a car's price using the classic UCI Automobile dataset (1985 Ward's Automotive Yearbook — 205 vehicles, 26 spec/technical variables). Cleaned and explored the data in Python, then built an interactive Power BI dashboard on top of it.

**Tools used:** Python (pandas, seaborn, matplotlib), Power BI (Power Query, DAX)

![Dashboard preview](Automobile_dashboard_screenshot.png)

## Cleaning the data

Six columns had missing values hidden as the string `"?"` instead of actual nulls: `normalized-losses`, `num-of-doors`, `bore`, `stroke`, `horsepower`, `peak-rpm`, and `price`.

`normalized-losses` was missing in 41 of the 205 rows — about 20%. That's too much to drop or impute without skewing things, and it's an insurance risk metric that's not really central to a price analysis anyway, so I dropped the whole column instead of trying to patch it.

The other five columns only had a handful of missing values each (2–4 rows), so I just dropped those rows. Went from 205 cars down to 193, which felt like a reasonable trade-off.

Also had to explicitly convert `bore`, `stroke`, `horsepower`, `peak-rpm`, and `price` to numeric with `pd.to_numeric()` — once pandas sees a single non-numeric value like `"?"` in a column, it treats the whole column as text, even after the questionable values are gone.

## Checking outliers

Ran the IQR method on `price`, `horsepower`, and `engine-size`, then checked who the outliers actually were before deciding what to do with them:

- **Price:** 14 outliers, every one of them a BMW, Jaguar, Mercedes-Benz, or Porsche.
- **Horsepower:** a handful of outliers, mostly the same luxury brands — with one odd one out, a 200hp Nissan priced at just $19,699.
- **Engine size:** same story, concentrated in the same four premium brands.

Kept all of them. These weren't data entry mistakes — they're just what genuinely premium, high-performance cars look like in this dataset.

## What I found

**Price basically comes down to one thing: size and power.** Engine size (r = 0.89), curb weight (r = 0.84), and horsepower (r = 0.81) are all strongly correlated with price — but they're also highly correlated with *each other* (0.82–0.88). So rather than three independent price drivers, it's really one underlying "how big and powerful is this car" factor showing up three different ways.

**There's a clear luxury tier.** Jaguar averages $34,600, Mercedes-Benz $33,647, Porsche $31,401, and BMW $26,119 — roughly double the mid-tier brands, and up to 6x the cheapest brand average (Chevrolet, at $6,007).

**Body style matters on its own, separate from brand.** Hardtops ($22,209) and convertibles ($21,891) average about 2.3x the price of hatchbacks ($9,764), regardless of who makes them.

**The size-price relationship holds across the board**, not just for luxury brands — you can see it in the scatter plot across every body style. Sedans in particular span almost the whole price range, from some of the cheapest cars in the dataset to some of the priciest.

## Inside the dashboard

- KPI cards for total cars (193), average price ($13,290), and average engine size (128.12)
- Average price by make — shows the luxury tier pretty clearly
- Average price by body style — hardtop/convertible premium
- Engine size vs. price scatter, colored by body style — recreates the Python plot interactively. Had to add an index column in Power Query for this one since the default aggregation was collapsing all 193 individual cars down into just 5 dots (one per body style), which obviously wasn't the point.

## Files here

- `automobile-price-analysis.ipynb` — the full Python cleaning + EDA notebook
- `automobile_cleaned.csv` — cleaned dataset
- `Automobile Price Analysis.pbix` — the Power BI dashboard
- `Automobile_dashboard_screenshot.png` — preview image

## Why this project

More than anything, this was practice in the unglamorous parts of data work — deciding what to do with a column that's 20% missing, checking whether outliers are errors or just real extreme values before touching them, noticing when "three predictors" are actually one thing wearing three different masks, and fixing a dashboard bug that was quietly showing wrong numbers until I caught it.
