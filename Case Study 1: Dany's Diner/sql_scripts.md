
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

    WITH first_order
    AS (
    	SELECT *
    	FROM dannys_diner.sales AS sales
    	LEFT JOIN dannys_diner.menu using (product_id)
    	)
    SELECT customer_id
    	,count(product_name) number_of_times_bought
    FROM first_order
    WHERE product_name = (
    		SELECT product_name
    		FROM first_order
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

