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

* List of every customer and how much they have ever spent at the market 

```
SELECT 
	customer_id, 
  sum(quantity * cost_to_customer_per_qty) AS total_spent 
FROM farmers_market.customer_purchases 
GROUP BY customer_id
ORDER BY customer_id
```

* Can aggregate on joined tables as well 

```
SELECT 
	c.customer_first_name, 
  c.customer_last_name, 
  cp.customer_id, 
  v.vendor_name,
  cp.quantity * cp.cost_to_customer_per_qty AS price 
FROM farmers_market.customer c 
	LEFT JOIN farmers_market.customer_purchases cp 
		ON c.customer_id = cp.customer_id 
	LEFT JOIN farmers_market.vendor v 
		ON cp.vendor_id = v.vendor_id
WHERE 
	cp.customer_id = 3
ORDER BY cp.customer_id, cp.vendor_id
```

* Summarize at the level of one row per customer per vendor 

```
SELECT 
	c.customer_first_name, 
  c.customer_last_name, 
  cp.customer_id, 
  v.vendor_name,
  cp.vendor_id,
  ROUND(SUM(cp.quantity * cp.cost_to_customer_per_qty), 2) AS total_spent 
FROM farmers_market.customer c 
	LEFT JOIN farmers_market.customer_purchases cp 
		ON c.customer_id = cp.customer_id 
	LEFT JOIN farmers_market.vendor v 
		ON cp.vendor_id = v.vendor_id
WHERE 
	cp.customer_id = 3
GROUP BY 
	c.customer_first_name, 
    c.customer_last_name, 
    cp.customer_id, 
    v.vendor_name, 
    cp.vendor_id
ORDER BY cp.customer_id, cp.vendor_id
```

* Can also filter for a single vendor 
    * Removing the `WHERE` clause would return one row for every customer-vendor pair. 

```
SELECT 
	c.customer_first_name, 
  c.customer_last_name, 
  cp.customer_id, 
  v.vendor_name,
  cp.vendor_id,
  ROUND(SUM(quantity * cost_to_customer_per_qty), 2) AS total_spent 
FROM farmers_market.customer c 
	LEFT JOIN farmers_market.customer_purchases cp 
		ON c.customer_id = cp.customer_id 
	LEFT JOIN farmers_market.vendor v 
		ON cp.vendor_id = v.vendor_id
WHERE 
	cp.vendor_id = 4
GROUP BY 
	c.customer_first_name, 
    c.customer_last_name, 
    cp.customer_id, 
    v.vendor_name, 
    cp.vendor_id
ORDER BY cp.customer_id, cp.vendor_id
```

## `MIN` and `MAX`

* Take a look at vendor_inventory table:

```
SELECT * 
FROM farmers_market.vendor_inventory
ORDER BY original_price 
LIMIT 10
```

* Get the least and most expensive item prices in table 

```
SELECT 
	MIN(original_price) AS minimum_price,
  MAX(original_price) AS maximum_price
FROM farmers_market.vendor_inventory
ORDER BY original_price 
```

* What is the least/most expensive product per product category 

```
SELECT
	product_category.product_category_name, 
  product.product_category_id,
	MIN(vendor_inventory.original_price) AS minimum_price,
  MAX(vendor_inventory.original_price) AS maximum_price
FROM farmers_market.vendor_inventory
	INNER JOIN farmers_market.product
		ON vendor_inventory.product_id = product.product_id
	INNER JOIN farmers_market.product_category
		ON product.product_category_id = product_category.product_category_id
GROUP BY product_category.product_category_name, product.product_category_id
```

## `COUNT` and `COUNT DISTINCT`

Question = How many products were for sale on each market date, or how many different 
products each vendor offered?

```
SELECT 
	market_date,
  COUNT(product_id) AS product_count 
FROM farmers_market.vendor_inventory
GROUP BY market_date 
ORDER BY market_date 
```

Question = How many different products each vendor brought to market during a date range 


```
SELECT 
	vendor_id,
  COUNT(DISTINCT product_id) AS different_products_offered 
FROM farmers_market.vendor_inventory
WHERE market_date BETWEEN '2019-04-03' AND '2019-04-17'
GROUP BY vendor_id
ORDER BY vendor_id
```

## Average 

* What if we want the average original price of the product per vendor 

```
SELECT 
	vendor_id,
  COUNT(DISTINCT product_id) AS different_products_offered, 
  AVG(original_price) AS average_original_price
FROM farmers_market.vendor_inventory
WHERE market_date BETWEEN '2019-04-03' AND '2019-04-17'
GROUP BY vendor_id
ORDER BY vendor_id
```

* The above result isn't really what we're after. It would probably make more sense to 
multiply the quantity of each type of item by the price of that item. This would occur per row, 
and then sum that up and divide by the total quantity of items

