# 🛒 Case Study #5: Data Mart
<p align="center">
<img width="400px"  src="https://8weeksqlchallenge.com/images/case-study-designs/5.png" />
</p>

<p align="center">
View the case study <a href="https://8weeksqlchallenge.com/case-study-5/">here</a> and my <b>solution</b> <a href="https://github.com/nguyennhatquan/8-Week-SQL-Challenge/blob/main/Case%20Study%20%235%20-%20Data%20Mart/Answers.md">here</a>
</p>

## **Key learning points**
* Perform Data Cleaning:
    * Use `TO_DATE` to convert string to `DATE` data type based on the specified format
    * Extract components from `DATE` data type using `DATE_PART`
    * Mapping new columns using `CASE WHEN` and `LEFT`, `RIGHT`
* Use `GENERATE SERIES` to create new values based on specified range
* Practice dealing with "the average of the averages" problem
* Perform Before & After Data Analysis:
    * Use `CASE WHEN` to create new column with 'Before' or 'After' tag
    * Use **Analytic Function** `LAG` to calculate rate of growth/reduction + percent of change
	    * Rate = new value - original value / original value
	    * Percent of change = Rate * 100
* Although it is common knowledge that `GROUP BY` clause is executed before `SELECT` clause, it appears that we can use the following elements regarding `CASE WHEN` in `GROUP BY` clause:
    * The whole `CASE WHEN` clause
    * The column alias or column number (1, 2, etc.) of `CASE WHEN` column

