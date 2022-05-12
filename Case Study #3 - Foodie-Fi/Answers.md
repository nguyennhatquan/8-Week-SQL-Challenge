# 🙆‍♂️ A. Customer Journey

Based off the 8 sample customers provided in the sample from the `subscriptions` table, write a brief description about each customer’s onboarding journey.

Try to keep it as short as possible - you may also want to run some sort of join to make your explanations a bit easier!

| customer\_id | plan\_id | start\_date |
| ------------ | -------- | ----------- |
| 1            | 0        | 2020-08-01  |
| 1            | 1        | 2020-08-08  |
| 2            | 0        | 2020-09-20  |
| 2            | 3        | 2020-09-27  |
| 11           | 0        | 2020-11-19  |
| 11           | 4        | 2020-11-26  |
| 13           | 0        | 2020-12-15  |
| 13           | 1        | 2020-12-22  |
| 13           | 2        | 2021-03-29  |
| 15           | 0        | 2020-03-17  |
| 15           | 2        | 2020-03-24  |
| 15           | 4        | 2020-04-29  |
| 16           | 0        | 2020-05-31  |
| 16           | 1        | 2020-06-07  |
| 16           | 3        | 2020-10-21  |
| 18           | 0        | 2020-07-06  |
| 18           | 2        | 2020-07-13  |
| 19           | 0        | 2020-06-22  |
| 19           | 2        | 2020-06-29  |
| 19           | 3        | 2020-08-29  |
# 🔍 B. Data Analysis Questions

### 1.  How many customers has Foodie-Fi ever had?
````sql
SELECT
  COUNT(DISTINCT customer_id) AS total_customers
FROM
  foodie_fi.subscriptions
````
**Answer**
| total_customers|
| --------- | 
| 1000      | 
### 2.  What is the monthly distribution of `trial` plan `start_date` values for our dataset - use the start of the month as the group by value

````sql
SELECT
  DATE_TRUNC('month', start_date) AS month,
  COUNT(*)
FROM
  foodie_fi.subscriptions
WHERE
  plan_id = 0
GROUP BY
  1
ORDER BY
  1
````
**Answer**
| month      | count |
| ---------- | ----- |
| 2020-01-01 | 88    |
| 2020-02-01 | 68    |
| 2020-03-01 | 94    |
| 2020-04-01 | 81    |
| 2020-05-01 | 88    |
| 2020-06-01 | 79    |
| 2020-07-01 | 89    |
| 2020-08-01 | 88    |
| 2020-09-01 | 87    |
| 2020-10-01 | 79    |
| 2020-11-01 | 75    |
| 2020-12-01 | 84    |

**Step**
* Use `DATE_TRUNC ('month', TIMESTAMP)` to return the first day of each month

### 3.  What plan `start_date` values occur after the year 2020 for our dataset? Show the breakdown by count of events for each `plan_name`
````sql
SELECT
  t1.plan_id,
  t2.plan_name,
  COUNT(*) AS number_of_events
FROM
  foodie_fi.subscriptions AS t1
  INNER JOIN foodie_fi.plans AS t2 ON t1.plan_id = t2.plan_id
WHERE
  DATE_PART('year', start_date) > 2020
GROUP BY
  1,2
ORDER BY
  1
````
**Answer**
| plan\_id | plan\_name    | number\_of\_events |
| -------- | ------------- | ------------------ |
| 1        | basic monthly | 8                  |
| 2        | pro monthly   | 60                 |
| 3        | pro annual    | 63                 |
| 4        | churn         | 71                 |

**Step**
* Use `DATE_PART('year', TIMESTAMP)` to return the year as an `INTEGER`
### 4.  What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
````sql
SELECT
  SUM(
    CASE
      WHEN plan_id = 4 THEN 1
      ELSE 0
    END
  ) AS number_of_churned_customers,
  ROUND(
    100 * SUM(
      CASE
        WHEN plan_id = 4 THEN 1
        ELSE 0
      END
    ) / SUM(COUNT(DISTINCT customer_id)) OVER(),
    2
  ) percentage
