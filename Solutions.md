**1.What is the total amount each customer spent at the restaurant?**

```sql

SELECT s.customer_id,
       sum(m.price) AS total_spent
          FROM sales s
          JOIN menu m ON s.product_id = m.product_id
GROUP BY customer_id
ORDER BY customer_id

```
 
| customer_id |	total_spent  |
|-------------|--------------|
| A |	76 |
| B |	74 |
| C |	36 |

**2. How many days has each customer visited the restaurant?**

```sql

SELECT customer_id,
       count(distinct(order_date))
          FROM sales
         GROUP BY customer_id
         ORDER BY customer_id

```

| customer_id |	count |
|-------------|-------|
| A |	4 |
| B |	6 |
| C |	2 |

**3. What was the first item from the menu purchased by each customer?**

```sql

WITH Rank AS (SELECT s.customer_id, 
                     m.product_name, 
	                   s.order_date,
	            DENSE_RANK() OVER (PARTITION BY S.Customer_ID ORDER BY s.order_date) AS rank
                 FROM Menu m
                JOIN Sales s ON m.product_id = s.product_id
                GROUP BY s.customer_id, m.product_name, s.order_date)
SELECT Customer_id,
       product_name
    FROM Rank
    WHERE rank = 1;

```

| customer_id |	product_name |
|-------------|--------------|
| A |	curry |
| A |	sushi |
| B |	curry |
| C | ramen |



