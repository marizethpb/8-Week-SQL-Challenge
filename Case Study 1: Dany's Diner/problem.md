# The following are the business problem, objective of the analysis, and business questions.

## Background
Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen.

Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.

## Business Problem Statement
Danny's Diner is deciding to expand existing customer loyalty program (beta program). Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

## Objective of analysis
1. Customers' visiting patterns.
2. How much money each customer spent.
3. Which menu items are customers' favourite.

## Business Questions


#### 1. What is the total amount each customer spent at the restaurant?
Looks like customer A and B has highest and relatively close spending, Maybe we can interview them through incentive of product to determine what makes them loyal to our business.

| customer_id | total_spending |
| ----------- | -------------- |
| A           | 76             |
| B           | 74             |
| C           | 36             |

#### 2. How many days has each customer visited the restaurant?
| customer_id | total_visit |
| ----------- | ----------- |
| B           | 6           |
| A           | 4           |
| C           | 2           |

#### 3. What was the first item from the menu purchased by each customer?
| customer_id | product_name |
| ----------- | ------------ |
| A           | curry        |
| B           | sushi        |
| C           | ramen        |

#### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
| customer_id | number_of_times_bought |
| ----------- | ---------------------- |
| A           | 3                      |
| C           | 3                      |
| B           | 2                      |

#### 5. Which item was the most popular for each customer?
| customer_id | product_name | number_of_times_bought |
| ----------- | ------------ | ---------------------- |
| A           | ramen        | 3                      |
| B           | ramen        | 2                      |
| B           | curry        | 2                      |
| B           | sushi        | 2                      |
| C           | ramen        | 3                      |


#### 6. Which item was purchased first by the customer after they became a member?\
| customer_id | product_name |
| ----------- | ------------ |
| A           | ramen        |
| B           | sushi        |


#### 7. Which item was purchased just before the customer became a member?
| customer_id | product_name | 
| ----------- | ------------ |
| A           | curry        |
| B           | sushi        |

---

#### 8. What is the total items and amount spent for each member before they became a member?

**BEFORE**
| customer_id | total_items_purchased | total_spent |
| ----------- | --------------------- | ----------- |
| A           | 2                     | 25          |
| B           | 3                     | 40          |

**AFTER**
| customer_id | total_items_purchased | total_spent |
| ----------- | --------------------- | ----------- |
| A           | 3                     | 36          |
| B           | 3                     | 34          |

#### 9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
| customer_id | total_points |
| ----------- | ------------ |
| B           | 940          |
| A           | 860          |
| C           | 360          |


#### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
| customer_id | total_points |
| ----------- | ------------ |
| A           | 1370         |
| B           | 820          |

