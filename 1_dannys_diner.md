### 1. What is the total amount each customer spent at the restaurant?

```sql
SELECT
    s.customer_id,
    SUM(m.price) as total_spent
FROM
    sales s
JOIN
    menu m on
    m.product_id = s.product_id
GROUP BY
    s.customer_id
ORDER BY
    total_spent DESC
```

| customer_id | total_spent |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |


### 2. How many days has each customer visited the restaurant?
```sql
WITH distinct_visits as (
    SELECT
        DISTINCT customer_id as customer_id,
        order_date
    FROM
        sales
)
SELECT
    customer_id,
    COUNT(order_date) as num_visits
FROM
    distinct_visits
GROUP BY
    customer_id
ORDER BY
    num_visits DESC
```

| customer_id | num_visits |
| ----------- | ---------- |
| B           | 6          |
| A           | 4          |
| C           | 2          |


### 3. What was the first item FROM the menu purchased by each customer?
```sql
WITH ranked as (
    SELECT
        customer_id,
        order_date,
        product_id,
        ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY order_date) as rn
    FROM
        sales
)
SELECT
    customer_id,
    product_id
FROM
    ranked
WHERE
    rn = 1
```

| customer_id | product_id |
| ----------- | ---------- |
| A           | 1          |
| B           | 2          |
| C           | 3          |


### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
```sql
SELECT
    product_id,
    COUNT(*) as num_times
FROM
    sales
GROUP BY
    product_id
ORDER BY
    num_times DESC
```

| product_id | num_times |
| ---------- | --------- |
| 3          | 8         |
| 2          | 4         |
| 1          | 3         |


### 5. Which item was the most popular for each customer?
```sql
WITH counted as (
    SELECT
        customer_id,
        product_id,
        COUNT(*) as num_times
    FROM
        sales
    GROUP BY
        customer_id,
        product_id
),
ranked as (
    SELECT
        customer_id,
        product_id,
        RANK() OVER(PARTITION BY customer_id ORDER BY num_times DESC) as rak
    FROM
        counted
)
SELECT
    customer_id,
    product_id
FROM
    ranked
WHERE
    rak = 1
```

| customer_id | product_id |
| ----------- | ---------- |
| A           | 3          |
| B           | 3          |
| B           | 1          |
| B           | 2          |
| C           | 3          |


### 6. Which item was purchased first by the customer after they became a member?
```sql
WITH joined as (
    SELECT
        s.customer_id as customer_id,
        s.product_id as product_id,
        s.order_date,
        m.join_date,
        RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date) as rak
    FROM
        sales s
    JOIN
        members m on
        m.customer_id = s.customer_id
    WHERE
    	s.order_date >= m.join_date
)
SELECT
    customer_id,
    product_id
FROM
    joined
WHERE
    rak = 1
```

| customer_id | product_id |
| ----------- | ---------- |
| A           | 2          |
| B           | 1          |


### 7. Which item was purchased just before the customer became a member?
```sql
WITH joined as (
    SELECT
        s.customer_id as customer_id,
        s.product_id as product_id,
        s.order_date,
        m.join_date,
        RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date) as rak
    FROM
        sales s
    JOIN
        members m on
        m.customer_id = s.customer_id
    WHERE
    	s.order_date < m.join_date
)
SELECT
    DISTINCT customer_id,
    product_id
FROM
	joined
```

| customer_id | product_id |
| ----------- | ---------- |
| A           | 1          |
| B           | 2          |
| A           | 2          |
| B           | 1          |


### 8. What is the total items and amount spent for each member before they became a member?
```sql
SELECT
    s.customer_id as customer_id,
    COUNT(s.product_id) as num_items,
    SUM(me.price) as amount_spent
FROM
    sales s
JOIN
    members m on
    m.customer_id = s.customer_id
JOIN
    menu me on
    me.product_id = s.product_id
WHERE
    s.order_date < m.join_date
GROUP BY
    s.customer_id
```

| customer_id | num_items | amount_spent |
| ----------- | --------- | ------------ |
| B           | 3         | 40           |
| A           | 2         | 25           |


### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
```sql
SELECT
    s.customer_id,
    SUM(CASE
                WHEN LOWER(me.product_name) = 'sushi' THEN me.price * 10 * 2 
                ELSE me.price * 10
            END
            ) as points
FROM
    sales s
JOIN
    menu me on
    me.product_id = s.product_id
GROUP BY
    s.customer_id
ORDER BY
    points DESC
```

| customer_id | points |
| ----------- | ------ |
| B           | 940    |
| A           | 860    |
| C           | 360    |


### 10. In the first week after a customer JOINs the program (including their JOIN date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the END of January?
```sql
SELECT
    s.customer_id as customer_id,
    SUM(
            CASE
                WHEN s.order_date >= m.join_date and s.order_date < m.join_date + 7 THEN me.price * 10 * 2
                WHEN LOWER(me.product_name) = 'sushi' THEN me.price * 10 * 2 
                ELSE me.price * 10
            END
        ) as points
FROM
    sales s
JOIN
    members m on
    m.customer_id = s.customer_id
JOIN
    menu me on
    me.product_id = s.product_id
WHERE
    s.order_date < make_date(2021, 02, 01)
GROUP BY
    s.customer_id
```

| customer_id | points |
| ----------- | ------ |
| A           | 1370   |
| B           | 820    |


## Bonus Questions
### 1. Join All The Things
```sql
SELECT
    s.customer_id,
    s.order_date,
    me.product_name,
    me.price,
    CASE
        WHEN s.order_date < m.join_date or m.join_date is null THEN 'N'
        ELSE 'Y'
    END as member
FROM
    sales s
JOIN
    menu me on
    me.product_id = s.product_id
LEFT JOIN
    members m on
    m.customer_id = s.customer_id
ORDER BY
    s.customer_id,
    s.order_date,
    me.product_name
```

| customer_id | order_date | product_name | price | member |
| ----------- | ---------- | ------------ | ----- | ------ |
| A           | 2021-01-01 | curry        | 15    | N      |
| A           | 2021-01-01 | sushi        | 10    | N      |
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


### 2. Rank All The Things
```sql
SELECT
    s.customer_id,
    s.order_date,
    me.product_name,
    me.price,
    CASE
            WHEN s.order_date < m.join_date or m.join_date is null THEN 'N'
            ELSE 'Y'
    END as member,
    CASE
        WHEN s.order_date >= m.JOIN_date THEN 
              RANK() OVER(
                            PARTITION BY s.customer_id 
                            ORDER BY 
                              CASE
                                  WHEN s.order_date >= m.join_date THEN s.order_date 
                              END
                          )
            ELSE null
    END as rank
FROM
    sales s
JOIN
    menu me on
    me.product_id = s.product_id
LEFT JOIN
    members m on
    m.customer_id = s.customer_id
ORDER BY
    s.customer_id,
    s.order_date,
    me.product_name
```

| customer_id | order_date | product_name | price | member | rank |
| ----------- | ---------- | ------------ | ----- | ------ | ---- |
| A           | 2021-01-01 | curry        | 15    | N      | null |
| A           | 2021-01-01 | sushi        | 10    | N      | null |
| A           | 2021-01-07 | curry        | 15    | Y      | 1    |
| A           | 2021-01-10 | ramen        | 12    | Y      | 2    |
| A           | 2021-01-11 | ramen        | 12    | Y      | 3    |
| A           | 2021-01-11 | ramen        | 12    | Y      | 3    |
| B           | 2021-01-01 | curry        | 15    | N      | null |
| B           | 2021-01-02 | curry        | 15    | N      | null |
| B           | 2021-01-04 | sushi        | 10    | N      | null |
| B           | 2021-01-11 | sushi        | 10    | Y      | 1    |
| B           | 2021-01-16 | ramen        | 12    | Y      | 2    |
| B           | 2021-02-01 | ramen        | 12    | Y      | 3    |
| C           | 2021-01-01 | ramen        | 12    | N      | null |
| C           | 2021-01-01 | ramen        | 12    | N      | null |
| C           | 2021-01-07 | ramen        | 12    | N      | null |