# 🍕Case Study #2 - Pizza Runner

## C. Ingredient Optimisation

**Q1. What are the standard ingredients for each pizza?**

```sql
SELECT pizza_name
 , STRING_AGG(topping_name, ', ') toppings
FROM (
	SELECT p3.pizza_name
	 , p2.topping_name
	FROM pizza_runner.pizza_recipes_temp p1
	JOIN pizza_runner.pizza_toppings p2
	 ON p1.topping_id = p2.topping_id
	JOIN pizza_runner.pizza_names p3
	 ON p1.pizza_id = p3.pizza_id
	) AS t1
GROUP BY 1
ORDER BY 1;
```

#### Output:
| pizza_name | toppings                                                              |
|------------|-----------------------------------------------------------------------|
| Meatlovers | BBQ Sauce, Pepperoni, Cheese, Salami, Chicken, Bacon, Mushrooms, Beef |
| Vegetarian | Tomato Sauce, Cheese, Mushrooms, Onions, Peppers, Tomatoes            |

***

**Q2. What was the most commonly added extra?**

```sql
SELECT topping_name
 , COUNT(*) count_extra
FROM pizza_runner.extras e1
JOIN pizza_runner.pizza_toppings t
 ON e1.extras = t.topping_id
GROUP BY 1
ORDER BY 2 DESC;
```

#### Output:
| topping_name | count_extra |
|--------------|-------------|
| Bacon        | 4           |
| Chicken      | 1           |
| Cheese       | 1           |

- Bacon was the most commonly added extra

***

**Q3. What was the most common exclusion?**

```sql
SELECT topping_name
 , COUNT(*) count_exclusion
FROM pizza_runner.exclusions e2
JOIN pizza_runner.pizza_toppings t
 ON e2.exclusions = t.topping_id
GROUP BY 1
ORDER BY 2 DESC;
```

#### Output:
| topping_name | count_exclusion |
|--------------|-----------------|
| Cheese       | 4               |
| Mushrooms    | 1               |
| BBQ Sauce    | 1               |

- Cheese was the most commonly exclusion

***

**Q4. Generate an order item for each record in the customers_orders table in the format of one of the following:**

**- Meat Lovers**

**- Meat Lovers - Exclude Beef**

**- Meat Lovers - Extra Bacon**

**- Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers**

```sql
WITH extras_cte AS (
	SELECT e1.record_id
	, CONCAT('Extra ', STRING_AGG(t.topping_name, ', ')) as record_options
	FROM pizza_runner.extras e1
	JOIN pizza_runner.pizza_toppings t
	 ON e1.extras = t.topping_id
	GROUP BY 1
	),
	
		exclusions_cte AS (
	SELECT e2.record_id
	, CONCAT('Exclude ', STRING_AGG(t.topping_name, ', ')) as record_options
	FROM pizza_runner.exclusions e2
	JOIN pizza_runner.pizza_toppings t
	 ON e2.exclusions = t.topping_id
	GROUP BY 1
	),
	
		union_cte AS (
	SELECT * FROM extras_cte
	UNION
	SELECT * FROM exclusions_cte
	)
	
SELECT c.record_id
 , c.order_id
 , CONCAT_WS(' - ', n.pizza_name, STRING_AGG(u.record_options, ' - ')) as order_item
FROM pizza_runner.customer_orders_temp c
JOIN pizza_runner.pizza_names n
 ON c.pizza_id = n.pizza_id
LEFT JOIN union_cte u
 ON u.record_id = c.record_id
GROUP BY c.record_id, n.pizza_name, c.order_id
ORDER BY c.record_id;
```

#### Output:
| record_id | order_id | order_item                                                      |
|-----------|----------|-----------------------------------------------------------------|
| 1         | 1        | Meatlovers                                                      |
| 2         | 2        | Meatlovers                                                      |
| 3         | 3        | Meatlovers                                                      |
| 4         | 3        | Vegetarian                                                      |
| 5         | 4        | Meatlovers - Exclude Cheese                                     |
| 6         | 4        | Meatlovers - Exclude Cheese                                     |
| 7         | 4        | Vegetarian - Exclude Cheese                                     |
| 8         | 5        | Meatlovers - Extra Bacon                                        |
| 9         | 6        | Vegetarian                                                      |
| 10        | 7        | Vegetarian - Extra Bacon                                        |
| 11        | 8        | Meatlovers                                                      |
| 12        | 9        | Meatlovers - Extra Bacon, Chicken - Exclude Cheese              |
| 13        | 10       | Meatlovers                                                      |
| 14        | 10       | Meatlovers - Extra Bacon, Cheese - Exclude BBQ Sauce, Mushrooms |

***
 
**Q5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients.
For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"**