FROM
  foodie_fi.subscriptions
````

**Answer**
| number\_of\_churned\_customers | percentage |
| ------------------------------ | ---------- |
| 307                            | 30.70      |
❗ **Note**
* This is a good example of how we can use SUM CASE WHEN for different situations. Another approach is to use CTE but it might be less efficient
### 5.  How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
````sql
WITH cte AS(
  SELECT
    customer_id,
    plan_id,
    ROW_NUMBER() OVER(
      PARTITION BY customer_id
      ORDER BY start_date
    ) AS rank
  FROM
    foodie_fi.subscriptions
)
SELECT
  SUM(
    CASE
      WHEN plan_id = 4
      AND rank = 2 THEN 1
      ELSE 0
    END
  ) AS frequency,
  ROUND(
    100 * SUM(
      CASE
        WHEN plan_id = 4
        AND rank = 2 THEN 1
        ELSE 0
      END
    ) / SUM(COUNT(DISTINCT customer_id)) OVER(),
    1
  ) AS percentage
FROM
  cte
````

**Answer**
| frequency | percentage |
| --------- | ---------- |
| 92        | 9.2        |

**Step**
* Combine window function `ROW_NUMBER` along with `SUM CASE WHEN` 
### 6.  What is the number and percentage of customer plans after their initial free trial?
````sql
WITH cte AS(
  SELECT
    customer_id,
    plan_id,
    ROW_NUMBER() OVER(
      PARTITION BY customer_id
      ORDER BY start_date
    ) AS rank
  FROM
    foodie_fi.subscriptions
)
SELECT
  t1.plan_id,
  t2.plan_name,
  SUM(
    CASE
      WHEN rank = 2 THEN 1
      ELSE 0
    END
  ) AS frequency,
  CEILING(
    100 * SUM(
      CASE
        WHEN rank = 2 THEN 1
        ELSE 0
      END
    ) / SUM(COUNT(DISTINCT customer_id)) OVER()
  ) AS percentage
FROM
  cte AS t1
  INNER JOIN foodie_fi.plans AS t2 ON t1.plan_id = t2.plan_id
WHERE
  rank = 2
GROUP BY
  1,
  2
  ````
 **Answer**
 | plan\_id | plan\_name    | frequency | percentage |
| -------- | ------------- | --------- | ---------- |
| 1        | basic monthly | 546       | 55         |
| 2        | pro monthly   | 325       | 33         |
| 3        | pro annual    | 37        | 4          |
| 4        | churn         | 92        | 10         |

### 7.  What is the customer count and percentage breakdown of all 5 `plan_name` values at `2020-12-31`?
````sql
WITH cte AS(
  SELECT
    *,
    ROW_NUMBER() OVER(
      PARTITION BY customer_id
      ORDER BY start_date DESC
    ) AS rank
  FROM
    foodie_fi.subscriptions
  WHERE
    start_date <= '2020-12-31'
)
SELECT
  plan_id,
  COUNT(DISTINCT customer_id) AS number_of_customer,
  ROUND(100 * COUNT(*) / SUM(COUNT(*)) OVER(), 1) AS percentage
FROM
  cte
WHERE
  rank = 1
GROUP BY
  plan_id
````

**Answer**
| plan\_id | number\_of\_customer | percentage |
| -------- | -------------------- | ---------- |
| 0        | 19                   | 1.9        |
| 1        | 224                  | 22.4       |
| 2        | 326                  | 32.6       |
| 3        | 195                  | 19.5       |
| 4        | 236                  | 23.6       |
**Step**
* In other words, the question ask for the distribution of customers by subscribed plan on `2020-12-31`. Thus, we perform calculations based on the lastest subscribed plan up to `2020-12-31`
❗ **Note**
	* Be careful that we are dealing with a "slowly changing dimension" (SCD) table
		> A Slowly Changing Dimension (SCD) is **a dimension that stores and manages both current and historical data over time in a data warehouse**
	* That's why we use ROW_NUMBER() to identify the lastest purchased plan of each customer ( assuming they didn't subcribe to different plan on the same day, that would required the time value to identify the correct record) 
