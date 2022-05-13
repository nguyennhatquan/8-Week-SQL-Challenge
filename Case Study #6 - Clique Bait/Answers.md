# ⚡ A. Digital Analysis
### 1.  How many users are there?
````sql
SELECT
  COUNT(DISTINCT user_id) AS user_count
FROM
  clique_bait.users
 ````
 **Answer**
 
| user_count |  
|--|
| 500 |  

### 2.  How many cookies does each user have on average?
**CTE Approach**
````sql
WITH cte AS (
  SELECT
    user_id,
    COUNT(DISTINCT cookie_id) AS cookie_count
  FROM
    clique_bait.users
  GROUP BY
    1
)
SELECT
  ROUND(AVG(cookie_count),2) AS avg_cookie_count
FROM
  cte
````
**Window Function Approach**
````sql
SELECT
  ROUND(AVG(COUNT(DISTINCT cookie_id)) OVER(), 2) AS avg_cookie_count
FROM
  clique_bait.users
GROUP BY
  user_id
LIMIT
  1
````
**Answer**
| avg_cookie_count|  
|--|
| 3.56|  

### 3.  What is the unique number of visits by all users per month?
````sql
SELECT
  DATE_PART('month', t1.event_time) AS month,
  COUNT(DISTINCT visit_id) visit_count
FROM
  clique_bait.events t1
  INNER JOIN clique_bait.users t2 ON t1.cookie_id = t2.cookie_id
GROUP BY
  1
````
**Answer**
| month | visit\_count |
| ----- | ------------ |
| 1     | 876          |
| 2     | 1488         |
| 3     | 916          |
| 4     | 248          |
| 5     | 36           |

### 4.  What is the number of events for each event type?
````sql
SELECT
  t1.event_type,
  t2.event_name,
  COUNT(t1.event_type)
FROM
  clique_bait.events t1
  INNER JOIN clique_bait.event_identifier t2 ON t1.event_type = t2.event_type
GROUP BY
  1,
  2
ORDER BY
  3 DESC
````
**Answer**
| event\_type | event\_name   | count |
| ----------- | ------------- | ----- |
| 1           | Page View     | 20928 |
| 2           | Add to Cart   | 8451  |
| 3           | Purchase      | 1777  |
| 4           | Ad Impression | 876   |
| 5           | Ad Click      | 702   |

### 5.  What is the percentage of visits which have a purchase event?
We know that event `Purchase` has `event_type = 3`. First, we must check if each unique `visit_id` only has 1 event `Purchase`, otherwise we will mistakenly calculate the percentage of all `Purchase` event
````sql
SELECT
  visit_id,
  COUNT(*)
FROM
  clique_bait.events
WHERE
  event_type = 3
GROUP BY
  1
ORDER BY
  2 DESC
LIMIT
  1 
````
**Answer**
| visit\_id | count |
| --------- | ----- |
| b91d27    | 1     |

The query returns 1, now we can be sure each unique `visit_id` only has 1 `Purchase` event. Thus, we can calculate the percentage of visits having `Purchase` event based on the count of `Purchase` event
````sql
SELECT
  ROUND(
    100 * SUM(
      CASE
        WHEN t2.event_name = 'Purchase' THEN 1
        ELSE 0
      END
    ) :: NUMERIC / COUNT(DISTINCT visit_id),
    2
  ) AS percentage
FROM
  clique_bait.events t1
  INNER JOIN clique_bait.event_identifier t2 ON t1.event_type = t2.event_type
````
**Answer**
| percentage|  
|--|
| 49.86|  

### 6.  What is the percentage of visits which view the checkout page but do not have a purchase event?
````sql
WITH cte AS(
  SELECT
    visit_id,
    MAX(
      CASE
        WHEN page_id = 12
        AND event_type = 1 THEN 1
        ELSE 0
      END
    ) AS check_out_flag,
    MAX(
      CASE
        WHEN event_type = 3 THEN 1
        ELSE 0
      END
    ) AS purchase_flag
  FROM
    clique_bait.events
  GROUP BY
    1
)
SELECT
  ROUND(
    100 * (
      1 - (
        SUM(purchase_flag) :: NUMERIC / SUM(check_out_flag)
      )
    ),
    2
  ) AS percentage
