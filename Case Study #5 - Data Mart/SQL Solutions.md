# 🛒Case Study #5 - Data Mart

## Data Exploration
**1. What day of the week is used for each week_date value?**

```sql
SELECT DISTINCT to_char(week_date,'Day') day_of_week
FROM data_mart.clean_weekly_sales;
```

#### Output:
| day_of_week |
|-------------|
| Monday      |

- Monday is used for each week_date value

**2. What range of week numbers are missing from the dataset?**

```sql
SELECT *
FROM generate_series(1,52) 
WHERE generate_series NOT IN (SELECT DISTINCT week_number FROM data_mart.clean_weekly_sales)
```

#### Output:
| generate_series |
|-----------------|
| 1               |
| 2               |
| 3               |
| 4               |
| 5               |
| 6               |
| 7               |
| 8               |
| 9               |
| 10              |
| 11              |
| 12              |
| 37              |
| 38              |
| 39              |
| 40              |
| 41              |
| 42              |
| 43              |
| 44              |
| 45              |
| 46              |
| 47              |
| 48              |
| 49              |
| 50              |
| 51              |
| 52              |

- Range 1 from 12 and range 37 from 52 are missing 

**3. How many total transactions were there for each year in the dataset?**

```sql
SELECT calendar_year
	, SUM(transactions) AS total_transaction
FROM data_mart.clean_weekly_sales
GROUP BY 1
ORDER BY 1;
```

#### Output:
| calendar_year | total_transaction |
|---------------|-------------------|
| 2018          | 346406460         |
| 2019          | 365639285         |
| 2020          | 375813651         |

- In 2018, there are a total of 346.406.460 transactions
- In 2019, there are a total of 365.639.285 transactions
- In 2020, there are a total of 375.813.651 transactions
- The number of yearly transactions increased every year

**4. What is the total sales for each region for each month?**

```sql
SELECT region
	, month_number
	, SUM(sales) AS total_sale
FROM data_mart.clean_weekly_sales
GROUP BY 1,2
ORDER BY 1,2;
```

#### Output:
| region        | month_number | total_sale |
|---------------|--------------|------------|
| AFRICA        | 3            | 567767480  |
| AFRICA        | 4            | 1911783504 |
| AFRICA        | 5            | 1647244738 |
| AFRICA        | 6            | 1767559760 |
| AFRICA        | 7            | 1960219710 |
| AFRICA        | 8            | 1809596890 |
| AFRICA        | 9            | 276320987  |
| ASIA          | 3            | 529770793  |
| ASIA          | 4            | 1804628707 |
| ASIA          | 5            | 1526285399 |
| ASIA          | 6            | 1619482889 |
| ASIA          | 7            | 1768844756 |
| ASIA          | 8            | 1663320609 |
| ASIA          | 9            | 252836807  |
| CANADA        | 3            | 144634329  |
| CANADA        | 4            | 484552594  |
| CANADA        | 5            | 412378365  |
| CANADA        | 6            | 443846698  |
| CANADA        | 7            | 477134947  |
| CANADA        | 8            | 447073019  |
| CANADA        | 9            | 69067959   |
| EUROPE        | 3            | 35337093   |
| EUROPE        | 4            | 127334255  |
| EUROPE        | 5            | 109338389  |
| EUROPE        | 6            | 122813826  |
| EUROPE        | 7            | 136757466  |
| EUROPE        | 8            | 122102995  |
| EUROPE        | 9            | 18877433   |
| OCEANIA       | 3            | 783282888  |
| OCEANIA       | 4            | 2599767620 |
| OCEANIA       | 5            | 2215657304 |
| OCEANIA       | 6            | 2371884744 |
| OCEANIA       | 7            | 2563459400 |
| OCEANIA       | 8            | 2432313652 |
| OCEANIA       | 9            | 372465518  |
| SOUTH AMERICA | 3            | 71023109   |
| SOUTH AMERICA | 4            | 238451531  |
| SOUTH AMERICA | 5            | 201391809  |
| SOUTH AMERICA | 6            | 218247455  |
| SOUTH AMERICA | 7            | 235582776  |
| SOUTH AMERICA | 8            | 221166052  |
| SOUTH AMERICA | 9            | 34175583   |
| USA           | 3            | 225353043  |
| USA           | 4            | 759786323  |
| USA           | 5            | 655967121  |
| USA           | 6            | 703878990  |
| USA           | 7            | 760331754  |
| USA           | 8            | 712002790  |
| USA           | 9            | 110532368  |

