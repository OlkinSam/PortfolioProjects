# <p align="center"> Case Study - Pizza Runner

## <p align="center"> Data Cleaning & Transformation

### Table: customer_orders

```sql
SELECT * FROM  customer_orders;
```

### Output:
|order_id|customer_id|pizza_id|exclusions|extras|order_time          |
|----------|-----------|--------|----------|------|--------------------|
|1         | 101       | 1      |          |      | 2020-01-01 18:05:02|
|2         | 101       | 1      |          |      | 2020-01-01 19:00:52|
|3         | 102       | 1      |          |      | 2020-01-02 23:51:23|
|3         | 102       | 2      |          |      | 2020-01-02 23:51:23|
|4         | 103       | 1      | 4        |      | 2020-01-04 13:23:46|
|4         | 103       | 1      | 4        |      | 2020-01-04 13:23:46|
|4         | 103       | 2      | 4        |      | 2020-01-04 13:23:46|
|5         | 104       | 1      | null     | 1    | 2020-01-08 21:00:29|
|6         | 101       | 2      | null     | null | 2020-01-08 21:03:13|
|7         | 105       | 2      | null     | 1    | 2020-01-08 21:20:29|
|8         | 102       | 1      | null     | null | 2020-01-09 23:54:33|
|9         | 103       | 1      | 4        | 1 5  | 2020-01-10 11:22:59|
|10        | 104       | 1      | null     | null | 2020-01-11 18:34:49|
|10        | 104       | 1      | 2 6      | 1 4  | 2020-01-11 18:34:49|



```sql
DROP TABLE IF EXISTS temp_customer_orders;
CREATE TEMPORARY TABLE temp_customer_orders AS 
	SELECT order_id,
		customer_id,
		pizza_id,
		CASE WHEN exclusions = 'null' or exclusions = '' THEN null
			ELSE exclusions END AS exclusions,
		CASE WHEN extras = 'null' or extras IS NULL or extras = '' THEN null
			ELSE extras END AS extras,
		order_time
	FROM customer_orders;

SELECT * FROM temp_customer_orders;
```

### Output:
|order_id|customer_id|pizza_id|exclusions|extras|order_time          |
|----------|-----------|--------|----------|------|--------------------|
|1         | 101       | 1      |          |      | 2020-01-01 18:05:02|
|2         | 101       | 1      |          |      | 2020-01-01 19:00:52|
|3         | 102       | 1      |          |      | 2020-01-02 23:51:23|
|3         | 102       | 2      |          |      | 2020-01-02 23:51:23|
|4         | 103       | 1      | 4        |      | 2020-01-04 13:23:46|
|4         | 103       | 1      | 4        |      | 2020-01-04 13:23:46|
|4         | 103       | 2      | 4        |      | 2020-01-04 13:23:46|
|5         | 104       | 1      |          | 1    | 2020-01-08 21:00:29|
|6         | 101       | 2      |          |      | 2020-01-08 21:03:13|
|7         | 105       | 2      |          | 1    | 2020-01-08 21:20:29|
|8         | 102       | 1      |          |      | 2020-01-09 23:54:33|
|9         | 103       | 1      | 4        | 1 5  | 2020-01-10 11:22:59|
|10        | 104       | 1      |          |      | 2020-01-11 18:34:49|
|10        | 104       | 1      | 2 6      | 1 4  | 2020-01-11 18:34:49|

---

### Table: runner_orders

```sql
SELECT * FROM runner_orders;
```
### Output:
|order_id|runner_id|pickup_time|distance|duration|cancellation        |
|----------|---------|-----------|--------|--------|--------------------|
|1         | 1       | 2020-01-01 18:15:34| 20km   | 32 minutes|                    |
|2         | 1       | 2020-01-01 19:10:54| 20km   | 27 minutes|                    |
|3         | 1       | 2020-01-03 00:12:37| 13.4km | 20 mins|                    |
|4         | 2       | 2020-01-04 13:53:03| 23.4   | 40     |                    |
|5         | 3       | 2020-01-08 21:10:57| 10     | 15     |                    |
|6         | 3       | null      | null   | null   | Restaurant Cancellation|
|7         | 2       | 2020-01-08 21:30:45| 25km   | 25mins | null               |
|8         | 2       | 2020-01-10 00:15:02| 23.4 km| 15 minute| null               |
|9         | 2       | null      | null   | null   | Customer Cancellation|
|10        | 1       | 2020-01-11 18:50:20| 10km   | 10minutes| null               |

