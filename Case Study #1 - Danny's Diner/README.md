# Case Study Questions

Each of the following case study questions can be answered using a single SQL statement:
***
### 1.  What is the total amount each customer spent at the restaurant?
````sql
SELECT
  t1.customer_id,
  SUM(t2.price) total_spending
FROM
  dannys_diner.sales AS t1
  INNER JOIN dannys_diner.menu AS t2 
  ON t1.product_id = t2.product_id
GROUP BY
  customer_id
ORDER BY
  customer_id
````	
**Answer**
| customer\_id | total\_spending |
| ------------ | --------------- |
| A            | 76              |
| B            | 74              |
| C            | 36              |
**Step**
* `INNER JOIN` two tables `sales`  and `menu` with the former as the base table as we need the `price` column from the latter
* Use `SUM` and `GROUP BY` to calculate the total spendings for Danny's dishes by each customer
***
### 2.  How many days has each customer visited the restaurant?
````sql
SELECT
  customer_id,
  COUNT(DISTINCT order_date) AS number_of_visited_days
FROM
  dannys_diner.sales
GROUP BY
  1
ORDER BY
  1
````
**Answer**
| customer\_id | number_of_visited_days|
| ------------ | ----- |
| A            | 4     |
| B            | 6     |
| C            | 2     |
**Steps**
* Use `COUNT(DISTINCT order_date)` to have the number of unique date in `sales` table as each `order_date` appear multiple times 
* Then, use `GROUP BY` to show the results by `customer_id`
***
### 3.  What was the first item from the menu purchased by each customer?
````sql
WITH cte AS(
  SELECT
    DISTINCT customer_id,
    t2.product_name,
    DENSE_RANK() OVER(
      PARTITION BY t1.customer_id
      ORDER BY t1.order_date
    ) AS rank
  FROM
    dannys_diner.sales AS t1
    INNER JOIN dannys_diner.menu AS t2 
    ON t1.product_id = t2.product_id
)
SELECT
  customer_id,
  product_name
FROM
  cte
WHERE
  rank = 1
ORDER BY
  1,
  2
````
**Answer**
| customer\_id | product\_name |
| ------------ | ------------- |
| A            | curry         |
| A            | sushi         |
| B            | curry         |
| C            | ramen         |
**Step**
* Create a `CTE`, using window function `DENSE_RANK()` to sort `order_date` values by each `customer_id` . Plus, we `INNER JOIN` two tables `menu` and `sales` as we need the `product_name` along the `customer_id` and `order_date` columns
* We also use `DISTINCT` to exclude duplicates. For example, customer C order two ramen on the same day
* Finally, we only `SELECT` rows having `rank = 1`, which indicates first items each customer ordered at Danny's restaurant
***
### 4.  What is the most purchased item on the menu and how many times was it purchased by all customers?
````sql
SELECT
  t2.product_name,
  COUNT(t1.*) AS frequency
FROM
  dannys_diner.sales AS t1
  INNER JOIN dannys_diner.menu AS t2 
  ON t1.product_id = t2.product_id
GROUP BY
  t2.product_name
ORDER BY
  frequency DESC
LIMIT
  1
  ````
 **Answer**
| product\_name | frequency |
| ------------- | --------- |
| ramen         | 8         |
**Step**
* `INNER JOIN` two tables similarly to the previous questions
* Use `COUNT(*)` & `GROUP BY` to count the frequency of each product_name 
***
### 5.  Which item was the most popular for each customer?

````sql
WITH cte AS(
  SELECT
    t1.customer_id,
    t2.product_name,
    COUNT(t1.*) AS frequency,
    DENSE_RANK() OVER(
      PARTITION BY 
	      t1.customer_id
      ORDER BY
        COUNT(t1.*) DESC
    ) as rank
  FROM
    dannys_diner.sales AS t1
    INNER JOIN dannys_diner.menu AS t2 
    ON t1.product_id = t2.product_id
  GROUP BY
    1,
    2
)

SELECT
  customer_id,
  product_name,
  frequency AS order_frequency
FROM
  cte
WHERE 
  rank = 1
ORDER BY
  1,
  2
````
**Answer**
| customer\_id | product\_name | order\_frequency |
| ------------ | ------------- | ---------------- |
| A            | ramen         | 3                |
| B            | curry         | 2                |
| B            | ramen         | 2                |
| B            | sushi         | 2                |
| C            | ramen         | 3                |
**Step**
This answer is similar to that of question 5, except that:
* We use `GROUP BY` with two columns: customer_id & product_id
* `DENSE_RANK()` is used with `ORDER BY COUNT(*) DESC` and `PARTITION BY customer_id` to find the top 1 dish ordered by each customer
***
### 6.  Which item was purchased first by the customer after they became a member?

````sql
WITH cte AS(
  SELECT
    t1.customer_id,
    t3.product_name,
    t1.order_date,
    t2.join_date,
    DENSE_RANK() OVER(
      PARTITION BY t1.customer_id
      ORDER BY t1.order_date
    ) AS rank
  FROM
    dannys_diner.sales AS t1
    INNER JOIN dannys_diner.members AS t2 ON t1.customer_id = t2.customer_id
    AND t1.order_date >= t2.join_date
    INNER JOIN dannys_diner.menu AS t3 ON t1.product_id = t3.product_id
)
SELECT
  customer_id,
  product_name,
  order_date
