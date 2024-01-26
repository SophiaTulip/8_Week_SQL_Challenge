# Case Study #1 - Danny's Diner

<img src="https://github.com/SophiaTulip/8_Week_SQL_Challenge/assets/157422079/1b507d0d-4903-4dc6-916c-3725cb673b68" width=500/>

## Table of Contents

- [Business Task](#business-task)
- [Case Study Topics](#case-study-topics)
- [ERD](#erd)
- [Questions and Solutions](#questions-and-solutions)

Original source for this case study can be found [here](https://8weeksqlchallenge.com/case-study-1/).
## Business Task

Danny aims to leverage the provided data to analyze customers' visiting and spending patterns. This analysis will enable him to create a more personalized experience for his customers and provide the insights necessary to make informed decisions about expanding his customer loyalty program.

## Case Study Topics

- Common Table Expressions
- Group By Aggregates
- Window Functions for ranking
- Table Joins

## ERD

<img width="500" alt="ERD1" src="https://github.com/SophiaTulip/8_Week_SQL_Challenge/assets/157422079/461b54d1-2b3d-42ec-b57b-e33b1dc3fe7b">

## Questions and Solutions

The questions were answered using PostgreSQL 15.

**1. What is the total amount each customer spent at the restaurant?**

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

**2. How many days has each customer visited the restaurant?**
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

**3. What was the first item from the menu purchased by each customer?**

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
ORDER BY customer_id ASC
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

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**

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

**5. Which item was the most popular for each customer?**

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
ORDER BY customer_id ASC, product_name ASC
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

**6. Which item was purchased first by the customer after they became a member?**

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
WHERE rank = 1
```
**Answer:**
| customer_id | product_name |
|---|---|
| A | curry |
| B | sushi |

*Given the absence of timestamps, I used a >= operator because Customer A joined the loyalty program on the same day as placing an order.*
<br>

**7. Which item was purchased just before the customer became a member?**

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
ORDER BY product_name ASC, customer_id ASC
```
**Answer:**
| customer_id | product_name |
|---|---|
| A | curry |
| A | sushi |
| B | sushi |

*Customer A purchased 2 products in their order.*
<br>

**8. What is the total items and amount spent for each member before they became a member?**

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

**9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**

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

**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**

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
ORDER BY sales.customer_id ASC
```
**Answer:**
| customer_id | point_total |
|---|---|
| A | 1370 |
| B | 940 |
<br>

**Bonus Questions:**