```sql
DROP TEMPORARY TABLE IF EXISTS temp_runner_orders;

CREATE TEMPORARY TABLE temp_runner_orders AS
SELECT 
	order_id,
    runner_id,
    CASE
		WHEN pickup_time LIKE 'null' THEN null
        ELSE pickup_time
        END AS pickup_time,
	CASE
		WHEN distance LIKE 'null' THEN null
        ELSE regexp_replace(distance, '[A-Za-z]+', '')
        END AS distance_km,
	CASE
		WHEN duration LIKE 'null' THEN null
        ELSE regexp_replace(duration, '[A-Za-z]+', '')
        END AS duration_min,
	CASE 
		WHEN cancellation LIKE 'null' OR cancellation IS NULL or cancellation = ''
        THEN null
        ELSE cancellation
        END AS cancellation
FROM pizza.runner_orders;

ALTER TABLE temp_runner_orders
MODIFY COLUMN pickup_time DATETIME,
MODIFY COLUMN distance_km FLOAT,
MODIFY COLUMN duration_min INT;

SELECT * FROM temp_runner_orders;
```

### Output:

|order_id|runner_id|pickup_time|distance_km|duration_min|cancellation        |
|----------|---------|-----------|-----------|------------|--------------------|
|1         | 1       | 2020-01-01 18:15:34| 20        | 32         |                    |
|2         | 1       | 2020-01-01 19:10:54| 20        | 27         |                    |
|3         | 1       | 2020-01-03 00:12:37| 13.4      | 20         |                    |
|4         | 2       | 2020-01-04 13:53:03| 23.4      | 40         |                    |
|5         | 3       | 2020-01-08 21:10:57| 10        | 15         |                    |
|6         | 3       |           |           |            | Restaurant Cancellation|
|7         | 2       | 2020-01-08 21:30:45| 25        | 25         |                    |
|8         | 2       | 2020-01-10 00:15:02| 23.4      | 15         |                    |
|9         | 2       |           |           |            | Customer Cancellation|
|10        | 1       | 2020-01-11 18:50:20| 10        | 10         |                    |

---

## <p align="center"> Section A. Pizza Metrics

### 1. How many pizzas were ordered?

```sql
SELECT COUNT(pizza_id) AS pizza_ordered
FROM temp_customer_orders;
```

### Output:
|pizza_ordered|
|-------------|
| 14          |


### 2. How many unique customer orders were made?

```sql
SELECT COUNT(DISTINCT(order_id)) AS unique_orders
FROM temp_customer_orders;
```

### Output:
|unique_orders|
|-------------|
| 10          |


### 3. How many successful orders were delivered by each runner?

```sql
SELECT runner_id, COUNT(order_id) AS successful_orders_num
FROM temp_runner_orders
WHERE cancellation IS NULL
GROUP BY runner_id;
```

### Output:
|  runner_id|successful_orders_num|
|-----------|---------------------|
|1          | 4                   |
|2          | 3                   |
|3          | 1                   |


### 4. How many of each type of pizza was delivered?

```sql
SELECT pizza_name, COUNT(order_id) AS pizza_delivered_num
FROM temp_runner_orders
JOIN temp_customer_orders USING (order_id)
JOIN pizza_names USING(pizza_id)
WHERE cancellation IS NULL
GROUP BY pizza_name;
```

### Output:
|  pizza_name|pizza_delivered_num|
|------------|-------------------|
|Meatlovers  | 9                 |
|Vegetarian  | 3                 |


### 5. How many Vegetarian and Meatlovers were ordered by each customer?

```sql
SELECT customer_id, pizza_name, COUNT(order_id) AS pizza_delivered_num
FROM temp_runner_orders
JOIN temp_customer_orders USING (order_id)
JOIN pizza_names USING(pizza_id)
GROUP BY customer_id, pizza_name
ORDER BY customer_id;
```

### Output:
|  customer_id|pizza_name |pizza_delivered_num|
|-------------|-----------|-------------------|
|101          | Meatlovers| 2                 |
|101          | Vegetarian| 1                 |
|102          | Meatlovers| 2                 |
|102          | Vegetarian| 1                 |
|103          | Meatlovers| 3                 |
|103          | Vegetarian| 1                 |
|104          | Meatlovers| 3                 |
|105          | Vegetarian| 1                 |


### 6. What was the maximum number of pizzas delivered in a single order?

```sql
SELECT order_id, COUNT(pizza_id) pizza_num
FROM temp_customer_orders
JOIN temp_runner_orders USING(order_id)
WHERE cancellation IS NULL
GROUP BY order_id
ORDER BY pizza_num DESC
LIMIT 1;
```

### Output:
|  order_id|pizza_num  |
|----------|-----------|
|4         | 3         |


### 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

```sql
SELECT 
	customer_id,
	SUM(CASE WHEN exclusions IS NOT NULL or extras IS NOT NULL THEN 1 ELSE 0 END)
		AS pizza_with_changes,
    SUM(CASE WHEN exclusions IS NULL AND extras IS NULL THEN 1 ELSE 0 END)
		AS pizza_without_changes
FROM temp_customer_orders
JOIN temp_runner_orders USING(order_id)
WHERE cancellation IS NULL
GROUP BY customer_id;
```

### Output:
|  customer_id|pizza_with_changes|pizza_without_changes|
|-------------|------------------|---------------------|
|101          | 0                | 2                   |
|102          | 0                | 3                   |
|103          | 3                | 0                   |
|104          | 2                | 1                   |
|105          | 1                | 0                   |


### 8. How many pizzas were delivered that had both exclusions and extras?

