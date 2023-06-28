# 🛍️Case Study #7 - Balanced Tree Clothing Co.

### High Level Sales Analysis
**1. What was the total quantity sold for all products?**
```sql
SELECT SUM(qty) total_qty
FROM balanced_tree.sales
```

#### Output:
| total_qty |
|-----------|
| 45216     |

**2. What is the total generated revenue for all products before discounts?**
```sql
SELECT SUM(qty*price) AS revenue
FROM balanced_tree.sales 
```

#### Output:
| revenue |
|---------|
| 1289453 |

**3. What was the total discount amount for all products?**
```sql
SELECT ROUND(SUM(price*qty*(discount::numeric/100)),2) AS total_discount_amount
FROM balanced_tree.sales
```

#### Output:
| total_discount_amount |
|-----------------------|
| 156229.14             |

***

### Transaction Analysis
**1. How many unique transactions were there?**
```sql
SELECT COUNT(DISTINCT txn_id)
FROM balanced_tree.sales;
```

#### Output:
| count |
|-------|
| 2500  |


**2. What is the average unique products purchased in each transaction?**
```sql
SELECT ROUND(AVG(product_count)) avg_unique_prod
FROM (
	SELECT txn_id
		, COUNT (DISTINCT prod_id) AS product_count
	FROM balanced_tree.sales
	GROUP BY 1
	) AS t1;
```

#### Output:
| avg_unique_prod |
|-----------------|
| 6               |


**3. What are the 25th, 50th and 75th percentile values for the revenue per transaction?**
```sql
WITH rank_rev_per_trans AS (
	SELECT txn_id
		, SUM(price*qty) AS revenue
	FROM balanced_tree.sales
	GROUP BY 1
	)
	
SELECT PERCENTILE_CONT(0.25) WITHIN GROUP(ORDER BY revenue) AS perc_25th
	, PERCENTILE_CONT(0.50) WITHIN GROUP(ORDER BY revenue) AS perc_50th
	, PERCENTILE_CONT(0.75) WITHIN GROUP(ORDER BY revenue) AS perc_75th
FROM rank_rev_per_trans;
```

#### Output:
| perc_25th | perc_50th | perc_75th |
|-----------|-----------|-----------|
| 375.75    | 509.5     | 647       |


**4. What is the average discount value per transaction?**
```sql
SELECT ROUND(AVG(total_discount),2) AS avg_discount
FROM (
	SELECT txn_id
		, SUM(price*qty*(discount::numeric/100)) AS total_discount
	FROM balanced_tree.sales
	GROUP BY 1
	) AS t1;
```

#### Output:
| avg_discount |
|--------------|
| 62.49        |


**5. What is the percentage split of all transactions for members vs non-members?**
```sql
SELECT ROUND(100.0*member_count/(member_count + non_member_count),2) AS perc_member
	, ROUND(100.0*non_member_count/(member_count + non_member_count),2) AS perc_non_member
FROM (
	SELECT COUNT(DISTINCT txn_id) FILTER(WHERE member = 'true') AS member_count
		, COUNT(DISTINCT txn_id) FILTER(WHERE member = 'false') AS non_member_count
	FROM balanced_tree.sales
	) AS t1;
```

#### Output:
| perc_member | perc_non_member |
|-------------|-----------------|
| 60.20       | 39.80           |


**6. What is the average revenue for member transactions and non-member transactions?**
```sql
SELECT member
	, ROUND(AVG(total_rev),2) AS avg_rev
FROM (
	SELECT member
		, txn_id
		, SUM(price*qty) AS total_rev
	FROM balanced_tree.sales
	GROUP BY 1,2
	) AS t1
GROUP BY 1;
```

#### Output:
| member | avg_rev |
|--------|---------|
| False  | 515.04  |
| True   | 516.27  |

***

### Product Analysis
**1. What are the top 3 products by total revenue before discount?**
```sql
SELECT *
FROM (
	SELECT p.product_name
		, SUM(s.price*s.qty) AS total_revevue
		, rank() OVER(ORDER BY SUM(s.price*s.qty) DESC) AS ranking
	FROM balanced_tree.sales s
  JOIN balanced_tree.product_details p
    ON s.prod_id = p.product_id
	GROUP BY 1
  ORDER BY 2 DESC
	) AS t1
WHERE ranking <= 3
```

#### Output:
| product_name                 | total_revevue | ranking |
|------------------------------|---------------|---------|
| Blue Polo Shirt - Mens       | 217683        | 1       |
| Grey Fashion Jacket - Womens | 209304        | 2       |
| White Tee Shirt - Mens       | 152000        | 3       |



