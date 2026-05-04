# EDA-Using-SQL
# Laptop Data Analysis - SQL Case Study

## Questions and Answers


```sql
Q1
Question:
How do you switch to the sql_cx_live database?
Answer:
USE sql_cx_live;

Q2
Question: How do you view all records from the laptops table?
Answer:
SELECT * FROM laptops;

Q3
Question: How do you view the first 5 records from laptops table?
Answer:
SELECT * FROM laptops ORDER BY `index` LIMIT 5;

Q4
Question: How do you view the last 5 records from laptops table?
Answer:
SELECT * FROM laptops ORDER BY `index` DESC LIMIT 5;

Q5
Question: How do you randomly sample 5 records from laptops table?
Answer:
SELECT * FROM laptops ORDER BY rand() LIMIT 5;

Q6
Question: How do you calculate statistical summary (COUNT, MIN, MAX, AVG, STD, Q1, Median, Q3) for Price column?
Answer:
SELECT COUNT(Price) OVER(),
MIN(Price) OVER(),
MAX(Price) OVER(),
AVG(Price) OVER(),
STD(Price) OVER(),
PERCENTILE_CONT(0.25) WITHIN GROUP(ORDER BY Price) OVER() AS 'Q1',
PERCENTILE_CONT(0.5) WITHIN GROUP(ORDER BY Price) OVER() AS 'Median',
PERCENTILE_CONT(0.75) WITHIN GROUP(ORDER BY Price) OVER() AS 'Q3'
FROM laptops
ORDER BY `index` LIMIT 1;

Q7
Question: How do you count missing values in the Price column?
Answer:
SELECT COUNT(Price) FROM laptops WHERE Price IS NULL;

Q8
Question: How do you detect outliers in the Price column using IQR method?
Answer:
SELECT * FROM (
SELECT *,
PERCENTILE_CONT(0.25) WITHIN GROUP(ORDER BY Price) OVER() AS 'Q1',
PERCENTILE_CONT(0.75) WITHIN GROUP(ORDER BY Price) OVER() AS 'Q3'
FROM laptops
) t
WHERE t.Price < t.Q1 - (1.5*(t.Q3 - t.Q1)) OR
t.Price > t.Q3 + (1.5*(t.Q3 - t.Q1));

Q9
Question: How do you create price buckets and visualize distribution with asterisks?
Answer:
SELECT t.buckets, REPEAT('*', COUNT(*)/5) FROM (
SELECT price, 
CASE 
    WHEN price BETWEEN 0 AND 25000 THEN '0-25K'
    WHEN price BETWEEN 25001 AND 50000 THEN '25K-50K'
    WHEN price BETWEEN 50001 AND 75000 THEN '50K-75K'
    WHEN price BETWEEN 75001 AND 100000 THEN '75K-100K'
    ELSE '>100K'
END AS 'buckets'
FROM laptops
) t
GROUP BY t.buckets;

Q10
Question: How do you count number of laptops by company?
Answer:
SELECT Company, COUNT(Company) FROM laptops GROUP BY Company;

Q11
Question: How do you analyze relationship between cpu_speed and price?
Answer:
SELECT cpu_speed, Price FROM laptops;

Q12
Question: How do you count touchscreen and non-touchscreen laptops by company?
Answer:
SELECT Company,
SUM(CASE WHEN Touchscreen = 1 THEN 1 ELSE 0 END) AS 'Touchscreen_yes',
SUM(CASE WHEN Touchscreen = 0 THEN 1 ELSE 0 END) AS 'Touchscreen_no'
FROM laptops
GROUP BY Company;

Q13
Question: How do you view distinct CPU brands?
Answer:
SELECT DISTINCT cpu_brand FROM laptops;

Q14
Question: How do you count laptops by CPU brand for each company?
Answer:
SELECT Company,
SUM(CASE WHEN cpu_brand = 'Intel' THEN 1 ELSE 0 END) AS 'intel',
SUM(CASE WHEN cpu_brand = 'AMD' THEN 1 ELSE 0 END) AS 'amd',
SUM(CASE WHEN cpu_brand = 'Samsung' THEN 1 ELSE 0 END) AS 'samsung'
FROM laptops
GROUP BY Company;

Q15
Question: How do you calculate price statistics (MIN, MAX, AVG, STD) grouped by company?
Answer:
SELECT Company, MIN(price), MAX(price), AVG(price), STD(price)
FROM laptops
GROUP BY Company;

Q16
Question: How do you find records where price is NULL?
Answer:
SELECT * FROM laptops WHERE price IS NULL;

Q17
Question: How do you replace missing price values with the overall mean price?
Answer:
UPDATE laptops
SET price = (SELECT AVG(price) FROM laptops)
WHERE price IS NULL;

Q18
Question: How do you replace missing price values with the mean price of the corresponding company?
Answer:
UPDATE laptops l1
SET price = (SELECT AVG(price) FROM laptops l2 WHERE l2.Company = l1.Company)
WHERE price IS NULL;

Q19
Question: How do you add a PPI (Pixels Per Inch) column?
Answer:
ALTER TABLE laptops ADD COLUMN ppi INTEGER;

Q20
Question: How do you calculate and update PPI values using resolution and screen size?
Answer:
UPDATE laptops
SET ppi = ROUND(SQRT(resolution_width*resolution_width + resolution_height*resolution_height)/Inches);

Q21
Question: How do you view laptops sorted by PPI descending?
Answer:
SELECT * FROM laptops ORDER BY ppi DESC;

Q22
Question: How do you add a screen_size categorical column?
Answer:
ALTER TABLE laptops ADD COLUMN screen_size VARCHAR(255) AFTER Inches;

Q23
Question: How do you categorize screen_size as small, medium, or large based on Inches?
Answer:
UPDATE laptops
SET screen_size = 
CASE 
    WHEN Inches < 14.0 THEN 'small'
    WHEN Inches >= 14.0 AND Inches < 17.0 THEN 'medium'
    ELSE 'large'
END;

Q24
Question: How do you calculate average price by screen_size category?
Answer:
SELECT screen_size, AVG(price) FROM laptops GROUP BY screen_size;

Q25
Question: How do you perform one-hot encoding on the gpu_brand column?
Answer:
SELECT gpu_brand,
CASE WHEN gpu_brand = 'Intel' THEN 1 ELSE 0 END AS 'intel',
CASE WHEN gpu_brand = 'AMD' THEN 1 ELSE 0 END AS 'amd',
CASE WHEN gpu_brand = 'nvidia' THEN 1 ELSE 0 END AS 'nvidia',
CASE WHEN gpu_brand = 'arm' THEN 1 ELSE 0 END AS 'arm'
FROM laptops;
