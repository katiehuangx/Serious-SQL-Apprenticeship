# 📚 Serious SQL

## 🌎 Data Exploration

### 📌 Select and Sort Data

**What is the name of the category with the highest category_id in the dvd_rentals.category table?**

- Use `LIMIT` to restrict result to the highest category_id only.

````sql
SELECT 
  name, 
  category_id
FROM dvd_rentals.category
ORDER BY category_id DESC
LIMIT 1;
````

**For the films with the longest length, what is the title of the “R” rated film with the lowest replacement_cost in dvd_rentals.film table?**

- There are 3 parts to this question. 
  - 1st, filter results to find 'R' rating,
  - 2nd, sort films' length from longest to shortest, and
  - 3rd, sort from least to most replacement cost   

```sql
SELECT 
  title, 
  length, 
  replacement_cost, 
  rating
FROM dvd_rentals.film
WHERE rating = 'R'
ORDER BY 
  length DESC, 
  replacement_cost ASC;
````

**Who was the manager of the store with the highest total_sales in the dvd_rentals.sales_by_store table?**

````sql
SELECT manager
FROM dvd_rentals.sales_by_store
ORDER BY total_sales DESC -- To find out highest to lowest total sales
LIMIT 1; - To return 1 row for highest total sales
````

**What is the postal_code of the city with the 5th highest city_id in the dvd_rentals.address table?**


````sql
SELECT postal_code
FROM dvd_rentals.address
ORDER BY city_id DESC
LIMIT 1 OFFSET 4; -- To return 5th row, offset the 1st 4 rows and limit results to 5th row only
````

***

## 📌 Returns Count and Distinct Values

**What is the frequency of values in the rating column in the film table?**

- Cast either `COUNT(*)` or `SUM(COUNT(*))` as numeric to avoid integer floor division whereby instead of resulting in 0.75, the answer is 0.
- To cast as numeric, either use `::NUMERIC` or `CAST(column AS NUMERIC)`.  

To use `::NUMERIC`
````sql
SELECT 
  rating, 
  COUNT(*) AS frequency,  
  ROUND(100 * COUNT(*)::NUMERIC / SUM(COUNT(*)) OVER (),2) AS percentage
FROM dvd_rentals.film_list
GROUP BY rating
ORDER BY percentage DESC;
````

To use CAST function
````sql
SELECT 
  rating, 
  COUNT(*) AS frequency,  
  ROUND(100 * CAST(COUNT(*) AS NUMERIC) / SUM(COUNT(*)) OVER (),2) AS percentage
FROM dvd_rentals.film_list
GROUP BY rating
ORDER BY percentage DESC;
````

**Which actor_id has the most number of unique film_id records in the dvd_rentals.film_actor table?**

- "number" and "unique" are hints to use `COUNT()` and `DISTINCT`.
- The question is asking "which actor_id has the most" meaning the result should be 1 row only, so we use `LIMIT` 1.

````sql
SELECT 
  actor_id, 
  COUNT(DISTINCT film_id) AS film_count
FROM dvd_rentals.film_actor 
GROUP BY actor_id
ORDER BY film_count DESC
LIMIT 1;
````

**How many distinct fid values are there for the 3rd most common price value in the dvd_rentals.nicer_but_slower_film_list table?**

- "how many" and "distinct" are hints to use `COUNT()` and `DISTINCT`. 
- Question asks for "most common price value" meaning we have to `GROUP BY` prices in common groups in order to find the 3rd group.

````sql
SELECT 
  price, 
  COUNT(DISTINCT fid) as unique_fid
FROM dvd_rentals.nicer_but_slower_film_list  
GROUP BY price
ORDER BY unique_fid
````

**How many unique country_id values exist in the dvd_rentals.city table?**

````sql
SELECT COUNT(DISTINCT country_id)
FROM dvd_rentals.city;
````

**What percentage of overall total_sales does the Sports category make up in the dvd_rentals.sales_by_film_category table?**

- To count percentage of "Sports" sales against total sales, we cannot use `WHERE` to filter "Sports" only as it would result in comparing "Sports" sales against itself. 

````sql
SELECT 
  category,
  ROUND(100 * total_sales::NUMERIC / SUM(total_sales) OVER (),2) AS sales_percentage
FROM dvd_rentals.sales_by_film_category;
````

**What percentage of unique fid values are in the Children category in the dvd_rentals.film_list table?**

- To calculate percentage, use `COUNT(DISTINCT )` to get number of distinct categories. 

````sql
SELECT 
  category, 
  COUNT(DISTINCT fid) AS unique_fid,
  ROUND(100 * COUNT(DISTINCT fid)::NUMERIC / SUM(COUNT(DISTINCT fid)) OVER (), 2) AS percentage
FROM dvd_rentals.film_list
GROUP BY category;
````

***

## 📌 Identifying Duplicate Records

**Which id value has the most number of duplicate records in the health.user_logs table?*

- Use `CTE` and generate table with new column, frequency to count number of rows. 
- Filter frequency > 1 to generate only duplicated records, then `SUM` frequency to find total number of the duplicated records for each id regardless of the log_date and other measures.

````sql
WITH grouped_cte AS (
  SELECT 
    id, 
    log_date, 
    measure, 
    measure_value, 
    systolic, 
    diastolic, 
    COUNT(*) AS frequency
  FROM health.user_logs
  GROUP BY 
    id, 
    log_date, 
    measure, 
    measure_value, 
    systolic, 
    diastolic, 
)

SELECT 
  id, 
  SUM(frequency) AS total_frequency
FROM grouped_cte
WHERE frequency > 1 -- To return records that appear more than once only (eg, 2 or more duplicated rows)
GROUP BY id
ORDER BY total_frequency DESC; -- To show highest duplicated rows
````

**Which log_date value had the most duplicate records after removing the max duplicate id value from question 1?**

- Questions asks for log_date with highest duplicate values ***after*** filtering out id with highest duplicate rows. 
- Taking the `CTE` from previous question, filter out the id with the most duplicate rows.

````sql
WITH groupby_counts AS (
  SELECT 
    id, 
    log_date, 
    measure, 
    measure_value, 
    systolic, 
    diastolic, 
    COUNT(*) AS frequency
  FROM health.user_logs
  WHERE id != '054250c692e07a9fa9e62e345231df4b54ff435d'
  GROUP BY 
    id, 
    log_date, 
    measure, 
    measure_value, 
    systolic, 
    diastolic
)

SELECT 
  log_date, 
  SUM(frequency) AS total_frequency
FROM groupby_counts
WHERE frequency > 1
GROUP BY log_date
ORDER BY total_frequency DESC;
````

**Which measure_value had the most occurences in the health.user_logs value when measure = 'weight'?**

````sql
SELECT 
  measure_value, 
  COUNT(*) AS frequency
FROM health.user_logs
WHERE measure = 'weight'
GROUP BY measure_value
ORDER BY frequency DESC;
````
**How many single duplicated rows exist when measure = 'blood_pressure' in the health.user_logs? How about the total number of duplicate records in the same table?**

- **Single duplicated rows** means rows which have been duplicated at least once where **multiple duplicated rows** means the same value has been duplicated more than once.
- Here, use `CTE` to pull results with frequency where measure = 'blood_pressure'.
- To find out the frequency of single duplicated rows, filter more than 1. 

````sql
WITH blood_pressure_cte AS 
(
SELECT 
    id, 
    log_date, 
    measure, 
    measure_value, 
    systolic, 
    diastolic, 
    COUNT(*) AS frequency
FROM health.user_logs
WHERE measure = 'blood_pressure'
GROUP BY
    id, 
    log_date, 
    measure, 
    measure_value, 
    systolic, 
    diastolic
ORDER BY frequency DESC
)

SELECT 
  COUNT(*) AS duplicate_rows, 
  SUM(frequency) AS total_duplicate_rows
FROM blood_pressure_cte
WHERE frequency > 1
ORDER BY total_duplicate_rows DESC
````

**What percentage of records measure_value = 0 when measure = 'blood_pressure' in the health.user_logs table? How many records are there also for this same condition?**

````sql
WITH blood_pressure_measure AS
(
SELECT 
  measure_value,
  COUNT(*) AS frequency, -- frequency of measure = 'blood_pressure'
  SUM(COUNT(*)) OVER () AS total_frequency -- sum of frequency where measure = 'blood_pressure'
FROM health.user_logs
WHERE measure = 'blood_pressure'
GROUP BY measure_value
)

SELECT 
  measure_value, 
  frequency, -- frequency of measure = 'blood_pressure'
  total_frequency, -- sum of frequency where measure = 'blood_pressure'
  ROUND(100 * frequency / total_frequency, 2) AS percentage
FROM blood_pressure_measure
WHERE measure_value = 0;
````

**What percentage of records are duplicates in the health.user_logs table?**

````sql
WITH grouped_cte AS
(
SELECT 
    id, 
    log_date, 
    measure, 
    measure_value, 
    systolic, 
    diastolic, 
    COUNT(*) AS frequency
FROM health.user_logs
GROUP BY
    id, 
    log_date, 
    measure, 
    measure_value, 
    systolic, 
    diastolic
ORDER BY frequency DESC)

SELECT
  ROUND(100 * SUM(CASE
      WHEN frequency > 1 THEN frequency -1 -- Need to subtract 1 from the frequency to count actual duplicates
      ELSE 0 END)::NUMERIC / SUM(frequency), 2) AS duplicate_row_percentage
FROM grouped_cte;
````

***

## 📌 Summary Statistics

### Mean, Median and Mode

- Mean - Central tendency of distributions or a set of observations. Also known as the average.
- Median - Middle value of distribution.
- Mode - Most frequent values.

````sql
SELECT 
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_value) AS median_value,
  MODE() WITHIN GROUP (ORDER BY measure_value) AS mode_value,
  AVG(measure_value) AS mean_value
FROM health.user_logs
WHERE measure = 'weight';
````

### Min, Max and Range

- Use to find out how the is data being distributed.
- `MIN` and `MAX` identifies the boundaries of the data set.

````sql
SELECT 
  MIN(measure_value) AS min_value,
  MAX(measure_value) AS max_value,
  MAX(measure_value) - MIN(measure_value) AS range
FROM health.user_logs
WHERE measure = 'weight';
````

### Variance and Standard Deviation

For example,

````sql
EXPLAIN ANALYZE
WITH min_max_values AS (
  SELECT
    MIN(measure_value) AS minimum_value,
    MAX(measure_value) AS maximum_value
  FROM health.user_logs
  WHERE measure = 'weight'
)
SELECT
  minimum_value,
  maximum_value,
  maximum_value - minimum_value AS range_value
FROM min_max_values;
````

````sql
WITH sample_data (example_values) AS (
 VALUES
 (82), (51), (144), (84), (120), (148), (148), (108), (160), (86)
)
SELECT
  ROUND(VARIANCE(example_values), 2) AS variance_value,
  ROUND(STDDEV(example_values), 2) AS standard_dev_value,
  ROUND(AVG(example_values), 2) AS mean_value,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY example_values) AS median_value,
  MODE() WITHIN GROUP (ORDER BY example_values) AS mode_value
FROM sample_data;
````

Revisiting the exercise,
````sql
SELECT 
  'weight' as measure, 
  ROUND(MIN(measure_value),2) AS minimum_value,
  ROUND(MAX(measure_value),2) AS maximum_value,
  ROUND(AVG(measure_value),2) AS mean_value,
  ROUND(MODE() WITHIN GROUP (ORDER BY measure_value),2) as mode_value,
  CAST(PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_value) AS NUMERIC) AS median_value,
  ROUND(STDDEV(measure_value),2) AS stddev_value,
  ROUND(VARIANCE(measure_value),2) AS variance_value
FROM health.user_logs
WHERE measure = 'weight';
````

**What is the average, median and mode values of blood glucose values to 2 decimal places?**

````sql
SELECT
  ROUND(AVG(measure_value), 2) AS average_value,
  ROUND(CAST(PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_value) AS NUMERIC),2) AS median_value,
  ROUND(MODE() WITHIN GROUP (ORDER BY measure_value),2)AS mode_value
FROM health.user_logs
WHERE measure = 'blood_glucose';
````

**What is the most frequently occuring measure_value value for all blood glucose measurements?**

````sql
SELECT 
  measure_value, 
  COUNT(*) AS blood_glucose_count
FROM health.user_logs
WHERE measure = 'blood_glucose'
GROUP BY measure_value
ORDER BY blood_glucose_count DESC
LIMIT 10;
````

**Calculate the 2 Pearson Coefficient of Skewness for blood glucose measures given the following formulas:**

<img width="478" alt="image" src="https://user-images.githubusercontent.com/81607668/128620549-84f03d4f-a626-4548-8af1-f36f1fadfd8a.png">

- The skewness terms are quantitative measure of how lopsided a certain distribution is.
- To find out whether a specific table index has a “skew” in the values - leading to disproportionate allocation of data points to specific buckets or nodes.

````sql
WITH blood_glucose_cte AS
(SELECT 
  ROUND(AVG(measure_value),2) AS mean_value,
  ROUND(MODE() WITHIN GROUP (ORDER BY measure_value),2) AS mode_value,
  CAST(PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_value) AS NUMERIC) AS median_value,
  ROUND(STDDEV(measure_value),2) AS stddev_value
FROM health.user_logs
WHERE measure = 'blood_glucose')

SELECT 
  ROUND((mean_value - mode_value) / stddev_value,2) AS pearson_corr_1,
  ROUND(3 * (mean_value - median_value) / stddev_value,2) AS pearson_corr_2
FROM blood_glucose_cte;
````

## 📌 Distribution Functions

### Cumulative Distributions Table

````sql
WITH weight_cte AS 
(SELECT
  measure_value, 
  NTILE(100) OVER (ORDER BY measure_value) AS percentile
FROM health.user_logs
WHERE measure = 'weight')

SELECT
  percentile,
  MIN(measure_value) AS floor_value,
  MAX(measure_value) AS ceiling_value,
  COUNT(*) AS percentile_count
FROM weight_cte
GROUP BY percentile
ORDER BY percentile
````

### Finding for Large Outliers
````sql
WITH percentile_cte AS 
(SELECT
  measure_value, 
  NTILE(100) OVER (ORDER BY measure_value) AS percentile
FROM health.user_logs
WHERE measure = 'weight')

SELECT
  measure_value,
  ROW_NUMBER() OVER (ORDER BY measure_value DESC) AS row_number_order,
  RANK() OVER (ORDER BY measure_value DESC) AS rank_order,
  DENSE_RANK() OVER (ORDER BY measure_value DESC) AS dense_rank_order
FROM percentile_cte
ORDER BY measure_value DESC;
````

<img width="942" alt="image" src="https://user-images.githubusercontent.com/81607668/128622564-3b2f8feb-5e19-48f0-b1fd-2f9777279cdf.png">


### Finding for Small Outliers
````sql
WITH percentile_cte AS 
(SELECT
  measure_value, 
  NTILE(100) OVER (ORDER BY measure_value) AS percentile
FROM health.user_logs
WHERE measure = 'weight')

SELECT
  measure_value,
  ROW_NUMBER() OVER (ORDER BY measure_value) AS row_number_order,
  RANK() OVER (ORDER BY measure_value) AS rank_order,
  DENSE_RANK() OVER (ORDER BY measure_value) AS dense_rank_order
FROM percentile_cte
ORDER BY measure_value;
 ````
 
 <img width="950" alt="image" src="https://user-images.githubusercontent.com/81607668/128622852-42dd903e-b970-42b4-95a4-14a902813543.png">


**What is the difference between ROW_NUMBER, RANK and DENSE_RANK window functions?**

Let's use the Olympics game results as an example. There are 5 players and 2 players received the same 2nd best score. 

| Player | Score | Result | ROW_NUMBER() | RANK() | DENSE_RANK() |
| ------ | ----- | ------ | ------------ | ------ | ------------ |
| A      | 96    | Gold  | 1             | 1      | 1            |
| B      | 90    | Silver | 2            | 2      | 2            | 
| C      | 90    | Silver | 3            | 2      | 2            | 
| D      | 85    | Bronze | 4            | 4      | 3            | 
| E      | 78    | N/A    | 5            | 5      | 4            | 

ROW_NUMBER() orders each value depending on its position in the bucket, regardless of whether there are duplicates. Equation = N + 1.

RANK() ensures that duplicates have equal position, where the subsequent position after a set of duplicates is N + X (where X = number of duplicates). 

DENSE_RANK() also ensures that duplicates have equal position, but the subsequent position after a set of duplicates is N + 1. 

### Removing outliers

First, create a temporary clean table with outliers removed. 

````sql
DROP TABLE IF EXISTS clean_weight_logs;

CREATE TEMP TABLE clean_weight_logs AS (
SELECT *
FROM health.user_logs
WHERE measure = 'weight' 
  AND measure_value > 0
  AND measure_value < 201);
````

### Cumulative Distribution Visualization

````sql
WITH clean_percentile_cte AS (
SELECT
  measure_value,
  NTILE(100) OVER (ORDER BY measure_value) AS percentile
FROM clean_weight_logs)

SELECT
  percentile,
  MIN(measure_value) AS floor_value,
  MAX(measure_value) AS ceiling_value,
  COUNT(*) AS percentile_count
FROM clean_percentile_cte
GROUP BY percentile
ORDER BY percentile;
````

**Line Plot**
<img width="562" alt="image" src="https://user-images.githubusercontent.com/81607668/128623701-b27a83eb-8e9a-4081-b866-6a3ddd15cb3f.png">

**Histogram**

To create Histogram, we have to perform a **Frequency Distribution** whereby we put our values into buckets and get the frequency/counts for each bucket. 

````sql
SELECT
  WIDTH_BUCKET(measure_value, 0, 200, 50) AS bucket, -- Creating equally spaced buckets of 50
  AVG(measure_value) AS avg_value,
  COUNT(*) AS frequency
FROM clean_weight_logs
GROUP BY bucket
ORDER BY bucket;
````

<img width="440" alt="image" src="https://user-images.githubusercontent.com/81607668/128623800-dc689fdd-2de3-4c89-9e55-75be3c4a941c.png">

<img width="676" alt="image" src="https://user-images.githubusercontent.com/81607668/128623822-3a3f3a91-fd67-45ff-985a-79fa0312e59a.png">


***
