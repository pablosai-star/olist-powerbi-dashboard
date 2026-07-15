Olist E-Commerce — Executive Power BI Dashboard

## Overview

End-to-end Power BI solution built on the **Olist Brazilian E-Commerce** public dataset (Kaggle). The project replaces three separate Excel reports with a single governed executive app covering revenue performance, product analysis, and regional breakdown — with dynamic Row-Level Security restricting data access by region.

Dataset: 100K+ real anonymized orders from Brazil's largest e-commerce marketplace (2016–2018)

---

## Business Context

Olist leadership needed a single source of truth to replace fragmented Excel reports used by the finance, product, and regional teams. The solution had to:

- Consolidate 9 raw CSV files into a clean analytical model
- Enable drill-down from executive KPIs to product-level detail
- Restrict regional managers to their own region's data
- Be publishable as a governed app with scheduled refresh

---

## Project Architecture

```
Raw CSVs (9 files)
      ↓
Power Query — Clean & Transform
      ↓
Star Schema — 6 Dimensions + 1 Fact Table
      ↓
DAX — 11 Measures (Revenue, Profit, Time Intelligence)
      ↓
3-Page Report + RLS Security
      ↓
Power BI App (Power BI Service)
```

---

## Data Model

Star Schema with Sales as the central fact table:

| Table | Type | Description |
|-------|------|-------------|
| Sales | Fact | Orders × Products × Sellers |
| Date | Dimension | Calendar with Year, Quarter, Month |
| Products | Dimension | Product catalog with English categories |
| Customers | Dimension | Customer location data |
| Sellers | Dimension | Seller info + Brazilian region |
| Geolocation | Dimension | State-level lat/long coordinates |
| Reviews | Dimension | Customer review scores |
| Payments | Dimension | Payment methods and values |

---

## Power Query Transformations

- Filtered Orders to `delivered` status only (~96K orders)
- Built Sales fact table by merging Order Items + Orders on `order_id`
- Added synthetic `estimated_cost` column (`price × 0.65`) for profit analysis
- Translated Portuguese product categories to English via lookup merge
- Added Brazilian region mapping to Sellers (5 regions, 27 states)
- Grouped Geolocation from millions of rows to 27 state-level coordinates
- Disabled 4 staging queries (Orders, Order Items, Brazilian Regions, Product Categories)

---

## DAX Measures

### Base Measures
```dax
Total Revenue = SUM('Sales'[revenue])
Gross Profit = [Total Revenue] - [Estimated Cost]
Profit Margin = DIVIDE([Gross Profit], [Total Revenue])
Order Count = DISTINCTCOUNT('Sales'[order_id])
Avg Order Value = DIVIDE([Total Revenue], [Order Count])
Avg Review Score = AVERAGE('Reviews'[review_score])
```

### Time Intelligence
```dax
YTD Revenue = TOTALYTD([Total Revenue], 'Date'[Date])

PY Revenue = CALCULATE([Total Revenue], SAMEPERIODLASTYEAR('Date'[Date]))

YoY Growth % =
VAR Current = [Total Revenue]
VAR Prior = [PY Revenue]
RETURN DIVIDE(Current - Prior, Prior)
```

---

## Report Pages

### Page 1 — Executive Summary
- 4 KPI cards: Total Revenue, Gross Profit, Profit Margin, Order Count
- Line chart: Monthly revenue trend (2016–2018)
- Bar chart: Top 8 product categories by revenue
- Map: Revenue by seller state
- Year slicer (dropdown)
- Conditional formatting on Profit Margin (green ≥ 35%, red < 20%)
- **Drill-through** to Product Analysis page

### Page 2 — Product Analysis
- Matrix: category → product with Revenue, Cost, Profit, Margin, YoY Growth %
- Treemap: Revenue by category
- Category slicer
- Back button

### Page 3 — Regional Performance
- Bubble map: Revenue by Brazilian state
- Clustered bar: Revenue & Gross Profit by region
- Table: Top 10 states by revenue
- KPI visual: Revenue vs prior year target
- Back button

---

## Row-Level Security (RLS)

Dynamic RLS implemented via a `SecurityMapping` table:

```dax
-- Role: Regional Access
-- Filter on SecurityMapping table:
[UserEmail] = USERPRINCIPALNAME()
```

| User | Access |
|------|--------|
| manager.sudeste@olist.com | Sudeste only |
| manager.sul@olist.com | Sul only |
| manager.nordeste@olist.com | Nordeste only |
| manager.norte@olist.com | Norte only |
| manager.co@olist.com | Centro-Oeste only |
| ceo@olist.com | All regions |

---

## Key Results

| Metric | Value |
|--------|-------|
| Total Revenue | 13.59M BRL |
| Gross Profit | 4.77M BRL |
| Profit Margin | 35.06% |
| Total Orders | 99K+ |
| Period Covered | Sep 2016 – Oct 2018 |

---

## Tech Stack

- Power BI Desktop — Report authoring
- Power Query (M) — Data transformation
- DAX — Measures and calculated columns
- Power BI Service** — Publishing and governance (Pro required)

---

## Dataset

[Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) — Kaggle

---

## Author

Built as part of a structured Power BI learning path covering:
Power Query · Star Schema Modeling · DAX Fundamentals · Advanced DAX Patterns · Report Design · Row-Level Security
