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

PICK UP ON PG 104






















