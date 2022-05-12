# 🍕 A. Pizza Metrics

### 1. How many pizzas were ordered?
````sql
SELECT
  COUNT(*) AS number_of_ordered_pizzas
FROM  pizza_runner.customer_orders
````
**Answer**
| number_of_ordered_pizzas | 
| ------------ | 
| 14            | 
***
### 2.  How many unique customer orders were made?
````sql
SELECT
  COUNT(DISTINCT order_id) AS unique_customer_order
FROM
  pizza_runner.customer_orders
````
 **Answer**
| unique_customer_order | 
| ------------ | 
| 10           | 
***
### 3.  How many successful orders were delivered by each runner?
````sql
SELECT 
  runner_id,
  COUNT(DISTINCT order_id)
FROM pizza_runner.runner_orders
WHERE distance <> 'null'
GROUP BY runner_id
````
**Answer**
| runner\_id | count |
| ---------- | ----- |
| 1          | 4     |
| 2          | 3     |
| 3          | 1     |
***
### 4.  How many of each type of pizza was delivered?
````sql
SELECT
  t2.pizza_name,
  COUNT(t1.pizza_id) AS number_of_order
FROM
  pizza_runner.customer_orders AS t1
  INNER JOIN pizza_runner.pizza_names AS t2 ON t1.pizza_id = t2.pizza_id
  INNER JOIN pizza_runner.runner_orders AS t3 ON t1.order_id = t3.order_id
WHERE
  t3.distance <> 'null'
GROUP BY
  t2.pizza_name
````

**New Approach**
>A more efficient approach would be using LEFT SEMI JOIN with `WHERE EXISTS`*
````sql
SELECT
  t2.pizza_name,
  COUNT(t1.pizza_id) AS number_of_order
FROM
  pizza_runner.customer_orders AS t1
  INNER JOIN pizza_runner.pizza_names AS t2 ON t1.pizza_id = t2.pizza_id
WHERE
  EXISTS(
    SELECT
      1
    FROM
      pizza_runner.runner_orders
    WHERE
      t1.order_id = pizza_runner.runner_orders.order_id AND distance <> 'null'
  )
GROUP BY
  t2.pizza_name
````

**Answer**
| pizza\_name | number\_of\_order |
| ----------- | ----------------- |
| Meatlovers  | 9                 |
| Vegetarian  | 3                 |
**Step**
* Be cautious that we have to exclude the canceled orders , that's why we have to join 3 different tables in this question

🧠 **Best Practice**
* Instead of only using `INNER JOIN` like in the first approach, using `WHERE EXISTS` is more efficient as it doesn't return the whole table
***
### 5.  How many Vegetarian and Meatlovers were ordered by each customer?
````sql
SELECT
  t1.customer_id,
  t2.pizza_name,
  COUNT(t1.pizza_id) AS number_of_order
FROM
  pizza_runner.customer_orders AS t1
  INNER JOIN pizza_runner.pizza_names AS t2 ON t1.pizza_id = t2.pizza_id
GROUP BY
  1,
  2
ORDER BY 
  1,
  2
 ````
 **Answer**
 | customer\_id | pizza\_name | number\_of\_order |
| ------------ | ----------- | ----------------- |
| 101          | Meatlovers  | 2                 |
| 101          | Vegetarian  | 1                 |
| 102          | Meatlovers  | 2                 |
| 102          | Vegetarian  | 1                 |
| 103          | Meatlovers  | 3                 |
| 103          | Vegetarian  | 1                 |
| 104          | Meatlovers  | 3                 |
| 105          | Vegetarian  | 1      

**A New Approach**          
>Assume we already know the pizza_id of each type of pizza, we can use a different approach for a clearer result*
````sql
SELECT
  customer_id,
  SUM(CASE WHEN pizza_id = 1 THEN 1 ELSE 0 END) AS Meatlovers,
  SUM(CASE WHEN pizza_id = 2 THEN 1 ELSE 0 END) AS Vegetarian
FROM
  pizza_runner.customer_orders
GROUP BY
  customer_id
ORDER BY 
  customer_id
````
**Answer**
| customer\_id | meatlovers | vegetarian |
| ------------ | ---------- | ---------- |
| 101          | 2          | 1          |
| 102          | 2          | 1          |
| 103          | 3          | 1          |
| 104          | 3          | 0          |
| 105          | 0          | 1          |
**Step**
* As the question doesn't mention anything about successful delivered, we don't need to exclude cancelled orders
*  The second approach with `SUM CASE WHEN` can be especially helpful if there are more than just 2 types of pizza
***
### 6.  What was the maximum number of pizzas delivered in a single order?
````sql
WITH cte AS(
  SELECT
    order_id,
    COUNT(pizza_id) AS number_of_pizzas,
    DENSE_RANK() OVER(
      ORDER BY
        COUNT(pizza_id) DESC
    ) AS rank
  FROM
    pizza_runner.customer_orders
  WHERE
    EXISTS(
      SELECT
        1
      FROM
        pizza_runner.runner_orders
      WHERE
        pizza_runner.customer_orders.order_id = pizza_runner.runner_orders.order_id
        AND distance <> 'null'
    )
  GROUP BY
    order_id
)
SELECT
  number_of_pizzas
FROM
  cte
WHERE
  rank = 1
````
**Answer**
| number_of_pizzas | 
| ------------ | 
| 3            | 

