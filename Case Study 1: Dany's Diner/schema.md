# The following is the schema of Danny's Diner
```
CREATE SCHEMA dannys_diner;
SET search_path = dannys_diner;
```


Shown below is the relationship between tables in the schema. The image belows shows that Sales Table is the fact table and Menu & and Members Table are dimension tables that are connected by product and customer id.
![image](https://github.com/marizethpb/8-Week-SQL-Challenge/assets/79640443/2ab71570-e3c4-46d8-ac8f-50fa2a06fb3d)



### The Sales Table 
This includes information about the transactions in Danny's Diner. The table includes data about customer's id, the date purchasing, and the id of the product they bought. 

```
CREATE TABLE sales (
  "customer_id" VARCHAR(1),
  "order_date" DATE,
  "product_id" INTEGER
);

INSERT INTO sales
  ("customer_id", "order_date", "product_id")
VALUES
  ('A', '2021-01-01', '1'),
  ('A', '2021-01-01', '2'),
  ('A', '2021-01-07', '2'),
  ('A', '2021-01-10', '3'),
  ('A', '2021-01-11', '3'),
  ('A', '2021-01-11', '3'),
  ('B', '2021-01-01', '2'),
  ('B', '2021-01-02', '2'),
  ('B', '2021-01-04', '1'),
  ('B', '2021-01-11', '1'),
  ('B', '2021-01-16', '3'),
  ('B', '2021-02-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-07', '3');
```

### The Menu Table
The table which contains the products that Danny's Diner offer, together with id and price.
```
CREATE TABLE menu (
  "product_id" INTEGER,
  "product_name" VARCHAR(5),
  "price" INTEGER
);

INSERT INTO menu
  ("product_id", "product_name", "price")
VALUES
  ('1', 'sushi', '10'),
  ('2', 'curry', '15'),
  ('3', 'ramen', '12');
```

### The Members Table
The members refers to the customers that are part of customer loyalty program (beta). The table display their id and date of joining the program.
```
INSERT INTO members
  ("customer_id", "join_date")
VALUES
  ('A', '2021-01-07'),
  ('B', '2021-01-09');
```
