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

## C. Challenge Payment Question
