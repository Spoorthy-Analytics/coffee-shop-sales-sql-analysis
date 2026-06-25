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

## Business Questions Solved

1. What are the total sales?
2. How many orders were placed?
3. What is the total quantity sold?
4. What is the Month-over-Month sales growth?
5. What is the Month-over-Month order growth?
6. What is the Month-over-Month quantity growth?
7. What were the sales, orders, and quantity sold on a specific date?

---

## Business Solutions

* Monitor monthly revenue trends.
* Identify seasonal demand.
* Improve inventory planning.
* Optimize staffing based on customer demand.
* Track business performance using KPIs.
* Support management with data-driven decision-making.

---

## SQL Queries Used

      ## Total Sales
SELECT SUM(unit_price*transaction_qty) AS TOTAL_SALES FROM coffee_shop_sales 
WHERE MONTH(TRANSACTION_DATE) =5;-- FOR MAY MONTH 
SELECT 
    MONTH(transaction_date) AS month,-- NO OF MONTHS
    ROUND(SUM(unit_price * transaction_qty)) AS total_sales,-- TOTAL SALES COLUMN
    (SUM(unit_price * transaction_qty) - LAG(SUM(unit_price * transaction_qty), 1) --- MONTHSALESDIFFERENCE
    OVER (ORDER BY MONTH(transaction_date))) / LAG(SUM(unit_price * transaction_qty), 1) --DIVISION BY PREV MONTH
    OVER (ORDER BY MONTH(transaction_date)) * 100 AS mom_increase_percentage -- PERCENTAGE
FROM 
    coffee_shop_sales
WHERE 
    MONTH(transaction_date) IN (4, 5) -- for months of April(PM) and May(CURRENT MONTH)
GROUP BY 
    MONTH(transaction_date)
ORDER BY 
    MONTH(transaction_date);
    
    ## TOTAL_ORDERS
    
SELECT COUNT(Transactionid) as Total_Orders
FROM coffee_shop_sales 
WHERE MONTH (transaction_date)= 5 ;
    SELECT 
    MONTH(transaction_date) AS month,
    ROUND(COUNT(Transactionid)) AS total_orders,
    (COUNT(Transactionid) - LAG(COUNT(Transactionid), 1) 
    OVER (ORDER BY MONTH(transaction_date))) / LAG(COUNT(Transactionid), 1) 
    OVER (ORDER BY MONTH(transaction_date)) * 100 AS mom_increase_percentage
FROM 
    coffee_shop_sales
WHERE 
    MONTH(transaction_date) IN (4, 5) -- for April and May
GROUP BY 
    MONTH(transaction_date)
ORDER BY 
    MONTH(transaction_date);
    
    ####TOTAL_QUANTITY
    
    SELECT SUM(transaction_qty) as Total_Quantity_Sold
FROM coffee_shop_sales 
WHERE MONTH(transaction_date) = 5 ;-- for month of (CM-May)

SELECT 
    MONTH(transaction_date) AS month,
    ROUND(SUM(transaction_qty)) AS total_quantity_sold,
    (SUM(transaction_qty) - LAG(SUM(transaction_qty), 1) 
    OVER (ORDER BY MONTH(transaction_date))) / LAG(SUM(transaction_qty), 1) 
    OVER (ORDER BY MONTH(transaction_date)) * 100 AS mom_increase_percentage
FROM 
    coffee_shop_sales
WHERE 
    MONTH(transaction_date) IN (4, 5)   -- for April and May
GROUP BY 
    MONTH(transaction_date)
ORDER BY 
    MONTH(transaction_date);
    ###########
    SELECT
    SUM(unit_price * transaction_qty) AS total_sales,
    SUM(transaction_qty) AS total_quantity_sold,
    COUNT(Transactionid) AS total_orders
FROM 
    coffee_shop_sales
WHERE 
    transaction_date = '2023-05-18'; ###--For 18 May 2023
###
SELECT 
    CONCAT(ROUND(SUM(unit_price * transaction_qty) / 1000, 1),'K') AS total_sales,
    CONCAT(ROUND(COUNT(transactionid) / 1000, 1),'K') AS total_orders,
    CONCAT(ROUND(SUM(transaction_qty) / 1000, 1),'K') AS total_quantity_sold
FROM 
    coffee_shop_sales
WHERE 
    transaction_date = '2023-05-18'; ###  --For 18 May 2023
    ###

##sales analaysis for weekdays and weekends
#for us weekends will be sat n sundays
#weekday _mon to fri
#in sql sun=1,mon=2.....>>>sat=7

