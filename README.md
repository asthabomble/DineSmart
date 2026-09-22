# DineSmart

## Data Mining and Business Intelligence for Food Delivery Platforms

DineSmart is a Data Mining and Business Intelligence project that analyzes food delivery data to uncover customer behavior, ordering patterns, restaurant performance, and actionable business insights.

The project combines data preprocessing, exploratory data analysis, data mining techniques, machine learning, and interactive Business Intelligence dashboards to support data-driven decision-making for food delivery businesses.

---

## Status

All data preprocessing, exploratory analysis, and data-mining notebooks are complete and runnable end to end. The Power BI dashboard itself is not yet built - a full data model and DAX reference is ready at [`dashboard/data_model.md`](dashboard/data_model.md) for whoever builds it next.

| Notebook | Component | Key finding |
| --- | --- | --- |
| [`notebooks/data_preprocessing.ipynb`](notebooks/data_preprocessing.ipynb) | Cleans all three raw datasets into `Data/processed/` | - |
| [`notebooks/exploratory_analysis.ipynb`](notebooks/exploratory_analysis.ipynb) | Revenue, customer, restaurant, delivery trends | Revenue and order volume are both trending down over the dataset's history |
| [`notebooks/customer_segmentation.ipynb`](notebooks/customer_segmentation.ipynb) | K-Means customer segmentation | Segments separate mainly by spend and order count; recency has a much smaller effect than expected |
| [`notebooks/market_basket_analysis.ipynb`](notebooks/market_basket_analysis.ipynb) | Apriori market basket analysis | Top rules are mostly same-dish flavor-variant pairs (e.g. two flavors of the same seekh dish), not classic cross-sell pairs |
| [`notebooks/customer_prediction.ipynb`](notebooks/customer_prediction.ipynb) | Retention prediction (Logistic Regression, Decision Tree, Random Forest) | Negative result - ROC-AUC ~0.5 for all three models; re-ordering is statistically independent of a customer's own history in this dataset |

See each notebook's own summary section for the full reasoning behind these results.

---

## Problem Statement

Food delivery platforms generate large volumes of data from customer orders, restaurants, food categories, ratings, delivery times, locations, and purchasing behavior.

However, raw transactional data alone does not provide meaningful business insights.

DineSmart aims to transform food delivery data into actionable information by identifying:

* Customer segments and purchasing behavior
* Popular food categories and restaurants
* Peak ordering periods
* Frequently ordered food combinations
* High-value and low-engagement customers
* Factors affecting customer retention
* Business opportunities for improving sales and customer engagement

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

K-Means Clustering will be used to group customers based on their purchasing behavior.

Potential features include:

* Number of orders
* Total spending
* Average order value
* Order frequency
* Recency of purchase

Possible customer segments:

* High-value customers
* Regular customers
* Occasional customers
* At-risk customers

**Status:** done in [`notebooks/customer_segmentation.ipynb`](notebooks/customer_segmentation.ipynb). Output: `Data/processed/customer_segments.csv`. The four segments came out fairly balanced (12,600-22,000 customers each), but separate mainly along spend/order-count - recency turned out to be a much weaker signal than the RFM framing above assumes for this dataset.

---

### 2. Market Basket Analysis

Apriori Association Rule Mining will be used to identify food items or categories that are frequently purchased together.

Example patterns:

```text
Pizza → Coke
Burger → Fries
Biryani → Raita
Pizza → Garlic Bread
```

The analysis will use:

* Support
* Confidence
* Lift

These patterns can help identify opportunities for product recommendations, combo offers, and cross-selling.

**Status:** done in [`notebooks/market_basket_analysis.ipynb`](notebooks/market_basket_analysis.ipynb) using `mlxtend`. Output: `Data/processed/association_rules.csv` (62 rules, `min_support=0.003`, filtered to `lift > 1`). The strongest rules in this dataset are same-dish flavor-variant pairs (e.g. `Murgh Amritsari Seekh Pide -> Mutton Seekh Pide`, lift ~9.2) rather than the generic combos above - a "mixed flavor bundle" is the more natural product here than a generic add-on offer.

---

### 3. Customer Retention Prediction

Machine learning models may be used to identify customers who are likely to place another order or become inactive.

Potential algorithms include:

* Logistic Regression
* Decision Tree
* Random Forest

Potential features include:

* Order frequency
* Total spending
* Average order value
* Recency
* Ratings
* Discount usage
* Delivery experience

**Status:** done in [`notebooks/customer_prediction.ipynb`](notebooks/customer_prediction.ipynb), comparing all three named algorithms with a time-based 180-day holdout to avoid leakage. `Ratings`/`Discount usage`/`Delivery experience` come from the Food Delivery Order History dataset, which uses its own anonymized customer ID that doesn't join to the Zomato_Database customers used for this model - so those three features aren't included. **Result: none of the three models beat random chance (ROC-AUC ~0.49-0.51).** Direct correlation checks confirmed there's no signal to find - in this dataset, whether a customer re-orders is statistically independent of their past order count, spend, or recency. Output (`Data/processed/customer_retention_predictions.csv`) should be read as a methodology demonstration, not a production retention score.

---

## Business Intelligence

An interactive dashboard will be developed using Microsoft Power BI to provide an overview of the food delivery business.

**Status:** not yet built. [`dashboard/data_model.md`](dashboard/data_model.md) has the full data model (star schema, relationships, Power Query prep steps, and DAX for every metric below) ready to wire up in Power BI Desktop against the CSVs in `Data/processed/`.

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
│   ├── data_model.md
│   └── DineSmart.pbix        (not yet built)
|
├── requirements.txt
└── README.md
```

Every notebook reads from and writes to `Data/processed/` using paths relative to `notebooks/`, so they run correctly from the `notebooks/` working directory without any project installation step.

---

## Dataset

DineSmart uses three public Kaggle datasets, kept under `Data/raw/`. They come from different platforms and **do not share customer or restaurant IDs with each other** - each is cleaned and analyzed as its own domain rather than merged into one table:

1. **`Zomato_Database/`** - a relational set (`users`, `restaurant`, `food`, `menu`, `orders`) keyed by `user_id`/`r_id`/`f_id`. The relational core used for customer segmentation, restaurant/food performance, and retention prediction.
2. **`Food Delivery Order History Data/`** - a single flat file of itemized orders (`order_history_kaggle_data.csv`) with baskets, ratings, discounts, and cancellations, keyed by its own hashed customer ID. Used for market basket analysis and order-level revenue/rating analysis.
3. **`Zomato Delivery Operations Analytics Dataset/`** - a single flat file of delivery legs (`Zomato Dataset.csv`) with weather, traffic, and delivery time, with no customer or restaurant keys at all. Used for delivery performance analysis.

`notebooks/data_preprocessing.ipynb` documents every cleaning decision (missing values, malformed fields, referential integrity) made against each dataset, and writes the cleaned output to `Data/processed/`.

---

## Expected Outcomes

DineSmart aims to answer key business questions such as:

* Which customer segments generate the most revenue?
* What food combinations are frequently ordered together?
* Which restaurants and food categories perform best?
* What are the peak ordering periods?
* Which customers are likely to become inactive?
* What factors influence customer retention?
* What strategies can improve customer engagement and revenue?

---

## Future Scope

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
