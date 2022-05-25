# A. Data Exploration and Cleansing

### 1.  Update the `fresh_segments.interest_metrics` table by modifying the `month_year` column to be a date data type with the start of the month
````sql
UPDATE fresh_segments.interest_metrics
SET month_year = TO_DATE(month_year, 'MM-YYYY');

ALTER TABLE fresh_segments.interest_metrics
ALTER month_year TYPE DATE;
````
❗**Note**
* Use `UPDATE` and `ALTER` to solve this DML (Data Manipulation Language) question

### 2.  What is count of records in the `fresh_segments.interest_metrics` for each `month_year` value sorted in chronological order (earliest to latest) with the null values appearing first?
````sql
SELECT
  month_year,
  COUNT(*)
FROM
  fresh_segments.interest_metrics
GROUP BY
  1
ORDER BY
  1 NULLS FIRST
````
**Answer**
| month\_year | count |
| ----------- | ----- |
| null        | 1194  |
| 2018-07-01  | 729   |
| 2018-08-01  | 767   |
| 2018-09-01  | 780   |
| 2018-10-01  | 857   |
| 2018-11-01  | 928   |
| 2018-12-01  | 995   |
| 2019-01-01  | 973   |
| 2019-02-01  | 1121  |
| 2019-03-01  | 1136  |
| 2019-04-01  | 1099  |
| 2019-05-01  | 857   |
| 2019-06-01  | 824   |
| 2019-07-01  | 864   |
| 2019-08-01  | 1149  |

### 3.  What do you think we should do with these null values in the `fresh_segments.interest_metrics`
❗**Note**
* Let's review some methods to deal with missing values 
	* Remove them
	* Infer them from available data points
	* Replace them with mean, mode or median of the columns
---
Thus, the most suitable approach in our case is to remove those `NULL` values as we're unable to speficy which date those records are assigned to and they won't be useful for us

🧠 **Best Practice**
* In reality, remember to check for the percentage of `NULL` values we're about to remove. If the value is unacceptably high, the best practice is to check the data source, or discuss with your stakeholder
---
````sql
DELETE FROM fresh_segments.interest_metrics WHERE month_year IS NULL;
-- Let's check again for NULL values
SELECT
  *
from
  fresh_segments.interest_metrics
WHERE
  month_year IS NULL
````
### 4.  How many `interest_id` values exist in the `fresh_segments.interest_metrics` table but not in the `fresh_segments.interest_map` table? What about the other way around?
* `ANTI JOIN` approach
	````sql
	SELECT
	  (
	    SELECT
	      COUNT(interest_id)
	    FROM
	      fresh_segments.interest_metrics
	    WHERE
	      NOT EXISTS(
	        SELECT
	          1
	        FROM
	          fresh_segments.interest_map
	        WHERE
	          fresh_segments.interest_metrics.interest_id = fresh_segments.interest_map.id
	      )
	  ) AS not_in_maps,
	  (
	    SELECT
	      COUNT(id)
	    FROM
	      fresh_segments.interest_map
	    WHERE
	      NOT EXISTS(
	        SELECT
	          1
	        FROM
	          fresh_segments.interest_metrics
	        WHERE
	          fresh_segments.interest_metrics.interest_id = fresh_segments.interest_map.id
	      )
	  ) AS not_in_metrics
	````
	**Answer**
	| not\_in\_maps | not\_in\_metrics |
	| ------------- | ---------------- |
	| 0             | 7                |
* `SUM CASE` and `FULL OUTER JOIN` approach
	````sql
	SELECT
	  COUNT(DISTINCT id) AS unique_id_map,
	  COUNT(DISTINCT interest_id) AS unique_id_metrics,
	  SUM(
	    CASE
	      WHEN id IS NULL THEN 1
	      ELSE 0
	    END
	  ) AS not_in_map,
	  SUM(
	    CASE
	      WHEN interest_id IS NULL THEN 1
	      ELSE 0
	    END
	  ) AS not_in_metric
	FROM
	  fresh_segments.interest_map 
	  FULL OUTER JOIN fresh_segments.interest_metrics ON id = interest_id
	````

	**Answer**
	| unique\_id\_map | unique\_id\_metrics | not\_in\_map | not\_in\_metric |
	| --------------- | ------------------- | ------------ | --------------- |
	| 1209            | 1202                | 0            | 7               |
	
### 5.  Summarise the `id` values in the `fresh_segments.interest_map` by its total record count in this table
````sql
WITH cte AS(
  SELECT
    id,
    COUNT(*) AS id_frequency
  FROM
    fresh_segments.interest_map
  GROUP BY
    1
)
SELECT
  id_frequency,
  COUNT(*)
FROM
  cte
GROUP BY
  1
