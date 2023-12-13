## Available Data
  
<details><summary>
    All datasets exist in database schema.
  </summary> 
  
 #### ``Table 1: Regions``
region_id | region_name
-- | --
1 | Africa
2 | America
3 | Asia
4 | Europe
5 | Oceania

#### ``Table 2: subscriptions``
*Note: this is only customer sample*
customer_id | region_id | node_id | start_date | end_date
-- | -- | -- | -- | --
1 | 3 | 4 | 2020-01-02 | 2020-01-03
2 | 3 | 5 | 2020-01-03 | 2020-01-17
3 | 5 | 4 | 2020-01-27 | 2020-02-18
4 | 5 | 4 | 2020-01-07 | 2020-01-19
5 | 3 | 3 | 2020-01-15 | 2020-01-23
6 | 1 | 1 | 2020-01-11 | 2020-02-06
7 | 2 | 5 | 2020-01-20 | 2020-02-04
8 | 1 | 2 | 2020-01-15 | 2020-01-28
9 | 4 | 5 | 2020-01-21 | 2020-01-25
10 | 3 | 4 | 2020-01-13 | 2020-01-14

#### ``Table 3: Customer Transactions``
*Note: this is only customer sample*
customer_id | txn_date | txn_type | txn_amount
-- | -- | -- | --
429 | 2020-01-21 | deposit | 82
155 | 2020-01-10 | deposit | 712
398 | 2020-01-01 | deposit | 196
255 | 2020-01-14 | deposit | 563
185 | 2020-01-29 | deposit | 626
309 | 2020-01-13 | deposit | 995
312 | 2020-01-20 | deposit | 485
376 | 2020-01-03 | deposit | 706
188 | 2020-01-13 | deposit | 601
138 | 2020-01-11 | deposit | 520

  </details>


## <p align="center">  A. Customer Nodes Exploration


### 1. How many nodes are there on the Data Bank System?

```sql
SELECT COUNT(DISTINCT(node_id)) AS "Number of Unique Nodes" 
FROM customer_nodes;
```
### Output:
|unique_nodes |
| -- |
|5|


### 2. What is the number of nodes per region?

```sql
SELECT region_id, 
  region_name, 
  COUNT(node_id) AS num_of_nodes
FROM customer_nodes
JOIN regions USING(region_id)
GROUP BY regions.region_id, region_name
ORDER BY region_id ASC;
```
### Output:
region_id | region_name | num_of_nodes
-- | -- | -- 
1 | Australia | 770
2 | America | 735
3 | Africa | 714
4 | Asia | 665
5 | Europe | 616


### 3. How many customers are allocated to each region?

```sql
SELECT region_id, 
  region_name, 
  COUNT(DISTINCT(customer_id)) AS num_of_customers
FROM customer_nodes
JOIN regions USING(region_id)
GROUP BY regions.region_id, region_name
ORDER BY num_of_customers DESC;
```
### Output:

region_id | region_name | num_of_customers
-- | -- | -- 
1 | Australia | 110
2 | America | 105
3 | Africa | 102
4 | Asia | 95
5 | Europe | 88


### 4. How many days on average are customers reallocated to a different region?

```sql
SELECT DISTINCT(start_date)
FROM customer_nodes
ORDER BY start_date DESC;

SELECT DISTINCT(end_date)
FROM customer_nodes
ORDER BY end_date DESC;

SELECT 
  ROUND(AVG(DATEDIFF(end_date, start_date))) AS avg_days
FROM customer_nodes
WHERE end_date != "9999-12-31";
```
### Output:

|avg_number_of_day |
| -- |
|15|


### 5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?
Unfortinately, there's no percentile function in mysql (which I used for this project), so I had to create mine.
```sql
WITH days_metrics AS (
      SELECT 
        DATEDIFF(end_date, start_date) AS num_days,
        regions.region_id,
        regions.region_name
      FROM customer_nodes
      JOIN regions USING(region_id)
      WHERE end_date != "9999-12-31"
)
SELECT DISTINCT
  region_id,
  region_name,
  MAX(x1) OVER (PARTITION BY region_id, region_name ORDER BY x1) median,
  MAX(x2) OVER (PARTITION BY region_id, region_name ORDER BY x2) percentile_80,
  MAX(x3) OVER (PARTITION BY region_id, region_name ORDER BY x3) percentile_95
FROM (
  SELECT
    region_id,
    region_name,
    first_value(num_days) OVER (PARTITION BY region_id, region_name 
      ORDER BY CASE WHEN p <= 0.5 THEN p END DESC) x1,
    first_value(num_days) OVER (PARTITION BY region_id, region_name
      ORDER BY CASE WHEN p <= 0.80 THEN p END DESC) x2,
    first_value(num_days) OVER (PARTITION BY region_id, region_name
      ORDER BY CASE WHEN p <= 0.95 THEN p END DESC) x3
  FROM (
      SELECT
        region_id,
        region_name,
        num_days,
        percent_rank() OVER (PARTITION BY region_id, region_name ORDER BY num_days) p
      FROM days_metrics
  ) t
) t
ORDER BY region_name;
```
### Output:

