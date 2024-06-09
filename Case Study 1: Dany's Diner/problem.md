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
Looks like customer A and B, which have the highest and relatively close spending, are also members of the customer loyalty program (as seen schema.md, Members table) . This is a good indicator of the effectiveness of the program.  

| customer_id | total_spending |
| ----------- | -------------- |
| A           | 76             |
| B           | 74             |
| C           | 36             |

#### 2. How many days has each customer visited the restaurant?
Note that Customer A which has the highest spending (as stated above) is not the most frequent visitor of the store on a daily basis. This indicates that we should not expect customer with highest spending with highest visit.

| customer_id | total_visit |
| ----------- | ----------- |
| B           | 6           |
| A           | 4           |
| C           | 2           |

#### 3. What was the first item from the menu purchased by each customer?
The table belows shows that first bought item varies for each customer. Perhaps when we introduce variation of our products, for instance, sushi, we can offer it to customer B which shows interest in sushi. 

| customer_id | product_name |
| ----------- | ------------ |
| A           | curry        |
| B           | sushi        |
| C           | ramen        |

#### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
Given this, we could take advantage of the ramen popularity by creating different variation of ramen. 

| product_name | number_of_purchases |
| ------------ | ------------------- |
| ramen        | 8                   |

#### 5. Which item was the most popular for each customer?
As shown below, ramen is the most sought food among all customers. Additionally, we can see that customer B has variety in purchases.

| customer_id | product_name | number_of_times_bought |
| ----------- | ------------ | ---------------------- |
| A           | ramen        | 3                      |
| B           | ramen        | 2                      |
| B           | curry        | 2                      |
| B           | sushi        | 2                      |
| C           | ramen        | 3                      |


#### 6. Which item was purchased first by the customer after they became a member?
Customer A bought ramen(not similar with first purchase on the diner) and upon inspection of transaction after its membership, Customer A consistently bought ramen. Additionally, Customer B bought sushi (its first item purchased not being a member and after being a member). This shows that behavior of customer A changed while B do not.

| customer_id | product_name |
| ----------- | ------------ |
| A           | ramen        |
| B           | sushi        |


#### 7. Which item was purchased just before the customer became a member?
Before customer A become a member, he was purchasing relatively expensive food (curry) but started patronizing less expensive food such as ramen when he became a member (curry-15 and ramen-12). Additionally, customer bought sushi before and after she become a member.

| customer_id | product_name | 
| ----------- | ------------ |
| A           | curry        |
| B           | sushi        |

---

#### 8. What is the total items and amount spent for each member before they became a member?
Interestingly, even when customer A bought ramen ($3 cheaper than curry) before they become a member, its spending grew due to total items purchased. However, customer B's number of items purchased remained the same but the total spending become smaller after they became member. This means that customer B settled for relative cheaper food after being a member. 

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
The table shows that customer B, which is avid patronizer of sushi has the highest points on customer loyalty. 

| customer_id | total_points |
| ----------- | ------------ |
| B           | 940          |
| A           | 860          |
| C           | 360          |


#### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
For the month of January, which customer A and B become members, shows that customer A has the highest points. This is not suprising since A increased its total items purchased as shown in question number 8.

| customer_id | total_points |
| ----------- | ------------ |
| A           | 1370         |
| B           | 820          |

