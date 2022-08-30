# Danny's Diner - Danny Ma SQL challenge (Case Study #1)

![](Output%20Images/Sticker%20-%20Danny's%20Diner.png)

This challenge is a proof of my SQL skills. 
----

# Introduction
Danny wants to use this data to answer a few simple questions about his customers. 

He plans on using answers from these questions to help him decide whether he should expand the existing customer loyalty program - additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

Danny has provided you with a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough for you to write fully functioning SQL queries to help him answer his questions!

Below are the key datasets for this study:

* [Sales](https://github.com/Nancy9ice/Danny-s-Diner/blob/main/Datasets/Sales.csv)
* [Menu](https://github.com/Nancy9ice/Danny-s-Diner/blob/main/Datasets/Menu.csv)
* [Members](https://github.com/Nancy9ice/Danny-s-Diner/blob/main/Datasets/Members.csv)

Below is an Entity Relationship Diagram that shows the relationship between the tables.

![](Entity%20Relationship%20Diagram/Entity%20Relationship%20Diagram%20(Danny's%20Diner).PNG)

I'll be using SQL to answer the following questions

# 1. What is the total amount each customer spent at the restaurant?

I used the SUM function to add the amount spent by the customers. To divide these prices into the different customers that spent the money, I used the GROUP BY function. 

    SELECT customer_id, 
           SUM(price) AS money_spent
    FROM sales AS s
    JOIN menu AS m
    ON m.product_id = s.product_id
    GROUP BY customer_id

Customers A, B, and C spent $76, $74, and $36 respectively. 

![](Output%20Images/Answer%20(Number%20One%20-%20Danny's%20Diner).PNG)

# 2. How many days has each customer visited the restaurant?
Some of the dates in the order_date column had duplicates so I used the DISTINCT function to pick count only the unique dates. 

Then I divided it into the different customers using the GROUP BY function. 

    SELECT customer_id, 
           COUNT(DISTINCT order_date) AS days_visited
    FROM sales
    GROUP BY customer_id

Customers A, B, and C visited the restaurant for 4, 6, and 2 days respectively. 

![](Output%20Images/Answer%20(Number%20Two%20-%20Danny's%20Diner).PNG)

# 3. What was the first item from the menu purchased by each customer?
The product names was in the Menu table and the customer details and their purchases were in the sales table. I used the JOIN function to merge both tables using the product_id column as the join point. 

Since the question needed answers to the first item purchased by each customer, I ordered the date from the earliest to the oldest. 

    SELECT customer_id, 
        order_date, 
        product_name 
    FROM sales AS S
    JOIN menu AS m
    ON s.product_id = m.product_id
    ORDER BY order_date

Customers A and B both purchased ramen first while customer C purchased curry.

I would have used the LIMIT function to cut it to the three customers but there would be a change in the output probably because Customer A and C ordered for two different foods in the same day. 

This output however is preferable because this is the order in the menu table.

![](Output%20Images/Answer%20(Number%20Three%20-%20Danny's%20Diner).PNG)

# 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

I used the COUNT function to count each products purchased. The ORDER function was used to arrange the product count from the highest to lowest.

Then because Danny needs just the most purchased item, I used the LIMIT function to cut it down to the top 1. 

    SELECT  product_name,
            COUNT(product_name) AS purchase_count
    FROM sales AS s
    JOIN menu AS m
    ON m.product_id = s.product_id
    GROUP BY product_name
    ORDER BY purchase_count DESC
    LIMIT 1

The top product sold is ramen and it was sold 8 times. 

![](Output%20Images/Answer%20(Number%20Four%20-%20Danny's%20Diner).PNG)

# 5. Which item was the most popular for each customer?

As usual, the COUNT function was used to count the products sold. To find out the product that was most popular by customer, I used the GROUP BY function to segment the customers and product. 

The ORDER BY function was used for both the purchase_count and customers in descending order to make sure that each customer was present in the output after the LIMIT function was used. 

    SELECT  customer_id,
            product_name,
            COUNT(product_name) AS purchase_count
    FROM sales AS s
    JOIN menu AS m
    ON m.product_id = s.product_id
    GROUP BY customer_id, product_name
    ORDER BY purchase_count DESC, customer_id DESC
    LIMIT 3

Most popular products for the three customers were ramen for both customers A and C. Sushi was the most popular product for Customer B. 

![](Output%20Images/Answer%20(Number%20five%20-%20Danny's%20Diner).PNG)

# 6. Which item was purchased first by the customer after they became a member?

To know customers that were members I used the JOIN function to merge the three tables. I merged the sales and menu tables using the product_id key but I did something different for the members table. 

Because we were looking for the products purchased after customers turned members, I JOINed the members table by two conditions where the order_date will be after the join_date using the greater than operator. 

Then I also joined only for customers that were in the members table. This will eliminate any customer that wasn't a member. 

To get the first item purchased after joining, I ordered the order_date by earliest to oldest. 

    SELECT s.customer_id,
        product_name,
        order_date
    FROM sales AS s
    JOIN menu AS m
    ON s.product_id = m.product_id
    JOIN members AS mem
    ON s.order_date > mem.join_date 
        AND s.customer_id = mem.customer_id
    ORDER BY order_date, s.customer_id DESC

Customer A and B bought ramen and sushi respectively after they became members. 

I showed you all output because I want you to notice that customer C is not in it. He never became a member so using the JOIN function eliminated him from the ouput. 

![](Output%20Images/Answer%20(Number%20six%20-%20Danny's%20Diner).PNG)

# 7. Which item was purchased just before the customer became a member?

Again, I used the JOIN operator to merge the three tables but I joined the members table using a different operator because I was looking for customers' purchases before they became members. 

The ORDER BY function was used in a descending format because the date needed was purchase date **just** before membership.

    SELECT s.customer_id,
        product_name,
        order_date
    FROM sales AS s
    JOIN menu AS m
    ON s.product_id = m.product_id
    JOIN members AS mem
    ON s.order_date <= mem.join_date 
        AND s.customer_id = mem.customer_id
    ORDER BY order_date DESC, s.customer_id DESC

Customer A and B purchases sushi and curry just before they became members. 

![](Output%20Images/Answer%20(Number%20Seven%20-%20Danny's%20Diner).PNG)

# 8. What is the total items and amount spent for each member before they became a member?

This question is just a little bit different from question 7. I used the SUM function to sum up the total amount they spent on the products and also counted the products they bought with the COUNT function before the turned members.

    SELECT s.customer_id,
        COUNT(product_name) AS total_items,
        SUM(price) AS total_amount
    FROM sales AS s
    JOIN menu AS m
    ON s.product_id = m.product_id
    JOIN members AS mem
    ON s.order_date <= mem.join_date 
        AND s.customer_id = mem.customer_id
    GROUP BY s.customer_id
    ORDER BY s.customer_id DESC

Customer A and B spent the same amount and bought same number of items before they turned members. Remember Customer C never turned a member so it will be invalid for them to be included in the ouput.

![](Output%20Images/Answer%20(Number%20Eight%20-%20Danny's%20Diner).PNG)

# 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

The solution to this was a bit complicated so I used sub queries and CASE WHEN functions. 

I needed to create another column that bore the points for these customers. So a column called, "points" was derived using certain conditions suitable to answer the question. 

First condition stated that if a product was curry or ramen, 10 should be inputted in the points column. The second condition was anything other than the first condition should have 20 points. 

I did this because I was already aware that there were only three foods.

I made a subquery with this while selecting the customers and the count of the products and price. 

Then I made the main query selecting only the customers and the total points for each customers while also ordering these customers alphabetically.

    SELECT customer_id,
            SUM(points) AS total_points
    FROM
            (SELECT s.customer_id,
                COUNT(product_name) AS total_items,
                SUM(price) AS total_amount,
                product_name,
                CASE WHEN product_name IN ('curry', 'ramen') THEN (SUM(price)*10)
                    ELSE (SUM(price)*20) END AS points
            FROM sales AS s
            JOIN menu AS m
            ON s.product_id = m.product_id
            GROUP BY s.customer_id, product_name
            ORDER BY s.customer_id DESC) sub
    GROUP BY customer_id
    ORDER BY customer_id

Customers A, B, and C had 860, 940, and 360 points each.

![](Output%20Images/Answer%20(Number%20Nine%20-%20Danny's%20Diner).PNG)

# 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

Subquery was used in this question because the solution to this was also complicated. 

This time, when deriving the column, "points, I used three conditions. Notice that I added six days to the join date from members table so oly dates within one week of joining will have 20 points.

My main query only included the customers and the total points in the output. Because the customer only wanted to see points in January, I filtered the order date to dates less than 1st day of february. 

This included only January because the records started from January.

    SELECT customer_id,
        SUM(points) AS total_points
    FROM
        (SELECT s.customer_id,
                COUNT(product_name) AS total_items,
                SUM(price) AS total_amount,
                product_name,
                order_date,
                join_date,
                CASE WHEN product_name IN ('curry', 'ramen', 'sushi') AND (order_date BETWEEN join_date AND join_date + 6) THEN SUM(price)*20
                WHEN product_name IN ('curry', 'ramen') AND (order_date NOT BETWEEN join_date AND join_date + 6) THEN SUM(price)*10
                WHEN product_name = 'sushi' AND (order_date NOT BETWEEN join_date AND join_date + 6) THEN SUM(price)*20
                END AS points
            FROM sales AS s
            JOIN menu AS m
            ON s.product_id = m.product_id
            JOIN members AS mem
            ON s.customer_id = mem.customer_id
            GROUP BY s.customer_id, product_name, order_date, mem.join_date
            ORDER BY s.customer_id DESC) sub
    WHERE order_date < '2021-02-01'
    GROUP BY customer_id

At the end of January, Customer A and Customer B had 1370 and 820 points respectively. 

Customer C is not in the ouput because he was never a member.

![](Output%20Images/Answer%20(Number%20Ten%20-%20Danny's%20Diner).PNG)

# Bonus Questions

# 1. Write the query that recreates the image below: 

![](Output%20Images/Bonus%20Question%201%20-%20Danny's%20Diner.PNG)

I derived the "member" column using the CASE WHEN statement to identify the status of customers using certain conditions. 


    SELECT s.customer_id,
        order_date,
        product_name,
        price,
        CASE WHEN order_date < join_date THEN 'N'
        WHEN join_date IS NULL THEN 'N'
        ELSE 'Y' END AS member
    FROM sales AS s
    JOIN menu AS m
    ON s.product_id = m.product_id
    LEFT JOIN members AS mem
    ON mem.customer_id = s.customer_id
    ORDER BY s.customer_id, order_date

# 2. Write a query that recreates the image below:

![](Output%20Images/Bonus%20Question%202%20-%20Danny's%20Diner.PNG)

To make the code readable, I used Common Table Expressions instead of subqueries. I created a table containing all columns in the image above except the ranking column. 

Then my final query selected all the columns in the derived table and added an extra column called ranking which was derived as a condition based on the member column. 

    WITH T1 AS
        (SELECT s.customer_id,
            order_date,
            product_name,
            price,
            CASE WHEN order_date < join_date THEN 'N'
            WHEN join_date IS NULL THEN 'N'
            ELSE 'Y' END AS member
        FROM sales AS s
        JOIN menu AS m
        ON s.product_id = m.product_id
        LEFT JOIN members AS mem
        ON mem.customer_id = s.customer_id
        ORDER BY s.customer_id, order_date) 


    SELECT *,
            CASE WHEN T1.member = 'Y' THEN RANK() OVER(PARTITION BY T1.customer_id ORDER BY order_date)
            END AS ranking
    FROM T1


This is the end of my first case study on the Danny Ma SQL challenge (smiles). To join the challenge, you can click [HERE](https://8weeksqlchallenge.com/getting-started/). 

If you have any questions or suggestions as regards my solutions to this, you can send me a [message on twitter](https://twitter.com/AmandiNancy). 

Thank you for reading! Hopefully you'll read about my next case study too, right? See you soon. 