````
**Answer**
| id\_frequency | count |
| ------------- | ----- |
| 1             | 1209  |

🧠**Best Practice**
* This is a useful pratice to  both check for # of duplicates (if there's any) and # of unique keys
### 6.  What sort of table join should we perform for our analysis and why? Check your logic by checking the rows where `interest_id = 21246` in your joined output and include all columns from `fresh_segments.interest_metrics` and all columns from `fresh_segments.interest_map` except from the `id` column.
Let's review what we've known about our tables so far
* All values of `interest_id` in `interest_metrics` are also in `interest_map` (Question 4)
* All `id` values in `interest_map` are unique (Question 5)

Thus, to join the 2 table, we can do either way
* Use `LEFT JOIN` or `INNER JOIN` with the base table being `interest_metrics`
* Use `INNER JOIN` with the base table being `interest_map`

````sql
SELECT
  t1.*,
  interest_name,
  interest_summary,
  created_at,
  last_modified
FROM
  fresh_segments.interest_metrics t1
  LEFT JOIN fresh_segments.interest_map t2 ON interest_id = id
WHERE
  interest_id = 21246
````
**Answer**
| \_month | \_year | month\_year              | interest\_id | composition | index\_value | ranking | percentile\_ranking | interest\_name                   | interest\_summary                                     | created\_at              | last\_modified           |
| ------- | ------ | ------------------------ | ------------ | ----------- | ------------ | ------- | ------------------- | -------------------------------- | ----------------------------------------------------- | ------------------------ | ------------------------ |
| 7       | 2018   | 2018-07-01T00:00:00.000Z | 21246        | 2.26        | 0.65         | 722     | 0.96                | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 8       | 2018   | 2018-08-01T00:00:00.000Z | 21246        | 2.13        | 0.59         | 765     | 0.26                | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 9       | 2018   | 2018-09-01T00:00:00.000Z | 21246        | 2.06        | 0.61         | 774     | 0.77                | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 10      | 2018   | 2018-10-01T00:00:00.000Z | 21246        | 1.74        | 0.58         | 855     | 0.23                | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 11      | 2018   | 2018-11-01T00:00:00.000Z | 21246        | 2.25        | 0.78         | 908     | 2.16                | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 12      | 2018   | 2018-12-01T00:00:00.000Z | 21246        | 1.97        | 0.7          | 983     | 1.21                | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 1       | 2019   | 2019-01-01T00:00:00.000Z | 21246        | 2.05        | 0.76         | 954     | 1.95                | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 2       | 2019   | 2019-02-01T00:00:00.000Z | 21246        | 1.84        | 0.68         | 1109    | 1.07                | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 3       | 2019   | 2019-03-01T00:00:00.000Z | 21246        | 1.75        | 0.67         | 1123    | 1.14                | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 4       | 2019   | 2019-04-01T00:00:00.000Z | 21246        | 1.58        | 0.63         | 1092    | 0.64                | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |

### 7.  Are there any records in your joined table where the `month_year` value is before the `created_at` value from the `fresh_segments.interest_map` table? Do you think these values are valid and why?
````sql
WITH cte AS(
  SELECT
    t1.*,
    interest_name,
    interest_summary,
    created_at,
    last_modified
  FROM
    fresh_segments.interest_metrics t1
    LEFT JOIN fresh_segments.interest_map t2 ON interest_id = id
)
SELECT
  COUNT(*)
FROM
  cte
WHERE
  month_year < created_at
````
|count  |  
|--|
|188  | 

Remember, our `month_year` column was created from question 1 by using the initial values of month and year. Thus, as long as the extracted month of `created_at` is equal or larger than that of `month_year`, our data is valid 
````sql

````

# B. Interest Analysis

### 1.  Which interests have been present in all `month_year` dates in our dataset?
````sql
WITH cte AS(
  SELECT
    interest_id
  FROM
    fresh_segments.interest_metrics
  GROUP BY
    1
  HAVING
    COUNT(DISTINCT month_year) = (
      SELECT
        COUNT(DISTINCT month_year)
      FROM
        fresh_segments.interest_metrics
    )
  ORDER BY
    1
)
SELECT
  (
    SELECT
      COUNT(DISTINCT interest_id)
    FROM
      fresh_segments.interest_metrics
  ) AS unique_id,(
    SELECT
      COUNT(*)
    FROM
      cte
  ) AS id_presented_all_months
````
**Answer**
| unique\_id | id\_presented\_all\_months |
| ---------- | -------------------------- |
| 1202       | 480                        |
To see the detailed `interest_id`, we only need to run the query inside the CTE above
### 2.  Using this same `total_months` measure - calculate the cumulative percentage of all records starting at 14 months - which `total_months` value passes the 90% cumulative percentage value?
As we approach the first question in a differently compared to that of Danny Ma, let's first create the `total_months` measure as follows 
````sql
WITH cte AS(
  SELECT
    interest_id,
    COUNT(DISTINCT month_year) total_months
  FROM
    fresh_segments.interest_metrics
  GROUP BY
    1
)
SELECT
  total_months,
  COUNT(*) AS id_counts
