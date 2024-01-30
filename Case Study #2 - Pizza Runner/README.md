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

- [A. Pizza Metrics](#pizza-metrics)
- [B. Runner and Customer Experience](#runner-and-customer-experience)
- [C. Ingredient Optimisation](#ingredient-optimisation)
- [D. Pricing and Ratings](#pricing-and-rating)
- [E. Bonus DML Challenges](#bonus-dml-challenges)

## A. Pizza Metrics

**1. How many pizzas were ordered?**

```sql
SELECT
  sales.customer_id,
  SUM(menu.price) AS total
FROM dannys_diner.sales
JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY sales.customer_id ASC;
```
**Answer:**
| customer_id | total |
|---|---|
| A | 76 |
| B | 74 |
| C | 36 |
<br>

**2. How many unique customer orders were made?**
```sql
SELECT
  customer_id,
  COUNT(DISTINCT order_date) AS total_visits
FROM dannys_diner.sales
GROUP BY customer_id;
```
**Answer:**
| customer_id | total_visits |
|---|---|
| A | 4 |
| B | 6 |
| C | 2 |
<br>

**3. How many successful orders were delivered by each runner?**

```sql
SELECT
  customer_id,
  product_name
FROM (
  SELECT
    sales.customer_id,
    menu.product_name,
    sales.order_date,
    DENSE_RANK() OVER (ORDER BY sales.order_date ASC) AS rank
  FROM dannys_diner.sales
  JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
) AS answer3
WHERE rank = 1
GROUP BY product_name, customer_id
ORDER BY customer_id ASC;
```
**Answer:**
| customer_id | product_name |
|---|---|
| A | curry |
| A | sushi |
| B | curry |
| C | ramen |

*Customer A purchased 2 products in their order.*
<br>

**4. How many of each type of pizza was delivered?**

```sql
SELECT
  menu.product_name,
  COUNT (sales.product_id) AS num_of_purchases
 FROM dannys_diner.sales
 JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
 GROUP BY menu.product_name
 ORDER BY num_of_purchases DESC
 LIMIT 1;
```
**Answer:**
| product_name | num_of_purchases |
|---|---|
| ramen | 8 |
<br>

**5. How many Vegetarian and Meatlovers were ordered by each customer?**

```sql
SELECT
  customer_id,
  product_name,
  num_of_purchases
FROM (
  SELECT 
    sales.customer_id, 
    menu.product_name, 
    COUNT(menu.product_id) AS num_of_purchases,
    DENSE_RANK() OVER (PARTITION BY sales.customer_id
      ORDER BY COUNT(sales.customer_id) DESC) AS rank
  FROM dannys_diner.menu
  INNER JOIN dannys_diner.sales
    ON menu.product_id = sales.product_id
  GROUP BY sales.customer_id, menu.product_name
) AS answer5
WHERE rank = 1
ORDER BY customer_id ASC, product_name ASC;
```
**Answer:**
| customer_id | product_name | num_of_purchases |
|---|---|---|
| A | ramen | 3 |
| B | curry | 2 |
| B | ramen | 2 |
| B | sushi | 2 |
| C | ramen | 3 |
<br>

**6. What was the maximum number of pizzas delivered in a single order?**

```sql
SELECT
  customer_id,
  product_name
FROM(
  SELECT
    members.customer_id,
    menu.product_name,
    members.join_date,
    sales.order_date,
    DENSE_RANK() OVER (PARTITION BY members.customer_id
      ORDER BY sales.order_date) AS rank
  FROM dannys_diner.sales
  JOIN dannys_diner.members
    ON sales.customer_id = members.customer_id
  JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
  WHERE sales.order_date >= members.join_date
  ORDER BY members.join_date ASC, sales.order_date ASC
) AS answer6
WHERE rank = 1;
```
**Answer:**
| customer_id | product_name |
|---|---|
| A | curry |
| B | sushi |

*Given the absence of timestamps, I used a >= operator because Customer A joined the loyalty program on the same day as placing an order.*
<br>

**7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?**

```sql
SELECT
  customer_id,
  product_name
FROM (
  SELECT
    members.customer_id,
    menu.product_name,
    members.join_date,
    sales.order_date,
    DENSE_RANK() OVER (PARTITION BY members.customer_id
      ORDER BY sales.order_date DESC) AS rank
  FROM dannys_diner.sales
  JOIN dannys_diner.members
    ON sales.customer_id = members.customer_id
  JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
  WHERE sales.order_date < members.join_date
  ORDER BY members.join_date ASC, sales.order_date DESC
) AS answer7
WHERE rank = 1
ORDER BY product_name ASC, customer_id ASC;
```
**Answer:**
| customer_id | product_name |
|---|---|
| A | curry |
| A | sushi |
| B | sushi |

*Customer A purchased 2 products in their order.*
<br>

**8. How many pizzas were delivered that had both exclusions and extras?**

```sql
SELECT
  sales.customer_id,
  COUNT(sales.product_id) AS product_total,
  SUM(menu.price) AS price_total
FROM dannys_diner.sales
JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
JOIN dannys_diner.members
  ON sales.customer_id = members.customer_id
WHERE sales.order_date < members.join_date
GROUP BY sales.customer_id
ORDER BY sales.customer_id ASC;
```
**Answer:**
| customer_id | product_total | price_total |
|---|---|---|
| A | 2 | 25 |
| B | 3 | 40 | 
<br>

**9. What was the total volume of pizzas ordered for each hour of the day?**

```sql
SELECT
  customer_id,
  COALESCE(x1, 0) + COALESCE(x2, 0) AS point_total
FROM (
  SELECT
    sales.customer_id, 
    SUM(menu.price) FILTER (WHERE sales.product_id > 1)*10 AS x1,
    SUM(menu.price) FILTER (WHERE sales.product_id = 1)*20 AS x2
  FROM dannys_diner.sales
  JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
  GROUP BY sales.customer_id
  ORDER BY sales.customer_id ASC
) AS answer9;
```
**Answer:**
| customer_id | point_total |
|---|---|
| A | 860 |
| B | 940 |
| C | 360 |
<br>

**10. What was the volume of orders for each day of the week?**

```sql
SELECT
  sales.customer_id,
  SUM(CASE
      WHEN EXTRACT(MONTH FROM sales.order_date) = 1 THEN
        CASE
          WHEN sales.order_date >= members.join_date
          AND sales.order_date <= members.join_date + INTERVAL '1 week' THEN
            menu.price * 20
          WHEN sales.product_id = 1 THEN
            menu.price * 20
          ELSE
            menu.price * 10
          END
      END) AS point_total
FROM dannys_diner.sales
JOIN dannys_diner.members
  ON sales.customer_id = members.customer_id
JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY sales.customer_id ASC;
```
**Answer:**
| customer_id | point_total |
|---|---|
| A | 1370 |
| B | 940 |
<br>

## B. Runner and Customer Experience

## C. Ingredient Optimisation

## D. Pricing and Ratings

## E. Bonus DML Challenges 
(DML = Data Manipulation Language)

