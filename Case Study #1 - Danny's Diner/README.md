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