```sql
WITH ingredient AS (
	SELECT record_id
	 , pizza_name
	 , CASE WHEN p1.topping_id IN (
				  SELECT extras
				  FROM pizza_runner.extras e1
				  WHERE c.record_id = e1.record_id
				 ) 
		  THEN CONCAT('2x', p3.topping_name)
		  ELSE p3.topping_name
		  END AS topping
	FROM pizza_runner.customer_orders_temp c 
	JOIN pizza_runner.pizza_names p2 ON c.pizza_id = p2.pizza_id
	JOIN pizza_runner.pizza_recipes_temp p1 ON c.pizza_id = p1.pizza_id
	JOIN pizza_runner.pizza_toppings p3 ON p3.topping_id = p1.topping_id
	WHERE p1.topping_id NOT IN (
				SELECT exclusions 
				FROM pizza_runner.exclusions e2
				WHERE e2.record_id = c.record_id)
				)

SELECT record_id 
 , CONCAT(pizza_name, ': ', STRING_AGG(topping, ', ')) AS ingredient_list
FROM ingredient
GROUP BY record_id, pizza_name
ORDER BY record_id;
```

#### Output:
| record_id | ingredient_list                                                                     |
|-----------|-------------------------------------------------------------------------------------|
| 1         | Meatlovers: Mushrooms, BBQ Sauce, Pepperoni, Cheese, Beef, Bacon, Chicken, Salami   |
| 2         | Meatlovers: Bacon, BBQ Sauce, Chicken, Mushrooms, Pepperoni, Beef, Cheese, Salami   |
| 3         | Meatlovers: Mushrooms, Cheese, Pepperoni, Bacon, Chicken, BBQ Sauce, Salami, Beef   |
| 4         | Vegetarian: Mushrooms, Onions, Cheese, Peppers, Tomato Sauce, Tomatoes              |
| 5         | Meatlovers: Pepperoni, Mushrooms, Bacon, BBQ Sauce, Beef, Salami, Chicken           |
| 6         | Meatlovers: Mushrooms, Salami, Bacon, Chicken, Pepperoni, BBQ Sauce, Beef           |
| 7         | Vegetarian: Onions, Tomato Sauce, Peppers, Tomatoes, Mushrooms                      |
| 8         | Meatlovers: Mushrooms, 2xBacon, BBQ Sauce, Beef, Cheese, Chicken, Pepperoni, Salami |
| 9         | Vegetarian: Cheese, Onions, Tomato Sauce, Tomatoes, Mushrooms, Peppers              |
| 10        | Vegetarian: Tomatoes, Tomato Sauce, Cheese, Mushrooms, Peppers, Onions              |
| 11        | Meatlovers: Pepperoni, BBQ Sauce, Chicken, Salami, Mushrooms, Cheese, Bacon, Beef   |
| 12        | Meatlovers: 2xBacon, Pepperoni, Salami, BBQ Sauce, Beef, Mushrooms, 2xChicken       |
| 13        | Meatlovers: Pepperoni, Mushrooms, Salami, Bacon, Chicken, Cheese, Beef, BBQ Sauce   |
| 14        | Meatlovers: 2xBacon, 2xCheese, Chicken, Salami, Pepperoni, Beef                     |

***

**Q6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?**

```sql
WITH ingredient_used_count AS (
	SELECT record_id
	 , pizza_name 
	 , topping_name
	 , CASE WHEN p1.topping_id in (
				SELECT extras
				FROM pizza_runner.extras e1
				WHERE c.record_id = e1.record_id
				) THEN 2 ELSE 1 END AS times_used_topping
	FROM pizza_runner.customer_orders_temp c 
	JOIN pizza_runner.pizza_names p2 ON c.pizza_id = p2.pizza_id
	JOIN pizza_runner.pizza_recipes_temp p1 ON c.pizza_id = p1.pizza_id
	JOIN pizza_runner.pizza_toppings p3 ON p3.topping_id = p1.topping_id
	JOIN pizza_runner.runner_orders r ON c.order_id = r.order_id
	WHERE p1.topping_id NOT IN (
				SELECT exclusions
				FROM pizza_runner.exclusions e2 
				WHERE e2.record_id = c.record_id) 
				and r.distance IS NOT NULL
				)			

SELECT topping_name 
 , SUM(times_used_topping) AS times_used_topping
FROM ingredient_used_count
GROUP BY 1
ORDER BY 2 DESC;
```

#### Output:
| topping_name | times_used_topping |
|--------------|--------------------|
| Mushrooms    | 13                 |
| Bacon        | 13                 |
| Chicken      | 11                 |
| Cheese       | 11                 |
| Pepperoni    | 10                 |
| Salami       | 10                 |
| Beef         | 10                 |
| BBQ Sauce    | 9                  |
| Tomatoes     | 4                  |
| Onions       | 4                  |
| Peppers      | 4                  |
| Tomato Sauce | 4                  |

- Mushrooms and Bacon were the most frequent ingredient used in all delivered pizzas.
