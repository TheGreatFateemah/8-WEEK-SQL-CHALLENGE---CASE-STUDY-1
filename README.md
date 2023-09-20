# 8-WEEK-SQL-CHALLENGE---CASE-STUDY-1
This is the first case study of Danny's 8-week SQL challenge.

## Introduction

Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen.

Danny’s Diner is in need of my assistance to help the restaurant stay afloat - the restaurant has captured some fundamental data from its few months of operation but has no idea how to use their data to help them run the business.

## Problem Statement

Danny wants to use the data to get answers to a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

He plans on using these insights to help him decide whether he should expand the existing customer loyalty program.

Here are the questions:

1. What is the total amount each customer spent at the restaurant?
2. How many days has each customer visited the restaurant?
3. What was the first item from the menu purchased by each customer?
4. What is the most purchased item on the menu and how many times was it purchased by all customers?
5. Which item was the most popular for each customer?
6. Which item was purchased first by the customer after they became a member?
7. Which item was purchased just before the customer became a member?
8. What are the total items and amount spent for each member before they became a member?
9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customers A and B have at the end of January?

## Datasets Used

Three key datasets for this case study

- sales: The sales table captures all customer_id level purchases with a corresponding order_date and product_id information for when and what menu items were ordered.
- menu: The menu table maps the product_id to the actual product_name and price of each menu item.
- members: The members table captures the join_date when a customer_id joined the beta version of the Danny’s Diner loyalty program.

## Entity Relationship Diagram (ERD)