FROM
  cte
GROUP BY
  1
ORDER BY
  1 DESC
````
| total_months | id\_counts |
| ---------------- | ---------- |
| 14               | 480        |
| 13               | 82         |
| 12               | 65         |
| 11               | 94         |
| 10               | 86         |
| 9                | 95         |
| 8                | 67         |
| 7                | 90         |
| 6                | 33         |
| 5                | 38         |
| 4                | 32         |
| 3                | 15         |
| 2                | 12         |
| 1                | 13         |
____
We solve the question as follow
````sql
WITH cte AS(
  SELECT
    interest_id,
    COUNT(DISTINCT month_year) total_months
  FROM
    fresh_segments.interest_metrics
  GROUP BY
    1
)
SELECT
  total_months,
  COUNT(*) AS id_counts,
  ROUND(
    100 * SUM(COUNT(*)) OVER(
      ORDER BY
        total_months DESC
    ) / SUM(COUNT(*)) OVER(),
    2
  ) AS cum_percentage
FROM
  cte
GROUP BY
  1
ORDER BY
  1 DESC
````
**Answer**
| total\_months | id\_counts | cum\_percentage |
| ------------- | ---------- | --------------- |
| 14            | 480        | 39.93           |
| 13            | 82         | 46.76           |
| 12            | 65         | 52.16           |
| 11            | 94         | 59.98           |
| 10            | 86         | 67.14           |
| 9             | 95         | 75.04           |
| 8             | 67         | 80.62           |
| 7             | 90         | 88.10           |
| 6             | 33         | 90.85           |
| 5             | 38         | 94.01           |
| 4             | 32         | 96.67           |
| 3             | 15         | 97.92           |
| 2             | 12         | 98.92           |
| 1             | 13         | 100.00          |

Thus, `total_months` with values equal 6 has the cumulative percentage passing 90%
❗**Note**
* To calculate the cumulative values, apply window function `SUM OVER` with `ORDER BY` to use the default **window frame**, which calculates the sum of values starting from the first record to the current one
### 3.  If we were to remove all `interest_id` values which are lower than the `total_months` value we found in the previous question - how many total data points would we be removing?
````sql
WITH cte AS(
  SELECT
    interest_id,
    COUNT(DISTINCT month_year) total_months
  FROM
    fresh_segments.interest_metrics
  GROUP BY
    1
  HAVING
    COUNT(DISTINCT month_year) < 6
)
SELECT
  COUNT(*)
FROM
  fresh_segments.interest_metrics
WHERE
  EXISTS (
    SELECT
      1
    FROM
      cte
    WHERE
      cte.interest_id = fresh_segments.interest_metrics.interest_id
  )
````
**Answer**
|removed_rows|  
|--|
|400  |  

### 4.  Does this decision make sense to remove these data points from a business perspective? Use an example where there are all 14 months present to a removed `interest` example for your arguments - think about what it means to have less months present from a segment perspective. 
[*To be updated*]

Let's assume that Danny Ma's team is trying to identify `interest_id` that are worth the most to their clients during the 14 months of their dataset. Would they be:
1. `interest_id` having high frequencies     ❌
	> This wouldn't be enough as we haven't  account for the case when an `interest_id` only appear multiple times in a small amount of months and might not provide promising insight to Danny Ma's clients 
2. `interest_id` having high frequencies  of months that they appear in ✅
	> This would be more appropriate as we can identify `interest_id` that seem to be trendy over a significant amount of time 
### 5. If we include all of our interests regardless of their counts - how many unique interests are there for each month?
[*To be updated*]
# C. Segment Analysis

### 1.  Using the complete dataset - which are the top 10 and bottom 10 interests which have the largest composition values in any `month_year`? Only use the maximum composition value for each interest but you must keep the corresponding `month_year`
````sql
WITH cte AS(
  SELECT
    interest_id,
    MAX(composition) AS max_composition,
    ROUND(
      100 * PERCENT_RANK() OVER (
        ORDER BY
          MAX(composition) DESC
      ) :: NUMERIC,
      2
    ) AS ranking
  FROM
    fresh_segments.interest_metrics
  GROUP BY
    1
),
cte_all AS(
  SELECT
    month_year,
    interest_name,
    composition,
    cte.ranking
  FROM
    fresh_segments.interest_metrics t1
    LEFT JOIN fresh_segments.interest_map t2 ON t1.interest_id = t2.id
    LEFT JOIN cte ON cte.interest_id = t1.interest_id
  WHERE
    EXISTS(
      SELECT
        1
      FROM
        cte
      WHERE
        cte.interest_id = t1.interest_id
        AND cte.max_composition = t1.composition
    )
    AND cte.ranking <= 10
    OR cte.ranking >= 90
  ORDER BY
    3 DESC
) (
  SELECT
    *
  FROM
    cte_all
  ORDER BY
    composition DESC
  LIMIT
    10
)
UNION ALL
  (
    SELECT
      *
    FROM
      cte_all
    ORDER BY
      composition
    LIMIT
      10
  )