**Step**
* As the question asks for the number of "ordered pizzas", remember to exclude the cancelled ones
***
### 7.  For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
````sql
SELECT
  customer_id,
  SUM(
    CASE
      WHEN exclusions NOT IN ('', 'null')
      OR extras NOT IN ('', 'null') OR extras IS NOT NULL THEN 1
      ELSE 0
    END
  ) AS at_least_1_change,
  SUM(
    CASE
      WHEN exclusions IN ('', 'null')
      AND (extras IN ('', 'null') OR extras IS NULL) THEN 1
      ELSE 0
    END
  ) AS no_change
FROM
  pizza_runner.customer_orders
WHERE
  EXISTS(
    SELECT
      1
    FROM
      pizza_runner.runner_orders
    WHERE
      pizza_runner.customer_orders.order_id = pizza_runner.runner_orders.order_id
      AND distance <> 'null'
  )
GROUP BY
  1
ORDER BY
  1
````
❗ **Note**
* Use `''` to indicate whitespace 
* Differentiate between *`null`*, `'NULL' `and `''`

***
### 8.  How many pizzas were delivered that had both exclusions and extras?
````sql
SELECT
  SUM(
    CASE
      WHEN exclusions NOT IN ('', 'null')
      AND extras NOT IN ('', 'null') THEN 1
      ELSE NULL
    END
  ) AS count
FROM
  pizza_runner.customer_orders
WHERE
  EXISTS(
    SELECT
      1
    FROM
      pizza_runner.runner_orders
    WHERE
      pizza_runner.customer_orders.order_id = pizza_runner.runner_orders.order_id
      AND distance <> 'null'
  )
````
**Answer**
| count				 | 
| ------------ | 
| 2            | 
**Step**
* As the question asks for "delivered pizzas", remember to exclude cancelled orders

🧠 **Best Practice**
 *Final note of `SUM CASE WHEN`*
 
After having sold the above questions, you may notice some signs indicating possible application of `SUM CASE WHEN`:
*  When we must use `COUNT` together with `GROUP BY` 
	* Question 5
* When we must create new columns using `CASE WHEN` and return a sum number
	* Question 7
	* Question 8
***
### 9.  What was the total volume of pizzas ordered for each hour of the day?
````sql
SELECT 
  DATE_PART('hour',order_time) AS hour,
  COUNT(*)
FROM pizza_runner.customer_orders
GROUP BY 1
ORDER BY 1
````
**Answer**
| hour | count |
| ---- | ----- |
| 11   | 1     |
| 13   | 3     |
| 18   | 3     |
| 19   | 1     |
| 21   | 3     |
| 23   | 3     |
❗ **Note**
* Differentiate between `DATE_TRUNC` and `DATE_PART` when grouping by hour
	* `DATE_TRUNC` returns a `TIMESTAMP` based off the field input, which means 11:00:00 on April 11th is different from 11:00:00 April 12th
	* `DATE_PART` returns an `INTEGER`, which allows us to group hours from different date as above
***
### 10.  What was the volume of orders for each day of the week?
````sql
SELECT
  TO_CHAR(order_time,'Day') as day_of_week,
  COUNT(*)
FROM pizza_runner.customer_orders
GROUP BY 1
ORDER BY 2
````
**Answer**
| day\_of\_week | count |
| ------------- | ----- |
| Sunday        | 1     |
| Saturday      | 3     |
| Monday        | 5     |
| Friday        | 5     |

❗ **Note**
* Notice how the `TO_CHAR` second argument can be changed slightly to change the output based off the exact format.
<a href="https://imgbb.com/"><img src="https://i.ibb.co/WnXHyJx/Capture.png" alt="Capture" border="0"></a>
***
# 🛵 B. Runner and Customer Experience

### 1.  How many runners signed up for each 1 week period? (i.e. week starts `2021-01-01`)
````sql
SELECT
  (
    DATE_TRUNC('week', registration_date - INTERVAL '4 DAY') + INTERVAL '4 DAY'
  ) :: DATE AS start_of_week,
  COUNT(DISTINCT runner_id)
FROM
  pizza_runner.runners
GROUP BY
  1
````
**Answer**
| start\_of\_week| count |
| ---------------| ----- |
| 2021-01-01 | 2     |
| 2021-01-08 | 1     |
| 2021-01-15| 1     |

**Step**
* Use `DATE_TRUNCT('week', registration_date)::DATE` to
* As week starts on `2021-01-01` as the question requires, we must

### 2.  What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
````sql
WITH cte AS(
  SELECT
    DISTINCT t1.order_id,
    t2.runner_id,
    (
      EXTRACT(
        EPOCH
        FROM
          t2.pickup_time :: TIMESTAMP
      ) - EXTRACT(
        EPOCH
        FROM
          t1.order_time
      )
    ) / 60 as difference_in_minutes
  FROM
    pizza_runner.customer_orders AS t1
    INNER JOIN pizza_runner.runner_orders AS t2 ON t1.order_id = t2.order_id
  WHERE
    t2.pickup_time <> 'null'
)
SELECT
  runner_id,
  CEILING(AVG(difference_in_minutes) :: NUMERIC) AS average_time_in_minutes
FROM
  cte
GROUP BY
  1
ORDER BY
  1
