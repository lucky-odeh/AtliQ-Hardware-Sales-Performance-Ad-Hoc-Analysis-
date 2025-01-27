# AtliQ-Hardware-Sales-Performance-Ad-Hoc-Analysis-
AtliQ hardware is a rapidly growing company that manufactures PC, Accessories and peripherals, and in recent years, they have decided to implement data analytics into their business process for the first time to enable them track trends, stay competitive and to make data driven decision.
## Objectives
The objective of this project is to leverage SQL for ad-hoc analysis to evaluate sales performance, assess profitability, understand customer behavior, and efficiently address dynamic data requests from stakeholders of AtliQ. The goal is to provide actionable insights and support decision-making through robust and flexible data querying techniques.
## Key Question
1. What is the total revenue broken down by customer, platform, and sales channel?
2. Which region contributed the highest total revenue?
3. What is the total revenue segmented by product category, division, and market segment?
4. Which top five markets have generated the highest lifetime revenue?
5. What is the impact of freight costs on revenue across different markets?
6. Who are our highest-value customers based on total revenue generated?
7. What is the average selling price (ASP) across all markets?

## Data Collection
The dataset for this analysis comprises over **500,000** records, covering the period from **2017 to 2021**. It includes the following key tables stored in a MySQL database:
- **dim_customers:** Contains customer-related attributes.
- **dim_products:** Includes details about product categories, divisions, and segments.
- **fact_sales_monthly:** Provides monthly sales data, including revenue and units sold.
- **fact_gross_price:** Captures gross product pricing details.
- **fact_freight_cost:** Records freight costs associated with sales transactions.
- **fact_post_invoice_deductions:** Tracks post-invoice deductions, such as discounts or rebates.
- **fact_manufacturing_cost:** Contains manufacturing cost data for products.
 SQL queries will be used to extract and analyze this data to address the key business questions effectively.

## Data Preparation
We’ll create a sales performance model by combining these tables in MySQL database:
- **fact_sales_monthly:** Sales quantity data.
- **fact_gross_price:** Gross price per product.
- **fact_manufacturing_cost:** Manufacturing costs.
- **fact_freight_cost:** Freight costs by market and year.
- **fact_post_invoice_deductions:** Discounts impacting revenue.
- **dim_customer** 
- **dim_product**

We will write **SQL queries** to clean and join this data into a cohesive dataset for analysis.

## Model Building
We’ll write **SQL queries** step by step to address the key questions. Here’s how:

**Query 1:** **Total Revenue by customer, platform, and sales channel**

```sql
SELECT 
    c.customer,
    c.platform,
    c.channel,
    SUM(ROUND(sm.sold_quantity * gp.gross_price,2)) AS total_revenue
FROM fact_sales_monthly sm
JOIN dim_customer c ON sm.customer_code = c.customer_code
JOIN fact_gross_price gp ON sm.product_code = gp.product_code
GROUP BY c.customer, c.platform, c.channel
ORDER BY total_revenue DESC;
```
MySQL Result                          |                Visualization
:------------------------------------:|:-----------------------------------:
![](customer_platform.PNG)            |      ![](new_customer_platform.PNG)


**Query 2:** **Total Revenue by Region**

```sql
SELECT 
    c.region,
    SUM(ROUND(sm.sold_quantity * gp.gross_price,2)) AS total_revenue
FROM fact_sales_monthly sm
JOIN dim_customer c ON sm.customer_code = c.customer_code
JOIN fact_gross_price gp ON sm.product_code = gp.product_code
GROUP BY c.region
ORDER BY total_revenue DESC;

```
MySQL Result                          |                Visualization
:------------------------------------:|:-----------------------------------:
![](region.PNG)            |      ![](new_region.PNG)

**Query 3:** **Total Revenue by Division, Segment and Product Category**

```sql
SELECT 
    p.division,
    p.segment,
    p.category,
    SUM(ROUND(sm.sold_quantity * gp.gross_price,2)) AS total_revenue
FROM fact_sales_monthly sm
JOIN dim_product p ON sm.product_code = p.product_code
JOIN fact_gross_price gp ON sm.product_code = gp.product_code
GROUP BY p.division, p.segment, p.category
ORDER BY total_revenue DESC;

```
MySQL Result                          |                Visualization
:------------------------------------:|:-----------------------------------:
![](division_segment.PNG)            |      ![](new_division_segment.PNG)

**Query 4:** **Top 5 Markets with Highest Revenue**

```sql
SELECT 
	market,
    total_revenue,
    revenue_rank
FROM (
    SELECT 
		c.market,
        SUM(ROUND(sm.sold_quantity * gp.gross_price,2)) AS total_revenue,
        RANK() OVER (ORDER BY SUM(sm.sold_quantity * gp.gross_price) DESC) AS revenue_rank
    FROM fact_sales_monthly sm
    JOIN dim_customer c ON sm.customer_code = c.customer_code
    JOIN fact_gross_price gp ON sm.product_code = gp.product_code
    GROUP BY c.market
) ranked_markets
WHERE revenue_rank <= 5
ORDER BY revenue_rank;

```
MySQL Result                          |                Visualization
:------------------------------------:|:-----------------------------------:
![](top_five_market.PNG)            |      ![](new_top_five_market.PNG)




