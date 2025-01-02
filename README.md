# Coffee-Shop-Sales | SQL Practical Project

## Problem Questions

### KPI's Requirement
**1. Total Sales Analysis**

```sql
SELECT ROUND(SUM(Sales),0) AS TotalSales
FROM coffee_shop_sales;
```
**2. Calculate the total sales for each respective month**
```sql
SELECT
	DATEPART(MONTH, transaction_date) AS Month_No,
	DATENAME(MONTH, transaction_date) AS Months,
	ROUND(SUM(Sales),0) AS Total_sales
FROM coffee_shop_sales
GROUP BY DATEPART(MONTH, transaction_date), DATENAME(MONTH, transaction_date)
ORDER BY DATEPART(MONTH, transaction_date);
```
**3. Determine the month-to-month increase or decrease in sales + Calculate the difference in sales between the selected month and the previous month.**
```sql
WITH MonthlySales AS (
    SELECT 
        MONTH(transaction_date) AS SaleMonth_No,
        DATENAME(MONTH, transaction_date) AS SalesMonth,
        SUM(Sales) AS TotalSales
    FROM 
        coffee_shop_sales
	GROUP BY 
        MONTH(transaction_date), DATENAME(MONTH, transaction_date)
)
SELECT 
    SaleMonth_No,
    SalesMonth,
    TotalSales,
    ISNULL(LAG(TotalSales) OVER (ORDER BY SaleMonth_No), 0) AS PreviousMonthSales,
    ISNULL(TotalSales - LAG(TotalSales) OVER (ORDER BY SaleMonth_No), 0) AS SalesChange,
    CONCAT(ISNULL(ROUND(
        CASE 
            WHEN LAG(TotalSales) OVER (ORDER BY SaleMonth_No) = 0 THEN 0
            ELSE ((TotalSales - LAG(TotalSales) OVER (ORDER BY SaleMonth_No)) * 100.0 
                  / LAG(TotalSales) OVER (ORDER BY SaleMonth_No))
        END
		,1)	,0), '%') AS PercentageChange
FROM 
    MonthlySales
ORDER BY 
    SaleMonth_No;
```
**4.	Total Order Analysis**
```sql
SELECT	COUNT(transaction_id) AS TotalOrder
FROM coffee_shop_sales;
```
**5. Calculate the total number of orders for each respective month
```sql
SELECT
	MONTH(transaction_date) AS SaleMonth_No,
    DATENAME(MONTH, transaction_date) AS SalesMonth,
	COUNT(transaction_id) AS TotalOrder
FROM coffee_shop_sales
GROUP BY MONTH(transaction_date), DATENAME(MONTH, transaction_date)
ORDER BY SaleMonth_No;
```
**6•	Determine the month-to-month increase or decrease in the number of orders + Calculate the difference in the number of orders between the selected month and the previous month.**
```sql
WITH MonthlyOrders AS
(	
	SELECT
		MONTH(transaction_date) AS SalesMonth_No,
		DATENAME(MONTH, transaction_date) AS SalesMonth,
		COUNT(transaction_id) AS TotalOrder
	FROM coffee_shop_sales
	GROUP BY MONTH(transaction_date), DATENAME(MONTH, transaction_date)
)
	SELECT
		SalesMonth_No,
		SalesMonth,
		TotalOrder,
		ISNULL(LAG(TotalOrder) OVER(ORDER BY SalesMonth_No, SalesMonth),0) AS PreviousMonthOrder,
		ISNULL(TotalOrder - LAG(TotalOrder) OVER(ORDER BY SalesMonth_No, SalesMonth),0) AS OrderChange,

CONCAT(ISNULL(
	CASE
	WHEN LAG(TotalOrder) OVER(ORDER BY SalesMonth_No, SalesMonth) = 0 THEN 0
	ELSE ((TotalOrder - LAG(TotalOrder) OVER(ORDER BY SalesMonth_No, SalesMonth)) * 100
		/ LAG(TotalOrder) OVER(ORDER BY SalesMonth_No, SalesMonth))
	END 
		,0),'%') AS PercentageChange

	FROM MonthlyOrders
	ORDER BY SalesMonth_No, SalesMonth;
```
**7.	Total Quantity Sold Analysis**
```sql
SELECT 
	SUM(transaction_qty) AS TotalQuantitySold
FROM coffee_shop_sales;
```
**8•	Calculate the total quantity sold for each respective month**
```sql
SELECT
	MONTH(transaction_date) AS SalesMonth_No,
	DATENAME(MONTH, transaction_date) AS SalesMonth,
	SUM(transaction_qty) AS TotalQuantitySold
FROM coffee_shop_sales
GROUP BY MONTH(transaction_date), DATENAME(MONTH, transaction_date) 
ORDER BY SalesMonth_No;
```
**9•	Determine the month-to-month increase or decrease in the total quantity sold + Calculate the difference in the total quantity sold between the selected month and the previous month.**
```sql
SELECT
	SalesMonth_No,
	SalesMonth,
	TotalQuantitySold,
	ISNULL(LAG(TotalQuantitySold) OVER(ORDER BY SalesMonth_No, SalesMonth),0) AS PrevMonthlyQuantitySold,
	ISNULL(TotalQuantitySold - LAG(TotalQuantitySold) OVER(ORDER BY SalesMonth_No, SalesMonth),0) AS MonthlyQuantitySoldChange,
		
	CONCAT(ISNULL(
		CASE	
		WHEN LAG(TotalQuantitySold) OVER(ORDER BY SalesMonth_No, SalesMonth) = 0 THEN 0
		ELSE (TotalQuantitySold - LAG(TotalQuantitySold) OVER(ORDER BY SalesMonth_No, SalesMonth)) * 100
			/LAG(TotalQuantitySold) OVER(ORDER BY SalesMonth_No, SalesMonth)
		END
		,0),'%')
		AS PercentageChange 
FROM 
	(
		SELECT
		MONTH(transaction_date) AS SalesMonth_No,
		DATENAME(MONTH, transaction_date) AS SalesMonth,
		SUM(transaction_qty) AS TotalQuantitySold
	FROM coffee_shop_sales
	GROUP BY MONTH(transaction_date), DATENAME(MONTH, transaction_date)  
	) AS MonthlyQuantitySold

	ORDER BY SalesMonth_No, SalesMonth;
```
### CHARTS REQUIREMENT

