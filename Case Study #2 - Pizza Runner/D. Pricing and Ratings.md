# 🍕Case Study #2 - Pizza Runner

## D. Pricing and Ratings

**Q1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?**

```sql
SELECT SUM(CASE WHEN c.pizza_id = 1 THEN 12 ELSE 10 END) total_revenue
FROM pizza_runner.customer_orders_temp c 
JOIN pizza_runner.runner_orders_temp r 
 ON c.order_id = r.order_id
WHERE r.distance IS NOT NULL
```

#### Output:
| total_revenue |
|---------------|
| 138           |

- If there were no charges for changes and no delivery fees, Pizza Runner would earn $138

***

**Q2. What if there was an additional $1 charge for any pizza extras? Add cheese is $1 extra**

```sql
WITH revenue AS (
	SELECT SUM(CASE WHEN c.pizza_id = 1 THEN 12 ELSE 10 END) AS revenue
	FROM pizza_runner.customer_orders_temp c 
	JOIN pizza_runner.runner_orders_temp r 
	 ON c.order_id = r.order_id
	WHERE r.distance IS NOT NULL
	),

	extra_fee AS (
	SELECT SUM(CASE WHEN e1.extras = 4 THEN 2 ELSE 1 END) AS total_extra_fee
	FROM pizza_runner.extras e1
	JOIN pizza_runner.pizza_toppings t
	 ON e1.extras = t.topping_id
	) 

SELECT r.revenue + e.total_extra_fee AS revenue_and_extra_fee
FROM revenue r, extra_fee e
```

#### Output:
| revenue_and_extra_fee |
|-----------------------|
| 145                   |

- If there was an additional $1 charge for any pizza extras and cheese was added $1 more, Pizza Runner would earn $145 

***

**Q3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset 
generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.**

```sql
CREATE TABLE pizza_runner.ratings (
	order_id INT,
	rating INT
	);
INSERT INTO pizza_runner.ratings (order_id, rating)
VALUES 	(1,2),
		(2,3),
		(3,4),
		(4,4),
		(5,5),
		(6,1),
		(7,4),
		(8,5),
		(9,1),
		(10,5);
```

#### Output:
| order_id | rating |
|----------|--------|
| 1        | 2      |
| 2        | 3      |
| 3        | 4      |
| 4        | 4      |
| 5        | 5      |
| 6        | 1      |
| 7        | 4      |
| 8        | 5      |
| 9        | 1      |
| 10       | 5      |

***

**Q4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?**

**- customer_id**

**- order_id**

**- runner_id**

**- rating**

**- order_time**

**- pickup_time**

**- Time between order and pickup**

**- Delivery duration**

**- Average speed**

**- Total number of pizzas**

```sql
SELECT c.customer_id
 , r.order_id
 , r.runner_id
 , rt.rating
 , c.order_time
 , r.pickup_time
 , ROUND(EXTRACT(EPOCH FROM (r.pickup_time - c.order_time))/60) AS diff_time
 , r.duration
 , ROUND(CAST(AVG(r.distance/r.duration*60) AS numeric), 2) AS avg_speed
 , COUNT(c.pizza_id) AS total_pizza
FROM pizza_runner.customer_orders_temp c
JOIN pizza_runner.runner_orders_temp r 
 ON c.order_id = r.order_id
JOIN pizza_runner.ratings rt
 ON rt.order_id = r.order_id
WHERE r.distance IS NOT NULL
GROUP BY 1,2,3,4,5,6,8
ORDER BY 2
```

#### Output:
| customer_id | order_id | runner_id | rating | order_time          | pickup_time         | diff_time | duration | avg_speed | total_pizza |
|-------------|----------|-----------|--------|---------------------|---------------------|-----------|----------|-----------|-------------|
| 101         | 1        | 1         | 2      | 2020-01-01 18:05:02 | 2020-01-01 18:15:34 | 11        | 32       | 37.50     | 1           |
| 101         | 2        | 1         | 3      | 2020-01-01 19:00:52 | 2020-01-01 19:10:54 | 10        | 27       | 44.44     | 1           |
| 102         | 3        | 1         | 4      | 2020-01-02 23:51:23 | 2020-01-03 00:12:37 | 21        | 20       | 40.20     | 2           |
| 103         | 4        | 2         | 4      | 2020-01-04 13:23:46 | 2020-01-04 13:53:03 | 29        | 40       | 35.10     | 3           |
| 104         | 5        | 3         | 5      | 2020-01-08 21:00:29 | 2020-01-08 21:10:57 | 10        | 15       | 40.00     | 1           |
| 105         | 7        | 2         | 4      | 2020-01-08 21:20:29 | 2020-01-08 21:30:45 | 10        | 25       | 60.00     | 1           |
| 102         | 8        | 2         | 5      | 2020-01-09 23:54:33 | 2020-01-10 00:15:02 | 20        | 15       | 93.60     | 1           |
| 104         | 10       | 1         | 5      | 2020-01-11 18:34:49 | 2020-01-11 18:50:20 | 16        | 10       | 60.00     | 2           |

***

**Q5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?**

```sql
WITH revenue AS (
	SELECT SUM(CASE WHEN c.pizza_id = 1 THEN 12 ELSE 10 END) AS revenue
	FROM pizza_runner.customer_orders_temp c 
	JOIN pizza_runner.runner_orders_temp r 
	 ON c.order_id = r.order_id
	WHERE r.distance IS NOT NULL
	),
	
	delivery_fee AS (
	SELECT SUM(distance*0.3) AS delivery_fee
	FROM pizza_runner.runner_orders_temp
	WHERE distance IS NOT NULL
	)
	
SELECT revenue - delivery_fee AS profit
FROM revenue, delivery_fee
```

#### Output:
| profit |
|--------|
| 94.44  |

- If each runner was paid $0.3 per km travelled, Pizza Runner would had $94.44 left after subtracting shipping costs.
