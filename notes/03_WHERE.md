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
