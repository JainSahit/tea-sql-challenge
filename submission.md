# Challenge 1
# The leadership team has asked us to graph total monthly sales over time. Write a query that returns the data we need to complete this request.
**Query**

```sql
SELECT DATENAME(MONTH, DATEADD(M, MONTH(TransactionDate), - 1)) [Month],
   YEAR(TransactionDate) [Year],
   FORMAT(SUM(TransactionAmount), 'C') [Total Price]
FROM Sales.CustomerTransactions
WHERE TransactionTypeID=1
GROUP BY YEAR(TransactionDate), MONTH(TransactionDate)
ORDER BY YEAR(TransactionDate), MONTH(TransactionDate)
```

**Output**

| Month     | Year | Total Price   |
|-----------|------|---------------|
| January   | 2013 | $4,335,972.97 |
| February  | 2013 | $3,193,304.60 |
| March     | 2013 | $4,451,081.62 |
| April     | 2013 | $4,668,548.40 |
| May       | 2013 | $5,080,661.05 |
| June      | 2013 | $4,679,392.14 |
| July      | 2013 | $5,039,033.18 |
| August    | 2013 | $4,020,390.18 |
| September | 2013 | $4,345,897.56 |
| October   | 2013 | $4,315,500.25 |
| November  | 2013 | $4,252,081.74 |
| December  | 2013 | $4,181,408.95 |
| January   | 2014 | $4,677,669.28 |
| February  | 2014 | $3,990,741.07 |
| March     | 2014 | $4,441,218.55 |
| April     | 2014 | $4,709,520.43 |
| May       | 2014 | $5,279,235.69 |
| June      | 2014 | $4,906,641.36 |
| July      | 2014 | $5,504,246.85 |
| August    | 2014 | $4,698,313.60 |
| September | 2014 | $4,465,414.61 |
| October   | 2014 | $5,104,486.71 |
| November  | 2014 | $4,621,813.04 |
| December  | 2014 | $5,019,615.70 |
| January   | 2015 | $5,061,954.63 |
| February  | 2015 | $4,824,617.67 |
| March     | 2015 | $5,207,351.93 |
| April     | 2015 | $5,834,255.09 |
| May       | 2015 | $5,152,840.72 |
| June      | 2015 | $5,193,217.15 |
| July      | 2015 | $5,929,023.38 |
| August    | 2015 | $4,528,888.44 |
| September | 2015 | $5,361,990.68 |
| October   | 2015 | $5,165,857.34 |
| November  | 2015 | $4,702,590.28 |
| December  | 2015 | $5,127,633.50 |
| January   | 2016 | $5,103,948.25 |
| February  | 2016 | $4,596,534.78 |
| March     | 2016 | $5,330,250.56 |
| April     | 2016 | $5,236,062.81 |
| May       | 2016 | $5,704,232.71 |

**Visualization**