```
SELECT 
	vendor_id, 
  COUNT(DISTINCT product_id) AS different_products_offered, 
  SUM(quantity * original_price) AS value_of_inventory,
  SUM(quantity) AS inventory_item_count,
  ROUND(SUM(quantity * original_price) / SUM(quantity), 2) AS average_item_price
FROM farmers_market.vendor_inventory
WHERE market_date BETWEEN '2019-04-03' AND '2019-04-17'
GROUP BY vendor_id 
ORDER BY vendor_id
```

* Note that `quantity * original_price` inside the aggregate function is performed 
per row, then the aggregate SUMs are calculate, then the division of one SUM into the other 
to determine the "average item price" (i.e., we're performing operations both before and after the GROUP BY summarization occurs)

## Filter with `HAVING`

* Allows you to filter after the aggregation 

```
SELECT 
	vendor_id, 
  COUNT(DISTINCT product_id) AS different_products_offered, 
  SUM(quantity * original_price) AS value_of_inventory,
  SUM(quantity) AS inventory_item_count,
  ROUND(SUM(quantity * original_price) / SUM(quantity), 2) AS average_item_price
FROM farmers_market.vendor_inventory
WHERE market_date BETWEEN '2019-04-03' AND '2019-04-17'
GROUP BY vendor_id
HAVING inventory_item_count <= 180 
ORDER BY vendor_id
```

* TIP: If you group by all fields that are supposed to be distinct and add a HAVING 
clause that filters to aggregated rows with a `COUNT(*) > 1`, any result returned 
indicates that there is more than one row with a supposedly "unique" combination of values = 
there is duplicated values in either the data or query

## `CASE` Statements Inside Aggregate Functions 

* Can use `CASE` to specify which type of item quantities to add together using each SUM aggregate function

Starting JOIN

```
SELECT 
	cp.market_date, 
  cp.vendor_id, 
  cp.customer_id, 
  cp.product_id, 
  cp.quantity, 
  p.product_name, 
  p.product_size, 
  p.product_qty_type
FROM farmers_market.customer_purchases AS cp 
	INNER JOIN farmers_market.product AS p
		ON cp.product_id = p.product_id
```

Second step 

```
SELECT 
	cp.market_date, 
  cp.vendor_id, 
  cp.customer_id, 
  cp.product_id,
  CASE WHEN product_qty_type = "unit" THEN quantity ELSE 0 END AS quantity_units, 
  CASE WHEN product_qty_type = "lbs" THEN quantity ELSE 0 END AS quantity_lbs, 
  CASE WHEN product_qty_type NOT IN ("unit", "lbs") THEN quantity ELSE 0 END quantity_other, 
  p.product_qty_type
FROM farmers_market.customer_purchases AS cp 
	INNER JOIN farmers_market.product AS p
		ON cp.product_id = p.product_id
```

Last stop is to add in CASE statements 

```
SELECT 
	cp.market_date, 
  cp.customer_id, 
  SUM(CASE WHEN product_qty_type = "unit" THEN quantity ELSE 0 END) AS qty_units_purchased, 
  SUM(CASE WHEN product_qty_type = "lbs" THEN quantity ELSE 0 END) AS qty_lbs_purchased, 
  SUM(CASE WHEN product_qty_type NOT IN ("unit", "lbs") THEN quantity ELSE 0 END) qty_other_purchased
FROM farmers_market.customer_purchases AS cp 
	INNER JOIN farmers_market.product AS p
		ON cp.product_id = p.product_id
GROUP BY market_date, customer_id 
ORDER BY market_date, customer_id
```

## Exercises 

1. Write a query that determines how many times each vendor has rented a booth at the farmer's market (i.e., count the vendor booth assignments per vendor_id)

```
SELECT 
	vendor_id, 
  COUNT(*)
FROM farmers_market.vendor_booth_assignments
GROUP BY vendor_id
ORDER BY vendor_id
```

2. Write a query that displays the product category name, product_name, earliest date available, and latest date available for every product in "Fresh Fruits & Vegetables".

```
SELECT 
	pc.product_category_name, 
  p.product_name, 
  MIN(v.market_date) AS earliest_available, 
  MAX(v.market_date) AS last_available 
FROM farmers_market.vendor_inventory AS v 
	INNER JOIN farmers_market.product AS p
		ON v.product_id = p.product_id
	INNER JOIN farmers_market.product_category AS pc
		ON p.product_category_id = pc.product_category_id
WHERE product_category_name = "Fresh Fruits & Vegetables"
```

3. Everyone who has ever spent more than $50 gets a sticker. Write a query that generates a list of customers, sorted by last name, first name.

```
SELECT
	c.customer_id,
	c.customer_first_name, 
  c.customer_last_name,
  cp.customer_id, 
  SUM(cp.quantity * cp.cost_to_customer_per_qty) AS total_spent
FROM farmers_market.customer as c
	LEFT JOIN farmers_market.customer_purchases as cp
		ON c.customer_id = cp.customer_id
GROUP BY cp.customer_id, c.customer_first_name, c.customer_last_name
HAVING total_spent > 50
ORDER BY customer_last_name, customer_first_name 
```

