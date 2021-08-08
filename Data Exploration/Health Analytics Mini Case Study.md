# 👩🏻‍⚕️ Health Analytics Mini Case Study

## 📚 Business Questions
Before we start digging into the SQL script - let’s cover the business questions that we need to help the GM answer!

1. How many unique users exist in the logs dataset?
2. How many total measurements do we have per user on average?
3. What about the median number of measurements per user?
4. How many users have 3 or more measurements?
5. How many users have 1,000 or more measurements?

Looking at the logs data - what is the number and percentage of the active user base who:
1. Have logged blood glucose measurements?
2. Have at least 2 types of measurements?
3. Have all 3 measures - blood glucose, weight and blood pressure?

For users that have blood pressure measurements:
1. What is the median systolic/diastolic blood pressure values?

## 📌 Solution

### 1. How many unique users exist in the logs dataset?

````sql
SELECT 
  COUNT(DISTINCT id) AS unique_users
FROM health.user_logs
````

<img width="141" alt="image" src="https://user-images.githubusercontent.com/81607668/128624690-edfb1c8e-2a46-4bd9-a699-d2e678d42664.png">

### To answer Q2 to Q8, we create a temporary table.

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


### 2. How many total measurements do we have per user on average?

I was slightly confused with this question as I interpreted it as finding for the average number of measurement ***per user***. After studying the solution, the correct interpretation is **to find for the average number of measurements for all users**.

````sql
SELECT 
  ROUND(AVG(measure_count),2) AS avg_measurement
FROM user_measure_count;
````

<img width="161" alt="image" src="https://user-images.githubusercontent.com/81607668/128625883-1fd5191b-0347-4d16-87f6-a26f50e75b5d.png">

### 3. What about the median number of measurements per user?

````sql
SELECT 
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_count) AS median_value
FROM user_measure_count;
````

<img width="134" alt="image" src="https://user-images.githubusercontent.com/81607668/128625985-18f389cd-a56e-4c2a-b37b-2db2e9bebe17.png">


### 4. How many users have 3 or more measurements?

````sql
SELECT 
  COUNT(*)
FROM user_measure_count
WHERE measure_count >= 3
````

<img width="105" alt="image" src="https://user-images.githubusercontent.com/81607668/128626070-33506b6c-83e6-43f8-a04e-98feac608d96.png">


### 5. How many users have 1,000 or more measurements?

````sql
SELECT 
  COUNT(*)
FROM user_measure_count
WHERE measure_count >= 1000
````

<img width="98" alt="image" src="https://user-images.githubusercontent.com/81607668/128626077-b7dfae74-b93c-4cc5-b8ba-581d7edf4f19.png">


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

<img width="750" alt="image" src="https://user-images.githubusercontent.com/81607668/128626812-8358e599-6025-41ca-81ea-31679997a922.png">

### 7. What is the number and percentage of the active user base who have at least 2 types of measurements?


### 8. What is the number and percentage of the active user base who have all 3 measures - blood glucose, weight and blood pressure?




For users that have blood pressure measurements:
1. What is the median systolic/diastolic blood pressure values?