region_id | region_name | median | percentile_80 | percentile_95
--| -- | -- | -- | --
1 | Africa | 15 | 24 | 28
2 | America | 15 | 23 | 28
3 | Asia | 15 | 23 | 28
4 | Australia | 15 | 23 | 28
5 | Europe | 15 | 24 | 28



## <p align="center">  B. Customer Transactions


### 1. What is the unique count and total amount for each transaction type?

```sql
SELECT 
  txn_type,
  COUNT(*) AS unique_count,
  SUM(txn_amount) AS total_amount
FROM customer_transactions
GROUP BY txn_type
ORDER BY txn_type;
```
### Output:

txn_type | unique_count | total_amount
-- | -- |--
deposit | 2671 | 1359168
purchase | 1617 | 806537
withdrawal | 1580 | 793003


### 2. What is the average total historical deposit counts and amounts for all customers?

```sql
WITH deposit_type AS (
      SELECT 
        customer_id,
        COUNT(*) counts,
        SUM(txn_amount) amounts
      FROM customer_transactions
      WHERE txn_type = 'deposit'
      GROUP BY customer_id
)
SELECT 
  ROUND(AVG(counts)) avg_counts,
  ROUND(AVG(amounts)) avg_amounts
FROM deposit_type;
```
### Output:

avg_counts | avg_amounts
-- | --
5 | 2718


### 3. For each month - how many Data Bank customers make more than 1 deposit and either one purchase or withdrawal in a single month?

```sql
WITH cte AS (
      SELECT 
        customer_id,
        DATE_FORMAT(txn_date, '%M') AS month,
        COUNT(CASE WHEN txn_type = 'deposit' THEN 1 END) AS deposit_num,
        COUNT(CASE WHEN txn_type = 'withdrawal' THEN 1 END) AS withdrawal_num,
        COUNT(CASE WHEN txn_type = 'purchase' THEN 1 END) AS purchase_num
      FROM customer_transactions
      GROUP BY customer_id, month
)
SELECT 
  month, 
  COUNT(DISTINCT(customer_id)) AS active_customers
FROM cte
WHERE deposit_num > 1
	AND (purchase_num > 0 OR withdrawal_num > 0)
GROUP BY month
ORDER BY active_customers DESC;
```
### Output:

month_name | active_customer_count
-- | --
March | 192
February | 181
January | 168
April | 70


### 4. What is the closing balance for each customer at the end of the month?

```sql
WITH cte AS (
      SELECT
        customer_id,
        MONTH(txn_date) AS month_num,
        DATE_FORMAT(txn_date, '%M') AS month,
        (CASE WHEN txn_type = 'deposit' THEN txn_amount ELSE -txn_amount END)
      		AS balance
      FROM customer_transactions
)
SELECT DISTINCT
  customer_id,
  month,
  SUM(balance) OVER(PARTITION BY customer_id ORDER BY month_num) 
  AS closing_balance
FROM cte;
```
### Output:

**NOTE*** : *Not all output is displayed, considering the number of results that will take up space*

customer_id | month_name | closing_balance
-- | -- | --
1 | January | 312
1 | March | -640
2 | January | 549
2 | March | 610
3 | January | 144
3 | February | -821
3 | March | -1222
3 | April | -729
4 | January | 848
4 | March | 655
5 | January | 954
5 | March | -1923
5 | April | -2413


### 5. What is the percentage of customers who increase their closing balance by more than 5%?

```sql
WITH monthly_transactions AS (
      SELECT
        customer_id,
        LAST_DAY(txn_date) last_day,
        (CASE WHEN txn_type = 'deposit' THEN txn_amount ELSE -txn_amount END)
          AS balance
      FROM customer_transactions
      ORDER BY customer_id
),
closing_balance AS (
      SELECT DISTINCT
        customer_id,
        last_day,
        SUM(balance) OVER(PARTITION BY customer_id ORDER BY last_day) 
          AS closing_balance
      FROM monthly_transactions
), 
pct_change AS (
      SELECT 
        customer_id,
        last_day,
        closing_balance,
        100 * (closing_balance - LAG(closing_balance) OVER (PARTITION BY customer_id ORDER BY last_day)) / 
          NULLIF((LAG(closing_balance) OVER (PARTITION BY customer_id ORDER BY last_day)), 0) 
            AS percent_change
      FROM closing_balance
),
final_pct_balance AS (
      SELECT DISTINCT
        customer_id,
        LAST_VALUE(percent_change) 
          OVER (PARTITION BY customer_id 
          ORDER BY last_day 
          RANGE BETWEEN UNBOUNDED PRECEDING 
          AND UNBOUNDED FOLLOWING)
      	AS balance
      FROM pct_change
)
SELECT 
	ROUND(((COUNT(CASE WHEN balance > 5 THEN 1 END) / COUNT(*)) * 100), 1) AS percent_of_customers
FROM final_pct_balance;
```
### Output:

|percent_of_customers|
| -- |
|46.4|





