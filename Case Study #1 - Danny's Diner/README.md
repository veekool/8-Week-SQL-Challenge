# Case Study #1 - Danny's Diner

View original link [here](https://8weeksqlchallenge.com/case-study-1/) by Danny Ma.

![image](https://github.com/veekool/8-Week-SQL-Challenge/assets/114795923/4a64c7c0-d44c-4e99-946b-3b45ffdc4a34)


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

### Question 2: Find all employees who are not managers.