````
**Answer**
Here are the first 10 values of each group 
| month\_year | interest\_name                           | composition | ranking |
| ----------- | ---------------------------------------- | ----------- | ------- |
| 2018-12-01  | Work Comes First Travelers               | 21.2        | 0.00    |
| 2018-07-01  | Gym Equipment Owners                     | 18.82       | 0.08    |
| 2018-07-01  | Furniture Shoppers                       | 17.44       | 0.17    |
| 2018-07-01  | Luxury Retail Shoppers                   | 17.19       | 0.25    |
| 2018-10-01  | Luxury Boutique Hotel Researchers        | 15.15       | 0.33    |
| 2018-12-01  | Luxury Bedding Shoppers                  | 15.05       | 0.42    |
| 2018-07-01  | Shoe Shoppers                            | 14.91       | 0.50    |
| 2018-07-01  | Cosmetics and Beauty Shoppers            | 14.23       | 0.58    |
| 2018-07-01  | Luxury Hotel Guests                      | 14.1        | 0.67    |
| 2018-07-01  | Luxury Retail Researchers                | 13.97       | 0.75    |
| 2019-03-01  | World of Warcraft Enthusiasts            | 1.52        | 99.50   |
| 2018-08-01  | Readers of Jamaican Content              | 1.52        | 99.25   |
| 2019-04-01  | Minnesota Vikings Fans                   | 1.52        | 96.59   |
| 2018-10-01  | Scifi Movie and TV Enthusiasts           | 1.53        | 99.83   |
| 2019-04-01  | Super Mario Bros Fans                    | 1.54        | 97.17   |
| 2019-06-01  | Retired Government Employees             | 1.55        | 93.42   |
| 2019-05-01  | Retired Government Employees             | 1.55        | 93.42   |
| 2019-04-01  | Hair Color Shoppers                      | 1.55        | 98.83   |
| 2019-05-01  | Retirement Financial Account Researchers | 1.56        | 94.25   |
| 2018-11-01  | Xbox Enthusiasts                         | 1.56        | 98.00   |

❗**Note**
* Use  window function`PERCENT_RANK` instead of `NTILE` so that equal values of `composition` get equal ranks

**Step**
* Use `LEFT SEMI JOIN` to join `max_composition` of each `interest_id` with its corresponding `month_year`  
### 2.  Which 5 interests had the lowest average `ranking` value?
````sql
SELECT
  interest_name,
  ROUND(AVG(ranking), 2) AS avg_ranking
FROM
  fresh_segments.interest_metrics t1
  INNER JOIN fresh_segments.interest_map t2 ON t1.interest_id = t2.id
GROUP BY
  1
ORDER BY
  2
LIMIT
  5
````
**Answer**
| interest\_name                 | avg\_ranking |
| ------------------------------ | ------------ |
| Winter Apparel Shoppers        | 1.00         |
| Fitness Activity Tracker Users | 4.11         |
| Men's Shoe Shoppers            | 5.93         |
| Elite Cycling Gear Shoppers    | 7.80         |
| Shoe Shoppers                  | 9.36         |

### 3.  Which 5 interests had the largest standard deviation in their `percentile_ranking` value?
### 4.  For the 5 interests found in the previous question - what was minimum and maximum `percentile_ranking` values for each interest and its corresponding `year_month` value? Can you describe what is happening for these 5 interests?
### 5.  How would you describe our customers in this segment based off their composition and ranking values? What sort of products or services should we show to these customers and what should we avoid?

# D. Index Analysis

The `index_value` is a measure which can be used to reverse calculate the average composition for Fresh Segments’ clients.

Average composition can be calculated by dividing the `composition` column by the `index_value` column rounded to 2 decimal places.

### 1.  What is the top 10 interests by the average composition for each month?
### 2.  For all of these top 10 interests - which interest appears the most often?
### 3.  What is the average of the average composition for the top 10 interests for each month?
### 4.  What is the 3 month rolling average of the max average composition value from September 2018 to August 2019 and include the previous top ranking interests in the same output shown below.
### 5.  Provide a possible reason why the max average composition might change from month to month? Could it signal something is not quite right with the overall business model for Fresh Segments?