**5. What is the total count of transactions for each platform**

```sql
SELECT platform
	, SUM(transactions) total_transaction
FROM data_mart.clean_weekly_sales
GROUP BY 1
ORDER BY 2;
```

#### Output:
| platform | total_transaction |
|----------|-------------------|
| Shopify  | 5925169           |
| Retail   | 1081934227        |

- Retail has a total of 1.081.934.227 transactions
- Shopify has a total of 5.925.169 transactions
- Platform Retail has more transactions than Shopify.

**6. What is the percentage of sales for Retail vs Shopify for each month?**

```sql
WITH sale_per_month_each_platform AS (
	SELECT platform
		, month_number
		, SUM(sales) AS sale_per_month
	FROM data_mart.clean_weekly_sales
	GROUP BY 1,2
	ORDER BY 1,2
	),

	total_sale_per_month AS (
	SELECT month_number
		, SUM(sale_per_month) AS total_sale
	FROM sale_per_month_each_platform
	GROUP BY 1
	)

SELECT t1.platform
	, t1.month_number
	, ROUND(t1.sale_per_month/t2.total_sale*100,2) AS perc_total_sale
FROM sale_per_month_each_platform t1
JOIN total_sale_per_month t2
	ON t1.month_number = t2.month_number
ORDER BY 2;	
```

#### Output:
| platform | month_number | perc_total_sale |
|----------|--------------|-----------------|
| Retail   | 3            | 97.54           |
| Shopify  | 3            | 2.46            |
| Retail   | 4            | 97.59           |
| Shopify  | 4            | 2.41            |
| Retail   | 5            | 97.30           |
| Shopify  | 5            | 2.70            |
| Retail   | 6            | 97.27           |
| Shopify  | 6            | 2.73            |
| Retail   | 7            | 97.29           |
| Shopify  | 7            | 2.71            |
| Retail   | 8            | 97.08           |
| Shopify  | 8            | 2.92            |
| Retail   | 9            | 97.38           |
| Shopify  | 9            | 2.62            |

- Retail always contributes to total revenue more than Shopify, accounting for over 97% from March to September.

**7. What is the percentage of sales by demographic for each year in the dataset?**

```sql
WITH sale_per_year_demographic AS (
	SELECT demographic
		, calendar_year
		, SUM(sales) AS sale_per_year
	FROM data_mart.clean_weekly_sales
	GROUP BY 1,2
	ORDER BY 1,2
	),

	total_sale_per_year AS (
	SELECT calendar_year
		, SUM(sale_per_year) AS total_sale
	FROM sale_per_year_demographic
	GROUP BY 1
	)
	
SELECT t1.demographic
	, t1.calendar_year
	, ROUND(sale_per_year/total_sale*100,2) AS perc_sale
FROM sale_per_year_demographic t1
JOIN total_sale_per_year t2
	ON t1.calendar_year = t2.calendar_year
ORDER BY 2,1;
```

#### Output:
| demographic | calendar_year | perc_sale |
|-------------|---------------|-----------|
| Couples     | 2018          | 26.38     |
| Families    | 2018          | 31.99     |
| unknown     | 2018          | 41.63     |
| Couples     | 2019          | 27.28     |
| Families    | 2019          | 32.47     |
| unknown     | 2019          | 40.25     |
| Couples     | 2020          | 28.72     |
| Families    | 2020          | 32.73     |
| unknown     | 2020          | 38.55     |


**8. Which age_band and demographic values contribute the most to Retail sales?**

```sql
SELECT age_band
	, demographic
	, SUM(sales) AS total_sale
	, ROUND(100*SUM(sales)::numeric / SUM(SUM(sales)) OVER(), 2) AS perc_contribute
FROM data_mart.clean_weekly_sales
WHERE platform = 'Retail'
GROUP BY 1,2
ORDER BY 3 DESC
```

