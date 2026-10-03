# BrewMetrics BI

## 1. Project Overview

BrewMetrics Coffee Co. is the business case for this college Business Intelligence lab mini-project. The repository contains a version-controlled Power BI Project (`.pbip`) with a semantic model and report. It is designed to summarize sales and support time-based analysis using sales data, product and city dimensions, and DAX measures.

The current report is a compact prototype: it displays total sales and a monthly sales table with month-over-month growth. Although the model includes product, city, store-format, and date fields, the report does not currently visualize all of those analysis areas.

## 2. Business Problem

Management wants to understand sales performance across cities, store formats, products, and time. The business questions include whether Cold Brew sales show a seasonal pattern, how sales differ by city, and which products contribute the most sales.

The model provides sales and date/product/city attributes for analysis. The report currently answers only a subset of the requested questions directly: it shows total sales and monthly sales with month-over-month growth. It has no city, store-format, or item visual, and has no slicer in the saved report definition.

## 3. Dataset

The Power Query source is a CSV named `brewmetrics_sales.csv`. The file is referenced through a machine-specific absolute path in the `Fact_Sales` query and is **not included in this repository**. The path must be updated for another computer or checkout before refreshing the model.

The locally available source CSV contains **15,482 records**, with **15,482 distinct `sale_id` values**. Its dates cover **April 1 through July 1, 2026**: April, May, and June are complete calendar months, and July contains only July 1 records. The model's Power Query removes duplicate full rows before loading. The currency for `sales_amount` is not specified in the model or CSV metadata.

| Column | Meaning |
|---|---|
| `sale_id` | Identifier for a sale record; each ID is unique in the available CSV. |
| `date` | Date of the sale. |
| `city` | City associated with the sale. |
| `store_format` | Store format recorded for the sale. |
| `category` | Product category. |
| `item` | Product/item sold. |
| `quantity` | Quantity sold. |
| `unit_price` | Unit price recorded for the item. |
| `sales_amount` | Sales amount used by the `Total Sales` measure. |

## 4. Data Model

The semantic model contains a sales fact table and three business dimensions. Sales rows are connected to product, date, and city attributes through relationships defined in `relationships.tmdl`.

| Table | Purpose and important columns | Relationships |
|---|---|---|
| `Fact_Sales` | Sales fact table. Its source columns are `sale_id`, `date`, `city`, `store_format`, `category`, `item`, `quantity`, `unit_price`, and `sales_amount`. It also hosts all four DAX measures. | `date` to `Dim_Date[date]`; `item` to `Dim_Product[item]`; `city` to `Dim_City[city]`. `store_format` is present in the fact table but has no separate dimension relationship. |
| `Dim_Product` | Product lookup with `item` and `category`. Power Query selects these columns from sales and removes duplicate pairs. | `Dim_Product[item]` is related to `Fact_Sales[item]`. |
| `Dim_Date` | Date attributes derived from distinct dates in the sales data: `date`, `Year`, `Month Number`, `Month`, `Month-Year`, and `Day`. `Month` is sorted by `Month Number`. | `Dim_Date[date]` is related to `Fact_Sales[date]`. The model also contains Power BI's generated local date table connected to `Dim_Date[date]`. |
| `Dim_City` | City lookup with `city` and `store_format`, constructed from those two fact columns. | `Dim_City[city]` is related to `Fact_Sales[city]`. |

The relationships are intended to support the usual many-sales-rows-to-one-dimension-member analysis. The TMDL relationship declarations identify the linked columns but do not explicitly serialize cardinality or cross-filter direction; **[NEEDS VERIFICATION]**: confirm those settings in Power BI Desktop.

There is a modeling limitation for store-format analysis: `Dim_City` removes duplicates on `city` alone while retaining `store_format`. The available source data has all three store formats (Drive-Thru, Flagship, and Kiosk) in each city, so the dimension cannot preserve every city/format combination. Do not use `Dim_City[store_format]` as a complete store-format breakdown. The fact table's `store_format` column is available, but there is no dedicated store-format dimension.

