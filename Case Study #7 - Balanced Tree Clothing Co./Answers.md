# 👑 A. High Level Sales Analysis

### 1.  What was the total quantity sold for all products?
````sql
SELECT
  t2.product_name,
  SUM(t1.qty) AS total_qty
FROM
  balanced_tree.sales t1
  INNER JOIN balanced_tree.product_details t2 ON t1.prod_id = t2.product_id
GROUP BY
  1
ORDER BY
  2 DESC
````
**Answer**
| product\_name                    | total\_qty |
| -------------------------------- | ---------- |
| Grey Fashion Jacket - Womens     | 3876       |
| Navy Oversized Jeans - Womens    | 3856       |
| Blue Polo Shirt - Mens           | 3819       |
| White Tee Shirt - Mens           | 3800       |
| Navy Solid Socks - Mens          | 3792       |
| Black Straight Jeans - Womens    | 3786       |
| Pink Fluro Polkadot Socks - Mens | 3770       |
| Indigo Rain Jacket - Womens      | 3757       |
| Khaki Suit Jacket - Womens       | 3752       |
| Cream Relaxed Jeans - Womens     | 3707       |
| White Striped Socks - Mens       | 3655       |
| Teal Button Up Shirt - Mens      | 3646       |
### 2.  What is the total generated revenue for all products before discounts?
````sql
SELECT
  SUM(qty * price) AS total_rev
FROM
  balanced_tree.sales
````
**Answer**
| total_rev | 
|--|
| 1289453 | 

### 3.  What was the total discount amount for all products?
````sql
SELECT
  ROUND(SUM(qty * price * discount :: NUMERIC / 100), 2) AS total_discount_amount
FROM
  balanced_tree.sales
````
**Answer**
| total_discount_amount| 
|--|
| 156229.14| 

❗ **Note**
* Remember to cast column into `NUMERIC` type before division, or you would get an integer floor division result instead (E.x., a result of 0.5 would be returned as 0)
# 💱 B. Transaction Analysis
❗ **Note**
* As `balanced_tree.sales` table contains product-level information for all transactions, we must remember to 
	* Account for unique transaction_id using `DISTINCT txn_id`  
	* Or aggregate data for unique transactions using `GROUP BY  txn_id` before performing any further calculations
### 1.  How many unique transactions were there?
````sql
SELECT
  COUNT(DISTINCT txn_id)
FROM
  balanced_tree.sales
````
**Answer**
| count| 
|--|
| 2500|

### 2.  What is the average unique products purchased in each transaction?
````sql
WITH cte AS(
  SELECT
    COUNT(DISTINCT prod_id) AS frequency
  FROM
    balanced_tree.sales
  GROUP BY
    txn_id
)
SELECT
  CEILING(AVG(frequency)) avg_unique_product_per_transaction
FROM cte 
````

**Answer**
| avg_unique_product_per_transaction| 
|--|
| 7|
### 3.  What are the 25th, 50th and 75th percentile values for the revenue per transaction?
````sql
WITH cte AS(
  SELECT
    ROUND(
      SUM((1 - discount :: NUMERIC / 100) * price * qty),
      2
    ) AS total_revenue
  FROM
    balanced_tree.sales
  GROUP BY
    txn_id
)
SELECT
  PERCENTILE_CONT(0.25) WITHIN GROUP (
    ORDER BY
      total_revenue
  ) AS percentile_cont_25,
  PERCENTILE_CONT(0.5) WITHIN GROUP (
    ORDER BY
      total_revenue
  ) AS percentile_cont_5,
  PERCENTILE_CONT(0.75) WITHIN GROUP (
    ORDER BY
      total_revenue
  ) AS percentile_cont_75,
  PERCENTILE_DISC(0.25) WITHIN GROUP (
    ORDER BY
      total_revenue
  ) AS percentile_disc_5,
  PERCENTILE_DISC(0.5) WITHIN GROUP (
    ORDER BY
      total_revenue
  ) AS percentile_disc_25,
  PERCENTILE_DISC(0.75) WITHIN GROUP (
    ORDER BY
      total_revenue
  ) AS percentile_disc_75
FROM
  cte
