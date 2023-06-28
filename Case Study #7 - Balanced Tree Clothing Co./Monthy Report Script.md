# 🛍️Case Study #7 - Balanced Tree Clothing Co.

First, create a temporary table `monthly_sales` in a particular month (Ex: January 2021)

```sql
CREATE TEMP TABLE monthly_sales AS (
	SELECT *
	FROM balanced_tree.sales
	WHERE EXTRACT(month FROM start_txn_time) = 1
  AND   EXTRACT(year FROM start_txn_time) = 2021
	);
```

### High Level Sales Analysis
**1. What was the total quantity sold for all products?**
```sql
SELECT SUM(qty) total_qty
FROM monthly_sales
```

**2. What is the total generated revenue for all products before discounts?**
```sql
SELECT SUM(qty*price) AS revenue
FROM monthly_sales
```

**3. What was the total discount amount for all products?**
```sql
SELECT ROUND(SUM(price*qty*(discount::numeric/100)),2) AS total_discount_amount
FROM monthly_sales
```

***

### Transaction Analysis
**1. How many unique transactions were there?**
```sql
SELECT COUNT(DISTINCT txn_id)
FROM monthly_sales
```

**2. What is the average unique products purchased in each transaction?**
```sql
SELECT ROUND(AVG(product_count)) avg_unique_prod
FROM (
	SELECT txn_id
		, COUNT (DISTINCT prod_id) AS product_count
	FROM monthly_sales
	GROUP BY 1
	) AS t1;
```

**3. What are the 25th, 50th and 75th percentile values for the revenue per transaction?**
```sql
WITH rank_rev_per_trans AS (
	SELECT txn_id
		, SUM(price*qty) AS revenue
	FROM monthly_sales
	GROUP BY 1
	)
	
SELECT PERCENTILE_CONT(0.25) WITHIN GROUP(ORDER BY revenue) AS perc_25th
	, PERCENTILE_CONT(0.50) WITHIN GROUP(ORDER BY revenue) AS perc_50th
	, PERCENTILE_CONT(0.75) WITHIN GROUP(ORDER BY revenue) AS perc_75th
FROM rank_rev_per_trans;
```

**4. What is the average discount value per transaction?**
```sql
SELECT ROUND(AVG(total_discount),2) AS avg_discount
FROM (
	SELECT txn_id
		, SUM(price*qty*(discount::numeric/100)) AS total_discount
	FROM monthly_sales
	GROUP BY 1
	) AS t1;
```

**5. What is the percentage split of all transactions for members vs non-members?**
```sql
SELECT ROUND(100.0*member_count/(member_count + non_member_count),2) AS perc_member
	, ROUND(100.0*non_member_count/(member_count + non_member_count),2) AS perc_non_member
FROM (
	SELECT COUNT(DISTINCT txn_id) FILTER(WHERE member = 'true') AS member_count
		, COUNT(DISTINCT txn_id) FILTER(WHERE member = 'false') AS non_member_count
	FROM monthly_sales
	) AS t1;
```

**6. What is the average revenue for member transactions and non-member transactions?**
```sql
SELECT member
	, ROUND(AVG(total_rev),2) AS avg_rev
FROM (
	SELECT member
		, txn_id
		, SUM(price*qty) AS total_rev
	FROM monthly_sales
	GROUP BY 1,2
	) AS t1
GROUP BY 1;
```

***

### Product Analysis
**1. What are the top 3 products by total revenue before discount?**
```sql
SELECT *
FROM (
	SELECT p.product_name
		, SUM(s.price*s.qty) AS total_revevue
		, rank() OVER(ORDER BY SUM(s.price*s.qty) DESC) AS ranking
	FROM monthly_sales s
  JOIN balanced_tree.product_details p
    ON s.prod_id = p.product_id
	GROUP BY 1
  ORDER BY 2 DESC
	) AS t1
WHERE ranking <= 3
```