FROM
  cte
WHERE
  rank = 1
 ````
**Answer**
| customer\_id | product\_name | order\_date|
| ------------ | ------------- | -----------|
| A            | curry         | 2021-01-07 |
| B            | sushi         | 2021-01-11 |
**Step**
* Use INNER JOIN to join all 3 tables. However, we only keep the records having `order_date >= join_date` by adding one more condition after the `ON` clause (as we include the date on which each customer join Danny's restaurant membership)
* Use `DENSE_RANK()` similarly to Question 3
***
### 7.  Which item was purchased just before the customer became a member?

````sql
WITH cte AS(
  SELECT
    t1.customer_id,
    t3.product_name,
    t1.order_date,
    t2.join_date,
    DENSE_RANK() OVER(
      PARTITION BY t1.customer_id
      ORDER BY t1.order_date DESC
    ) AS rank
  FROM
    dannys_diner.sales AS t1
    INNER JOIN dannys_diner.members AS t2 ON t1.customer_id = t2.customer_id
    AND t1.order_date < t2.join_date
    INNER JOIN dannys_diner.menu AS t3 ON t1.product_id = t3.product_id
)
SELECT
  customer_id,
  product_name,
  order_date
FROM
  cte
WHERE
  rank = 1
 ````
 **Answer**
| customer\_id | product\_name | order\_date|
| ------------ | ------------- | -----------|
| A            | sushi         | 2021-01-01 |
| A            | curry         | 2021-01-01 |
| B            | sushi         | 2021-01-04 |
 **Step**
 This question is answered similarly to Question 6, the only differences are:
* `INNER JOIN` condition `t1.order_date < t2.join_date` (or using `WHERE` clause after joining 3 tables)
* `ORDER BY t1.order_date DESC` for `DENSE_RANK()`
***
### 8.  What is the total items and amount spent for each member before they became a member?
````sql
SELECT
  t1.customer_id,
  COUNT(DISTINCT t3.product_name),
  SUM(t3.price) AS total_spending
FROM
  dannys_diner.sales AS t1
  INNER JOIN dannys_diner.members AS t2 ON t1.customer_id = t2.customer_id
  AND t1.order_date < t2.join_date
  INNER JOIN dannys_diner.menu AS t3 ON t1.product_id = t3.product_id
GROUP BY
  t1.customer_id
ORDER BY
  t1.customer_id
````
**Answer**
| customer\_id | count | total\_spending |
| ------------ | ----- | --------------- |
| A            | 2     | 25              |
| B            | 2     | 40              |
**Step**
* After joining tables, perform aggregation calculations with `COUNT(DISTINCT)` and `SUM()`
***
### 9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
````sql
WITH cte AS(
  SELECT 
    *,
    CASE WHEN product_name='sushi' THEN 20
    ELSE 10 END AS point
  FROM dannys_diner.sales AS t1 
  INNER JOIN dannys_diner.menu AS t2 
  ON t1.product_id = t2.product_id
)

SELECT
  customer_id,
  SUM(point*price) AS total_point
FROM cte 
GROUP BY customer_id
ORDER BY customer_id
````
**Answer**
| customer\_id | total\_point |
| ------------ | ------------ |
| A            | 860          |
| B            | 940          |
| C            | 360          |
**Step**
* Create a new column using `CASE WHEN` showing the point value for each type of dish
* Use `SUM(price*point)` to calculate total points for each customer
***
### 10.  In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

````sql
WITH cte AS(
  SELECT
    t1.customer_id,
    t3.product_name,
    t3.price,
    t1.order_date,
    t2.join_date,
    CASE
      WHEN t3.product_name = 'sushi' THEN 20
      WHEN t1.order_date BETWEEN t2.join_date
      AND t2.join_date + 6 THEN 20
      ELSE 10
    END AS point
  FROM
    dannys_diner.sales AS t1
    INNER JOIN dannys_diner.members AS t2 ON t1.customer_id = t2.customer_id
    INNER JOIN dannys_diner.menu AS t3 ON t1.product_id = t3.product_id
)
SELECT
  customer_id,
  SUM(point * price) AS total_point
FROM
  cte
WHERE
  order_date <= '2021-01-31'
GROUP BY
  1
ORDER BY
  1
````

**Answer**
| customer\_id | total\_point |
| ------------ | ------------ |
| A            | 1370         |
| B            | 820          |
**Step**
* This question is similar to Question 9, except that we must modify the condition for `points` column. Plus, we must also consider all orders of each customer whether they have become a member or not to calculate the total point up to the end of January
***
# Bonus Questions

## Join All The Things

The following questions are related creating basic data tables that Danny and his team can use to quickly derive insights without needing to join the underlying tables using SQL.

11.  Recreate the following table output using the available data:

## Rank All The Things

12.  Danny also requires further information about the `ranking` of customer products, but he purposely does not need the ranking for non-member purchases so he expects null `ranking` values for the records when customers are not yet part of the loyalty program.