#### Output:
| age_band     | demographic | total_sale  | perc_contribute |
|--------------|-------------|-------------|-----------------|
| unknown      | unknown     | 16067285533 | 40.52           |
| Retirees     | Families    | 6634686916  | 16.73           |
| Retirees     | Couples     | 6370580014  | 16.07           |
| Middle Aged  | Families    | 4354091554  | 10.98           |
| Young Adults | Couples     | 2602922797  | 6.56            |
| Middle Aged  | Couples     | 1854160330  | 4.68            |
| Young Adults | Families    | 1770889293  | 4.47            |

- Unknown age_band and unknown demographic contribute the most to Retail sales with 40.52%

**9. Can we use the avg_transaction column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?**

```sql
SELECT calendar_year
	, platform
	, SUM(sales)/SUM(transactions) AS avg_transaction_size
FROM data_mart.clean_weekly_sales
GROUP BY 1,2
ORDER BY 1,2;
```

#### Output:
| calendar_year | platform | avg_transaction_size |
|---------------|----------|----------------------|
| 2018          | Retail   | 36                   |
| 2018          | Shopify  | 192                  |
| 2019          | Retail   | 36                   |
| 2019          | Shopify  | 183                  |
| 2020          | Retail   | 36                   |
| 2020          | Shopify  | 179                  |

***
    
## Before & After Analysis
Taking the week_date value of 2020-06-15 as the baseline week where the Data Mart sustainable packaging changes came into effect.

We would include all week_date values for 2020-06-15 as the start of the period after the change and the previous week_date values would be before.

**1. What is the total sales for the 4 weeks before and after 2020-06-15? What is the growth or reduction rate in actual values and percentage of sales?**

```sql
CREATE TEMP TABLE total_sale_per_week_2020 AS (
	SELECT week_date
		, week_number
		, SUM(sales) AS total_sale
	FROM data_mart.clean_weekly_sales
	WHERE calendar_year = 2020
	GROUP BY 1,2
	ORDER BY 2,1
	);

-- 2020-06-15 is week 25
SELECT *
	, after - before AS sale_change
	, ROUND(100.0*(after - before)/before,2) AS change_rate 
FROM (
	SELECT SUM(CASE WHEN week_number BETWEEN 21 AND 24 THEN total_sale END) AS before
		, SUM(CASE WHEN week_number BETWEEN 25 AND 28 THEN total_sale END) AS after
	FROM total_sale_per_week_2020
	) AS t1;
```

#### Output:
| before     | after      | sale_change | change_rate |
|------------|------------|-------------|-------------|
| 2345878357 | 2318994169 | -26884188   | -1.15       |

- There was a slight decrease in sales by 1.15% after Data Mart sustainable packaging changes came into effect

**2. What about the entire 12 weeks before and after?**

```sql
SELECT *
	, after - before AS sale_change
	, ROUND(100.0*(after - before)/before,2) AS change_rate 
FROM (
	SELECT SUM(CASE WHEN week_number BETWEEN 13 AND 24 THEN total_sale END) AS before
		, SUM(CASE WHEN week_number BETWEEN 25 AND 36 THEN total_sale END) AS after
	FROM total_sale_per_week_2020
	) AS t1
```

#### Output:
| before     | after      | sale_change | change_rate |
|------------|------------|-------------|-------------|
| 7126273147 | 6973947753 | -152325394  | -2.14       |

- Sales dropped by 2.14% in the 12-week period after Data Mart performed sustainable packaging changes compared to the 12-week period before the changes.

**3. How do the sale metrics for these 2 periods before and after compare with the previous years in 2018 and 2019?**

```sql
CREATE TEMP TABLE total_sale_per_week AS (
	SELECT calendar_year
		, week_number
		, SUM(sales) AS total_sale
	FROM data_mart.clean_weekly_sales
	GROUP BY 1,2
	ORDER BY 2,1
	);

-- 4 weeks before and after week 25th	
SELECT *
	, after - before AS sale_change
	, ROUND(100.0*(after - before)/before,2) AS change_rate 
FROM (
	SELECT calendar_year
		, SUM(CASE WHEN week_number BETWEEN 21 AND 24 THEN total_sale END) AS before
		, SUM(CASE WHEN week_number BETWEEN 25 AND 28 THEN total_sale END) AS after
	FROM total_sale_per_week
	GROUP BY 1
	) AS t1;
```

