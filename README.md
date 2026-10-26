# DineSmart

## Data Mining and Business Intelligence for Food Delivery Platforms

DineSmart is a Data Mining and Business Intelligence project that analyzes food delivery data to uncover customer behavior, ordering patterns, restaurant performance, and actionable business insights.

The project combines data preprocessing, exploratory data analysis, data mining techniques, machine learning, and interactive Business Intelligence dashboards to support data-driven decision-making for food delivery businesses.

---

## Status

All five analysis notebooks are complete, and the five-page Power BI report is built. Its pages cover Business Overview, Customer Analytics, Food & Restaurant Analytics, Market Basket Analysis, and Delivery Performance. The completed [business insights and recommendations](business_insights.md) answer the seven Expected Outcomes using the project results. See [dashboard/README.md](dashboard/README.md) for setup and refresh instructions.

| Component | Implementation | Status / key finding |
| --- | --- | --- |
| Data preprocessing | [`notebooks/data_preprocessing.ipynb`](notebooks/data_preprocessing.ipynb) | Complete; cleans all three source datasets into `Data/processed/` |
| Exploratory analysis | [`notebooks/exploratory_analysis.ipynb`](notebooks/exploratory_analysis.ipynb) | Complete; historical order and revenue trends decline over the Zomato dataset period |
| Customer segmentation | [`notebooks/customer_segmentation.ipynb`](notebooks/customer_segmentation.ipynb) | Complete; segments separate mainly by spend and order count, with a weaker recency effect |
| Market basket analysis | [`notebooks/market_basket_analysis.ipynb`](notebooks/market_basket_analysis.ipynb) | Complete; 62 association rules, mostly same-dish flavor pairs |
| Retention prediction | [`notebooks/customer_prediction.ipynb`](notebooks/customer_prediction.ipynb) | Complete as an analysis; all models are near chance (ROC-AUC ~0.49–0.51), so scores are not a production targeting tool |
| Power BI dashboard | [`dashboard/DineSmart.pbip`](dashboard/DineSmart.pbip) | Complete; five report pages cover all three data domains; requires local `DataFolder` setup and refresh |
| Business insights and recommendations | [`business_insights.md`](business_insights.md) | Complete; answers all seven Expected Outcomes and records interpretation limits |

Each notebook's summary section records its method and interpretation. The run order is documented below.

---

## How to Run

There are two parts to look at: the **notebooks** (analysis and matplotlib charts) and the **Power BI dashboard** (interactive report pages). The cleaned CSVs in `Data/processed/` are already committed, so you can open the dashboard without running any notebook first.

### 1. Get the project

```bash
git clone https://github.com/asthabomble/DineSmart.git
cd DineSmart
```

### 2. View the charts in the notebooks

The notebooks are saved with their outputs, so you can see every chart without running anything: open any `.ipynb` file under [`notebooks/`](notebooks/) on GitHub or in VS Code.

To re-run them yourself (Python 3.11+ recommended):

```bash
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
python -m pip install -r requirements.txt
cd notebooks
jupyter notebook                   # or: jupyter lab
```

Start Jupyter from inside `notebooks/`, because the notebooks read `Data/processed/` through relative paths. Run them in this order, using **Kernel > Restart & Run All** for each:

1. `data_preprocessing.ipynb`: rebuilds everything in `Data/processed/` from `Data/raw/`
2. `exploratory_analysis.ipynb`: revenue, order, customer, restaurant and delivery trend charts
3. `customer_segmentation.ipynb`: K-Means segments (writes `customer_segments.csv`)
4. `market_basket_analysis.ipynb`: Apriori rules (writes `association_rules.csv`)
5. `customer_prediction.ipynb`: retention models (writes `customer_retention_predictions.csv`)

In VS Code you can instead open a notebook, pick the `.venv` interpreter as the kernel, and click **Run All**. Make sure the notebook's working directory is `notebooks/`.

### 3. Open the Power BI dashboard

The dashboard needs **Power BI Desktop**, which is free but **Windows only** (on macOS or Linux, use a Windows VM).