select case
when dayofweek(transaction_date) in (1,7) then 'weekends'
else 'weekdays'
end as day_type,
concat(round(sum(unit_price*transaction_qty)/1000,1),'K')as  total_sales
from coffee_shop_sales
where month(transaction_date)=5 
group by case
when dayofweek(transaction_date) in (1,7) then 'weekends'
else 'weekdays'
end ;
###
#Sales analysis by store location
SELECT *from coffee_shop_sales;
SELeCT store_location,
CONCAT(ROUND(SUM(UNIT_PRICE * TRANSACTION_QTY)/1000,2),'K') AS total_sales
FROM coffee_shop_sales
WHERE MONTH(TRANSACTION_DATE)=5  --MAY
group by store_location
ORDER BY SUM(UNIT_PRICE * TRANSACTION_QTY) DESC;
####
#DAILY SALES WITH AVERAGE LINE
SELECT AVG(UNIT_PRICE * TRANSACTION_QTY) AS AVG_SALES FROM coffee_shop_sales 
WHERE MONTH(TRANSACTION_DATE)=5;
SELECT concat(ROUND(AVG( total_sales)/1000,1),'K') as avg_sales 
FROM(SELECT sum(UNIT_PRICE * TRANSACTION_QTY) as  total_sales
from coffee_shop_sales
where MONTH(TRANSACTION_DATE)=5
group by TRANSACTION_DATE) as internal_query;
SELECT 
DAY(TRANSACTION_DATE) AS DAY_OF_MONTH,
SUM(UNIT_PRICE*TRANSACTION_QTY) AS TOTAL_SALES
FROM coffee_shop_sales
WHERE MONTH(TRANSACTION_DATE)=5
GROUP BY DAY(TRANSACTION_DATE)
ORDER BY DAY(TRANSACTION_DATE);

SELECT 
    day_of_month,
    CASE 
        WHEN total_sales > avg_sales THEN 'Above Average'
        WHEN total_sales < avg_sales THEN 'Below Average'
        ELSE 'Equal to Average'
    END AS sales_status,
    total_sales
FROM (
    SELECT 
        DAY(transaction_date) AS day_of_month,
        SUM(unit_price * transaction_qty) AS total_sales,
        AVG(SUM(unit_price * transaction_qty)) OVER () AS avg_sales
    FROM 
        coffee_shop_sales
    WHERE 
        MONTH(transaction_date) = 5  -- Filter for May
    GROUP BY 
        DAY(transaction_date)
) AS sales_data
ORDER BY 
    day_of_month;

###
##SALES ANALYSIS BY PRODUCT CATEGORY
SELECT *from coffee_shop_sales;
select 
product_category,
concat(ROUND(sum(unit_price*transaction_qty)/1000,1),'K') as total_sales
from coffee_shop_sales 
where month(transaction_date)=5 
group by product_category 
order by sum(unit_price*transaction_qty) desc ;

###
##TOP10 PRODUCT_TYPE BY SALES
select 
product_type,
concat(ROUND(sum(unit_price*transaction_qty)/1000,1),'K') as total_sales
from coffee_shop_sales 
where month(transaction_date)=5 
group by product_type
order by sum(unit_price*transaction_qty) desc
LIMIT 10 ;
##
##SALES ANALYSIS BY DAYS AND HOURS
#SALES BY DAY | 
SELECT 
    ROUND(SUM(unit_price * transaction_qty)) AS Total_Sales,
    SUM(transaction_qty) AS Total_Quantity,
    COUNT(*) AS Total_Orders
FROM 
    coffee_shop_sales
WHERE 
    DAYOFWEEK(transaction_date) = 3 -- Filter for Tuesday (1 is Sunday, 2 is Monday, ..., 7 is Saturday)
    AND HOUR(transaction_time) = 8 -- Filter for hour number 8
    AND MONTH(transaction_date) = 5; -- Filter for May (month number 5)
#TO GET SALES FOR ALL HOURS FOR MONTH OF MAY
SELECT 
    HOUR(transaction_time) AS Hour_of_Day,
    ROUND(SUM(unit_price * transaction_qty)) AS Total_Sales
FROM 
    coffee_shop_sales
WHERE 
    MONTH(transaction_date) = 5 -- Filter for May (month number 5)
GROUP BY 
    HOUR(transaction_time)
ORDER BY 
    HOUR(transaction_time);
###
##TO GET SALES FROM MONDAY TO SUNDAY FOR MONTH OF MAY
SELECT 
    CASE 
        WHEN DAYOFWEEK(transaction_date) = 2 THEN 'Monday'
        WHEN DAYOFWEEK(transaction_date) = 3 THEN 'Tuesday'
        WHEN DAYOFWEEK(transaction_date) = 4 THEN 'Wednesday'
        WHEN DAYOFWEEK(transaction_date) = 5 THEN 'Thursday'
        WHEN DAYOFWEEK(transaction_date) = 6 THEN 'Friday'
        WHEN DAYOFWEEK(transaction_date) = 7 THEN 'Saturday'
        ELSE 'Sunday'
    END AS Day_of_Week,
    ROUND(SUM(unit_price * transaction_qty)) AS Total_Sales
FROM 
    coffee_shop_sales
WHERE 
    MONTH(transaction_date) = 5 -- Filter for May (month number 5)
GROUP BY 
    CASE 
        WHEN DAYOFWEEK(transaction_date) = 2 THEN 'Monday'
        WHEN DAYOFWEEK(transaction_date) = 3 THEN 'Tuesday'
        WHEN DAYOFWEEK(transaction_date) = 4 THEN 'Wednesday'
        WHEN DAYOFWEEK(transaction_date) = 5 THEN 'Thursday'
        WHEN DAYOFWEEK(transaction_date) = 6 THEN 'Friday'
        WHEN DAYOFWEEK(transaction_date) = 7 THEN 'Saturday'
        ELSE 'Sunday'
        END;

---

## Author

Spoorthy

GitHub:
https://github.com/Spoorthy-Analytics

