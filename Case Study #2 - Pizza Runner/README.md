# Case Study #2 - Pizza Runner

<img width="500" alt="ERD2" src="https://github.com/SophiaTulip/8_Week_SQL_Challenge/assets/157422079/8604923e-ad4c-449b-a4ed-ef281ced71ed">

## Table of Contents

- [Business Task](#business-task)
- [Case Study Topics](#case-study-topics)
- [ERD](#erd)
- [Questions and Solutions](#questions-and-solutions)

Original source for this case study can be found [here](https://8weeksqlchallenge.com/case-study-2/).
## Business Task

Enhance efficiency in Danny's Pizza Runner's operations by cleaning the data and applying fundamental calculations using the provided dataset.

## Case Study Topics

- Common Table Expressions
- Group By Aggregates
- Table Joins
- String Transformations
- Dealing With Null Values
- Regular Expressions

## ERD

<img width="500" alt="ERD2" src="https://github.com/SophiaTulip/8_Week_SQL_Challenge/assets/157422079/c7479918-1c8c-4927-b968-178a0af36153">

## Questions and Solutions

Recreated the dataset in MySQL to solve this week's questions.

- [A. Pizza Metrics](#a-pizza-metrics)
- [B. Runner and Customer Experience](#b-runner-and-customer-experience)
- [C. Ingredient Optimisation](#c-ingredient-optimisation)

## Cleaning the Data

```sql
-- Created a table to remove null values
CREATE TABLE customer_orders_cln AS
SELECT 
  order_id, 
  customer_id, 
  pizza_id, 
  CASE
    WHEN exclusions IS null OR exclusions LIKE 'null' THEN ''
    ELSE exclusions
  END AS exclusions,
  CASE
    WHEN extras IS NULL or extras LIKE 'null' THEN ''
    ELSE extras
  END AS extras,
  order_time
FROM pizza_runner.customer_orders;

-- Noticed the date was 2020 when it should be 2021
UPDATE customer_orders_cln
SET order_time = date_add(order_time, INTERVAL 1 YEAR);

-- Created a table to set real null values in order to change datatypes, remove other null values and trim units of measurement, moved the units into the column header 
CREATE TABLE runner_orders_cln AS
SELECT 
  order_id, 
  runner_id,  
  CASE
    WHEN pickup_time = 'null' THEN NULL
    ELSE pickup_time
  END AS pickup_time,
  CASE
    WHEN distance LIKE 'null' THEN NULL
    WHEN distance LIKE '%km' THEN TRIM('km' from distance)
    ELSE distance 
  END AS distance_km,
  CASE
    WHEN duration LIKE 'null' THEN NULL
    WHEN duration LIKE '%mins' THEN TRIM('mins' from duration)
    WHEN duration LIKE '%minute' THEN TRIM('minute' from duration)
    WHEN duration LIKE '%minutes' THEN TRIM('minutes' from duration)
    ELSE duration
  END AS duration_mins,
  CASE
    WHEN cancellation IS NULL or cancellation LIKE 'null' THEN ''
    ELSE cancellation
  END AS cancellation
FROM pizza_runner.runner_orders;

-- Changed columns from varchar datatype to following
ALTER TABLE runner_orders_cln
MODIFY COLUMN pickup_time TIMESTAMP,
MODIFY COLUMN distance_km FLOAT,
MODIFY COLUMN duration_mins INT;

-- Noticed the date was 2020 when it should be 2021
UPDATE runner_orders_cln
SET pickup_time = date_add(pickup_time, INTERVAL 1 YEAR);
```

## A. Pizza Metrics

**1. How many pizzas were ordered?**

```sql
SELECT
  count(order_id) AS pizza_tot
FROM pizza_runner.customer_orders_cln;
```
**Answer:**
| pizza_tot|
|---|
| 14 |
<br>

**2. How many unique customer orders were made?**
```sql
SELECT
  count(DISTINCT customer_id) AS unique_customers
FROM pizza_runner.customer_orders_cln;
```
**Answer:**
| unique_customers |
|---|
| 5 |
<br>

**3. How many successful orders were delivered by each runner?**

```sql
SELECT
  runner_id,
  count(order_id) AS successful_orders
FROM pizza_runner.runner_orders_cln
WHERE cancellation NOT LIKE '%cancellation%'
GROUP BY runner_id
ORDER BY runner_id ASC;
```
**Answer:**
| runner_id | successful_orders |
|---|---|
| 1 | 4 |
| 2 | 3 |
| 3 | 1 |
<br>

**4. How many of each type of pizza was delivered?**

```sql
SELECT
  pn.pizza_name,
  count(co.pizza_id) AS pizza_type_tot
FROM pizza_runner.customer_orders_cln AS co
JOIN pizza_runner.pizza_names AS pn
  ON co.pizza_id = pn.pizza_id
JOIN pizza_runner.runner_orders_cln AS ro
  ON co.order_id = ro.order_id
WHERE ro.cancellation NOT LIKE '%cancellation%'
GROUP BY pn.pizza_name;
```
**Answer:**
| pizza_name | pizza_type_tot |
|---|---|
| Meatlovers | 9 |
| Vegetarian | 3 |
<br>

**5. How many Vegetarian and Meatlovers were ordered by each customer?**

```sql
SELECT
  co.customer_id,
  pn.pizza_name,
  count(co.pizza_id) AS pizza_type_tot
FROM pizza_runner.customer_orders_cln AS co
JOIN pizza_runner.pizza_names AS pn
  ON co.pizza_id = pn.pizza_id
GROUP BY co.customer_id, pn.pizza_name
ORDER BY co.customer_id ASC;
```
**Answer:**
| customer_id | pizza_name | pizza_type_tot |
|---|---|---|
| 101 | Meatlovers | 2 |
| 101 | Vegetarian | 1 |
| 102 | Meatlovers | 2 |
| 102 | Vegetarian | 1 |
| 103 | Meatlovers | 3 |
| 103 | Vegetarian | 1 |
| 104 | Meatlovers | 3 |
| 105 | Vegetarian | 1 |
<br>

**6. What was the maximum number of pizzas delivered in a single order?**

```sql
SELECT
  count(co.pizza_id) AS max_pizzas_del
FROM pizza_runner.customer_orders_cln AS co
JOIN pizza_runner.runner_orders_cln AS ro
  ON co.order_id = ro.order_id
WHERE ro.cancellation NOT LIKE '%cancellation%'
GROUP BY co.order_id
ORDER BY max_pizzas_del DESC
LIMIT 1;
```
**Answer:**
| max_pizzas_del |
|---|
| 3 |
<br>

**7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?**

```sql
SELECT
  co.customer_id,
  count(CASE
    WHEN co.exclusions NOT LIKE ''
    OR co.extras NOT LIKE '' THEN 1
    ELSE NULL
  END) AS tot_changes,
  count(CASE
    WHEN co.exclusions LIKE ''
    AND co.extras LIKE '' THEN 1
    ELSE NULL
  END) AS no_changes
FROM pizza_runner.customer_orders_cln AS co
JOIN pizza_runner.runner_orders_cln AS ro
  ON co.order_id = ro.order_id
WHERE ro.cancellation NOT LIKE '%cancellation%'
GROUP BY co.customer_id
ORDER BY co.customer_id ASC;
```
**Answer:**
| customer_id | tot_changes | no_changes |
|---|---|---|
| 101 | 0 | 2 |
| 102 | 0 | 3 |
| 103 | 3 | 0 |
| 104 | 2 | 1 |
| 105 | 1 | 0 |
<br>

**8. How many pizzas were delivered that had both exclusions and extras?**

```sql
SELECT
  count(CASE
    WHEN co.exclusions NOT LIKE ''
    AND co.extras NOT LIKE '' THEN 1
    ELSE NULL
  END) AS pizzas_del
FROM pizza_runner.customer_orders_cln AS co
JOIN pizza_runner.runner_orders_cln AS ro
  ON co.order_id = ro.order_id
WHERE ro.cancellation NOT LIKE '%cancellation%';
```
**Answer:**
| pizzas_del |
|---|
| 1 | 
<br>

**9. What was the total volume of pizzas ordered for each hour of the day?**

```sql
SELECT
  DATE_FORMAT(co.order_time, '%H') AS order_hour,
  count(pizza_id) AS tot_volume
FROM pizza_runner.customer_orders_cln AS co
GROUP BY order_hour
ORDER BY order_hour ASC;
```
**Answer:**
| order_hour | tot_volume |
|---|---|
| 11 | 1 |
| 13 | 3 |
| 18 | 3 |
| 19 | 1 |
| 21 | 3 |
| 23 | 3 |
<br>

**10. What was the volume of orders for each day of the week?**

```sql
SELECT
  DAYNAME(co.order_time) AS day_of_week,
  count(pizza_id) AS tot_volume
FROM pizza_runner.customer_orders_cln AS co
GROUP BY day_of_week
ORDER BY DAYOFWEEK(co.order_time);
```
**Answer:**
| day_of_week | tot_volume |
|---|---|
| Sunday | 1 |
| Monday | 5 |
| Friday | 5 |
| Saturday | 3 |
<br>

## B. Runner and Customer Experience

**1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)**

```sql
SELECT 
  DATE_ADD('2021-01-01', INTERVAL (FLOOR(DATEDIFF(r.registration_date, '2021-01-01') / 7) * 7) DAY) AS week_of,
  COUNT(*) AS sign_ups
FROM pizza_runner.runners AS r
GROUP BY FLOOR(DATEDIFF(r.registration_date, '2021-01-01') / 7)
ORDER BY week_of;
```
**Answer:**
| week_of | sign_ups |
|---|---|
| 2021-01-01 | 2 |
| 2021-01-08 | 1 |
| 2021-01-15 | 1 |
<br>

**2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?**

```sql
SELECT
  ro.runner_id,
  ROUND(AVG(TIMESTAMPDIFF(MINUTE, co.order_time, ro.pickup_time))) AS avg_time
FROM pizza_runner.runner_orders_cln AS ro
JOIN pizza_runner.customer_orders_cln AS co
  ON ro.order_id = co.order_id
GROUP BY ro.runner_id;
```
**Answer:**
| runner_id | avg_time |
|---|---|
| 1 | 15 |
| 2 | 23 |
| 3 | 10 |
<br>

**3. Is there any relationship between the number of pizzas and how long the order takes to prepare?**

```sql
SELECT
num_of_pizzas,
ROUND(AVG(avg_time)) AS avg_prep_time
FROM (
  SELECT
    co.order_id,
    COUNT(co.pizza_id) AS num_of_pizzas,
    TIMESTAMPDIFF(MINUTE, co.order_time, ro.pickup_time) AS avg_time
  FROM pizza_runner.runner_orders_cln AS ro
  JOIN pizza_runner.customer_orders_cln AS co
    ON ro.order_id = co.order_id
  WHERE ro.cancellation NOT LIKE '%cancellation%'
  GROUP BY co.order_id
) AS answerB3
GROUP BY num_of_pizzas;
```
**Answer:**
| num_of_pizzas | avg_prep_time |
|---|---|
| 1 | 12 |
| 2 | 18 |
| 3 | 29 |
<br>

**4. What was the average distance travelled for each customer?**

```sql
SELECT
  co.customer_id,
  ROUND(AVG(ro.distance_km), 2) AS avg_distance_km
FROM pizza_runner.runner_orders_cln AS ro
JOIN pizza_runner.customer_orders_cln AS co
  ON ro.order_id = co.order_id
GROUP BY co.customer_id
ORDER BY customer_id ASC;
```
**Answer:**
| customer_id | avg_distance_km |
|---|---|
| 101 | 20 |
| 102 | 16.73 |
| 103 | 23.4 |
| 104 | 10 |
| 105 | 25 |
<br>

**5. What was the difference between the longest and shortest delivery times for all orders?**

```sql
SELECT
  MAX(ro.duration_mins) - MIN(ro.duration_mins) AS difference_mins
FROM pizza_runner.runner_orders_cln AS ro
WHERE ro.cancellation NOT LIKE '%cancellation%';
```
**Answer:**
| difference_mins |
|---|
| 30 |
<br>

**6. What was the average speed for each runner for each delivery and do you notice any trend for these values?**

```sql
SELECT
  ro.runner_id,
ROUND(ro.distance_km / (ro.duration_mins / 60), 2) AS avg_speed_kph
FROM pizza_runner.runner_orders_cln AS ro
WHERE ro.cancellation NOT LIKE '%cancellation%';
```
**Answer:**
| runner_id | avg_speed_kph |
|---|---|
| 1 | 37.5 |
| 1 | 44.44 |
| 1 | 40.2 |
| 2 | 35.1 |
| 3 | 40 |
| 2 | 60 |
| 2 | 93.6 |
| 1 | 60 |
<br>

**7. What is the successful delivery percentage for each runner?**

```sql
SELECT
runner_id,
ROUND(AVG(success) * 100) AS percentage
FROM (
  SELECT
    ro.runner_id,
    CASE
      WHEN ro.duration_mins NOT LIKE '' THEN 1
      ELSE 0
    END AS success
  FROM pizza_runner.runner_orders_cln AS ro
) AS answerB7
GROUP BY runner_id;
```
**Answer:**
| runner_id | percentage |
|---|---|
| 1 | 100 |
| 2 | 75 |
| 3 | 50 |
<br>

## C. Ingredient Optimisation

**1. What are the standard ingredients for each pizza?**

```sql
WITH pizza_recipes_temp AS (
SELECT 
  pizza_id,
  SUBSTRING_INDEX(SUBSTRING_INDEX(toppings, ',', n), ',', -1) AS topping_id
FROM pizza_runner.pizza_recipes
JOIN (SELECT 1 AS n UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5 UNION ALL SELECT 6 UNION ALL SELECT 7 UNION ALL SELECT 8) AS numbers
  ON CHAR_LENGTH(toppings) - CHAR_LENGTH(REPLACE(toppings, ',', '')) >= n - 1
ORDER BY pizza_id)

SELECT
  pn.pizza_name,
  pt.topping_name
FROM pizza_runner.pizza_names AS pn
JOIN pizza_recipes_temp AS pr
  ON pn.pizza_id = pr.pizza_id
JOIN pizza_runner.pizza_toppings AS pt
  ON pr.topping_id = pt.topping_id
ORDER BY pn.pizza_name, pt.topping_name;
```
**Answer:**
| pizza_name | topping_name |
|---|---|
| Meatlovers | Bacon |
| Meatlovers | BBQ Sauce |
| Meatlovers | Beef |
| Meatlovers | Cheese |
| Meatlovers | Chicken |
| Meatlovers | Mushrooms |
| Meatlovers | Pepperoni |
| Meatlovers | Salami |
| Vegetarian | Cheese |
| Vegetarian | Mushrooms |
| Vegetarian | Onions |
| Vegetarian | Peppers |
| Vegetarian | Tomato Sauce |
| Vegetarian | Tomatoes |
<br>

**2. What was the most commonly added extra?**

```sql
WITH extras_temp AS (
SELECT
  order_id,
  TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(extras, ',', n), ',', -1)) AS extras
FROM pizza_runner.customer_orders_cln
JOIN (SELECT 1 AS n UNION ALL SELECT 2) AS numbers
  ON CHAR_LENGTH(extras) - CHAR_LENGTH(REPLACE(extras, ',', '')) >= n - 1
WHERE TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(extras, ',', numbers.n), ',', -1)) != '')
  
SELECT
  pt.topping_name AS extras_name,
  COUNT(extras) as total
FROM extras_temp
JOIN pizza_runner.pizza_toppings AS pt
  ON extras_temp.extras = pt.topping_id
WHERE extras IS NOT NULL
GROUP BY extras
ORDER BY total DESC;
```
**Answer:**
| extras_name | total |
|---|---|
| Bacon | 4 |
| Chicken | 1 |
| Cheese | 1 |
<br>

**3. What was the most common exclusion?**

```sql
WITH exclusions_temp AS (
SELECT
  order_id,
  TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(exclusions, ',', n), ',', -1)) AS exclusions
FROM pizza_runner.customer_orders_cln
JOIN (SELECT 1 AS n UNION ALL SELECT 2) AS numbers
  ON CHAR_LENGTH(exclusions) - CHAR_LENGTH(REPLACE(exclusions, ',', '')) >= n - 1
WHERE TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(exclusions, ',', numbers.n), ',', -1)) != '')
  
SELECT
   pt.topping_name AS exclusions_name,
  COUNT(exclusions) as total
FROM exclusions_temp
JOIN pizza_runner.pizza_toppings AS pt
  ON exclusions_temp.exclusions = pt.topping_id
WHERE exclusions IS NOT NULL
GROUP BY exclusions
ORDER BY total DESC;
```
**Answer:**
| exclusions_name | total |
|---|---|
| Cheese | 4 |
| Mushrooms | 1 |
| BBQ Sauce | 1 |
<br>

**4. Generate an order item for each record in the customers_orders table in the format of one of the following:<br>
◽ Meat Lovers<br>
◽ Meat Lovers - Exclude Beef<br>
◽ Meat Lovers - Extra Bacon<br>
◽ Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers**

```sql
-- Creating a row ID number and a column that identifies the name for each type of pizza

SET @row_number = 0;

CREATE TABLE customer_orders_cln_2 AS
SELECT
  (@row_number:=@row_number + 1) AS row_num,
  customer_orders_cln.*,
  CASE
    WHEN pizza_id = 1 THEN 'Meatlovers'
	WHEN pizza_id = 2 THEN 'Vegetarian'
  END AS pizza_name
FROM customer_orders_cln;
    
-- Converting the topping IDs in the 'exclusions' and 'extras' columns to names then merging them together with the pizza name.
  
WITH RECURSIVE SplitExtras AS (
  SELECT 
    row_num,
    SUBSTRING_INDEX(SUBSTRING_INDEX(extras, ',', n.digit + 1), ',', -1) AS topping_id,
    pizza_name
  FROM 
    customer_orders_cln_2
  CROSS JOIN 
    (SELECT 0 AS digit UNION ALL SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4) AS n
  WHERE 
    LENGTH(extras) - LENGTH(REPLACE(extras, ',', '')) >= n.digit OR extras IS NULL
),
SplitExclusions AS (
  SELECT 
    row_num,
    SUBSTRING_INDEX(SUBSTRING_INDEX(exclusions, ',', n.digit + 1), ',', -1) AS topping_id,
    pizza_name
  FROM 
    customer_orders_cln_2
  CROSS JOIN 
    (SELECT 0 AS digit UNION ALL SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4) AS n
  WHERE 
    LENGTH(exclusions) - LENGTH(REPLACE(exclusions, ',', '')) >= n.digit OR exclusions IS NULL
),
AllToppings AS (
  SELECT 
    co.row_num,
    se.topping_id AS extra_topping_id,
    sx.topping_id AS exclusion_topping_id,
    te.topping_name AS extra_topping_name,
    tx.topping_name AS exclusion_topping_name,
    co.pizza_name
  FROM 
    customer_orders_cln_2 co
  LEFT JOIN 
    SplitExtras se ON co.row_num = se.row_num
  LEFT JOIN 
    SplitExclusions sx ON co.row_num = sx.row_num
  LEFT JOIN 
    pizza_toppings te ON se.topping_id = te.topping_id
  LEFT JOIN 
    pizza_toppings tx ON sx.topping_id = tx.topping_id
),
JoinedToppings AS (
SELECT 
  row_num,
  CONCAT('Extra', GROUP_CONCAT(DISTINCT CONCAT(' ', extra_topping_name)
    ORDER BY CAST(extra_topping_id AS UNSIGNED))) AS extras_with_extra_name,
  CONCAT('Exclude', GROUP_CONCAT(DISTINCT CONCAT(' ', exclusion_topping_name)
    ORDER BY CAST(exclusion_topping_id AS UNSIGNED))) AS exclusions_with_extra_name,
  pizza_name
FROM 
  AllToppings
GROUP BY 
  row_num
)
SELECT
row_num,
CONCAT_WS(' - ',
  pizza_name,
  exclusions_with_extra_name,
  extras_with_extra_name
  ) AS ordered_items  
FROM JoinedToppings;
```
**Answer:**
| row_num | ordered_items |
|---|---|
| 1 | Meatlovers |
| 2 | Meatlovers |
| 3 | Meatlovers |
| 4 | Vegetarian |
| 5 | Meatlovers - Exclude Cheese |
| 6 | Meatlovers - Exclude Cheese |
| 7 | Vegetarian - Exclude Cheese |
| 8 | Meatlovers - Extra Bacon |
| 9 | Vegetarian |
| 10 | Vegetarian - Extra Bacon |
| 11 | Meatlovers |
| 12 | Meatlovers - Exclude Cheese - Extra Bacon, Chicken |
| 13 | Meatlovers |
| 14 | Meatlovers - Exclude BBQ Sauce, Mushrooms - Extra Bacon, Cheese |
<br>
