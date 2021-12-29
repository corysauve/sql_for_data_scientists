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

```
SELECT * FROM product 
  LEFT JOIN product_category 
    ON product.id = product_category.id
```