````
**Answer**
| runner\_id | average\_time\_in\_minutes |
| ---------- | -------------------------- |
| 1          | 14                         |
| 2          | 20                         |
| 3          | 10                         |
**Step**
* As `pickup_time` is type `varchar`, we must firstly cast this column into `timestamp`
* We'll use `EXTRACT(EPOCH FROM TIMESTAMP) to calculate difference between two TIMESTAMP`
* Be cautious that `EXTRACT(EPOCH FROM t2.pickup_time :: TIMESTAMP) - EXTRACT(EPOCH FROM t1.order_time)` will return the difference of timestamp in seconds, so we must divide the result by 60
### 3.  Is there any relationship between the number of pizzas and how long the order takes to prepare?
````sql
WITH cte AS(
  SELECT
    t1.order_id,
    (
      EXTRACT(
        EPOCH
        FROM
          t2.pickup_time :: TIMESTAMP
      ) - EXTRACT(
        EPOCH
        FROM
          t1.order_time
      )
    ) / 60 as difference_in_minutes
  FROM
    pizza_runner.customer_orders AS t1
    INNER JOIN pizza_runner.runner_orders AS t2 ON t1.order_id = t2.order_id
  WHERE
    t2.pickup_time <> 'null'
)
SELECT
  order_id,
  COUNT(*) AS number_of_pizzas,
  CEILING(AVG(difference_in_minutes) :: NUMERIC) AS average_time_in_minutes
FROM
  cte
GROUP BY
  1
ORDER BY
  1
````
**Answer**
| order\_id | number\_of\_pizzas| average\_time\_in\_minutes |
| --------- | ----- | -------------------------- |
| 1         | 1     | 11                         |
| 2         | 1     | 10                         |
| 3         | 2     | 21                         |
| 4         | 3     | 29                         |
| 5         | 1     | 10                         |
| 7         | 1     | 10                         |
| 8         | 1     | 20                         |
| 10        | 2     | 16                         

**Step**
* Just by simply scanning through the values above, we can suggest there might exist a positive correlation between the number of pizzas and the average time to finish an order, which makes perfect sense in real life! 
### 4.  What was the average distance travelled for each customer?
````sql
WITH cte AS (
  SELECT
    DISTINCT t1.order_id,
    t2.customer_id,
    UNNEST(REGEXP_MATCH(distance, '^[0-9,.]+')) :: NUMERIC AS new_distance
  FROM
    pizza_runner.runner_orders t1
    INNER JOIN pizza_runner.customer_orders AS t2 ON t1.order_id = t2.order_id
  WHERE
    distance <> 'null'
)
SELECT
  customer_id,
  ROUND(AVG(new_distance), 1) average_distance
FROM
  cte
GROUP BY
  1
ORDER BY
  1
````
**Answer**
| customer\_id | average\_distance |
| ------------ | ----------------- |
| 101          | 20.0              |
| 102          | 18.4              |
| 103          | 23.4              |
| 104          | 10.0              |
| 105          | 25.0              |
**Step**
* Use `UNNEST(REGEXP_MATCH())` to only return what matches our regular expression  
* For the CTE, use `DISTINCT` with both `customer_id` and `order_id` to make sure there isn't any missing data as there are duplicates of 
	* `order_id` from table `customer_orders`
	* `new_distance` values

### 5.  What was the difference between the longest and shortest delivery times for all orders?
````sql
WITH cte AS(
  SELECT
    (
      UNNEST(REGEXP_MATCH(duration, '^[0-9]{2}')) :: NUMERIC
    ) AS new_duration
  FROM
    pizza_runner.runner_orders
  WHERE
    distance <> 'null'
)
SELECT
  MAX(new_duration)-MIN(new_duration) AS difference_between_max_and_min
FROM
  cte
````
**Answer**
| difference_between_max_and_min| 
| ------------ | 
| 30| 

### 6.  What was the average speed for each runner for each delivery and do you notice any trend for these values?
````sql
WITH cte AS(
  SELECT
    DISTINCT t1.runner_id,
    t1.order_id,
    t2.customer_id,
    DATE_PART('hour', pickup_time :: TIMESTAMP) AS PU_hour,
    (
      UNNEST(REGEXP_MATCH(distance, '^[0-9,.]+')) :: NUMERIC
    ) AS new_distance,
    (
      UNNEST(REGEXP_MATCH(duration, '^[0-9]{2}')) :: NUMERIC
    ) AS new_duration
  FROM
    pizza_runner.runner_orders AS t1
    INNER JOIN pizza_runner.customer_orders AS t2 ON t1.order_id = t2.order_id
  WHERE
    distance <> 'null'
)
SELECT
  runner_id,
  order_id,
  customer_id,
  PU_hour,
  CEILING(new_distance / (new_duration / 60)) as average_speed
FROM
  cte
ORDER BY
  1,
  5
````
**Answer**
| runner\_id | order\_id | customer\_id | pu\_hour | average\_speed |
| ---------- | --------- | ------------ | -------- | -------------- |
| 1          | 1         | 101          | 18       | 38             |
| 1          | 3         | 102          | 0        | 40             |
| 1          | 2         | 101          | 19       | 44             |
| 1          | 10        | 104          | 18       | 60             |
| 2          | 4         | 103          | 13       | 35             |
| 2          | 7         | 105          | 21       | 60             |
| 2          | 8         | 102          | 0        | 94             |
| 3          | 5         | 104          | 21       | 40             |
**Step**
*  Regarding the runners and their speed per order, we can have some comments:
	* Runner_id 1's speed ranges from 38 to 60 km/h
	* Runner_id 2's speed ranges from 35 to 94 km/h, which is quite absurd and Danny might want to deep dive into this
	* Runner_id 3's speed is 40 km/h