## 5. DAX Measures

The following are the **four measures currently defined** in `Fact_Sales.tmdl`. `Total Sales` is the additional/custom base measure; the other three provide growth, running-total, and ranking calculations.

| Measure | Calculation | Business use |
|---|---|---|
| `Total Sales` | `SUM(Fact_Sales[sales_amount])` | Base sales total for cards and other sales analyses. This is the custom/base measure used by the other measures. |
| `Sales MoM Growth %` | Compares `[Total Sales]` with `[Total Sales]` for the prior month using `DATEADD(Dim_Date[date], -1, MONTH)`, then divides the difference by prior-month sales using `DIVIDE`. | Shows the percentage change from the previous month. |
| `Cumulative Total Sales` | Uses `CALCULATE` with `FILTER(ALLSELECTED(Dim_Date), Dim_Date[date] <= MAX(Dim_Date[date]))` around `[Total Sales]`. | Accumulates sales up to the current date while respecting the selected date range. |
| `Product Sales Rank` | Uses `RANKX(ALL(Dim_Product[item]), [Total Sales], , DESC, DENSE)`. | Ranks items by sales from highest to lowest; equal sales receive the same dense rank. |

Only `Total Sales` and `Sales MoM Growth %` are currently used in the saved report visuals. The running total and item rank are model measures but are not displayed in the current report.

## 6. Dashboard

The report has one page, named **Page 1**, and two saved visuals:

| Visual | Fields/metric | Business question |
|---|---|---|
| Card | `Fact_Sales[Total Sales]` | What is total sales for the current report filter context? |
| Table | `Dim_Date[Month]`, `[Total Sales]`, and `[Sales MoM Growth %]` | How do sales and month-over-month growth compare by month? |

No city, store-format, or product visual is currently present. **Slicer:** no slicer visual is defined in the saved report files. **Drill-down hierarchy:** no drill hierarchy is used by the visuals; the table groups by the `Dim_Date[Month]` column. The date column has a generated Power BI date hierarchy in the model, but the report table does not use it.

## 7. Key Business Insights

The observations below are computed from the locally available CSV currently referenced by the model. They are not displayed in the existing report visuals, and the CSV is not committed to this repository.

1. **Cold Brew leads item sales and peaks in May in this data window.** Its sales total is 276,593.46 in April, 301,280.66 in May, and 185,282.81 in June. July 1 contributes 8,104.77 for that single day. This is a short, single-year window and does not establish a repeating seasonal trend.
2. **Bengaluru has the highest city sales total in the available data.** City totals are Bengaluru 1,119,895.31, Chennai 1,054,801.19, Hyderabad 968,966.26, and Coimbatore 822,665.12.
3. **Cold Brew is the top-selling item by sales amount.** It totals 771,261.70, ahead of Tumbler at 628,389.11. The current report does not include a product visual, so this finding comes from aggregating the source data rather than reading a displayed report visual.

These amounts have no currency symbol because the data source does not specify a currency. **[NEEDS VERIFICATION]**: reproduce these totals in a fresh checkout after making the external CSV available and refreshing the model.

## 8. Version Control Workflow

The repository's Git history records incremental development:

1. **Initial repository documentation:** `ea89753` added the initial `Readme.MD`.
2. **Power BI project and star schema:** `2d4aea2` added the PBIP project structure, report/model definitions, and fact/dimension tables.
3. **DAX measures and report visuals:** `14e5da8` added the month-over-month measure and the two report visual definitions. `853d100` and `5f73451` separately added the running-total and product-ranking measures.
4. **Dashboard:** the card and monthly table visual files are versioned; they were added in the month-over-month measure commit rather than in a separate dashboard-only commit.
5. **Documentation:** `Notes.md` records measure task prompts/formulas, and this README documents the project and its verified state.