FROM
  cte
WHERE
  check_out_flag = 1
````
**Answer**
| percentage |
|--|
| 15.50 |

❗ **Note**
* Use `MAX` with `CASE WHEN` to flag a column. In this case, to flag if a `visit_id` contains any `check_out` page view and a `visit_id` contains a `purchase` event
### 7.  What are the top 3 pages by number of views?
````sql
SELECT
  t1.page_id,
  t2.page_name,
  COUNT(*)
FROM
  clique_bait.events t1
  INNER JOIN clique_bait.page_hierarchy t2 ON t1.page_id = t2.page_id
WHERE
  event_type = 1
GROUP BY
  1,
  2
ORDER BY
  3 DESC
LIMIT
  3
````
**Answer**
| page\_id | page\_name   | count |
| -------- | ------------ | ----- |
| 2        | All Products | 3174  |
| 12       | Checkout     | 2103  |
| 1        | Home Page    | 1782  |

### 8.  What is the number of views and cart adds for each product category?
````sql
SELECT
  t2.product_category,
  SUM(
    CASE
      WHEN event_type = 1 THEN 1
      ELSE 0
    END
  ) AS page_view,
  SUM(
    CASE
      WHEN event_type = 2 THEN 1
      ELSE 0
    END
  ) AS cart_count
FROM
  clique_bait.events t1
  INNER JOIN clique_bait.page_hierarchy t2 ON t1.page_id = t2.page_id
WHERE
  t2.product_category IS NOT NULL
GROUP BY
  1
ORDER BY
  2,
  3
````
**Answer**
| product\_category | page\_view | cart\_count |
| ----------------- | ---------- | ----------- |
| Luxury            | 3032       | 1870        |
| Fish              | 4633       | 2789        |
| Shellfish         | 6204       | 3792        |

### 9.  What are the top 3 products by purchases?
````sql
WITH cte AS(
  SELECT
    visit_id,
    SUM(
      CASE
        WHEN event_type = 3 THEN 1
        ELSE 0
      END
    ) AS purchase_count,
    SUM(
      CASE
        WHEN event_type = 2 THEN 1
        ELSE 0
      END
    ) AS cart_count
  FROM
    clique_bait.events
  GROUP BY
    1
)
SELECT
  t1.visit_id,
  t1.sequence_number,
  t2.event_name,
  t3.page_name
FROM
  clique_bait.events t1
  INNER JOIN clique_bait.event_identifier t2 ON t1.event_type = t2.event_type
  INNER JOIN clique_bait.page_hierarchy t3 ON t1.page_id = t3.page_id
WHERE
  t1.visit_id IN (
    SELECT
      visit_id
    FROM
      cte
    WHERE
      cart_count > 0
      AND purchase_count = 0
  )
ORDER BY
  1,
  2
````
Executing the query above and we can get a general sense of how a customer interacts with Danny's website. From the result, we can conclude that there are cases when a customer adds products into cart without purchasing them
Thus, we only account for  `visit_id` having `add to cart` and  `purchase` event

**Sub Query in `WHERE` clause Approach**
````sql
WITH cte AS(
	SELECT
	  visit_id
	FROM
	  clique_bait.events
	WHERE
	  event_type = 3
	GROUP BY
	  1
)
SELECT
  t2.page_name AS product,
  SUM(
    CASE
      WHEN t1.event_type = 2 THEN 1
      ELSE 0
    END
  ) as purchase_count
FROM
  clique_bait.events t1
  INNER JOIN clique_bait.page_hierarchy t2 ON t1.page_id = t2.page_id
WHERE
  t1.visit_id IN (
    SELECT
      visit_id
    FROM
      cte
  )
  AND t1.page_id NOT IN (1, 2, 12, 13)