#### Output:
| calendar_year | before     | after      | sale_change | change_rate |
|---------------|------------|------------|-------------|-------------|
| 2018          | 2125140809 | 2129242914 | 4102105     | 0.19        |
| 2019          | 2249989796 | 2252326390 | 2336594     | 0.10        |
| 2020          | 2345878357 | 2318994169 | -26884188   | -1.15       |

```sql
-- entire 12 weeks before and after week 25th
SELECT *
	, after - before AS sale_change
	, ROUND(100.0*(after - before)/before,2) AS change_rate 
FROM (
	SELECT calendar_year
		, SUM(CASE WHEN week_number BETWEEN 13 AND 24 THEN total_sale END) AS before
		, SUM(CASE WHEN week_number BETWEEN 25 AND 36 THEN total_sale END) AS after
	FROM total_sale_per_week
	GROUP BY 1
	) AS t1;
```

#### Output:
| calendar_year | before     | after      | sale_change | change_rate |
|---------------|------------|------------|-------------|-------------|
| 2018          | 6396562317 | 6500818510 | 104256193   | 1.63        |
| 2019          | 6883386397 | 6862646103 | -20740294   | -0.30       |
| 2020          | 7126273147 | 6973947753 | -152325394  | -2.14       |

- In 2018, after the changes, sales increased by 1.63%
- In the other hand, in 2019 and 2020, Data Mart saw a decrease in sales by 0.3% and 2.14, respectively.
- It could be seen that customers prefered the previous version packaging or they were not aware of the changes.
  
## Bonus Question
**Which areas of the business have the highest negative impact in sales metrics performance in 2020 for the 12 week before and after period?**
- region
- platform
- age_band
- demographic
- customer_type

### - region

```sql
WITH region_sale AS (
	SELECT region
		, week_number
		, SUM(sales) AS total_sale
	FROM data_mart.clean_weekly_sales
	WHERE calendar_year = 2020
	GROUP BY 1,2
	ORDER BY 2,1
	)
	
SELECT *
	, after - before AS sale_change
	, ROUND(100.0*(after - before)/before,2) AS change_rate
FROM (
	SELECT region
		, SUM(CASE WHEN week_number BETWEEN 13 AND 24 THEN total_sale END) AS before
		, SUM(CASE WHEN week_number BETWEEN 25 AND 36 THEN total_sale END) AS after
	FROM region_sale
	GROUP BY 1
	) AS t1
ORDER BY sale_change
```

#### Output:
| region        | before     | after      | sale_change | change_rate |
|---------------|------------|------------|-------------|-------------|
| OCEANIA       | 2354116790 | 2282795690 | -71321100   | -3.03       |
| ASIA          | 1637244466 | 1583807621 | -53436845   | -3.26       |
| USA           | 677013558  | 666198715  | -10814843   | -1.60       |
| AFRICA        | 1709537105 | 1700390294 | -9146811    | -0.54       |
| CANADA        | 426438454  | 418264441  | -8174013    | -1.92       |
| SOUTH AMERICA | 213036207  | 208452033  | -4584174    | -2.15       |
| EUROPE        | 108886567  | 114038959  | 5152392     | 4.73        |

- OCEANIA had the highest negative impact in sales performance in 2020 for the 12-week period after the changes.
- Meanwhile, EUROPE had positive effect in sales performance.
### - platform

```sql
WITH platform_sale AS (
	SELECT platform
		, week_number
		, SUM(sales) AS total_sale
	FROM data_mart.clean_weekly_sales
	WHERE calendar_year = 2020
	GROUP BY 1,2
	ORDER BY 2,1
	)
	
SELECT *
	, after - before AS sale_change
	, ROUND(100.0*(after - before)/before,2) AS change_rate
FROM (
	SELECT platform
		, SUM(CASE WHEN week_number BETWEEN 13 AND 24 THEN total_sale END) AS before
		, SUM(CASE WHEN week_number BETWEEN 25 AND 36 THEN total_sale END) AS after
	FROM platform_sale
	GROUP BY 1
	) AS t1
ORDER BY sale_change
```

#### Output:
| platform | before     | after      | sale_change | change_rate |
|----------|------------|------------|-------------|-------------|
| Retail   | 6906861113 | 6738777279 | -168083834  | -2.43       |
| Shopify  | 219412034  | 235170474  | 15758440    | 7.18        |

