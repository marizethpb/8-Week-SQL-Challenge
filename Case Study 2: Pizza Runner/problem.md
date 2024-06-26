# The following are the business problem, objective of the analysis, and business questions.

## Background
Danny was scrolling through his Instagram feed when something really caught his eye - “80s Retro Styling and Pizza Is The Future!”

Danny was sold on the idea, but he knew that pizza alone was not going to help him get seed funding to expand his new Pizza Empire - so he had one more genius idea to combine with it - he was going to Uberize it - and so Pizza Runner was launched!

Danny started by recruiting “runners” to deliver fresh pizza from Pizza Runner Headquarters (otherwise known as Danny’s house) and also maxed out his credit card to pay freelance developers to build a mobile app to accept orders from customers.

## Business Problem Statement
Danny wanted to explore various metrics of the business from pizza metrics to pricing and ratings.

## Objective of analysis
1. Pizza Metrics
2. Runner and Customer Experience
3. Ingredient Optimisation
4. Pricing and Ratings

## Business Questions


### A. Pizza Metrics

#### 1.	How many pizzas were ordered?
There were a total of 14 orders. 

| total_orders |
| ------------ |
| 14           |

#### 2.	How many unique customer orders were made?
There are 7 unique customer orders. 

| total_unique_orders |
| ------------------- |
| 7                   |

#### 3.	How many successful orders were delivered by each runner?
As shown below, runner 1 had the highest delivered orders while runner 3 had the lowest delivered orders. Given this, we should investigate the factors about the huge difference in delivered orders by perhaps interviewing runner 1 and 3.

| runner_id | successful_orders |
| --------- | ----------------- |
| 3         | 1                 |
| 2         | 5                 |
| 1         | 6                 |

#### 4.	How many of each type of pizza was delivered?
Shown below are the successful deliveries per type of pizza. It is evident that Meatlovers had the highest orders which also means that people like meat pizzas. Given this, we should increase the supply of our meat pizzas and/or decrease the vegetarian pizza as the former is relatively in demand. 

| pizza_name | successful_delivers |
| ---------- | ------------------- |
| Meatlovers | 9                   |
| Vegetarian | 3                   |

#### 5.	How many Vegetarian and Meatlovers were ordered by each customer?
Now to drill down who likes vegetarian and meatlovers, we look at the table below. Apparently, most of the customers likes Meatlovers more than Vegetarian. 

| customer_id | pizza_name | total_orders |
| ----------- | ---------- | ------------ |
| 101         | Meatlovers | 2            |
| 101         | Vegetarian | 1            |
| 102         | Meatlovers | 2            |
| 102         | Vegetarian | 1            |
| 103         | Meatlovers | 3            |
| 103         | Vegetarian | 1            |
| 104         | Meatlovers | 3            |
| 105         | Vegetarian | 1            |

#### 6.	What was the maximum number of pizzas delivered in a single order?
3 pizzas was the maximum number of ordered pizza in a single delivery. Specifically, it was ordered by customer 103.

| customer_id | maximum_pizza |
| ----------- | ------------- |
| 103         | 3             |

#### 7.	For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
As shown below, every customer picked a side. It's either they'll have changes in their order or not and they let it stay that way.

| customer_id | changes | no_changes |
| ----------- | ------- | ---------- |
| 101         | 0       | 2          |
| 102         | 0       | 3          |
| 103         | 3       | 0          |
| 104         | 3       | 0          |
| 105         | 1       | 0          |

#### 8.	How many pizzas were delivered that had both exclusions and extras?
So far, there is only one delivered order that had both exlusions and extras in the pizza.

| both_changes |
| ------------ |
| 1            |

#### 9.	What was the total volume of pizzas ordered for each hour of the day?
Apparently, the pizza orders culminate at night, specifically from 6pm to 11 pm. Given this, we should decrease pizza workers at day shift and then reallocate them during night time.

| hours | total_orders |
| ----- | ------------ |
| 1     | 0            |
| 2     | 0            |
| 3     | 0            |
| 4     | 0            |
| 5     | 0            |
| 6     | 0            |
| 7     | 0            |
| 8     | 0            |
| 9     | 0            |
| 10    | 0            |
| 11    | 1            |
| 12    | 0            |
| 13    | 3            |
| 14    | 0            |
| 15    | 0            |
| 16    | 0            |
| 17    | 0            |
| 18    | 3            |
| 19    | 1            |
| 20    | 0            |
| 21    | 3            |
| 22    | 0            |
| 23    | 3            |
| 24    | 0            |