GROUP BY
  1
ORDER BY
  2 DESC
LIMIT
  3
````
**LEFT SEMI JOIN Approach**
````sql
WITH cte AS(
  SELECT
    visit_id
  FROM
    clique_bait.events
  WHERE
    event_type = 3

)
SELECT
  t2.page_name AS product,
  SUM(
    CASE
      WHEN t1.event_type = 2 THEN 1
      ELSE 0
    END
  ) as purchase_count
FROM
  clique_bait.events t1
  INNER JOIN clique_bait.page_hierarchy t2 ON t1.page_id = t2.page_id
WHERE
  EXISTS (
    SELECT
      1
    FROM
      cte
    WHERE
      t1.visit_id = cte.visit_id
  )
  AND t1.page_id NOT IN (1, 2, 12, 13)
GROUP BY
  1
ORDER BY
  2 DESC
LIMIT
  3
````
**Answer**
| product | purchase\_count |
| ------- | --------------- |
| Lobster | 754             |
| Oyster  | 726             |
| Crab    | 719             |

# 📦 B. Product Funnel Analysis

Using a single SQL query - create a new output table which has the following details:

-   How many times was each product viewed?
-   How many times was each product added to cart?
-   How many times was each product added to a cart but not purchased (abandoned)?
-   How many times was each product purchased?

Additionally, create another table which further aggregates the data for the above points but this time for each product category instead of individual products.
### Table Creation Step
````sql
DROP TABLE IF EXISTS product_info;
CREATE TEMP TABLE product_info AS(
  WITH cte_1 AS(
    SELECT
      page_id,
      SUM(
        CASE
          WHEN event_type = 1 THEN 1
          ELSE 0
        END
      ) AS product_view,
      SUM(
        CASE
          WHEN event_type = 2 THEN 1
          ELSE 0
        END
      ) AS product_cart
    FROM
      clique_bait.events
    WHERE
      page_id NOT IN (1, 2, 12, 13)
    GROUP BY
      1
  ),
  cte_2 AS (
    SELECT
      page_id,
      SUM(
        CASE
          WHEN event_type = 2 THEN 1
          ELSE 0
        END
      ) AS product_purchase
    FROM
      clique_bait.events
    WHERE
      EXISTS (
        SELECT
          1
        FROM
          clique_bait.events t1
        WHERE
          clique_bait.events.visit_id = t1.visit_id
          AND t1.event_type = 3
      )
      AND page_id NOT IN (1, 2, 12, 13)
    GROUP BY
      1
  ),
  cte_3 AS (
    SELECT
      page_id,
      SUM(
        CASE
          WHEN event_type = 2 THEN 1
          ELSE 0
        END
      ) AS product_cart_not_purchase
    FROM
      clique_bait.events
    WHERE
      NOT EXISTS (
        SELECT
          1
        FROM
          clique_bait.events t1
        WHERE
          clique_bait.events.visit_id = t1.visit_id
          AND t1.event_type = 3
      )
      AND page_id NOT IN (1, 2, 12, 13)
    GROUP BY
      1
  )
  SELECT
    t2.product_id,
    t2.page_name AS product_name,
    t2.product_category,
    cte_1.product_view AS page_views,
    cte_1.product_cart AS cart_adds,
    cte_2.product_purchase AS purchases,
    cte_3.product_cart_not_purchase AS abandoned
  FROM
    clique_bait.page_hierarchy t2
    INNER JOIN cte_1 ON t2.page_id = cte_1.page_id
    INNER JOIN cte_2 ON cte_1.page_id = cte_2.page_id
    INNER JOIN cte_3 ON cte_2.page_id = cte_3.page_id
  ORDER BY
    1
);
DROP TABLE IF EXISTS product_category;
CREATE TEMP TABLE product_category AS(
  SELECT
    product_category,
    SUM(page_views) AS page_views,
    SUM(cart_adds) AS cart_adds,
    SUM(purchases) AS purchases,
    SUM(abandoned) AS abandoned
  FROM
    product_info
  GROUP BY
    1
  ORDER BY
    1
);

