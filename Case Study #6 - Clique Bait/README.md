# 🎣 Case Study #6: Clique Bait
<p align="center">
<img width="400px"  src="https://8weeksqlchallenge.com/images/case-study-designs/6.png" />
</p>

<p align="center">
View the case study <a href="https://8weeksqlchallenge.com/case-study-6/">here</a> and my <b>solution</b> <a href="https://github.com/nguyennhatquan/8-Week-SQL-Challenge/blob/main/Case%20Study%20%236%20-%20Clique%20Bait/Answers.md">here</a>
</p>

## **Key learning points**
* Deal with marketing-related data, i.e data about users online behaviors, events and marketing campaigns
    * [Suggested resource to understand "cookies"](https://www.youtube.com/watch?v=HFyaW50GFOs) 🍪 
* Use `ORDER BY` within `STRING_AGG` to concatenate strings in the desired order
    * E.g, `STRING_AGG(column_1,',' ORDER BY column 2)`
* Use `MAX` with `CASE WHEN` to flag a column
* Thinking in **Funnel** to analyse customer behavior towards products
    * A product can be viewEd, then added to cart, and lastly purchased, all of which form a funnel
    * The further to the end of this funnel, the fewer product views are added to cart, and the fewer products in cart are actually purchased