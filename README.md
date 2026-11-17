# FMCG Sales & Inventory Decision Intelligence | Microsoft Fabric

> **Portfolio project:** An analytics workflow built in Microsoft Fabric using PySpark, Spark SQL, Pandas and a Lakehouse to turn distributor-level FMCG reports into product, retailer, sales, returns and inventory insights.

## Project overview

This project explores how operational Excel reports from an FMCG distribution environment can be ingested into a Microsoft Fabric Lakehouse and analysed to support business decisions.

The notebook uses Pandas to read Excel files, PySpark to convert the data into Spark DataFrames, and Spark SQL to build analytical summaries. The analysis focuses on product demand, sales performance, discounts, retailer activity, inventory health, salesperson performance and product returns.

**Important:** The source data belongs to a company and is not included in this repository. The project notebook references Fabric Lakehouse file paths and tables, so it is intended to be reviewed as a Fabric project—not run unchanged in a local Python environment.

## Business questions addressed

| Area | Business question | Main analysis |
|---|---|---|
| Seasonal demand | Which products sell in Summer, Monsoon and Winter? | Season classification, product-level quantity and sales aggregation, top products by season |
| Product performance | Which products lead or lag in sales? | Top products by quantity and revenue, product ranking |
| ABC analysis | Which products contribute most to revenue? | Cumulative revenue contribution and A/B/C classification |
| Retailer analysis | Which retailers contribute the most sales? | Retailer-level sales, quantity and ranking |
| Inventory optimization | How much stock cover does each product have? | Average daily sales, stock days and inventory status |
| Stock recommendations | Which products may need action? | Reorder, maintain stock, reduce purchase/run promotion or liquidate recommendations |
| Discount analysis | How much discount is being given? | Discount value and percentage, discounted vs. non-discounted sales, product-level discount analysis |
| Salesperson performance | Which salespeople and routes generate sales? | Bills, cases, pieces, net sales, ranking and monthly/route summaries |
| Returns analysis | Which products and periods have the most returns? | Return quantity/value by product, month and retailer |

## Technology stack

- **Microsoft Fabric** – notebook and Lakehouse environment
- **OneLake / Lakehouse** – storage and managed analytical tables
- **Python** – notebook orchestration and data preparation
- **Pandas** – Excel ingestion and column preparation
- **PySpark** – conversion of Pandas DataFrames into Spark DataFrames
- **Spark SQL** – transformations, aggregations, CTEs, window functions, ranking and business rules
- **Jupyter Notebook (`.ipynb`)** – project implementation and execution record

## Data architecture

```text
Operational Excel reports
          |
          v
Pandas (read Excel / inspect headers / clean column names)
          |
          v
PySpark DataFrames
          |
          v
Microsoft Fabric Lakehouse / OneLake
          |
          v
Lakehouse tables
          |
          v
Spark SQL transformations and aggregations
          |
          v
Business analysis outputs
(product, retailer, inventory, discount, salesperson and returns)
```

## Lakehouse tables used

The project reference records the following Lakehouse tables:

- `transaction_report`
- `current_stock_report`
- `bill_wise_sales_report`
- `sales_return_report`
- `product_wise_sales_report`
- `billwise_productwise_sales_report`
- `product_purchase_report`
- `sales_analysis_report`
- `sales_value_bill_wise_report`

The main transaction-level analysis uses `transaction_report`. Inventory analysis combines transaction history with `current_stock_report`. Salesperson analysis uses `bill_wise_sales_report`, while returns analysis uses `sales_return_report`.

## Selected analytical logic

### 1. Seasonal product demand

The notebook creates a season from the transaction month:

- March–June: Summer
- July–September: Monsoon
- October–February: Winter

It then aggregates quantity and sales by season and product and ranks products within each season.

These month ranges are **project assumptions**, not confirmed company policy.

### 2. Product ranking and ABC classification

Products are aggregated by sales value and ranked. ABC classification is based on cumulative revenue contribution:

- A: up to 80%
- B: next 15% (up to 95% cumulative)
- C: remaining contribution

The cut-offs are analytical assumptions and should be reviewed with business stakeholders before operational use.

### 3. Inventory health and stock cover

Average daily sales are calculated from total quantity divided by the number of distinct selling dates. Stock cover is then estimated as:

```text
Stock days = Current saleable stock / Average daily sales
```

The notebook applies these proposed status bands:

- Below 15 stock days: Reorder
- 15–45 stock days: Healthy
- Above 45 stock days: Overstock
- No recorded sales velocity: Liquidate / investigate

A separate stock-out risk concept uses fewer than 7 stock days. These are **illustrative project thresholds**, not official company rules. The calculation also uses the available transaction history and should be checked for date-window effects, seasonality and zero-sales products.

### 4. Discount analysis

The notebook calculates discount value and discount percentage from transaction-level sales and discount fields, and compares discounted with non-discounted transactions.

This is **discount performance analysis, not true promotion ROI**. True ROI or profitability requires reliable product cost and a suitable incremental-sales or control-group methodology.

## Validation and modelling considerations