The latest commit is titled “Add average sale per transaction measure,” but its change is to `Notes.md`; no average-sale-per-transaction measure is defined in the current semantic model. Commit history and diffs make changes reviewable and provide an audit trail of how the model, measures, visuals, and documentation evolved.

## 9. GitHub Copilot Usage

GitHub Copilot was used as development assistance for drafting the DAX measures and supporting project documentation. The measures described here were checked against the current semantic model. `Notes.md` records measure requests and formula notes; it does not document specific Copilot corrections, so no particular correction is claimed.

## 10. Tools and Technologies

- **Power BI Project (`.pbip`):** project/report artifact format present in the repository.
- **Power Query (M):** used in the semantic model's table partitions to load and shape CSV data.
- **DAX:** used for the four measures in `Fact_Sales`.
- **Git and GitHub:** repository history is present and the configured `origin` is a GitHub remote.
- **Power BI Desktop, Visual Studio Code, and GitHub Copilot:** used in the project workflow as described for this lab.

## 11. Project Structure

```text
brewmetrics-bi/
├── .gitignore
├── BrewMetrics.pbip
├── README.md
├── Notes.md
├── BrewMetrics.Report/
│   ├── definition.pbir
│   ├── definition/
│   │   ├── report.json
│   │   └── pages/
│   │       ├── pages.json
│   │       └── da5d94b0d064e674f16d/
│   │           ├── page.json
│   │           └── visuals/
│   │               ├── 2a191af2447b30e4d4f2/visual.json
│   │               └── 2a3f45c0848e745478ee/visual.json
│   └── StaticResources/SharedResources/BaseThemes/
│       └── Fluent2-CY26SU09.json
└── BrewMetrics.SemanticModel/
    ├── definition.pbism
    ├── diagramLayout.json
    └── definition/
        ├── model.tmdl
        ├── relationships.tmdl
        └── tables/
            ├── Fact_Sales.tmdl
            ├── Dim_Product.tmdl
            ├── Dim_City.tmdl
            ├── Dim_Date.tmdl
            ├── DateTableTemplate_9c0123b0-d601-4708-bb30-325deb38a4f0.tmdl
            └── LocalDateTable_dcc2085b-d622-48f1-aacf-332206ccce4d.tmdl
```

The source CSV is external to this tree and is not version-controlled. Local Power BI cache and settings files are excluded by `.gitignore`.

## 12. Conclusion

The current BrewMetrics BI solution provides a version-controlled Power BI semantic model with sales, date, product, and city fields, four sales measures, and a report page showing total sales and monthly sales growth. The model and report form a foundation for management analysis, while city/product/store-format visuals and a complete store-format dimension remain outside the current implementation.

## Verification Checklist

- [x] PBIP report and semantic-model artifacts exist in the repository.
- [x] `Fact_Sales`, `Dim_Product`, `Dim_City`, and `Dim_Date` are defined, with the relationships described above.
- [x] Four DAX measures are present: `Total Sales`, `Sales MoM Growth %`, `Cumulative Total Sales`, and `Product Sales Rank`.
- [x] The saved report has one page, a Total Sales card, and a monthly table containing Total Sales and MoM Growth.
- [x] The locally available CSV has 15,482 records with 15,482 distinct sale IDs and covers 2026-04-01 through 2026-07-01.
- [x] No slicer visual or report drill-down hierarchy is defined in the saved report visuals.
- [x] The current source CSV's Cold Brew, city, and item totals were aggregated locally; they are not displayed in the report and cannot be reproduced from a clean checkout unless the source file is supplied.
- [x] No average-sale-per-transaction measure is present in the current model, despite the latest commit title.
- [NEEDS VERIFICATION] Relationship cardinality and cross-filter direction in Power BI Desktop.
- [NEEDS VERIFICATION] The absolute CSV source path must be updated for another machine; the source is not tracked.
- [NEEDS VERIFICATION] Currency/unit for `sales_amount`.
- [NEEDS VERIFICATION] Accurate store-format analysis through a dedicated store-format dimension; the current `Dim_City[store_format]` is lossy.
