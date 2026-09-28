# DineSmart Business Insights and Recommendations

This report translates the completed notebook analyses and dashboard measures into practical next steps. It is based on the three cleaned datasets in `Data/processed/` and their existing analysis outputs; it does not combine records across the datasets.

## Executive summary

- The Zomato database dashboard reports revenue down **24.0%** in the last 12 months compared with the first 12 months in its 2017–2020 history. Investigate where the decline is concentrated before choosing a response.
- Customer segments are separated mainly by spend and order count. The retention model does not reliably identify who will order again, so its probabilities are not suitable for customer targeting.
- Food-order baskets show a few strong flavor-pair associations, but each rule covers a small share of orders. Treat them as candidates for small, measured bundle tests.
- Delivery time is substantially higher for heavily stacked trips and in dense traffic. Delivery operations are a candidate for targeted operational experiments.

## Findings and recommended actions

### 1. Investigate the sales decline before applying broad promotions

The Zomato database contains 146,979 cleaned orders. Its dashboard reports a **24.0% decrease in revenue between the first and last 12-month periods**. The notebook also shows revenue and order-volume trends over time. The history ends in 2020, so this is a historical pattern in this dataset, not evidence about current platform performance.

**Recommended action:** Use the Business Overview page to break the trend down by year, city, and restaurant brand. Identify whether the change is concentrated in a few locations or brands, then investigate those areas before testing a targeted offer or availability change. Track order count and revenue alongside average order value so an apparent recovery in one measure does not hide a decline in another.

### 2. Treat customer segments as spend profiles, not churn predictions

The segmentation output covers **77,223 customers with valid orders**: 21,280 High-value, 21,854 Regular, 12,619 Occasional, and 21,470 At-risk. The clusters are driven mostly by spending and order counts; recency has a weaker effect. The dashboard reports that High-value customers account for **74.1% of customer revenue**. The segment label “At-risk” should therefore be read as a segment name, not a proven prediction that those customers will leave.

The retention models achieve ROC-AUC values around **0.49–0.51**, and the notebook finds repeat-order rates are nearly flat across the available history-based features. This means the saved probabilities do not provide a useful ranking for win-back campaigns.

**Recommended action:** Use segment profiles to describe historical value and to frame exploratory, separately measured engagement tests. Do not target customers using `retention_probability` or interpret the “At-risk” label as confirmed churn risk. A future retention model would need stronger behavioral signals, such as app engagement or abandoned carts, plus validation on a more representative, current dataset.

### 3. Test flavor-pair bundles as a small, measurable experiment

The Food Delivery Order History dataset contains **21,321 orders** and 62 association rules. The highest-lift pair, Murgh Amritsari Seekh Pide → Mutton Seekh Pide, has lift **9.18**, but support is only **0.33%** and confidence is about **12%** in that direction. Many of the strongest rules pair flavor variants of the same dish rather than unrelated products.

**Recommended action:** If product assortment and margins allow, test a small number of flavor-pair recommendations or bundles against a control group. Evaluate conversion, incremental order value, and contribution margin. The low support means these patterns should not be assumed to apply to most customers or orders.

### 4. Pilot delivery changes around stacking and congestion

In the Delivery Operations dataset (45,584 delivery legs), average delivery time is about **22.9 minutes** with no additional orders on a trip, **40.5 minutes** with two additional orders, and **47.8 minutes** with three. Average time is also **31.2 minutes** in Jam traffic versus **21.3 minutes** in Low traffic. These are associations in observational data; they do not establish that stacking or traffic alone caused the delays.

**Recommended action:** Review dispatch rules for heavily stacked trips and busy traffic periods. Pilot a lower stacking threshold or a traffic-aware dispatch adjustment in a limited area and compare delivery time and completed deliveries with a comparable baseline. Festival deliveries average about **45.5 minutes**, compared with **26.0 minutes** on non-festival days, but there are only 896 festival records, so validate this result before committing extra capacity.

### 5. Review discount use alongside order economics

In the separate Food Delivery Order History dataset, **74.2% of orders are marked as discounted**. This is not the same dataset as the Zomato transaction history, so the figure cannot be generalized to all DineSmart orders or joined to its customer segments.

**Recommended action:** Use the order-history page to compare discounted and non-discounted orders by order value, status, and ratings. Assess margin impact before broadening or reducing discounts; discount prevalence alone does not show whether promotions are incremental or profitable.

## Scope and interpretation limits

- The three datasets represent different platforms and use unrelated identifiers. Keep their findings separate; do not join customers, restaurants, or orders across domains.
- The datasets cover different periods: the Zomato transaction history is 2017–2020, delivery operations is from 2022, and itemized order history is from 2024–2025. Their measures are not a single synchronized view of one business.
- These are descriptive and observational analyses. They identify patterns to investigate, not causal effects or guaranteed outcomes from the recommended actions.
- The retention model's chance-level performance is a negative result. Its scores are retained as a modeling demonstration, not a production decision tool.

## Where to explore the evidence

- [Business Overview and Customer Analytics dashboard pages](dashboard/README.md#pages)
- [Market Basket Analysis and Delivery Performance dashboard pages](dashboard/README.md#pages)
- [Notebook findings and project methods](README.md#status)
- [Dataset separation and model caveats](dashboard/data_model.md#the-one-rule-that-matters-most-three-unrelated-domains)
