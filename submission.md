# Challenge 1
# The leadership team has asked us to graph total monthly sales over time. Write a query that returns the data we need to complete this request.

```sql
SELECT DATENAME(MONTH, DATEADD(M, MONTH(TransactionDate), - 1)) [Month],
   YEAR(TransactionDate) [Year],
   SUM(TransactionAmount) [Total Price]
FROM Sales.CustomerTransactions
WHERE TransactionTypeID=1
GROUP BY YEAR(TransactionDate), MONTH(TransactionDate)
ORDER BY YEAR(TransactionDate), MONTH(TransactionDate)
```
![alt text](https://github.com/JainSahit/tea-sql-challenge/blob/main/images/Screen%20Shot%202021-06-13%20at%205.42.06%20PM.png?raw=true)
# Challenge 2
# What is the fastest growing customer category in Q1 2016 (compared to same quarter sales in the previous year)? What is the growth rate?

There are two possible ways of tracking quarterly customer category growth. First, using quarterly order count, and Second, using quarterly revenue. 
Category "Customer Store" had the most growth in Q1 2016 according to both the order count and revenue. 

There is a **23.62%** increase in the number of orders in Q1 2016 compared to Q1 2015, while a **18.88%** increase in quarterly revenues.

![alt text](https://github.com/JainSahit/tea-sql-challenge/blob/main/images/Screen%20Shot%202021-06-13%20at%205.57.37%20PM.png?raw=true)

# Challenge 3
# Write a query to return the list of suppliers that WWI has purchased from, along with # of invoices paid, # of invoices still outstanding, and average invoice amount.

| SupplierID | SupplierName             | # Invoices Paid | # Invoices Outstanding | AVG Invoice Amount |
|------------|--------------------------|-----------------|------------------------|--------------------|
| 1          | A Datum Corporation      | 5               | 0                      | 5505.060000        |
| 2          | Contoso, Ltd.            | 1               | 0                      | 360.530000         |
| 4          | Fabrikam, Inc.           | 1053            | 1                      | 739825.574297      |
| 5          | Graphic Design Institute | 13              | 0                      | 574.034615         |
| 7          | Litware, Inc.            | 983             | 1                      | 311009.072621      |
| 10         | Northwind Electric Cars  | 10              | 0                      | 9063.900000        |
| 12         | The Phone Company        | 5               | 0                      | 11688.602000       |

```sql
SELECT S.SupplierID, SupplierName, 
    COUNT(TransactionAmount) - COUNT(NULLIF(OutstandingBalance,0)) [# Invoices Paid], 
    COUNT(NULLIF(OutstandingBalance,0)) [# Invoices Outstanding],
    AVG(TransactionAmount) [AVG Invoice Amount]
FROM Purchasing.Suppliers AS S
JOIN Purchasing.SupplierTransactions 
ON S.SupplierID=Purchasing.SupplierTransactions.SupplierID
WHERE TransactionTypeID=5
GROUP BY S.SupplierID, S.SupplierName
ORDER BY S.SupplierID
```

# Challenge 4
# Using "unit price" and "recommended retail price", which item in the warehouse has the lowest gross profit amount? Which item has the highest? What is the median gross profit across all items in the warehouse?

Item with the lowest gross profit, _**3 kg Courier post bag (White) 300x190x95mm**_ with just __$0.33__ profit.
```sql
SELECT TOP 1 WITH TIES StockItemName, UnitPrice, RecommendedRetailPrice, 
    RecommendedRetailPrice-UnitPrice [MinProfit]
FROM Warehouse.StockItems
ORDER BY MinProfit
```
| StockItemID | StockItemName                              | UnitPrice | RecommendedRetailPrice | MinProfit |
|-------------|--------------------------------------------|-----------|------------------------|-----------|
| 188         | 3 kg Courier post bag (White) 300x190x95mm | 0.66      | 0.99                   | 0.33      |


Item with the highest gross profit, _**Air cushion machine (Blue)**_ with __$940.01__ profit.
```sql
SELECT TOP 1 WITH TIES StockItemName, UnitPrice, RecommendedRetailPrice, 
    RecommendedRetailPrice-UnitPrice [MaxProfit]
FROM Warehouse.StockItems
ORDER BY MaxProfit DESC
```
| StockItemID | StockItemName              | UnitPrice | RecommendedRetailPrice | MaxProfit |
|-------------|----------------------------|-----------|------------------------|-----------|
| 215         | Air cushion machine (Blue) | 1899.00   | 2839.01                | 940.01    |


Median gross profit across all items in the warehouse _$8.91_. That is item _**"The Gu" red shirt XML tag t-shirt (White) 6XL**_.
```sql
DECLARE @c BIGINT = (SELECT COUNT(*) FROM Warehouse.StockItems)

SELECT StockItemID, StockItemName, AVG(1.0 * Profit) [Median Gross Profit]
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
| 87          | "The Gu" red shirt XML tag t-shirt (White) 6XL | 8.910000            |
