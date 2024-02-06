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
- [D. Pricing and Ratings](#d-pricing-and-ratings)
- [E. Bonus DML Challenges](#e-bonus-dml-challenges)

## Cleaning the Data

```sql
-- Created a temp table to remove null values
CREATE TEMPORARY TABLE customer_orders_cln AS
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

-- Created a temp table to set real null values in order to change datatypes, remove other null values and trim units of measurement, moved the units into the column header 
CREATE TEMPORARY TABLE runner_orders_cln AS
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

```
**Answer:**
| customer_id | column2 |
|---|---|
| 101 | example |
| 101 | example |
| 102 | example |
| 102 | example |
<br>

**2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?**

```sql

```
**Answer:**
| customer_id | column2 |
|---|---|
| 101 | example |
| 101 | example |
| 102 | example |
| 102 | example |
<br>

**3. Is there any relationship between the number of pizzas and how long the order takes to prepare?**

```sql

```
**Answer:**
| customer_id | column2 |
|---|---|
| 101 | example |
| 101 | example |
| 102 | example |
| 102 | example |
<br>

**4. What was the average distance travelled for each customer?**

```sql

```
**Answer:**
| customer_id | column2 |
|---|---|
| 101 | example |
| 101 | example |
| 102 | example |
| 102 | example |
<br>

**5. What was the difference between the longest and shortest delivery times for all orders?**

```sql

```
**Answer:**
| customer_id | column2 |
|---|---|
| 101 | example |
| 101 | example |
| 102 | example |
| 102 | example |
<br>

**6. What was the average speed for each runner for each delivery and do you notice any trend for these values?**

```sql

```
**Answer:**
| customer_id | column2 |
|---|---|
| 101 | example |
| 101 | example |
| 102 | example |
| 102 | example |
<br>

**7. What is the successful delivery percentage for each runner?**

```sql

```
**Answer:**
| customer_id | column2 |
|---|---|
| 101 | example |
| 101 | example |
| 102 | example |
| 102 | example |
<br>

## C. Ingredient Optimisation

## D. Pricing and Ratings

## E. Bonus DML Challenges 
(DML = Data Manipulation Language)

