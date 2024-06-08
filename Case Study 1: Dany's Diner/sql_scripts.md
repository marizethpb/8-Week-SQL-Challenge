
## 1. What is the total amount each customer spent at the restaurant?
To answer this question, I did left join on the sales and product table to get the price of products on customer's transaction. I then sum the price of each product for each customer as each row is one transaction, one product only. 

    SELECT customer_id
    	,sum(price) AS total_spending
    FROM dannys_diner.sales AS sales
    LEFT JOIN dannys_diner.menu using (product_id)
    GROUP BY customer_id
    ORDER BY total_spending DESC;

| customer_id | total_spending |
| ----------- | -------------- |
| A           | 76             |
| B           | 74             |
| C           | 36             |

---

## 2. How many days has each customer visited the restaurant?
To answer this question, I counted the distinct order date 

    SELECT customer_id
    	,count(DISTINCT (order_date)) AS total_visit
    FROM dannys_diner.sales
    GROUP BY customer_id
    ORDER BY total_visit DESC;

| customer_id | total_visit |
| ----------- | ----------- |
| B           | 6           |
| A           | 4           |
| C           | 2           |

---

## 3. What was the first item from the menu purchased by each customer?

    WITH first_order
    AS (
    	SELECT *
    		,row_number() OVER (PARTITION BY customer_id) order_of_purchase
    	FROM dannys_diner.sales AS sales
    	LEFT JOIN dannys_diner.menu using (product_id)
    	)
    
    SELECT customer_id
    	,product_name
    FROM first_order
    WHERE order_of_purchase = 1
    ORDER BY customer_id;

| customer_id | product_name |
| ----------- | ------------ |
| A           | curry        |
| B           | sushi        |
| C           | ramen        |

---

## 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

    SELECT product_name, count(product_name) number_of_purchases
    FROM dannys_diner.sales AS sales
    LEFT JOIN dannys_diner.menu using (product_id)
    GROUP BY product_name
    ORDER BY count(product_name) DESC limit 1;

| product_name | number_of_purchases |
| ------------ | ------------------- |
| ramen        | 8                   |
--- 

## 5. Which item was the most popular for each customer?

    WITH total_purchases_by_product
         AS (SELECT customer_id,
                    product_name,
                    Count(product_name) number_of_times_bought
             FROM   dannys_diner.sales AS sales
                    LEFT JOIN dannys_diner.menu using (product_id)
             GROUP  BY customer_id,
                       product_name
             ORDER  BY customer_id,
                       Count(product_name) DESC)
    SELECT TPa.customer_id,
           TPa.product_name,
           Tpa.number_of_times_bought
    FROM   total_purchases_by_product AS TPa
    WHERE  TPa.number_of_times_bought = (SELECT Max(TPb.number_of_times_bought)
                                         FROM   total_purchases_by_product AS TPb
                                         WHERE  TPa.customer_id = TPb.customer_id
                                         GROUP  BY TPb.customer_id)
    ORDER  BY TPa.customer_id;

| customer_id | product_name | number_of_times_bought |
| ----------- | ------------ | ---------------------- |
| A           | ramen        | 3                      |
| B           | ramen        | 2                      |
| B           | curry        | 2                      |
| B           | sushi        | 2                      |
| C           | ramen        | 3                      |

---

## 6. Which item was purchased first by the customer after they became a member?
I didn't arrange the date because theres not timestammp

     WITH member_purchases
         AS (SELECT *,
                    Rank()
                      OVER (
                         partition BY customer_id 
                         order by order_date ASC) order_of_purchase
             FROM   dannys_diner.sales AS sales
             LEFT JOIN dannys_diner.menu using (product_id)
             LEFT JOIN dannys_diner.members using (customer_id)
             WHERE  order_date > join_date)
             
    SELECT customer_id,
           product_name
    FROM   member_purchases
    WHERE  order_of_purchase = 1;

| customer_id | product_name |
| ----------- | ------------ |
| A           | ramen        |
| B           | sushi        |

---
## 7. Which item was purchased just before the customer became a member?

    WITH member_purchases
         AS (SELECT *,
                    Rank ()
                      OVER (
                        partition BY customer_id
                        order by order_date DESC) order_of_purchase
             FROM   dannys_diner.sales AS sales
             LEFT JOIN dannys_diner.menu using (product_id)
             LEFT JOIN dannys_diner.members using (customer_id)
             WHERE  order_date < join_date)
             
    SELECT customer_id, 
           product_name
    FROM   member_purchases 
    WHERE  order_of_purchase = 1;
             

| customer_id | product_name | 
| ----------- | ------------ |
| A           | curry        |
| B           | sushi        |
---

## 8. What is the total items and amount spent for each member before they became a member?

    SELECT    customer_id,
              Count(product_name) AS total_items_purchased,
              Sum(price)          AS total_spent
    FROM      dannys_diner.sales
    LEFT JOIN dannys_diner.menu using  (product_id)
    LEFT JOIN dannys_diner.members using  (customer_id)
    WHERE     sales.order_date < join_date
    GROUP BY  customer_id
    ORDER BY  customer_id;

| customer_id | total_items_purchased | total_spent |
| ----------- | --------------------- | ----------- |
| A           | 2                     | 25          |
| B           | 3                     | 40          |

---
## What is the total items and amount spent for each member after they became a member?

        SELECT    customer_id,
                  Count(product_name) AS total_items_purchased,
                  Sum(price)          AS total_spent
        FROM      dannys_diner.sales
        LEFT JOIN dannys_diner.menu using  (product_id)
        LEFT JOIN dannys_diner.members using  (customer_id)
        WHERE     sales.order_date > join_date
        GROUP BY  customer_id
        ORDER BY  customer_id;

| customer_id | total_items_purchased | total_spent |
| ----------- | --------------------- | ----------- |
| A           | 3                     | 36          |
| B           | 3                     | 34          |

---
## 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

    WITH customer_points AS
    (
              SELECT    customer_id,
                        product_name,
                        price,
                        order_date,
                        CASE
                                  WHEN product_name = 'sushi' THEN price*20
                                  ELSE price *10
                        END                AS points
              FROM      dannys_diner.sales AS sales
              LEFT JOIN dannys_diner.menu
              using     (product_id)
              ORDER BY  customer_id)
    
    SELECT   customer_id,
             Sum(points) AS total_points
    FROM     customer_points
    GROUP BY customer_id
    ORDER BY total_points DESC;

| customer_id | total_points |
| ----------- | ------------ |
| B           | 940          |
| A           | 860          |
| C           | 360          |

---
## 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

WITH customer_points AS
(
          SELECT    customer_id,
                    product_name,
                    price,
                    order_date,
                    CASE
                              WHEN product_name = 'sushi'
                              OR  order_date BETWEEN join_date AND join_date + integer '6' THEN price*20
                              ELSE price *10
                    END AS adjusted_points
          FROM      dannys_diner.sales AS sales
          LEFT JOIN dannys_diner.menu using (product_id)
  		  INNER JOIN dannys_diner.members using (customer_id)
  		  WHERE extract(month from order_date) = 1
          ORDER BY  customer_id)

SELECT   customer_id,
         sum(adjusted_points) AS total_points
FROM     customer_points
GROUP BY customer_id
ORDER BY total_points DESC;

| customer_id | total_points |
| ----------- | ------------ |
| A           | 1370         |
| B           | 820          |


