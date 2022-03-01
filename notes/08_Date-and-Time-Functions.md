# Chapter 8 - Data and Time Functions 

## Setting datetime field values 

* Create table with datetime column

```sql
CREATE TABLE farmers_market.datetime_demo AS 
(
	SELECT 
		market_date, 
    market_start_time, 
    market_end_time, 
    STR_TO_DATE(CONCAT(market_date, ' ', market_start_time), '%Y-%m-%d %h:%i %p') AS market_start_datetime,
    STR_TO_DATE(CONCAT(market_date, ' ', market_end_time), '%Y-%m-%d %h:%i %p') AS market_end_datetime
	FROM farmers_market.market_date_info
)
```

## `EXTRACT` and `DATE_PART`

* Functions to extract datetime data types vary by database system: 
    * MySQL = `EXTRACT` 
    * Redshift = `DATE_PART` 
    * Oracle and SQL Server = `DATEPART` 

* Other MySQL datetime functions:
    * `DATE` 
    * `TIME` 

* 5 different datetime parts that can be extracted with `EXTRACT`

```sql
SELECT 
	market_start_datetime, 
	EXTRACT(DAY FROM market_start_datetime) AS mktsrt_day,
  EXTRACT(MONTH FROM market_start_datetime) AS mktsrt_month, 
  EXTRACT(YEAR FROM market_start_datetime) AS mktsrt_year, 
  EXTRACT(HOUR FROM market_start_datetime) AS mktsrt_hour, 
  EXTRACT(MINUTE FROM market_start_datetime) AS mktsrt_minute
FROM farmers_market.datetime_demo
WHERE market_start_datetime = '2019-03-02 08:00:00'
```
* Select the entire date and time elements from datetime field 

```sql
SELECT 
	market_start_datetime, 
	DATE(market_start_datetime) AS mktsrt_date, 
  TIME(market_start_datetime) AS mktsrt_time
FROM farmers_market.datetime_demo 
WHERE market_start_datetime = '2019-03-02 08:00:00'
```

## `DATE_ADD` and `DAT_SUB`

* Able to do date calculations with datetime columns 

* How many sales occurred within the first 30 minutes of the market's opening 

```sql
SELECT
	market_start_datetime, 
  DATE_ADD(market_start_datetime, INTERVAL 30 MINUTE) AS mktstrt_date_plus_30min 
FROM farmers_market.datetime_demo
WHERE market_start_datetime = '2019-03-02 08:00:00'
```

* 30 days past a date 

```sql
SELECT 
	market_start_datetime, 
  DATE_ADD(market_start_datetime, INTERVAL 30 DAY) AS mktstrt_date_plus_30days 
FROM farmers_market.datetime_demo
WHERE market_start_datetime = '2019-03-02 08:00:00'
```

* `DATA_SUB` subtracts intervals from datetimes 

```sql 
SELECT 
	market_start_datetime, 
  DATE_ADD(market_start_datetime, INTERVAL -30 DAY) AS mkstrt_date_plus_neg30days, 
  DATE_SUB(market_start_datetime, INTERVAL 30 DAY) AS mktstrt_date_minus_30days
FROM farmers_market.datetime_demo
WHERE market_start_datetime = '2019-03-02 08:00:00'
```

## `DATEDIFF` 

* Returns the difference between two dates in days 

```sql
SELECT 
	x.first_market, 
  x.last_market, 
  DATEDIFF(x.last_market, x.first_market) days_first_to_last
FROM 
(
	SELECT
		min(market_start_datetime) first_market, 
    max(market_start_datetime) last_market
	FROM farmers_market.datetime_demo
) x
```

## `TIMESTAMPDIFF`

* Returns difference between two dates in any chosen interval 

```sql
SELECT market_start_datetime, market_end_datetime,
	TIMESTAMPDIFF(HOUR, market_start_datetime, market_end_datetime)
		AS market_duration_hours, 
	TIMESTAMPDIFF(MINUTE, market_start_datetime, market_end_datetime)
		AS market_duration_mins
FROM farmers_market.datetime_demo
```

## Date Functions in Aggregate Summaries and Window Functions 

* Get a profile of farmers's market customer's habits over time 

1. Customer purchase detail records 

