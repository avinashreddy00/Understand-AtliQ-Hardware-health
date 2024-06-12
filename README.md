# AtliQ Hardware Health Analysis (Data Analysis with SQL)

## Project Overview
AtliQ Hardware is experiencing rapid growth and aims to implement OLAP (Online Analytical Processing) for data analytics using SQL. The objective is to gain competitive advantage, make data-driven decisions, and address stakeholder inquiries across finance, sales, marketing, and supply chain domains.

## Company Background
AtliQ Hardware is a global company specializing in the sale of computers and accessories through various channels including retailers, direct sales, and distributors. Facing unexpected losses and competition with analytics-equipped competitors, AtliQ Hardware recognizes the need to establish an analytics team for better decision-making.

## Dataset Understanding
The dataset comprises dimension and fact tables:
- **Dimension Tables**:
  - `dim_customer`: Details of customers and products.
  - `dim_market`: Market details including regions and sub-zones.
  - `dim_product`: Product divisions, categories, and variants.
- **Fact Tables**:
  - `fact_forecast_monthly`: Forecasted customer needs.
  - `fact_sales_monthly`: Sales transactions.

## Finance Analytics
### User-Defined SQL Functions
#### Retrieve Customer Codes for Croma India
```sql
SELECT * FROM dim_customer WHERE customer LIKE "%croma%" AND market = "India";
```
### Get Sales Transactions for Croma India in FY 2021
```sql
SELECT * FROM fact_sales_monthly 
WHERE customer_code = 90002002 AND YEAR(DATE_ADD(date, INTERVAL 4 MONTH)) = 2021 
ORDER BY date ASC
LIMIT 100000;
```
### Create a Function to Get Fiscal Year
```sql
CREATE FUNCTION `get_fiscal_year`(calendar_date DATE) 
RETURNS INT
DETERMINISTIC
BEGIN
    DECLARE fiscal_year INT;
    SET fiscal_year = YEAR(DATE_ADD(calendar_date, INTERVAL 4 MONTH));
    RETURN fiscal_year;
END
```
### Replace Function in Sales Transaction Query
```sql
SELECT * FROM fact_sales_monthly 
WHERE customer_code = 90002002 AND get_fiscal_year(date) = 2021 
ORDER BY date ASC
LIMIT 100000;
```
## Gross Sales Reports
### Monthly Product Transactions
```sql
SELECT s.date, s.product_code, p.product, p.variant, s.sold_quantity 
FROM fact_sales_monthly s
JOIN dim_product p ON s.product_code = p.product_code
WHERE customer_code = 90002002 AND get_fiscal_year(date) = 2021     
LIMIT 1000000;
```
### Total Sales Amount
```sql
SELECT s.date, SUM(ROUND(s.sold_quantity * g.gross_price, 2)) AS monthly_sales
FROM fact_sales_monthly s
JOIN fact_gross_price g ON g.fiscal_year = s.fiscal_year AND g.product_code = s.product_code
WHERE customer_code = 90002002
GROUP BY date;
```
##Stored Procedures
###Monthly Gross Sales Report for Any Customer
```sql
CREATE PROCEDURE `get_monthly_gross_sales_for_customer`(
    in_customer_codes TEXT)
BEGIN
    SELECT s.date, 
           SUM(ROUND(s.sold_quantity * g.gross_price, 2)) AS monthly_sales
    FROM fact_sales_monthly s
    JOIN fact_gross_price g ON g.fiscal_year = s.fiscal_year AND g.product_code = s.product_code
    WHERE FIND_IN_SET(s.customer_code, in_customer_codes) > 0
    GROUP BY s.date
    ORDER BY s.date DESC;
END
```
### Market Badge
```sql
CREATE PROCEDURE `get_market_badge`(
    IN in_market VARCHAR(45),
    IN in_fiscal_year YEAR,
    OUT out_level VARCHAR(45)
)
BEGIN
    DECLARE qty INT DEFAULT 0;
    
    IF in_market = "" THEN
        SET in_market = "India";
    END IF;
    
    SELECT SUM(s.sold_quantity) INTO qty
    FROM fact_sales_monthly s
    JOIN dim_customer c ON s.customer_code = c.customer_code
    WHERE get_fiscal_year(s.date) = in_fiscal_year AND c.market = in_market;
    
    IF qty > 5000000 THEN
        SET out_level = 'Gold';
    ELSE
        SET out_level = 'Silver';
    END IF;
END
```
## Market Analytics
### Problem Statement and Pre-Invoice Discount Report
### Include Pre-Invoice Deductions in Croma Detailed Report
```sql
SELECT s.date, s.product_code, p.product, p.variant, s.sold_quantity, 
       g.gross_price AS gross_price_per_item,
       ROUND(s.sold_quantity * g.gross_price, 2) AS gross_price_total,
       pre.pre_invoice_discount_pct
FROM fact_sales_monthly s
JOIN dim_product p ON s.product_code = p.product_code
JOIN fact_gross_price g ON g.fiscal_year = s.fiscal_year AND g.product_code = s.product_code
JOIN fact_pre_invoice_deductions AS pre ON pre.customer_code = s.customer_code AND pre.fiscal_year = s.fiscal_year
WHERE s.customer_code = 90002002 AND get_fiscal_year(s.date) = 2021     
LIMIT 1000000;
```
### Performance Improvement
### Creating Dim_Date Table and Avoiding 'get_fiscal_year()' Function
```sql
SELECT s.date, s.customer_code, s.product_code, p.product, p.variant, s.sold_quantity, 
       g.gross_price AS gross_price_per_item,
       ROUND(s.sold_quantity * g.gross_price, 2) AS gross_price_total,
       pre.pre_invoice_discount_pct
FROM fact_sales_monthly s
JOIN dim_date dt ON dt.calendar_date = s.date
JOIN dim_product p ON s.product_code = p.product_code
JOIN fact_gross_price g ON g.fiscal_year = dt.fiscal_year AND g.product_code = s.product_code
JOIN fact_pre_invoice_deductions AS pre ON pre.customer_code = s.customer_code AND pre.fiscal_year = dt.fiscal_year
WHERE dt.fiscal_year = 2021     
LIMIT 1500000;
```
### Adding Fiscal Year in Fact_Sales_Monthly Table
```sql
SELECT s.date, s.customer_code, s.product_code, p.product, p.variant, s.sold_quantity, 
       g.gross_price AS gross_price_per_item,
       ROUND(s.sold_quantity * g.gross_price, 2) AS gross_price_total,
       pre.pre_invoice_discount_pct
FROM fact_sales_monthly s
JOIN dim_product p ON s.product_code = p.product_code
JOIN fact_gross_price g ON g.fiscal_year = s.fiscal_year AND g.product_code = s.product_code
JOIN fact_pre_invoice_deductions AS pre ON pre.customer_code = s.customer_code AND pre.fiscal_year = s.fiscal_year
WHERE s.fiscal_year = 2021     
LIMIT 1500000;
```
