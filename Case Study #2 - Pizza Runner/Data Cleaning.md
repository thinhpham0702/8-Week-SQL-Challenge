# 🍕Case Study #2: Pizza Runner

## Data Cleaning

### Table: Customer_orders 

First, take a look at `customer_orders` table, as we can see:
- In `exclusions` and `extras` columns, there are some blank spaces and null string.
  
<img src= "https://github.com/thinhpham0702/8-Week-SQL-Challenge/assets/136966635/0486c4ec-bf07-4529-8b3b-c05bdad76e56" alt="Image">

From the above table, we need to:
- Create temporary table for customer_orders
- Replace all blank and null string with NULL value for consistency
- To be able to uniquely identify each pizza ordered, we have to add an `record_id` column

```sql
CREATE TABLE pizza_runner.customer_orders_temp AS
SELECT order_id 
 , customer_id 
 , pizza_id 
 , CASE WHEN exclusions LIKE '%null%' OR exclusions = '' THEN NULL
		ELSE exclusions END AS exclusions
 , CASE WHEN extras LIKE '%null%' OR extras = '' THEN NULL
	  	ELSE extras END AS extras
 , order_time
 , row_number() OVER (ORDER BY order_id) record_id
FROM pizza_runner.customer_orders;
```

<img src= "https://github.com/thinhpham0702/8-Week-SQL-Challenge/assets/136966635/21c9c916-36ec-4e38-ac86-cf72280ba7db" alt="Image">

Second, we notice that both columns are related to `topping_id` column from `pizza_topping` table so they are in wrong data type, our solutions are:
- Create temporary table for extras and exclusions.
- Splitting the exclusions & extras comma delimited lists into rows
- Transform their data type for consistency

```sql
CREATE TABLE pizza_runner.extras AS
SELECT record_id
 , TRIM(UNNEST(STRING_TO_ARRAY(extras, ','))) AS extras
FROM pizza_runner.customer_orders_temp;

ALTER TABLE pizza_runner.extras
 ALTER COLUMN extras TYPE int USING extras::int;
```
<img src= "https://github.com/thinhpham0702/8-Week-SQL-Challenge/assets/136966635/9c574cec-4d9b-400a-9403-cb522f3b8c7b" alt="Image">

```sql
CREATE TABLE pizza_runner.exclusions AS
SELECT record_id
 , TRIM(UNNEST(STRING_TO_ARRAY(exclusions, ','))) AS exclusions
FROM pizza_runner.customer_orders_temp;

ALTER TABLE pizza_runner.exclusions
 ALTER COLUMN exclusions TYPE int USING exclusions::int;
```
<img src= "https://github.com/thinhpham0702/8-Week-SQL-Challenge/assets/136966635/1934159c-582b-4730-bab0-54e6810d4366" alt="Image">

*** 

### Table: Runner_orders

First, take a look at `runner_orders` table, as we can see:
- In `pickup_time`, `distance`, `duration` and `cancellation` columns, there are some blank spaces and null string.

<img src= "https://github.com/thinhpham0702/8-Week-SQL-Challenge/assets/136966635/80b2467f-3009-4dda-8509-ade3978aca02" alt="Image">

From the above table, we need to:
- Create temporary table for runner_orders
- Replace all blank and null string with NULL value for consistency
- Remove 'km' from `distance` column
- Remove 'mins', 'minute' and 'minutes' from `duration` column
- Changing date type for `pickup_time`, `distance` and `duration` columns for consistency.

```sql
CREATE TABLE pizza_runner.runner_orders_temp AS
SELECT order_id
 , runner_id  
 , CASE WHEN pickup_time LIKE '%null%' OR pickup_time = '' THEN NULL
		ELSE pickup_time END AS pickup_time
 , CASE WHEN distance LIKE '%null%' OR distance = '' THEN NULL
	  	WHEN distance LIKE '%km' THEN TRIM(REPLACE(distance, 'km', ''))
		ELSE distance END AS distance
 , CASE WHEN duration LIKE '%null%' OR duration = '' THEN NULL
		WHEN duration LIKE '%mins' THEN TRIM(REPLACE(duration, 'mins', ''))
		WHEN duration LIKE '%minute' THEN TRIM(REPLACE(duration, 'minute', ''))
		WHEN duration LIKE '%minutes' THEN TRIM(REPLACE(duration, 'minutes', ''))
		ELSE duration END AS duration
 , CASE WHEN cancellation LIKE '%null%' OR cancellation = '' THEN NULL
		ELSE cancellation END AS cancellation
FROM pizza_runner.runner_orders;
```
```sql
ALTER TABLE pizza_runner.runner_orders_temp
 ALTER COLUMN pickup_time TYPE timestamp USING pickup_time::timestamp,
 ALTER COLUMN distance TYPE float USING distance::float,
 ALTER COLUMN duration TYPE int USING duration::int;
```
<img src= "https://github.com/thinhpham0702/8-Week-SQL-Challenge/assets/136966635/f6d20106-1b9f-4b10-ba27-b799d9602325" alt="Image">

***

### Table: Pizza_recipes

First, take a look at `pizza_recipes` table, as we can see:
- In `toppings` column, there is a list of topping_id for each pizza_id separated by comma.

<img src= "https://github.com/thinhpham0702/8-Week-SQL-Challenge/assets/136966635/0da60f42-a580-4405-9d17-9c876b408f68" alt="Image">

To make querying easier, we need to:
- Create temporary table for pizza_recipes
- Splitting comma dilimited lists into rows
- Changing data type for `toppings` column and rename `toppings` column to topping_id

```sql
CREATE TABLE pizza_runner.pizza_recipes_temp AS
SELECT pizza_id
 , TRIM(UNNEST(STRING_TO_ARRAY(toppings, ','))) AS topping_id
FROM pizza_runner.pizza_recipes;
```
```sql
ALTER TABLE
 ALTER COLUMN toppings TYPE int USING toppings:int,
 RENAME COLUMN toppings TO topping_id;
```
	
<img src= "https://github.com/thinhpham0702/8-Week-SQL-Challenge/assets/136966635/9ab29172-6789-4bb0-8509-6482eba97cd5" alt="Image">