- The notebook inspects table columns and schemas before building queries.
- It uses confirmed Lakehouse table names rather than assumed names.
- It avoids an unnecessary join for salesperson analysis because the bill-wise table already contains salesperson, route and bill-level sales fields.
- Product-to-stock matching uses `current_stock_report.prddcode` and `transaction_report.item_code`, as recorded in the project reference.
- Joins should be checked for key uniqueness and row multiplication before using the results in a production dashboard.
- The retailer sales and returns keys still require validation before combining them into a retailer risk score.

## Current implementation status

### Implemented in the uploaded notebook

- Excel ingestion and column-name cleaning for source reports
- Creation/overwrite of Lakehouse tables from Spark DataFrames
- Table and schema inspection
- Seasonal demand aggregation and top-product ranking by season
- Product quantity/revenue ranking and ABC classification
- Retailer sales aggregation and ranking
- Inventory average-daily-sales and stock-days calculations
- Inventory status and recommended-action classification
- Discount and discount-percentage analysis
- Salesperson, monthly and route-level performance summaries
- Product return summaries by product, month and retailer
- Monthly product-demand aggregation as preparation for forecasting

### Deferred for a future project phase

- Training and evaluating a demand forecasting model (monthly demand aggregation is prepared, but no model has been trained)
- Retailer risk score (retailer key matching and score design remain to be validated)
- Market basket analysis
- Power BI dashboards and a published semantic model
- Reorder recommendation model beyond the current rule-based stock action
- AI-generated insights

Do not describe the planned items as completed deliverables unless they are implemented in a later version.

## Repository structure

A suggested public repository structure:

```text
fmcg-sales-inventory-fabric/
│
├── README.md
├── notebooks/
│   └── nivea_fabric_analysis.ipynb
├── screenshots/
│   ├── lakehouse_tables.png
│   ├── seasonal_demand.png
│   ├── product_abc_analysis.png
│   ├── inventory_recommendations.png
│   ├── discount_analysis.png
│   ├── salesperson_performance.png
│   └── returns_analysis.png
└── docs/
    └── project-notes.md
```

Only add screenshots that have been approved for public sharing and have been redacted or recreated with synthetic data. Do not commit source Excel files, exported company data, credentials, connection strings, or confidential notebook outputs.

## Running the notebook

1. Open a Microsoft Fabric workspace where you have permission to create and use a Lakehouse.
2. Upload the approved source Excel files to the expected Lakehouse `Files` location, or update the notebook paths to match your environment.
3. Attach the notebook to the correct Lakehouse.
4. Review the Excel header-row settings and column-cleaning logic for each source file.
5. Run the ingestion cells to create the Lakehouse tables.
6. Confirm table names and schemas.
7. Run the Spark SQL analysis cells in order.
8. Validate row counts, join keys, date coverage and business assumptions before using the outputs for decisions.

The cleaned notebook uses Fabric-specific paths such as `/lakehouse/default/Files/Nivea/...`; those paths will not work outside the corresponding Fabric Lakehouse setup. Attach the `Nivea_LH` Lakehouse as the notebook's default Lakehouse and confirm the source filenames/header-row settings before running it. The notebook's saved cell outputs have been cleared.

## Data privacy and usage notice

This repository should contain **code and documentation only** unless the data owner has explicitly approved wider disclosure.

Before making the repository public:

- Obtain written permission to publish the project name, business context, notebook code and screenshots.
- Remove or replace company names, distributor/retailer names, salesperson names, bill numbers, product/customer identifiers, tax details and any other confidential fields.
- Do not upload the original Excel files or raw/processed company datasets.
- Remove credentials, tokens, workspace identifiers, private URLs and connection details.
- Review notebook cell outputs and metadata; clearing outputs is not enough if sensitive values remain in code or metadata.
- Prefer synthetic or anonymized sample data and recreated screenshots for a public portfolio.
- If permission is not available, keep the repository private or describe the work generically as an FMCG distributor analytics project.

## Limitations

- The analysis is based on the columns available in the supplied reports.
- True retailer profitability and promotion ROI cannot be established without validated cost and incremental-sales data.
- Season definitions and inventory thresholds are assumptions requiring business validation.
- Retailer sales/returns key equivalence has not yet been confirmed.
- Forecasting data preparation exists, but a trained and evaluated forecasting model is not present in the uploaded notebook.
- A Power BI dashboard is part of the broader project plan, but no completed Power BI report was present in the uploaded notebook/reference materials.

## Interview-ready project summary

“I worked on an FMCG sales and inventory analytics project using Microsoft Fabric. I ingested Excel-based distributor reports into a Fabric Lakehouse using Pandas and PySpark, cleaned the column names, and created Lakehouse tables. I then used Spark SQL—including aggregations, CTEs, CASE expressions and window functions—to analyse seasonal product demand, product rankings, ABC revenue contribution, retailer sales, inventory cover, discount patterns, salesperson performance and product returns. The objective was to convert operational reports into decision-support outputs such as product priorities and stock action recommendations. I also documented data-quality considerations, join validation and business-rule assumptions. Forecasting and dashboarding were identified as next steps rather than completed components in this notebook.”

---

**Author:** Add your name here  
**Project type:** FMCG analytics / Microsoft Fabric / Data Engineering & Business Analytics  
**Status:** Exploratory implementation; see “Current implementation status” above.
