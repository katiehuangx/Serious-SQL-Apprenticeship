# 🎬 Marketing Analytics Case Study

## 📌 Introduction

***

## 📌 Understanding the Data
  
<img width="773" alt="image" src="https://user-images.githubusercontent.com/81607668/128663547-9b73770f-7505-47f0-a62f-9d44375504a5.png">

***

## 📌 Data Exploration

Before we dive into problem-solving, let's explore the data! We will develop a few hypotheses and test them using SQL.

### Hypothesis 1
> The number of unique `inventory_id` records will be equal in both `dvd_rentals.rentals` and `dvd_rentals.inventory` tables.

````sql
SELECT 
  COUNT(DISTINCT inventory_id) -- As there are multiple inventory_id for each film_id, run DISTINCT to find unique inventory_id
FROM dvd_rentals.rental
````

<img width="138" alt="image" src="https://user-images.githubusercontent.com/81607668/129321232-9aef17ed-3abc-4f7c-a650-16b7744b059d.png">

````sql
SELECT 
  COUNT(inventory_id)
FROM dvd_rentals.inventory;
````
<img width="103" alt="image" src="https://user-images.githubusercontent.com/81607668/129321177-34d7d75d-3881-45be-822c-28ca7e05d8b0.png">

Looks like there are 4,580 `inventory_id` records in `dvd_rentals.rentals` and 4,581 `inventory_id` records in `dvd_rentals.inventory`. 

There seems to be an additional 1 record `inventory_id` record in `dvd_rentals.inventory`. 

### Hypothesis 2
> There will be a multiple records per unique `inventory_id` in the `dvd_rentals.rental` table.

````sql
WITH inventory_cte AS ( -- Generate a CTE using GROUP BY count of inventory_id
SELECT 
  inventory_id, 
  COUNT(*) AS inventory_id_count
FROM dvd_rentals.rental
GROUP BY inventory_id)
````

<img width="327" alt="image" src="https://user-images.githubusercontent.com/81607668/129326477-d136a198-2f04-4bf9-8dbb-78f201f7c7e9.png">

The table above shows the number of inventory records for each unique `inventory_id`. Then, we create a `CTE` and perform another grouping below.

````sql
SELECT 
  inventory_id_count,
  COUNT(*) AS inventory_id_grouping
FROM inventory_cte
GROUP BY inventory_id_count
ORDER BY inventory_id_count;
````

<img width="521" alt="image" src="https://user-images.githubusercontent.com/81607668/129326263-74b31410-9517-4a40-8847-5ef572d38ac9.png">

Ok, `inventory_id_count` represents the number of inventory records for each film and `inventory_id_grouping` is `inventory_id_count` grouped by the number of records.

For example, in 1st row, there is 1 inventory_id/ film that has 4 copies of inventory/film.

### Hypothesis 3
> There will be multiple `inventory_id` records per unique `film_id` value in the `dvd_rentals.inventory` table

````sql
WITH inventory_grouped AS (
SELECT 
  DISTINCT(film_id) AS unique_film_id, 
  COUNT(*) AS inventory_count
FROM dvd_rentals.inventory
GROUP BY film_id)
````

<img width="291" alt="image" src="https://user-images.githubusercontent.com/81607668/129327670-a840f098-45fd-4851-a1ec-5d00d4336e0a.png">

````sql
SELECT 
  inventory_count, 
  COUNT(*) AS 
FROM inventory_grouped
GROUP BY inventory_count
ORDER BY inventory_count;
````

<img width="224" alt="image" src="https://user-images.githubusercontent.com/81607668/129328276-ae08e35e-c355-4db7-ad0f-3c3ee6d48f96.png">

There are 2 inventory_id 

***

## 📌 Solution 

***

## ✅ Learning Outcomes

<details>
<summary>
Click to view my learning outcomes!
  
</summary>
  
The following SQL skills and concepts will be covered in this section of the Serious SQL course:

1. Learning how to interpret ERDs for data literacy and business context (entity-relationship diagrams)
- Identify key columns of interest and how they are linked to other tables via foreign keys
- Use ERDs to analyze the data types for different columns in database tables
- Understand data context for various tables that cause inherent natural relationships between fields

2. Introduction to all types of table joining operations
- Simple joins: left, inner
- More involved joins: cross, anti, left-semi, full outer
- Combination set operations: union, union all, intersect, except

3. Practical application of table joins
- Joining multiple tables to combine disparate datasets into a single data structure
- Joining interim SQL outputs for more advanced group-by, split, merge data hacking strategies
- Performing table joins that use 2 or more table references in the ON condition
- Using anti joins to exclude records in a sequential fashion

4. Filtering, window functions and aggregates for analytics and ranking
- Using ROW_NUMBER to effecively rank order records with equal ties
- Using WHERE filters to extract ranked records
- Using multiple aggregate functions with different target partitions and ordering expressions for efficient data analysis
- Using aggregate group by clauses to generate unique customer level insights

5. Case statements for data transformation and manipulation
- Pivoting datasets from long to wide using MAX and CASE WHEN
- Manipulating actual data values using conditional logic for business translation purposes

6. SQL scripting basics
- Designing SQL workflows which can be easily understood and implemented
- Managing multiple dependencies for downstream table joining operations by using temporary tables to store interim datasets

7. Manipulating text data
- Converting text columns to title case
- Combining multiple text and numeric data type columns into a single text expression

</details>
  
***
