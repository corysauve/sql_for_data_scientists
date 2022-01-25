# Chapter 5 -SQL Joins 

## `LEFT JOIN`

* Tells query to pull all records from the tbl on the left side and only the 
matching records from the table on the right side 
* `NULL` us inserted where missing values on the right 

* Basic Scaffolding 

```
SELECT [columns to return]
FROM [left tbl]
[JOIN TYPE] [right tbl]
ON [left tbl].[field to match] = [right tbl].[field]
```

### Join all fields from both tables 

```
SELECT * FROM product 
	LEFT JOIN product_category
		ON product.product_category_id = product_category.product_category_id```
```
* Since we used `*` to join both tables, there are two columns for `product_category_id`. 
If we don't want this, two options: 

    * Specify the list of fields to be returned and only include the product_category_id 
    from one of the tables 
    * Alias the column names to indicate which table each came from 

```
SELECT 
	product.product_id, 
    product.product_name,
    product.product_category_id AS product_prod_cat_id, 
    product_category.product_category_id AS category_prod_cat_id, 
    product_category.product_category_name
FROM product 
	LEFT JOIN product_category 
		ON product.product_category_id = product_category.product_category_id
```

* You can also use table aliasing to shorten the reference (the `AS` keyword is optional)

```
SELECT 
	p.product_id, 
    p.product_name,
    pc.product_category_id,
    pc.product_category_name
FROM product AS p
	LEFT JOIN product_category AS pc
		ON p.product_category_id = pc.product_category_id
ORDER BY pc.product_category_name, p.product_name
```

## `RIGHT JOIN`

* All records from the right table are returned, but only matching records from the left table

```
SELECT * FROM product 
	RIGHT JOIN product_category
		ON product.product_category_id = product_category.product_category_id
```

## `INNER JOIN`

* Return only rows from the right table and left table where values in the specified fields have matches in both tables 

Left join example for comparison 
```
SELECT * 
FROM customer AS c
	LEFT JOIN customer_purchases AS cp
		ON c.customer_id = cp.customer_id
```

Right join

```
SELECT * 
FROM customer AS c
RIGHT JOIN customer_purchases AS cp
	ON c.customer_id = cp.customer_id
```

Inner join

```
SELECT * 
FROM customer AS c
INNER JOIN customer_purchases AS cp
	ON c.customer_id = cp.customer_id
```
## Common pitfall when filtering joined data 

Look at this query 

```
SELECT * 
FROM customer AS c
LEFT JOIN customer_purchases as cp 
  ON c.customer_id = cp.customer_id
WHERE cp.customer_id > 0
```

This may initally appear like the `WHERE` clause 


















