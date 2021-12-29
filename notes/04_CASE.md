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