**2. What is the total quantity, revenue and discount for each segment?**
```sql
SELECT p.segment_name
	, SUM(s.qty) AS total_qty
	, SUM(s.price*s.qty) AS total_revenue
	, ROUND(SUM(s.price*s.qty*(s.discount::numeric/100)),2) AS total_discount
FROM balanced_tree.sales s
JOIN balanced_tree.product_details p
	ON s.prod_id = p.product_id
GROUP BY 1
ORDER BY 1;
```

#### Output:
| segment_name | total_qty | total_revenue | total_discount |
|--------------|-----------|---------------|----------------|
| Jacket       | 11385     | 366983        | 44277.46       |
| Jeans        | 11349     | 208350        | 25343.97       |
| Shirt        | 11265     | 406143        | 49594.27       |
| Socks        | 11217     | 307977        | 37013.44       |

- Jeans had the highest total quantity sold
- Shirt had the highest total revenue

**3. What is the top selling product for each segment?**
```sql
SELECT segment_name
	, product_name
  , total_qty
FROM (
	SELECT p.segment_name
		, p.product_name
		, SUM(s.qty) AS total_qty
		, rank() OVER(PARTITION BY p.segment_name ORDER BY SUM(s.qty) DESC) AS ranking
	FROM balanced_tree.sales s
	JOIN balanced_tree.product_details p
		ON s.prod_id = p.product_id
	GROUP BY 1,2
	) AS t1
WHERE ranking = 1;
```

#### Output:
| segment_name | product_name                  | total_qty |
|--------------|-------------------------------|-----------|
| Jacket       | Grey Fashion Jacket - Womens  | 3876      |
| Jeans        | Navy Oversized Jeans - Womens | 3856      |
| Shirt        | Blue Polo Shirt - Mens        | 3819      |
| Socks        | Navy Solid Socks - Mens       | 3792      |


**4. What is the total quantity, revenue and discount for each category?**
```sql
SELECT p.category_name
	, SUM(s.qty) AS total_qty
	, SUM(s.price*s.qty) AS total_revenue
	, ROUND(SUM(s.price*s.qty*(s.discount::numeric/100)),2) AS total_discount
FROM balanced_tree.sales s
JOIN balanced_tree.product_details p
	ON s.prod_id = p.product_id
GROUP BY 1
ORDER BY 1;
```

#### Output:
| category_name | total_qty | total_revenue | total_discount |
|---------------|-----------|---------------|----------------|
| Mens          | 22482     | 714120        | 86607.71       |
| Womens        | 22734     | 575333        | 69621.43       |


**5. What is the top selling product for each category?**
```sql
SELECT category_name
	, product_name
FROM (
	SELECT p.category_name
		, p.product_name
		, SUM(s.qty) AS total_qty
		, rank() OVER(PARTITION BY p.category_name ORDER BY SUM(s.qty) DESC) AS ranking
	FROM balanced_tree.sales s
	JOIN balanced_tree.product_details p
		ON s.prod_id = p.product_id
	GROUP BY 1,2
	) AS t1
WHERE ranking = 1;
```

#### Output:
| category_name | product_name                 |
|---------------|------------------------------|
| Mens          | Blue Polo Shirt - Mens       |
| Womens        | Grey Fashion Jacket - Womens |


**6. What is the percentage split of revenue by product for each segment?**
```sql
WITH revenue_prod AS (
	SELECT p.segment_name
		, p.product_id
		, SUM(s.price*s.qty) AS rev_prod
	FROM balanced_tree.sales s
	JOIN balanced_tree.product_details p
		ON s.prod_id = p.product_id
	GROUP BY 1,2
	),
	
	revenue_seg AS (
	SELECT p.segment_name
		, SUM(s.price*s.qty) AS rev_seg
	FROM balanced_tree.sales s
	JOIN balanced_tree.product_details p
		ON s.prod_id = p.product_id
	GROUP BY 1
	)

SELECT prod.segment_name
	, prod.product_id
	, ROUND(100.0*prod.rev_prod/seg.rev_seg,2) AS perc_rev
FROM revenue_prod AS prod
JOIN revenue_seg AS seg
	ON prod.segment_name = seg.segment_name
ORDER BY 1
```

