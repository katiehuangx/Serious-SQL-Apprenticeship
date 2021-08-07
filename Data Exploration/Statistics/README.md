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