SELECT
  *
FROM
  product_info;
  
SELECT
  *
FROM
  product_category;
````
**Answer**
| product\_id | product\_name  | product\_category | page\_views | cart\_adds | purchases | abandoned |
| ----------- | -------------- | ----------------- | ----------- | ---------- | --------- | --------- |
| 1           | Salmon         | Fish              | 1559        | 938        | 711       | 227       |
| 2           | Kingfish       | Fish              | 1559        | 920        | 707       | 213       |
| 3           | Tuna           | Fish              | 1515        | 931        | 697       | 234       |
| 4           | Russian Caviar | Luxury            | 1563        | 946        | 697       | 249       |
| 5           | Black Truffle  | Luxury            | 1469        | 924        | 707       | 217       |
| 6           | Abalone        | Shellfish         | 1525        | 932        | 699       | 233       |
| 7           | Lobster        | Shellfish         | 1547        | 968        | 754       | 214       |
| 8           | Crab           | Shellfish         | 1564        | 949        | 719       | 230       |
| 9           | Oyster         | Shellfish         | 1568        | 943        | 726       | 217       |

| product\_category | page\_views | cart\_adds | purchases | abandoned |
| ----------------- | ----------- | ---------- | --------- | --------- |
| Fish              | 4633        | 2789       | 2115      | 674       |
| Luxury            | 3032        | 1870       | 1404      | 466       |
| Shellfish         | 6204        | 3792       | 2898      | 894       |
Use your 2 new output tables - answer the following questions:

### 1.  Which product had the most views, cart adds and purchases?
````sql
(
  SELECT
    product_name,
    'Most views' AS Type
  FROM
    product_info
  ORDER BY
    page_views DESC
  LIMIT
    1
)
UNION
  (
    SELECT
      product_name,
      'Most cart adds' AS Type
    FROM
      product_info
    ORDER BY
      cart_adds DESC
    LIMIT
      1
  )
UNION
  (
    SELECT
      product_name,
      'Most purhchases' AS Type
    FROM
      product_info
    ORDER BY
      purchases DESC
    LIMIT
      1
  )
````
**Answer**
| product\_name | type            |
| ------------- | --------------- |
| Oyster        | Most views      |
| Lobster       | Most purhchases |
| Lobster       | Most cart adds  |

### 2.  Which product was most likely to be abandoned?
````sql

````
**Answer**

❗**Note**
* The question mentions "most likely", which means it asks for the **probability** of product to be abandoned
### 3.  Which product had the highest view to purchase percentage?
````sql
SELECT
  product_name,
  ROUND(100 * purchases :: NUMERIC / page_views, 2)
FROM
  product_info
ORDER BY
  2 DESC
LIMIT
  1
````
**Answer**
|product_name|round  |
|--|--|
|Lobster  |48.74  |

### 4.  What is the average conversion rate from view to cart add?
````sql
SELECT
  ROUND(
    AVG(100 * cart_adds :: NUMERIC / page_views),
    2
  ) AS average_view_to_cart_rate
FROM
  product_info
````
**Answer**
| average_view_to_cart_rate |  
|--|
| 60.95 |  

### 5.  What is the average conversion rate from cart add to purchase?
````sql
SELECT
  ROUND(
    AVG(100 * purchases :: NUMERIC / cart_adds),
    2
  ) AS average_cart_to_purchase_rate
FROM
  product_info
````
**Answer**
| average_cart_to_purchase_rate|  
|--|
| 75.93 | 

# C. Campaigns Analysis
Use the subsequent dataset to generate at least 5 insights for the Clique Bait team - bonus: prepare a single A4 infographic that the team can use for their management reporting sessions, be sure to emphasise the most important points from your findings.

Some ideas you might want to investigate further include:

