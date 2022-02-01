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

* This may initially appear like the `WHERE` clause is selecting all integers above 0. However, the filtering clause is applied to the *right* table. The result of this query omits all customers without purchases. This behavior is making the LEFT JOIN actually behave like a INNER JOIN. 

* If you use a LEFT JOIN to return all rows from the left table, make sure to not filter out any fields from the right table without also allowing NULL results on the right side 1

One solution would be to add an `OR` statement to allow NULL values: 

```
SELECT * 
FROM customer AS c
LEFT JOIN customer_purchases as cp 
  ON c.customer_id = cp.customer_id
WHERE cp.customer_id > 0 OR cp.customer_id IS NULL
```

## JOINs with more than two tables 

* Join booth, vendor_booth_assignments, and vendor together; include all booths 

```
SELECT 
  b.booth_number,
  b.booth_type, 
  vba.market_date, 
  v.vendor_id, 
  v.vendor_name, 
  v.vendor_type
FROM booth AS b
  LEFT JOIN vendor_booth_assignments AS vba ON b.booth_number = vba.booth_number 
  LEFT JOIN vendor as v on v.vendor_id = vba.vendor_id
ORDER BY b.booth_number, vba.market_date
```

## Exercises 

1. Write a query that INNER JOINS the vendor table to the vendor_booth_assignments table on the vendor_id field they both have in common, and sorts the results by vendor_name, then market_date

```
SELECT * 
FROM vendor AS v 
	INNER JOIN vendor_booth_assignments AS vba 
	  ON v.vendor_id = vba.vendor_id
ORDER BY v.vendor_name, vba.market_date
```

2. IS it possible to write a query that produces an output identical to the output of the following query, but using a LEFT JOIN instead of a RIGHT JOIN?

```
SELECT * 
FROM customer AS c
RIGHT JOIN customer_purchases AS cp
  ON c.customer_id = cp.customer_id
```

Answer:

```
SELECT c.*, cp.* 
FROM customer_purchases AS cp 
LEFT JOIN customer AS c 
	ON cp.customer_id = c.customer_id
```
