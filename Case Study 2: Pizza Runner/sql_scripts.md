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
For this question, I discovered the date_trunc function as a life saver. I used date_trunc to group the date into weeks and added 4 days to make the start date into '2021-01-01' because originally, date_trunc starts date at '2020-12-28'. I also counted the runner_id per week to account for registered_runners.

    SELECT    DATE_TRUNC('week', registration_date) + INTERVAL '4 days' AS WEEK
            , count(runner_id) AS registered_runners
    FROM pizza_runner.runners
    GROUP BY 1 -- WEEK
    ORDER BY 1 -- WEEK;

| week                     | registered_runners |
| ------------------------ | ------------------ |
| 2021-01-01T00:00:00.000Z | 2                  |
| 2021-01-08T00:00:00.000Z | 1                  |
| 2021-01-15T00:00:00.000Z | 1                  |


#### 2.	What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
To answer this question, I first did inner join on runner_orders and customers_orders table and filtered only delivered orders (not null or empty). I then get the            average minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order through average difference of pickup_time and order_time. I grouped the result per runner.

    SELECT runner_id,
           ROUND(AVG(EXTRACT(EPOCH
                             FROM pickup_time :: timestamp - order_time :: timestamp) / 60):: numeric, 2) AS average_minutes
    FROM pizza_runner.runner_orders RO
    INNER JOIN pizza_runner.customer_orders CO USING (order_id)
    WHERE RO.pickup_time not in ('null','')
    GROUP BY runner_id
    ORDER BY runner_id;

| runner_id | average_minutes |
| --------- | --------------- |
| 1         | 15.68           |
| 2         | 23.72           |
| 3         | 10.47           |


#### 3.	Is there any relationship between the number of pizzas and how long the order takes to prepare?
To answer this question, I created a CTE (orders_duration) that calculates the number of pizza and average duration per order. I did the same approach as the previous question, I just modified the runner_id to order_id and added total number of pizza.

Then, I used the orders_duration CTE to calculate for the correlation between the number of pizza and duration. The resulting output is the correlation coefficient.

     WITH orders_duration AS
      (SELECT order_id,
              count(order_id) number_of_pizza,
              ROUND(AVG(EXTRACT(EPOCH
                                FROM pickup_time :: timestamp - order_time :: timestamp) / 60):: numeric, 2) AS average_minutes
       FROM pizza_runner.runner_orders RO
       INNER JOIN pizza_runner.customer_orders CO USING (order_id)
       WHERE RO.pickup_time not in ('null','')
       GROUP BY order_id
       ORDER BY order_id)
   
    SELECT round(corr(number_of_pizza, average_minutes):: numeric, 2)
    FROM orders_duration;

| round |
| ----- |
| 0.84  |

#### 4.	What was the average distance travelled for each customer?
For this question, I did inner join on runner_orders and customer_orders and filtered delivered orders only. After that, I cleaned the row value by using regular expression in substring function and cast the resulting value as numeric. Now that the values are cleaned, I calculated the average distance per customer.

    SELECT customer_id, avg(substring(distance, '(\d+\.*\d*)')::numeric)::numeric(4,2) AS average_distance
    FROM pizza_runner.runner_orders INNER JOIN pizza_runner.customer_orders using (order_id)
    WHERE pickup_time not in ('null', '')
    GROUP BY customer_id
    ORDER BY customer_id;

| customer_id | average_distance |
| ----------- | ---------------- |
| 101         | 20.00            |
| 102         | 16.73            |
| 103         | 23.40            |
| 104         | 10.00            |
| 105         | 25.00            |


#### 5.	What was the difference between the longest and shortest delivery times for all orders?
In calculating the difference between the longest and shortest delivery times, I cleaned first the duration column to get the integers using regular expression and filtered the table to return delivered orders only. Finally, I used max and min function on the clean duration column and calculated the difference.

    SELECT max(substring(duration, '([\d]+)')::int) -  min(substring(duration, '([\d]+)')::int) AS max_min_diff
    FROM pizza_runner.runner_orders
    WHERE pickup_time not in ('null', '');

| max_min_diff |
| ------------ |
| 30           |


#### 6.	What was the average speed for each runner for each delivery and do you notice any trend for these values?
I calculated the average speed for each delivery by distance/duration on the table with delivered orders only. I did similar approach in previous question about cleaning the distance and duration column. Finally, I grouped the result by runner_id and order_id. 

    SELECT runner_id,
           order_id,
           avg(substring(distance, '(\d+\.*\d*)')::numeric / substring(duration, '([0-9]+)')::int)::numeric(3,2) as average_speed_duration_km_per_min
    FROM pizza_runner.runner_orders
    WHERE pickup_time not in ('null','')
    GROUP BY runner_id, order_id
    ORDER BY runner_id, order_id;

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


I wasn't satisfied with looking at the raw speed for each table, I wanted to know the correlation between the order_id and average speed as I have seen that increasing order_id (which estimates the latter-ness of the order) is also increasing as the speed. This was confirmed by the correlation coefficient displayed below. In calculating this result, I have used the previous select statement and used it as a CTE. Then, I used CORR function on the CTE to get the correlation coefficient of order_id and speed.

    WITH delivery_speed as (
    SELECT runner_id,
           order_id,
           avg(substring(distance, '(\d+\.*\d*)')::numeric / substring(duration, '([0-9]+)')::int)::numeric(3,2) as average_speed_duration_km_per_min
    FROM pizza_runner.runner_orders
    WHERE pickup_time not in ('null','')
    GROUP BY runner_id, order_id)
    
    SELECT
       CORR(order_id, average_speed_duration_km_per_min) order_id_and_speed 
    FROM
       delivery_speed;

| order_id_and_speed |
| ------------------ |
| 0.7055260437275165 |

I also calculated the correlation coefficient of total orders and average speed for each runner using the approach I did above.

    with delivery_speed_agg as (
           SELECT runner_id,
           count(order_id) total_orders,
           avg(substring(distance, '(\d+\.*\d*)')::numeric / substring(duration, '([0-9]+)')::int)::numeric(3,2) as average_speed_duration_km_per_min
    FROM pizza_runner.runner_orders
    WHERE pickup_time not in ('null','')
    GROUP BY runner_id);
    
    SELECT
       CORR(total_orders, average_speed_duration_km_per_min) total_order_and_speed 
    FROM
       delivery_speed_agg;

| total_order_and_speed |
| --------------------- |
| 0.40659340659340626   |


#### 7.	What is the successful delivery percentage for each runner?
The successful delivery percentage was calculated by ratio of total number of delivered orders to total number of orders assigned. In this query, I used 'case when' to count the numbers of successful deliveries. I grouped the result by runner_id and the following are the results.

    SELECT runner_id,
           concat(
             round(
             sum(CASE WHEN pickup_time not in ('null','') then 1 else 0 END) / count(order_id)::numeric
           *100)
           , ' %') AS success_rate
    FROM pizza_runner.runner_orders RO
    GROUP BY runner_id
    ORDER BY runner_id;

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
