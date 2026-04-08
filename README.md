# End-to-End Analytics Engineering & Business Intelligence
 
> *From raw transactional data to executive-ready insights — built with the Modern Data Stack.*
 
---
 
## The Story
 
Every restaurant group, e-commerce brand, or distribution business sits on a goldmine of transactional data — and most of it goes unanalyzed. This project asks a simple question: **what does it actually look like to build a proper analytics pipeline from scratch?**
 
Starting with a fictional but realistic dataset modeled after the classic Northwind trading company, I built a full analytics stack: ingested raw data into BigQuery, transformed it through a structured dbt pipeline, and surfaced the results in a multi-page Power BI dashboard designed for both executive and operational use.
 
The goal wasn't just to make charts. It was to demonstrate how a modern analyst thinks about **data modeling, business logic, and storytelling with numbers** — the full stack from warehouse to insight.
 
---
 
## Tech Stack
 
| Layer | Tool |
|---|---|
| Cloud Data Warehouse | Google BigQuery |
| Transformation | dbt (dbt-core) |
| BI & Visualization | Power BI (DAX) |
| Environment | WSL2 (Ubuntu) |
| Language | SQL, Python |
 
---
 
## Architecture
 
The pipeline follows a three-layer dbt architecture:
 
```
Raw Data (BigQuery)
      │
      ▼
┌─────────────┐
│  Staging    │  Views — light cleaning & renaming
└─────────────┘
      │
      ▼
┌─────────────┐
│  Warehouse  │  Tables — star schema, business logic
└─────────────┘
      │
      ▼
┌─────────────┐
│ Analytics   │  Incremental OBTs — aggregated, BI-ready
└─────────────┘
      │
      ▼
  Power BI Dashboard
```
 
### Data Model (Star Schema)
 
The warehouse layer is built as a clean star schema optimized for BI performance:
 
- **`fact_orders`** — order-level transactional data
- **`fact_order_details`** — line-item detail (product, quantity, unit price, discount)
- **`dim_customers`** — 29 international customers across multiple regions
- **`dim_products`** — product catalog with category and supplier references
- **`dim_employees`** — 9 employees at Northwind HQ
- **`dim_suppliers`** — 10 supplier records
- **`dim_date`** — standard date spine for time intelligence
 
> Dataset covers two full years of transactions (Jan 2024 – Dec 2025) with realistic seasonal weighting.
 
---
 
## dbt Model Walkthrough
 
### Staging Layer (`stg_*`)
Materialised as **views**. Each model maps 1:1 to a source table, standardises column names, and casts data types. No business logic lives here.
 
### Warehouse Layer (`dim_*` / `fact_*`)
Materialised as **tables**. This is where the star schema is assembled:
- Surrogate keys generated for all dimension tables
- Fact tables joined to dimension keys
- Business rules applied (e.g. revenue = unit_price × quantity × (1 - discount))
 
### Analytics OBT Layer
Materialised as **incremental** models. Pre-aggregated tables built for Power BI performance — reducing query load and enabling faster dashboard rendering.
 
---
 
## Power BI Dashboard
 
The dashboard is structured across four pages, each serving a distinct audience:
 
### 1. Executive Summary
High-level KPIs for leadership at a glance:
- Total Revenue, Orders, Average Order Value (AOV)
- Year-over-Year and Month-over-Month trends
- Revenue vs. Prior Year comparison (SMLY%)
 
### 2. Sales Performance
Operational detail for sales and commercial teams:
- Monthly revenue trend with Prior Year overlay
- Top-performing categories and products
- Order volume by region
 
### 3. Customer Analytics
Customer health and lifecycle metrics:
- New Customers YTD vs. Prior YTD
- Customer Lifetime Value (LTV) — calculated using `ALL(dim_date)` to reflect true lifetime totals regardless of date slicer context
- Repeat purchase behavior
 
### 4. Product Analysis
Inventory and product performance:
- Revenue and margin by product and category
- Top 10 products by revenue
- Supplier contribution analysis
 
### DAX Highlights
 
Key measures built for this dashboard:
 
```dax
-- Customer LTV (ignores date slicer to show true lifetime value)
Customer LTV =
CALCULATE(
    [Revenue],
    ALL(dim_date)
)
 
-- YoY Change %
YoY % =
DIVIDE(
    [Revenue TY] - [Revenue PY],
    [Revenue PY],
    BLANK()
)
 
-- New Customers YTD
New Customers YTD =
CALCULATE(
    DISTINCTCOUNT(fact_orders[customer_id]),
    DATESYTD(dim_date[date]),
    [Is New Customer Flag] = TRUE()
)
```
 
---
 
## Key Business Insights
 
A few findings from the two-year dataset:
 
- **Seasonality is real** — Q4 consistently outperforms Q1–Q2 driven by product mix and order volume, not just pricing.
- **A small customer segment drives outsized revenue** — the top 20% of customers account for a disproportionate share of LTV, validating a retention-first commercial strategy.
- **AOV is a lagging indicator** — month-over-month AOV changes are better explained by product mix shifts than by pricing changes, which matters for how you set commercial targets.
- **Employee productivity is uneven** — a handful of employees drive the majority of high-value orders, suggesting potential for coaching or incentive structure review.
 
---
 
## How to Reproduce
 
### Prerequisites
- Google Cloud account with BigQuery access
- dbt-core installed (`pip install dbt-bigquery`)
- Power BI Desktop
 
### Steps
 
```bash
# 1. Clone the repo
git clone https://github.com/your-username/northwind-analytics.git
cd northwind-analytics
 
# 2. Configure dbt profile
# Edit ~/.dbt/profiles.yml with your BigQuery project credentials
 
# 3. Run the pipeline
dbt deps
dbt run
dbt test
 
# 4. Open Power BI
# Load the .pbix file and update the BigQuery data source connection
```
 
---
 
## Project Structure
 
```
northwind-analytics/
├── dbt/
│   ├── models/
│   │   ├── staging/        # stg_* views
│   │   ├── warehouse/      # dim_* and fact_* tables
│   │   └── analytics_obt/  # incremental OBTs
│   ├── tests/
│   └── dbt_project.yml
├── sql/
│   └── seed_data/          # BigQuery insert scripts (split by table)
├── powerbi/
│   └── northwind.pbix
└── README.md
```
 
---
 
## About
 
Built by **Ziad Ehab** — Data Analyst & Analytics Engineer
 
Focused on the Modern Data Stack (BigQuery · dbt · Power BI) and growthanalytics.
 
