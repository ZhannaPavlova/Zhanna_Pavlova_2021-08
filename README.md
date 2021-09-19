# Zhanna_Pavlova_2021-08
MS SQL Server Developer

Задание №2 по CTE

USE WideWorldImporters

--Сотрудники, которые не сделали ни одной продажи 04 июля 2015--

SELECT
		 [PersonID]
		,[FullName]
FROM [Application].[People] 
WHERE [PersonID] not in	(SELECT
						[SalespersonPersonID]
						FROM [Sales].[Invoices]
						WHERE [InvoiceDate] = '2015-06-04'
						)
ORDER BY [PersonID]


  -- Товары с минимальной ценой--

SELECT [StockItemID]
      ,[StockItemName]
	  ,[UnitPrice]
FROM [Warehouse].[StockItems]
WHERE [UnitPrice] = (SELECT MIN (UnitPrice)
					FROM [Warehouse].[StockItems]);

SELECT [StockItemID]
      ,[StockItemName]
	  ,[UnitPrice]
FROM [Warehouse].[StockItems]
WHERE [UnitPrice]<= ALL (SELECT [UnitPrice]
							FROM [Warehouse].[StockItems]);

---top 5 максимальных платежей--

SELECT TOP 5
		 c.[CustomerName]
		,[TransactionAmount] as Amount
FROM [WideWorldImporters].[Sales].[CustomerTransactions] tr
INNER JOIN [WideWorldImporters].[Sales].[Customers] c on c.[CustomerID] = tr.[CustomerID]
Order by [TransactionAmount] desc;

 WITH cte_transactions  AS  (
SELECT TOP 5
		 [CustomerID]
		,[TransactionAmount] as Amount
FROM [WideWorldImporters].[Sales].[CustomerTransactions]
Order by [TransactionAmount] desc) 


SELECT CustomerName
,Amount
FROM cte_transactions
 JOIN [WideWorldImporters].[Sales].[Customers] c on c.[CustomerID] = cte_transactions.[CustomerID];

 ---ТОР 3 самых дорогих товара

 WITH Items as
 (select TOP 3 nm.[UnitPrice] as price
			,[OrderID]
			,sord.[StockItemID] as ItemID
			,nm.[StockItemName] as ItemName
	From [Sales].[OrderLines] sord
	  join [Warehouse].[StockItems] nm On nm.StockItemID =sord.StockItemID
	 	GROUP BY 	[OrderID]
					,sord.[StockItemID]
					,nm.[StockItemName]
Order by nm.[UnitPrice] desc )

,PackedbyPerson as
(select [OrderID]
		,[CustomerID]
		,[PackedByPersonID] as PackedbyPerson
	  From [Sales].[Invoices] pp)

,Customer as
(select [CustomerID]
      ,[CustomerName] as CustomerName
	  ,[DeliveryCityID]
	  From [Sales].[Customers])

,Cities as 
(Select  [CityID]
		,[CityName] as City
	  From [Application].[Cities])

SELECT ItemID
		,ItemName
		,price
		,CustomerName
		,City
		,PackedbyPerson
	From Items s
		join PackedbyPerson pp ON pp.OrderID = s.OrderID
		join Customer c ON c.CustomerID = pp.CustomerID
		join Cities ci ON ci.CityID = c.DeliveryCityID
