### A. Pizza Metrics

#### 1.	How many pizzas were ordered?
I counted the 

    SELECT 
       count(customer_id) total_orders 
    FROM 
      pizza_runner.customer_orders;

| total_orders |
| ------------ |
| 14           |

#### 2.	How many unique customer orders were made?

    SELECT 
      count(
        distinct(customer_id)
      ) total_unique_orders 
    FROM 
      pizza_runner.customer_orders;

| total_unique_orders |
| ------------------- |
| 5                   |

#### 3.	How many successful orders were delivered by each runner?

    SELECT 
      runner_id, 
      count(order_id) successful_orders 
    FROM 
      pizza_runner.customer_orders 
      INNER JOIN pizza_runner.runner_orders USING (order_id) 
    WHERE 
      pickup_time NOT IN ('null', '') 
    GROUP BY 
      runner_id;

| runner_id | successful_orders |
| --------- | ----------------- |
| 3         | 1                 |
| 2         | 5                 |
| 1         | 6                 |


#### 4.	How many of each type of pizza was delivered?

    SELECT 
      pizza_name, 
      count(order_id) successful_delivers 
    FROM 
      pizza_runner.customer_orders as CO 
      INNER JOIN pizza_runner.runner_orders as RO USING (order_id) 
      INNER JOIN pizza_runner.pizza_names as PN USING(pizza_id) 
    WHERE 
      pickup_time NOT IN ('null', '') 
    GROUP BY 
      pizza_name;

| pizza_name | successful_delivers |
| ---------- | ------------------- |
| Meatlovers | 9                   |
| Vegetarian | 3                   |

#### 5.	How many Vegetarian and Meatlovers were ordered by each customer?

    SELECT 
      CO.customer_id, 
      pizza_name, 
      count(pizza_name) total_orders 
    FROM 
      pizza_runner.customer_orders as CO 
      INNER JOIN pizza_runner.runner_orders as RO USING (order_id) 
      INNER JOIN pizza_runner.pizza_names as PN USING(pizza_id) 
    WHERE 
      pickup_time NOT IN ('null', '') 
    GROUP BY 
      CO.customer_id, 
      pizza_name 
    ORDER BY 
      CO.customer_id;

| customer_id | pizza_name | total_orders |
| ----------- | ---------- | ------------ |
| 101         | Meatlovers | 2            |
| 102         | Meatlovers | 2            |
| 102         | Vegetarian | 1            |
| 103         | Meatlovers | 2            |
| 103         | Vegetarian | 1            |
| 104         | Meatlovers | 3            |
| 105         | Vegetarian | 1            |


#### 6.	What was the maximum number of pizzas delivered in a single order?


    SELECT 
      customer_id, 
      count(order_id) maximum_pizza 
    FROM 
      pizza_runner.customer_orders CO 
      INNER JOIN pizza_runner.runner_orders RO USING (order_id) 
    WHERE 
      pickup_time NOT IN ('null', '') 
    GROUP BY 
      CO.customer_id, 
      CO.order_id 
    ORDER BY 
      count(order_id) DESC 
    LIMIT 
      1;

| customer_id | maximum_pizza |
| ----------- | ------------- |
| 103         | 3             |


#### 7.	For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

    WITH TOTAL_CHANGES AS (
      SELECT 
        CO.customer_id, 
        sum(
          CASE WHEN exclusions NOT IN ('null', '') THEN 1 ELSE 0 END
        ) + sum(
          CASE WHEN extras NOT IN ('null', '') THEN 1 ELSE 0 END
        ) AS changes, 
        count(customer_id) total_orders 
      FROM 
        pizza_runner.customer_orders CO 
        INNER JOIN pizza_runner.runner_orders RO USING (order_id) 
      WHERE 
        pickup_time NOT IN ('null', '') 
      GROUP BY 
        CO.customer_id
    ) 
    SELECT 
      customer_id, 
      changes, 
      total_orders - changes as no_changes 
    FROM 
      TOTAL_CHANGES 
    ORDER BY 
      customer_id;