* Take into account the customer_id, we also notice that 
	* customer_id 2 tend to have his order picked up around midnight
	* customer_id 1's order tend to be picked up around dinner time
### 7.  What is the successful delivery percentage for each runner?
````sql
SELECT
  runner_id,
  CEILING(
    100 * (
      SUM(
        CASE
          WHEN distance != 'null' THEN 1
          ELSE 0
        END
      ) / SUM(COUNT(*)) OVER(PARTITION BY runner_id)
    )
  ) AS success_rate
FROM
  pizza_runner.runner_orders
GROUP BY
  1
ORDER BY
  1
````
**Answer**
| runner\_id | success\_rate |
| ---------- | ------------- |
| 1          | 100           |
| 2          | 75            |
| 3          | 50            |

# 🧂 C. Ingredient Optimisation

### 1.  What are the standard ingredients for each pizza?
````sql
WITH new_pizza_recipes AS (
  SELECT
    pizza_id,
    REGEXP_SPLIT_TO_TABLE(toppings, ',\s') :: INTEGER AS topping_id
  FROM
    pizza_runner.pizza_recipes
)
SELECT
  t1.pizza_id,
  STRING_AGG(t2.topping_name, ', ') as ingredients
FROM
  new_pizza_recipes AS t1
  INNER JOIN pizza_runner.pizza_toppings AS t2 ON t1.topping_id = t2.topping_id
GROUP BY
  1
ORDER BY
  1
````
**Answer**
| pizza\_id | ingredients                                                           |
| --------- | --------------------------------------------------------------------- |
| 1         | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami |
| 2         | Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce            |
**Step** 
* First, use `REGEXP_SPLIT_TO_TABLE(toppings, ',\s')` to create new rows for each part that is split from the text field using a specific regular expression, in this case, `',\s` (`\s` indicates whitespace)
* Then, cast the new column into `INTEGER` to eliminate whitespace, which may cause trouble when calculating frequencies of values or joining
* Next, use `STRING_AGG(t2.topping_name, ', ')`  along with `GROUP BY` to concatenate all ingredients by each `pizza_id`
### 2.  What was the most commonly added extra?
````sql
WITH cte AS(
  SELECT
    REGEXP_SPLIT_TO_TABLE(extras, ',\s') AS new_extras
  FROM
    pizza_runner.customer_orders
  WHERE
    extras IS NOT NULL
    AND extras <> 'null'
)
SELECT
  t2.topping_name,
  COUNT(*) AS frequency
FROM
  cte AS t1
  INNER JOIN pizza_runner.pizza_toppings AS t2 ON t1.new_extras::INTEGER = t2.topping_id
WHERE
  new_extras <> ''
GROUP BY
  1
ORDER BY
  2 DESC,
  1 
````
**Answer**
| topping\_name | frequency |
| ------------- | --------- |
| Bacon         | 4         |
| Cheese        | 1         |
| Chicken       | 1         |

### 3.  What was the most common exclusion?
````sql
WITH cte AS(
  SELECT
    REGEXP_SPLIT_TO_TABLE(exclusions, ',\s') AS new_exclusions
  FROM
    pizza_runner.customer_orders
  WHERE
    exclusions <> 'null'
)
SELECT
  t2.topping_name,
  COUNT(*) AS frequency
FROM
  cte AS t1
  INNER JOIN pizza_runner.pizza_toppings AS t2 ON t1.new_exclusions :: INTEGER = t2.topping_id
WHERE
  new_exclusions <> ''
GROUP BY
  1
ORDER BY
  2 DESC,
  1
````

**Answer**
| topping\_name | frequency |
| ------------- | --------- |
| Cheese        | 4         |
| BBQ Sauce     | 1         |
| Mushrooms     | 1         |

### 4.  Generate an order item for each record in the `customers_orders` table in the format of one of the following:

-   `Meat Lovers`
-   `Meat Lovers - Exclude Beef`
-   `Meat Lovers - Extra Bacon`
-   `Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers`

````sql
WITH cte_cleaned_customer_orders AS(
  SELECT
    ROW_NUMBER() OVER() as index,
    -- for STRING_AGG and GROUP BY
    *,
    CASE
      WHEN exclusions IN ('','null') THEN NULL
      ELSE exclusions
    END AS cleaned_exclusions,
    CASE
      WHEN extras IN ('','null') THEN NULL
      ELSE extras
    END AS cleaned_extras
  FROM
    pizza_runner.customer_orders
),
cte_spLit AS (
  SELECT
    *,
    REGEXP_SPLIT_TO_TABLE(cleaned_exclusions, ',\s') :: INTEGER AS full_exclusions,
    REGEXP_SPLIT_TO_TABLE(cleaned_extras, ',\s') :: INTEGER AS full_extras
  FROM
    cte_cleaned_customer_orders
),
cte_join AS(
  SELECT
    t1.index,
    t1.order_id,
    t1.pizza_id,
    t1.customer_id,
    t1.order_time,
    t2.pizza_name,
    t3.topping_name AS exclusions_name,
    t4.topping_name AS extras_name
  FROM
    cte_split AS t1
    INNER JOIN pizza_runner.pizza_names AS t2 ON t1.pizza_id = t2.pizza_id
    LEFT JOIN pizza_runner.pizza_toppings AS t3 ON t1.full_exclusions = t3.topping_id
    LEFT JOIN pizza_runner.pizza_toppings AS t4 ON t1.full_extras = t4.topping_id
  UNION ALL
  SELECT
    t1.index,
    t1.order_id,
    t1.pizza_id,
    t1.customer_id,
    t1.order_time,
    t2.pizza_name,
    cleaned_exclusions AS exclusions_name,
    -- Because they're all NULL
    cleaned_extras AS extras_name
  FROM
    cte_cleaned_customer_orders AS t1
    INNER JOIN pizza_runner.pizza_names AS t2 ON t1.pizza_id = t2.pizza_id
  WHERE
    cleaned_exclusions IS NULL
    AND cleaned_extras IS NULL
),
cte_string_aggregation AS (
  SELECT
    index,
    order_id,
    pizza_id,
    pizza_name,
    customer_id,
    order_time,
    STRING_AGG(exclusions_name, ', ') AS exclusions,
    STRING_AGG(extras_name, ', ') AS extras
  FROM
    cte_join cte_join
  GROUP BY
    index,
    order_id,
    pizza_id,
    pizza_name,
    customer_id,
    order_time
  ORDER BY
    index
)
SELECT
  order_id,
  pizza_id,
  customer_id,
  CASE
    WHEN exclusions IS NULL
    AND extras IS NULL THEN pizza_name
    WHEN exclusions IS NOT NULL
    AND extras IS NOT NULL THEN pizza_name || ' - Exclude ' || exclusions || ' - Extra ' || extras
    WHEN exclusions IS NOT NULL
    AND extras IS NULL THEN pizza_name || ' - Exclude ' || exclusions
    WHEN exclusions IS NULL
    AND extras IS NOT NULL THEN pizza_name || ' - Extra ' || extras
  END AS ordered_item,
  order_time
