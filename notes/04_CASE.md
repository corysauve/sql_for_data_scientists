# Chapter 4 - `CASE` Statements 

## `CASE`

* Used to create new columns based on existing columns 
    * Often described as *derived columns* or *calculated fields*
        * Similar to *feature engineering* in ML

### Basic scaffolding 

```
CASE 
  WHEN [first conditional statement]
    THEN [value or calculation]
  WHEN [second conditional statement]
    THEN [value of calculation]
  ELSE [value or calculation]
END
```

* `WHEN`s are evaluated in order, from top to bottom
* `ELSE` is optional 
    * If not included, `NULL` is added if `WHEN` conditions are not met 
* Always alias columns so resulting columns are readable 

```
SELECT 
  vendor_id, 
  vendor_name,
  vendor_type, 
  CASE 
    WHEN LOWER(vendor_type) LIKE '%fresh%'
      THEN 'Fresh Produce'
    ELSE 'Other'
  END AS vendor_type_condensed
FROM farmers_market.vendor
```

### Create binary flag 

```
SELECT market_date
  CASE 
    WHEN market_day = 'Saturday'
      THEN 1 
    ELSE 0
  END AS weekend_flag 
```

### Grouping or Binning Continuous values w/ CASE 

```
SELECT 
  quantity,
  cost_to_customer,
  price 
  CASE 
    WHEN quantity * cost_to_customer < 5.00
      THEN 'UNDER $5'
    WHEN quantity * cost_to_customer > 5.00 
      THEN 'Over $5'
    END AS price_bin
FROM ...
LIMIT ...
```
* values can be numeric (also could include both)

### Categorical encoding using `CASE`

```
SELECT 
  col_a,
  col_b,
  CASE 
    WHEN col_b = 'A' THEN 1 
    WHEN col_b = 'B' THEN 0
  END AS new_name
FROM ...
LIMIT ...
```

### One hot encoding 

```
SELECT 
  vendor_id, 
  vendor_name,
  vendor_type,
  CASE WHEN 
    vendor_type = 'Arts & Jewerly' 
      THEN 1
      ELSE 0
  END AS vendor_type_arts_jewerly,
  CASE WHEN vendor_type = 'Eggs & Meat'
    THEN 1
    ELSE 0
  END AS vendor_type_eggs_meats,
  CASE WHEN vendor_type = 'Fresh Focused'
    THEN 1 
    ELSE 0
  END AS vendor_type_fresh_focused, 
  CASE WHEN vendor_type = 'Fresh Variety: Veggies & More'
    THEN 1 
    ELSE 0
  END AS vendor_type_fresh_variety,
  CASE WHEN vendor_type = 'Prepared Foods'
    THEN 1
    ELSE 0
  END AS vendor_type_prepared
FROM farmers_market.vendor
```

## Exercises 

1. Products can be sold by the individual unit or by bulk measures like lbs or oz. 
Write a query that outputs the `product_id` and `product_name` columns from the `product`
table, and add a column called `prod_qty_types` is "unit", and otherwise displays the word "bulk".

```
SELECT 
	product_id,
  product_name,
  CASE 
		WHEN product_qty_type = 'unit'
			THEN 'unit'
		ELSE 'bulk'
	END AS prod_qty_type_condensed
FROM farmers_market.product;
```

2. We want to flag all of the different types of pepper products that are sold at 
the market. Add a column to the previous query called `pepper_flag` that outputs 
a 1 if the `product_name` contains the word "pepper" (regardless of capitalization), 
and otherwise outputs 0.

```
SELECT 
	product_id,
  product_name,
  CASE 
		WHEN product_qty_type = 'unit'
			THEN 'unit'
		ELSE 'bulk'
	END AS prod_qty_type_condensed, 
    CASE 
		WHEN product_name LIKE '%pepper%' 
			THEN 'pepper'
        ELSE 'bulk'
	END AS pepper_flag
FROM farmers_market.product;
```
3. Can you think of a situation when a pepper product might not get flagged as a pepper 
product using the code from the previous exercise? 

If the type of pepper doesn't have the word 'pepper' in it, such as serrano, jalapeno, etc. Or if the word 
pepper is just simply replaced with a synonym like 'chile'
