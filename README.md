# Retail Sales Performance & Customer Behaviour Dashboard (Power BI)

An interactive Power BI dashboard that helps retail management identify top-performing products, profitable regions, customer segments and month-wise sales trends, and compares actual sales against targets to highlight areas needing improvement.

> **Dataset:** Superstore sales data (9,994 order lines, orders from Jan 2014 to Dec 2017)
> **Tool:** Microsoft Power BI Desktop (Power Query, DAX)
> **Repository:** [Retail Sales Performance & Customer Behaviour Dashboard (Power BI)](https://github.com/Nebin-Biju/Retail-Sales-Performance-Customer-Behaviour-Dashboard-Power-BI-)

---

## Table of Contents
1. [Project Objective](#project-objective)
2. [Dataset](#dataset)
3. [Repository Structure](#repository-structure)
4. [Data Preparation](#data-preparation)
5. [Data Model](#data-model)
6. [DAX Measures](#dax-measures)
7. [Dashboard Pages](#dashboard-pages)
8. [Key Insights](#key-insights)
9. [Recommendations](#recommendations)
10. [Assumptions](#assumptions)
11. [How to Run](#how-to-run)
12. [Author](#author)

---

## Project Objective

Build a dashboard that answers:
- Which products and customers perform best?
- Which regions and customer segments are most profitable?
- How do sales trend month by month and year over year?
- Are we meeting sales targets, and where do we need to improve?

## Dataset

| Item | Detail |
|---|---|
| File | `Sample_Superstore.csv` |
| Rows / Columns | 9,994 / 21 |
| Period | 3 Jan 2014 to 30 Dec 2017 |
| Source | Superstore Dataset on Kaggle (vivek468) |

Key columns: Order ID, Order Date, Ship Date, Customer ID, Customer Name, Segment, Region, State, City, Category, Sub-Category, Product Name, Sales, Quantity, Discount, Profit.

The CSV uses Windows-1252 encoding and M/D/YYYY dates.

## Repository Structure

```
.
├── data/
│   └── Sample_Superstore.csv
├── dashboard/
│   └── Superstore_Sales_Dashboard.pbix
├── screenshots/
│   ├── 01_sales_overview.png
│   ├── 02_product_performance.png
│   ├── 03_customer_insights.png
│   └── 04_tooltip.png
└── README.md
```

## Data Preparation

1. Loaded the CSV in Power BI (File Origin: 1252 Western European).
2. Converted `Order Date` and `Ship Date` to Date type using the **English (United States)** locale.
3. Set Sales, Profit and Discount to Decimal and Quantity to Whole Number.
4. Created a **Calendar** table (2014 to 2017) with Year, Quarter, Month, Month No and Year-Month, and sorted Month by Month No.

## Data Model

- `Calendar[Date]` → `Superstore[Order Date]` (one-to-many, single direction)
- All time slicers and chart axes use Calendar fields so time-intelligence measures work correctly.

## DAX Measures

```DAX
Total Sales      = SUM(Superstore[Sales])
Total Profit     = SUM(Superstore[Profit])
Profit Margin %  = DIVIDE([Total Profit], [Total Sales])
Total Quantity   = SUM(Superstore[Quantity])
Total Orders     = DISTINCTCOUNT(Superstore[Order ID])

Sales LY         = CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Calendar'[Date]))
YoY Growth %     = DIVIDE([Total Sales] - [Sales LY], [Sales LY])

Sales Target         = [Sales LY] * 1.10
Target Achievement % = DIVIDE([Total Sales], [Sales Target])
Gap to Target        = [Total Sales] - [Sales Target]
Target Status        = IF([Target Achievement %] >= 1, "On Target", "Needs Improvement")

Product Rank       = RANKX(ALL(Superstore[Product Name]), [Total Sales])
Customer Rank      = RANKX(ALL(Superstore[Customer Name]), [Total Sales])
Contribution %     = DIVIDE([Total Sales], CALCULATE([Total Sales], ALLSELECTED()))
Last Purchase Date = MAX(Superstore[Order Date])

MoM Growth % =
VAR PrevMonth = CALCULATE([Total Sales], DATEADD('Calendar'[Date], -1, MONTH))
RETURN DIVIDE([Total Sales] - PrevMonth, PrevMonth)

Selected Product  = SELECTEDVALUE(Superstore[Product Name], "All Products")
Selected Customer = SELECTEDVALUE(Superstore[Customer Name], "All Customers")
Customer Segment  = SELECTEDVALUE(Superstore[Segment], "Multiple")
Customer Regions  = CONCATENATEX(VALUES(Superstore[Region]), Superstore[Region], ", ", Superstore[Region], ASC)
```

## Dashboard Pages

### 1. Sales Overview (main page)
- KPI cards: Total Sales, Total Profit, Profit Margin %, YoY Growth %, Target Achievement %
- YoY line chart (Month on the axis, Year as legend)
- Sales vs Target column chart with Achievement % line
- Top 5 Products and Top 5 Customers (Top N filter)
- Region-wise sales performance
- Slicers: **Region, Product Category, Customer Segment, Time (Year / Quarter / Month)**, synced across pages
- Conditional formatting: red for below target, green for on target

### 2. Product Performance (drill-through)
Right-click any product → Drill through. Shows product name, monthly sales trend, profit, quantity, region-wise sales and ranking.

### 3. Customer Insights (drill-through)
Right-click any customer → Drill through. Shows customer name, sales history, profit and margin, product-wise purchases, segment, region and rank.

### 4. Tooltip Page (quick insights)
Shown on hover: YoY sales comparison, product sales ranking, contribution % to total sales, last purchase date (customers) and an optional sales forecast.

## Screenshots

| Sales Overview | Product Performance |
|---|---|
| ![Overview](screenshots/01_sales_overview.png) | ![Product](screenshots/02_product_performance.png) |

| Customer Insights | Tooltip |
|---|---|
| ![Customer](screenshots/03_customer_insights.png) | ![Tooltip](screenshots/04_tooltip.png) |

## Key Insights

**Overall:** Sales **$2,297,201**, profit **$286,397**, profit margin **12.47%**, 5,009 orders, 793 customers.

| # | Question | Answer |
|---|---|---|
| 1 | Region with highest sales | **West**, $725,458 (31.6% of sales); also highest profit ($108,418) |
| 2 | Monthly trend, last two years | Sales grew from $609K (2016) to $733K (2017), **+20.4%**. Strong seasonal pattern: low Jan to Feb, peak Sep to Dec. Best month: Nov 2017 ($118,448) |
| 3 | Top 5 customers by revenue | Sean Miller ($25,043), Tamara Chand ($19,052), Raymond Buch ($15,117), Tom Ashbrook ($14,596), Adrian Barton ($14,474) |
| 4 | Category with highest margin | **Technology**, 17.4% (Office Supplies 17.0%, Furniture only 2.5%) |
| 5 | Target achieved this quarter (Q4 2017) | **107.8%** ($280,054 vs target $259,709) |
| 6 | Quantity vs profit correlation | Very weak (**r = 0.07**). Discount vs profit is negative (r = -0.22) |
| 7 | Month with significant drop | **January and February** (Jan 2017 -54.7%, Feb 2017 -53.8% vs previous month). Seasonal, repeats yearly |
| 8 | Most profitable segment | **Consumer** earns the most profit ($134,119); **Home Office** has the highest margin (14.0%) |

**Year-over-year sales:** 2015: -2.8%, 2016: +29.5%, 2017: +20.4%.

**Top 5 products by sales:** Canon imageCLASS 2200 Advanced Copier ($61,600), Fellowes PB500 Binding Machine ($27,453), Cisco TelePresence System EX90 ($22,638), HON 5400 Series Task Chairs ($21,871), GBC DocuBind TL300 ($19,823).

**Areas needing improvement (2017 vs target):** Central region 90.7%, Q2 89.4%, and the months of Feb, Apr, May and Dec.

## Recommendations

1. Cap discounts at around 20%. Discounts of 30% and above lose money in total.
2. Review pricing and costs for Furniture, especially Tables (-$17,725 profit) and Bookcases (-$3,473).
3. Investigate the Central region, which has the lowest margin (7.9%) and missed its 2017 target.
4. Run promotions in January and February to offset the seasonal low.
5. Invest in Home Office and Technology, which combine growth and healthy margins.
6. Re-engage lapsed high-value customers such as Tamara Chand (last order Nov 2016).

## Assumptions

- The dataset has no target column, so **Sales Target = same period last year + 10%**. Targets are therefore not available for 2014.
- "This quarter" refers to the latest quarter in the data (Q4 2017).
- Figures are in US dollars as given in the dataset.

## How to Run

1. Clone the repository:
   ```bash
   git clone https://github.com/Nebin-Biju/Retail-Sales-Performance-Customer-Behaviour-Dashboard-Power-BI-.git
   ```
2. Install **Power BI Desktop** (free, Windows).
3. Open `dashboard/Superstore_Sales_Dashboard.pbix`.
4. If prompted, update the data source path: Home → Transform data → Data source settings → point to `data/Sample_Superstore.csv`.
5. Click **Refresh**.

## Author

**Nebin Biju**  
[GitHub Profile](https://github.com/Nebin-Biju)  
[Project Repository](https://github.com/Nebin-Biju/Retail-Sales-Performance-Customer-Behaviour-Dashboard-Power-BI-)