**2. What is the total quantity, revenue and discount for each segment?**
```sql
SELECT p.segment_name
	, SUM(s.qty) AS total_qty
	, SUM(s.price*s.qty) AS total_revenue
	, ROUND(SUM(s.price*s.qty*(s.discount::numeric/100)),2) AS total_discount
FROM monthly_sales s
JOIN balanced_tree.product_details p
	ON s.prod_id = p.product_id
GROUP BY 1
ORDER BY 1;
```

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
	FROM monthly_sales s
	JOIN balanced_tree.product_details p
		ON s.prod_id = p.product_id
	GROUP BY 1,2
	) AS t1
WHERE ranking = 1;
```

**4. What is the total quantity, revenue and discount for each category?**
```sql
SELECT p.category_name
	, SUM(s.qty) AS total_qty
	, SUM(s.price*s.qty) AS total_revenue
	, ROUND(SUM(s.price*s.qty*(s.discount::numeric/100)),2) AS total_discount
FROM monthly_sales s
JOIN balanced_tree.product_details p
	ON s.prod_id = p.product_id
GROUP BY 1
ORDER BY 1;
```

**5. What is the top selling product for each category?**
```sql
SELECT category_name
	, product_name
FROM (
	SELECT p.category_name
		, p.product_name
		, SUM(s.qty) AS total_qty
		, rank() OVER(PARTITION BY p.category_name ORDER BY SUM(s.qty) DESC) AS ranking
	FROM monthly_sales s
	JOIN balanced_tree.product_details p
		ON s.prod_id = p.product_id
	GROUP BY 1,2
	) AS t1
WHERE ranking = 1;
```

**6. What is the percentage split of revenue by product for each segment?**
```sql
WITH revenue_prod AS (
	SELECT p.segment_name
		, p.product_id
		, SUM(s.price*s.qty) AS rev_prod
	FROM monthly_sales s
	JOIN balanced_tree.product_details p
		ON s.prod_id = p.product_id
	GROUP BY 1,2
	),
	
	revenue_seg AS (
	SELECT p.segment_name
		, SUM(s.price*s.qty) AS rev_seg
	FROM monthly_sales s
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

**7. What is the percentage split of revenue by segment for each category?**
```sql
WITH revenue_seg AS (
	SELECT p.category_name
		, p.segment_name
		, SUM(s.price*s.qty) AS rev_seg
	FROM monthly_sales s
	JOIN balanced_tree.product_details p
		ON s.prod_id = p.product_id
	GROUP BY 1,2
	),
	
	revenue_cate AS (
	SELECT p.category_name
		, SUM(s.price*s.qty) AS rev_cate
	FROM monthly_sales s
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

**8. What is the percentage split of total revenue by category?**
```sql
WITH total_revenue AS (
	SELECT SUM(price*qty) AS total_rev
	FROM monthly_sales
	),
	
	revenue_cate AS (
	SELECT p.category_name
		, SUM(s.price*s.qty) AS rev_cate
	FROM monthly_sales s
	JOIN balanced_tree.product_details p
		ON s.prod_id = p.product_id
	GROUP BY 1
	)

SELECT category_name
	, ROUND(100.0*rev_cate/total_rev,2) AS perc_rev
FROM total_revenue, revenue_cate
```

**9. What is the total transaction “penetration” for each product? (hint: penetration = number of transactions where at least 1 quantity of a product was purchased divided by total number of transactions)**
```sql
SELECT prod_id
	, product_name
	, ROUND(COUNT(txn_id)::numeric / (SELECT COUNT(DISTINCT txn_id) 
                                      FROM monthly_sales),3) AS txn_penetration
FROM monthly_sales s
JOIN balanced_tree.product_details p 
	ON s.prod_id = p.product_id
GROUP BY 1,2;
```

**10. What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?**
```sql
SELECT s1.prod_id
	, s2.prod_id
	, s3.prod_id
	, COUNT(*) AS combination_cnt       
FROM monthly_sales s1
JOIN monthly_sales s2 
	ON s1.txn_id = s2.txn_id 
	AND s1.prod_id < s2.prod_id
JOIN monthly_sales s3 
	ON s2.txn_id = s3.txn_id
	AND s2.prod_id < s3.prod_id
GROUP BY 1, 2, 3
ORDER BY 4 DESC
LIMIT 1;
```
