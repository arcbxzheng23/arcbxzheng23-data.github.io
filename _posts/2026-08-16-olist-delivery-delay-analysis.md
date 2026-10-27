---
layout: post
title: How Late Deliveries Drive 1-Star Reviews at Olist
image: "/posts/olist-delivery-cover.png"
tags: [Python, SQL, DuckDB, Tableau]
---

In this project I analyzed about 100,000 orders from Olist, a Brazilian e-commerce marketplace, to answer two questions: how much do late deliveries cost in customer satisfaction, and where should Olist fix them first?

# Table of Contents

- [00. Project Overview](#overview-main)
    - [Context](#overview-context)
    - [Actions](#overview-actions)
    - [Results](#overview-results)
- [01. Data Overview & Preparation](#data-overview)
- [02. Late Deliveries vs Review Scores](#reviews)
- [03. Where Late Deliveries Happen](#geography)
- [04. Recommendation & Impact](#recommendation)
- [05. Next Steps](#next-steps)

---

# 00. Project Overview <a name="overview-main"></a>

### Context <a name="overview-context"></a>

Olist connects small sellers to customers across Brazil, and every order is shipped against a delivery date that Olist itself promises at checkout. Customers then leave a 1 to 5 star review. If late deliveries are dragging those reviews down, the business needs to know how big the effect is and which shipping lanes to fix first.

### Actions <a name="overview-actions"></a>

- Explored and cleaned the public Olist dataset in Python (pandas, matplotlib)
- Re-derived every key number in SQL using DuckDB, querying the raw CSVs directly
- Built a one-row-per-order extract and published two interactive Tableau Public dashboards
- Turned the findings into a prioritized recommendation with a quantified impact estimate

### Results <a name="overview-results"></a>

- **8.1%** of delivered orders (7,826 of 96,470) arrived after the promised date
- **46%** of late orders received a 1-star review, compared with **6.6%** of on-time orders (a 7x rate)
- Lateness is **geographic, not product-driven**: category late rates are flat at around 8 to 10%, while long-haul states in the Northeast reach 15 to 24%
- Bringing Rio de Janeiro to the national average would recover an estimated **250 to 360 one-star reviews**

---

# 01. Data Overview & Preparation <a name="data-overview"></a>

The data is the [Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce), covering roughly 100K orders from 2016 to 2018 across orders, customers, items, products, and reviews tables.

A few data quality decisions shaped the analysis:

- 8 orders marked "delivered" had no delivery timestamp, so they were excluded
- 551 orders had multiple reviews; I kept the most recent review per order (this did not change the results, the gap was 1.73 stars before and after)
- An order counts as **late** when it arrived after Olist's own estimated delivery date
- State rankings only include states with at least 500 delivered orders, so low-volume states don't top the list on noise

---

# 02. Late Deliveries vs Review Scores <a name="reviews"></a>

Late orders average **2.57 stars** compared with **4.29 stars** for on-time orders. The share of 5-star reviews drops from 62% to 22%, and the share of 1-star reviews jumps from 6.6% to 46%.

![Share of orders with a 1-star review: on-time vs late](/img/posts/one_star_rate.png)

---

# 03. Where Late Deliveries Happen <a name="geography"></a>

When I broke late rates down by product category, they were nearly identical everywhere. The real driver is **geography**: lateness is concentrated on the long-haul lanes from the São Paulo hub to the Northeast and to Rio de Janeiro.

![Late delivery rate by state](/img/posts/state_late_rate.png)

You can explore both views interactively on Tableau Public:

- [State late-rate map](https://public.tableau.com/views/OlistDeliveryAnalysis_17869366581830/Sheet1)
- [On-time vs late review scores](https://public.tableau.com/views/OlistDeliveryAnalysis_17869366581830/Sheet2)

---

# 04. Recommendation & Impact <a name="recommendation"></a>

Late deliveries are the single biggest driver of 1-star reviews, and they are a shipping-lane problem rather than a product problem. I'd prioritize two moves:

1. **Target Rio de Janeiro first.** It pairs a high late rate (13.5%) with the largest order volume among late-prone states (12,350 orders), so carrier and SLA improvements there recover the most reviews per dollar.
2. **Recalibrate delivery-date promises for the Northeast.** Because "late" is measured against Olist's own estimate, setting more realistic dates for Alagoas, Maranhão and Ceará would shrink the gap that triggers 1-star reviews, even before shipping gets faster.

**Quantified impact:** bringing Rio de Janeiro's late rate down to the national average would recover an estimated 250 to 360 one-star reviews from a single lane.

---

# 05. Next Steps <a name="next-steps"></a>

- Identify which sellers and origin regions drive lateness, now that product category has been ruled out
- Estimate what a late order is worth in lost repeat purchases

The full code (pandas exploration notebook, DuckDB SQL layer, and Tableau build notes) is on [GitHub](https://github.com/arcbxzheng23/olist-delivery-analysis).
