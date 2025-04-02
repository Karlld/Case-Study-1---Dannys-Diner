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

4. What is the most purchased item on the menu and how many times was it purchased by all customers?

```sql
SELECT m.product_name,
       COUNT(s.product_id) AS amount_sold
     FROM sales s
     JOIN menu as m ON s.product_id = m.product_id
     GROUP BY s.product_id,
              m.product_name
     ORDER BY amount_sold desc
     LIMIT 1;

```

| product_name | amount_sold |
|--------------|-------------|
|   ramen  |	8   |

5. Which item was the most popular for each customer?

```sql

WITH rank AS (SELECT S.customer_id, 
                     M.product_name, 
	             DENSE_RANK() OVER (PARTITION BY S.Customer_ID ORDER BY COUNT(S.product_id)) AS rank
              FROM Menu m
              JOIN Sales ON m.product_id = s.product_id
             GROUP BY s.customer_id,
                      m.product_name,
                      s.product_id)
     SELECT customer_id,
            product_name
  FROM rank 
  WHERE rank = 1
```

| customer_id |	product_name |
|-------------|--------------|
| A  |	sushi  |
| B  |	curry  |
| B |	sushi  |
| B  |	ramen  |
| C  |	ramen  |

6. Which item was purchased first by the customer after they became a member?

```sql
WITH rank AS (SELECT s.customer_id,
                     m.product_name,
	             s.order_date,
	             DENSE_RANK() OVER (PARTITION by s.customer_id ORDER BY s.order_date) AS rank
	       FROM sales s 
	   JOIN menu m ON s.product_id = m.product_id
	   JOIN members c ON s.customer_id = c.customer_id
	   WHERE s.order_date >= c.join_date)
	   
SELECT customer_id,
       product_name
 FROM rank
 WHERE rank= 1
```

|customer_id |	product_name |
|------------|---------------|
|A |	curry  |
|B |	sushi  |

7. Which item was purchased just before the customer became a member?

```sql

WITH rank AS (SELECT s.customer_id,
                     m.product_name,
	             s.order_date,
	             DENSE_RANK() OVER (PARTITION BY s.customer_id ORDER BY s.order_date) AS rank
	       FROM sales s 
	       JOIN menu m ON s.product_id = m.product_id
	       JOIN members c ON s.customer_id = c.customer_id
	       WHERE s.order_date < c.join_date)

SELECT customer_id,
        product_name
FROM rank 
WHERE rank= 1

```

| customer_id  | product_name |
|--------------|--------------|
| A  |	 sushi  |
| A  |	curry  |
| B  |  curry  |
	  
8. What is the total items and amount spent for each member before they became a member?

```sql
SELECT s.customer_id,
       COUNT(s.product_id) AS items,
       SUM(p.price) AS total_sales
       FROM sales s
	   JOIN menu p ON s.product_id = p.product_id
           JOIN members c ON s.customer_id = c.customer_id
         WHERE s.order_date < c.join_date
	  GROUP BY s.customer_id
```
	   
| customer_id  | items  |  total_sales  |
|--------------|--------|---------------|
| B  |	3    |   40  |
| A  |	2    |	25   |


9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```sql
   WITH points AS (SELECT *,
                   CASE WHEN p.product_id = 1 THEN p.price*20
	           ELSE p.price*10
	           END AS points
               FROM menu p)
		 
   SELECT s.customer_id,
          SUM(l.points) AS total_points
   FROM sales s
   JOIN points l ON s.product_id = l.product_id
   GROUP BY customer_id
```

| customer_id |	total_points |
|-------------|--------------|
| B |	940  |
| C |	360  |
 | A |	860  |
	  
	  
	  
10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

```sql

WITH points AS (SELECT s.customer_id,
                       s.order_date,
                       CASE WHEN s.order_date BETWEEN c.join_date AND (c.join_date+7) 
                       THEN m.price*20
                           ELSE CASE WHEN m.product_id = 1 THEN m.price*20
                           ELSE m.price*10
                           END
	               END AS points
                       FROM menu m
                       JOIN sales s ON s.product_id = m.product_id
                       JOIN members c ON c.customer_id = s.customer_id)
		 
 SELECT customer_id,
        SUM(points) AS total_points
        FROM points
        WHERE order_date BETWEEN '2021-01-01' AND '2021-01-31'
        GROUP BY customer_id
        ORDER BY customer_id 
```
	   
| customer_id |	 total_points |
|-------------|---------------|
| A  |	1370  |
| B  |	940   |




