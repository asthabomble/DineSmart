# DineSmart Power BI dashboard

`DineSmart.pbip` is a **Power BI Project**: the same report as a `.pbix`, but saved as text files so it can be diffed and merged in git.

- `DineSmart.SemanticModel/`: the data model (Power Query, relationships, DAX), in TMDL
- `DineSmart.Report/`: the report pages and visuals, in PBIR JSON

The design follows [`data_model.md`](data_model.md).

## Opening it (Windows, Power BI Desktop)

1. **Enable the preview features.** Go to File > Options and settings > Options > Preview features and tick these, if they're listed (newer versions have some of them on by default):
   - **Power BI Project (.pbip) save option**
   - **Store semantic model using TMDL format**
   - **Store reports using enhanced metadata format (PBIR)**

   Restart Power BI Desktop.
2. **Open `dashboard/DineSmart.pbip`.**
3. **Point the model at your data.** Go to Home > Transform data > Edit parameters and set **DataFolder** to the absolute path of `Data\processed\`, **including the trailing backslash**, e.g. `C:\Users\you\DineSmart\Data\processed\`. Power BI doesn't support relative paths.
4. **Load the data.** Click **Refresh**. The model has no data until the first refresh.
5. **Save** (Ctrl+S). Power BI Desktop writes the project back in place and adds its own IDs (`lineageTag`s) and layout files. Commit those changes. To get a single shareable file, use File > Save as > `.pbix`.

## Pages

| Page | What's on it |
|---|---|
| **Business Overview** | Revenue, orders, AOV, active customers, 12-month revenue change; monthly revenue and order trends; top 10 restaurant brands by revenue; top 10 cuisines by restaurant count |
| **Customer Analytics** | Ordering vs registered customers, avg spend, high-value revenue share, actual retention rate; customers / spend / revenue share by segment; spend and retention by monthly income |
| **Food & Restaurant Analytics** | Restaurant count, cities, avg rating, % rated, median cost for two; rating and cost-for-two distributions; revenue by city (top 10 + full table); avg rating for the 15 largest cuisines |
| **Market Basket Analysis** | Orders, revenue, items per order, % discounted, avg rating, cancellation rate; association rules table sorted by lift; basket-size distribution; orders by hour of day; undelivered orders by status; rating distribution |
| **Delivery Performance** | Deliveries, avg delivery time, festival delay, % heavily stacked trips, avg rider rating; avg delivery time by traffic, stacked orders, festival, weather, vehicle condition, area type |

The first three pages use the Zomato_Database data. The last two each use their own separate dataset: Food Delivery Order History for Market Basket Analysis, and Zomato Delivery Operations for Delivery Performance. The datasets don't share IDs, so a page's filters only affect that page's own data.

Each page ends with a "Read with care" note that carries the notebooks' caveats onto the dashboard.

## Check the numbers after the first refresh

These are computed straight from the CSVs in `Data/processed/`. With no slicer applied, the cards should show:

| Card | Expected |
|---|---|
| Total Revenue | ₹963,846,486 |
| Total Orders | 146,979 |
| Average Order Value | ₹6,558 |
| Customers With Orders / Ordering Customers | 77,223 |
| Revenue Change (Last vs First 12M) | -24.0% |
| Total Registered Customers | 100,000 |
| Avg Customer Spend | ₹12,481 |
| High-Value Revenue Share | 74.1% |
| Historical Retention Rate | 19.0% |
| Total Restaurants | 148,455 |
| Cities Served | 552 |
| Avg Restaurant Rating | 3.89 |
| Share of Restaurants Rated | 41.4% |
| Median Cost For Two | ₹250 |
| OH Total Orders | 21,321 |
| OH Total Revenue | ₹14,554,058 |
| OH Avg Items Per Order | 1.79 |
| OH Discounted Share | 74.2% |
| OH Avg Rating | 4.36 |
| OH Cancellation Rate | 0.86% |
| Total Deliveries | 45,584 |
| Avg Delivery Time (min) | 26.3 |
| Festival Delay (min) | +19.5 |
| Heavily Stacked Share | 5.1% |
| Avg Rider Rating | 4.63 |

If any of these differ, the refresh read the CSVs differently than intended. Check the column types in Transform data first.
