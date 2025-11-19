# B. Runner and Customer Experience
### 1. How many runners signed up for each 1 week period? (i.e. week starts ```2021-01-01```)
```sql
SELECT
	TO_CHAR(registration_date, 'W') as week,
	COUNT(runner_id) as number_of_signups
FROM
	runners
GROUP BY
	1
ORDER BY
	2 DESC
```
| week | number_of_signups |
| ---- | ----------------- |
| 1    | 2                 |
| 2    | 1                 |
| 3    | 1                 |


### 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
```sql
SELECT
	r.runner_id,
	ROUND(AVG(
				EXTRACT (EPOCH from (r.pickup_time - c.order_time))
			) 
		/ 60.0, 2) as average_pickup_time_min
FROM
	stg_runner_orders r
JOIN
	stg_customer_orders c ON
	c.order_id = r.order_id
WHERE
	r.pickup_time IS NOT null
GROUP BY
	1
ORDER BY
	2
```
| runner_id | average_pickup_time_min |
| --------- | ----------------------- |
|     3     |         10.47           |
|     1     |         15.68           |
|     2     |         23.72           |


### 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?
```sql
```


### 4. What was the average distance travelled for each customer?
```sql
SELECT
	c.customer_id,
	ROUND(CAST(AVG(r.distance) AS NUMERIC(10, 2)), 2) as average_distance
FROM
	stg_customer_orders c
JOIN
	stg_runner_orders r ON
	r.order_id = c.order_id
GROUP BY
	c.customer_id
ORDER BY
	2, 1
```
| customer_id | average_distance |
| ----------- | ---------------- |
|     104     |      10.00       |
|     102     |      16.73       |
|     101     |      20.00       |
|     103     |      23.40       |
|     105     |      25.00       |


### 5. What was the difference between the longest and shortest delivery times for all orders?
```sql
SELECT
	MAX(duration) - MIN(duration) as difference
FROM
	stg_runner_orders
```
| difference |
| ---------- |
|     30     |


### 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?