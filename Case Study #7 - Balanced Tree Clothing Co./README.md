# 👚 Case Study #7: Balanced Tree Co.
<p align="center">
<img width="400px"  src="https://8weeksqlchallenge.com/images/case-study-designs/7.png" />
</p>

<p align="center">
View the case study <a href="https://8weeksqlchallenge.com/case-study-7/">here</a> and my <b>solution</b> <a href="https://github.com/nguyennhatquan/8-Week-SQL-Challenge/blob/main/Case%20Study%20%237%20-%20Balanced%20Tree%20Clothing%20Co./Answers.md">here</a>
</p>

## **Key learning points**
* Be cautious of **integer floor division** problem
* Learn to always include `id` field of `products` or `categories` in tables, as this is a commnon pracitce in retail industry
* Differentiate between
    > `NTILE()`
    >
    >Divide an ordered data set into a number of buckets indicated by expr and assigns the appropriate bucket number to each row

    >`PERCENTILE_DISC()`
    >
    >Return the percentile value as a number from the distribution

    >`PERCENTILE_CONT()`
    >
    > Not guarantee to return one of the value in the distribution. Instead, this functions interpolates the percentile. This might not be accepted in business practice, so use `PERCENTILE_DISC()` instead in such cases
* Be cautious when using **Aggregation Function** along with **Window Function**:
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

    * In the query above, once the `GROUP BY` clause is executed, individual records can not longer be accessed. Thus, another aggregation `SUM` function must be put into the window function `SUM` to calculate the total sum as follows:
        * The inner `SUM` will be executed after the `GROUP BY` clause
        * Then, the outer `SUM` will sum up the results from the inner `SUM`
* Learn that `DISTINCT` is not compatible with window function. Thus, `COUNT(DISTINCT txn_id)` will return an error