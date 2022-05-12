# A. Customer Nodes Exploration

### 1.  How many unique nodes are there on the Data Bank system?
````sql
WITH cte AS(
  SELECT
    region_id,
    COUNT(DISTINCT node_id) AS number_nodes
  FROM
    data_bank.customer_nodes
  GROUP BY
    region_id
)
SELECT
  SUM(number_nodes) AS total_nodes
FROM
  cte
````
**Answer**
| total\_nodes |
| ------------ |
| 25           |

**Step**
* As each region have its own nodes, we must count the number of nodes grouped by their regions before summing them up
### 2.  What is the number of nodes per region?
````sql
SELECT
  t2.region_name,
  COUNT(DISTINCT t1.node_id) AS number_nodes
FROM
  data_bank.customer_nodes t1
  INNER JOIN data_bank.regions t2 ON t1.region_id = t2.region_id
GROUP BY
  1
````
**Answer**
| region\_id | number\_nodes |
| ---------- | ------------- |
| Africa| 5             |
| America| 5             |
| Asia| 5             |
| Australia| 5             |
| Europe| 5             |

### 3.  How many customers are allocated to each region?
````sql
SELECT
  t2.region_name,
  COUNT(DISTINCT t1.customer_id) AS number_nodes
FROM
  data_bank.customer_nodes t1
  INNER JOIN data_bank.regions t2 ON t1.region_id = t2.region_id
GROUP BY
  1
````
**Answer**
| region\_name | number\_nodes |
| ------------ | ------------- |
| Africa       | 102           |
| America      | 105           |
| Asia         | 95            |
| Australia    | 110           |
| Europe       | 88            |

### 4.  How many days on average are customers reallocated to a different node?
* Recursive CTE approach

	[ *To be updated...*]

* Excluding "unwanted data" approach
````sql
WITH cte_1 AS(
  SELECT
    customer_id,
    node_id,
    start_date,
    end_date,
    LAG(node_id, 1) OVER(
      PARTITION BY customer_id
      ORDER BY
        start_date
    ) AS previous_node
  FROM
    data_bank.customer_nodes
  WHERE
    DATE_PART('year', end_date) <> 9999
  ORDER BY
    1,
    3
),
cte_2 AS (
  SELECT
    customer_id,
    AVG(end_date - start_date) avg_duration
  FROM
    cte_1
  WHERE
    node_id <> previous_node
  GROUP BY
    1
)
SELECT
  CEILING(AVG(avg_duration))::VARCHAR || ' days' AS avg_duration
FROM
  cte_2
````
**Answer**
| avg\_duration |
| ------------- |
| 15 days       |

**Step**
* First, we exclude records with `end_date` having the year '9999'. However, in practice, there should be a cleaning step to replace such values in our table
* Using window function `LAG` so that we can compare nodes between different records
* As the question ask for the allocation duration to a different node of each customer, we must also exlucde records with the same node as the previous one
* We must calculate the average duration for each customer in prior to calculating the final average value, or else we only calculate the average allocation duration of different nodes
### 5.  What is the median, 80th and 95th percentile for this same reallocation days metric for each region?
[ *To be updated...*]
# B. Customer Transactions

### 1.  What is the unique count and total amount for each transaction type?
````sql
SELECT
  txn_type,
  COUNT(*) AS txn_count,
  SUM(txn_amount) AS total_amount
FROM
  data_bank.customer_transactions
GROUP BY
  1
 ````
 **Answer**
 | txn\_type  | txn\_count | total\_amount |
| ---------- | ---------- | ------------- |
| purchase   | 1617       | 806537        |
| withdrawal | 1580       | 793003        |
| deposit    | 2671       | 1359168       |

### 2.  What is the average total historical deposit counts and amounts for all customers?
````sql
WITH CTE AS(
  SELECT
    customer_id,
    COUNT(*) AS avg_count_per_cus,
    AVG(txn_amount) AS avg_deposit_amount_per_cus
  FROM
    data_bank.customer_transactions
  WHERE
    txn_type = 'deposit'
  GROUP BY
    1
)
SELECT
  ROUND(AVG(avg_count_per_cus)) AS avg_count,
  ROUND(AVG(avg_deposit_amount_per_cus)) AS avg_deposit_amount
FROM
  cte
````
**Answer**
| avg\_count | avg\_deposit\_amount |
| ---------- | -------------------- |
| 5          | 509                  |

❗ **Note**
* Similar to question 4 part A, we must be careful when dealing with "**the  average of the averages**"
	* The result would be entirely different and wrong if we take the average of the total deposit amount
	* In this question, we must first take the average by customer, then calculate the final average value from them, as the question asks for "the average of all customers"
### 3.  For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?
````sql
WITH cte AS (
  SELECT
    customer_id,
    DATE_PART('MONTH', txn_date) AS month,
    SUM(
      CASE
        WHEN txn_type = 'deposit' THEN 1
        ELSE 0
      END
    ) AS deposit,
    SUM(
      CASE
        WHEN txn_type = 'purchase' THEN 1
        ELSE 0
      END
    ) AS purchase,
    SUM(
      CASE
        WHEN txn_type = 'withdrawal' THEN 1
        ELSE 0
      END
    ) AS withdrawal
  FROM
    data_bank.customer_transactions
  GROUP BY
    1,
    2
)
SELECT
  month,
  COUNT(customer_id) AS customer_count
FROM
  cte
WHERE
  deposit > 1
  AND (
    purchase >= 1
    OR withdrawal >= 1
  )
GROUP BY
  1
ORDER BY
  1
````
**Answer**
| month | customer\_count |
| ----- | --------------- |
| 1     | 168             |
| 2     | 181             |
| 3     | 192             |
| 4     | 70              |

### 4.  What is the closing balance for each customer at the end of the month? Also show the change in balance each month in the same table output.
### 5.  Comparing the closing balance of a customer’s first month and the closing balance from their second nth, what percentage of customers:

-   Have a negative first month balance?
-   Have a positive first month balance?
-   Increase their opening month’s positive closing balance by more than 5% in the following month?
-   Reduce their opening month’s positive closing balance by more than 5% in the following month?
-   Move from a positive balance in the first month to a negative balance in the second month?