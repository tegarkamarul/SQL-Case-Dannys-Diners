# SQL-Case-Dannys-Diners
# STUDY CASE : DANNYS DINER
# This case from https://8weeksqlchallenge.com
## 1. What is the total amount each customer spent at the restaurant?
``` sql
SELECT
	customer_id,
    sum(price) AS total_spent
FROM
	dannys_diner.sales as s
JOIN
	dannys_diner.menu as m
ON
	s.product_id = m.product_id
GROUP BY 
	customer_id
ORDER BY
	customer_id;
```
## 2. How many days has each customer visited the restaurant?
``` sql
SELECT
	customer_id,
    COUNT (DISTINCT(order_date)) AS total_days
FROM
	dannys_diner.sales
GROUP BY 
	customer_id
ORDER BY
	customer_id;
```   
## 3. What was the first item from the menu purchased by each customer?
``` sql
WITH  purchase AS(
  SELECT
  	sales.customer_id,
  	sales.order_date,
  	menu.product_name,
  	ROW_NUMBER () OVER (PARTITION BY customer_id order by order_date) as orders
  FROM dannys_diner.sales
  JOIN dannys_diner.menu
  ON dannys_diner.sales.product_id = dannys_diner.menu.product_id
)
SELECT
	customer_id,
    product_name
FROM purchase
WHERE orders = 1;
```
## 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
``` sql 
SELECT
	product_name,
    count(*) as total_purchase
FROM 
	dannys_diner.sales as s
JOIN
	dannys_diner.menu as m
ON
	s.product_id = m.product_id
GROUP BY product_name
LIMIT 1;
```
## 5. Which item was the most popular for each customer?
``` sql
WITH  purchases AS(
  SELECT
  	sales.customer_id,
  	menu.product_name,
  	COUNT(*) as total_purchase,
  	DENSE_RANK () OVER (PARTITION BY sales.customer_id order by count(*) DESC) as orders
  FROM dannys_diner.sales
  JOIN dannys_diner.menu
  ON dannys_diner.sales.product_id = dannys_diner.menu.product_id
  GROUP BY customer_id, product_name
)
SELECT 
	customer_id,
    product_name,
    total_purchase
FROM 
	purchases
WHERE
	orders = 1;
```

## 6. Which item was purchased first by the customer after they became a member?
``` sql
WITH after_member AS(
	SELECT
  		sales.customer_id,
  		sales.order_date,
  		menu.product_name,
    	ROW_NUMBER () OVER (PARTITION BY sales.customer_id order by sales.order_date) as orders
	FROM 
  		dannys_diner.sales 
  	JOIN 
  		dannys_diner.menu 
  	ON 
  		dannys_diner.sales.product_id = dannys_diner.menu.product_id
  	JOIN 
  		dannys_diner.members
  	ON 
  		dannys_diner.sales.customer_id = dannys_diner.members.customer_id
  	WHERE 
  		dannys_diner.sales.order_date >= dannys_diner.members.join_date
)
SELECT
	customer_id,
    product_name
FROM 
	after_member
WHERE orders = 1;
```

## 7. Which item was purchased just before the customer became a member?
``` sql
WITH before_member AS(
	SELECT
  		sales.customer_id,
  		sales.order_date,
  		menu.product_name,
    	ROW_NUMBER () OVER (PARTITION BY sales.customer_id order by sales.order_date DESC) as orders
	FROM 
  		dannys_diner.sales 
  	JOIN 
  		dannys_diner.menu 
  	ON 
  		dannys_diner.sales.product_id = dannys_diner.menu.product_id
  	JOIN 
  		dannys_diner.members
  	ON 
  		dannys_diner.sales.customer_id = dannys_diner.members.customer_id
  	WHERE 
  		dannys_diner.sales.order_date < dannys_diner.members.join_date
)
SELECT
	customer_id,
    product_name
FROM 
	before_member
WHERE orders = 1;
```

## 8. What is the total items and amount spent for each member before they became a member?
``` sql
SELECT
  	dannys_diner.sales.customer_id,
  	COUNT(product_name) as total_items,
    SUM(price) as total_spent
FROM 
  	dannys_diner.sales 
JOIN 
  	dannys_diner.menu 
ON 
  	dannys_diner.sales.product_id = dannys_diner.menu.product_id
JOIN 
  	dannys_diner.members
ON 
  	dannys_diner.sales.customer_id = dannys_diner.members.customer_id
WHERE 
  	dannys_diner.sales.order_date < dannys_diner.members.join_date
GROUP BY dannys_diner.sales.customer_id
ORDER BY dannys_diner.sales.customer_id;
```

## 9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?       	
``` sql
SELECT
	customer_id,
    SUM(CASE
    	WHEN product_name = 'sushi' THEN price * 10 * 2
        ELSE price * 10
   	END) AS points
FROM 
  	dannys_diner.sales 
JOIN 
  	dannys_diner.menu 
ON 
  	dannys_diner.sales.product_id = dannys_diner.menu.product_id
GROUP BY dannys_diner.sales.customer_id
ORDER BY dannys_diner.sales.customer_id;
```

## 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
``` sql
SELECT
  s.customer_id,
  SUM(CASE
    WHEN m.product_name = 'sushi' THEN price * 10 * 2
    WHEN s.order_date BETWEEN me.join_date AND me.join_date + INTERVAL '6 day' THEN price * 10 * 2
    ELSE price * 10
  END) AS points
FROM
  dannys_diner.sales s
JOIN
  dannys_diner.menu m
ON
  s.product_id = m.product_id
LEFT JOIN
  dannys_diner.members me
ON
  s.customer_id = me.customer_id
WHERE
  order_date <= '2021-01-31'
GROUP BY
  s.customer_id
ORDER BY
  s.customer_id;
```

## BONUS QUESTION

## Join All the Things
``` sql
SELECT
	dannys_diner.sales.customer_id,
    dannys_diner.sales.order_date,
    dannys_diner.menu.product_name,
    dannys_diner.menu.price,
    CASE
      WHEN dannys_diner.sales.order_date >= dannys_diner.members.join_date  then 'Y'
      ELSE 'N'
      END AS member
FROM 
  	dannys_diner.sales 
JOIN 
  	dannys_diner.menu 
ON 
  		dannys_diner.sales.product_id = dannys_diner.menu.product_id
LEFT JOIN 
  	dannys_diner.members
ON 
  		dannys_diner.sales.customer_id = dannys_diner.members.customer_id;
```

## Rank All the Things
``` sql
WITH big_table AS(
SELECT
    dannys_diner.sales.customer_id,
    dannys_diner.sales.order_date,
    dannys_diner.menu.product_name,
    dannys_diner.menu.price,
    CASE
      WHEN dannys_diner.sales.order_date >= dannys_diner.members.join_date  then 'Y'
      ELSE 'N'
      END AS member
FROM 
  	dannys_diner.sales 
JOIN 
  	dannys_diner.menu 
ON 
  		dannys_diner.sales.product_id = dannys_diner.menu.product_id
LEFT JOIN 
  	dannys_diner.members
ON 
  		dannys_diner.sales.customer_id = dannys_diner.members.customer_id
)
SELECT
	*,
	CASE
    	WHEN member ='Y' THEN DENSE_RANK () OVER (PARTITION BY customer_id, member order by order_date)
	ELSE NULL
    END ASranking
FROM
	big_table;
```
