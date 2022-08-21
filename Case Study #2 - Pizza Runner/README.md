# 🍕 Case Study #2: Pizza Runner
<p align="center">
<img width="400px"  src="https://8weeksqlchallenge.com/images/case-study-designs/2.png" />
</p>

<p align="center">
View the case study <a href="https://8weeksqlchallenge.com/case-study-2/">here</a> and my <b>solution</b> <a href="https://github.com/nguyennhatquan/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/Answers.md">here</a>
</p>

## **Key learning points**
* Be cautious of the difference between **`NULL`**, **'null'** and whitespace **''** when filtering table
* Use `WHERE EXISTS` to perform **LEFT SEMI JOIN** instead of `LEFT JOIN` for efficiency purpose as the former method doesn't return the whole table
* Learn to detect signs indicating possible application of `SUM CASE WHEN` for a more readable result:
    * When we must use `COUNT` together with `GROUP BY` (Question 5)
    * When we must create new columns using `CASE WHEN` and return a sum number (Question 7, Question 8)
* Learn the difference between `DATE_TRUNC` and `DATE_PART` 
    * `DATE_TRUNC` returns a `TIMESTAMP` based off the field input, which means 11:00:00 on April 11th is different from 11:00:00 April 12th
    * `DATE_PART` returns an `INTEGER`
* Use `TO_CHAR` to convert a `timestamp`, `interval`, ... to string under the desired format
* Use a trick with DATE_TRUNC to custom the start of the week
    * E.g., shifting the starting day of week 2 days forwards by using `DATE_TRUNC('week', date_column - INTERVAL '2 DAY') + INTERVAL '2 DAY'`
* Calculate differences between `TIMESTAMP` values using `EXTRACT(EPOCH FROM column)`
    * Note that `EXTRACT(EPOCH FROM column_1 :: TIMESTAMP) - EXTRACT(EPOCH FROM column_2)` will return the difference of timestamp in seconds, so we must divide the result by 60 to get result in minutes and so on
* Use `UNNEST(REGEXP_MATCH())` to only return records matching our regular expression
    * An alternative to `REGEXP_MATCH` is `SUBTRING`. The only difference between those two is that while the former only return records with matching **regexp**, the latter return all records with `NULL` values for those not matching the specified *regexp*
* Learn to use `REGEXP_SPLIT_TO_TABLE` to split strings into new records and then `STRING_AGG` along with `GROUP BY` to concatenate strings from different records into one
    * When using the `REGEXP_SPLIT_TO_TABLE` ,only records where there aren't  `NULL`  are returned
* Learn that `EXCEPT` has a default `DISTINCT` built in so it’s doing the same deduplication as the `UNION`
* Learn to use the following functions together
    * Use `REGEXP_SPLIT_TO_ARRAY` to return arrays of splitted strings in accordance with the specified **regex**
    * Use `CARDINALITY` to return the length of an array
    * Use `COALESCE` to return the first non-NULL value among the selected columns/values
* Learn to use `SETSEED` to create a reproducible result when using `RANDOM` to return a number that is >=0 and <1

