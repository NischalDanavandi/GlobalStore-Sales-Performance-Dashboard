# GlobalStore Sales Performance Dashboard

## Table of Contents
- [Overview](#overview)
- [Problem Statement](#problem-statement)
- [Dataset Summary](#dataset-summary)
- [Tools Used](#tools-used)
- [Data Modeling & DAX](#data-modeling--dax)
- [Business Questions Addressed](#business-questions-addressed)
- [Key Findings](#key-findings)
- [Insights & Strategic Recommendations](#insights--strategic-recommendations)
- [Dashboard Preview](#dashboard-preview)
- [How to Run](#how-to-run)
- [Author](#author)

---

## Overview
Sales performance analysis for GlobalStore, a global electronics retailer, built in Power BI on a five-table relational dataset (Sales, Customers, Products, Stores, Exchange Rates). The dashboard tracks revenue, profitability, and growth trends across markets and product categories, with drill-through detail by country.

---

## Problem Statement
GlobalStore needed a single view to answer: is the business growing, which markets and channels drive revenue, and which product categories are actually profitable — as opposed to simply high-volume. This project builds that view from raw transactional data, including identifying and correcting data quality issues that would otherwise have produced misleading conclusions.

---

## Dataset Summary
Source: Maven Analytics — Global Electronics Retailer dataset

| Table | Rows | Description |
|---|---|---|
| Sales | 62,884 | Order-line level transactions |
| Products | 2,517 | Product catalog with cost/price |
| Customers | 15,266 | Customer demographics |
| Stores | 67 | Physical store locations + one Online placeholder |
| Exchange_Rates | 11215 | Currency conversion rates by date |

---

## Tools Used
Power BI Desktop, Power Query (M), DAX, Data Modelling

---

## Data Modeling & DAX

### Data Model
Star schema: `Sales` (fact) connected to `Customers`, `Products`, `Stores`, and a custom `Dim_Date` (dimensions).

- **Dim_Date**: DAX calculated table, marked as the date table, connected to `Sales[Order Date]`. Required for `SAMEPERIODLASTYEAR` and other time-intelligence functions.
- **Exchange_Rates**: modeled with a composite key (`Currency & Date`, concatenated in Power Query), since its natural key spans two fields and Power BI relationships support only a single join column. Not connected in the final model — all revenue/cost measures use the pre-converted USD columns in `Products`, so currency conversion was not required for this analysis.
- All relationships: one-to-many, single-direction cross-filter.

### DAX Measures

```dax
Dim_Date =
VAR MinDate = MIN(Sales[Order Date])
VAR MaxDate = MAX(Sales[Order Date])
RETURN
ADDCOLUMNS(
    CALENDAR(MinDate, MaxDate),
    "Year", YEAR([Date]),
    "Month Name", FORMAT([Date], "Mmm"),
    "Month Number", MONTH([Date]),
    "Quarter", "Q" & FORMAT([Date], "Q")
)

Total Revenue =
SUMX(Sales, Sales[Quantity] * RELATED(Products[Unit Price USD]))

Total Cost =
SUMX(Sales, Sales[Quantity] * RELATED(Products[Unit Cost USD]))

Total Profit = [Total Revenue] - [Total Cost]

Profit Margin % = DIVIDE([Total Profit], [Total Revenue], 0)

Prior Year Revenue =
CALCULATE([Total Revenue], SAMEPERIODLASTYEAR(Dim_Date[Date]))

YOY Growth % =
DIVIDE([Total Revenue] - [Prior Year Revenue], [Prior Year Revenue], 0)

Online Revenue =
CALCULATE([Total Revenue], Stores[Country] = "Online")

Physical Revenue =
CALCULATE([Total Revenue], Stores[Country] <> "Online")
```

---

## Business Questions Addressed
1. How has revenue trended by month across each year, and is the business growing year-over-year?
2. Which country/channel generates the most revenue?
3. Which product category and subcategory drive the highest revenue and profit?
4. Does the highest profit margin % correspond to the highest absolute profit contribution?
5. For the top-performing country, which subcategories specifically drive that revenue? *(drill-through)*

---

## Key Findings

**Revenue grew 20.46% YoY (2016–2020).**
2021 was excluded from growth calculations — the dataset contains only 2 months of 2021 data, which otherwise produces a misleading negative growth figure.

**Seasonal pattern:** Revenue peaks in February and drops sharply in April, consistent across multiple years.

**2020 dip:** Revenue declined 11.25% YoY in 2020 vs. 2019, consistent with pandemic-related disruption to retail activity. This is an isolated exception within an otherwise positive 5-year trend.

**Online channel:** "Online" is recorded as a placeholder store (not a country) within the Stores table, representing ~21% of Sales transactions (13,165 of 62,884 rows).
- Online revenue: $11.12M (~20% of total)
- Physical stores combined: $43.60M (~80% of total)
- Top physical market: United States, ~$23M (~42% of total), followed by the United Kingdom

**Margin vs. profit contribution:** Music, Movies and Audio Books has the highest profit margin (~61%), driven by low per-unit cost. Desktops contributes the highest absolute profit ($5.50M), driven by volume and price point. Margin % and absolute profit rank differently — a decision based on either metric alone would be incomplete.

---

## Insights & Strategic Recommendations
- **Channel investment:** Online already contributes ~20% of revenue from a single, unmanaged placeholder listing — formal tracking and investment in this channel is likely underexploited.
- **Category strategy is two-dimensional:** Desktops should remain a volume/scale priority, while Music/Movies/Audio Books-type categories (high margin, low cost) are strong candidates for margin-focused promotion rather than volume growth.
- **2020 dip does not indicate a structural problem:** the decline is explained by an external, one-off event rather than a demand or pricing issue, so no corrective business action is implied by that year alone.
- **US market concentration:** with ~42% of revenue from a single country, geographic diversification may reduce reliance on one market.

---

## Dashboard Preview
<img width="1917" height="1016" alt="Screenshot 2026-09-02 192452" src="https://github.com/user-attachments/assets/1a55430f-9874-4877-9310-6e585a323efd" />
<img width="1917" height="1017" alt="Screenshot 2026-09-02 192509" src="https://github.com/user-attachments/assets/618f2fcc-9b1e-4085-b8ce-0b7db76b71fb" />
<img width="1917" height="972" alt="Screenshot 2026-09-02 192554" src="https://github.com/user-attachments/assets/96363a10-f143-4a5d-b525-36bc8dbbd769" />


---

## How to Run
1. Download `GlobalStore.pbix` from this repository
2. Open in Power BI Desktop
3. If prompted, update the data source file paths to point to your local copy of the source CSVs (Maven Analytics — Global Electronics Retailer)
4. Refresh the data model


---

## Author
Nischal Danavandi
