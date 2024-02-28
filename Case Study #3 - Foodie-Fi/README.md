# Case Study #3 - Foodie-Fi

<img width="500" src="https://github.com/SophiaTulip/8_Week_SQL_Challenge/assets/157422079/56e0530d-3b8d-4091-adda-dd9faeb5d008">


## Table of Contents

- [Business Task](#business-task)
- [ERD](#erd)
- [Questions and Solutions](#questions-and-solutions)

Original source for this case study can be found [here](https://8weeksqlchallenge.com/case-study-3/).
## Business Task

Utilize data from Foodie-Fi's subscription-based model to analyze customer journeys, identify trends, and inform strategic decisions for optimizing subscription conversions, reducing churn, and enhancing customer satisfaction and retention.

## ERD

<img width="500" src="https://github.com/SophiaTulip/8_Week_SQL_Challenge/assets/157422079/1f0d8e0e-590d-4445-adff-a3a919cc096d">


## Questions and Solutions

- [A. Customer Journey](#a-customer-journey)
- [B. Data Analysis Questions](#b-data-analysis-questions)
- [C. Challenge Payment Question](#c-challenge-payment-question)

## A. Customer Journey

**Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.**

The samples that were giving in the table were for customers 1, 2, 11, 13, 15, 16, 18, and 19.

```sql
SELECT
  s.customer_id,
  p.plan_name,
  s.start_date
FROM foodie_fi.subscriptions AS s
JOIN foodie_fi.plans AS p
  ON s.plan_id = p.plan_id
WHERE customer_id IN (1, 2, 11, 13, 15, 16, 18, 19)
ORDER BY s.customer_id ASC, s.start_date ASC;
```
**Answer:**

Customer 1:<br>
Start Date: August 1, 2020<br>
Journey:
- August 1, 2020: Started with a free 7-day trial account.
- August 8, 2020: Upgraded to a basic monthly plan.

Customer 2:<br>
Start Date: September 20, 2020<br>
Journey:
- September 20, 2020: Started with a free 7-day trial account.
- September 27, 2020: Upgraded to a pro annual plan.

Customer 11:<br>
Start Date: November 19, 2020<br>
Journey:
- November 19, 2020: Started with a free 7-day trial account.
- November 26, 2020: Account transitioned to churn.

Customer 13:<br>
Start Date: December 15, 2020<br>
Journey:
- December 15, 2020: Started with a free 7-day trial account.
- December 22, 2020: Upgraded to a basic monthly plan.
- March 29, 2021: Upgraded to a pro monthly plan.

Customer 15:<br>
Start Date: March 17, 2020<br>
Journey:
- March 17, 2020: Started with a free 7-day trial account.
- March 24, 2020: Upgraded to a pro monthly plan.
- April 29, 2020: Account transitioned to churn.

Customer 16:<br>
Start Date: May 31, 2020<br>
Journey:
- May 31, 2020: Started with a free 7-day trial account.
- June 7, 2020: Upgraded to a basic monthly plan.
- October 21, 2020: Upgraded to a pro annual plan.

Customer 18:<br>
Start Date: July 6, 2020<br>
Journey:
- July 6, 2020: Started with a free 7-day trial account.
- July 13, 2020: Upgraded to a pro monthly plan.

Customer 19:<br>
Start Date: June 22, 2020<br>
Journey:
- June 22, 2020: Started with a free 7-day trial account.
- June 29, 2020: Upgraded to a pro monthly plan.
- August 29, 2020: Upgraded to a pro annual plan.

## B. Data Analysis Questions

**1. How many customers has Foodie-Fi ever had?**

```sql
SELECT
  COUNT(DISTINCT customer_id) AS total_customers
FROM foodie_fi.subscriptions;
```
**Answer:**
| total_customers |
|---|
| 1000 |
<br>

**2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value.**

```sql
SELECT
  EXTRACT(MONTH FROM start_date) AS month,
  COUNT(DISTINCT customer_id) AS total_trials
FROM foodie_fi.subscriptions
WHERE plan_id = 0
GROUP BY EXTRACT(MONTH FROM start_date)
ORDER BY month;
```
**Answer:**
| month | total_trials |
|---|---|
| 1 | 88 |
| 2 | 68 |
| 3 | 94 |
| 4 | 81 |
| 5 | 88 |
| 6 | 79 |
| 7 | 89 |
| 8 | 88 |
| 9 | 87 |
| 10 | 79 |
| 11 | 74 |
| 12 | 84 |
<br>

**3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name.**

```sql
SELECT
  p.plan_name,
  COUNT(s.start_date) AS plans_after_2020
FROM foodie_fi.subscriptions AS s
JOIN foodie_fi.plans AS p
  ON s.plan_id = p.plan_id
WHERE DATE_PART('year', start_date) > 2020
GROUP BY p.plan_name
ORDER BY p.plan_name;
```
**Answer:**
| plan_name | plans_after_2020 |
|---|---|
| basic monthly | 8 |
| churn | 71 |
| pro annual | 63 |
| pro monthly | 60 |
<br>

**4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?**

```sql
SELECT
  COUNT(CASE WHEN plan_id = 4 THEN customer_id END) AS churn_count,
  ROUND(100.0 * SUM(CASE WHEN plan_id = 4 THEN 1 ELSE 0 END) / COUNT(DISTINCT customer_id), 1) AS churn_percentage
FROM foodie_fi.subscriptions;
```
**Answer:**
| churn_count | churn_percentage |
|---|---|
| 307 | 30.7 |
<br>

**5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?**

```sql
WITH churned AS(
SELECT
  customer_id,
  CASE 
    WHEN plan_id = 4
    AND
    LAG(plan_id) OVER (PARTITION BY customer_id ORDER BY start_date) = 0
    THEN 1
    ELSE 0
  END as churned
FROM foodie_fi.subscriptions
)

SELECT 
  SUM(churned) as churned_customers,
  ROUND(SUM(churned) / CAST(COUNT(DISTINCT customer_id) AS float) * 100) as churn_percentage
FROM churned;
```
**Answer:**
| churned_customers | churn_percentage |
|---|---|
| 92 | 9 |
<br>

**6. What is the number and percentage of customer plans after their initial free trial?**

```sql
WITH after_trial AS(
SELECT
  customer_id,
  plan_id,
  ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY start_date) AS plan_rank
FROM foodie_fi.subscriptions
)

SELECT
  plan_name,
  COUNT(plan_rank) AS plan_total,
  ROUND(COUNT(plan_rank) * 100.0 / (SELECT COUNT(DISTINCT customer_id) FROM after_trial), 1)  AS percentage
FROM after_trial
JOIN foodie_fi.plans
  ON after_trial.plan_id = plans.plan_id
WHERE plan_rank = 2
GROUP BY plan_name
ORDER BY plan_name;
```
**Answer:**
| plan_name | plan_total | percentage |
|---|---|---|
| basic monthly | 546 | 54.6 |
| churn | 92 | 9.2 |
| pro annual | 37 | 3.7 |
| pro monthly | 325 | 32.5 |
<br>

**7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?**

```sql
WITH TotalCount AS(
SELECT
  customer_id,
  plan_id,
  COUNT(DISTINCT customer_id) AS tot_count,
  ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY start_date DESC) AS plan_rank
FROM foodie_fi.subscriptions
WHERE start_date <= '2020-12-31'
GROUP BY plan_id, customer_id, start_date
)

SELECT
  plan_name,
  COUNT(tot_count) AS tot_customers,
  ROUND(COUNT(tot_count) * 100.0 / (SELECT COUNT(DISTINCT customer_id) FROM TotalCount), 1) AS percentage
FROM TotalCount
JOIN foodie_fi.plans
  ON TotalCount.plan_id = plans.plan_id
WHERE plan_rank = 1
GROUP BY plan_name, tot_count
ORDER BY plan_name ASC;

```
**Answer:**
| plan_name | tot_customers | percentage |
|---|---|---|
| basic monthly | 224 | 22.4 |
| churn | 236 | 23.6 |
| pro annual | 195 | 19.5 |
| pro monthly | 326 | 32.6 |
| trial | 19 | 1.9 |
<br>

**8. How many customers have upgraded to an annual plan in 2020?**

```sql
SELECT
  p.plan_name,
  COUNT(s.customer_id) AS tot_customers
FROM foodie_fi.subscriptions s
JOIN foodie_fi.plans p
  ON s.plan_id = p.plan_id
WHERE s.plan_id = 3
  AND DATE_PART('year', s.start_date) = 2020
GROUP BY p.plan_name;
```
**Answer:**
| plan_name | tot_customers |
|---|---|
| pro annual | 195 |
<br>

**9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?**

```sql

```
**Answer:**
| column1 | column2 |
|---|---|
| example | example |
| example | example |
| example | example |
| example | example |
<br>

**10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc).**

```sql

```
**Answer:**
| column1 | column2 |
|---|---|
| example | example |
| example | example |
| example | example |
| example | example |
<br>

**11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?**

```sql

```
**Answer:**
| column1 | column2 |
|---|---|
| example | example |
| example | example |
| example | example |
| example | example |
<br>

## C. Challenge Payment Question