**1.	Calendar Heat Map**

•	Implement a calendar heat map that dynamically adjusts based on the selected month from a slicer.

•	Each day on the calendar will be color-coded to represent sales volume, with the darker shades indicating higher sales.

•	Implement tooltips to display detailed metrics (Sales, Order and Quantity) when hovering over a specific day.
```sql
SELECT
	DATEPART(DAY, transaction_date) AS DayNo,
	DATENAME(WEEKDAY, transaction_date) AS [DayName],
	ROUND(SUM(Sales),0) AS TotalSales,
	COUNT(transaction_id) AS TotalOrders,
	SUM(transaction_qty) AS TotalQuantitySold
FROM coffee_shop_sales
WHERE DATEPART(MONTH, transaction_date) = 4-----Filter for the Month of April
GROUP BY DATEPART(DAY, transaction_date), DATENAME(WEEKDAY, transaction_date)
ORDER BY DayNo;
```
**2.	Sales Analysis by Weekdays and Weekends**

•	Segment sales data into weekdays and weekends to analyze performance variations.

•	Provide insights into whether sales patterns differ significantly between weekdays and weekends.
```sql
SELECT
    CASE 
        WHEN DATEPART(WEEKDAY, transaction_date) IN (1, 7) THEN 'Weekends' -- Sunday = 1 and Saturday = 7
        ELSE 'Weekdays'
    END AS DayType,
    ROUND(SUM(Sales), 0) AS TotalSales
FROM coffee_shop_sales
WHERE MONTH(transaction_date) = 5 -- For the month of May
GROUP BY 
    CASE 
        WHEN DATEPART(WEEKDAY, transaction_date) IN (1, 7) THEN 'Weekends'
        ELSE 'Weekdays'
    END;
```
**3.	Sales Analysis by Store Location**

