# Power BI Setup Guide: Supermart Sales Report

How to rebuild the Power BI report on a clean data model, using the files in this folder.

## Files

- `supermart_sales.pbix`: the current Power BI report.
- `DAX_Calendar.txt`: calendar table and relationship instructions.
- `DAX_Measures.txt`: 11 DAX measures (sales, profit, margin, orders, AOV, discount, customers, YoY).
- `Supermart_Theme.json`: report colours and typography.

## 1. Prepare the data

The raw CSV mixes two date formats (`DD-MM-YYYY` with hyphens and `MM/DD/YYYY` with slashes), so Power BI can misread dates on import. Create a clean copy with ISO dates first:

```bash
python src/sales_analysis.py --input "Supermart Grocery Sales - Retail Analytics Dataset.csv" --export-clean
```

This writes `sales_analysis_output/SupermartGrocerySales_PowerBI_Clean.csv`. It has all 9,994 original rows and columns, `Order Date` as `YYYY-MM-DD`, and the helper columns `Year`, `Month Number`, `Year Month` and `Year Month Sort`.

## 2. Build the model in Power BI Desktop

1. Open `supermart_sales.pbix` and save a working copy.
2. In **Home > Transform data**, point the `SupermartGrocerySales` query's Source step at the clean CSV. Don't load it as a second table, or you'll end up with duplicate fact tables.
3. Set data types: `Order Date` as Date, `Sales` and `Profit` as decimal numbers, and `Discount` as a decimal formatted as a percentage.
4. Create the calendar table from `DAX_Calendar.txt` (**Modeling > New table**). Add the relationship, mark it as a date table and set the sort-by columns as described in that file.
5. Add each measure from `DAX_Measures.txt` one at a time (**Modeling > New measure**). Format margin, discount and YoY measures as percentages. Use the average discount as a KPI, never the sum.
6. Import the theme: **View > Themes > Browse for themes > `Supermart_Theme.json`**.

## 3. Report pages

**Executive Overview**
- KPI cards: Total Sales, Total Profit, Profit Margin %, Total Orders, Average Order Value.
- Line chart of `Calendar[Year Month]` vs Total Sales, plus bar charts of Sales by Category and Profit by Region.
- Slicers: `Calendar[Year]` and Region.

**Product & Region Performance**
- Sales and profit by category, the top 10 sub-categories by sales, and a Region × Category matrix with Sales, Profit and Profit Margin %.
- Slicers: Year, Region and Category.

**Customer & Discount Analysis**
- Cards: Total Customers, Average Order Value, Average Discount %.
- Top 10 customers by sales, profit margin by discount band, and a Discount vs Profit scatter.
- The Discount–Profit correlation is about zero, so don't claim a causal effect from the scatter.

## 4. Validate

Check the totals against the Python output in `results/overview.csv`:

| Metric | Expected |
| --- | ---: |
| Total Sales | 14,956,982 |
| Total Profit | 3,747,121.20 |
| Total Orders | 9,994 |
| Profit Margin % | 25.05% |

Then test the slicers and cross-filtering, and add screenshots of each finished page to `assets/`.

## Data-model caveats

- There are 9,994 rows and 9,994 distinct Order IDs here, so rows equal orders. In general the two mean different things.
- There's no Quantity column, so never label row counts as items sold.
- The "North" region contains a single record. Exclude it from regional visuals or add a footnote.
- `Profit Margin %` is total profit ÷ total sales, not an average of row-level margins.
- `Sales YoY %` needs the continuous calendar table and returns blank for the first year.
- The dataset has region names but no coordinates, so avoid precise location maps.
