# SQL Case Study #2 (Week 2) - Pizza Runner

Note: All source material and respected credit is from: https://8weeksqlchallenge.com/

Online SQL instance used to test queries: https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65
>
<img src="https://user-images.githubusercontent.com/81607668/127271856-3c0d5b4a-baab-472c-9e24-3c1e3c3359b2.png" alt="Image" width="500" height="520">

## Table of Contents:
1. [Dataset Structure](#data)
2. [Cleaned Dataset](#clean_data)
3. [Entity Relationship Diagram](#diagram)
4. [Case Study Questions + Answers](#questions)





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
<div id='diagram'/>
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

select DATEPART(WEEK, registration_date) as runners_signed, count(runner_id) as count_runner
from runners
where year(registration_date) = 2021
group by DATEPART(WEEK, registration_date)

| runner_signed | count_runner |
| ------------  | ---- 	       |
| 1             | 1            |
| 2             | 1            |
| 3             | 2            |

---

#### 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

**Query #2**

with avg_time as (
select 
	c.order_id,
	ru.runner_id,
	c.order_time,
	r.pickup_time,
	DATEDIFF(MINUTE, c.order_time, r.pickup_time) as minute_order

from customer_order_temp1 c
join runner_orders_temp1 r on c.order_id = r.order_id
join runners ru on r.runner_id = ru.runner_id
where distance != 0
group by c.order_id, ru.runner_id, c.order_time,r.pickup_time )

select 
	runner_id,
	avg(minute_order) as minute_order
from avg_time
where minute_order > 0
group by runner_id

| runner_id | minute_order |
| --------- | ------------ |
| 1         | 10           |
| 2         | 30           |
| 3         | 10           |

Note: TIMEDIFF(later time, earlier time)

---

#### 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?

Based on the limited data, there is a possible relationship. 

As the number of pizzas increases per order, the time to prepare an order increases. 
This is shown with order 4 with customer id 103 ranking at the highest in time prep of 29 mins. 
There seems to be a slight variance with orders consisting of 2 pizzas taking anywhere from 15 - 21 mins, while an order of 1 pizza takes as short as 10 minutes.

#### 4. What was the average distance travelled for each customer?

**Query #4**

select c.customer_id, round(avg(r.distance),2) as average_distance
from customer_order_temp1 c
join runner_orders_temp1 r on c.order_id = r.order_id
where distance != 0
group by c.customer_id

| customer_id | average_distance |
| ----------- | -----------------|
| 101         | 20    	 	 |
| 102         | 16.73    	 |
| 103         | 23.4    	 |
| 104         | 10    	 	 |
| 105         | 25   	 	 |

---

#### 5. What was the difference between the longest and shortest delivery times for all orders?

**Query #5**

select order_id, duration from runner_orders_temp1
where duration != 0

| order_id | duration |
| -------- |--------- |
| 1        | 32       |
| 2        | 27       |
| 3        | 20       |
| 4        | 40       |
| 5        | 15       |
| 7        | 25       |
| 8        | 15       |
| 10       | 10       |

Then I'll type to find max and min for range of dilivery time

with duration1 as (
select order_id, duration from runner_orders_temp1
where duration != 0 )
select (max(duration) - min(duration)) as delivery_time_difference
from duration1

| delivery_time_difference |
| ------------------------ |	
|30			   | 
---

#### 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?

**Query #6**

SELECT 
    	runner_id,
        round(AVG(distance),2) as distance,
        AVG(duration) as duration
FROM runner_orders_temp1
GROUP BY runner_id;

| runner_id | distance_km | avg(duration_mins) |
| --------- | ------------| ------------------ |
| 1         | 15.85       | 22                 |
| 2         | 23.93       | 26                 |
| 3         | 10          | 15                 |

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
        	WHEN cancellation IS NULL THEN 1
		ELSE 0
        END AS no_cancellation_count,
        CASE
        	WHEN cancellation IS NOT NULL THEN 1
		ELSE 0
        END AS cancellation_count
    FROM runner_orders_temp1
    )
        
    SELECT 
    	runner_id,
		SUM(no_cancellation_count) / (SUM(no_cancellation_count) + SUM(cancellation_count))*100 AS delivery_success_percentage
    FROM cancellation_counter
	group by runner_id

| runner_id | delivery_success_percentage |
| --------- | --------------------------- |
| 1         | 100.0000                    |
| 2         | 75.0000                     |
| 3         | 50.0000                     |

---

### C. Ingredient Optimisation
Furthermore, complex nested functions that could normally be used in a programming language could not be used in the same manner in SQL (typical nuances of working in any new language/tool & because SQL is a query language) and each major aggregation/operation had to be split into multiple separate subqueries.
> 
Granted, it is possible that the queries on my end were not optimized to further reduce the number of steps required, however this was also due to working in "chunks" prior to reaching the final solution.
>
By working in this manner, data can be checked to ensure accuracy within the process, rather than after. 
>
In addition, taking into consideration a real life scenario and the concept of future proofing, as data comes in, each chunk allows for direct debugging and modifying to fit new data.

#### 1. What are the standard ingredients for each pizza?

**Query #1**

with split_value as (
SELECT *  
FROM pizza_recipes  
    CROSS APPLY STRING_SPLIT(toppings, ',')
)
select count(distinct(pizza_id)) as pizza_id, pt.topping_name from split_value sv
join pizza_toppings pt on sv.[value] = pt.topping_id
group by pt.topping_name
order by pizza_id

| pizza_id | topping_name |
| -------- | ------------ |
| 1        | BBQ Sauce    |
| 1        | Beef         |
| 1        | Chicken      |
| 1        | Pepperoni    |
| 1        | Salami       |
| 1        | Tomatoes     |
| 1        | Onions       |
| 1        | Peppers      |
| 2        | Mushrooms    |
| 2        | Cheese 	  |

#### ANSWER:

Pizza 1 (Meatlovers) = [ BBQ Sauce, Beef, Chicken, Pepperoni, Salami, Tomatoes, Onions, Peppers]
>
Pizza 2 (Vegetarian) = [Cheese, Mushrooms]

---

#### 2. What was the most commonly added extra?

**Query #2**

select extras, count(extras) as count_extras from customer_order_temp1
where extras like '%'
group by extras


| extras | extras_counted |
| ------ | -------------- |
| 1      | 2              |
| 1, 4   | 1              |
| 1, 5   | 1              |

---

#### 3. What was the most common exclusion?

**Query #3**

SELECT
    	exclusions,
        COUNT(exclusions) AS exclusions_count
FROM customer_order_temp1
WHERE exclusions LIKE '%'
GROUP BY exclusions;

| exclusions_cleaned | exclusions_count |
| ------------------ | ---------------- |
| 4                  | 4                |

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
FROM customer_order_temp1
WHERE pizza_id = 1 AND exclusions = 3 OR exclusions LIKE '%3%'
GROUP BY order_id

Note: There is no query result, as the dataset does not have any existing exclusions with Beef (3), oddly enough.

#### Meat Lovers - Extra Bacon

** **

SELECT order_id
FROM customer_order_temp1
WHERE pizza_id = 1 AND extras = 1 OR extras LIKE '%1%'
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

with split_extras as (
select * 
from customer_order_temp1
	cross apply string_split(extras, ',')
),
take_exrea as (
select 
	order_id,
	case
		when exclusions in (1 , 4) or exclusions like '%1' or exclusions like '%4' then 1
		when exclusions in (6 , 9) or exclusions like '%6' or exclusions like '%9' then 1
	end as exc_ext_count
from split_extras
where pizza_id = 1
)

select order_id from take_exrea
where exc_ext_count = 1
group by order_id

| order_id |
| -------- |
| 9        |

---

### D. Pricing and Ratings

#### 1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?

**Query #1**

    SELECT
	SUM(CASE
		WHEN pizza_id = 1 THEN 12
		WHEN pizza_id = 2 THEN 10
    	END) AS pizza_cost
    FROM customer_order_temp1;

| pizza_cost |
| ---------- |
| 160        |

---