### 8.  How many customers have upgraded to an annual plan in 2020?
````sql
SELECT
  COUNT(DISTINCT customer_id)
FROM
  foodie_fi.subscriptions
WHERE
  plan_id = 3
  AND start_date BETWEEN '2020-01-01'
  AND '2020-12-31'
````

**Answer**
| count |  
|--|
| 195 |  

### 9.  How many days on average does it take for a customer to subscribe an annual plan from the day they join Foodie-Fi?
````sql
SELECT
  COUNT(DISTINCT customer_id)
FROM
  foodie_fi.subscriptions
WHERE
  plan_id <> 0
  ````
  **Answer**
| count |  
|-------|
| 1000	|

**Step**  
* As the number of uniquer customers subscribed to trial plan is 1000, which is equal to the number of unique customers Foodie Fi served so far, we can conclude that all customers use trial plan on the first day they use this service
*  Thus, we can simply find the difference between the join day and when customers upgrade to annual plan as follows 
````sql
WITH cte_1 AS (
  SELECT
    customer_id,
    start_date
  FROM
    foodie_fi.subscriptions
  WHERE
    plan_id = 0
),
cte_2 AS(
  SELECT
    customer_id,
    start_date
  FROM
    foodie_fi.subscriptions
  WHERE
    plan_id = 3
)
SELECT
  CEILING(AVG(t2.start_date - t1.start_date)) "average_time_to_conversion_(day)"
FROM
  cte_1 AS t1
  INNER JOIN cte_2 AS t2 ON t1.customer_id = t2.customer_id
````
**Answer**
| average_time_to_conversion_(day)|  
|-------|
| 105|

❗ **Note**
* When dealing `DATE` type data, to calculate the difference between them, there are 2 possible approach
	*  `DATE_1 - DATE_`2 will return the difference of days in `INTEGER`
	*  `DATE_PART('day',AGE(DATE_1 - DATE_2))`

### 10.  Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)

### 11.  How many customers downgraded from a pro monthly to a basic monthly plan in 2020?
````sql
WITH cte AS(
  SELECT
    customer_id,
    start_date,
    plan_id,
    LEAD(plan_id, 1) OVER(
      PARTITION BY customer_id
      ORDER BY
        customer_id,
        start_date
    ) AS next_plan
  FROM
    foodie_fi.subscriptions
  WHERE
    DATE_PART('year', start_date :: TIMESTAMP) = 2020
)
SELECT
  COUNT(DISTINCT customer_id)
FROM
  cte
WHERE
  plan_id = 2
  AND next_plan = 1
````
**Answer**
|count|  
|-------|
| 0|

* There are no customers downgrading from pro monthly plan to basic monthly plan in 2020

**Step**
* As we will need to perform some sort of value comparison from one row to the next, we can use either `LAG` or `LEAD` to answer this question
* Remember to add appropriate columns after `PARTITION BY` and `ORDER BY`
# 💰 C. Challenge Payment Question
[ *To be updated...*]
# 🎁 D. Outside The Box Questions

The following are open ended questions which might be asked during a technical interview for this case study - there are no right or wrong answers, but answers that make sense from both a technical and a business perspective make an amazing impression!

### 1.  How would you calculate the rate of growth for Foodie-Fi?

### 2.  What key metrics would you recommend Foodie-Fi management to track over time to assess performance of their overall business?
* User Acquisition
	* Download
	* Cum Download
	* % Organic Download
* Product
	* Retention
	* DAU/WAU/MAU
* Moneytization
	* Revenue
	* RPD
	* **LTV**
### 3.  What are some key customer journeys or experiences that you would analyse further to improve customer retention?

### 4.  If the Foodie-Fi team were to create an exit survey shown to customers who wish to cancel their subscription, what questions would you include in the survey?

### 5.  What business levers could the Foodie-Fi team use to reduce the customer churn rate? How would you validate the effectiveness of your ideas?