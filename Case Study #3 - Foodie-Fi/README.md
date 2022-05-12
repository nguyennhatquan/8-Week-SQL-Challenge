# 🍕 Case Study #3: Foodie Fi
<p align="center">
<img width="400px"  src="https://8weeksqlchallenge.com/images/case-study-designs/3.png" />
</p>

<p align="center">
View the case study <a href="https://8weeksqlchallenge.com/case-study-3/">here</a> and my <b>solution</b> <a href="https://github.com/nguyennhatquan/8-Week-SQL-Challenge/blob/main/Case%20Study%20%233%20-%20Foodie-Fi/Answers.md">here</a>
</p>

## **Key learning points**
* When dealing with `DATE` data type, to calculate the difference between them, there are 2 possible approach:
	*  `DATE_1 - DATE_`2 will return the difference of days in `INTEGER`
	*  `DATE_PART('day',AGE(DATE_1 - DATE_2))`
        * `AGE` will return an `interval` data type
        * `DATE PART` will extract our desired part from this `interval`