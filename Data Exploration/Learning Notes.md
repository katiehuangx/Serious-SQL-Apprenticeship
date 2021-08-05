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

````
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

- Use `CTE` and generate a table with new column frequency to count number of duplicated rows. 
- `SUM` frequency to find total number of duplicate records for each id regardless of the log_date and other measures.

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
FROM groupby_counts
WHERE frequency > 1
GROUP BY id
ORDER BY total_frequency DESC;
````

**Which log_date value had the most duplicate records after removing the max duplicate id value from question 1?**

- Questions asks for the 2nd highest duplicate value. 
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
