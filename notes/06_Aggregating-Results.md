# Chapter 6 - Aggregating Results for Analysis 

## Basic syntax for `GROUP BY` and `HAVING`

SELECT [columns to return]
FROM [table]
WHERE [conditional filter statements]
GROUP BY [columns to group on]
HAVING [conditional filter statements that are run after grouping]
ORDER BY [columns to sort on]

## `GROUP BY`

* keywords are followed by comma-separated list of column names that indicate how you want 
to summarize the query results 

Without grouping: 

```
SELECT 
  market_date, 
  customer_id
FROM farmers_market.customer_purchases
ORDER BY market_date, customer_id 
```

With grouping (this returns one row per customer per market date)

```
SELECT 
  market_date, 
  customer_id
FROM farmers_market.customer_purchases
GROUP BY market_date, customer_id
ORDER BY market_date, customer_id 
```

## Display group summaries 

* Can add aggregate functions (`SUM`, `COUNT`) to return summaries of the customer_purchases 
date per group 

```
SELECT 
  market_date,
  customer_id, 
  COUNT(*) AS items_purchased
FROM farmers_market.customer_purchases 
GROUP BY market_date, customer_id
ORDER BY market_date, customer_id
LIMIT 10
```

* Add quantities 

```
SELECT 
  market_date,
  customer_id, 
  SUM(quantity) AS items_purchased 
FROM farmers_market.customer_purchases 
GROUP BY market_date, customer_id
ORDER BY market_date, customer_id 
LIMIT 10 
```

* Want to know how many different kinds of items were purchased by each customer 

```
SELECT 
  market_date, 
  customer_id, 
  COUNT(DISTINCT product_id) AS different_products_purchased 
FROM farmers_market.customer_purchases c 
GROUP BY market_date, customer_id 
ORDER BY market_date, customer_id
LIMIT 10
```

* Combine summaries into single query (summarizes per market date per customer ID)

```
SELECT 
	market_date, 
  customer_id, 
  SUM(quantity) AS items_purchased,
  COUNT(DISTINCT product_id) AS different_products_purchased
FROM farmers_market.customer_purchases 
GROUP BY market_date, customer_id 
ORDER BY market_date, customer_id 
LIMIT 10
```

## Performing calculations inside aggregate functions 

* Calculated at the row level prior to summarization 

```
SELECT 
	market_date, 
  customer_id, 
  vendor_id, 
  quantity * cost_to_customer_per_qty AS price 
FROM farmers_market.customer_purchases
WHERE 
	customer_id = 3
ORDER BY market_date, vendor_id
```

* Want to know how much money this customer spent on each market_date, regardless of item or vendor

```
SELECT 
	market_date, 
  customer_id, 
  SUM(quantity * cost_to_customer_per_qty) AS total_spent 
FROM farmers_market.customer_purchases
WHERE 
	customer_id = 3
GROUP BY market_date
ORDER BY market_date
```

* How much customer had spent at each vendor, regardless of date 

```
SELECT 
	customer_id,
  vendor_id,
  SUM(quantity * cost_to_customer_per_qty) AS total_price
FROM farmers_market.customer_purchases
WHERE 
	customer_id = 3
GROUP BY customer_id, vendor_id
ORDER BY customer_id, vendor_id
```

