# Case Study #1 - Danny's Diner

![logo](https://github.com/user-attachments/assets/a4fdf205-cc93-446a-b0c7-8be9170ceeb1)

You can find all the details here - 

### Table of Contents:
- Problem Statement
- ER Diagram
- Questions & Solutions

### Problem Statement
---
Danny seriously loves Japanese food so in the beginning of 2021, he opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen. Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation.
He wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.
Danny has shared with you 3 key datasets for this case study:
- sales
- menu
- members

### Entity Relationship Diagram
---
![ER](https://github.com/user-attachments/assets/f8b6d3cb-a5ed-4a4f-9510-d0323f3eafef)

### Questions & Solutions
---
#### Question 1. What is the total amount each customer spent at the restaurant?

#### Solution:

```sql
SELECT
    customer_id,
    SUM(price) AS Total_spent
FROM
    sales
JOIN
    menu ON sales.product_id = menu.product_id
GROUP BY
    customer_id;
```

#### Output:
![S1](https://github.com/user-attachments/assets/859ed1c5-627c-4bac-9e77-e25b62cf307c)

- Customer A spent $76.
- Customer B spent $74.
- Customer C spent $36.

---
#### Question 2. How many days has each customer visited the restaurant?

#### Solution:

```sql
SELECT 
  customer_id, 
  COUNT(DISTINCT order_date) AS visit_days
FROM 
  sales 
GROUP BY 
  customer_id;
```
#### Output:

![S2](https://github.com/user-attachments/assets/30704b17-4d1b-474e-b863-f6b797766504)

- Customer A has visited the restaurant 4 days.
- Customer B has visited the restaurant 6 days.
- Customer C has visited the restaurant 2 days.

---

#### Question 3. What was the first item from the menu purchased by each customer?

#### Solution:

```sql
WITH first_orders AS
(SELECT 
    customer_id,
    product_id,
    order_date,
    RANK() OVER(PARTITION BY customer_id ORDER BY order_date) AS ranks
FROM
    sales
GROUP BY 
    customer_id,
    product_id,
    order_date)

SELECT
    orders.customer_id, 
    menu.product_name
FROM 
    first_orders AS orders
JOIN
    menu ON orders.product_id = menu.product_id
WHERE 
    orders.ranks = 1;
```
#### Output:

![S3](https://github.com/user-attachments/assets/ee6d7805-13e4-4b24-84bb-9e50f19ee318)

- Customer A placed an order for both curry and sushi simultaneously as his/her first order.
- Customer B's first order is curry.
- Customer C's first order is ramen.

---

#### Question 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

#### Solution:

```sql
SELECT 
  menu.product_name, COUNT(sales.product_id) AS total_times_ordered
FROM 
  sales
JOIN
  menu ON sales.product_id = menu.product_id
GROUP BY 
  menu.product_name
ORDER BY 
  total_times_ordered DESC
LIMIT 1;
```
#### Output:

![S4](https://github.com/user-attachments/assets/c97d23a9-500b-4e44-b12b-1b2edb64f4f9)

- Ramen is the item that is purchased the most by all the customers and is purchased 8 times in total.
---

#### Question 5. Which item was the most popular for each customer?

#### Solution:

```sql
WITH ranked_items AS 
  (SELECT customer_id, 
      product_id, 
      COUNT(product_id) AS product_count, 
      RANK() OVER(PARTITION BY customer_id ORDER BY COUNT(product_id) DESC) AS ranks
  FROM
      sales
  GROUP BY
      customer_id, product_id)

SELECT
  item.customer_id,
  menu.product_name,
  item.product_count
FROM 
  ranked_items AS item
JOIN 
  menu ON item.product_id = menu.product_id 
WHERE
  item.ranks = 1
ORDER BY
  item.customer_id;
```
#### Output:

![S5](https://github.com/user-attachments/assets/a2b78e4f-c9c0-4091-8953-9fb5b0ffcdf3)

- For both the customers A and C, ramen is the most popular item.
- For customer B, all the three items are equally popular which are curry, sushi and ramen.
---

#### Question 6. Which item was purchased first by the customer after they became a member?

#### Solution:

```sql
WITH ranked_products AS 
  (SELECT 
      sales.customer_id, 
      menu.product_name, 
      DENSE_RANK() OVER(PARTITION BY sales.customer_id ORDER BY sales.order_date) as ranks
  FROM 
    sales JOIN members JOIN menu 
  WHERE 
      sales.order_date >= members.join_date AND
      sales.customer_id = members.customer_id AND
      sales.product_id = menu.product_id)

SELECT 
  products.customer_id, 
  products.product_name
FROM 
  ranked_products AS products	
WHERE 
  products.ranks = 1;
```
#### Output:

![S6](https://github.com/user-attachments/assets/9e9d0545-6eb3-4e59-939a-679db3b1150e)

- After becoming a member, A's first order was curry.
- After becoming a member, B's first order was sushi.
---

#### Question 7. Which item was purchased just before the customer became a member?

#### Solution:

```sql
WITH ranked_products AS 
  (SELECT 
    sales.customer_id, 
    menu.product_name, 
    DENSE_RANK() OVER(PARTITION BY sales.customer_id ORDER BY sales.order_date DESC) as ranks
  FROM 
    sales JOIN members JOIN menu 
  WHERE 
    sales.order_date < members.join_date AND
    sales.customer_id = members.customer_id AND
    sales.product_id = menu.product_id)

SELECT 
  products.customer_id, 
  products.product_name
FROM 
  ranked_products AS products	
WHERE 
  products.ranks = 1;
```
#### Output:

![S7](https://github.com/user-attachments/assets/44763cc0-9654-403a-b238-9dfeeb79354d)

- Just before becoming a member, A purchased 2 items sushi and curry.
- Just before becoming a member, B purchased sushi.
---

#### Question 8. What is the total items and amount spent for each member before they became a member?

#### Solution:

```sql
SELECT 
  sales.customer_id, 
  COUNT(sales.product_id) as total_items, 
  SUM(menu.price) as amount_spent
FROM 
  sales JOIN menu JOIN members  
WHERE 
  sales.order_date < members.join_date AND
  sales.product_id = menu.product_id AND
  sales.customer_id = members.customer_id
GROUP BY 
  sales.customer_id
ORDER BY 
  sales.customer_id;
```
#### Output:

![S8](https://github.com/user-attachments/assets/1b1d44ea-2330-40ec-8830-74ddf2f6a2b7)

- Before becoming member, Customer A spent $25 on 2 items.
- Before becoming member, Customer B spent $40 on 3 items.
---

#### Question 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

#### Solution:

```sql
SELECT  
  sales.customer_id, 
  SUM((menu.price) * (CASE WHEN menu.product_name = 'sushi' THEN 20 ELSE 10 END)) AS total_points
FROM 
  sales
JOIN 
  menu ON sales.product_id = menu.product_id
GROUP BY 
  sales.customer_id;
```
#### Output:

![S9](https://github.com/user-attachments/assets/9ce844b8-4767-4748-a702-1a309d82969e)

- Total points for Customer A = $860.
- Total points for Customer B = $940.
- Total points for Customer C = $360.
---

#### Question 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

#### Solution:

```sql
SELECT
    sales.customer_id,  
    SUM(
        CASE 
          WHEN menu.product_name = 'sushi' THEN 20 * menu.price 
          WHEN datediff(sales.order_date, members.join_date) BETWEEN 0 AND 6 THEN 20 * menu.price
          ELSE 10 * menu.price
        END) AS total_points
FROM 
    sales
JOIN 
    menu ON sales.product_id = menu.product_id
JOIN 
    members ON sales.customer_id = members.customer_id
WHERE 
    sales.order_date <= '2021-01-31'
GROUP BY 
    sales.customer_id
ORDER BY
    sales.customer_id;
```
#### Output:

![S10](https://github.com/user-attachments/assets/783b80c2-b363-476e-8a26-fc4d5a37a042)

- Total points for Customer A = 1,370.
- Total points for Customer B = 820.
---
