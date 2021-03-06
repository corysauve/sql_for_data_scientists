# Chapter 2 - The `SELECT` Statement 

* `SELECT` = SQL code that retrieves data from a database 

## Some uses 

* View data from set of cols in table 
* Combine data from multiple tables 
* Filter results from above, perform calcs, etc 
    
## Fundamental `SELECT` Syntax 

```
SELECT [columns to return]
FROM [schema.tbl]
WHERE [conditional filter statements]
GROUP BY [cols to group by]
HAVING [conditional filter statements after grouping]
ORDER BY [columns to sort by]
```

## Select all fields from table 

```
SELECT * FROM farmers_market.product
```

## `LIMIT` 

* Sets maximum number of rows to return 
    * always last line of query

```
SELECT * 
FROM farmers_market.product
LIMIT 5
```

* Specify which columns to return 

```
SELECT product_id, product_name
FROM farmers_market.product
LIMIT 5
```

## `ORDER BY`

* Sort output rows 
* Add `ASC` or `DESC`
    * In MySQL, `NULL` values appear first w/ `ASC`

```
SELECT product_id, product_name 
FROM farmers_market.product 
ORDER BY product_name DESC
LIMIT 5
```

## Inline Calculations 

### Multiply 2 columns together 

```
SELECT 
  col_a,
  col_b,
  col_a * col_b
FROM schema.tbl 
LIMIT 10
```

Can rename columns using `AS` (`AS` is technically optionally but helps for clarity)

```
SELECT 
  col_a,
  col_b,
  col_a * col_b AS the_best_name
FROM schema.tbl 
LIMIT 10
```

* Don't need to reference cols before you do an inline calculation 

### `ROUND`

```
SELECT 
  col_a,
  col_b,
  ROUND(col_c * col_a, 2) AS the_best_name
FROM schema.tbl
LIMIT 10
```

* You can also round with negative (-) numbers 
    * Will round from the left of the decimal 
    * e.g., `ROUND(1245, -2` = 1200

### Concatenating Strings w/ `CONCAT()`

```
SELECT 
  col_a,
  col_b,
  CONCAT(first_name, " ", last_name) AS name 
FROM schema.tbl 
ORDER BY last_name, first_name 
LIMIT 10
```

* Can use `UPPER()` and `LOWER` w/ `CONCAT()`

## Exercises 

1. Write a query that returns everything in the customer field.. 

```
SELECT * FROM customer;
```

2. Write a query that displays all of the columns and 10 rows from the customer 
table, sorted by first name. 

```
SELECT * FROM farmers_market.customer
ORDER BY customer_first_name 
LIMIT 10;
```

3. Write a query that lists all customer IDs and first names in the customer table, 
sorted by first_name.

```
SELECT
  customer_id, 
  customer_first_name 
FROM farmers_market.customer
ORDER BY customer_first_name
```
