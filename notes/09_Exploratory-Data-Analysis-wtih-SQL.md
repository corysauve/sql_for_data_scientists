# Chapter 9 - Exploratory Data Analysis with SQL 

## Exploring the products table 

* Some databases have a `DESCRIBE` function to see table metadata 

```
DESCRIBE farmers_market.booth
```

* Looking at product table 

```
SELECT * FROM farmers_market.product
LIMIT 10
```

* Check to see if the `product_id` is the primary key (no results = unique)
    * Can say that the table has *graularity* of one row per product 

```
SELECT product_id, count(*)
FROM farmers_market.product
GROUP BY product_id 
HAVING count(*) > 1
```

* Looking at the product_category table 
```
SELECT * FROM farmers_market.product_category
```

* How many different products are there in the catalog-like product metadata table 

```
SELECT count(*) FROM farmers_market.product
```

* How many products are there per product category?

```
SELECT 
	pc.product_category_id, 
  pc.product_category_name, 
  count(product_id) AS count_of_products 
FROM farmers_market.product_category AS pc 
LEFT JOIN farmers_market.product AS p 
	ON pc.product_category_id = p.product_category_id
GROUP BY pc.product_category_id
```

## Exploring possible column values 

* What is in the product_qty_table and how many different quantity types are there?

```
SELECT DISTINCT product_qty_type 
FROM farmers_market.product
```

* Now look at the vendor_inventory table 

```
SELECT * FROM farmers_market.vendor_inventory
```

* Check primary key
    * Granularity = one record per market date, vendor, and product 
```
SELECT market_date, vendor_id, product_id, count(*)
FROM farmers_market.vendor_inventory
GROUP BY market_date, vendor_id, product_id
HAVING count(*) > 1
```

* How far back does the data go?

```
SELECT min(market_date), max(market_date) 
FROM farmers_market.vendor_inventory
```

* How many different vendors are there, and when did they each staret selling at the market?

```
SELECT vendor_id, min(market_date), max(market_date)
FROM farmers_market.vendor_inventory
GROUP BY vendor_id 
ORDER BY min(market_date), max(market_date)
```