FROM
  cte_string_aggregation
````

**Answer**
| order\_id | pizza\_id | customer\_id | ordered\_item                                                   | order\_time              |
| --------- | --------- | ------------ | --------------------------------------------------------------- | ------------------------ |
| 1         | 1         | 101          | Meatlovers                                                      | 2021-01-01T18:05:02.000Z |
| 2         | 1         | 101          | Meatlovers                                                      | 2021-01-01T19:00:52.000Z |
| 3         | 1         | 102          | Meatlovers                                                      | 2021-01-02T23:51:23.000Z |
| 3         | 2         | 102          | Vegetarian                                                      | 2021-01-02T23:51:23.000Z |
| 4         | 1         | 103          | Meatlovers - Exclude Cheese                                     | 2021-01-04T13:23:46.000Z |
| 4         | 1         | 103          | Meatlovers - Exclude Cheese                                     | 2021-01-04T13:23:46.000Z |
| 4         | 2         | 103          | Vegetarian - Exclude Cheese                                     | 2021-01-04T13:23:46.000Z |
| 5         | 1         | 104          | Meatlovers - Extra Bacon                                        | 2021-01-08T21:00:29.000Z |
| 6         | 2         | 101          | Vegetarian                                                      | 2021-01-08T21:03:13.000Z |
| 7         | 2         | 105          | Vegetarian - Extra Bacon                                        | 2021-01-08T21:20:29.000Z |
| 8         | 1         | 102          | Meatlovers                                                      | 2021-01-09T23:54:33.000Z |
| 9         | 1         | 103          | Meatlovers - Exclude Cheese - Extra Bacon, Chicken              | 2021-01-10T11:22:59.000Z |
| 10        | 1         | 104          | Meatlovers                                                      | 2021-01-11T18:34:49.000Z |
| 10        | 1         | 104          | Meatlovers - Exclude Mushrooms, BBQ Sauce - Extra Cheese, Bacon | 2021-01-11T18:34:49.000Z |

**Step**
* Create index with window function `ROW_NUMBER()` to uniquely identify each ordered pizza
* We must first create a CTE with values of `extras` and `exclusions` having been splitted into separate rows
❗ **Note**
	* When using the `REGEXP_SPLIT_TO_TABLE` ,only records where there aren't  *`null`*  are returned, thus we don't need to filter with `WHERE` before using this function
* Afterwards, we use `UNION ALL` (literally `UNION` but with no imxplicit `DISTINCT`) to merge the current CTE with the one having
* Use `LEFT JOIN` as we want to also keep all records from our base tables (including those with *`null`* in `extras` and `exlusions`) when looking up ingredients names
*  Use `STRING_AGG` along with `GROUP BY` using the newly created index column from the start to finish our final values for `extras` and `exclusions` columns
* Use `CASE WHEN` to finish our final table
### 5.  Generate an alphabetically ordered comma separated ingredient list for each pizza order from the `customer_orders` table and add a `2x` in front of any relevant ingredients

-   For example: `"Meat Lovers: 2xBacon, Beef, ... , Salami"`

