# Coffee Shop Sales Analysis | MySQL

## Project Overview

This SQL project analyzes coffee shop sales data using MySQL. It demonstrates data cleaning, KPI calculations, sales trend analysis, and business reporting using SQL queries.

The objective is to transform raw transactional data into meaningful business insights that support operational and strategic decision-making.

---

## Tools Used

* MySQL
* SQL
* Window Functions
* Aggregate Functions
* Date Functions
* Data Cleaning

---

## Database Preparation

The project includes:

* Database Creation
* Date Conversion
* Time Conversion
* Column Renaming
* Data Type Modification

---

## Key KPIs

* Total Sales
* Total Orders
* Total Quantity Sold
* Month-over-Month Sales Growth
* Month-over-Month Order Growth
* Month-over-Month Quantity Growth

---

## SQL Concepts Used

* CREATE DATABASE
* ALTER TABLE
* UPDATE
* STR_TO_DATE()
* SUM()
* COUNT()
* GROUP BY
* ORDER BY
* LAG()
* Window Functions
* Aggregate Functions

---

## Key Business Insights

* Monthly sales performance can be tracked to identify business growth.
* Month-over-Month analysis highlights increasing or declining sales trends.
* Daily sales KPIs help monitor operational performance.
* Quantity sold provides insights into customer demand.
* Total orders indicate transaction volume and customer activity.
* Sales trends can support inventory planning and staffing decisions.

---

## SQL Queries Used

  # SQL Queries

This project uses MySQL to analyze coffee shop sales performance through KPI calculations, sales trends, customer purchasing behavior, and product performance.

---

## 1. Total Sales & Month-over-Month (MoM) Sales Growth

### Total Sales for May

```sql
SELECT SUM(unit_price * transaction_qty) AS total_sales
FROM coffee_shop_sales
WHERE MONTH(transaction_date) = 5;
```

### Month-over-Month Sales Growth

```sql
SELECT
    MONTH(transaction_date) AS month,
    ROUND(SUM(unit_price * transaction_qty)) AS total_sales,
    (
        SUM(unit_price * transaction_qty)
        - LAG(SUM(unit_price * transaction_qty))
          OVER (ORDER BY MONTH(transaction_date))
    )
    /
    LAG(SUM(unit_price * transaction_qty))
    OVER (ORDER BY MONTH(transaction_date))
    * 100 AS mom_increase_percentage
FROM coffee_shop_sales
WHERE MONTH(transaction_date) IN (4,5)
GROUP BY MONTH(transaction_date)
ORDER BY MONTH(transaction_date);
```

---

## 2. Total Orders & MoM Growth

### Total Orders for May

```sql
SELECT COUNT(transactionid) AS total_orders
FROM coffee_shop_sales
WHERE MONTH(transaction_date) = 5;
```

### Month-over-Month Order Growth

```sql
SELECT
    MONTH(transaction_date) AS month,
    COUNT(transactionid) AS total_orders,
    (
        COUNT(transactionid)
        - LAG(COUNT(transactionid))
          OVER (ORDER BY MONTH(transaction_date))
    )
    /
    LAG(COUNT(transactionid))
    OVER (ORDER BY MONTH(transaction_date))
    * 100 AS mom_increase_percentage
FROM coffee_shop_sales
WHERE MONTH(transaction_date) IN (4,5)
GROUP BY MONTH(transaction_date)
ORDER BY MONTH(transaction_date);
```

---

## 3. Total Quantity Sold & MoM Growth

### Quantity Sold for May

```sql
SELECT SUM(transaction_qty) AS total_quantity_sold
FROM coffee_shop_sales
WHERE MONTH(transaction_date) = 5;
```

### Month-over-Month Quantity Growth

```sql
SELECT
    MONTH(transaction_date) AS month,
    SUM(transaction_qty) AS total_quantity_sold,
    (
        SUM(transaction_qty)
        - LAG(SUM(transaction_qty))
          OVER (ORDER BY MONTH(transaction_date))
    )
    /
    LAG(SUM(transaction_qty))
    OVER (ORDER BY MONTH(transaction_date))
    * 100 AS mom_increase_percentage
FROM coffee_shop_sales
WHERE MONTH(transaction_date) IN (4,5)
GROUP BY MONTH(transaction_date)
ORDER BY MONTH(transaction_date);
```

---

## 4. Daily KPI Analysis

### Sales, Orders & Quantity Sold for 18-May-2023

```sql
SELECT
    SUM(unit_price * transaction_qty) AS total_sales,
    SUM(transaction_qty) AS total_quantity_sold,
    COUNT(transactionid) AS total_orders
FROM coffee_shop_sales
WHERE transaction_date = '2023-05-18';
```

### KPI Values in Thousands

```sql
SELECT
    CONCAT(ROUND(SUM(unit_price * transaction_qty)/1000,1),'K') AS total_sales,
    CONCAT(ROUND(COUNT(transactionid)/1000,1),'K') AS total_orders,
    CONCAT(ROUND(SUM(transaction_qty)/1000,1),'K') AS total_quantity_sold
FROM coffee_shop_sales
WHERE transaction_date='2023-05-18';
```

