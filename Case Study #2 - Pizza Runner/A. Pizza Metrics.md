# 🍕Case Study #2 - Pizza Runner

## A. Pizza Metrics

**Q1. How many pizzas were ordered?**

```sql
SELECT COUNT(pizza_id) pizza_order_count
FROM pizza_runner.customer_orders_temp;
```

#### Output:
| pizza_order_count |
|-------------------|
| 14                |

- There are 14 pizzas were ordered

***

**Q2. How many unique customer orders were made?**

```sql
SELECT COUNT(DISTINCT order_id) unique_order_count
FROM pizza_runner.customer_orders_temp;
```

#### Output:
| unique_order_count |
|--------------------|
| 10                 |

- There are 10 unique customer orders

***

**Q3. How many successful orders were delivered by each runner?**

```sql
SELECT runner_id
 , COUNT(order_id) AS successful_order
FROM pizza_runner.runner_orders_temp
WHERE distance IS NOT NULL
GROUP BY 1
ORDER BY 1;
```

#### Output:
| runner_id | successful_order |
|-----------|------------------|
| 1         | 4                |
| 2         | 3                |
| 3         | 1                |

- Runner_id 1 has 4 successful orders.
- Runner_id 2 has 3 successful orders.
- Runner_id 3 has 1 successful order.

***

**Q4. How many of each type of pizza was delivered?**

```sql
SELECT t1.pizza_id
 , t1.pizza_name
 , COUNT(t2.order_id) pizza_delivered_count 
FROM pizza_runner.pizza_names t1
JOIN pizza_runner.customer_orders_temp t2
 ON t1.pizza_id = t2.pizza_id
JOIN pizza_runner.runner_orders_temp t3
 ON t2.order_id = t3.order_id
WHERE t3.distance IS NOT NULL
GROUP BY 1,2;
```

#### Output:
| pizza_id | pizza_name | pizza_delivered_count |
|----------|------------|-----------------------|
| 1        | Meatlovers | 9                     |
| 2        | Vegetarian | 3                     |

- There are 9 Meatlovers pizzas were delivered
- There are 3 Vegetarian pizzas were delivered

***

**Q5. How many Vegetarian and Meatlovers were ordered by each customer?**

```sql
SELECT t2.customer_id
 , t1.pizza_name
 , COUNT(t2.order_id) pizza_ordered_count 
FROM pizza_runner.pizza_names t1
JOIN pizza_runner.customer_orders_temp t2
 ON t1.pizza_id = t2.pizza_id
GROUP BY 1,2
ORDER BY 1;
```

#### Output:
| customer_id | pizza_name | pizza_ordered_count |
|-------------|------------|---------------------|
| 101         | Meatlovers | 2                   |
| 101         | Vegetarian | 1                   |
| 102         | Meatlovers | 2                   |
| 102         | Vegetarian | 1                   |
| 103         | Meatlovers | 3                   |
| 103         | Vegetarian | 1                   |
| 104         | Meatlovers | 3                   |
| 105         | Vegetarian | 1                   |

- Customer id_101 ordered 2 Meatlovers and 1 Vegetarian
- Customer id_102 ordered 2 Meatlovers and 1 Vegetarian
- Customer id_103 ordered 3 Meatlovers and 1 Vegetarian
- Customer id_104 ordered 3 Meatlovers
- Customer id_105 ordered 1 Vegetarian

***

**Q6. What was the maximum number of pizzas delivered in a single order?**

```sql
SELECT t1.order_id
 , COUNT(t1.pizza_id) max_deliver_per_order
FROM pizza_runner.customer_orders_temp t1
JOIN pizza_runner.runner_orders_temp t2
 ON t1.order_id = t2.order_id
WHERE t2.distance IS NOT NULL
GROUP BY 1
ORDER BY 1;
```

#### Output:
| order_id | max_deliver_per_order |
|----------|-----------------------|
| 1        | 1                     |
| 2        | 1                     |
| 3        | 2                     |
| 4        | 3                     |
| 5        | 1                     |
| 7        | 1                     |
| 8        | 1                     |
| 10       | 2                     |

- Order id_1, id_2, id_5, id_7, id_8 had only 1 pizza
- Order id_3 and id_10 had 2 pizzas
- Order id_4 had 3 pizzas

***

**Q7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?**

```sql
SELECT t1.customer_id
 , SUM(CASE WHEN t1.exclusions IS NULL AND t1.extras IS NULL THEN 1 ELSE 0 END) AS no_change_count
 , SUM(CASE WHEN t1.exclusions IS NOT NULL OR t1.extras IS NOT NULL THEN 1 ELSE 0 END) AS at_least_1_change_count
FROM pizza_runner.customer_orders_temp t1
JOIN pizza_runner.runner_orders_temp t2
 ON t1.order_id = t2.order_id
WHERE t2.distance IS NOT NULL
GROUP BY 1
ORDER BY 1;
```

#### Output:
| customer_id | no_change_count | at_least_1_change_count |
|-------------|-----------------|-------------------------|
| 101         | 2               | 0                       |
| 102         | 3               | 0                       |
| 103         | 0               | 3                       |
| 104         | 1               | 2                       |
| 105         | 0               | 1                       |

- Customer id_101 had 2 delivered pizzas with no change
- Customer id_102 had 3 delivered pizzas with no change
- Customer id_103 had 3 delivered pizzas with at least 1 change
- Customer id_104 had 1 delivered pizza with no change and 2 delivered pizzas with at least 1 change
- Customer id_105 had 1 delivered pizza with at least 1 change

***

**Q8. How many pizzas were delivered that had both exclusions and extras?**

```sql
SELECT SUM(CASE WHEN t1.exclusions IS NOT NULL AND t1.extras IS NOT NULL THEN 1 ELSE 0 END) AS pizza_count_exclu_extra
FROM pizza_runner.customer_orders_temp t1
JOIN pizza_runner.runner_orders_temp t2
 ON t1.order_id = t2.order_id
WHERE t2.distance IS NOT NULL;
```

#### Output:
| pizza_count_exclu_extra |
|-------------------------|
| 1                       |

- There is only 1 pizza was delivered that had both exclusions and extras.

***

**Q9. What was the total volume of pizzas ordered for each hour of the day?**

```sql
SELECT EXTRACT(hour FROM order_time) AS hour_of_day
 , COUNT(order_id) AS pizza_order_count
FROM pizza_runner.customer_orders_temp
GROUP BY 1
ORDER BY 1;
```

#### Output:
| hour_of_day | pizza_order_count |
|-------------|-------------------|
| 11          | 1                 |
| 13          | 3                 |
| 18          | 3                 |
| 19          | 1                 |
| 21          | 3                 |
| 23          | 3                 |

- There is only 1 pizza ordered at 11 and 19.
- There are 3 pizzas ordered at 13, 18, 21 and 23.

***

**Q10. What was the volume of orders for each day of the week?**

```sql
SELECT to_char(order_time,'Day') day_of_week
 , COUNT(order_id) AS pizza_order_count
FROM pizza_runner.customer_orders_temp
GROUP BY 1
ORDER BY 1;
```

#### Output:
| day_of_week | pizza_order_count |
|-------------|-------------------|
| Friday      | 1                 |
| Saturday    | 5                 |
| Thursday    | 3                 |
| Wednesday   | 5                 |

- There is only 1 pizza ordered on Friday
- There are 3 pizzas ordered on Thursday
- There are 5 pizzas ordered on Saturday and Wednesday