````
**Answer**
| percentile\_cont\_25 | percentile\_cont\_5 | percentile\_cont\_75 | percentile\_disc\_5 | percentile\_disc\_25 | percentile\_disc\_75 |
| -------------------- | ------------------- | -------------------- | ------------------- | -------------------- | -------------------- |
| 326.405              | 441.225             | 572.7625             | 326.18              | 441.00               | 572.75               |

❗ **Note**
* Differentiate between
	* `NTILE()`
		> Divide an ordered data set into a number of buckets indicated by expr and assigns the appropriate bucket number to each row
	* `PERCENTILE_DISC()`
		> Return the percentile value as a number from the distribution
	* `PERCENTILE_CONT()`
		> Not guarantee to return one of the value in the distribution. Instead, this functions interpolates the percentile. This might not be accepted in business practice, so use `PERCENTILE_DISC()` instead in such cases
### 4.  What is the average discount value per transaction?
````sql
WITH cte AS(
  SELECT
    SUM((discount :: NUMERIC / 100) * price * qty) AS total_revenue
  FROM
    balanced_tree.sales
  GROUP BY
    txn_id
)
SELECT
  ROUND(AVG(total_revenue), 2) avg_discount_per_transaction
FROM
  cte
````
**Answer**
| avg_discount_per_transaction| 
|--|
| 62.49|
### 5.  What is the percentage split of all transactions for members vs non-members?
````sql
SELECT
  member,
  COUNT(DISTINCT txn_id) AS frequency,
  ROUND(
    100 *(
      COUNT(DISTINCT txn_id) / SUM(COUNT(DISTINCT txn_id)) OVER()
    ),
    2
  ) AS percentage
FROM
  balanced_tree.sales
GROUP BY
  member
````

**Answer**
| member | frequency | percentage |
| ------ | --------- | ---------- |
| false	| 995       | 39.80      |
| true	| 1505      | 60.20      |

### 6.  What is the average revenue for member transactions and non-member transactions?
````sql
WITH cte AS(
  SELECT
    txn_id,
    member,
    SUM((1 - discount::NUMERIC / 100) * price * qty) AS total_revenue
  FROM
    balanced_tree.sales
  GROUP BY
    1,
    2
)
SELECT
  member,
  ROUND(AVG(total_revenue), 2) AS avg_rev_by_member
FROM
  cte
GROUP BY
  1
````
**Answer**
| member | avg\_rev\_by\_member |
| ------ | -------------------- |
| false  | 452.01              |
| true   | 454.14              |
# 👔 C. Product Analysis

### 1.  What are the top 3 products by total revenue before discount?
````sql
SELECT
  t2.product_id,
  t2.product_name,
  SUM(t1.price * t1.qty) total_rev
FROM
  balanced_tree.sales t1
  INNER JOIN balanced_tree.product_details t2 ON t1.prod_id = t2.product_id
GROUP BY
  1,
  2
ORDER BY
  3 DESC
LIMIT
  3
````
**Answer**
| product\_id | product\_name                | total\_rev |
| ----------- | ---------------------------- | ---------- |
| 2a2353      | Blue Polo Shirt - Mens       | 217683     |
| 9ec847      | Grey Fashion Jacket - Womens | 209304     |
| 5d267b      | White Tee Shirt - Mens       | 152000     |

❗ **Note**
* In retail industry practice, people tend to manipulate data using `id` instead of `name`, this will be applied to `product`, `segment`, `category`, etc.
### 2.  What is the total quantity, revenue and discount for each segment?
````sql
SELECT
  t2.segment_id,
  t2.segment_name,
  SUM(t1.qty) AS total_qty,
  SUM(t1.price * t1.qty *(1 - t1.discount::NUMERIC / 100)) AS total_rev,
  ROUND(
    SUM(t1.discount * t1.price * t1.qty :: NUMERIC / 100),
    2
  ) AS discount
FROM
  balanced_tree.sales t1
  INNER JOIN balanced_tree.product_details t2 ON t1.prod_id = t2.product_id
GROUP BY
  1,
  2
ORDER BY
  4 DESC
````

