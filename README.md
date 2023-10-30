# SQL Case Study #2 (Week 2) - Pizza Runner

Note: All source material and respected credit is from: https://8weeksqlchallenge.com/

Online SQL instance used to test queries: https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65
>
<img src="https://user-images.githubusercontent.com/81607668/127271856-3c0d5b4a-baab-472c-9e24-3c1e3c3359b2.png" alt="Image" width="500" height="520">

## Table of Contents:
1. [Dataset Structure](#data)
2. [Cleaned Dataset](#clean_data)
3. [Optional: Using Python to clean](#clean_data_python)
4. [Entity Relationship Diagram](#diagram)
5. [Case Study Questions + Answers](#questions)
6. [Bonus Questions + Answers](#bonus)




<div id='data'/>

## Dataset Structure:

Note: The original data was built around PostgreSQL, but was swapped to fit MySQL syntax.

** **
	CREATE SCHEMA pizza_runner;
	
	DROP TABLE IF EXISTS runners;
	CREATE TABLE runners (
	  runner_id INTEGER,
	  registration_date DATE
	);
	
	INSERT INTO runners
	  (runner_id, registration_date)
	VALUES
	  (1, '2021-01-01'),
	  (2, '2021-01-03'),
	  (3, '2021-01-08'),
	  (4, '2021-01-15');

	DROP TABLE IF EXISTS customer_orders;
	CREATE TABLE customer_orders (
	  order_id INTEGER,
	  customer_id INTEGER,
	  pizza_id INTEGER,
	  exclusions VARCHAR(4),
	  extras VARCHAR(4),
	  order_time TIMESTAMP
	);

	INSERT INTO customer_orders
	  (order_id, customer_id, pizza_id, exclusions, extras, order_time)
	VALUES
	  ('1', '101', '1', '', '', '2020-01-01 18:05:02'),
	  ('2', '101', '1', '', '', '2020-01-01 19:00:52'),
	  ('3', '102', '1', '', '', '2020-01-02 23:51:23'),
	  ('3', '102', '2', '', NULL, '2020-01-02 23:51:23'),
	  ('4', '103', '1', '4', '', '2020-01-04 13:23:46'),
	  ('4', '103', '1', '4', '', '2020-01-04 13:23:46'),
	  ('4', '103', '2', '4', '', '2020-01-04 13:23:46'),
	  ('5', '104', '1', 'null', '1', '2020-01-08 21:00:29'),
	  ('6', '101', '2', 'null', 'null', '2020-01-08 21:03:13'),
	  ('7', '105', '2', 'null', '1', '2020-01-08 21:20:29'),
	  ('8', '102', '1', 'null', 'null', '2020-01-09 23:54:33'),
	  ('9', '103', '1', '4', '1, 5', '2020-01-10 11:22:59'),
	  ('10', '104', '1', 'null', 'null', '2020-01-11 18:34:49'),
	  ('10', '104', '1', '2, 6', '1, 4', '2020-01-11 18:34:49');

	DROP TABLE IF EXISTS runner_orders;
	CREATE TABLE runner_orders (
	  order_id INTEGER,
	  runner_id INTEGER,
	  pickup_time VARCHAR(19),
	  distance VARCHAR(7),
	  duration VARCHAR(10),
	  cancellation VARCHAR(23)
	);

	INSERT INTO runner_orders
	  (order_id, runner_id, pickup_time, distance, duration, cancellation)
	VALUES
	  ('1', '1', '2020-01-01 18:15:34', '20km', '32 minutes', ''),
	  ('2', '1', '2020-01-01 19:10:54', '20km', '27 minutes', ''),
	  ('3', '1', '2020-01-03 00:12:37', '13.4km', '20 mins', NULL),
	  ('4', '2', '2020-01-04 13:53:03', '23.4', '40', NULL),
	  ('5', '3', '2020-01-08 21:10:57', '10', '15', NULL),
	  ('6', '3', 'null', 'null', 'null', 'Restaurant Cancellation'),
	  ('7', '2', '2020-01-08 21:30:45', '25km', '25mins', 'null'),
	  ('8', '2', '2020-01-10 00:15:02', '23.4 km', '15 minute', 'null'),
	  ('9', '2', 'null', 'null', 'null', 'Customer Cancellation'),
	  ('10', '1', '2020-01-11 18:50:20', '10km', '10minutes', 'null');

	DROP TABLE IF EXISTS pizza_names;
	CREATE TABLE pizza_names (
	  pizza_id INTEGER,
	  pizza_name TEXT
	);
	
	INSERT INTO pizza_names
	  (pizza_id, pizza_name)
	VALUES
	  (1, 'Meatlovers'),
	  (2, 'Vegetarian');

	DROP TABLE IF EXISTS pizza_recipes;
	CREATE TABLE pizza_recipes (
	  pizza_id INTEGER,
	  toppings TEXT
	);
	
	INSERT INTO pizza_recipes
	  (pizza_id, toppings)
	VALUES
	  (1, '1, 2, 3, 4, 5, 6, 8, 10'),
	  (2, '4, 6, 7, 9, 11, 12');

	DROP TABLE IF EXISTS pizza_toppings;
	CREATE TABLE pizza_toppings (
	  topping_id INTEGER,
	  topping_name TEXT
	);
	
	INSERT INTO pizza_toppings
	  (topping_id, topping_name)
	VALUES
	  (1, 'Bacon'),
	  (2, 'BBQ Sauce'),
	  (3, 'Beef'),
	  (4, 'Cheese'),
	  (5, 'Chicken'),
	  (6, 'Mushrooms'),
	  (7, 'Onions'),
	  (8, 'Pepperoni'),
	  (9, 'Peppers'),
	  (10, 'Salami'),
	  (11, 'Tomatoes'),
	  (12, 'Tomato Sauce');

<div id='clean_data'/>
## Dataset of `Customer_orders`
Looking at the `customer_orders` table below, we can see that there are
- In the `exclusions` column, there are missing/ blank spaces ' ' and null values. 
- In the `extras` column, there are missing/ blank spaces ' ' and null values.

<img width="1063" alt="image" src="https://user-images.githubusercontent.com/81607668/129472388-86e60221-7107-4751-983f-4ab9d9ce75f0.png">

Our course of action to clean the table:
- Create a temporary table with all the columns
- Remove null values in `exlusions` and `extras` columns and replace with blank space ' '

## Cleaned Dataset:

### Cleaned version of customer_orders:

In this table, we remove all the blank spaces and replace all stringed (null/NaN) values as a NULL value for constistency. 

````sql

	SELECT 
	  order_id, 
	  customer_id, 
	  pizza_id,
	CASE
	  WHEN exclusions IS null OR exclusions LIKE 'null' THEN null
	  ELSE exclusions
    END AS exclusions,
  CASE
	  WHEN extras IS NULL or extras LIKE 'null' THEN null
	  ELSE extras
    END AS extras,
	  order_time
INTO #customer_orders_temp 
FROM customer_orders
`````
This is how the clean `customers_orders_temp` table looks like and we will use this table to run all our queries.
<img width="1058" alt="image" src="https://img.upanh.tv/2023/10/30/371501620_1425583731334890_1898848364683823006_n.png">

***
### 🔨 Table: runner_orders

Looking at the `runner_orders` table below, we can see that there are
- In the `exclusions` column, there are missing/ blank spaces ' ' and null values. 
- In the `extras` column, there are missing/ blank spaces ' ' and null values

<img width="1037" alt="image" src="https://user-images.githubusercontent.com/81607668/129472585-badae450-52d2-442e-9d50-e4d0d8fce83a.png">

Our course of action to clean the table:
- In `pickup_time` column, remove nulls and replace with blank space.
- In `distance` column, remove "km" and nulls and replace with null.
- In `duration` column, remove "minutes", "minute" and nulls and replace with null.
- In `cancellation` column, remove NULL and null and and replace with blank null.

````sql
SELECT 
	order_id, 
  runner_id,  
  CASE
	  WHEN pickup_time LIKE 'null' THEN ' '
	  when pickup_time is NULL then ' '
	  ELSE pickup_time
  END AS pickup_time,
  CASE
	  WHEN distance LIKE 'null' THEN null
	  WHEN distance LIKE '%km' THEN TRIM('km' from distance)
	  ELSE distance 
  END AS distance,
  CASE
	  WHEN duration LIKE 'null' THEN null
	  WHEN duration LIKE '%mins' THEN TRIM('mins' from duration)
	  WHEN duration LIKE '%minute' THEN TRIM('minute' from duration)
	  WHEN duration LIKE '%minutes' THEN TRIM('minutes' from duration)
	  ELSE duration
	  END AS duration,
  CASE
	  WHEN cancellation IS NULL or cancellation LIKE 'null' THEN null
	  when cancellation  like 'NaN' then null
	  ELSE cancellation
	  END AS cancellation
INTO #runner_orders_temp1 
FROM runner_orders
`````
Then, we alter the `pickup_time`, `distance` and `duration` columns to the correct data type.

````sql
ALTER TABLE runner_orders_temp1
ALTER COLUMN pickup_time DATETIME

ALTER TABLE runner_orders_temp1
ALTER COLUMN distance FLOAT

ALTER TABLE runner_orders_temp1
ALTER COLUMN duration INT;
````

This is how the clean `runner_orders_temp1` table looks like and we will use this table to run all our queries.

<img width="915" alt="image" src="https://img.upanh.tv/2023/10/30/371525089_314526621322669_1203908455306706998_n.png">

***
## Entity Relationship Diagram View

Original image source: 
https://dbdiagram.io/d/5f3e085ccf48a141ff558487/?utm_source=dbdiagram_embed&utm_medium=bottom_open
>
![image](https://github.com/TheCaptainFalcon/8wksql-cs2/blob/master/ERD-CS2.JPG)

<div id='questions'/>

## Case Study Questions:

### A. Pizza Metrics

#### 1. How many pizzas were ordered?

**Query #1**

    SELECT COUNT(pizza_id) AS pizza_ordered_count
    FROM customer_orders;

| pizza_ordered_count |
| --------------------|
| 14                  |

---

#### 2. How many unique customer orders were made?

**Query #2**

    select count(distinct(order_id)) as unique_customer_orders from customer_orders 

| unique_customer_orders |
| ---------------------- |
| 10                     |

---

#### 3. How many successful orders were delivered by each runner?

**Query #3**

    SELECT 
        runner_id, 
        COUNT(order_id) AS order_count
    FROM runner_orders_temp1
    WHERE distance != 0
    GROUP BY runner_id;

| runner_id | order_count |
| --------- | ----------- |
| 1         | 4           |
| 2         | 3           |
| 3         | 1           |

---


#### 4. How many of each type of pizza was delivered?

**Query #4**

   select 
     pn.pizza_name, 
      count(cuot.pizza_id) as type_of_pizza_delivered 
   from customer_order_temp1 cuot
   join runner_orders_temp1 rot on cuot.order_id = rot.order_id
   join pizza_names pn on cuot.pizza_id = pn.pizza_id
   where rot.distance != 0
   group by pn.pizza_name

| pizza_name | type_of_pizza_delivered | 
| -----------| ------------------------|
| Meat Lovers| 9                       |
| Vegetarian | 3                       |

---

#### 5. How many Vegetarian and Meatlovers were ordered by each customer?

**Query #5**

    WITH pizza_counter_1 AS (
    SELECT 
        customer_id,
        COUNT(pizza_id) AS pizza_count
    FROM customer_orders
    WHERE pizza_id = 1
    GROUP BY customer_id
    ),

    pizza_counter_2 AS (
    SELECT
        customer_id,
        COUNT(pizza_id) AS pizza_count
    FROM customer_orders
    WHERE pizza_id = 2
    GROUP BY customer_id
    )

    SELECT DISTINCT 
        pc1.customer_id,
        pc1.pizza_count AS total_meatlovers,
        pc2.pizza_count AS total_vegetarian
    FROM pizza_counter_1 pc1, pizza_counter_2 pc2
    ORDER BY 1;

| customer_id | total_meatlovers | total_vegetarian |
| ----------- | ---------------- | ---------------- |
| 101         | 2                | 1                |
| 102         | 2                | 1                |
| 103         | 3                | 1                |
| 104         | 3                | 1                |

---

#### 6. What was the maximum number of pizzas delivered in a single order?

**Query #6**

  with pizza_order as (
	  select cuot.order_id, count(cuot.pizza_id) as pizza_delivered
	  from customer_order_temp1 cuot
	  join runner_orders_temp1 rot on cuot.order_id = rot.order_id
	  where distance != 0
	  group by cuot.order_id )

select max(pizza_delivered) as the_maximum_order from pizza_order

| the_maximum_order |
| ----------------- | 
| 3                 | 

---

#### 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

**Query #7**

  select 
	  c.customer_id,
	  sum(case
		  when c.extras like '%' or c.exclusions like '%' then 1
		  else 0
	  end) as at_least_1_change,
	  sum(case
		  when c.extras is null and c.exclusions is null then 1
		  else 0
	  end) as no_change
  from customer_order_temp1 c
  join runner_orders_temp1  r on c.order_id = r.order_id
  where distance != 0
  group by c.customer_id

| customer_id | total_pizzas_with_changes | total_pizzas_without_changes |
| ----------- | ------------------------- | ---------------------------- |
| 101         | 0                         | 2                            |
| 102         | 0                         | 3                            |
| 103         | 3                         | 0                            |
| 104         | 2                         | 1                            |
| 105         | 1                         | 0                            |

---

#### 8. How many pizzas were delivered that had both exclusions and extras?

**Query #8**

select customer_id, count(pizza_id) as pizza_orders from customer_order_temp1
where exclusions like '%' and extras like '%'

| customer_id | pizzas_orders |
| ----------- | --------------|
| 104         | 1             |


Normally, the answer would be 2, but because the question specified 'delivered', we would exclude the orders that were cancelled.

---

#### 9. What was the total volume of pizzas ordered for each hour of the day?

**Query #9**

select 
	DATEPART(HOUR, order_time) as hour_ordered, 
	count(order_id) pizza_order
from customer_order_temp1 
group by DATEPART(HOUR, order_time)

| hour_ordered | pizza_order|
| -----------  | ----       |
| 11           | 1          |
| 13           | 3          |
| 18           | 3          |
| 19           | 1          |
| 21           | 3          |
| 23           | 3          |

At 11h00 am, the total volumn of pizzas is 1, and the similiar total pizzas orders by hour in the dataset. We can identify detail hour to increase the employee working
---

#### 10. What was the volume of orders for each day of the week?

**Query #10**

    WITH orders_by_day AS (
    SELECT
    	COUNT(order_id) AS order_count,
    	WEEKDAY(order_time) AS day
    FROM customer_order_temp1
    GROUP BY day
    ORDER BY day
    )
    
    SELECT	
    	order_count,
        CASE 
		WHEN day = 0 THEN 'Monday'
    		WHEN day = 1 THEN 'Tuesday'
		WHEN day = 2 THEN 'Wednesday'
		WHEN day = 3 THEN 'Thursday'
		WHEN day = 4 THEN 'Friday'
		WHEN day = 5 THEN 'Saturday'
		WHEN day = 6 THEN 'Sunday'
       END AS day
    FROM orders_by_day;

| order_count | day       |
| ----------- | --------- |
| 5           | Wednesday |
| 3           | Thursday  |
| 1           | Friday    |
| 5           | Saturday  |

---

### B. Runner and Customer Experience

#### 1. How many runner signed up for each 1 week period? (ie.week starts 2021-01-01)

**Query #1**

    SELECT
    	COUNT(runner_id) AS runner_count,
        WEEK(registration_date) AS week
    FROM runners
    GROUP BY week;

| runner_count | week |
| ------------ | ---- |
| 1            | 0    |
| 2            | 1    |
| 1            | 2    |

---

#### 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

**Query #2**

    SELECT
        r.runner_id,
        AVG(MINUTE(TIMEDIFF(r.pick_up_time, c.order_time))) AS time_mins
    FROM cust_orders c
    LEFT JOIN runner_orders_post r
    	ON c.order_id = r.order_id
    GROUP BY r.runner_id;

| runner_id | time_mins |
| --------- | --------- |
| 1         | 15.3333   |
| 2         | 23.4000   |
| 3         | 10.0000   |

Note: TIMEDIFF(later time, earlier time)

---

#### 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?

Based on the limited data, there is a possible relationship. 

As the number of pizzas increases per order, the time to prepare an order increases. 
This is shown with order 4 with customer id 103 ranking at the highest in time prep of 29 mins. 
There seems to be a slight variance with orders consisting of 2 pizzas taking anywhere from 15 - 21 mins, while an order of 1 pizza takes as short as 10 minutes.

#### 4. What was the average distance travelled for each customer?

**Query #4**

    SELECT
    	c.customer_id,
        AVG(r.distance_km) AS avg_dist_km
    FROM cust_orders c 
    LEFT JOIN runner_orders_post r
    	ON c.order_id = r.order_id
    GROUP BY c.customer_id;

| customer_id | avg_dist_km |
| ----------- | ----------- |
| 101         | 20.00000    |
| 102         | 16.73333    |
| 103         | 23.40000    |
| 104         | 10.00000    |
| 105         | 25.00000    |

---

#### 5. What was the difference between the longest and shortest delivery times for all orders?

**Query #5**

    SELECT
        MAX(duration_mins) - MIN(duration_mins) AS delivery_time_diff
    FROM runner_orders_post;

| delivery_time_diff |
| ------------------ |
| 30                 |

---

#### 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?

**Query #6**

    SELECT 
    	runner_id,
        AVG(distance_km),
        AVG(duration_mins)
    FROM runner_orders_post
    GROUP BY runner_id;

| runner_id | avg(distance_km) | avg(duration_mins) |
| --------- | ---------------- | ------------------ |
| 1         | 15.85000         | 22.2500            |
| 2         | 23.93333         | 26.6667            |
| 3         | 10.00000         | 15.0000            |

#### PART 2 ANSWER: 

Yes, as expected. 
>
As the distance increases, the time it takes to deliver an order, increases as well.

---
#### 7. What is the successful delivery percentage for each runner?

**Query #7**

    WITH cancellation_counter AS (
    SELECT
    	runner_id,
        CASE
        	WHEN cancellation IS NULL OR cancellation = 'NaN' THEN 1
		ELSE 0
        END AS no_cancellation_count,
        CASE
        	WHEN cancellation IS NOT NULL OR cancellation != 'NaN' THEN 1
		ELSE 0
        END AS cancellation_count
    FROM runner_orders_post
    )
        
    SELECT 
    	runner_id,
        SUM(no_cancellation_count) / (SUM(no_cancellation_count) + SUM(cancellation_count))*100 AS delivery_success_percentage
    FROM cancellation_counter
    GROUP BY runner_id;

| runner_id | delivery_success_percentage |
| --------- | --------------------------- |
| 1         | 100.0000                    |
| 2         | 75.0000                     |
| 3         | 50.0000                     |

---

### C. Ingredient Optimisation

Note: At this point, the pizza_recipes data that was cleaned using Python (Pandas) come into light here.
>
Questions 5 and 6 were obnoxiously difficult, as the trend of utilizing a max of roughly 3 multi-step queries rose to about 7-8.
>
Furthermore, complex nested functions that could normally be used in a programming language could not be used in the same manner in SQL (typical nuances of working in any new language/tool & because SQL is a query language) and each major aggregation/operation had to be split into multiple separate subqueries.
> 
Granted, it is possible that the queries on my end were not optimized to further reduce the number of steps required, however this was also due to working in "chunks" prior to reaching the final solution.
>
By working in this manner, data can be checked to ensure accuracy within the process, rather than after. 
>
In addition, taking into consideration a real life scenario and the concept of future proofing, as data comes in, each chunk allows for direct debugging and modifying to fit new data.

#### 1. What are the standard ingredients for each pizza?

**Query #1**

    SELECT 
    	pizza_id, 
	topping_name
    FROM clean_pizza_recipes cpr
    LEFT JOIN pizza_toppings pt 
        ON cpr.toppings = pt.topping_id;

| pizza_id | topping_name |
| -------- | ------------ |
| 1        | Bacon        |
| 1        | BBQ Sauce    |
| 1        | Beef         |
| 1        | Cheese       |
| 1        | Chicken      |
| 1        | Mushrooms    |
| 1        | Pepperoni    |
| 1        | Salami       |
| 2        | Cheese       |
| 2        | Mushrooms    |
| 2        | Onions       |
| 2        | Peppers      |
| 2        | Tomatoes     |
| 2        | Tomato Sauce |

#### ANSWER:

Pizza 1 (Meatlovers) = [Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami]
>
Pizza 2 (Vegetarian) = [Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce]

---

#### 2. What was the most commonly added extra?

**Query #2**

    SELECT 
    	extras_cleaned as extras,
        COUNT(extras_cleaned) AS extras_counted
    FROM cust_orders
    WHERE extras_cleaned LIKE '%'
    GROUP BY extras_cleaned;

| extras | extras_counted |
| ------ | -------------- |
| 1      | 2              |
| 1, 5   | 1              |
| 1, 4   | 1              |

---

#### 3. What was the most common exclusion?

**Query #3**

    SELECT
    	exclusions_cleaned,
        COUNT(exclusions_cleaned) AS exclusions_count
    FROM cust_orders
    WHERE exclusions_cleaned LIKE '%'
    GROUP BY exclusions_cleaned;

| exclusions_cleaned | exclusions_count |
| ------------------ | ---------------- |
| 4                  | 4                |
| 2, 6               | 1                |

---

#### 4. Generate an order item for each record in the customers_orders table in the format of one of the following:

#### Meat Lovers

** **

    SELECT order_id
    FROM cust_orders
    WHERE pizza_id = 1
    GROUP BY order_id;

| order_id |
| -------- |
| 1        |
| 2        |
| 3        |
| 4        |
| 5        |
| 8        |
| 9        |
| 10       |

---

#### Meat Lovers - Exclude Beef

** **

    SELECT order_id
    FROM cust_orders
    WHERE pizza_id = 1 AND exclusions_cleaned = 3 OR exclusions_cleaned LIKE '%3%'
    GROUP BY order_id

Note: There is no query result, as the dataset does not have any existing exclusions with Beef (3), oddly enough.

#### Meat Lovers - Extra Bacon

** **

    SELECT order_id
    FROM cust_orders
    WHERE pizza_id = 1 AND extras_cleaned = 1 OR extras_cleaned LIKE '%1%'
    GROUP BY order_id

| order_id |
| -------- |
| 5        |
| 7        |
| 9        |
| 10       |

---

#### Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers

** **

    WITH exc_ext_counter AS (
    SELECT
    	order_id,
        CASE
    		WHEN exclusions_cleaned IN (1,4) OR exclusions_cleaned LIKE '%1%' OR exclusions_cleaned LIKE '%4%' THEN 1
		WHEN extras_cleaned IN (6,9) AND extras_cleaned LIKE '%6%' OR extras_cleaned LIKE '%9%' THEN 1
    	END AS exc_ext_count
    FROM cust_orders
    WHERE pizza_id = 1
    )
    
    SELECT order_id
    FROM exc_ext_counter
    WHERE exc_ext_count = 1
    GROUP BY order_id;

| order_id |
| -------- |
| 4        |
| 9        |

---

#### 5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients. For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"

Starting off, what I am trying to do is to use an IF statement to create an indicator for whether a row within the exclusions or extras columns contains a comma delimiter.
This will be used later to extract the values and to determine whether those values 'stack' on top of the standard ingredients of the pizza, thus resulting in a 2x.

It is also worth noting, that while this would not work if there were more than 2 values (within the extras/exclusions column), the proof of concept could stil be utilized to fit this edge case.

** **

    WITH exc_ext_bool AS (
    	SELECT
		order_id,
		pizza_id,
		exclusions_cleaned,
		extras_cleaned,
		IF(LOCATE(',', exclusions_cleaned), TRUE, FALSE) AS exclusions_bool,
		IF(LOCATE(',', extras_cleaned), TRUE, FALSE) AS extras_bool 
    	FROM cust_orders
    ),

| order_id | pizza_id | exclusions_cleaned | extras_cleaned | exclusions_bool | extras_bool |
| -------- | -------- | ------------------ | -------------- | --------------- | ----------- |
| 1        | 1        | null               | null           | 0               | 0           |
| 2        | 1        | null               | null           | 0               | 0           |
| 3        | 1        | null               | null           | 0               | 0           |
| 3        | 2        | null               | null           | 0               | 0           |
| 4        | 1        | 4                  | null           | 0               | 0           |
| 4        | 1        | 4                  | null           | 0               | 0           |
| 4        | 2        | 4                  | null           | 0               | 0           |
| 5        | 1        | null               | 1              | 0               | 0           |
| 6        | 2        | null               | null           | 0               | 0           |
| 7        | 2        | null               | 1              | 0               | 0           |
| 8        | 1        | null               | null           | 0               | 0           |
| 9        | 1        | 4                  | 1, 5           | 0               | 1           |
| 10       | 1        | null               | null           | 0               | 0           |
| 10       | 1        | 2, 6               | 1, 4           | 1               | 1           |

---


#### We now see that the exclusions_bool and the extras_bool act as our indicators. 

1 = TRUE
>
2 = FALSE

#### From those indicators, we use the substring_index to extract the values from the left and right of the comma delimiter for those extras/exclusions that have a 1 value from the columns we created.

In the instances where this applies, another set of columns are used to house the extracted values.

#### Note: The following are part of the existing query, not as separate queries. 

The formatting is displayed in this manner to better showcase everything that is going on in 'chunks.'

** **
    base_exc_ext AS (
    SELECT
    	eeb.order_id,
        eeb.pizza_id,
        pn.pizza_name,
        IF(LOCATE(0, extras_bool), extras_cleaned, NULL) AS base_extras,
        IF(LOCATE(0, exclusions_bool), exclusions_cleaned, NULL) AS base_exclusions,
    	IF(LOCATE(1, exclusions_bool), SUBSTRING_INDEX(exclusions_cleaned, ',', 1), NULL) AS exclusions_1,
        IF(LOCATE(1, exclusions_bool), SUBSTRING_INDEX(exclusions_cleaned, ',', -1), NULL) AS exclusions_2,
        IF(LOCATE(1, extras_bool), SUBSTRING_INDEX(extras_cleaned, ',', 1), NULL) AS extras_1,
        IF(LOCATE(1, extras_bool), SUBSTRING_INDEX(extras_cleaned, ',', -1), NULL) AS extras_2
    FROM exc_ext_bool eeb
    INNER JOIN pizza_names pn 
        ON eeb.pizza_id = pn.pizza_id
    ),

| order_id | pizza_id | pizza_name | base_exclusions | base_extras | exclusions_1 | exclusions_2 | extras_1 | extras_2 |
| -------- | -------- | ---------- | --------------- | ----------- | ------------ | ------------ | -------- | -------- |
| 1        | 1        | Meatlovers |                 |             |              |              |          |          |
| 2        | 1        | Meatlovers |                 |             |              |              |          |          |
| 3        | 1        | Meatlovers |                 |             |              |              |          |          |
| 3        | 2        | Vegetarian |                 |             |              |              |          |          |
| 4        | 1        | Meatlovers | 4               |             |              |              |          |          |
| 4        | 1        | Meatlovers | 4               |             |              |              |          |          |
| 4        | 2        | Vegetarian | 4               |             |              |              |          |          |
| 5        | 1        | Meatlovers |                 | 1           |              |              |          |          |
| 6        | 2        | Vegetarian |                 | null        |              |              |          |          |
| 7        | 2        | Vegetarian |                 | 1           |              |              |          |          |
| 8        | 1        | Meatlovers |                 | null        |              |              |          |          |
| 9        | 1        | Meatlovers | 4               |             |              |              | 1        |  5       |
| 10       | 1        | Meatlovers |                 | null        |              |              |          |          |
| 10       | 1        | Meatlovers |                 |             | 2            |  6           | 1        |  4       |


#### For whatever reason, db-fiddle does a terrible job with displaying the correct differentiation between 'null' and NULL, which explains the blank spaces, which I am not going back to fix everytime. 

The blank spaces should display null in all of them.

---


#### What follows next in the series of nearly identical queries, is where we join the topping_id with the values that we extracted from the previous step. This will help us in determining the topping names associated with the topping_id.

** **
    
    m1 AS (
    SELECT
        order_id, 
        pizza_id, 
        pizza_name, 
        base_exclusions, 
        base_extras, 
        exclusions_1, 
        exclusions_2, 
        extras_1, 
        extras_2, 
        topping_name AS exclusions_1_txt
    FROM base_exc_ext
    LEFT JOIN pizza_toppings pt 
        ON pt.topping_id = exclusions_1
    ),
    
    m2 AS (
    SELECT 
        order_id, 
        pizza_id, 
        pizza_name, 
        base_exclusions, 
        base_extras, 
        exclusions_1, 
        exclusions_2, 
        extras_1, 
        extras_2, 
        exclusions_1_txt, 
        topping_name AS exclusions_2_txt
    FROM m1
    LEFT JOIN pizza_toppings pt 
        ON pt.topping_id = exclusions_2
    ),
    
    m3 AS (
    SELECT 
        order_id, 
        pizza_id, 
        pizza_name, 
        base_exclusions, 
        base_extras, 
        exclusions_1, 
        exclusions_2, 
        extras_1, 
        extras_2, 
        exclusions_1_txt, 
        exclusions_2_txt, 
        topping_name AS extras_1_txt
    FROM m2
    LEFT JOIN pizza_toppings pt 
        ON pt.topping_id = extras_1
    ),
    
    m4 AS (
    SELECT 
        order_id, 
        pizza_id, 
        pizza_name, 
        base_exclusions,
        base_extras,
        exclusions_1,
        exclusions_2,
        extras_1,
        extras_2,
        exclusions_1_txt,
        exclusions_2_txt, 
        extras_1_txt, 
        topping_name AS extras_2_txt
    FROM m3
    LEFT JOIN pizza_toppings pt 
        ON pt.topping_id = extras_2
    ),
    
    m5 AS (
    SELECT 
        order_id, 
        pizza_id, 
        pizza_name, 
        base_exclusions, 
        base_extras, 
        exclusions_1, 
        exclusions_2, 
        extras_1, 
        extras_2, 
        exclusions_1_txt, 
        exclusions_2_txt, 
        extras_1_txt, 
        extras_2_txt, 
        topping_name AS base_exclusions_1
    FROM m4
    LEFT JOIN pizza_toppings pt 
        ON pt.topping_id = base_exclusions
    ),
    
    m6 AS (
    SELECT 
        order_id, 
        pizza_id, 
        pizza_name, 
        base_exclusions, 
        base_extras, 
        exclusions_1, 
        exclusions_2, 
        extras_1, 
        extras_2, 
        exclusions_1_txt, 
        exclusions_2_txt, 
        extras_1_txt, 
        extras_2_txt, 
        base_exclusions_1, 
        topping_name AS base_extras_1
    FROM m5
    LEFT JOIN pizza_toppings pt 
        ON pt.topping_id = base_extras
    ),

#### Here we are seeing whether any ingredients stack and assigning them a 2x through concatination.

** **
    abc_exc_ext AS (
    SELECT 
    	order_id, 
        pizza_id,
    	pizza_name,
        base_exclusions,
        base_extras,
        base_extras_1,
        exclusions_1,
        exclusions_2,
        extras_1,
        extras_2,
    CASE
        WHEN base_exclusions_1 IS NULL AND COALESCE(exclusions_1_txt, exclusions_2_txt) IS NOT NULL THEN CONCAT(exclusions_1_txt, ', ', exclusions_2_txt)
        WHEN base_exclusions_1 IS NOT NULL THEN CONCAT(base_exclusions_1)
        WHEN COALESCE(base_exclusions_1, exclusions_1_txt, exclusions_2_txt) IS NOT NULL THEN CONCAT(base_exclusions_1, ', ', exclusions_1_txt, ', ', exclusions_2_txt)
    END AS exclusions_list,
    CASE
        WHEN base_extras_1 IS NULL AND COALESCE(extras_1_txt, extras_2_txt) IS NOT NULL and pizza_id = 1 AND extras_1 in (1,2,3,4,5,6,8,10) AND extras_2 IN (1,2,3,4,5,6,8,10) THEN CONCAT('2x ', extras_1_txt, ', ', '2x ',extras_2_txt)
        WHEN base_extras_1 IS NOT NULL AND pizza_id = 1 AND base_extras IN (1,2,3,4,5,6,7,10) THEN CONCAT('2x ', base_extras_1)
        WHEN base_extras_1 IS NOT NULL THEN base_extras_1
    END AS extras_list
    FROM m6
    )

| order_id | pizza_id | pizza_name | base_exclusions | base_extras | base_extras_1 | exclusions_1 | exclusions_2 | extras_1 | extras_2 | exclusions_list      | extras_list          |
| -------- | -------- | ---------- | --------------- | ----------- | ------------- | ------------ | ------------ | -------- | -------- | -------------------- | -------------------- |
| 1        | 1        | Meatlovers |                 |             |               |              |              |          |          |                      |                      |
| 2        | 1        | Meatlovers |                 |             |               |              |              |          |          |                      |                      |
| 3        | 1        | Meatlovers |                 |             |               |              |              |          |          |                      |                      |
| 3        | 2        | Vegetarian |                 |             |               |              |              |          |          |                      |                      |
| 4        | 1        | Meatlovers | 4               |             |               |              |              |          |          | Cheese               |                      |
| 4        | 1        | Meatlovers | 4               |             |               |              |              |          |          | Cheese               |                      |
| 4        | 2        | Vegetarian | 4               |             |               |              |              |          |          | Cheese               |                      |
| 5        | 1        | Meatlovers |                 | 1           | Bacon         |              |              |          |          |                      | 2x Bacon             |
| 6        | 2        | Vegetarian |                 | null        |               |              |              |          |          |                      |                      |
| 7        | 2        | Vegetarian |                 | 1           | Bacon         |              |              |          |          |                      | Bacon                |
| 8        | 1        | Meatlovers |                 | null        |               |              |              |          |          |                      |                      |
| 9        | 1        | Meatlovers | 4               |             |               |              |              | 1        |  5       | Cheese               | 2x Bacon, 2x Chicken |
| 10       | 1        | Meatlovers |                 | null        |               |              |              |          |          |                      |                      |
| 10       | 1        | Meatlovers |                 |             |               | 2            |  6           | 1        |  4       | BBQ Sauce, Mushrooms | 2x Bacon, 2x Cheese  |

---

#### This is the final product, where we remove all the excess columns into a summarized query.

** **
    
    SELECT
    	order_id,
    	CASE
		WHEN exclusions_list IS NOT NULL AND extras_list IS NULL THEN CONCAT(pizza_name, ' - ', ' |Exclude| ', exclusions_list)
		WHEN extras_list IS NOT NULL AND exclusions_list IS NULL THEN CONCAT(pizza_name, ' - ', ' |Extras| ' , extras_list)
    		WHEN COALESCE(exclusions_list, extras_list) IS NULL THEN pizza_name
		WHEN COALESCE(exclusions_list, extras_list) IS NOT NULL THEN CONCAT(pizza_name, ' - ', ' |Exclude| ', exclusions_list, ' |Extras| ', extras_list)
    	END AS pizza_type
    FROM abc_exc_ext;

| order_id | pizza_type                                                                |
| -------- | ------------------------------------------------------------------------- |
| 1        | Meatlovers                                                                |
| 2        | Meatlovers                                                                |
| 3        | Meatlovers                                                                |
| 3        | Vegetarian                                                                |
| 4        | Meatlovers -  |Exclude| Cheese                                            |
| 4        | Meatlovers -  |Exclude| Cheese                                            |
| 4        | Vegetarian -  |Exclude| Cheese                                            |
| 5        | Meatlovers -  |Extras| 2x Bacon                                           |
| 6        | Vegetarian                                                                |
| 7        | Vegetarian -  |Extras| Bacon                                              |
| 8        | Meatlovers                                                                |
| 9        | Meatlovers -  |Exclude| Cheese |Extras| 2x Bacon, 2x Chicken              |
| 10       | Meatlovers                                                                |
| 10       | Meatlovers -  |Exclude| BBQ Sauce, Mushrooms |Extras| 2x Bacon, 2x Cheese |

---

#### 6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?

** **
	WITH delivered_bool AS (
	SELECT 
		c.order_id,
		c.pizza_id,
		c.exclusions_cleaned,
		c.extras_cleaned,
		IF(LOCATE(',', exclusions_cleaned), TRUE, FALSE) AS exc_bool,
		IF(LOCATE(',', extras_cleaned), TRUE, FALSE) AS ext_bool
	FROM cust_orders c
	LEFT JOIN runner_orders_post r 
		ON c.order_id = r.order_id
	WHERE duration_mins IS NOT NULL
	),

	exc_ext_list AS (
	SELECT
		order_id,
		pizza_id,
		CASE
			WHEN exc_bool = 0 THEN exclusions_cleaned
		END AS base_exc,
		CASE
			WHEN ext_bool = 0 THEN extras_cleaned
		END AS base_ext,
		CASE
			WHEN exc_bool = 1 THEN SUBSTRING_INDEX(exclusions_cleaned, ',', 1)
		END AS exc_1,
		CASE
			WHEN exc_bool = 1 THEN SUBSTRING_INDEX(exclusions_cleaned, ',', -1)
		END AS exc_2,
		CASE
			WHEN ext_bool = 1 THEN SUBSTRING_INDEX(extras_cleaned, ',', 1)
		END AS ext_1,
		CASE
			WHEN ext_bool = 1 THEN SUBSTRING_INDEX(extras_cleaned, ',', -1)
		END AS ext_2
	FROM delivered_bool
	    )

	SELECT
		order_id,
		pizza_id,
		base_exc,
		base_ext,
		exc_1,
		exc_2,
		ext_1,
		ext_2
	FROM exc_ext_list;


	WITH topping_list AS (
	SELECT
		cr.pizza_id,
		cr.toppings,
		pt.topping_name
	FROM clean_pizza_recipes cr
	LEFT JOIN pizza_toppings pt 
		ON cr.toppings = pt.topping_id
	),

	pizza_counter AS (
	SELECT
		c.order_id,
		c.pizza_id,
		COUNT(c.pizza_id) AS pizza_count
	FROM cust_orders c
	GROUP BY c.pizza_id
	)

	SELECT 
		topping_name, 
		COUNT(topping_name) X pizza_count AS total_topping_count
	FROM topping_list
	INNER JOIN pizza_counter 
		ON topping_list.pizza_id = pizza_counter.pizza_id
	GROUP BY topping_name
	ORDER BY total_topping_count DESC



### D. Pricing and Ratings

#### 1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?

**Query #1**

    SELECT
	SUM(CASE
		WHEN pizza_id = 1 THEN 12
		WHEN pizza_id = 2 THEN 10
    	END) AS pizza_cost
    FROM cust_orders;

| pizza_cost |
| ---------- |
| 160        |

---

#### 2. What if there was an additional $1 charge for any pizza extras?

Here, we are basing this off the same approach used in questions 6/7 (last section), where we use a locate the comma delimiter as part of a boolean based column.
We also use a case statement that reflects the standard costs of a meat lover and vegetarian pizza from the previous question. 

** **

    WITH ex_bool_list AS (
    SELECT
    	order_id,
        pizza_id,
        extras_cleaned,
        IF(LOCATE(',', extras_cleaned), TRUE, FALSE) AS ex_bool,
        CASE
    		WHEN pizza_id = 1 THEN 12
            	WHEN pizza_id = 2 THEN 10
    	END AS base_pizza_cost
    FROM cust_orders
    ),



| order_id | pizza_id | extras_cleaned | ex_bool | base_pizza_cost |
| -------- | -------- | -------------- | ------- | --------------- |
| 1        | 1        |                | 0       | 12              |
| 2        | 1        |                | 0       | 12              |
| 3        | 1        |                | 0       | 12              |
| 3        | 2        |                | 0       | 10              |
| 4        | 1        |                | 0       | 12              |
| 4        | 1        |                | 0       | 12              |
| 4        | 2        |                | 0       | 10              |
| 5        | 1        | 1              | 0       | 12              |
| 6        | 2        | null           | 0       | 10              |
| 7        | 2        | 1              | 0       | 10              |
| 8        | 1        | null           | 0       | 12              |
| 9        | 1        | 1, 5           | 1       | 12              |
| 10       | 1        | null           | 0       | 12              |
| 10       | 1        | 1, 4           | 1       | 12              |

---

#### Again, we use the 1 values generated from the previous query to extract the values to the left and right of the comma delimiter to a separate column.

** **

    ext_list AS (
    SELECT
    	order_id,
        base_pizza_cost,
        CASE
    		WHEN ex_bool = 0 AND extras_cleaned IS NOT NULL THEN extras_cleaned
    	END AS base_extras_cleaned,
        CASE
    		WHEN ex_bool = 1 THEN SUBSTRING_INDEX(extras_cleaned, ',', 1)
    	END AS ex_1,
        CASE
    		WHEN ex_bool = 1 THEN SUBSTRING_INDEX(extras_cleaned, ',', -1)
        END AS ex_2
    FROM ex_bool_list
    ),

| order_id | base_pizza_cost | base_extras_cleaned | ex_1 | ex_2 |
| -------- | --------------- | ------------------- | ---- | ---- |
| 1        | 12              |                     |      |      |
| 2        | 12              |                     |      |      |
| 3        | 12              |                     |      |      |
| 3        | 10              |                     |      |      |
| 4        | 12              |                     |      |      |
| 4        | 12              |                     |      |      |
| 4        | 10              |                     |      |      |
| 5        | 12              | 1                   |      |      |
| 6        | 10              | null                |      |      |
| 7        | 10              | 1                   |      |      |
| 8        | 12              | null                |      |      |
| 9        | 12              |                     | 1    |  5   |
| 10       | 12              | null                |      |      |
| 10       | 12              |                     | 1    |  4   |

---

#### At this point, we accumulate all the separated values to the associated standard pizza costs.

** **
    total_cost_list AS (
    SELECT
    	order_id,
        CASE
		WHEN COALESCE(base_extras_cleaned, ex_1, ex_2) IS NULL THEN base_pizza_cost
		WHEN base_extras_cleaned IS NOT NULL AND COALESCE(ex_1, ex_2) IS NULL THEN 1 + base_pizza_cost
		WHEN base_extras_cleaned IS NULL AND COALESCE(ex_1, ex_2) IS NOT NULL THEN 2 + base_pizza_cost
    	END AS total_pizza_cost
    FROM ext_list
    )

| order_id | total_pizza_cost |
| -------- | ---------------- |
| 1        | 12               |
| 2        | 12               |
| 3        | 12               |
| 3        | 10               |
| 4        | 12               |
| 4        | 12               |
| 4        | 10               |
| 5        | 13               |
| 6        | 11               |
| 7        | 11               |
| 8        | 13               |
| 9        | 14               |
| 10       | 13               |
| 10       | 14               |

---

#### Aggregate the sum of total pizza costs in each row.

** **

    SELECT SUM(total_pizza_cost)
    FROM total_cost_list;

| sum(total_pizza_cost) |
| --------------------- |
| 169                   |

---

#### Add cheese is $1 extra.

This is built on top of the previous answer.
The biggest difference is the conditional statement added for 4 (for cheese)

** **

    total_cost_list AS (
    SELECT
        order_id,
        CASE
            WHEN COALESCE(base_extras_cleaned, ex_1, ex_2) IS NULL THEN base_pizza_cost
            WHEN base_extras_cleaned IS NOT NULL AND COALESCE(ex_1, ex_2) IS NULL THEN 1 + base_pizza_cost
            WHEN base_extras_cleaned IS NULL AND ex_1 is not null AND ex_2 != 4 THEN 2 + base_pizza_cost
            WHEN base_extras_cleaned IS NULL AND ex_1 is not null AND ex_2 = 4 THEN 3 + base_pizza_cost
        END AS total_pizza_cost
    FROM ext_list
    )

| order_id | total_pizza_cost |
| -------- | ---------------- |
| 1        | 12               |
| 2        | 12               |
| 3        | 12               |
| 3        | 10               |
| 4        | 12               |
| 4        | 12               |
| 4        | 10               |
| 5        | 13               |
| 6        | 11               |
| 7        | 11               |
| 8        | 13               |
| 9        | 14               |
| 10       | 13               |
| 10       | 15               |

---

** **

    SELECT SUM(total_pizza_cost)
    FROM total_cost_list;

| sum(total_pizza_cost) |
| --------------------- |
| 170                   |

---

Skipped 3-4, as its very subjective and theoretical.
Only really interested in the puzzle solving questions.

#### 5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?

We use the first part from the answer in question 1.

** **
    WITH base_pizza_cost AS (
    SELECT 
        SUM(CASE
            WHEN pizza_id = 1 THEN 12
            WHEN pizza_id = 2 THEN 10
        END) AS pizza_cost
    FROM cust_orders
    ),

| pizza_cost |
| ---------- |
| 160        |

---

#### When a distance_km is labeled as null, it means that the delivery was cancelled, so we identify the orders that weren't cancelled and multiply it by 0.30 per the question.

** **
    runner_cost_list AS (
    SELECT distance_km,
        CASE
            WHEN distance_km IS NOT NULL THEN distance_km*0.30
        END AS runner_cost
    FROM runner_orders_post
    ),

| distance_km | runner_cost |
| ----------- | ----------- |
| 20.0        | 6.000       |
| 20.0        | 6.000       |
| 13.4        | 4.020       |
| 23.4        | 7.020       |
| 10.0        | 3.000       |
|             |             |
| 25.0        | 7.500       |
| 23.4        | 7.020       |
|             |             |
| 10.0        | 3.000       |

---

#### Here is the total sum of the runner costs.

** **
    runner_cost_total AS (
    SELECT SUM(runner_cost) AS total_runner_cost
    FROM runner_cost_list
    )

| total_runner_cost |
| ----------------- |
| 43.560            |

---

#### We then subtract the pizza_costs (which represent the profits) with the previous total query to account for the runner expenses, to get the true profit.

** **
    SELECT
        pizza_cost - total_runner_cost
    FROM base_pizza_cost, runner_cost_total


| pizza_cost - total_runner_cost  |
| ------------------------------- |
| 116.440                         |

---

<div id='bonus'/>

### E. Bonus Question

#### If Danny wants to expand his range of pizzas - how would this impact the existing data design? 

Because the pizza recipes table was modified to reflect foreign key designation for each topping linked to the base pizza, the pizza_id will have multiple 3s and align with the standard toppings (individually) within the toppings column.

In addition, because the data type was casted to an int to take advantage of numerical functions, insertion of data would not affect the existing data design, unlike the original dangerous approach of comma separated values in a singular row (list) 
