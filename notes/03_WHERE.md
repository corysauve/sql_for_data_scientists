# Chapter 3 - The WHERE Clause

## `WHERE`

* List conditions that are used to determine which rows in the table should be included 
in result set (i.e, filtering)
* Include before `ORDER BY`, `LIMIT`, etc 

```
SELECT 
FROM 
WHERE customer_id = 1
ORDER BY
```

* Can include boolean operators (e.g., `AND`, `OR`, `AND NOT`)

```
WHERE id > 3 
  AND id <= 5
```

```
WHERE 
  product_id = 10
  OR (product_id > 3)
  AND (product_id < 8)
```

* Multi-column filtering 

```
WHERE 
  customer_id = 4
  AND vendor_id = 7
```

### `BETWEEN`

* If a value is within a specific range of values 

```
SELECT ...
FROM ...
WHERE 
  vendor_id = 7
  AND market_date BETWEEN '2019-03-02' AND '2019-03-16'
```

### `IN`

* Include list of comma-separated values to compare 

```
SELECT ... 
FROM ...
WHERE 
  customer_name IN ('Diaz', 'Edwards')
```

### `LIKE`

* Search for partially matched strings 
    * `%` is a wild card operator 

```
WHERE 
  customer_name LIKE `JER%` 
```

### `IS NULL`

* Find rows in databse where field is `NULL`

```
WHERE product_size IS NULL
```

### `TRIM`

* Remove whitespace before and after string 
* Can use with `IS NULL` to remove `NULL` and blanks 

```
WHERE 
  product_size IS NULL
  OR TRIM(product_size = ' ')
```

### Subqueries 

* Can filter using subqueries (query inside a query)
  * Allows you to filter to a list of values that were returned by another query 
  (i.e., dynamic list)
  
1. First filter  

    ```
    SELECT data, flag 
    FROM schema.db
    WHERE flag = 1
    ```

2. Apply above filter inside another query 

    ```
    SELECT 
      date, id, price 
    FROM schema.db
    WHERE 
      date IN 
      (
        SELECT date 
        FROM schema.db2
        WHERE flag = 1
      )
    LIMIT 5
    ```

## Exercises 

1. Refer to the data in Table 3.1. Write a query that returns all customer purchases 
of product IDs 4 and 9. 

```
SELECT * FROM farmers_market.product
WHERE 
	product_id = 4
    OR product_id = 9;
```

2. Refer to the data in Table 3.1. Write two queries, one using two conditions with 
an `AND` operator, and one using the BETWEEN operator, that will return all customer purchases 
made from vendors with vendor IDs between 8 and 10 (inclusive).

```
SELECT
	product_id, vendor_id, market_date, 
  customer_id, quantity, cost_to_customer_per_qty
FROM farmers_market.customer_purchases
WHERE 
	vendor_id >= 8 
  AND vendor_id <= 10;
```

```
SELECT
	product_id, vendor_id, market_date, 
  customer_id, quantity, cost_to_customer_per_qty
FROM farmers_market.customer_purchases
WHERE 
	vendor_id BETWEEN 8 and 10;
```

3. Can you think of two different ways to change the final query in the chapter 
so that it would return purchases from days when it wasn't raining? 

Change the market_rain_flag from 1 to 0

```
SELECT 
	market_date,
  customer_id, 
  vendor_id, 
  quantity * cost_to_customer_per_qty price 
FROM farmers_market.customer_purchases
WHERE 
	market_date IN 
		(
      SELECT market_date 
      FROM farmers_market.market_date_info 
      WHERE market_rain_flag = 0
		)
LIMIT 5;
```

Change IN to NOT IN 

```
SELECT 
	market_date,
  customer_id, 
  vendor_id, 
  quantity * cost_to_customer_per_qty price 
FROM farmers_market.customer_purchases
WHERE 
	market_date NOT IN 
		(
      SELECT market_date 
      FROM farmers_market.market_date_info 
      WHERE market_rain_flag = 1
		)
LIMIT 5;
```
