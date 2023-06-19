# 🍜Case Study #1: Danny's Diner 
<img src= "https://8weeksqlchallenge.com/images/case-study-designs/1.png" alt="Image" width="500" height="520">

## 📚 Table of Contents
- **Business Task**
- **Entity Relationship Diagram**
- **Case Study Questions and Solutions**

Please keep in mind that all information about the case study has been derived from the following website [here](https://8weeksqlchallenge.com/case-study-1/) 

***

## Business Task
- How much did Danny's customers spend?
- Which menu items are their favorite?
- Should Danny expand existing customer loyalty program?

***

## Entity Relationship Diagram

![Danny's Diner](https://github.com/thinhpham0702/8-Week-SQL-Challenge/assets/136966635/53e1e093-4003-49cc-a170-dc9be23016a8)

## Questions and Solutions

**Q1. What is the total amount each customer spent at the restaurant?**

```sql
SELECT sa.customer_id
 , SUM(me.price) AS total_spent
FROM menu mu
JOIN sales sa
 ON me.product_id = sa.product_id
GROUP BY 1
ORDER BY 1
```

#### Output

| customer_id | total_spent |
|-------------|-------------|
| A           | 76          |
| B           | 74          |
| C           | 36          |

- Customer A spent 76$
- Customer B spent 74$
- Customer C spent 36$

***

**Q2. How many days has each customer visited the restaurant?**

```sql
SELECT customer_id
 , COUNT(DISTINCT order_date) day_count
FROM sales
GROUP BY 1
ORDER BY 1
```

#### Output

| customer_id | day_count |
|-------------|-----------|
| A           | 4         |
| B           | 6         |
| C           | 2         |

- Customer A visited the restaurant 4 days
- Customer B visited the restaurant 6 days
- Customer C visited the restaurant 2 days

***

**Q3. What was the first item from the menu purchased by each customer?**

```sql
SELECT DISTINCT customer_id
 , product_name
FROM (
	SELECT sa.customer_id
	 , sa.order_date
	 , me.product_name
	 , rank() OVER (PARTITION BY sa.customer_id ORDER BY sa.order_date) ranking
	FROM menu me
	JOIN sales sa
	 ON me.product_id = sa.product_id
	) AS t1
WHERE ranking = 1
```

#### Output

| customer_id | product_name |
|-------------|--------------|
| A           | curry        |
| A           | sushi        |
| B           | curry        |
| C           | ramen        |

- Customer A's first purchased item is curry and sushi
- Customer B's first purchased item is curry 
- Customer C's first purchased item is ramen

***

**Q4. What is the most purchased item on the menu and how many times was it purchased by all customers?**

```sql
WITH most_purchased AS (
	SELECT mu.product_id
	 , mu.product_name
	 , COUNT(*) AS num_item
	FROM sales sa
	JOIN menu mu
	 ON sa.product_id = mu.product_id
	GROUP BY 1,2
	ORDER BY num_item DESC
	LIMIT 1)
	
SELECT sa.customer_id
 , COUNT(sa.product_id)
FROM sales AS sa
JOIN most_purchased AS mp
 ON sa.product_id = mp.product_id
WHERE sa.product_id = mp.product_id
GROUP BY 1
```

#### Output 1

| product_id | product_name | num_item |
|------------|--------------|----------|
| 3          | ramen        | 8        |

#### Output 2

| customer_id | count |
|-------------|-------|
| A           | 3     |
| B           | 2     |
| C           | 3     |

- Ramen is the most popular food that customer A, B and C ordered 3, 2 and 3 times, respectively.

***

**Q5. Which item was the most popular for each customer?**

```
SELECT customer_id
 , product_name
 , count_item
FROM (
	SELECT sa.customer_id
	 , sa.product_id
	 , mu.product_name
	 , COUNT(mu.product_id) count_item 
	 , rank() OVER (PARTITION BY sa.customer_id ORDER BY COUNT(sa.product_id) DESC) ranking
	FROM sales sa
	JOIN menu mu
	 ON sa.product_id = mu.product_id
	GROUP BY 1,2,3
	ORDER BY 1,2
	) AS t1
WHERE ranking = 1
```

#### Output

| customer_id | product_name |
|-------------|--------------|
| A           | ramen        |
| B           | sushi        |
| B           | curry        |
| B           | ramen        |
| C           | ramen        |

- Ramen is the most favorite dish for both customer A and C
- While customer C purchased 3 dishes with equal number of times.

***

**Q6. Which item was purchased first by the customer after they became a member?**

```sql
SELECT customer_id
 , product_name
FROM (
	SELECT sa.customer_id
	 , sa.product_id
	 , mu.product_name
	 , sa.order_date
	 , rank() OVER (PARTITION BY sa.customer_id ORDER BY sa.order_date) ranking
	FROM members me
	JOIN sales sa
	 ON me.customer_id = sa.customer_id
	JOIN menu mu
	 ON mu.product_id = sa.product_id
	WHERE sa.order_date > me.join_date
	) AS t1
WHERE ranking = 1
```

#### Output

| customer_id | product_name |
|-------------|--------------|
| A           | ramen        |
| B           | sushi        |

- After becoming member, the first item purchased by customer A is ramen and customer B is sushi.

***

**Q7. Which item was purchased just before the customer became a member?**

```sql
SELECT customer_id
 , product_name
FROM (
	SELECT sa.customer_id
	 , mu.product_name
	 , sa.order_date
	 , rank() OVER (PARTITION BY sa.customer_id ORDER BY sa.order_date DESC) ranking
	FROM members me
	JOIN sales sa
	 ON me.customer_id = sa.customer_id
	JOIN menu mu
	 ON mu.product_id = sa.product_id
	WHERE sa.order_date < me.join_date
	) AS t1
WHERE ranking = 1
```

#### Output

| customer_id | product_name |
|-------------|--------------|
| A           | sushi        |
| A           | curry        |
| B           | sushi        |

- Before signing up for membership the last dish that customer A ordered was sushi and curry, while customer B only ordered sushi. 

***

**Q8. What is the total items and amount spent for each member before they became a member?**

```sql
SELECT sa.customer_id
 , COUNT(sa.product_id) count_item
 , SUM(mu.price) total_spent
FROM members me
JOIN sales sa
 ON me.customer_id = sa.customer_id
JOIN menu mu
 ON mu.product_id = sa.product_id
WHERE sa.order_date < me.join_date
GROUP BY 1
ORDER BY 1
```

#### Output

| customer_id | count_item | total_spent |
|-------------|------------|-------------|
| A           | 2          | 25          |
| B           | 3          | 40          |

Before registering as a member 
- Customer A purchased 2 items for a total cost of 25$ 
- Customer B purchased 3 items on the menu with 40$.

***

**Q9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier how many points would each customer have?**

```sql
WITH item_point AS (
	SELECT *
	, CASE 
		WHEN product_name IN ('sushi') THEN price * 10 * 2 
		ELSE price * 10 END AS points	
	FROM menu)
	
SELECT sa.customer_id
 , SUM(ip.points) AS total_point
FROM sales sa
JOIN item_point ip
ON sa.product_id = ip.product_id
GROUP BY 1
ORDER BY 1
```

#### Output

| customer_id | total_point |
|-------------|-------------|
| A           | 860         |
| B           | 940         |
| C           | 360         |

- Customer A had 860 points
- Customer B had 940 points
- Customer C had 360 points

***

**Q10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**

```sql
WITH item_point AS (
	SELECT sa.customer_id
	 , mu.product_name
	 , mu.price
	 , sa.order_date
	 , CASE	WHEN mu.product_name = 'sushi' THEN mu.price * 10 * 2
			WHEN sa.order_date < me.join_date THEN mu.price * 10
			WHEN sa.order_date BETWEEN me.join_date AND me.join_date + interval '6 day' THEN mu.price * 10 * 2
			WHEN sa.order_date > me.join_date + interval '6 day' THEN mu.price * 10 END AS points
	FROM members me
	JOIN sales sa
	 ON me.customer_id = sa.customer_id
	JOIN menu mu
	 ON mu.product_id = sa.product_id
	WHERE DATE_PART('month', sa.order_date) = 1
	ORDER BY 1,4)

SELECT customer_id
 , SUM(points) AS total_point
FROM item_point
GROUP BY 1
```

#### Output

| customer_id | total_point |
|-------------|-------------|
| A           | 1370        |
| B           | 820         |

At the end of January
- Customer A earned 1370 points
- Customer B earned 820 points

***

### Bonus Questions

**Q1. Create a data table that Danny and his team can use to quickly derive insights**

```sql
SELECT sa.customer_id
	 , sa.order_date
	 , mu.product_name
	 , mu.price
	 , CASE WHEN sa.order_date < me.join_date THEN 'N' 
			WHEN sa.order_date >= me.join_date THEN 'Y'
			ELSE 'N' END AS membership
	FROM sales sa
	LEFT JOIN members me
	 ON me.customer_id = sa.customer_id
	JOIN menu mu
	 ON sa.product_id = mu.product_id
	ORDER BY 1,2
 ```
 
#### Output
 
| customer_id | order_date | product_name | price | membership |
|-------------|------------|--------------|-------|------------|
| A           | 2021-01-01 | sushi        | 10    | N          |
| A           | 2021-01-01 | curry        | 15    | N          |
| A           | 2021-01-07 | curry        | 15    | Y          |
| A           | 2021-01-10 | ramen        | 12    | Y          |
| A           | 2021-01-11 | ramen        | 12    | Y          |
| A           | 2021-01-11 | ramen        | 12    | Y          |
| B           | 2021-01-01 | curry        | 15    | N          |
| B           | 2021-01-02 | curry        | 15    | N          |
| B           | 2021-01-04 | sushi        | 10    | N          |
| B           | 2021-01-11 | sushi        | 10    | Y          |
| B           | 2021-01-16 | ramen        | 12    | Y          |
| B           | 2021-02-01 | ramen        | 12    | Y          |
| C           | 2021-01-01 | ramen        | 12    | N          |
| C           | 2021-01-01 | ramen        | 12    | N          |
| C           | 2021-01-07 | ramen        | 12    | N          |

**Q2. Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program**

```sql
WITH customer_data AS (
	SELECT sa.customer_id
	 , sa.order_date
	 , mu.product_name
	 , mu.price
	 , CASE WHEN sa.order_date < me.join_date THEN 'N' 
			WHEN sa.order_date >= me.join_date THEN 'Y'
			ELSE 'N' END AS membership
	FROM sales sa
	LEFT JOIN members me
	 ON me.customer_id = sa.customer_id
	JOIN menu mu
	 ON sa.product_id = mu.product_id
	ORDER BY 1,2
	)

SELECT *
 , CASE WHEN membership = 'N' THEN NULL
 		ELSE rank() OVER(PARTITION BY customer_id, membership ORDER BY order_date) END AS ranking
FROM customer_data
```

#### Output

| customer_id | order_date | product_name | price | membership | ranking |
|-------------|------------|--------------|-------|------------|---------|
| A           | 2021-01-01 | sushi        | 10    | N          | NULL    |
| A           | 2021-01-01 | curry        | 15    | N          | NULL    |
| A           | 2021-01-07 | curry        | 15    | Y          | 1       |
| A           | 2021-01-10 | ramen        | 12    | Y          | 2       |
| A           | 2021-01-11 | ramen        | 12    | Y          | 3       |
| A           | 2021-01-11 | ramen        | 12    | Y          | 3       |
| B           | 2021-01-01 | curry        | 15    | N          | NULL    |
| B           | 2021-01-02 | curry        | 15    | N          | NULL    |
| B           | 2021-01-04 | sushi        | 10    | N          | NULL    |
| B           | 2021-01-11 | sushi        | 10    | Y          | 1       |
| B           | 2021-01-16 | ramen        | 12    | Y          | 2       |
| B           | 2021-02-01 | ramen        | 12    | Y          | 3       |
| C           | 2021-01-01 | ramen        | 12    | N          | NULL    |
| C           | 2021-01-01 | ramen        | 12    | N          | NULL    |
| C           | 2021-01-07 | ramen        | 12    | N          | NULL    |

 





