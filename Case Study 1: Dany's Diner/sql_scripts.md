
## 1. What is the total amount each customer spent at the restaurant?
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

   WITH total_purchases
    AS (
    	SELECT *
    	FROM dannys_diner.sales AS sales
    	LEFT JOIN dannys_diner.menu using (product_id)
    	)
        
    SELECT customer_id
    	,count(product_name) number_of_times_bought
    FROM total_purchases
    WHERE product_name = (
    		SELECT product_name
    		FROM total_purchases
    		GROUP BY product_name
    		ORDER BY count(product_name) DESC limit 1
    		)
    GROUP BY customer_id
    ORDER BY number_of_times_bought DESC;

| customer_id | number_of_times_bought |
| ----------- | ---------------------- |
| A           | 3                      |
| C           | 3                      |
| B           | 2                      |


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
           TPa.product_name
    FROM   total_purchases_by_product AS TPa
    WHERE  TPa.number_of_times_bought = (SELECT Max(TPb.number_of_times_bought)
                                         FROM   total_purchases_by_product AS TPb
                                         WHERE  TPa.customer_id = TPb.customer_id
                                         GROUP  BY TPb.customer_id)
    ORDER  BY TPa.customer_id;

| customer_id | product_name |
| ----------- | ------------ |
| A           | ramen        |
| B           | ramen        |
| B           | curry        |
| B           | sushi        |
| C           | ramen        |


## 6. Which item was purchased first by the customer after they became a member?

    WITH member_purchases
         AS (SELECT *,
                    Row_number()
                      OVER (
                        partition BY customer_id) order_of_purchase
             FROM   dannys_diner.sales AS sales
                    LEFT JOIN dannys_diner.menu using (product_id)
             WHERE  order_date > (SELECT join_date
                                  FROM   dannys_diner.members
                                  WHERE  sales.customer_id = members.customer_id))
    SELECT customer_id,
           product_name
    FROM   member_purchases
    WHERE  order_of_purchase = 1;

| customer_id | product_name |
| ----------- | ------------ |
| A           | ramen        |
| B           | sushi        |


## 7. Which item was purchased just before the customer became a member?

    WITH member_purchases
         AS (SELECT *,
                    Row_number()
                      OVER (
                        partition BY customer_id) order_of_purchase
             FROM   dannys_diner.sales AS sales
                    LEFT JOIN dannys_diner.menu using (product_id)
             WHERE  order_date < (SELECT join_date
                                  FROM   dannys_diner.members
                                  WHERE  sales.customer_id = members.customer_id))
    SELECT customer_id,
           product_name
    FROM   member_purchases AS MPa
    WHERE  order_of_purchase = (SELECT Max(MPb.order_of_purchase)
                                FROM   member_purchases MPb
                                WHERE  MPa.customer_id = MPb.customer_id);

| customer_id | product_name | 
| ----------- | ------------ |
| A           | curry        |
| B           | sushi        |

