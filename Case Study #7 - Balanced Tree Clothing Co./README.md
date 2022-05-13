# 🍕 Case Study #7: Balanced Tree Co.
<p align="center">
<img width="400px"  src="https://8weeksqlchallenge.com/images/case-study-designs/7.png" />
</p>

<p align="center">
View the case study <a href="https://8weeksqlchallenge.com/case-study-7/">here</a> and my <b>solution</b> <a href="https://github.com/nguyennhatquan/8-Week-SQL-Challenge/blob/main/Case%20Study%20%237%20-%20Balanced%20Tree%20Clothing%20Co./Answers.md">here</a>
</p>

## **Key learning points**
* Differentiate between
    > `NTILE()`
    > 
    >Divide an ordered data set into a number of buckets indicated by expr and assigns the appropriate bucket number to each row

    >`PERCENTILE_DISC()`
    >
    >Return the percentile value as a number from the distribution

    >`PERCENTILE_CONT()`
    >
    > Not guarantee to return one of the value in the distribution. Instead, this functions interpolates the percentile. This might not be accepted in business practice, so use PERCENTILE_DISC() instead in such cases