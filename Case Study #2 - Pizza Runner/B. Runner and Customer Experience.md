# 🍕Case Study #2 - Pizza Runner

## B. Runner and Customer Experience

**Q1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)**

In fact, the first week of January 2021 starts on 4th, while Q1 supposes that week starts on 2021-01-01. Therefore, we need to add 3 more days for the registration_date to get meaningful result.

```sql
SELECT EXTRACT(WEEK FROM registration_date + 3) AS week
 , COUNT(runner_id) runner_count
FROM pizza_runner.runners
GROUP BY 1
ORDER BY 1;
```

#### Output:
| week | runner_count |
|------|--------------|
| 1    | 2            |
| 2    | 1            |
| 3    | 1            |

- There are 2 runners signed up in the first week
- There is only 1 runner signed up in the 2nd and 3rd week

***

**Q2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?**

```sql
SELECT r.runner_id
 , ROUND(AVG(EXTRACT(EPOCH FROM (r.pickup_time - c.order_time)))/60,2) AS avg_time_to_arrive
FROM pizza_runner.customer_orders_temp c
JOIN pizza_runner.runner_orders_temp r
 ON c.order_id = r.order_id
GROUP BY 1
ORDER BY 1;
```

#### Output:
| runner_id | avg_time_to_arrive |
|-----------|--------------------|
| 1         | 15.68              |
| 2         | 23.72              |
| 3         | 10.47              |

- Runner id_1 need an average of 15.68 minutes
- Runner id_2 need an average of 23.72 minutes
- Runner id_3 need an average of 10.47 minutes

***

**Q3. Is there any relationship between the number of pizzas and how long the order takes to prepare?**

```sql
WITH prep_time AS (
	SELECT r.order_id
	 , COUNT(r.order_id) pizza_count
	 , EXTRACT(EPOCH FROM (r.pickup_time - c.order_time))/60 AS time_diff
	FROM pizza_runner.customer_orders_temp c
	JOIN pizza_runner.runner_orders_temp r
	 ON c.order_id = r.order_id
	GROUP BY r.order_id, r.pickup_time, c.order_time
    )

SELECT pizza_count
 , ROUND(AVG(time_diff),2) AS avg_prep
FROM prep_time
GROUP BY 1
ORDER BY 1;
```

#### Output:
| pizza_count | avg_prep |
|-------------|----------|
| 1           | 12.36    |
| 2           | 18.38    |
| 3           | 29.28    |

- The more pizzas, the longer the preparation time
- 1 pizza needs an average of 12.36 minutes
- 2 pizzas need an average of 18.38 minutes
- 3 pizzas need an average of 29.28 minutes

***

**Q4. What was the average distance travelled for each customer?**

```sql
SELECT c.customer_id
 , ROUND(CAST(AVG(r.distance) AS numeric),2) AS avg_distance
FROM pizza_runner.customer_orders_temp c
JOIN pizza_runner.runner_orders_temp r
 ON c.order_id = r.order_id
GROUP BY 1
ORDER BY 1;
```

#### Output:
| customer_id | avg_distance |
|-------------|--------------|
| 101         | 20.00        |
| 102         | 16.73        |
| 103         | 23.40        |
| 104         | 10.00        |
| 105         | 25.00        |

- Customer id_101 need an average of 20km 
- Customer id_102 need an average of 16.73km
- Customer id_103 need an average of 23.4km 
- Customer id_104 need an average of 10km
- Customer id_105 need an average of 25km
  
***

**Q5. What was the difference between the longest and shortest delivery times for all orders?**

```sql
SELECT MAX(duration) - MIN(duration) AS delivery_time_diff
FROM pizza_runner.runner_orders_temp;
```

#### Output:
| delivery_time_diff |
|--------------------|
| 30                 |

- The difference between the longest and shortest delivery is 30 minutes.

***

**Q6. What was the average speed for each runner for each delivery and do you notice any trend for these values?**

```sql
SELECT order_id
 , runner_id
 , ROUND(CAST(AVG(distance/duration*60) AS numeric), 2) AS avg_speed_km_h
FROM pizza_runner.runner_orders_temp
WHERE distance IS NOT NULL
GROUP BY 1,2
ORDER BY 2,1;
```

#### Output:
| order_id | runner_id | avg_speed_km_h |
|----------|-----------|----------------|
| 1        | 1         | 37.50          |
| 2        | 1         | 44.44          |
| 3        | 1         | 40.20          |
| 10       | 1         | 60.00          |
| 4        | 2         | 35.10          |
| 7        | 2         | 60.00          |
| 8        | 2         | 93.60          |
| 5        | 3         | 40.00          |

- The average speed of runner id_1 for the order id_1, id_2, id_3 and id_10 is 37.5, 44.4, 40.0, 60.0 respectively.
- The average speed of runner id_2 for the order id_4, id_7, and id_8 is 35.1, 60.0, 93.6 respectively.
- The average speed of runner id_3 for the order id_5 is 40km/h
- There is an upward trend in average speed for runner id_1 and id_2
  
***

**Q7. What is the successful delivery percentage for each runner?**

```sql
SELECT runner_id
 , ROUND(1.0*COUNT(distance)/COUNT(runner_id)*100) AS perc_delivery_success
FROM pizza_runner.runner_orders_temp
GROUP BY 1
ORDER BY 1;
```

#### Output:
| runner_id | perc_delivery_success |
|-----------|-----------------------|
| 1         | 100                   |
| 2         | 75                    |
| 3         | 50                    |

- Runner id_1 has 100% successful delivery
- Runner id_2 has 75% successful delivery
- Runner id_3 has 50% successful delivery