```sql
SELECT
	SUM(CASE WHEN exclusions IS NOT NULL AND extras IS NOT NULL THEN 1 ELSE 0 END)
		AS pizza_with_exclusions_and_extras
FROM temp_customer_orders
JOIN temp_runner_orders USING(order_id)
WHERE cancellation IS NULL;
```

### Output:
|pizza_with_exclusions_and_extras|
|--------------------------------|
| 1                              |


### 9. What was the total volume of pizzas ordered for each hour of the day?

```sql
SELECT 
	HOUR(order_time) AS hour,
    COUNT(order_id) AS pizza_num
FROM temp_customer_orders
GROUP BY hour
ORDER BY hour;
```

### Output:
|  hour|pizza_num|
|------|---------|
|11    | 1       |
|13    | 3       |
|18    | 3       |
|19    | 1       |
|21    | 3       |
|23    | 3       |


### 10. What was the volume of orders for each day of the week?

```sql
SELECT 
	dayname(order_time) AS day_of_week,
    COUNT(DISTINCT(order_id)) AS pizza_num
FROM temp_customer_orders
GROUP BY day_of_week
ORDER BY day_of_week;
```

### Output:
|  day_of_week|pizza_num|
|-------------|---------|
|Friday       | 1       |
|Saturday     | 2       |
|Thursday     | 2       |
|Wednesday    | 5       |

---

## <p align="center"> Section B. Runner and Customer Experience

### 1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

```sql
SELECT DATE_FORMAT(DATE_ADD(registration_date, INTERVAL 1 WEEK), '%u')
	AS week,
	COUNT(runner_id) runners_num
FROM runners
GROUP BY week;
```

### Output:
|  week|runners_num|
|------|-----------|
|01    | 2         |
|02    | 1         |
|03    | 1         |


### 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

```sql
SELECT runner_id,
	ROUND(AVG(MINUTE(timediff(pickup_time, order_time)))) avg_pickup_time_min
FROM temp_customer_orders
JOIN temp_runner_orders USING(order_id)
WHERE cancellation IS NULL
GROUP BY runner_id;
```

### Output:
|  runner_id|avg_pickup_time_min|
|-----------|-------------------|
|1          | 15                |
|2          | 23                |
|3          | 10                |


### 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?

```sql
WITH cte AS(
	SELECT
		COUNT(pizza_id) pizzas_num,
		ROUND(AVG(MINUTE(timediff(pickup_time, order_time)))) avg_pickup_time_min
	FROM temp_customer_orders
	JOIN temp_runner_orders USING(order_id)
	WHERE cancellation IS NULL
	GROUP BY order_id
	ORDER BY pizzas_num
)
SELECT pizzas_num,
	ROUND(AVG(avg_pickup_time_min)) avg_pickup_time_min
FROM cte
GROUP BY pizzas_num;
```

### Output:
|  pizzas_num|avg_pickup_time_min|
|------------|-------------------|
|1           | 12                |
|2           | 18                |
|3           | 29                |

### 4. What was the average distance travelled for each customer?

```sql
SELECT customer_id,
	ROUND(AVG(distance_km)) distance_km
FROM temp_customer_orders
JOIN temp_runner_orders USING(order_id)
GROUP BY customer_id
ORDER BY distance_km;
```

### Output:
|  customer_id|distance_km|
|-------------|-----------|
|104          | 10        |
|102          | 17        |
|101          | 20        |
|103          | 23        |
|105          | 25        |

### 5. What was the difference between the longest and shortest delivery times for all orders?

```sql
SELECT (max(duration_min) - min(duration_min)) 
	AS 'Difference Between Longest and Shortest Delivery Time (min)'
FROM temp_runner_orders;
```

### Output:
|Difference Between Longest and Shortest Delivery Time (min)|
|-----------------------------------------------------------|
| 30                                                        |


### 6. What was the average speed for each runner for each delivery?

```sql
SELECT order_id,
	runner_id,
	AVG(ROUND(distance_km / (duration_min / 60))) speed_km_h
FROM temp_runner_orders
WHERE cancellation IS NULL
GROUP BY order_id, runner_id
ORDER BY runner_id;
```

### Output:
|  order_id|runner_id|speed_km_h|
|----------|---------|----------|
|1         | 1       | 38       |
|2         | 1       | 44       |
|3         | 1       | 40       |
|10        | 1       | 60       |
|4         | 2       | 35       |
|7         | 2       | 60       |
|8         | 2       | 94       |
|5         | 3       | 40       |


### 7. What is the successful delivery percentage for each runner?

```sql
SELECT runner_id,
	ROUND((SUM(CASE WHEN cancellation IS NULL THEN 1 ELSE 0 END) / COUNT(*)) * 100)
		AS 'Successful Delivery Percentage'
FROM temp_runner_orders
GROUP BY runner_id;
```

### Output:
|  runner_id|Successful Delivery Percentage|
|-----------|------------------------------|
|1          | 100                          |
|2          | 75                           |
|3          | 50                           |













