---
layout: post
title: Medicare Inpatient Cost Variation (Databricks Lakehouse)
image: "/posts/cms-medicare-cover.png"
tags: [Databricks, Spark SQL, Python, Data Engineering]
---

How much does the same inpatient procedure cost across hospitals and regions, and where are the biggest outliers and savings opportunities? This project answers that with an end-to-end lakehouse pipeline on CMS Medicare data. *This is an ongoing project (IMT 600 Independent Study, Autumn 2026), and this page will be updated as new layers are completed.*

# Table of Contents

- [00. Project Overview](#overview-main)
    - [Context](#overview-context)
    - [Actions](#overview-actions)
    - [Progress](#overview-progress)
- [01. The Data](#data)
- [02. Architecture](#architecture)
- [03. Bronze Layer](#bronze)
- [04. Silver Layer](#silver)
- [05. Roadmap](#roadmap)

---

# 00. Project Overview <a name="overview-main"></a>

### Context <a name="overview-context"></a>

Hospitals can charge and be paid very different amounts for the same diagnosis-related group (DRG). Understanding that variation, and which hospitals and regions sit far from the norm, points to where cost savings are most likely.

### Actions <a name="overview-actions"></a>

- Profiled every column of the raw CMS file in Python before loading anything
- Built a medallion (Bronze, Silver, Gold) pipeline in Databricks using Delta Lake and Spark SQL
- Added validation checks and Delta constraints so every layer reconciles back to the source
- Planned: statistical analysis in Python, an interactive Tableau dashboard, and a written findings report

### Progress <a name="overview-progress"></a>

- Bronze and Silver layers complete and validated (all checks passing)
- 32 validation checks reconcile Spark's output to values computed independently in pandas
- Gold star schema, analysis, and dashboard in progress

---

# 01. The Data <a name="data"></a>

| | |
|---|---|
| Dataset | CMS *Medicare Inpatient Hospitals, by Provider and Service* |
| Data year | 2024 |
| Grain | One row = one hospital x one MS-DRG |
| Size | 145,879 rows x 15 columns |
| Coverage | Original Medicare Part A (fee-for-service) inpatient claims only |
| Suppression | Hospital x DRG cells with fewer than 11 discharges are suppressed by CMS |

---

# 02. Architecture <a name="architecture"></a>

```
CSV (data.cms.gov)
   │  upload
   ▼
Unity Catalog volume           landing zone
   ▼
bronze_cms_inpatient           raw, all STRING, + lineage columns
   ▼
silver_cms_inpatient           typed, cleaned, markup ratio
   ▼
gold_*                         star schema + variation tables
   ├──► Python analysis
   └──► Tableau dashboard
```

---

# 03. Bronze Layer <a name="bronze"></a>

The Bronze layer lands the file exactly as received, so every later layer can be rebuilt from it.

- **Everything as STRING.** Type inference can silently turn codes like ZIP `01234` into integers and drop the leading zero, so casting is left to Silver, where it is explicit and logged.
- **Malformed lines are kept and flagged** in a `_corrupt_record` column rather than dropped.
- **Full refresh.** Re-running the notebook produces an identical table, and Delta keeps the history of every write.
- **Built-in validation.** Asserts check row count, column count, data types, and zero corrupt records.

---

# 04. Silver Layer <a name="silver"></a>

The Silver layer turns Bronze's raw strings into a typed, analysis-ready table at the same grain, without dropping a single row.

- **No rows removed.** Problem rows are flagged instead, so totals always reconcile to Bronze.
- **Derived measures.** Totals (average x discharges), a `markup_ratio` (charge / total payment) and `medicare_share` (Medicare payment / total payment).
- **Weighted roll-ups only.** Because source values are averages, every aggregate uses `SUM(total) / SUM(discharges)`.
- **Maryland flagged.** Its all-payer waiver keeps charges close to payments (markup of about 1.1 vs 5.2 elsewhere), so it is separated to avoid skewing comparisons.
- **Delta CHECK constraints** enforce the 11-discharge floor, positive amounts, Medicare payment not exceeding total payment, and code formats.

---

# 05. Roadmap <a name="roadmap"></a>

- Gold star schema with per-DRG medians and hospital deviation
- State, regional, and rural vs urban comparisons, plus hospital outliers
- Python exploratory and statistical analysis
- Prioritization and savings estimate
- Tableau dashboard and written findings report

Code, notebooks, and data quality logs are on [GitHub](https://github.com/arcbxzheng23/cms-medicare-cost-variation).
