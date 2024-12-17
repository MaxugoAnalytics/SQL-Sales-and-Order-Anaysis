# Sales Data Analysis Project

## Overview

This project focuses on analyzing sales data for the year 2023. The dataset contains information on orders, sales prices, discounts, and other related metrics. The primary goal is to generate insights such as total monthly sales, previous month sales, and month-over-month growth percentages. The analysis is performed using SQL queries on a PostgreSQL database.

---

## Features

1. **Monthly Sales Analysis**  
   Calculates total sales for each month and compares it with the previous month's sales to determine growth percentages.

2. **Performance Metrics**  
   Key metrics like:
   - Total Sales
   - Previous Month's Sales
   - Month-over-Month Growth Percentages

3. **Data Aggregation**  
   The analysis aggregates sales data using SQL queries, focusing on:
   - Month
   - Sales Price
   - Discounts
   - Profit Margins

4. **Error-Free Execution**  
   The SQL query handles data grouping and ensures compatibility with window functions like `LAG()` for calculating trends.

---

## Dataset Information

The dataset used in this project includes the following columns:

| Column Name         | Description                                 |
|----------------------|---------------------------------------------|
| `Order_ID`          | Unique identifier for each order            |
| `Order_Date`        | Date the order was placed                   |
| `Month`             | Month extracted from `Order_Date`           |
| `Sales_Price`       | Final selling price after discount          |
| `Discount_Amount`   | Total discount applied to the sale          |
| `Cost_Price`        | Base cost price of the product              |
| `Profit`            | Profit earned from the sale                 |
| `Quantity`          | Quantity of items purchased                 |
| `Year`              | Year of the order                           |
| `Week`              | Week number in the year                     |

---

## SQL Query

Below is the core SQL query used for monthly sales analysis:

```sql
WITH MonthlySales AS (
    SELECT EXTRACT(MONTH FROM Order_Date) AS Month,
           SUM(Sales_Price) AS Total_Sales
    FROM ordersalesanalysis.Sales_data.Sales
    WHERE Year = 2023
    GROUP BY EXTRACT(MONTH FROM Order_Date)
)
SELECT Month,
       Total_Sales,
       LAG(Total_Sales) OVER (ORDER BY Month) AS Previous_Month_Sales,
       ROUND(((Total_Sales - LAG(Total_Sales) OVER (ORDER BY Month)) / 
       LAG(Total_Sales) OVER (ORDER BY Month)) * 100, 2) AS Growth_Percent
FROM MonthlySales
ORDER BY Month;
