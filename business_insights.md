# DineSmart Business Insights and Recommendations

This report answers the seven business questions in the project README using the completed notebook results, processed CSVs, and Power BI measures. The three input datasets come from separate platforms, cover different time periods, and use unrelated identifiers. Findings below stay within their source dataset; figures from different domains must not be joined or treated as one synchronized business view.

## Answers to the Expected Outcomes

### 1. Which customer segments generate the most revenue?

**High-value customers generate the largest revenue share.** The segmentation output contains 77,223 customers with valid orders. The High-value segment contains 21,280 customers, and the dashboard reports that it contributes **74.1% of customer revenue**. The other segments contain 21,854 Regular, 12,619 Occasional, and 21,470 At-risk customers.

The clusters separate mainly by spend and order count; recency contributes much less. “At-risk” is a descriptive cluster label, not a validated churn prediction.

**Recommendation:** Use the High-value segment to understand which products, restaurants, and experiences accompany historical value. If testing loyalty or recognition offers, compare outcomes with a control group and monitor incremental revenue and margin. Do not use the At-risk label alone to decide who receives a win-back campaign.

### 2. What food combinations are frequently ordered together?

The Market Basket Analysis notebook found **62 association rules** in 21,321 itemized orders. The strongest rule is Murgh Amritsari Seekh Pide → Mutton Seekh Pide: **9.18 lift**, **0.33% support**, and **12.0% confidence** in that direction. The reverse rule has the same support and 25.2% confidence. Many of the strongest rules pair flavor variants of the same dish, rather than unrelated products. The most frequent individual item is Bageecha Pizza, appearing in about **14.6%** of baskets; that frequency does not by itself imply a profitable cross-sell.

**Recommendation:** Pilot a small set of flavor-pair suggestions or bundles against a control group. Measure incremental attachment, order value, and contribution margin. The strongest rule is rare across all orders, so treat it as a focused test rather than a universal recommendation.

### 3. Which restaurants and food categories perform best?

In the Zomato transaction dataset, aggregating outlets by restaurant name puts **Domino’s Pizza** first, with about **5.03 million recorded sales units**. The ranking is by brand name across outlets: the dataset has 146,979 orders across 146,858 outlets, so ranking individual outlets would mostly rank one-off orders rather than sustained restaurant performance.

For cuisine, **Chinese** has the most restaurant listings (**36,464**), followed by North Indian (**32,537**) and Indian (**25,716**). This is a measure of restaurant availability, not cuisine sales; a restaurant can list multiple cuisines, so these counts overlap. In the separate itemized-order dataset, Bageecha Pizza is the most frequent individual item at 14.6% support, but the datasets cannot be joined to connect it to the Zomato restaurant ranking.

**Recommendation:** Use brand-level revenue to prioritize restaurant performance reviews, and cuisine counts to understand supply coverage. Do not describe the cuisine listing counts as customer demand. For demand-led category decisions, compare item or cuisine sales within a single source with a consistent category definition.

### 4. What are the peak ordering periods?

In the Food Delivery Order History dataset, orders peak at **8 PM (2,912 orders)**. The 7–10 PM hours account for **9,375 of 21,321 orders (44.0%)**. This hourly analysis belongs to that dataset and its 2024–2025 time period; it is not a time-of-day view of the separate Zomato transaction dataset.

**Recommendation:** Use 7–10 PM as a period to investigate for staffing, kitchen capacity, and delivery readiness in a comparable operation. Validate the pattern against current, local order data before changing schedules.

### 5. Which customers are likely to become inactive?

**The current data and model do not reliably identify individual customers likely to become inactive.** The three tested classifiers have ROC-AUC around **0.49–0.51**, close to random ranking. The saved `retention_probability` values are a modeling demonstration and should not be used to select customers for win-back campaigns.

**Recommendation:** Do not operationalize these scores. Collect richer engagement signals and evaluate retention with a time-based holdout on representative platform data before using a score for outreach.

### 6. What factors influence customer retention?

Among the tested Zomato customer-history and demographic features, the notebook found no meaningful predictive relationship with repeat ordering. Feature correlations with the retention label are close to zero (about **−0.0001 to 0.0065**). One-time customers reordered at **19.2%** (35,852 customers), compared with **18.7%** for customers with multiple prior orders (35,837). Recency buckets were similarly flat, mostly around **18.3–19.7%**.

This is evidence that the measured features in this dataset do not explain retention; it does not establish that retention has no drivers in a real platform. The Food Delivery Order History dataset’s ratings and discounts use unrelated customer IDs and cannot be joined to these Zomato customers.

**Recommendation:** Instrument and test additional signals such as app engagement, cart abandonment, support contacts, and delivery experience in a dataset where those events can be linked reliably to orders. Reassess feature effects only after validating data coverage and the retention label.

### 7. What strategies can improve customer engagement and revenue?

The analysis supports **experiments**, not guaranteed outcomes:

- Test flavor-pair recommendations for the strongest association rules; measure incremental orders and margin against a control group.
- Use the High-value segment to frame a measured loyalty or service test, but do not treat the At-risk segment or retention probabilities as proven churn signals.
- Investigate the **24.0% decrease in revenue** between the first and last 12-month periods in the 2017–2020 Zomato history by city and restaurant brand before choosing a response. This historical result does not describe current performance.
- Plan for the evening peak in comparable operations: 44.0% of itemized orders fall between 7 and 10 PM, with the maximum at 8 PM.
- In the separate 2022 Delivery Operations data, average delivery time is **31.2 minutes in Jam traffic versus 21.3 minutes in Low traffic**. Trips with two or three other orders average **40.5 and 47.8 minutes**, compared with **22.9 minutes** with no other orders. Pilot dispatch or capacity adjustments in limited settings and compare with a suitable baseline; these are observational associations, not proof of causation.
- **74.2%** of orders in the separate Food Delivery Order History dataset are marked discounted. Evaluate discounts by order value, outcome, and margin before increasing or reducing them; discount prevalence alone does not show incremental impact.

## Scope and interpretation limits

- Zomato transactions cover 2017–2020, Delivery Operations covers 2022, and Food Delivery Order History covers 2024–2025. The results are from different datasets, not one continuous platform history.
- Customer, restaurant, and delivery identifiers do not connect across the three domains. Do not combine their rows or attribute one domain’s outcomes to another domain’s customers.
- Restaurant cuisine counts describe listing availability, not cuisine revenue. Multi-cuisine restaurants appear under more than one cuisine.
- Association rules describe co-occurrence, not causation. Low support limits how widely a rule can be applied.
- The retention result is negative for the current features and dataset. The prediction output is not a production targeting tool.

## Dashboard evidence

The existing Power BI pages present the underlying figures and patterns: [Business Overview and Customer Analytics](dashboard/README.md#pages), [Food & Restaurant Analytics](dashboard/README.md#pages), [Market Basket Analysis](dashboard/README.md#pages), and [Delivery Performance](dashboard/README.md#pages). This report adds the question-by-question interpretation and recommended actions; it does not add or change dashboard visuals.