**Answer**
| segment\_id | segment\_name | total\_qty | total\_rev | discount |
| ----------- | ------------- | ---------- | ---------- | -------- |
| 5           | Shirt         | 11265      | 356548.73| 49594.27 |
| 4           | Jacket        | 11385      | 322705.54| 44277.46 |
| 6           | Socks         | 11217      | 270963.56| 37013.44 |
| 3           | Jeans         | 11349      | 183006.03| 25343.97 |
 
### 3.  What is the top selling product for each segment?
````sql
WITH cte AS(
  SELECT
    t2.segment_id,
    t2.segment_name,
    t2.product_id,
    t2.product_name,
    SUM(t1.qty) AS total_qty,
    DENSE_RANK() OVER(
      PARTITION BY t2.segment_id
      ORDER BY
        SUM(t1.qty) DESC
    ) AS rank
  FROM
    balanced_tree.sales t1
    INNER JOIN balanced_tree.product_details t2 ON t1.prod_id = t2.product_id
  GROUP BY
    1,
    2,
    3,
    4
)
SELECT
  segment_id,
  segment_name,
  product_id,
  product_name,
  total_qty
FROM
  cte
WHERE
  rank = 1
ORDER BY
  total_qty DESC
````
**Answer**
| segment\_id | segment\_name | product\_id | product\_name                 | total\_qty |
| ----------- | ------------- | ----------- | ----------------------------- | ---------- |
| 4           | Jacket        | 9ec847      | Grey Fashion Jacket - Womens  | 3876       |
| 3           | Jeans         | c4a632      | Navy Oversized Jeans - Womens | 3856       |
| 5           | Shirt         | 2a2353      | Blue Polo Shirt - Mens        | 3819       |
| 6           | Socks         | f084eb      | Navy Solid Socks - Mens       | 3792       |

**Step**
* For this question, I define the "top selling product" as the product having the largest sold quantity
### 4.  What is the total quantity, revenue and discount for each category?
````sql
SELECT
  t2.category_id,
  t2.category_name,
  SUM(t1.qty) AS total_qty,
  ROUND(
    SUM(
      t1.price * t1.qty *(1 - t1.discount :: NUMERIC / 100)
    ),
    2
  ) AS total_rev,
  ROUND(
    SUM(t1.qty * t1.price * t1.discount :: NUMERIC / 100),
    2
  ) AS total_discount
FROM
  balanced_tree.sales t1
  INNER JOIN balanced_tree.product_details t2 ON t1.prod_id = t2.product_id
GROUP BY
  1,
  2
ORDER BY
  1
````
**Answer**
| category\_id | category\_name | total\_qty | total\_rev | total\_discount |
| ------------ | -------------- | ---------- | ---------- | --------------- |
| 1            | Womens         | 22734      | 505711.57  | 69621.43        |
| 2            | Mens           | 22482      | 627512.29  | 86607.71        |

### 5.  What is the top selling product for each category?
````sql
WITH cte AS(
  SELECT
    t2.category_id,
    t2.category_name,
    t2.product_id,
    t2.product_name,
    SUM(t1.qty) AS total_qty,
    DENSE_RANK() OVER(
      PARTITION BY t2.category_id
      ORDER BY
        SUM(t1.qty) DESC
    ) AS rank
  FROM
    balanced_tree.sales t1
    INNER JOIN balanced_tree.product_details t2 ON t1.prod_id = t2.product_id
  GROUP BY
    1,
    2,
    3,
    4
)
SELECT
  category_id,
  category_name,
  product_id,
  product_name,
  total_qty
FROM
  cte
WHERE
  rank = 1
ORDER BY
  total_qty DESC
````
**Answer**
| category\_id | category\_name | product\_id | product\_name                | total\_qty |
| ------------ | -------------- | ----------- | ---------------------------- | ---------- |
| 1            | Womens         | 9ec847      | Grey Fashion Jacket - Womens | 3876       |
| 2            | Mens           | 2a2353      | Blue Polo Shirt - Mens       | 3819

