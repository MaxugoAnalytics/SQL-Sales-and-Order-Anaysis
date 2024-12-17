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

Below is the core SQL query used for the analysis:

```sql
CREATE TABLE ordersalesanalysis.Sales_data.Sales (
    Order_ID INT PRIMARY KEY,
    Order_Date DATE,
    Ship_Mode VARCHAR(50),
    Segment VARCHAR(50),
    Country VARCHAR(100),
    City VARCHAR(100),
    State VARCHAR(100),
    Postal_Code VARCHAR(20),
    Region VARCHAR(50),
    Category VARCHAR(100),
    Product_ID VARCHAR(50),
    Cost_Price DECIMAL(10, 2),
    List_Price DECIMAL(10, 2),
    Quantity INT,
    Discount_Percent DECIMAL(5, 2),
    Year INT,
    Week INT,
    Day_of_the_Week VARCHAR(20),
    Discount_Amount DECIMAL(10, 2),
    Sales_Price DECIMAL(10, 2),
    Profit DECIMAL(10, 2)
);

SELECT * FROM `ordersalesanalysis.Sales_data.Sales` LIMIT 1000;

--- Total Sales and Profit by Category
select Category,
sum(Sales_Price) as Total_Sales,
sum(Profit) as Total_Profit
from ordersalesanalysis.Sales_data.Sales
group by Category
order by  Total_Sales;

--Average Profit by Region
select Region,
avg(Profit) as Average_Profit
from ordersalesanalysis.Sales_data.Sales
group by Region
order by Average_Profit;

--Total Sales, Cost, and Profit for Each Year
SELECT Year,
sum(Sales_Price) as Total_Sales,
sum(Cost_Price) as Total_Cost,
sum(Profit) as Total_Profit
from ordersalesanalysis.Sales_data.Sales
group by Year
order by Total_Cost;

--Top  Best-Selling Categories by Sales
SELECT 
    Category,
    SUM(Sales_Price) AS Total_Sales
FROM 
   ordersalesanalysis.Sales_data.Sales
GROUP BY 
    Category
ORDER BY 
    Total_Sales DESC;

--- Top 5 Cities by Total Profit
select 
City,
sum(Profit) as Total_Profit
from ordersalesanalysis.Sales_data.Sales
group by City
order by Total_Profit DESC
limit 5;

--Average Discount and Profit by Ship Mode
select Ship_Mode,
avg(Discount_Amount) as Average_Discount,
AVG(Profit) AS Avg_Profit
from  ordersalesanalysis.Sales_data.Sales
group by Ship_Mode;

-- Total Sales and Profit by Region
SELECT Region,
      SUM(Sales_Price) AS Total_Sales,
      sum(Profit) as Total_Profit
from  ordersalesanalysis.Sales_data.Sales
group by Region
ORDER BY 
    Total_Sales DESC; 

---Week Performance: Sales by Day of the Week
select Week,
 SUM(Sales_Price) AS Total_Sales
from ordersalesanalysis.Sales_data.Sales
group by Week
order by Total_Sales DESC;

---Most Profitable Products
select Product_ID,
 SUM(Profit) AS Total_Profit,
    SUM(Sales_Price) AS Total_Sales
from ordersalesanalysis.Sales_data.Sales
group by Product_ID 
ORDER BY Total_Profit DESC
LIMIT 10;

--- Products with Negative Profit
SELECT  Product_ID,
         SUM(Profit) AS Total_Profit
FROM 
    ordersalesanalysis.Sales_data.Sales
GROUP BY 
    Product_ID
HAVING 
    SUM(Profit) < 0
ORDER BY 
    Total_Profit;

---Average Cost and Sales Price by Category
select  Category,
        avg(Cost_Price) as Average_Cost_Price,
        avg(Sales_Price) as  Average_Sales_Price
from ordersalesanalysis.Sales_data.Sales        
group by Category;

---Which region generated the highest profit in 2023?
select Region,
SUM(Profit) as Total_Profit
from ordersalesanalysis.Sales_data.Sales
where Year = 2023
group by Region
order by Total_Profit DESC
LIMIT 1;

---What are the top 5 products  based on the total Quantity sold?
select Product_ID,
sum(Quantity) as Total_Quantity
FROM  ordersalesanalysis.Sales_data.Sales
group by Product_ID
order by Total_Quantity DESC
LIMIT 5;

---What is the profit margin for each order, ordered by the highest margin?
SELECT Order_ID, 
       (Profit / Sales_Price) * 100 AS Profit_Margin_Percent
FROM ordersalesanalysis.Sales_data.Sales
WHERE Sales_Price > 0
ORDER BY Profit_Margin_Percent DESC;

--How does a high discount percentage (> 3%) affect total profit?
SELECT 
    CASE 
        WHEN Discount_Percent > 3 THEN 'High Discount'
        ELSE 'Low Discount'
    END AS Discount_Category,
    SUM(Profit) AS Total_Profit
FROM ordersalesanalysis.Sales_data.Sales
GROUP BY Discount_Category;

-- Which cities have the highest total sales?
select City,
sum(Sales_Price) as Total_Sales
from ordersalesanalysis.Sales_data.Sales
GROUP BY City
order by Total_Sales desc
limit 5;

--For each product, what is the total sales, total quantity sold, and average discount?
SELECT Product_ID, 
       SUM(Sales_Price) AS Total_Sales, 
       SUM(Quantity) AS Total_Quantity,
       AVG(Discount_Percent) AS Avg_Discount
FROM ordersalesanalysis.Sales_data.Sales
GROUP BY Product_ID
ORDER BY Total_Sales DESC;

---What is the month-over-month sales growth percentage in 2023?
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

---For each order, what is the difference between Sales_Price and Cost_Price?
SELECT Order_ID, 
       Sales_Price, 
       Cost_Price, 
       (Sales_Price - Cost_Price) AS Sales_vs_Cost
FROM ordersalesanalysis.Sales_data.Sales
ORDER BY Sales_vs_Cost DESC;

---How has sales and profit changed from year to year?
SELECT Year, 
       SUM(Sales_Price) AS Total_Sales, 
       SUM(Profit) AS Total_Profit,
       LAG(SUM(Sales_Price)) OVER (ORDER BY Year) AS Previous_Year_Sales,
       ROUND(((SUM(Sales_Price) - LAG(SUM(Sales_Price)) OVER (ORDER BY Year)) / 
       LAG(SUM(Sales_Price)) OVER (ORDER BY Year)) * 100, 2) AS Sales_Growth_Percent
FROM ordersalesanalysis.Sales_data.Sales
GROUP BY Year
ORDER BY Year;

---What is the distribution of sales across different Segments?
SELECT Segment, SUM(Sales_Price) AS Total_Sales
FROM ordersalesanalysis.Sales_data.Sales
GROUP BY Segment
ORDER BY Total_Sales DESC;

