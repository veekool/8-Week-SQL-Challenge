# Case Study #1 - Danny's Diner

View original link [here](https://8weeksqlchallenge.com/case-study-1/) by Danny Ma.

<p align="center">
<img src="https://8weeksqlchallenge.com/images/case-study-designs/1.png" width=50% height=50%>


<details><summary>Creating databases and Tables:</summary>

```sql
CREATE SCHEMA dannys_diner;
SET search_path = dannys_diner;

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
  

CREATE TABLE members (
  "customer_id" VARCHAR(1),
  "join_date" DATE
);

INSERT INTO members
  ("customer_id", "join_date")
VALUES
  ('A', '2021-01-07'),
  ('B', '2021-01-09');

```
</details>

SQL interactive playground >> [View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)

---

### Question 1: What is the total amount each customer spent at the restaurant?
<details><summary>Solution:</summary>
  
```sql
WITH sales_count AS (
	SELECT s.customer_id, m.product_name, COUNT(order_date) AS order_counts, SUM(m.price) AS total_amount
	FROM sales s
	INNER JOIN menu m ON s.product_id = m.product_id
	GROUP BY s.customer_id, m.product_name
	ORDER BY 1
)

SELECT customer_id, SUM(total_amount)
FROM sales_count
GROUP BY customer_id
```

</details>

<details><summary>Output:</summary>

  ![image](https://github.com/veekool/8-Week-SQL-Challenge/assets/114795923/6573f2c2-d040-4cd4-b1f2-6e0b1caa49c2)

</details>

### Question 2: How many days has each customer visited the restaurant?
<details><summary>Solution:</summary>
  
```sql
SELECT customer_id, COUNT(DISTINCT(order_date)) AS days_visited
FROM sales
GROUP BY customer_id
ORDER BY 1
```

</details>

<details><summary>Output:</summary>

  ![image](https://github.com/veekool/8-Week-SQL-Challenge/assets/114795923/c95e51ef-7b1a-4394-9c36-0e3bfa0bd5d6)

</details>


### Question 3: What was the first item from the menu purchased by each customer?
<details><summary>Solution:</summary>
  
```sql
WITH ranked_sales AS (
	SELECT customer_id, s.product_id, m.product_name, ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY order_date) AS customer_rank
	FROM sales s
	INNER JOIN menu m ON s.product_id = m.product_id
)

SELECT customer_id, product_id, product_name
FROM ranked_sales
WHERE customer_rank = 1
```

</details>

<details><summary>Output:</summary>

  ![image](https://github.com/veekool/8-Week-SQL-Challenge/assets/114795923/a3d4964a-6f8a-4a81-9b7b-f9c00155b51a)

</details>

### Question 4: What is the most purchased item on the menu and how many times was it purchased by all customers?
<details><summary>Solution:</summary>
  
```sql
SELECT product_name, COUNT(order_date)
FROM sales s
INNER JOIN menu m ON s.product_id = m.product_id
GROUP BY 1
ORDER BY 2 DESC
LIMIT 1
```

</details>

<details><summary>Output:</summary>

  ![image](https://github.com/veekool/8-Week-SQL-Challenge/assets/114795923/02d15f17-21fe-49f4-b7b2-71a63fa9f028)

</details>

### Question 5: Which item was the most popular for each customer?
<details><summary>Solution:</summary>
  
```sql
WITH popular_rank AS (
	SELECT customer_id, product_name, COUNT(order_date) as purchase_counts, DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY COUNT(order_date) DESC) AS rank_
	FROM sales s
	INNER JOIN menu m ON s.product_id = m.product_id
	GROUP BY 1,2
)

SELECT customer_id, product_name
FROM popular_rank 
WHERE rank_ = 1
```

</details>

<details><summary>Output:</summary>

  ![image](https://github.com/veekool/8-Week-SQL-Challenge/assets/114795923/ac50cefa-61b0-4084-a507-0d1fa8891cb4)

</details>

### Question 6: Which item was purchased first by the customer after they became a member?
<details><summary>Solution:</summary>
  
```sql
WITH purchase_rank_after_member AS (
	SELECT mb.customer_id, m.product_name, s.order_date, ROW_NUMBER() OVER (PARTITION BY s.customer_id ORDER BY s.order_date ASC) AS rank_
	FROM members mb 
	INNER JOIN sales s ON mb.customer_id = s.customer_id
	INNER JOIN menu m ON s.product_id = m.product_id
	WHERE s.order_date > mb.join_date
)

SELECT customer_id, product_name, order_date
FROM purchase_rank_after_member
WHERE rank_ = 1
```

</details>

<details><summary>Output:</summary>

  ![image](https://github.com/veekool/8-Week-SQL-Challenge/assets/114795923/62b7ccd2-4f20-45b2-9be6-ed61281d6baf)

</details>

### Question 7: Which item was purchased just before the customer became a member?
<details><summary>Solution:</summary>
  
```sql
WITH purchase_rank_before_member AS (
	SELECT mb.customer_id, m.product_name, s.order_date, ROW_NUMBER() OVER (PARTITION BY s.customer_id ORDER BY s.order_date DESC) AS rank_
	FROM members mb 
	INNER JOIN sales s ON mb.customer_id = s.customer_id
	INNER JOIN menu m ON s.product_id = m.product_id
	WHERE s.order_date < mb.join_date
)

SELECT customer_id, product_name, order_date
FROM purchase_rank_before_member
WHERE rank_ = 1
```

</details>

<details><summary>Output:</summary>

  ![image](https://github.com/veekool/8-Week-SQL-Challenge/assets/114795923/ca740525-33fe-49cc-a897-38d862aea364)

</details>

### Question 8: What is the total items and amount spent for each member before they became a member?
<details><summary>Solution:</summary>
  
```sql
WITH amount_spend_before_member AS (
	SELECT s.customer_id, m.product_name, COUNT(order_date) AS order_counts, SUM(m.price) AS total_amount
	FROM members mb 
	INNER JOIN sales s ON mb.customer_id = s.customer_id
	INNER JOIN menu m ON s.product_id = m.product_id
	WHERE s.order_date < mb.join_date
	GROUP BY 1,2
	)

SELECT customer_id AS customer_before_member, SUM(order_counts) AS total_items, SUM(total_amount) AS total_spent
FROM amount_spend_before_member
GROUP BY 1
ORDER BY 1
```

</details>

<details><summary>Output:</summary>

  ![image](https://github.com/veekool/8-Week-SQL-Challenge/assets/114795923/919a7f9d-23c7-4e53-9f40-adc3244e7453)

</details>

### Question 9: If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
<details><summary>Solution:</summary>
  
```sql
WITH sales_count_points AS (
    SELECT s.customer_id, m.product_name, COUNT(s.order_date) AS order_counts, SUM(m.price) AS total_amount,
		SUM(CASE 
			WHEN m.product_name = 'sushi' THEN (m.price * 2 * 10)
            ELSE (m.price * 10)
            END) AS total_points
    FROM sales s
    INNER JOIN menu m ON s.product_id = m.product_id
    GROUP BY s.customer_id, m.product_name
    ORDER BY s.customer_id
)

SELECT customer_id, SUM(total_amount) AS total_spent, SUM(total_points) as total_points
FROM sales_count_points
GROUP BY customer_id;
```

</details>

<details><summary>Output:</summary>

  ![image](https://github.com/veekool/8-Week-SQL-Challenge/assets/114795923/b77b74a5-1b43-4573-9144-d6c29135ae67)

</details>


### Question 10: In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
<details><summary>Solution:</summary>
  
```sql
WITH purchase_rank_after_member AS (
	SELECT mb.customer_id, m.product_name, s.order_date, COUNT(s.order_date) AS order_counts, SUM(m.price) AS total_amount,
		SUM(CASE 
			WHEN s.order_date >= mb.join_date AND s.order_date < mb.join_date + INTERVAL '1 week' THEN (m.price * 2 * 10)
            ELSE (m.price * 10)
            END) AS total_points
	FROM members mb 
	INNER JOIN sales s ON mb.customer_id = s.customer_id
	INNER JOIN menu m ON s.product_id = m.product_id
	WHERE s.order_date BETWEEN '2021-01-01' AND '2021-01-31' AND s.order_date >= mb.join_date
	GROUP BY 1,2,3
	ORDER BY 1,3
)

SELECT customer_id, SUM(total_points) as total_points
FROM purchase_rank_after_member
GROUP BY 1
```

</details>

<details><summary>Output:</summary>

  ![image](https://github.com/veekool/8-Week-SQL-Challenge/assets/114795923/e4728e08-029f-4b3d-86ef-8ce9f0ae1e77)

</details>


### BONUS QUESTION -> Question 11: Join all the things
##### The following questions are related creating basic data tables that Danny and his team can use to quickly derive insights without needing to join the underlying tables using SQL.
##### Recreate the following table output using the available data:

<details open><summary>Output:</summary>

| customer_id | order_date | product_name | price | member |
| ----------- | ---------- | -------------| ----- | ------ |
| A           | 2021-01-01 | sushi        | 10    | N      |
| A           | 2021-01-01 | curry        | 15    | N      |
| A           | 2021-01-07 | curry        | 15    | Y      |
| A           | 2021-01-10 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| B           | 2021-01-01 | curry        | 15    | N      |
| B           | 2021-01-02 | curry        | 15    | N      |
| B           | 2021-01-04 | sushi        | 10    | N      |
| B           | 2021-01-11 | sushi        | 10    | Y      |
| B           | 2021-01-16 | ramen        | 12    | Y      |
| B           | 2021-02-01 | ramen        | 12    | Y      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-07 | ramen        | 12    | N      |

</details>

<details><summary>Solution:</summary>
  
```sql
SELECT s.customer_id, s.order_date, m.product_name, m.price, 
CASE
	WHEN order_date >= join_date THEN 'Y'
	WHEN order_date < join_date THEN 'N'
	ELSE 'N'
	END AS members
FROM sales s
INNER JOIN menu m ON s.product_id = m.product_id
LEFT JOIN members mb ON s.customer_id = mb.customer_id
ORDER BY 1,2
```

</details>

### BONUS QUESTION -> Question 12: Rank all things
#### Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.

<details open><summary>Output:</summary>

| customer_id | order_date | product_name | price | member | ranking | 
| ----------- | ---------- | -------------| ----- | ------ |-------- |
| A           | 2021-01-01 | sushi        | 10    | N      | NULL
| A           | 2021-01-01 | curry        | 15    | N      | NULL
| A           | 2021-01-07 | curry        | 15    | Y      | 1
| A           | 2021-01-10 | ramen        | 12    | Y      | 2
| A           | 2021-01-11 | ramen        | 12    | Y      | 3
| A           | 2021-01-11 | ramen        | 12    | Y      | 3
| B           | 2021-01-01 | curry        | 15    | N      | NULL
| B           | 2021-01-02 | curry        | 15    | N      | NULL
| B           | 2021-01-04 | sushi        | 10    | N      | NULL
| B           | 2021-01-11 | sushi        | 10    | Y      | 1
| B           | 2021-01-16 | ramen        | 12    | Y      | 2
| B           | 2021-02-01 | ramen        | 12    | Y      | 3
| C           | 2021-01-01 | ramen        | 12    | N      | NULL
| C           | 2021-01-01 | ramen        | 12    | N      | NULL
| C           | 2021-01-07 | ramen        | 12    | N      | NULL

</details>

<details><summary>Solution:</summary>
  
```sql
with CTE1 AS (
SELECT s.customer_id, s.order_date, m.product_name, m.price, 
CASE
	WHEN order_date >= join_date THEN 'Y'
	WHEN order_date < join_date THEN 'N'
	ELSE 'N'
	END AS members
FROM sales s
INNER JOIN menu m ON s.product_id = m.product_id
LEFT JOIN members mb ON s.customer_id = mb.customer_id
ORDER BY 1,2
)

SELECT *, 
CASE 
	WHEN members = 'N' THEN NULL
	ELSE DENSE_RANK() OVER (PARTITION BY customer_id, members ORDER BY order_date)
	END AS ranking
FROM CTE1

```

</details>