### 6.  What is the percentage split of revenue by product for each segment?
````sql
WITH cte AS (
  SELECT
    t2.segment_id,
    t2.segment_name,
    t2.product_id,
    t2.product_name,
    SUM(
      t1.price * t1.qty *(1 - t1.discount :: NUMERIC / 100)
    ) AS total_rev
  FROM
    balanced_tree.sales t1
    INNER JOIN balanced_tree.product_details t2 ON t1.prod_id = t2.product_id
  GROUP BY
    1,
    2,
    3,
    4
)
SELECT
  segment_id,
  segment_name,
  product_id,
  product_name,
  ROUND(total_rev, 2) AS total_rev,
  ROUND(
    100 *(
      total_rev / SUM(total_rev) OVER(PARTITION BY segment_id)
    ),
    2
  ) AS percentage
FROM
  cte
ORDER BY
  1,
  6 DESC
````
**Answer**
| segment\_id | segment\_name | product\_id | product\_name                    | total\_rev | percentage |
| ----------- | ------------- | ----------- | -------------------------------- | ---------- | ---------- |
| 3           | Jeans         | e83aa3      | Black Straight Jeans - Womens    | 106407.04  | 58.14      |
| 3           | Jeans         | c4a632      | Navy Oversized Jeans - Womens    | 43992.39   | 24.04      |
| 3           | Jeans         | e31d39      | Cream Relaxed Jeans - Womens     | 32606.60   | 17.82      |
| 4           | Jacket        | 9ec847      | Grey Fashion Jacket - Womens     | 183912.12  | 56.99      |
| 4           | Jacket        | d5e9a6      | Khaki Suit Jacket - Womens       | 76052.95   | 23.57      |
| 4           | Jacket        | 72f5d4      | Indigo Rain Jacket - Womens      | 62740.47   | 19.44      |
| 5           | Shirt         | 2a2353      | Blue Polo Shirt - Mens           | 190863.93  | 53.53      |
| 5           | Shirt         | 5d267b      | White Tee Shirt - Mens           | 133622.40  | 37.48      |
| 5           | Shirt         | c8d436      | Teal Button Up Shirt - Mens      | 32062.40   | 8.99       |
| 6           | Socks         | f084eb      | Navy Solid Socks - Mens          | 119861.64  | 44.24      |
| 6           | Socks         | 2feb6b      | Pink Fluro Polkadot Socks - Mens | 96377.73   | 35.57      |
| 6           | Socks         | b9a74d      | White Striped Socks - Mens       | 54724.19   | 20.20      |
**Step**
* As the question ask for percentage of revenue for each product by segment, remember to add `PARTITION BY segment_id`
### 7.  What is the percentage split of revenue by segment for each category?
````sql
WITH cte AS (
  SELECT
    t2.category_id,
    t2.category_name,
    t2.segment_id,
    t2.segment_name,
    SUM(
      t1.price * t1.qty *(1 - t1.discount :: NUMERIC / 100)
    ) AS total_rev
  FROM
    balanced_tree.sales t1
    INNER JOIN balanced_tree.product_details t2 ON t1.prod_id = t2.product_id
  GROUP BY
    1,
    2,
    3,
    4
)
SELECT
  category_id,
  category_name,
  segment_id,
  segment_name,
  ROUND(total_rev, 2) AS total_rev,
  ROUND(
    100 *(
      total_rev / SUM(total_rev) OVER(PARTITION BY category_id)
    ),
    2
  ) AS percentage
FROM
  cte
ORDER BY
  1,
  6 DESC
````
**Answer**
| category\_id | category\_name | segment\_id | segment\_name | total\_rev | percentage |
| ------------ | -------------- | ----------- | ------------- | ---------- | ---------- |
| 1            | Womens         | 4           | Jacket        | 322705.54  | 63.81      |
| 1            | Womens         | 3           | Jeans         | 183006.03  | 36.19      |
| 2            | Mens           | 5           | Shirt         | 356548.73  | 56.82      |
| 2            | Mens           | 6           | Socks         | 270963.56  | 43.18      |