---

## 5. Weekday vs Weekend Sales Analysis

```sql
SELECT
    CASE
        WHEN DAYOFWEEK(transaction_date) IN (1,7)
        THEN 'Weekend'
        ELSE 'Weekday'
    END AS day_type,

    CONCAT(
        ROUND(SUM(unit_price * transaction_qty)/1000,1),
        'K'
    ) AS total_sales

FROM coffee_shop_sales

WHERE MONTH(transaction_date)=5

GROUP BY day_type;
```

---

## 6. Store-wise Sales Analysis

```sql
SELECT
    store_location,

    CONCAT(
        ROUND(SUM(unit_price * transaction_qty)/1000,2),
        'K'
    ) AS total_sales

FROM coffee_shop_sales

WHERE MONTH(transaction_date)=5

GROUP BY store_location

ORDER BY SUM(unit_price * transaction_qty) DESC;
```

---

## 7. Daily Sales vs Average Sales

### Average Daily Sales

```sql
SELECT
    CONCAT(
        ROUND(AVG(total_sales)/1000,1),
        'K'
    ) AS average_sales

FROM
(
    SELECT
        SUM(unit_price * transaction_qty) AS total_sales
    FROM coffee_shop_sales
    WHERE MONTH(transaction_date)=5
    GROUP BY transaction_date
) AS sales_data;
```

### Daily Sales Status

```sql
SELECT
    day_of_month,

    CASE
        WHEN total_sales > avg_sales THEN 'Above Average'
        WHEN total_sales < avg_sales THEN 'Below Average'
        ELSE 'Equal to Average'
    END AS sales_status,

    total_sales

FROM
(
    SELECT
        DAY(transaction_date) AS day_of_month,
        SUM(unit_price * transaction_qty) AS total_sales,
        AVG(SUM(unit_price * transaction_qty)) OVER() AS avg_sales

    FROM coffee_shop_sales

    WHERE MONTH(transaction_date)=5

    GROUP BY DAY(transaction_date)
) AS sales_data

ORDER BY day_of_month;
```

---

## 8. Product Category Analysis

```sql
SELECT
    product_category,

    CONCAT(
        ROUND(SUM(unit_price * transaction_qty)/1000,1),
        'K'
    ) AS total_sales

FROM coffee_shop_sales

WHERE MONTH(transaction_date)=5

GROUP BY product_category

ORDER BY SUM(unit_price * transaction_qty) DESC;
```

---

## 9. Top 10 Products by Sales

```sql
SELECT
    product_type,

    CONCAT(
        ROUND(SUM(unit_price * transaction_qty)/1000,1),
        'K'
    ) AS total_sales

FROM coffee_shop_sales

WHERE MONTH(transaction_date)=5

GROUP BY product_type

ORDER BY SUM(unit_price * transaction_qty) DESC

LIMIT 10;
```

---

## 10. Sales by Hour

```sql
SELECT
    HOUR(transaction_time) AS hour_of_day,

    ROUND(SUM(unit_price * transaction_qty)) AS total_sales

FROM coffee_shop_sales

WHERE MONTH(transaction_date)=5

GROUP BY HOUR(transaction_time)

ORDER BY HOUR(transaction_time);
```

---

## 11. Sales by Day of Week

```sql
SELECT

CASE
    WHEN DAYOFWEEK(transaction_date)=2 THEN 'Monday'
    WHEN DAYOFWEEK(transaction_date)=3 THEN 'Tuesday'
    WHEN DAYOFWEEK(transaction_date)=4 THEN 'Wednesday'
    WHEN DAYOFWEEK(transaction_date)=5 THEN 'Thursday'
    WHEN DAYOFWEEK(transaction_date)=6 THEN 'Friday'
    WHEN DAYOFWEEK(transaction_date)=7 THEN 'Saturday'
    ELSE 'Sunday'
END AS day_of_week,

ROUND(SUM(unit_price * transaction_qty)) AS total_sales

FROM coffee_shop_sales

WHERE MONTH(transaction_date)=5

GROUP BY day_of_week;
```

---
## Business Questions Solved

1. What are the total sales?
2. How many orders were placed?
3. What is the total quantity sold?
4. What is the Month-over-Month sales growth?
5. What is the Month-over-Month order growth?
6. What is the Month-over-Month quantity growth?
7. What were the sales, orders, and quantity sold on a specific date?

---

# Business Insights

* Calculated Total Sales, Orders, and Quantity Sold KPIs for monthly performance monitoring.
* Measured Month-over-Month (MoM) growth using SQL window functions.
* Compared sales performance between weekdays and weekends.
* Identified top-performing store locations based on revenue.
* Classified daily sales as Above Average or Below Average.
* Ranked product categories and top-selling products by revenue.
* Analyzed hourly and day-wise sales patterns to identify peak business periods.
* Generated actionable insights to support inventory planning, staffing, and sales optimization.


---

## Author

Spoorthy

GitHub:
https://github.com/Spoorthy-Analytics

