# 👩🏻‍⚕️ Health Analytics Mini Case Study

## 📌 Solution

### 1. How many unique users exist in the logs dataset?

````sql
SELECT 
  COUNT(DISTINCT id) AS unique_users
FROM health.user_logs
````
**Answer:**

<img width="141" alt="image" src="https://user-images.githubusercontent.com/81607668/128624690-edfb1c8e-2a46-4bd9-a699-d2e678d42664.png">

There are 554 unique users in the dataset.

***

### To answer Q2 to Q8, we will create a temporary table.

````sql
DROP TABLE IF EXISTS user_measure_count;

CREATE TEMP TABLE user_measure_count AS(
SELECT
  id,
  COUNT(*) AS measure_count,
  COUNT(DISTINCT measure) AS unique_measure_count
FROM health.user_logs
GROUP BY id);
````

<img width="801" alt="image" src="https://user-images.githubusercontent.com/81607668/128625477-926f9d69-f307-4e6a-bdd6-40d026338fed.png">

Alright, once we have created the temp table, let's move on to our questions.

***

### 2. How many total measurements do we have per user on average?

Question is asking for the **average number of measurements for all users**.

````sql
SELECT 
  ROUND(AVG(measure_count),2) AS avg_measurement
FROM user_measure_count;
````

**Answer:**

<img width="161" alt="image" src="https://user-images.githubusercontent.com/81607668/128625883-1fd5191b-0347-4d16-87f6-a26f50e75b5d.png">

The average measurements per user is 79 measurements. 

***

### 3. What about the median number of measurements per user?

````sql
SELECT 
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_count) AS median_value
FROM user_measure_count;
````
**Answer:**

<img width="134" alt="image" src="https://user-images.githubusercontent.com/81607668/128625985-18f389cd-a56e-4c2a-b37b-2db2e9bebe17.png">

The median measurement per user is 2. 

***

### 4. How many users have 3 or more measurements?

````sql
SELECT 
  COUNT(*)
FROM user_measure_count
WHERE measure_count >= 3
````
**Answer:**

<img width="105" alt="image" src="https://user-images.githubusercontent.com/81607668/128626070-33506b6c-83e6-43f8-a04e-98feac608d96.png">

209 users have 3 or more measurements.

***

### 5. How many users have 1,000 or more measurements?

````sql
SELECT 
  COUNT(*)
FROM user_measure_count
WHERE measure_count >= 1000
````
**Answer:**

<img width="98" alt="image" src="https://user-images.githubusercontent.com/81607668/128626077-b7dfae74-b93c-4cc5-b8ba-581d7edf4f19.png">

There are 5 users with 1,000 or more measurements.

***
### Looking at the logs data -
### 6. What is the number and percentage of the active user base who have logged blood glucose measurements?

````sql
SELECT
  measure,
  COUNT(DISTINCT id) AS unique_blood_glucose_user,
  ROUND(100 * COUNT(DISTINCT id)::NUMERIC / SUM(COUNT(DISTINCT id)) OVER (),2) AS blood_glucose_percentage
FROM health.user_logs
GROUP BY measure;
````
**Answer:**

<img width="750" alt="image" src="https://user-images.githubusercontent.com/81607668/128626812-8358e599-6025-41ca-81ea-31679997a922.png">

There are 325 users or 40% of user base who have logged blood glucose measurements.
***
### 7. What is the number and percentage of the active user base who have at least 2 types of measurements?

````sql
WITH measure_more_than_2 AS (
SELECT *
FROM user_measure_count
WHERE unique_measure_count >= 2)

SELECT
  COUNT(DISTINCT m.id) AS unique_user,
  ROUND(100 * COUNT(DISTINCT m.id)::numeric / COUNT(DISTINCT u.id),2) AS unique_user_percentage
FROM user_measure_count AS u
LEFT JOIN measure_more_than_2 AS m
  ON u.id = m.id;
````

**Answer:**

<img width="423" alt="image" src="https://user-images.githubusercontent.com/81607668/128656551-59b2ef0e-e35b-477f-9fa3-4ff4424e6d2e.png">

Out of 554 active users, 204 users have at least 2 types of measurements making up 37% of total active user base.
***
### 8. What is the number and percentage of the active user base who have all 3 measures - blood glucose, weight and blood pressure?

- First thing we will do is to create a `CTE` with results filtered to users with all measures.
- Then, we perform a `LEFT JOIN` on the user_measure_count and all_measures `CTE` meaning, we will retrieve all records in user_measure_count and matching records from all_measures `CTE`. 

````sql
WITH all_measures AS (
SELECT *
FROM user_measure_count
WHERE unique_measure_count = 3)

SELECT
  COUNT(DISTINCT m.id) AS unique_user,
  ROUND(COUNT(DISTINCT m.id)::numeric / COUNT(DISTINCT u.id),2) AS unique_user_percentage
FROM user_measure_count AS u
LEFT JOIN all_measures AS m
  ON u.id = m.id;
````
**Answer:**

<img width="434" alt="image" src="https://user-images.githubusercontent.com/81607668/128656575-ac81483f-2716-4820-8cf2-8acd35053278.png">

Out of 554 active users, 50 users have taken all 3 measures making up 9% of total active user base.
***
### 9. For users that have blood pressure measurements, what is the median systolic/diastolic blood pressure values?

- First, we filter results to users with blood pressure measurements.
- Then, we find the median of systolic and diastolic measurements.

````sql
SELECT
  'blood_pressure' AS measure_name,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY systolic) AS systolic_median,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY diastolic) AS diastolic_median
FROM health.user_logs
WHERE measure = 'blood_pressure';
````
**Answer:**

<img width="721" alt="image" src="https://user-images.githubusercontent.com/81607668/128656051-4743b034-0170-40ee-9c1a-38025652c55e.png">

The median for systolic and diastolic are 126 and 79.

***