#### 10. What was the volume of orders for each day of the week?
Given the table below, the orders cluster from Wednesday to Saturday. We could also realloate some pizza workers on days with no orders to days with higher volume of order such as Wednesday and Saturday.

| day       | total_orders |
| --------- | ------------ |
| Sunday    | 0            |
| Monday    | 0            |
| Tuesday   | 0            |
| Wednesday | 5            |
| Thursday  | 3            |
| Friday    | 1            |
| Saturday  | 5            |

#  
### B. Runner and Customer Experience
#### 1.	How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
Shown below is the registered runners per week. Evidently, the registered runners peak at first week of January.

| week                     | registered_runners |
| ------------------------ | ------------------ |
| 2021-01-01T00:00:00.000Z | 2                  |
| 2021-01-08T00:00:00.000Z | 1                  |
| 2021-01-15T00:00:00.000Z | 1                  |

#### 2.	What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
The average time shows that runner 3 is the first while runner 2 is the last reach to Pizza HQ. 

| runner_id | average_minutes |
| --------- | --------------- |
| 1         | 15.68           |
| 2         | 23.72           |
| 3         | 10.47           |


#### 3.	Is there any relationship between the number of pizzas and how long the order takes to prepare?
The correlation coefficient for number of pizzas and the preparation duration is 0.84. This means that there is a very strong positive association with two variables. An increase in the number of pizza is very strongly associated to increased preparation duration. 

| correl|
| ----- |
| 0.84  |

#### 4.	What was the average distance travelled for each customer?
Shown below are the average distance of deliveries for each customer. The table below shows that customer 104 and 105 have the nearest and farthest distances, respectively. 

| customer_id | average_distance |
| ----------- | ---------------- |
| 101         | 20.00            |
| 102         | 16.73            |
| 103         | 23.40            |
| 104         | 10.00            |
| 105         | 25.00            |

#### 5.	What was the difference between the longest and shortest delivery times for all orders?
30 KM is the difference between the longest and shortest delivery. This is quiet a far distance for deliveries and we might consider establishing branches as the distance will be crucial in order cancellation and overall customer experience.
| max_min_diff |
| ------------ |
| 30           |

#### 6.	What was the average speed for each runner for each delivery?
Table below shows the average speed per runner and order. Apparently, the average speed tend to increase as the order id increases. This means that for every additional delivery, the runner gets tired and it takes toll on their speed. One thing to consider is the distance of delivery. The farther the distance, the more tired the rider is and it leads to reduction of their speed.


| runner_id | order_id | average_speed_duration_km_per_min |
| --------- | -------- | --------------------------------- |
| 1         | 1        | 0.63                              |
| 1         | 2        | 0.74                              |
| 1         | 3        | 0.67                              |
| 1         | 10       | 1.00                              |
| 2         | 4        | 0.59                              |
| 2         | 7        | 1.00                              |
| 2         | 8        | 1.56                              |
| 3         | 5        | 0.67                              |


#### 7.	What is the successful delivery percentage for each runner?
Listed below are the success rate for each rider. Interestingly, runner 2 has 25% more success rate than runner 3 eventhough runner 3 is the first while runner 2 is the last runner to reach Pizza HQ. 
| runner_id | success_rate |
| --------- | ------------ |
| 1         | 100 %        |
| 2         | 75 %         |
| 3         | 50 %         |

#  
###  C. Ingredient Optimisation
#### 1.	What are the standard ingredients for each pizza?
#### 2.	What was the most commonly added extra?
#### 3.	What was the most common exclusion?
#### 4.	Generate an order item for each record in the customers_orders table in the format of one of the following:
        o	Meat Lovers
        o	Meat Lovers - Exclude Beef
        o	Meat Lovers - Extra Bacon
        o	Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers
#### 5.	Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
        o	For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"
#### 6.	What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?
#  
### D. Pricing and Ratings
#### 1.	If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?
#### 2.	What if there was an additional $1 charge for any pizza extras?
        o	Add cheese is $1 extra
#### 3.	The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.
#### 4.	Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
        o	customer_id
        o	order_id
        o	runner_id
        o	rating
        o	order_time
        o	pickup_time
        o	Time between order and pickup
        o	Delivery duration
        o	Average speed
        o	Total number of pizzas
#### 5.	If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?