#### Output:
| segment_name | product_name                     | perc_rev |
|--------------|----------------------------------|----------|
| Jacket       | Grey Fashion Jacket - Womens     | 57.03    |
| Jacket       | Khaki Suit Jacket - Womens       | 23.51    |
| Jacket       | Indigo Rain Jacket - Womens      | 19.45    |
| Jeans        | Black Straight Jeans - Womens    | 58.15    |
| Jeans        | Navy Oversized Jeans - Womens    | 24.06    |
| Jeans        | Cream Relaxed Jeans - Womens     | 17.79    |
| Shirt        | Blue Polo Shirt - Mens           | 53.60    |
| Shirt        | White Tee Shirt - Mens           | 37.43    |
| Shirt        | Teal Button Up Shirt - Mens      | 8.98     |
| Socks        | Navy Solid Socks - Mens          | 44.33    |
| Socks        | Pink Fluro Polkadot Socks - Mens | 35.50    |
| Socks        | White Striped Socks - Mens       | 20.18    |

- The largest revenue share in the Jeans segment - Black Straight Jeans - Womens item - over 58% of revenue for this segment.
- In the Jacket segment - Grey Fashion Jacket - Womens item - over 57% of revenue for this segment.
- In the Shirt segment - Blue Polo Shirt - Mens item - around 54% of revenue for this segment.
- In the Socks segment - Navy Solid Socks - Mens item - over 44% of revenue for this segment.

**7. What is the percentage split of revenue by segment for each category?**
```sql
WITH revenue_seg AS (
	SELECT p.category_name
		, p.segment_name
		, SUM(s.price*s.qty) AS rev_seg
	FROM balanced_tree.sales s
	JOIN balanced_tree.product_details p
		ON s.prod_id = p.product_id
	GROUP BY 1,2
	),
	
	revenue_cate AS (
	SELECT p.category_name
		, SUM(s.price*s.qty) AS rev_cate
	FROM balanced_tree.sales s
	JOIN balanced_tree.product_details p
		ON s.prod_id = p.product_id
	GROUP BY 1
	)

SELECT cate.category_name
	, seg.segment_name
	, ROUND(100.0*seg.rev_seg/cate.rev_cate,2) AS perc_rev
FROM revenue_seg AS seg
JOIN revenue_cate AS cate
	ON cate.category_name = seg.category_name
ORDER BY 1
```

#### Output:
| category_name | segment_name | perc_rev |
|---------------|--------------|----------|
| Mens          | Socks        | 43.13    |
| Mens          | Shirt        | 56.87    |
| Womens        | Jeans        | 36.21    |
| Womens        | Jacket       | 63.79    |

- The largest revenue share in the Womens category - Jacket segment was 64% approximately.
- In the Mens category, the largest revenue share had Shirt segment was around 57%.

**8. What is the percentage split of total revenue by category?**
```sql
WITH total_revenue AS (
	SELECT SUM(price*qty) AS total_rev
	FROM balanced_tree.sales 
	),
	
	revenue_cate AS (
	SELECT p.category_name
		, SUM(s.price*s.qty) AS rev_cate
	FROM balanced_tree.sales s
	JOIN balanced_tree.product_details p
		ON s.prod_id = p.product_id
	GROUP BY 1
	)

SELECT category_name
	, ROUND(100.0*rev_cate/total_rev,2) AS perc_rev
FROM total_revenue, revenue_cate
```

#### Output:
| category_name | perc_rev |
|---------------|----------|
| Mens          | 55.38    |
| Womens        | 44.62    |

**9. What is the total transaction “penetration” for each product? (hint: penetration = number of transactions where at least 1 quantity of a product was purchased divided by total number of transactions)**
```sql
SELECT prod_id
	, product_name
	, ROUND(COUNT(txn_id)::numeric / (SELECT COUNT(DISTINCT txn_id) 
                                      FROM balanced_tree.sales),3) AS txn_penetration
FROM balanced_tree.sales s
JOIN balanced_tree.product_details p 
	ON s.prod_id = p.product_id
GROUP BY 1,2;
```

#### Output:
| prod_id | product_name                     | txn_penetration |
|---------|----------------------------------|-----------------|
| e31d39  | Cream Relaxed Jeans - Womens     | 0.497           |
| c8d436  | Teal Button Up Shirt - Mens      | 0.497           |
| 2feb6b  | Pink Fluro Polkadot Socks - Mens | 0.503           |
| 72f5d4  | Indigo Rain Jacket - Womens      | 0.500           |
| 2a2353  | Blue Polo Shirt - Mens           | 0.507           |
| 5d267b  | White Tee Shirt - Mens           | 0.507           |
| f084eb  | Navy Solid Socks - Mens          | 0.512           |
| c4a632  | Navy Oversized Jeans - Womens    | 0.510           |
| d5e9a6  | Khaki Suit Jacket - Womens       | 0.499           |
| b9a74d  | White Striped Socks - Mens       | 0.497           |
| 9ec847  | Grey Fashion Jacket - Womens     | 0.510           |
| e83aa3  | Black Straight Jeans - Womens    | 0.498           |