```sql 
SELECT customer_id, market_date
FROM farmers_market.customer_purchases 
WHERE customer_id = 1
```

2. Purchase distribution 

```sql 
SELECT customer_id, 
	MIN(market_date) AS first_purchase,
  MAX(market_date) AS last_purchase, 
  COUNT(DISTINCT market_date) AS count_of_purchase_dates
FROM farmers_market.customer_purchases
WHERE customer_id = 1
GROUP BY customer_id
```

3. How long the customer has been a customer at the market 

```sql
SELECT customer_id, 
	MIN(market_date) AS first_purchase,
  MAX(market_date) AS last_purchase, 
  COUNT(DISTINCT market_date) AS count_of_purchase_dates, 
  DATEDIFF(MAX(market_date), MIN(market_date)) AS days_between_first_last_purchase
FROM farmers_market.customer_purchases 
GROUP BY customer_id
```

4. How long it's been since the customer last made a purchase 

```sql 
SELECT customer_id, 
	MIN(market_date) AS first_purchase,
  MAX(market_date) AS last_purchase, 
  COUNT(DISTINCT market_date) AS count_of_purchase_dates, 
  DATEDIFF(MAX(market_date), MIN(market_date)) AS days_between_first_last_purchase, 
  DATEDIFF(CURDATE(), MAX(market_date)) AS days_since_last_purchase 
FROM farmers_market.customer_purchases 
GROUP BY customer_id
```

5. Days between each purchase a customer makes 

```sql
SELECT customer_id, market_date, 
	RANK() OVER (PARTITION BY customer_id ORDER BY market_date) AS purchase_number, 
  LEAD(market_date, 1) OVER (PARTITION BY customer_id ORDER BY market_date) AS next_purchase
FROM farmers_market.customer_purchases 
WHERE customer_id = 1
```

The above query doesn't quite do what we want since it returns multiple rows with the same date in cases where the customer purchased multiple items on the same day 

Solution is to remove duplicates in the initial dateset and use a subquery to get the date differences 

```sql
SELECT 
	x.customer_id, 
  x.market_date, 
  RANK () OVER (PARTITION BY x.customer_id ORDER BY x.market_date) AS purchase_number,
  LEAD(x.market_date, 1) OVER (PARTITION BY x.customer_id ORDER BY x.market_date) AS next_purchase, 
  DATEDIFF(
	LEAD(x.market_date, 1) OVER 
      (PARTITION BY x.customer_id ORDER BY x.market_date), 
      x.market_date
      ) AS days_between_purchases 
FROM 
(
	SELECT DISTINCT customer_id, market_date
	FROM farmers_market.customer_purchases
	WHERE customer_id = 1
) x
```

If you want avoid inserting `LEAD()`

```sql
SELECT 
	a.customer_id, 
  a.market_date AS first_purchase, 
  a.next_purchase AS second_purchase,
  DATEDIFF(a.next_purchase, a.market_date) AS time_between_1st_2nd_purchase 
FROM 
(
	SELECT 
		x.customer_id,
    x.market_date, 
    RANK() OVER (PARTITION BY x.customer_id ORDER BY x.market_date) AS purchase_number, 
    LEAD(x.market_date, 1) OVER (PARTITION BY x.customer_id ORDER BY x.market_date) AS next_purchase
		FROM 
		(
			SELECT DISTINCT customer_id, market_date
            FROM farmers_market.customer_purchases
		) x
) a 
WHERE a.purchase_number = 1 
```

* Make list of everyone who made a purchase in the 31 days prior to March 31, 2019

1. 
```sql
SELECT DISTINCT customer_id, market_date
FROM farmers_market.customer_purchases
WHERE DATEDIFF('2019-03-31', market_date) <= 31
```

2. 
```sql
SELECT x.customer_id, 
  COUNT(DISTINCT x.market_date) AS market_count
FROM 
( 
	SELECT DISTINCT customer_id, market_date
    FROM farmers_market.customer_purchases
    WHERE DATEDIFF('2019-03-31', market_date) <= 31
) x 
GROUP BY x.customer_id 
HAVING COUNT(DISTINCT market_date) = 1
```

## Exercises 

1. 

2. 

3.
