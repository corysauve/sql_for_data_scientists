# Chapter 7 - Window Functions and Subqueries 

* **Window Functions**
    * Return group aggregate calculations alongside individual row-level information for items in a group
    * Also used to rank or sort values within each partition 
    * Operate across multiple records but those records don;t have to be grouped to the output 
    * Allows for the ability to put values from one row of data into context compared to a group of rows, or partition
    
* Window function uses 
    * Return information from a past record alongside the most recent detail record related to an entity 

## `ROW NUMBER`

* Allows you to rank rows by a value 
* Example: ranking products per vendor by price 

```sql
SELECT 
	vendor_id, 
  market_date, 
  product_id, 
  original_price,
  ROW_NUMBER() OVER (PARTITION BY vendor_id ORDER BY original_price DESC) AS price_rank 
FROM farmers_market.vendor_inventory ORDER BY vendor_id, original_price DESC
```

* Can use a **subquery** to return only the record of the highest priced item per vendor 
    * Sorting the records within each partition (vendor_id) and then filtering to a value (price_rank)
    
```sql
SELECT * FROM 
(
	SELECT 
  		vendor_id, 
  		market_date, 
  		product_id, 
  		original_price,
  		ROW_NUMBER() OVER (PARTITION BY vendor_id ORDER BY original_price DESC) AS price_rank 
		FROM farmers_market.vendor_inventory
		ORDER BY vendor_id) x 
    WHERE x.price_rank = 1
```

## `RANK` and `DENSE RANK`

* `RANK`: gives rows with the same value the same ranking 

```sql
SELECT 
	vendor_id, 
  market_date, 
  product_id, 
  original_price,
  RANK() OVER (PARTITION BY vendor_id ORDER BY original_price DESC) AS price_rank 
FROM farmers_market.vendor_inventory ORDER BY vendor_id, original_price DESC
```

* `DENSE RANK` does not skip ties
* `ROW_NUMBER` if you want each row to have own number, without ties 

## `NTILE`

* Can specify a number inside the parentheses (`NTILE (n)`) to indicate that you wan tthe results broken up into *n* blocks 
* To get the top tenth: 

```sql
SELECT 
	vendor_id, 
  market_date, 
  product_id, 
  original_price,
  NTILE(10) OVER (ORDER BY original_price DESC) AS price_ntile
FROM farmers_market.vendor_inventory 
ORDER BY original_price DESC
```

* If groups cannot be divided equally = some groups will end up with more rows than other 

## Aggregate Window Functions 

* One use is to compare each row's value to the aggregate value for a grouped category 
* Example: Want to know which of your products were above the average price per product on each market date? 

```sql
SELECT 
	vendor_id, 
  market_date, 
  product_id, 
  original_price, 
  AVG(original_price) OVER (PARTITION BY market_date ORDER BY market_date) AS average_cost_product_by_market_date
FROM farmers_market.vendor_inventory
```

* compare the original price per item to the average cost of products on each market date calculated from the window function 

```sql
SELECT * FROM 
(
	SELECT 
		vendor_id, 
    market_date, 
    product_id, 
    original_price, 
    ROUND(AVG(original_price) OVER (PARTITION BY market_date ORDER BY market_date), 2) AS average_cost_product_by_market_date
	FROM farmers_market.vendor_inventory
) x
WHERE x.vendor_id = 8
	AND x.original_price > x.average_cost_product_by_market_date 
ORDER BY x.market_date, x.original_price DESC
```
* count how many items are in each partition 

```sql
SELECT 
	vendor_id, 
  market_date, 
  product_id, 
  original_price, 
  COUNT(product_id) OVER (PARTITION BY market_date, vendor_id) vendor_product_count_per_market_date
FROM farmers_market.vendor_inventory
ORDER BY vendor_id, market_date, original_price DESC
```

* Aggregate window functions to calculate running totals 

```sql
SELECT 
	customer_id,
  market_date, 
  vendor_id, 
  product_id, 
  quantity * cost_to_customer_per_qty AS price,
	SUM(quantity * cost_to_customer_per_qty) OVER (ORDER BY market_date, transaction_time, customer_id, product_id) AS running_total_purchases 
FROM farmers_market.customer_purchases
```

* Same running total as above, but partitioned by customer_id 

```sql
SELECT 
	customer_id,
  market_date, 
  vendor_id, 
  product_id, 
  quantity * cost_to_customer_per_qty AS price,
	SUM(quantity * cost_to_customer_per_qty) OVER (PARTITION BY customer_id ORDER BY market_date, transaction_time, product_id) AS customer_spend_running_total
FROM farmers_market.customer_purchases
```

* Removing the ORDER BY clause calculates the total spend by the customer and displays the summary total on every row 

```sql
SELECT 
	customer_id,
  market_date, 
  vendor_id, 
  product_id, 
  quantity * cost_to_customer_per_qty AS price,
	ROUND(SUM(quantity * cost_to_customer_per_qty) OVER (PARTITION BY customer_id), 2) AS customer_spend_running_total
FROM farmers_market.customer_purchases
```
## `LAG` and `LEAD`

* `LAG`: Retrieves data from a row that is a selected number of rows back in the dataset 

```sql
SELECT 
	market_date, 
  vendor_id, 
  booth_number, 
  LAG(booth_number, 1) OVER (PARTITION BY vendor_id ORDER BY market_date, vendor_id) AS previous_booth_number 
FROM farmers_market.vendor_booth_assignments 
ORDER BY market_date, vendor_id, booth_number 
```

* Example: Find where booths were swapped on a specific date 

```sql
SELECT * FROM
(
	SELECT 
		market_date, 
		vendor_id, 
		booth_number, 
		LAG(booth_number, 1) OVER (PARTITION BY vendor_id ORDER BY market_date, vendor_id) AS previous_booth_number 
	FROM farmers_market.vendor_booth_assignments 
	ORDER BY market_date, vendor_id, booth_number 
) x 
WHERE x.market_date = '2019-04-10'
	AND (x.booth_number <> x.previous_booth_number OR x.previous_booth_number IS NULL)
```

* Total sales on each market date are higher or lower than they were on the previous market date 

1. Total sales per market date 
```sql
SELECT 
	market_date, 
  SUM(quantity * cost_to_customer_per_qty) AS market_date_total_sales
FROM farmers_market.customer_purchases
GROUP BY market_date 
ORDER BY market_date 
```

2. Add LAG() window function to output previous market date's calculated sum on each row 

```sql
SELECT 
	market_date, 
  SUM(quantity * cost_to_customer_per_qty) AS market_date_total_sales,
  LAG(SUM(quantity * cost_to_customer_per_qty), 1) OVER (ORDER BY market_date) AS previous_market_date_total_sales 
FROM farmers_market.customer_purchases
GROUP BY market_date 
ORDER BY market_date 
```

* `LEAD` works the same way as `LAG`, but gets the value from the next row instead of hte previous row 

## Exercises 

1. 

2.

3.





