````sql
WITH cte_cleaned_customer_orders AS(
  SELECT
    ROW_NUMBER() OVER() as index,
    *,
    CASE
      WHEN exclusions IN ('', 'null') THEN NULL
      ELSE exclusions
    END AS cleaned_exclusions,
    CASE
      WHEN extras IN ('', 'null') THEN NULL
      ELSE extras
    END AS cleaned_extras
  FROM
    pizza_runner.customer_orders
),
cte_recipes AS(
  SELECT
    t1.*,
    t2.toppings
  FROM
    cte_cleaned_customer_orders AS t1
    INNER JOIN pizza_runner.pizza_recipes AS t2 ON t1.pizza_id = t2.pizza_id
),
cte_toppings AS(
  SELECT
    index,
    order_id,
    pizza_id,
    REGEXP_SPLIT_TO_TABLE(toppings, ',\s') :: INTEGER AS ingredients
  FROM
    cte_recipes
),
cte_exclusions AS(
  SELECT
    index,
    order_id,
    pizza_id,
    REGEXP_SPLIT_TO_TABLE(cleaned_exclusions, ',\s') :: INTEGER AS ingredients
  FROM
    cte_recipes
),
cte_extras AS(
  SELECT
    index,
    order_id,
    pizza_id,
    REGEXP_SPLIT_TO_TABLE(cleaned_extras, ',\s') :: INTEGER AS ingredients
  FROM
    cte_recipes
),
cte_ingredients AS (
  SELECT
    *
  FROM
    cte_toppings
  EXCEPT
  SELECT
    *
  FROM
    cte_exclusions
  UNION ALL
  SELECT
    *
  FROM
    cte_extras
),
cte_frequency AS (
  SELECT
    t1.index,
    t1.ingredients,
    (
      CASE
        WHEN COUNT(*) = 1 THEN ' '
        ELSE COUNT(*) :: INTEGER || 'x '
      END
    ) || t2.topping_name AS frequency
  FROM
    cte_ingredients AS t1
    INNER JOIN pizza_runner.pizza_toppings AS t2 ON t1.ingredients = t2.topping_id
  GROUP BY
    1,
    2,
    t2.topping_name
  ORDER BY
    1,
    2
)
SELECT
  t1.index,
  t2.order_id,
  t2.customer_id,
  t2.pizza_id,
  t2.order_time,
  STRING_AGG(TRIM(t1.frequency), ', ') AS ingredients_list
FROM
  cte_frequency AS t1
  INNER JOIN cte_cleaned_customer_orders AS t2 ON t1.index = t2.index 
GROUP BY
  1,2,3,4,5
ORDER BY
  1
````

**Answer**
| index | order\_id | customer\_id | pizza\_id | order\_time              | ingredients\_list                                                        |
| ----- | --------- | ------------ | --------- | ------------------------ | ------------------------------------------------------------------------ |
| 1     | 1         | 101          | 1         | 2021-01-01T18:05:02.000Z | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |
| 2     | 2         | 101          | 1         | 2021-01-01T19:00:52.000Z | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |
| 3     | 3         | 102          | 1         | 2021-01-02T23:51:23.000Z | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |
| 4     | 3         | 102          | 2         | 2021-01-02T23:51:23.000Z | Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce               |
| 5     | 4         | 103          | 1         | 2021-01-04T13:23:46.000Z | Bacon, BBQ Sauce, Beef, Chicken, Mushrooms, Pepperoni, Salami            |
| 6     | 4         | 103          | 1         | 2021-01-04T13:23:46.000Z | Bacon, BBQ Sauce, Beef, Chicken, Mushrooms, Pepperoni, Salami            |
| 7     | 4         | 103          | 2         | 2021-01-04T13:23:46.000Z | Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce                       |
| 8     | 5         | 104          | 1         | 2021-01-08T21:00:29.000Z | 2x Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami |
| 9     | 6         | 101          | 2         | 2021-01-08T21:03:13.000Z | Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce               |
| 10    | 7         | 105          | 2         | 2021-01-08T21:20:29.000Z | Bacon, Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce        |
| 11    | 8         | 102          | 1         | 2021-01-09T23:54:33.000Z | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |
| 12    | 9         | 103          | 1         | 2021-01-10T11:22:59.000Z | 2x Bacon, BBQ Sauce, Beef, 2x Chicken, Mushrooms, Pepperoni, Salami      |
| 13    | 10        | 104          | 1         | 2021-01-11T18:34:49.000Z | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |
| 14    | 10        | 104          | 1         | 2021-01-11T18:34:49.000Z | 2x Bacon, Beef, 2x Cheese, Chicken, Pepperoni, Salami                    |

**Step**
* After cleaning the original table, create 3 separate CTE with splitted rows of `ingredients`, `extras` and `exclusions`
* Use `EXCEPT` and `UNION ALL` to merge the 3 CTEs 
❗ **Note**
	*  `EXCEPT` has a default `DISTINCT` built in so it’s doing the same deduplication as the `UNION`. Thus, we must use `EXCEPT` before `UNION ALL` in our case to not let the extras be deleted
 * Use `TRIM` to remove any redundant whitespace before using `STRING_AGG`
### 6.  What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?
````sql
WITH cte_cleaned_customer_orders AS(
  SELECT
    ROW_NUMBER() OVER() as index,
    *,
    CASE
      WHEN exclusions IN ('', 'null') THEN NULL
      ELSE exclusions
    END AS cleaned_exclusions,
    CASE
      WHEN extras IN ('', 'null') THEN NULL
      ELSE extras
    END AS cleaned_extras
  FROM
    pizza_runner.customer_orders
  WHERE
    EXISTS (
      SELECT
        1
      FROM
        pizza_runner.runner_orders
      WHERE
        pizza_runner.runner_orders.order_id = pizza_runner.customer_orders.order_id
        AND distance <> 'null'
    )
),
cte_recipes AS(
  SELECT
    t1.*,
    t2.toppings
  FROM
    cte_cleaned_customer_orders AS t1
    INNER JOIN pizza_runner.pizza_recipes AS t2 ON t1.pizza_id = t2.pizza_id
),
cte_toppings AS(
  SELECT
    index,
    order_id,
    pizza_id,
    REGEXP_SPLIT_TO_TABLE(toppings, ',\s') :: INTEGER AS ingredients
  FROM
    cte_recipes
),
cte_exclusions AS(
  SELECT
    index,
    order_id,
    pizza_id,
    REGEXP_SPLIT_TO_TABLE(cleaned_exclusions, ',\s') :: INTEGER AS ingredients
  FROM
    cte_recipes
),
cte_extras AS(
  SELECT
    index,
    order_id,
    pizza_id,
    REGEXP_SPLIT_TO_TABLE(cleaned_extras, ',\s') :: INTEGER AS ingredients
  FROM
    cte_recipes
),
cte_ingredients AS (
  SELECT
    *
  FROM
    cte_toppings
  EXCEPT
  SELECT
    *
  FROM
    cte_exclusions
  UNION ALL
  SELECT
    *
  FROM
    cte_extras
)
SELECT
  t2.topping_name,
  COUNT(*) AS frequency
