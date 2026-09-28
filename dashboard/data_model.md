# Power BI Data Model

> **Build status:** all three domains and all five pages are built in [`DineSmart.pbip`](DineSmart.pbip) (see [`README.md`](README.md)). Where the build differs from this spec, see [As built](#as-built) at the end.

Reference for building the dashboard on top of the CSVs in `Data/processed/`. Covers what to import, how to relate it, and the DAX measures for the README's Key Metrics. Written for Power BI Desktop's Power Query + model view + DAX - there's no code to run here, just steps to follow in the app.

## The one rule that matters most: three unrelated domains

The raw datasets are three separate platforms that don't share keys. This carried through preprocessing and is the single most important fact about this model:

| Domain | Grain | Key columns | Tables |
|---|---|---|---|
| **1. Zomato_Database** | one row per transaction | `user_id`, `r_id` | `orders_clean`, `users_clean`, `restaurants_clean`, `food_clean`, `menu_clean`, `customer_segments`, `customer_retention_predictions` |
| **2. Food Delivery Order History** | one row per order | `order_id`, hashed `customer_id`, `restaurant_id` | `order_history_clean`, `association_rules` |
| **3. Zomato Delivery Operations** | one row per delivery leg | `id`, `delivery_person_id` | `delivery_ops_clean` |

**Do not build a relationship between tables in different domains.** `order_history_clean.restaurant_id` and `restaurants_clean.r_id` look like they could join - they can't; they're ID spaces from different platforms and a Power BI relationship won't stop you from joining them incorrectly. Each domain gets its own dashboard page(s) and only relates to the shared `Calendar` table described below.

## Domain 1: Zomato_Database model (star schema)

```
        Restaurants (r_id)                Customers (user_id)
              |                                  |
              | 1                              1 |
              |                                  |
              *                                  *
                        Orders (user_id, r_id)
```

### Power Query prep

1. **`Customers`** - don't import `users_clean.csv`, `customer_features.csv`, and `customer_segments.csv` as three separate tables. Merge them in Power Query instead:
   - Start from `users_clean.csv` (100,000 rows - every registered user).
   - Merge (left outer join) with `customer_segments.csv` on `user_id`, expanding `total_orders, total_spending, avg_order_value, total_items, recency_days, order_frequency, segment`.
   - Merge (left outer join) with `customer_retention_predictions.csv` on `user_id`, expanding `retained, retention_probability`.
   - **22,777 of the 100,000 users have no valid order and will show blank behavioral columns** (`customer_segments.csv` only has the 77,223 users with at least one order that survived cleaning). Decide up front whether "Total Customers" means all registered users (100,000) or ordering customers (77,223) - see the measures below for both.
   - Drop `password` if it survived (it shouldn't - already dropped in preprocessing).
2. **`Restaurants`** - import `restaurants_clean.csv` as-is.
3. **`Orders`** - import `orders_clean.csv` as-is. Confirm `order_date` imports as a Date type, not Text.
4. **Skip `food_clean.csv` and `menu_clean.csv`.** Verified directly: every restaurant's `menu_clean.cuisine` is identical to its `restaurants_clean.cuisine` (0 mismatches across 12,013 restaurants) - the menu table adds no cuisine information beyond what's already on `Restaurants`, and importing its 1.18M rows would only bloat the file and slow refreshes. If a future page needs dish-level veg/non-veg splits, import `food_clean.csv` alone (371k rows, still light) rather than the full menu.
5. **`RestaurantCuisines`** (bridge table for cuisine-level visuals) - reference the `Restaurants` query, keep only `r_id` and `cuisine`, then **Split Column by Delimiter `,` -> Rows** on `cuisine`, and trim whitespace. This is the same one-cuisine-per-row explode the EDA notebook did in pandas.
   - **Relate `RestaurantCuisines[r_id]` -> `Restaurants[r_id]` (many-to-one).**
   - **Only use this bridge for counting restaurants per cuisine** (`Top Cuisines` below). A restaurant with `"Beverages,Pizzas"` gets a row for each - if you slice **revenue** by cuisine through this bridge, a pizza place that also serves beverages has its *entire* revenue counted under both tags (fan-out double counting). Don't build a revenue-by-cuisine visual without accounting for this; a restaurant count or rating average is safe, a sum of revenue is not.

### Relationships (Domain 1)

| From | To | Cardinality |
|---|---|---|
| `Orders[user_id]` | `Customers[user_id]` | many-to-one |
| `Orders[r_id]` | `Restaurants[r_id]` | many-to-one |
| `RestaurantCuisines[r_id]` | `Restaurants[r_id]` | many-to-one |

## Domain 2: Food Delivery Order History (flat, standalone)

- **`OrderHistory`** - import `order_history_clean.csv` as-is. It's already denormalized (restaurant name/city inline), so no dimension tables needed.
- **`AssociationRules`** - import `association_rules.csv` as-is. No relationships - it's rule-grain (antecedent/consequent pairs), not order-grain, so it only makes sense as its own table/matrix visual, filtered independently from `OrderHistory`.
- In Power Query, `order_placed_at` is a timestamp (`2024-09-10 23:38:00`) - add a calculated `OrderDate = Date.From([order_placed_at])` column before relating it to `Calendar`.

## Domain 3: Delivery Operations (flat, standalone)

- **`DeliveryOps`** - import `delivery_ops_clean.csv` as-is.

## Shared `Calendar` table

One calendar table can safely relate to all three domains' date columns at once - it doesn't connect the domains to each other, it just gives every page the same date slicer. Create it with a DAX calculated table (Modeling -> New Table):

```dax
Calendar =
CALENDAR (
    MIN (
        MINX ( Orders, Orders[order_date] ),
        MINX ( OrderHistory, OrderHistory[OrderDate] )
    ),
    MAX (
        MAXX ( Orders, Orders[order_date] ),
        MAXX ( OrderHistory, OrderHistory[OrderDate] )
    )
)
```

Then add `Year = YEAR([Date])` and `Month = FORMAT([Date], "MMM YYYY")` columns for the axis labels, and mark it as a Date table (Table tools -> Mark as date table -> `Date`).

### Relationships (Calendar)

| From | To | Cardinality | Active? |
|---|---|---|---|
| `Calendar[Date]` | `Orders[order_date]` | one-to-many | Yes |
| `Calendar[Date]` | `OrderHistory[OrderDate]` | one-to-many | Yes |
| `Calendar[Date]` | `DeliveryOps[order_date]` | one-to-many | Yes |

These three are independent (different fact tables), so all three can stay active without conflict - Power BI only forces a relationship inactive when two paths reach the *same* table.

## DAX measures

Put all of these in a dedicated empty `Measures` table (New Table -> `Measures = {}` won't work directly; instead right-click the model, add a blank query with zero columns, or just group measures under any one fact table - the important part is one consistent home, not scattering them across tables).

### Domain 1 - Business Overview / Customer Analytics / Food & Restaurant Analytics

```dax
Total Orders = COUNTROWS ( Orders )
Total Revenue = SUM ( Orders[sales_amount] )
Average Order Value = DIVIDE ( [Total Revenue], [Total Orders] )

Total Registered Customers = COUNTROWS ( Customers )
Customers With Orders = DISTINCTCOUNT ( Orders[user_id] )

Avg Customer Spend = AVERAGE ( Customers[total_spending] )   -- AVERAGE ignores blanks, so this is scoped to ordering customers automatically
High-Value Customers = CALCULATE ( COUNTROWS ( Customers ), Customers[segment] = "High-value" )
Regular Customers = CALCULATE ( COUNTROWS ( Customers ), Customers[segment] = "Regular" )
Occasional Customers = CALCULATE ( COUNTROWS ( Customers ), Customers[segment] = "Occasional" )
At-Risk Customers = CALCULATE ( COUNTROWS ( Customers ), Customers[segment] = "At-risk" )

Historical Retention Rate =
DIVIDE (
    CALCULATE ( COUNTROWS ( Customers ), Customers[retained] = 1 ),
    CALCULATE ( COUNTROWS ( Customers ), NOT ISBLANK ( Customers[retained] ) )
)
Avg Predicted Retention Probability = AVERAGE ( Customers[retention_probability] )

Total Restaurants = COUNTROWS ( Restaurants )
Avg Restaurant Rating = AVERAGE ( Restaurants[rating] )
Avg Cost For Two = AVERAGE ( Restaurants[cost_for_two] )
Restaurants Serving Cuisine = DISTINCTCOUNT ( RestaurantCuisines[r_id] )   -- use on a RestaurantCuisines[cuisine] axis only
```

> **Card note - Retention:** put "Historical Retention Rate" (actual repeat-purchase rate) on the dashboard as the headline retention KPI, not "Avg Predicted Retention Probability." `customer_prediction.ipynb` found the retention model performs at chance level (ROC-AUC ~0.5) - the predicted probability isn't a reliable signal in this dataset. If you show it at all, label it clearly as experimental/low-confidence rather than as a KPI to act on.

### Domain 2 - Market Basket Analysis page

```dax
OH Total Orders = COUNTROWS ( OrderHistory )
OH Total Revenue = SUM ( OrderHistory[total] )
OH Avg Rating = AVERAGE ( OrderHistory[rating] )
OH Avg Items Per Order = AVERAGE ( OrderHistory[item_count] )
OH Cancellation Rate =
DIVIDE (
    CALCULATE ( COUNTROWS ( OrderHistory ), OrderHistory[order_status] IN { "Returned", "Rejected", "Timed out" } ),
    [OH Total Orders]
)

Top Rules By Lift = RANKX ( ALL ( AssociationRules ), AssociationRules[lift] )
```

Use `AssociationRules` directly as a table visual (`antecedents`, `consequents`, `support`, `confidence`, `lift`, sorted descending by `lift`) - it's already exactly the shape the "Frequently purchased combinations / Association rules" section needs, no measures required beyond an optional rank.

### Domain 3 - Delivery Performance page

```dax
Avg Delivery Time (min) = AVERAGE ( DeliveryOps[time_taken_min] )
Total Deliveries = COUNTROWS ( DeliveryOps )
Festival Deliveries = CALCULATE ( COUNTROWS ( DeliveryOps ), DeliveryOps[festival] = "Yes" )
```

## Recommended pages

| Page | Tables | Key visuals |
|---|---|---|
| **Business Overview** | `Orders`, `Restaurants`, `Calendar` | Revenue & order trend (line, by `Calendar[Month]`), top 10 restaurants by revenue (bar), top cuisines by restaurant count (bar, from `RestaurantCuisines`) |
| **Customer Analytics** | `Customers`, `Orders` | Customers per segment (bar/donut), avg spend by segment, avg spend by income bracket, Historical Retention Rate card |
| **Food & Restaurant Analytics** | `Restaurants`, `RestaurantCuisines`, `Orders` | Rating distribution, cost-for-two distribution, orders/revenue by city, avg rating by cuisine (count-safe, not revenue) |
| **Market Basket Analysis** | `OrderHistory`, `AssociationRules` | `AssociationRules` table sorted by lift, order status breakdown, rating distribution |
| **Delivery Performance** | `DeliveryOps` | Avg delivery time by weather/traffic/vehicle/festival (bar) |

This mirrors the four sections named in the README plus a fifth page for Delivery Performance, which the README lists as a Key Metric but doesn't assign its own section to.

## Carry these caveats onto the dashboard

Findings from the notebooks that a dashboard viewer needs to see, not just a data analyst reading the notebooks:

- **Revenue and order volume are both trending down** across the `orders_clean` history (`exploratory_analysis.ipynb`) - a plain revenue card without the trend line would be misleading.
- **Customer segments separate mainly by spend, not recency** (`customer_segmentation.ipynb`) - the At-risk segment is really "low spend, one-time," not necessarily "hasn't ordered in a while."
- **Retention prediction has no real signal** (`customer_prediction.ipynb`, ROC-AUC ~0.5) - see the card note above.
- **"Frequently purchased combinations" are mostly same-dish flavor variants** (`market_basket_analysis.ipynb`), not classic cross-sell pairs like the README's pizza/drink example - frame the recommendation opportunity as "flavor combo bundles," not generic upsells.

## As built

These are the places where `DineSmart.pbip` differs from the spec above, each driven by something found in the data while building it.

- **Measures live in a `Key Measures` table**, organized into display folders by page. The table has one hidden placeholder column, which is the standard way to make a measures-only table.
- **A `DataFolder` parameter** holds the path to `Data/processed/`, because Power BI can't use relative paths.
- **Top restaurants are ranked by `Restaurants[name]`, not by outlet.**
  - 146,979 orders are spread across 146,858 outlets, so almost every outlet has exactly one order. A per-outlet top 10 would just show the 10 largest single orders.
  - Ranking by name adds up all outlets of a chain, so Domino's Pizza comes first.
- **`Restaurants[metro]` is added** to group revenue by city.
  - `city` is usually stored as "Area,City" ("Baner,Pune"), with 821 raw values.
  - `metro` is the text after the last comma, which gives 552 cities.
- **`RestaurantCuisines` carries a copy of `rating`**, so "average rating by cuisine" works with the single-direction relationship. Without the copy, a cuisine filter can't reach `Restaurants[rating]`, and the only fix would be a bidirectional relationship.
- **Rating and cost-for-two bands are added** as `rating_band` and `cost_band`, each with a hidden sort column.
  - About 59% of restaurants are unrated and fall under "Not rated".
  - `cost_for_two` has extreme outliers (up to 300,350), so the card uses `Median Cost For Two` rather than the average.
- **`Customers[segment]` is "No orders"** for the 22,777 users with no valid order, instead of blank. Segment and income have sort columns so charts show them in a logical order rather than alphabetically.
- **`Orders[currency]` is cleaned in Power Query.** 146,702 rows store it as a quoted `"INR\r"`.
- **Two USD orders are left in `Total Revenue` as-is.** They total ₹750 of ₹963.8M.
- **Retention is not split by segment. Don't add that chart.**
  - Segments come from lifetime order counts, which include the orders in the 180-day retention window.
  - So retention is 36.5% for High-value customers and 0% for single-order segments. That's circular by construction, not a signal.
  - The Customer Analytics page shows retention by monthly income instead. It's flat at 18.3-19.2%, which matches the notebook's "no signal" result.
- **`Calendar` spans all three domains**, which cover different years: Orders 2017-2020, DeliveryOps 2022, OrderHistory 2024-25.
  - The Zomato pages' year slicers use a new `Orders[order_year]` column instead of `Calendar[Year]`, so they list only 2017-2020 and not the years with no Zomato orders.

**Domains 2 and 3**

- **`OrderHistory`:**
  - Loads only the columns the page needs. `items_in_order` and `review` are dropped because they're large free text.
  - Adds `OrderDate` and `order_hour` columns, plus a `has_discount` flag.
  - Empty text fields (discount, cancellation reason, complaint tag) become real blanks.
- **`DeliveryOps`:**
  - `city` is renamed to `area_type`, because it holds Metropolitan/Urban/Semi-Urban, not a city. The source's "Metropolitian" typo is fixed.
  - `road_traffic_density` sorts Low to Jam through a hidden `traffic_order` column.
- **The Market Basket page adds measures beyond the spec:**
  - `OH Undelivered Orders` powers the status chart. Without it, Delivered (99%) would dwarf every other status.
  - `OH Rated Orders` powers the rating chart, since only about 12% of orders are rated.
  - `OH Discounted Share` shows how many orders used a discount.
  - It also adds an orders-by-hour chart. That answers the README's "peak ordering periods" question: orders peak from 7 to 10 pm, highest at 8 pm.
- **The Delivery page adds three cards:**
  - `Festival Delay (min)`: about +19.5 minutes on festival days.
  - `Heavily Stacked Share`: trips carrying 2 or more other orders. These take 40-48 minutes, against 23 with no other orders.
  - `Avg Rider Rating`, which excludes the 53 invalid ratings above 5.
