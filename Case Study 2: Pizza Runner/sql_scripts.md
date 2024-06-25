### A. Pizza Metrics

#### 1.	How many pizzas were ordered?
For this question, I determined number of pizzas ordered by counting the customer_id.

    SELECT 
       count(customer_id) total_orders 
    FROM 
      pizza_runner.customer_orders;

| total_orders |
| ------------ |
| 14           |

#### 2.	How many unique customer orders were made?
To answer this question, I first summarized the total count for each order. Then I counted orders in summarized table that only has one count
because "unique" means an object only exist once. 

    SELECT COUNT(order_id) total_unique_orders
    FROM 
        (SELECT order_id, count(order_id) total_orders
         FROM pizza_runner.customer_orders
         GROUP BY order_id) total_customer_orders
    WHERE total_orders = 1;

| total_unique_orders |
| ------------------- |
| 7                   |

#### 3.	How many successful orders were delivered by each runner?
For this question, I did inner join on customer_orders table and runner_orders table. Then I counted the order_id which has
specified pickup_time (null or blank pickup_time means it was cancelled) for each runner.

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
To answer this question, I did inner join on customer_orders table (to link runner_orders and pizza_names table), runner_orders (to determine delivered orders), and pizza_names (to get names for each type of pizza). After that, I counted the orders with specified pickup_time (sign that it was delivered) per pizza name.

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
Similar with previous question, I also did inner join on customer_orders table and and pizza_names to link the orders to the name of the pizza.
So for each customer and pizza name, I counted the name of pizza and that corresponds to total purchase of pizza per pizza type.

    SELECT 
      CO.customer_id, 
      pizza_name, 
      count(pizza_name) total_orders 
    FROM 
      pizza_runner.customer_orders as CO 
      INNER JOIN pizza_runner.pizza_names as PN USING(pizza_id) 
    GROUP BY 
      CO.customer_id, 
      pizza_name 
    ORDER BY 
      CO.customer_id;

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
To answer this question, I did inner join on customer_orders and runner_orders table and filtered the result to determined the delivered orders.
From there, I grouped the customer_id and counted the order_id to get sum of orders per customer. I ordered the table by number of orders and limit
returned query to 1st row only to get the top 1.

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
To determine first the number changes on delivered pizzas, I created a CTE that tracks the changes in extras and exlusions by inspecting whether the row
is populated by value (meaning that it had been changed). The data used in CTE was filtered table (to confirmed that order was delivered) from inner joined of customer_orders and runner_orders.

Then I used the resulting table in CTE to derive the number of pizzas that had no changes by calculating difference between total delivered orders and delivered
orders with change.

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
Similar approach to question above, the only difference is that the I am only counting the rows with both exclusions and extras populated to 
determine number of delivered pizzas that had both exclusions and extras.

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
For this question, I created the Hour_List CTE to create a table that represents 24 hrs a day and initial total order of 0. Additionally, I created Orders_Per_Hour CTE to get summarized information about total orders per hour and customer_id. 

Then I did left join on Hour_List and Orders_Per_Hour to get entire list of total orders per hour. 

    WITH 
      Hour_List as (
        SELECT 
          hours, 
          total_orders 
        from 
          generate_series(1, 24) as a(hours), 
          generate_series(0, 0) as total_orders
      ), 
      Orders_Per_Hour as (
        SELECT 
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
      HL.hours, 
      sum(CASE WHEN orders >= 0 THEN orders ELSE 0 END) as total_orders 
    FROM 
      Hour_List HL 
      LEFT JOIN Orders_Per_Hour OPH USING (hours) 
    GROUP BY
      HL.hours
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
| 18    | 3            |
| 19    | 1            |
| 20    | 0            |
| 21    | 3            |
| 22    | 0            |
| 23    | 3            |
| 24    | 0            |

#### 10. What was the volume of orders for each day of the week?
For this question, I created days_in_a_week CTE that returns a table that contains the names of days in a week and order of that day in the week. Also, I created Orders_Per_Day CTE to summarize the total orders per day.

I then queried day of the week and total order for that day from left joined table of days_in_a_week and Orders_Per_Day.

    WITH days_in_a_week AS (
      SELECT 
        day_order, 
        CASE WHEN day_order = 0 THEN 'Sunday' 
             WHEN day_order = 1 THEN 'Monday'
             WHEN day_order = 2 THEN 'Tuesday'
             WHEN day_order = 3 THEN 'Wednesday'
             WHEN day_order = 4 THEN 'Thursday'
             WHEN day_order = 5 THEN 'Friday'
             ELSE 'Saturday' END AS day_name 
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