FROM
  cte_ingredients AS t1
  INNER JOIN pizza_runner.pizza_toppings AS t2 ON t1.ingredients = t2.topping_id
GROUP BY
  1
ORDER BY
  2 DESC
````
**Answer**

**Step**
* Using the answer from the previous section with a few adjustments, we can solve this question with ease
* Still, remember to exclude cancelled orders, as the question ask for "delivered" orders
# 💸 D. Pricing and Ratings

### 1.  If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?
````sql
SELECT
  SUM(
    CASE
      WHEN pizza_id = 1 THEN 12
      WHEN pizza_id = 2 THEN 10
    END
  ) AS total_revenue
FROM
  pizza_runner.customer_orders
WHERE
  EXISTS(
    SELECT
      1
    FROM
      pizza_runner.runner_orders
    WHERE
      pizza_runner.customer_orders.order_id = pizza_runner.runner_orders.order_id
      AND distance <> 'null')
````

**Answer**
| total_revenue| 
| ------------ | 
| 138           | 

**Step**
* Exclude the cancelled orders
### 2.  What if there was an additional $1 charge for any pizza extras?

-   Add cheese is $1 extra
````sql
WITH cte_cleaned_customer_orders AS(
  SELECT
    ROW_NUMBER() OVER() as index,
    *,
    CASE
      WHEN exclusions IN ('', 'null') THEN NULL
      ELSE exclusions
    END AS cleaned_exclusions,
    CASE
      WHEN extras IN ('', 'null') THEN NULL
      ELSE extras
    END AS cleaned_extras
  FROM
    pizza_runner.customer_orders
  WHERE
    EXISTS(
      SELECT
        1
      FROM
        pizza_runner.runner_orders
      WHERE
        pizza_runner.customer_orders.order_id = pizza_runner.runner_orders.order_id
        AND distance <> 'null'
    )
)
SELECT
  (
    SUM(
      CASE
        WHEN pizza_id = 1 THEN 12
        WHEN pizza_id = 2 THEN 10
      END
    ) + SUM(
      COALESCE(
        CARDINALITY(REGEXP_SPLIT_TO_ARRAY(cleaned_extras, ',\s')),
        0
      )
    )
  ) AS total_revenue
FROM
  cte_cleaned_customer_orders
````
**Answer**
| total_revenue| 
| ------------ | 
| 142           | 
**Step**
* Clean `extras` & `eclusions` columns + exclude cancelled orders
* Use `REGEXP_SPLIT_TO_ARRAY` to return arrays for `extras`
* Use `CARDINALITY` to return the length of 
* Use `COALESCE`
### 3.  The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.
````sql
SELECT
  SETSEED(1);
  
DROP TABLE IF EXISTS pizza_runner.runner_rating;

CREATE TABLE pizza_runner.runner_rating ("order_id" INTEGER, "rating" INTEGER);

INSERT INTO
  pizza_runner.runner_rating
SELECT
  order_id,
  FLOOR(1 + 5 * RANDOM()) as rating
FROM
  pizza_runner.runner_orders;
WHERE 
	distance <> 'null'
 
SELECT
  *
FROM
  pizza_runner.runner_rating
````
**Answer**
| order\_id | rating |
| --------- | ------ |
| 1         | 3      |
| 2         | 4      |
| 3         | 4      |
| 4         | 3      |
| 5         | 3      |
| 7         | 2      |
| 8         | 2      |
| 10        | 3      |

**Step**
* Use `SETSEED`to create a reproducible result
* `CREATE TABLE` with two column: `order_id` and `rating`
* Use `INSERT INTO` to populate the new table
* Use `RANDOM` to return a number that is >=0 and <1
* Remember to exclude the cancelled orders
### 4.  Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?

-   `customer_id`
-   `order_id`
-   `runner_id`
-   `rating`
-   `order_time`
-   `pickup_time`
-   Time between order and pickup
-   Delivery duration
-   Average speed
-   Total number of pizzas

````sql
WITH cte AS(
  SELECT
    order_id,
    COUNT(pizza_id) as number_of_pizza
  FROM
    pizza_runner.customer_orders
  GROUP BY
    order_id
)
SELECT
  DISTINCT t1.customer_id,
  t1.order_id,
  t2.runner_id,
  t3.rating,
  t1.order_time,
  t2.pickup_time,
  CEILING(
    (
      EXTRACT(
        EPOCH
        FROM
          t2.pickup_time :: TIMESTAMP
      ) - EXTRACT(
        EPOCH
        FROM
          t1.order_time
      )
    ) / 60
  ) AS "cooking_time_(minutes)",
  UNNEST(REGEXP_MATCH(t2.duration, '[0-9]+')) AS "delivery_duration_(minutes)",
  ROUND(
    UNNEST(REGEXP_MATCH(t2.distance, '[0-9]+')) :: NUMERIC / (
      (
        UNNEST(REGEXP_MATCH(t2.duration, '[0-9]+')) :: NUMERIC
      ) / 60
    ),
    1
  ) AS "speed_(km/h)",
  t4.number_of_pizza