### 8.  What is the percentage split of total revenue by category?
````sql
WITH cte AS(
  SELECT
    t2.category_id,
    t2.category_name,
    ROUND(
      SUM(
        t1.price * t1.qty * (1 - discount :: NUMERIC / 100)
      ),
      2
    ) AS total_rev
  FROM
    balanced_tree.sales t1
    INNER JOIN balanced_tree.product_details t2 ON t1.prod_id = t2.product_id
  GROUP BY
    1,
    2
)
SELECT
  *,
  ROUND(100 * total_rev / SUM(total_rev) OVER(), 2) AS percentage
FROM
  cte
ORDER BY
  category_id
````
**A different approach**
> Use windown function instead of  CTE
````sql
SELECT
  t2.category_id,
  t2.category_name,
  ROUND(
    SUM(
      t1.price * t1.qty * (1 - discount :: NUMERIC / 100)
    ),
    2
  ) AS total_rev,
  ROUND(
    100 * SUM(
      t1.price * t1.qty * (1 - discount :: NUMERIC / 100)
    ) / SUM(
      SUM(
        t1.price * t1.qty * (1 - discount :: NUMERIC / 100)
      )
    ) OVER(),
    2
  ) AS percentage
FROM
  balanced_tree.sales t1
  INNER JOIN balanced_tree.product_details t2 ON t1.prod_id = t2.product_id
GROUP BY
  1,
  2
ORDER BY 1
````

**Answer**
| category\_id | category\_name | total\_rev | percentage |
| ------------ | -------------- | ---------- | ---------- |
| 1            | Womens         | 505711.57  | 44.63      |
| 2            | Mens           | 627512.29  | 55.37      |

❗ **Note**
* Once the `GROUP BY` clause is executed, individual records can not longer be accessed. Thus, another aggregation `SUM` function must be put into the window function `SUM` to calculate the total sum as follows:
	* The inner `SUM` will be executed after the `GROUP BY` clause
	* Then, the outer `SUM` will sum up the results from the inner `SUM`
### 9.  What is the total transaction “penetration” for each product? (hint: penetration = number of transactions where at least 1 quantity of a product was purchased divided by total number of transactions)
````sql
WITH cte_1 AS(
  SELECT
    prod_id,
    COUNT(DISTINCT txn_id) AS frequency
  FROM
    balanced_tree.sales
  GROUP BY
    1
),
cte_2 AS(
  SELECT
    COUNT(DISTINCT txn_id) AS total_txn
  FROM
    balanced_tree.sales
)
SELECT
  t1.prod_id,
  t2.product_name,
  ROUND(100 * frequency :: NUMERIC / total_txn, 2) AS percentage
FROM
  cte_1 t1
  CROSS JOIN cte_2
  INNER JOIN balanced_tree.product_details t2 ON t1.prod_id = t2.product_id
ORDER BY
  3 DESC
````
**Answer**
| prod\_id | product\_name                    | percentage |
| -------- | -------------------------------- | ---------- |
| f084eb   | Navy Solid Socks - Mens          | 51.24      |
| 9ec847   | Grey Fashion Jacket - Womens     | 51.00      |
| c4a632   | Navy Oversized Jeans - Womens    | 50.96      |
| 2a2353   | Blue Polo Shirt - Mens           | 50.72      |
| 5d267b   | White Tee Shirt - Mens           | 50.72      |
| 2feb6b   | Pink Fluro Polkadot Socks - Mens | 50.32      |
| 72f5d4   | Indigo Rain Jacket - Womens      | 50.00      |
| d5e9a6   | Khaki Suit Jacket - Womens       | 49.88      |
| e83aa3   | Black Straight Jeans - Womens    | 49.84      |
| e31d39   | Cream Relaxed Jeans - Womens     | 49.72      |
| b9a74d   | White Striped Socks - Mens       | 49.72      |
| c8d436   | Teal Button Up Shirt - Mens      | 49.68      |

❗ **Note**
* `DISTINCT` is not compatible with window function. Thus, `COUNT(DISTINCT txn_id)` will return an error
* This question is a good illustration of how `CROSS JOIN`can be used to match one record to multiple records. This approach increases readiblity compared to using SUBQUERY to answer this question 
### 10.  What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?

[ *To be updated...*]
# 👌 D. Bonus Challenge

Use a single SQL query to transform the `product_hierarchy` and `product_prices` datasets to the `product_details` table.

[ *To be updated...*]