**10. What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?**
```sql
SELECT s1.prod_id
	, s2.prod_id
	, s3.prod_id
	, COUNT(*) AS combination_cnt       
FROM balanced_tree.sales s1
JOIN balanced_tree.sales s2 
	ON s1.txn_id = s2.txn_id 
	AND s1.prod_id < s2.prod_id
JOIN balanced_tree.sales s3 
	ON s2.txn_id = s3.txn_id
	AND s2.prod_id < s3.prod_id
GROUP BY 1, 2, 3
ORDER BY 4 DESC
LIMIT 1;
```

#### Output:
| prod_id | prod_id-2 | prod_id-3 | combination_cnt |
|---------|-----------|-----------|-----------------|
| 5d267b  | 9ec847    | c8d436    | 352             |

***

### Reporting Challenge
Write a single SQL script that combines all of the previous questions into a scheduled report that the Balanced Tree team can run at the beginning of each month to calculate the previous month’s values.

Imagine that the Chief Financial Officer (which is also Danny) has asked for all of these questions at the end of every month.

He first wants you to generate the data for January only - but then he also wants you to demonstrate that you can easily run the samne analysis for February without many changes (if at all).

Feel free to split up your final outputs into as many tables as you need - but be sure to explicitly reference which table outputs relate to which question for full marks :)

Here is [monthly_report_script]()

***

### Bonus Challenge
**Use a single SQL query to transform the product_hierarchy and product_prices datasets to the product_details table.**
```sql
WITH product_cte AS (
	SELECT h1.id AS style_id
		, h1.level_text AS style_name
		, h2.id AS segment_id
		, h2.level_text AS segment_name
		, h3.id AS category_id
		, h3.level_text AS category_name
	FROM balanced_tree.product_hierarchy h1
	LEFT JOIN balanced_tree.product_hierarchy h2
		ON h1.parent_id = h2.id
	LEFT JOIN balanced_tree.product_hierarchy h3
		ON h2.parent_id = h3.id
	)
		
SELECT p1.product_id, p1.price
	, CONCAT(p2.style_name,' ',p2.segment_name,' - ',p2.category_name) AS product_name
	, p2.category_id, p2.segment_id, p2.style_id, p2.category_name, p2.segment_name, p2.style_name 
FROM balanced_tree.product_prices p1
JOIN product_cte p2
	ON p1.id = p2.style_id;
```

#### Output:
| product_id | price | product_name                     | category_id | segment_id | style_id | category_name | segment_name | style_name          |
|------------|-------|----------------------------------|-------------|------------|----------|---------------|--------------|---------------------|
| c4a632     | 13    | Navy Oversized Jeans - Womens    | 1           | 3          | 7        | Womens        | Jeans        | Navy Oversized      |
| e83aa3     | 32    | Black Straight Jeans - Womens    | 1           | 3          | 8        | Womens        | Jeans        | Black Straight      |
| e31d39     | 10    | Cream Relaxed Jeans - Womens     | 1           | 3          | 9        | Womens        | Jeans        | Cream Relaxed       |
| d5e9a6     | 23    | Khaki Suit Jacket - Womens       | 1           | 4          | 10       | Womens        | Jacket       | Khaki Suit          |
| 72f5d4     | 19    | Indigo Rain Jacket - Womens      | 1           | 4          | 11       | Womens        | Jacket       | Indigo Rain         |
| 9ec847     | 54    | Grey Fashion Jacket - Womens     | 1           | 4          | 12       | Womens        | Jacket       | Grey Fashion        |
| 5d267b     | 40    | White Tee Shirt - Mens           | 2           | 5          | 13       | Mens          | Shirt        | White Tee           |
| c8d436     | 10    | Teal Button Up Shirt - Mens      | 2           | 5          | 14       | Mens          | Shirt        | Teal Button Up      |
| 2a2353     | 57    | Blue Polo Shirt - Mens           | 2           | 5          | 15       | Mens          | Shirt        | Blue Polo           |
| f084eb     | 36    | Navy Solid Socks - Mens          | 2           | 6          | 16       | Mens          | Socks        | Navy Solid          |
| b9a74d     | 17    | White Striped Socks - Mens       | 2           | 6          | 17       | Mens          | Socks        | White Striped       |
| 2feb6b     | 29    | Pink Fluro Polkadot Socks - Mens | 2           | 6          | 18       | Mens          | Socks        | Pink Fluro Polkadot |