FROM
  pizza_runner.customer_orders AS t1
  INNER JOIN pizza_runner.runner_orders AS t2 ON t1.order_id = t2.order_id
  INNER JOIN pizza_runner.runner_rating AS t3 ON t1.order_id = t3.order_id
  INNER JOIN cte AS t4 ON t1.order_id = t4.order_id
ORDER BY
  1,
  2,
  3
````
**Answer**
| customer\_id | order\_id | runner\_id | rating | order\_time              | pickup\_time        | cooking\_time\_(minutes) | delivery\_duration\_(minutes) | speed\_(km/h) | number\_of\_pizza |
| ------------ | --------- | ---------- | ------ | ------------------------ | ------------------- | ------------------------ | ----------------------------- | ------------- | ----------------- |
| 101          | 1         | 1          | 3      | 2021-01-01T18:05:02.000Z | 2021-01-01 18:15:34 | 11                       | 32                            | 37.5          | 1                 |
| 101          | 2         | 1          | 4      | 2021-01-01T19:00:52.000Z | 2021-01-01 19:10:54 | 11                       | 27                            | 44.4          | 1                 |
| 102          | 3         | 1          | 4      | 2021-01-02T23:51:23.000Z | 2021-01-03 00:12:37 | 22                       | 20                            | 39.0          | 2                 |
| 102          | 8         | 2          | 2      | 2021-01-09T23:54:33.000Z | 2021-01-10 00:15:02 | 21                       | 15                            | 92.0          | 1                 |
| 103          | 4         | 2          | 3      | 2021-01-04T13:23:46.000Z | 2021-01-04 13:53:03 | 30                       | 40                            | 34.5          | 3                 |
| 104          | 5         | 3          | 3      | 2021-01-08T21:00:29.000Z | 2021-01-08 21:10:57 | 11                       | 15                            | 40.0          | 1                 |
| 104          | 10        | 1          | 3      | 2021-01-11T18:34:49.000Z | 2021-01-11 18:50:20 | 16                       | 10                            | 60.0          | 2                 |
| 105          | 7         | 2          | 2      | 2021-01-08T21:20:29.000Z | 2021-01-08 21:30:45 | 11                       | 25                            | 60.0          | 1                 |

### 5.  If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?
````sql
WITH cte_revenue AS(
  SELECT
    order_id,
    SUM(
      CASE
        WHEN pizza_id = 1 THEN 12
        ELSE 10
      END
    ) AS revenue
  FROM
    pizza_runner.customer_orders
  WHERE
    EXISTS(
      SELECT
        1
      FROM
        pizza_runner.runner_orders
      WHERE
        pizza_runner.runner_orders.order_id = pizza_runner.customer_orders.order_id
        AND distance <> 'null'
    )
  GROUP BY
    order_id
),
cte_cost AS (
  SELECT
    order_id,
    0.3 * UNNEST(REGEXP_MATCH(distance, '[0-9,.]+')) :: NUMERIC AS cost
  FROM
    pizza_runner.runner_orders
  WHERE
    distance <> 'null'
)
SELECT
  SUM(t1.revenue - t2.cost) AS profit
FROM
  cte_revenue AS t1
  INNER JOIN cte_cost AS t2 ON t1.order_id = t2.order_id
````

**Answer**
|  profit| 
|--|
| 94.44 | 

# E. Bonus Questions

If Danny wants to expand his range of pizzas - how would this impact the existing data design? Write an `INSERT` statement to demonstrate what would happen if a new `Supreme` pizza with all the toppings was added to the Pizza Runner menu?

* *Update `pizza_recipes` table*
	````sql
	DROP TABLE IF EXISTS temp_pizza_names;
	CREATE TEMP TABLE temp_pizza_names AS(
	  SELECT
	    *
	  FROM
	    pizza_runner.pizza_names
	);
	INSERT INTO
	  temp_pizza_names (pizza_id, pizza_name)
	VALUES(3, 'Supreme');
	SELECT
	  *
	FROM
	  temp_pizza_names
	````
	* Answer
	
	| pizza\_id | pizza\_name |
	| --------- | ----------- |
	| 1         | Meatlovers  |
	| 2         | Vegetarian  |
	| 3         | Supreme     |
	
* *Update `pizza_recipes` table*
	````sql
	DROP TABLE IF EXISTS temp_pizza_recipes;
	CREATE TEMP TABLE temp_pizza_recipes AS(
	  SELECT
	    *
	  FROM
	    pizza_runner.pizza_recipes
	);
	INSERT INTO
	  temp_pizza_recipes (pizza_id, toppings)
	VALUES(3, '1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12');
	SELECT
	  *
	FROM
	  temp_pizza_recipes
	````
	* Answer
	
	| pizza\_id | toppings                   |
	| --------- | -------------------------- |
	| 1         | 1, 2, 3, 4, 5, 6, 8, 10    |
	| 2         | 4, 6, 7, 9, 11, 12         |
	| 3         | 1,2,3,4,5,6,7,8,9,10,11,12 |

**Step**
* We create two temporary tables as above to not mess with our original data set