![alt text](https://github.com/JainSahit/tea-sql-challenge/blob/main/images/Screen%20Shot%202021-06-13%20at%205.42.06%20PM.png?raw=true)

# Challenge 2
# What is the fastest growing customer category in Q1 2016 (compared to same quarter sales in the previous year)? What is the growth rate?

There are two possible ways of finding the fastest growing customer category in Q1 2016. 

First, using quarterly order count; Second, using quarterly revenue. 

According to quartley order counts, **Supermarket** had the highest growth rate of **25.31%** from 1272 order in Q1 2015 to 1594 orders in Q1 2016. 
While according to quartley revenues, **Computer Store** had the highest growth rate of **3.74%%** from $919675.75 in Q1 2015 to $954037.45 in Q1 2016.

**Output**

| CustomerCategoryName | GrowthRate_OrderCount | GrowthRate_Revenue |
|----------------------|-----------------------|--------------------|
| Computer Store       | 12.83 %               | 3.74 %             |
| Corporate            | -0.16 %               | -3.68 %            |
| Gift Store           | -1.49 %               | 2.72 %             |
| Novelty Shop         | 3.84 %                | 1.89 %             |
| Supermarket          | 25.31 %               | 2.93 %             |

**Query**

```sql
CREATE VIEW Q1_View AS
    WITH Q1_CTE (CustomerCategoryName, OrderCount, Revenue, OrderDate)
    AS
    (
        SELECT [CustomerCategories].[CustomerCategoryName] AS [CustomerCategoryName],
        SUM(CAST(1 as BIGINT)) AS [OrderCount],
        SUM((([t0].[Quantity]) * [t0].[UnitPrice])) AS [Revenue],
        DATEPART(year,DATEADD(month,2,[t0].[OrderDate])) AS [OrderDate]
        FROM [Sales].[Customers] [Customers]
        INNER JOIN [Sales].[CustomerCategories] [CustomerCategories] ON ([Customers].[CustomerCategoryID] = [CustomerCategories].[CustomerCategoryID])
        INNER JOIN (
        SELECT [Orders].[CustomerID] AS [CustomerID],
            [Orders].[OrderDate] AS [OrderDate],
            [OrderLines].[Quantity] AS [Quantity],
            [OrderLines].[UnitPrice] AS [UnitPrice]
        FROM [Sales].[Orders] [Orders]
            INNER JOIN [Sales].[OrderLines] [OrderLines] ON ([Orders].[OrderID] = [OrderLines].[OrderID])
        ) [t0] ON ([Customers].[CustomerID] = [t0].[CustomerID])
        WHERE ((DATEPART(quarter,DATEADD(month,2,[t0].[OrderDate])) = 1) AND (DATEPART(year,DATEADD(month,2,[t0].[OrderDate])) IN (2015, 2016)))
        GROUP BY [CustomerCategories].[CustomerCategoryName],
        DATEPART(quarter,DATEADD(month,2,[t0].[OrderDate])),
        DATEPART(year,DATEADD(month,2,[t0].[OrderDate]))
    )
    SELECT CustomerCategoryName, OrderDate,
        LAG(OrderCount)
        OVER(PARTITION BY CustomerCategoryName ORDER BY OrderDate) AS PreviousYearOC,
        OrderCount - LAG(OrderCount)
        OVER (PARTITION BY CustomerCategoryName ORDER BY OrderDate) AS DifferencePreviousYearOC,
        LAG(Revenue)
        OVER(PARTITION BY CustomerCategoryName ORDER BY OrderDate) AS PreviousYearRevenue,
        Revenue - LAG(Revenue)
        OVER (PARTITION BY CustomerCategoryName ORDER BY OrderDate) AS DifferencePreviousYearRevenue
    FROM Q1_CTE
```
*Subquey for Commom Table Expression generated through Tableau.
```sql
SELECT CustomerCategoryName,
    FORMAT((CAST(DifferencePreviousYearOC AS FLOAT)/PreviousYearOC), 'P') AS [GrowthRate_OrderCount],
    FORMAT((DifferencePreviousYearRevenue/PreviousYearRevenue), 'P') AS [GrowthRate_Revenue]
FROM Q1_View WHERE OrderDate=2016
```

**Visualization**

![alt text](https://github.com/JainSahit/tea-sql-challenge/blob/main/images/Screen%20Shot%202021-06-14%20at%202.55.11%20AM.png?raw=true)

# Challenge 3
# Write a query to return the list of suppliers that WWI has purchased from, along with # of invoices paid, # of invoices still outstanding, and average invoice amount.

**Query**

```sql
SELECT S.SupplierID, SupplierName, 
    COUNT(TransactionAmount) - COUNT(NULLIF(OutstandingBalance,0)) [# Invoices Paid], 
    COUNT(NULLIF(OutstandingBalance,0)) [# Invoices Outstanding],
    FORMAT(AVG(TransactionAmount), 'C') [AVG Invoice Amount]
FROM Purchasing.Suppliers AS S
JOIN Purchasing.SupplierTransactions 
ON S.SupplierID=Purchasing.SupplierTransactions.SupplierID
WHERE TransactionTypeID=5
GROUP BY S.SupplierID, S.SupplierName
ORDER BY S.SupplierID
```

**Output**

| SupplierID | SupplierName             | # Invoices Paid | # Invoices Outstanding | AVG Invoice Amount |
|------------|--------------------------|-----------------|------------------------|--------------------|
| 1          | A Datum Corporation      | 5               | 0                      | $5,505.06          |
| 2          | Contoso, Ltd.            | 1               | 0                      | $360.53            |
| 4          | Fabrikam, Inc.           | 1053            | 1                      | $739,825.57        |
| 5          | Graphic Design Institute | 13              | 0                      | $574.03            |
| 7          | Litware, Inc.            | 983             | 1                      | $311,009.07        |
| 10         | Northwind Electric Cars  | 10              | 0                      | $9,063.90          |
| 12         | The Phone Company        | 5               | 0                      | $11,688.60         |

# Challenge 4
# Using "unit price" and "recommended retail price", which item in the warehouse has the lowest gross profit amount? Which item has the highest? What is the median gross profit across all items in the warehouse?

Item with the lowest gross profit, _**3 kg Courier post bag (White) 300x190x95mm**_ with just __$0.33__ profit.

```sql
SELECT TOP 1 WITH TIES StockItemName, UnitPrice, RecommendedRetailPrice, 
    FORMAT(RecommendedRetailPrice-UnitPrice, 'C') [MinProfit]
FROM Warehouse.StockItems
ORDER BY MinProfit
```

| StockItemID | StockItemName                              | UnitPrice | RecommendedRetailPrice | MinProfit |
|-------------|--------------------------------------------|-----------|------------------------|-----------|
| 188         | 3 kg Courier post bag (White) 300x190x95mm | 0.66      | 0.99                   | $0.33     |


Item with the highest gross profit, _**Air cushion machine (Blue)**_ with __$940.01__ profit.

```sql
SELECT TOP 1 WITH TIES StockItemName, UnitPrice, RecommendedRetailPrice, 
    FORMAT(RecommendedRetailPrice-UnitPrice, 'C') [MaxProfit]
FROM Warehouse.StockItems
ORDER BY MaxProfit DESC
```

| StockItemID | StockItemName              | UnitPrice | RecommendedRetailPrice | MaxProfit |
|-------------|----------------------------|-----------|------------------------|-----------|
| 215         | Air cushion machine (Blue) | 1899.00   | 2839.01                | $940.01   |


Median gross profit across all items in the warehouse _$8.91_. That is item _**"The Gu" red shirt XML tag t-shirt (White) 6XL**_.

```sql
DECLARE @c BIGINT = (SELECT COUNT(*) FROM Warehouse.StockItems)

SELECT StockItemID, StockItemName, FORMAT(AVG(1.0 * Profit), 'C') [Median Gross Profit]
FROM (
    SELECT StockItemID, StockItemName, RecommendedRetailPrice-UnitPrice [Profit] FROM Warehouse.StockItems
     ORDER BY Profit
     OFFSET (@c - 1) / 2 ROWS
     FETCH NEXT 1 + (1 - @c % 2) ROWS ONLY
) AS X
GROUP BY StockItemID, StockItemName
```

| StockItemID | StockItemName                                  | Median Gross Profit |
|-------------|------------------------------------------------|---------------------|
| 87          | "The Gu" red shirt XML tag t-shirt (White) 6XL | $8.91               |
