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
6. Who are our highest-value customers based on frequency of orders?
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

**Query 5:** **Freight Cost Impact on Revenue across Markets**

```sql
WITH MarketRevenue AS (
    SELECT 
        c.market,
        SUM(ROUND(sm.sold_quantity * gp.gross_price,2)) AS total_revenue
    FROM fact_sales_monthly sm
    JOIN dim_customer c ON sm.customer_code = c.customer_code
    JOIN fact_gross_price gp ON sm.product_code = gp.product_code
    GROUP BY c.market
),
FreightCost AS (
    SELECT 
        fc.market,
        AVG(ROUND(fc.freight_pct,2)) AS avg_freight_cost
    FROM fact_freight_cost fc
    GROUP BY fc.market
)
SELECT 
    mr.market,
    mr.total_revenue,
    fc.avg_freight_cost,
    ROUND(fc.avg_freight_cost * mr.total_revenue,2) AS freight_cost_impact
FROM MarketRevenue mr
JOIN FreightCost fc ON mr.market = fc.market
ORDER BY freight_cost_impact DESC;

```
MySQL Result                          |                Visualization
:------------------------------------:|:-----------------------------------:
![](freight_cost.PNG)            |      ![](new_freight_cost.PNG)

**Query 6:** **High Value Customers**

```sql
SELECT 
    c.customer,
    COUNT(DISTINCT YEAR(sm.date)) AS purchase_years,
    COUNT(sm.date) AS total_purchases,
    AVG(sm.sold_quantity) AS avg_quantity_per_purchase
FROM fact_sales_monthly sm
JOIN dim_customer c ON sm.customer_code = c.customer_code
GROUP BY c.customer
ORDER BY total_purchases DESC;

```

MySQL Result                          |                Visualization
:------------------------------------:|:-----------------------------------:
![](high_value_customers.PNG)            |      ![](new_high_value_customers.PNG)

**Query 7:** **Average Selling Price Across Markets**


```sql
SELECT 
    c.market,
    SUM(sm.sold_quantity * gp.gross_price) / SUM(sm.sold_quantity) AS avg_selling_price
FROM fact_sales_monthly sm
JOIN dim_customer c ON sm.customer_code = c.customer_code
JOIN fact_gross_price gp ON sm.product_code = gp.product_code
GROUP BY c.market
ORDER BY avg_selling_price DESC;

```

MySQL Result                          |                Visualization
:------------------------------------:|:-----------------------------------:
![](average_selling_price.PNG)            |      ![](new_average_selling_price.PNG)

## Ad-Hoc Task 1 
- Create a Stored procedure that returns the monthly sales figure across customers and market.

**Query 8**

```sql
DELIMITER $$

CREATE PROCEDURE GetMonthlySalesByCustomerRegion(
    IN start_date DATE,
    IN end_date DATE,
    IN market_filter VARCHAR(255)
)
BEGIN
    SELECT 
        DATE_FORMAT(sm.date, '%Y-%m') AS sales_month,
        c.customer,
        c.market,
        SUM(sm.sold_quantity * gp.gross_price) AS total_monthly_sales
    FROM fact_sales_monthly sm
    JOIN dim_customer c ON sm.customer_code = c.customer_code
    JOIN fact_gross_price gp ON sm.product_code = gp.product_code
    WHERE sm.date BETWEEN start_date AND end_date
    AND (market_filter IS NULL OR c.market = market_filter) 
    GROUP BY 
        DATE_FORMAT(sm.date, '%Y-%m'),
        c.customer,
        c.market
    ORDER BY 
        sales_month ASC,
        c.market ASC,
        total_monthly_sales DESC;
END$$

DELIMITER ;

```
## Ad-Hoc Task 2
- Generate the monthly sales report for United Kingdom for the period between 01/01/2020 to 31/12/2020
  
**MySQL Result**
![](uk_stored_procedure.PNG)