-   Identifying users who have received impressions during each campaign period and comparing each metric with other users who did not have an impression event
-   Does clicking on an impression lead to higher purchase rates?
-   What is the uplift in purchase rate when comparing users who click on a campaign impression versus users who do not receive an impression? What if we compare them with users who just an impression but do not click?
-   What metrics can you use to quantify the success or failure of each campaign compared to eachother?

````sql
WITH cte AS(
  SELECT
    visit_id,
    STRING_AGG(
      page_name,
      ', '
      ORDER BY
        sequence_number
    ) AS cart_products
  FROM
    clique_bait.events t1
    INNER JOIN clique_bait.page_hierarchy t2 ON t1.page_id = t2.page_id
  WHERE
    event_type = 2
  GROUP BY
    1
)
SELECT
  user_id,
  t1.visit_id,
  MIN(event_time) AS visit_start_time,
  SUM(
    CASE
      WHEN event_type = 1 THEN 1
      ELSE 0
    END
  ) AS page_views,
  SUM(
    CASE
      WHEN event_type = 2 THEN 1
      ELSE 0
    END
  ) AS cart_adds,
  SUM(
    CASE
      WHEN event_type = 3 THEN 1
      ELSE 0
    END
  ) AS purchase,
  campaign_name,
  SUM(
    CASE
      WHEN event_type = 4 THEN 1
      ELSE 0
    END
  ) AS impression,
  SUM(
    CASE
      WHEN event_type = 5 THEN 1
      ELSE 0
    END
  ) AS click,
  CASE
    WHEN cart_products IS NULL THEN ''
    ELSE cart_products
  END AS cart_products
FROM
  clique_bait.events t1
  LEFT JOIN clique_bait.users t2 ON t1.cookie_id = t2.cookie_id
  LEFT JOIN clique_bait.campaign_identifier t3 ON event_time BETWEEN t3.start_date
  AND end_date
  LEFT JOIN cte ON t1.visit_id = cte.visit_id
GROUP BY
  1,
  2,
  7,
  10
````
**Answer**
The first 5 visit_id are:
| user\_id | visit\_id | visit\_start\_time       | page\_views | cart\_adds | purchase | campaign\_name                    | impression | click | cart\_products                                                 |
| -------- | --------- | ------------------------ | ----------- | ---------- | -------- | --------------------------------- | ---------- | ----- | -------------------------------------------------------------- |
| 1        | 02a5d5    | 2020-02-26T16:57:26.261Z | 4           | 0          | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     |                                                                |
| 1        | 0826dc    | 2020-02-26T05:58:37.919Z | 1           | 0          | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     |                                                                |
| 1        | 0fc437    | 2020-02-04T17:49:49.603Z | 10          | 6          | 1        | Half Off - Treat Your Shellf(ish) | 1          | 1     | Tuna, Russian Caviar, Black Truffle, Abalone, Crab, Oyster     |
| 1        | 30b94d    | 2020-03-15T13:12:54.024Z | 9           | 7          | 1        | Half Off - Treat Your Shellf(ish) | 1          | 1     | Salmon, Kingfish, Tuna, Russian Caviar, Abalone, Lobster, Crab |
| 1        | 41355d    | 2020-03-25T00:11:17.861Z | 6           | 1          | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     | Lobster                                                        |

❗**Note**
* As each `user_id` can have multiple `cookie_id` in `clique_bait.users` and each `cookie_id` can have multiple `visit_id` in `clique_bait.events`, we use `LEFT JOIN` with  `clique_bait.events` as the base table to put `user_id` and `visit_id` together
* To `JOIN` `clique_bait.events` and `clique_bait.campaign_identifier`, use `event_time BETWEEN start_date AND end_date` as the condition for `ON` clause
* To merge products together in the order they were added to cart, we add `ORDER BY` to `STRING_AGG` as follows:
	````sql
	STRING_AGG(
	      page_name,
	      ', '
	      ORDER BY
	        sequence_number
  ) AS cart_products 
	````