![](https://github.com/TheGreatFateemah/8-WEEK-SQL-CHALLENGE---CASE-STUDY-1/blob/main/Danny's%20Diner%20-%20ERD.png)

## Solution To The Questions

### 1. What is the total amount each customer spent at the restaurant?

```sql
SELECT customer_id, SUM(price) AS total_amount_spent
FROM sales AS s
JOIN menu AS m
ON s.product_id = m.product_id
GROUP BY customer_id
ORDER BY total_amount_spent DESC;
```

#### Result

| customer_id | total_amount_spent |
|-------------| ------------------ |
| A | 76
| B | 74
| C | 36

Customer A spent $76, B spent $74 while C spent $36.
***

### 2. How many days has each customer visited the restaurant?

```sql
SELECT customer_id, COUNT(DISTINCT order_date) AS days_visited
FROM sales
GROUP BY customer_id;
```

#### Result

| customer_id | days_visited |
|-------------| ------------ |
| A | 4
| B | 6
| C | 2

Customer A visited the restaurant for four days while B visited for 6 days and C visited for 2 days.
***

### 3. What was the first item from the menu purchased by each customer?

```sql
SELECT TOP 4 customer_id, 
     order_date, 
    product_name 
FROM sales AS S
JOIN menu AS m
ON s.product_id = m.product_id
ORDER BY order_date;
```

#### Result

| customer_id | order_date | product_name |
|-------------| ---------- | -------------|
| A | 2021-01-01 | sushi
| A | 2021-01-01 | curry
| B | 2021-01-01 | curry
| C | 2021-01-01 | ramen

Customer A purchased sushi and curry first, B purchased curry first and C purchased ramen first.
***

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

```sql
SELECT TOP 1 product_name, COUNT(product_name) AS count
FROM sales
INNER JOIN menu
ON sales.product_id = menu.product_id
GROUP BY product_name
ORDER BY count DESC;
```

#### Result

| product_name | count |
|------------- | ------ |
| ramen | 8

ramen is the most purchased item on the menu and it was purchased 8 times by all customers.
***

### 5. Which item was the most popular for each customer?

```sql
SELECT TOP 3 customer_id, product_name, COUNT(product_name) AS count
FROM sales
JOIN menu
ON sales.product_id = menu.product_id
GROUP BY product_name, customer_id
ORDER BY count DESC, customer_id DESC;
```

#### Result

| customer_id | product_name | count |
|-------------| ------------ | ------|
| C | ramen | 3
| A | ramen | 3
| B | curry | 2

ramen was the most popular for customer C and A while curry was the most popular for customer B.
***

### 6. Which item was purchased first by the customer after they became a member?

```sql
SELECT s.customer_id, product_name, order_date
FROM sales AS s
JOIN menu AS m
ON s.product_id = m.product_id
JOIN members AS mem
ON s.order_date > mem.join_date
AND s.customer_id = mem.customer_id
ORDER BY order_date;
```

#### Result

| customer_id | product_name | order_date |
|-------------| ------------ | -----------|
| A | ramen | 2021-01-10
| A | ramen | 2021-01-11
| A | ramen | 2021-01-11
| B | sushi | 2021-01-11
| B | ramen | 2021-01-16
| B | ramen | 2021-02-01

Customer A purchased ramen first after they became member and B purchased sushi first.
***

### 7. Which item was purchased just before the customer became a member?

```sql
SELECT s.customer_id, m2.product_name, order_date, m.join_date
FROM sales AS s
INNER JOIN members AS m
ON s.customer_id = m.customer_id
INNER JOIN menu AS m2
ON s.product_id = m2.product_id
WHERE order_date < join_date;
```

#### Result

| customer_id | product_name | order_date | join_date |
|-------------| ------------ | -----------| ----------|
| A | sushi | 2021-01-01 | 2021-01-07
| A | curry | 2021-01-01 | 2021-01-07
| B | curry | 2021-01-01 | 2021-01-09
| B | curry | 2021-01-02 | 2021-01-09
| B | sushi | 2021-01-04 | 2021-01-09

Customer A and B both purchased sushi and curry before they became a member.
***

### 8. What is the total items and amount spent for each member before they became a member?

```sql
SELECT s.customer_id, COUNT(product_name) AS total_items, SUM(price) AS total_amount_spent
FROM sales AS s
INNER JOIN members AS m
ON s.customer_id = m.customer_id
INNER JOIN menu AS m2
ON s.product_id = m2.product_id
WHERE order_date < join_date
GROUP BY s.customer_id;
```

#### Result

| customer_id | total_items | total_amount_spent |
|-------------| ----------- | -------------------|
| A | 2 | 25
| B | 3 | 40

Customer A purchased 2 items and spent $25 in total while B purchased 3 items and spent $40 in total before they became members.
***

### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

##### Had the customer joined the loyalty program before making the purchases, total points that each customer would have accrued

```sql
SELECT customer_id,
       SUM(CASE
               WHEN product_name = 'sushi' THEN price*20
               ELSE price*10
           END) AS customer_points
FROM menu
INNER JOIN sales  ON menu.product_id = sales.product_id
GROUP BY customer_id
ORDER BY customer_id;
```

#### Result

| customer_id | customer_points |
|-------------| --------------- |
| A | 860
| B | 940
| C | 360

Customer A had 860 points, B 940 points, and C 360 points before acquiring membership.

##### Total points that each customer has accrued after taking a membership

```sql
SELECT s.customer_id,
       SUM(CASE
               WHEN product_name = 'sushi' THEN price*20
               ELSE price*10
           END) AS customer_points
FROM menu AS m
INNER JOIN sales AS s ON m.product_id = s.product_id
INNER JOIN members AS mem ON mem.customer_id = s.customer_id
WHERE order_date >= join_date
GROUP BY s.customer_id
ORDER BY s.customer_id;
```

#### Result

| customer_id | customer_points |
|-------------| --------------- |
| A | 510
| B | 440

Customer A had 510 points and B would have 440 points after acquiring membership.
***

### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customers A and B have at the end of January?

```sql
SELECT main.customer_id, ISNULL(Total_points, 0) + ISNULL(Total_points2, 0) AS Total_points_combined
FROM (
  SELECT sales.customer_id, SUM(price * 20) AS Total_points
  FROM sales
  JOIN menu ON sales.product_id = menu.product_id
  JOIN members ON sales.customer_id = members.customer_id
  WHERE order_date BETWEEN '2021-01-07' AND '2021-01-14' 
    OR order_date BETWEEN '2021-01-09' AND '2021-01-16'
  GROUP BY sales.customer_id
) main
LEFT JOIN (
  SELECT sales.customer_id, SUM(price * 10) AS Total_points2
  FROM sales
  JOIN menu ON sales.product_id = menu.product_id
  JOIN members ON sales.customer_id = members.customer_id
  WHERE order_date BETWEEN '2021-01-14' AND '2021-01-31'
    OR order_date BETWEEN '2021-01-16' AND '2021-01-31'
  GROUP BY sales.customer_id
) sub
ON main.customer_id = sub.customer_id
ORDER BY main.customer_id;
```

#### Result

| customer_id | Total_points_combined |
|-------------| --------------------- |
| A | 1020
| B | 560

Customer A has 1020 points while Customer B has 560 at the end of January.
***