1. Install Power BI Desktop from the Microsoft Store or the [Microsoft download page](https://www.microsoft.com/power-platform/products/power-bi/desktop).
2. In Power BI Desktop, go to **File > Options and settings > Options > Preview features** and enable these if they are listed, then restart:
   - Power BI Project (.pbip) save option
   - Store semantic model using TMDL format
   - Store reports using enhanced metadata format (PBIR)
3. Open **`dashboard/DineSmart.pbip`**.
4. Go to **Home > Transform data > Edit parameters** and set **DataFolder** to the full path of your `Data\processed\` folder, **ending with a backslash**, e.g. `C:\Users\you\DineSmart\Data\processed\`. If you cloned the repo to `C:\DineSmart`, the default value already works and you can skip this step.
5. Click **Home > Refresh**. The report is empty until the first refresh.
6. Use the page tabs at the bottom to switch between **Business Overview**, **Customer Analytics**, **Food & Restaurant Analytics**, **Market Basket Analysis** and **Delivery Performance**.

To check that the data loaded correctly, compare the KPI cards with the expected values in [`dashboard/README.md`](dashboard/README.md#check-the-numbers-after-the-first-refresh) (for example, Total Revenue should be ₹963,846,486 and Total Orders 146,979). To share the report as a single file, use **File > Save as** and choose `.pbix`.

---

## Problem and Solution

### The problem

A food delivery platform records every order, customer, restaurant, menu item and delivery, but the raw tables can't answer business questions on their own. The table `orders.csv` has 146,979 rows of dates, amounts and IDs, and none of them say on its own *why revenue is falling*, *which customers matter most* or *why deliveries are late*.

The platform needs to know:

1. Is the business growing or shrinking, and where does the money come from?
2. Which customers bring in the revenue, and which ones are slipping away?
3. Which dishes are ordered together, so they can be sold as combos?
4. When and where do orders peak?
5. What makes deliveries slow?
6. Can we predict which customers will order again?

### The solution

DineSmart turns the raw tables into answers in four steps:

1. **Clean** the three raw datasets (missing values, bad formats, broken IDs) into `Data/processed/`.
2. **Explore** the cleaned tables to find trends by time, city and area.
3. **Mine** them with K-Means (customer segments), Apriori (dishes ordered together) and classification models (retention).
4. **Show** the results in a five-page Power BI dashboard that a manager can filter without writing code.

### Demo walkthrough: from table to insight

Each finding below says which table it comes from, which region it covers, what the data shows, and what the business should do about it. All numbers come straight from the CSVs in `Data/processed/`.

#### Finding 1: Revenue is shrinking

| | |
|---|---|
| **Table** | `orders_clean.csv` (`order_date`, `sales_amount`) joined to `restaurants_clean.csv` (`city`) on `r_id` |
| **Region** | All of India: 820 cities and city areas, Oct 2017 to Jun 2020 |
| **What the data shows** | Revenue fell from ₹402.9M in 2018 to ₹331.3M in 2019 (-17.8%). The first half of 2020 brought ₹140.4M, about ₹23M a month against ₹34M a month in 2018. The top earning areas are Tirupati (₹42.5M), Electronic City, Bangalore (₹28.6M), Baner, Pune (₹26.8M), Raipur (₹22.6M) and Malviya Nagar, Delhi (₹16.5M). Together these five bring in only 14% of revenue, so sales are spread thin across many cities. |
| **Solution** | Look at why volume is dropping (fewer orders, not smaller orders) and protect the top cities first with targeted offers, since they give the most return per rupee spent. |
| **See it in** | `exploratory_analysis.ipynb`, dashboard page **Business Overview** |

#### Finding 2: A quarter of customers bring in three quarters of the money

| | |
|---|---|
| **Table** | `customer_segments.csv`, built from `orders_clean.csv` + `users_clean.csv` by K-Means on orders, spend, order value and recency |
| **Region** | All of India, 77,223 customers who ordered at least once |
| **What the data shows** | **High-value** customers (21,280, about 28%) bring in **74.1%** of all spending. **Occasional** customers (12,619) order once but spend big, ₹16,438 per order on average, and bring in 21.7%. **Regular** customers (21,854) order often but spend little (3.4%). **At-risk** customers (21,470) placed one small order about 540 days ago and bring in 0.7%. |
| **Solution** | Spend retention budget on High-value customers (loyalty rewards, priority support). Give Occasional customers a reason to place a second order. Send At-risk customers a cheap win-back offer, or nothing at all, since they are worth little. |
| **See it in** | `customer_segmentation.ipynb`, dashboard page **Customer Analytics** |

#### Finding 3: Customers buy two flavors of the same dish together

| | |
|---|---|
| **Table** | `order_history_clean.csv` (`items_in_order`), mined into `association_rules.csv` with Apriori |
| **Region** | Delhi NCR, Sep 2024 to Jan 2025, 21,321 orders |
| **What the data shows** | The strongest rule is *Murgh Amritsari Seekh Pide → Mutton Seekh Pide*: a customer who orders the first is **9.2 times** more likely than average to also order the second. Most of the 62 rules found are like this, two flavors of the same dish (e.g. *Grilled Chicken Jamaican Tender → Grilled Chicken Smoky BBQ Tender*, lift 6.1), not the classic "burger + fries" pairs. |
| **Solution** | Sell "mixed flavor" combos (two flavors of one dish at a small discount) instead of generic add-ons, and show the paired flavor as a suggestion when one is added to the cart. |
| **See it in** | `market_basket_analysis.ipynb`, dashboard page **Market Basket Analysis** |

#### Finding 4: Orders peak in the evening, in two areas of Delhi

| | |
|---|---|
| **Table** | `order_history_clean.csv` (`order_placed_at`, `subzone`, `order_status`) |
| **Region** | Delhi NCR subzones: Greater Kailash 2, Sector 4, DLF Phase 1, Sector 135, Vasant Kunj, Shahdara |
| **What the data shows** | **44%** of orders come in between 7 pm and 11 pm, peaking at 8 pm (2,912 orders). Greater Kailash 2 (7,380 orders) and Sector 4 (6,530) together account for **65%** of orders. 99.1% of orders are delivered; 158 were rejected by the restaurant. |
| **Solution** | Schedule more riders and kitchen staff from 7 to 11 pm in GK2 and Sector 4, and run promotions in the quiet afternoon hours (11 am to 5 pm) to spread demand. |
| **See it in** | `exploratory_analysis.ipynb`, dashboard page **Market Basket Analysis** |

#### Finding 5: Traffic, festivals and stacked orders slow deliveries down

| | |
|---|---|
| **Table** | `delivery_ops_clean.csv` (`time_taken_min`, `road_traffic_density`, `festival`, `weather_conditions`, `multiple_deliveries`, `city`) |
| **Region** | Indian cities grouped by type: Metropolitan, Urban, Semi-Urban; 45,584 deliveries |
| **What the data shows** | The average delivery takes 26.3 min. It takes **45.5 min on festival days** against 26.0 min on normal days (+19.5 min). It takes 31.2 min in traffic jams against 21.3 min in low traffic, and 21.9 min in sunny weather against 28.9 min in fog or cloud. A rider carrying 3 extra orders takes 47.8 min against 22.9 min with none. Metropolitan deliveries average 27.3 min and Urban 23.0 min. |
| **Solution** | Add riders on festival days, limit how many orders one rider can stack (at most 1 extra), and show longer delivery estimates when traffic is heavy or the weather is bad, so customers aren't let down. |
| **See it in** | `exploratory_analysis.ipynb`, dashboard page **Delivery Performance** |

#### Finding 6: Re-ordering can't be predicted from order history

| | |
|---|---|
| **Table** | `customer_features.csv` → `customer_retention_predictions.csv` |
| **Region** | All of India, 71,689 customers with enough history to test |
| **What the data shows** | Only **19.0%** of customers ordered again in the 180-day test window. Logistic Regression, Decision Tree and Random Forest all scored ROC-AUC of about 0.5, the same as a coin toss. A customer's past orders, spend and recency don't tell us whether they will come back. |
| **Solution** | Don't build retention campaigns on order history alone. Collect the data that's missing (ratings, complaints, delivery experience, discounts used) for the same customer IDs, then try again. This is a useful negative result: it stops the business from trusting a model that doesn't work. |
| **See it in** | `customer_prediction.ipynb`, dashboard page **Customer Analytics** (Historical Retention Rate card) |

### How the data flows

```text
Data/raw/                              notebooks/                    Data/processed/                     Dashboard page
-----------------------------------    --------------------------    --------------------------------    ---------------------------
Zomato_Database/                       data_preprocessing            users, restaurants, food,           Business Overview
  users, restaurant, food,       --->  exploratory_analysis    --->  menu, orders _clean.csv       --->  Customer Analytics
  menu, orders .csv                    customer_segmentation         customer_segments.csv               Food & Restaurant Analytics
                                       customer_prediction           customer_retention_predictions.csv

Food Delivery Order History Data/ ---> data_preprocessing      --->  order_history_clean.csv       --->  Market Basket Analysis
  order_history_kaggle_data.csv        market_basket_analysis        association_rules.csv

Zomato Delivery Operations       ---> data_preprocessing      --->  delivery_ops_clean.csv        --->  Delivery Performance
  Zomato Dataset.csv                   exploratory_analysis
```

The three datasets don't share customer or restaurant IDs, so each one is analyzed separately and each dashboard page uses only its own dataset.

---

## Objectives

* Analyze food delivery transaction data to identify meaningful patterns.
* Segment customers based on their purchasing behavior.
* Discover frequently purchased food combinations.
* Analyze restaurant and food-category performance.
* Identify trends in orders, revenue, ratings, and delivery behavior.
* Develop predictive models for customer retention or repeat ordering.
* Build an interactive Business Intelligence dashboard.
* Generate actionable business recommendations from the discovered insights.

---

## Data Mining Components

### 1. Customer Segmentation

The customer segmentation notebook applies K-Means to group customers by purchasing behavior.

The clustering uses order count, total spending, average order value, order frequency, and recency.

The resulting customer segments are:

* High-value customers
* Regular customers
* Occasional customers
* At-risk customers

**Status:** done in [`notebooks/customer_segmentation.ipynb`](notebooks/customer_segmentation.ipynb). Output: `Data/processed/customer_segments.csv`. The four segments came out fairly balanced (12,600-22,000 customers each), but separate mainly along spend/order-count - recency turned out to be a much weaker signal than the RFM framing above assumes for this dataset.

---

### 2. Market Basket Analysis

The market basket notebook applies Apriori association-rule mining to identify food items frequently ordered together.

Illustrative examples (not findings from this dataset):

```text
Pizza → Coke
Burger → Fries
Biryani → Raita
Pizza → Garlic Bread
```

The rules are evaluated using:

* Support
* Confidence
* Lift

These patterns can help identify opportunities for product recommendations, combo offers, and cross-selling.

**Status:** done in [`notebooks/market_basket_analysis.ipynb`](notebooks/market_basket_analysis.ipynb) using `mlxtend`. Output: `Data/processed/association_rules.csv` (62 rules, `min_support=0.003`, filtered to `lift > 1`). The strongest rules in this dataset are same-dish flavor-variant pairs (e.g. `Murgh Amritsari Seekh Pide -> Mutton Seekh Pide`, lift ~9.2) rather than the generic combos above - a "mixed flavor bundle" is the more natural product here than a generic add-on offer.

---

### 3. Customer Retention Prediction

The retention notebook evaluates whether available customer-history data can predict another order in the following six months.

Models evaluated:

* Logistic Regression
* Decision Tree
* Random Forest

The features tested are order count, total spending, average order value, order frequency, recency, age, and family size.

**Status:** done in [`notebooks/customer_prediction.ipynb`](notebooks/customer_prediction.ipynb), comparing all three named algorithms with a time-based 180-day holdout to avoid leakage. `Ratings`/`Discount usage`/`Delivery experience` come from the Food Delivery Order History dataset, which uses its own anonymized customer ID that doesn't join to the Zomato_Database customers used for this model - so those three features aren't included. **Result: none of the three models beat random chance (ROC-AUC ~0.49-0.51).** Direct checks found near-zero feature correlations and almost flat repeat-order rates by order count and recency. The available features do not support useful retention prediction in this dataset. Output (`Data/processed/customer_retention_predictions.csv`) should be read as a methodology demonstration, not a production retention score.

---

## Business Intelligence

DineSmart includes an interactive Microsoft Power BI dashboard for exploring the analyzed food delivery data.

**Status:** built as a Power BI Project at [`dashboard/DineSmart.pbip`](dashboard/DineSmart.pbip), with the four sections below plus a fifth Delivery Performance page. [`dashboard/README.md`](dashboard/README.md) covers opening it and the numbers to check after the first refresh. [`dashboard/data_model.md`](dashboard/data_model.md) is the design reference.

### Key Metrics

* Total Orders
* Total Revenue
* Total Customers
* Average Order Value
* Customer Retention
* Top Restaurants
* Top Food Categories
* Orders by Time
* Orders by Location
* Customer Segments
* Restaurant Ratings
* Delivery Performance

### Dashboard Sections

#### Business Overview

* Revenue trends
* Order trends
* Popular categories
* Top-performing restaurants

#### Customer Analytics

* Customer segments
* Spending behavior
* Order frequency
* High-value customers
* Customer retention

#### Food and Restaurant Analytics

* Popular food categories
* Restaurant performance
* Average ratings
* Revenue by restaurant
* Orders by location

#### Market Basket Analysis

* Frequently purchased combinations
* Association rules
* Product recommendation opportunities

---

## Project Workflow

```text
                 Food Delivery Dataset
                          |
                          v
                 Data Preprocessing
                          |
                          v
                Exploratory Analysis
                          |
              +-----------+-----------+
              |                       |
              v                       v
         Data Mining             BI Analysis
              |                        |
       +------+------+                 |
       |      |      |                 |
       v      v      v                 v
    K-Means Apriori  ML           Power BI
       |      |      |                 |
       +------+------+-----------------+
                          |
                          v
                   Business Insights
                          |
                          v
                   Recommendations
```

---

## Technology Stack

### Data Processing and Analysis

* Python
* Pandas
* NumPy
* Matplotlib

### Data Mining and Machine Learning

* Scikit-learn
* MLxtend
* K-Means Clustering
* Apriori Algorithm
* Classification Models

### Business Intelligence

* Microsoft Power BI

### Development Tools

* Jupyter Notebook
* VS Code
* Git
* GitHub

---

## Project Structure

```text
DineSmart/
|
├── Data/
│   ├── raw/
│   │   ├── Food Delivery Order History Data/
│   │   ├── Zomato Delivery Operations Analytics Dataset/
│   │   └── Zomato_Database/
│   └── processed/
│       ├── users_clean.csv
│       ├── restaurants_clean.csv
│       ├── food_clean.csv
│       ├── menu_clean.csv
│       ├── orders_clean.csv
│       ├── customer_features.csv
│       ├── customer_segments.csv
│       ├── customer_retention_predictions.csv
│       ├── order_history_clean.csv
│       ├── delivery_ops_clean.csv
│       └── association_rules.csv
|
├── notebooks/
│   ├── data_preprocessing.ipynb
│   ├── exploratory_analysis.ipynb
│   ├── customer_segmentation.ipynb
│   ├── market_basket_analysis.ipynb
│   └── customer_prediction.ipynb
|
├── dashboard/
│   ├── README.md
│   ├── data_model.md
│   ├── DineSmart.pbip             (open this in Power BI Desktop)
│   ├── DineSmart.SemanticModel/   (data model: Power Query, relationships, DAX)
│   └── DineSmart.Report/          (report pages and visuals)
|
├── business_insights.md
├── requirements.txt
└── README.md
```

Every notebook uses paths relative to `notebooks/`. Install the dependencies and start Jupyter from that directory before running the notebooks in the listed order.

---

## Dataset

DineSmart uses three public Kaggle datasets, kept under `Data/raw/`. They come from different platforms and **do not share customer or restaurant IDs with each other** - each is cleaned and analyzed as its own domain rather than merged into one table:

1. **`Zomato_Database/`** - a relational set (`users`, `restaurant`, `food`, `menu`, `orders`) keyed by `user_id`/`r_id`/`f_id`. The relational core used for customer segmentation, restaurant/food performance, and retention prediction.
2. **`Food Delivery Order History Data/`** - a single flat file of itemized orders (`order_history_kaggle_data.csv`) with baskets, ratings, discounts, and cancellations, keyed by its own hashed customer ID. Used for market basket analysis and order-level revenue/rating analysis.
3. **`Zomato Delivery Operations Analytics Dataset/`** - a single flat file of delivery legs (`Zomato Dataset.csv`) with weather, traffic, and delivery time, with no customer or restaurant keys at all. Used for delivery performance analysis.

`notebooks/data_preprocessing.ipynb` documents every cleaning decision (missing values, malformed fields, referential integrity) made against each dataset, and writes the cleaned output to `Data/processed/`.

---

## Expected Outcomes

The completed analyses answer these business questions; findings and recommendations are in [`business_insights.md`](business_insights.md):

* Which customer segments generate the most revenue?
* What food combinations are frequently ordered together?
* Which restaurants and food categories perform best?
* What are the peak ordering periods?
* Which customers are likely to become inactive?
* What factors influence customer retention?
* What strategies can improve customer engagement and revenue?

---

## Future Scope (not currently implemented)

The following are possible extensions beyond the completed notebooks, five-page dashboard, and business insights report:

* Real-time food delivery analytics
* Personalized food recommendation system
* Restaurant-level performance prediction
* Demand forecasting
* Geographical hotspot analysis
* Automated business reports
* Real-time Power BI integration

---

## Contributing

Contributions are welcome.

To contribute:

1. Fork the repository.
2. Create a new branch for your changes.
3. Make your changes and test them.
4. Commit your changes with a clear commit message.
5. Push the branch to your fork.
6. Open a Pull Request describing your changes.

For major changes, please open an issue first to discuss the proposed changes.

---

## License

This project is licensed under the **MIT License**. 