- The highest negative impact in sales metrics performance in 2020 for the 12 week before and after period was Retail, corresponding to a decrease of 2.43%
- On the other hand, Shopify had positive impact in sales.

### - age_band

```sql
WITH age_band_sale AS (
	SELECT age_band
		, week_number
		, SUM(sales) AS total_sale
	FROM data_mart.clean_weekly_sales
	WHERE calendar_year = 2020
	GROUP BY 1,2
	ORDER BY 2,1
	)
	
SELECT *
	, after - before AS sale_change
	, ROUND(100.0*(after - before)/before,2) AS change_rate
FROM (
	SELECT age_band
		, SUM(CASE WHEN week_number BETWEEN 13 AND 24 THEN total_sale END) AS before
		, SUM(CASE WHEN week_number BETWEEN 25 AND 36 THEN total_sale END) AS after
	FROM age_band_sale
	GROUP BY 1
	) AS t1
ORDER BY sale_change
```

#### Output:
| age_band     | before     | after      | sale_change | change_rate |
|--------------|------------|------------|-------------|-------------|
| unknown      | 2764354464 | 2671961443 | -92393021   | -3.34       |
| Retirees     | 2395264515 | 2365714994 | -29549521   | -1.23       |
| Middle Aged  | 1164847640 | 1141853348 | -22994292   | -1.97       |
| Young Adults | 801806528  | 794417968  | -7388560    | -0.92       |

- The highest negative impact in sales metrics performance in 2020 for the 12 week before and after period was Unknown age_band, corresponding to a decrease of 3.34%

### - demographic

```sql
WITH demographic_sale AS (
	SELECT demographic
		, week_number
		, SUM(sales) AS total_sale
	FROM data_mart.clean_weekly_sales
	WHERE calendar_year = 2020
	GROUP BY 1,2
	ORDER BY 2,1
	)
	
SELECT *
	, after - before AS sale_change
	, ROUND(100.0*(after - before)/before,2) AS change_rate
FROM (
	SELECT demographic
		, SUM(CASE WHEN week_number BETWEEN 13 AND 24 THEN total_sale END) AS before
		, SUM(CASE WHEN week_number BETWEEN 25 AND 36 THEN total_sale END) AS after
	FROM demographic_sale
	GROUP BY 1
	) AS t1
ORDER BY sale_change
```

#### Output:
| demographic | before     | after      | sale_change | change_rate |
|-------------|------------|------------|-------------|-------------|
| unknown     | 2764354464 | 2671961443 | -92393021   | -3.34       |
| Families    | 2328329040 | 2286009025 | -42320015   | -1.82       |
| Couples     | 2033589643 | 2015977285 | -17612358   | -0.87       |

- The highest negative impact in sales metrics performance in 2020 for the 12 week before and after period was Unknown demographic, corresponding to a decrease of 3.34%

### - customer_type

```sql
WITH customer_type_sale AS (
	SELECT customer_type
		, week_number
		, SUM(sales) AS total_sale
	FROM data_mart.clean_weekly_sales
	WHERE calendar_year = 2020
	GROUP BY 1,2
	ORDER BY 2,1
	)
	
SELECT *
	, after - before AS sale_change
	, ROUND(100.0*(after - before)/before,2) AS change_rate
FROM (
	SELECT customer_type
		, SUM(CASE WHEN week_number BETWEEN 13 AND 24 THEN total_sale END) AS before
		, SUM(CASE WHEN week_number BETWEEN 25 AND 36 THEN total_sale END) AS after
	FROM customer_type_sale
	GROUP BY 1
	) AS t1
ORDER BY sale_change
```

#### Output:
| customer_type | before     | after      | sale_change | change_rate |
|---------------|------------|------------|-------------|-------------|
| Existing      | 3690116427 | 3606243454 | -83872973   | -2.27       |
| Guest         | 2573436301 | 2496233635 | -77202666   | -3.00       |
| New           | 862720419  | 871470664  | 8750245     | 1.01        |

- The highest negative impact in sales metrics performance in 2020 for the 12 week before and after period was Guest, corresponding to a decrease of 3.00%
- Whereas, only New customer had positive effect.
