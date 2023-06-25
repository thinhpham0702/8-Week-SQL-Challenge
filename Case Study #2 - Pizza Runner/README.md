# 🍕Case Study #2: Pizza Runner
<img src=  "https://8weeksqlchallenge.com/images/case-study-designs/2.png" alt="Image" width="500" height="520" >

## 📚 Table of Contents
- **Business Task**
- **Entity Relationship Diagram**
- **Data Cleaning**
- **Case Study Questions and Solutions**

Please keep in mind that all information about the case study has been derived from the following website [here](https://8weeksqlchallenge.com/case-study-2/)   

***

## 🎯 Business Task
- Danny was attracted by an idea “80s Retro Styling and Pizza Is The Future!” while he was scrolling through his Instagram feed. He thought that pizza alone was not going to help him get seed funding to expand his new Pizza Empire, therefore, he was going to Uberize it - and so Pizza Runner was launched!
- However, Danny had a few years of experience as a data scientist, he was very aware that data collection was going to be critical for his business’ growth.
- He has prepared for us an entity relationship diagram of his database design but requires further assistance to clean his data and apply some basic calculations so he can better direct his runners and optimise Pizza Runner’s operations.

***

## Entity Relationship Diagram  

<img src= "https://github.com/thinhpham0702/8-Week-SQL-Challenge/assets/136966635/7ab5daa3-b679-4692-bb10-d81a6168306e" alt="Image">

*** 

## Data Cleaning

### Table: Customer_orders 

First, take a look at `customer_orders` table, as we can see:
- In `exclusions` and `extras` columns, there are some blank spaces and null string.
  
<img src= "https://github.com/thinhpham0702/8-Week-SQL-Challenge/assets/136966635/0486c4ec-bf07-4529-8b3b-c05bdad76e56" alt="Image">

From the above table, we need to:
- Create temporary table for customer_orders
- Replace all blank and null string with NULL value for consistency
- To be able to uniquely identify each pizza ordered, we have to add an `record_id` column

<img src= "https://github.com/thinhpham0702/8-Week-SQL-Challenge/assets/136966635/21c9c916-36ec-4e38-ac86-cf72280ba7db" alt="Image">

Second, we notice that both columns are related to `topping_id` column from `pizza_topping` table so they are in wrong data type, our solutions are:
- Create temporary table for extras and exclusions.
- Splitting the exclusions & extras comma delimited lists into rows
- Transform their data type for consistency

<img src= "https://github.com/thinhpham0702/8-Week-SQL-Challenge/assets/136966635/9c574cec-4d9b-400a-9403-cb522f3b8c7b" alt="Image">
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

<img src= "https://github.com/thinhpham0702/8-Week-SQL-Challenge/assets/136966635/f6d20106-1b9f-4b10-ba27-b799d9602325" alt="Image">


  
