•	Visualize sales data by different store location

•	Include month-over-month (MoM) difference metrics based on the selected month in the slicer.

•	Highlight MoM sales increase and decrease for each store location to identify trends.
```sql
SELECT
	store_location AS StoreLocation,
	ROUND(SUM(Sales),0) AS TotalSales
FROM coffee_shop_sales
WHERE MONTH(transaction_date) = 5 ---Month of May
GROUP BY store_location
ORDER BY TotalSales DESC;
```
**4.	Daily Sales Analysis with Average Line**

•	Display daily sales for the selected month with a line chart.
```sql
	SELECT
		DAY(transaction_date) AS DayNo,
		ROUND(SUM(Sales),2) AS TotalSales
	FROM coffee_shop_sales
	WHERE MONTH(transaction_date) = 5 ---- For the Month of May
	GROUP BY transaction_date
	ORDER BY DayNo;
```
•	Incorporate an average line on the chart to represent the average daily sales.
```sql
Average Sales
SELECT
	ROUND(AVG(TotalSales)/1000,1) AS AverageSales
FROM
(
	SELECT
		ROUND(SUM(Sales),2) AS TotalSales
	FROM coffee_shop_sales
	WHERE MONTH(transaction_date) = 5 ---- For the Month of May
	GROUP BY transaction_date
) AS Subquery
```
•	Highlight bars exceeding or falling below the average sales to identify exceptional sales days.
```sql
SELECT
	DayNo,
	CASE 
		WHEN TotalSales > AverageSales THEN 'Above Average'
		WHEN TotalSales < AverageSales THEN 'Below Average'
		ELSE 'Average'
	END AS SalesStatus,
	TotalSales
FROM
(
	SELECT
		DAY(transaction_date) AS DayNo,
		ROUND(SUM(Sales),2) AS TotalSales,
		AVG(SUM(Sales)) OVER() AS AverageSales
	FROM coffee_shop_sales
	WHERE MONTH(transaction_date) = 5 ---- For the Month of May
	GROUP BY DAY(transaction_date)
) AS AverageSales
ORDER BY DayNo
```
**5.	Sales Analysis by Product Category**

•	Analyze sales performance across different product categories.

•	Provide insights into which product categories contribute the most to overall sales.
```sql
	SELECT
		product_category,
		ROUND(SUM(Sales),2) AS TotalSales
	FROM coffee_shop_sales
	WHERE MONTH(transaction_date) = 5 ---- For the Month of May
	GROUP BY product_category
```
**6.	Top 10 products by Sales**

•	Identify and display the top 10 products based on sales volume.

•	Allow users to quickly visualize the best performing products in terms of sales.
```sql
	SELECT
		TOP (10)
		product_type,
		ROUND(SUM(Sales),2) AS TotalSales
	FROM coffee_shop_sales
	GROUP BY product_type
	ORDER BY TotalSales DESC;
```
**7.	Sales Analysis by Days and Hours**

•	Utilize a heat map to visualize sales patterns by days and hours.

•	Implement tooltips to display detailed metrics (Sales, orders and Quantity) when hovering over a specific day-hour
```sql
SELECT
	DATENAME(WEEKDAY, transaction_date) AS [Days],
	DATEPART(HOUR,transaction_time) AS [Hour],
	ROUND(SUM(Sales),2) AS TotalSales,
	COUNT(*) AS TotalOrders,
	SUM(transaction_qty) AS TotalQuantitySold
FROM coffee_shop_sales
WHERE MONTH(transaction_date) = 5 AND DATENAME(WEEKDAY, transaction_date) = 'Monday'
GROUP BY DATENAME(WEEKDAY, transaction_date), DATEPART(HOUR,transaction_time)
ORDER BY [Days]
```
