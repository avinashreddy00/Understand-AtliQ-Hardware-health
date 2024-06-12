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