| customer_id | changes | no_changes |
| ----------- | ------- | ---------- |
| 101         | 0       | 2          |
| 102         | 0       | 3          |
| 103         | 3       | 0          |
| 104         | 3       | 0          |
| 105         | 1       | 0          |


#### 8.	How many pizzas were delivered that had both exclusions and extras?


    SELECT 
      sum(
        CASE WHEN exclusions NOT IN ('null', '') 
        AND extras NOT IN ('null', '') THEN 1 ELSE 0 END
      ) AS both_changes 
    FROM 
      pizza_runner.customer_orders CO 
      INNER JOIN pizza_runner.runner_orders RO USING (order_id) 
    WHERE 
      pickup_time NOT IN ('null', '');

| both_changes |
| ------------ |
| 1            |

#### 9.	What was the total volume of pizzas ordered for each hour of the day?


    WITH Hour_List as (
      SELECT 
        hours, 
        total_orders 
      from 
        generate_series(1, 24) as a(hours), 
        generate_series(0, 0) as total_orders, 
        generate_series(1, 1)
    ), 
    Orders_Per_Hour as (
      SELECT 
        customer_id, 
        EXTRACT(
          hour 
          FROM 
            order_time
        ) hours, 
        count(customer_id) orders 
      FROM 
        pizza_runner.customer_orders CO 
      GROUP BY 
        customer_id, 
        EXTRACT(
          hour 
          FROM 
            order_time
        ) 
      ORDER BY 
        EXTRACT(
          hour 
          FROM 
            order_time
        )
    ) 
    SELECT 
      hours, 
      CASE WHEN orders >= 0 THEN orders ELSE 0 END as total_orders 
    FROM 
      Hour_List HL 
      LEFT JOIN Orders_Per_Hour OPH USING (hours) 
    ORDER BY 
      HL.hours;

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
| 18    | 2            |
| 18    | 1            |
| 19    | 1            |
| 20    | 0            |
| 21    | 1            |
| 21    | 1            |
| 21    | 1            |
| 22    | 0            |
| 23    | 3            |
| 24    | 0            |


#### 10. What was the volume of orders for each day of the week?

    WITH days_in_a_week AS (
      SELECT 
        day_order, 
        CASE WHEN day_order = 0 THEN 'Sunday' WHEN day_order = 1 THEN 'Monday' WHEN day_order = 2 THEN 'Tuesday' WHEN day_order = 3 THEN 'Wednesday' WHEN day_order = 4 THEN 'Thursday' WHEN day_order = 5 THEN 'Friday' ELSE 'Saturday' END AS day_name 
      FROM 
        generate_series(0, 6) AS a(day_order)
    ), 
    Orders_Per_Day AS (
      SELECT 
        EXTRACT(
          dow 
          FROM 
            order_time
        ) AS day_order, 
        COUNT(customer_id) AS orders 
      FROM 
        pizza_runner.customer_orders 
      GROUP BY 
        EXTRACT(
          dow 
          FROM 
            order_time
        )
    ) 
    SELECT 
      days_in_a_week.day_name AS day, 
      COALESCE(orders, 0) AS total_orders 
    FROM 
      days_in_a_week 
      LEFT JOIN Orders_Per_Day OPD ON days_in_a_week.day_order = OPD.day_order 
    ORDER BY 
      days_in_a_week.day_order;

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
#### 2.	What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
#### 3.	Is there any relationship between the number of pizzas and how long the order takes to prepare?
#### 4.	What was the average distance travelled for each customer?
#### 5.	What was the difference between the longest and shortest delivery times for all orders?
#### 6.	What was the average speed for each runner for each delivery and do you notice any trend for these values?
#### 7.	What is the successful delivery percentage for each runner?

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
