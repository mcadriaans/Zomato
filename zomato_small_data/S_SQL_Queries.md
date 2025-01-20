# Set up Database
```sql
CREATE DATABASE zomato_db;
```
## Create table in database
```sql
CREATE TABLE zomato (
    restaurant_id INT NOT NULL,
    restaurant_name VARCHAR(255) NOT NULL,
    country_code INT NOT NULL,
    city VARCHAR(255) NOT NULL,
    address VARCHAR(255) NOT NULL,
    locality VARCHAR(255) NOT NULL,
    locality_verbose VARCHAR(255) NOT NULL,
    longitude FLOAT NOT NULL,
    latitude FLOAT NOT NULL,
    cuisines VARCHAR(255) NOT NULL,
    average_cost_for_two INT NOT NULL,
    currency VARCHAR(50) NOT NULL,
    has_table_booking VARCHAR(50) NOT NULL,
    has_online_delivery VARCHAR(50) NOT NULL,
    is_delivering_now VARCHAR(50) NOT NULL,
    switch_to_order_menu VARCHAR(50) NOT NULL,
    price_range INT NOT NULL,
    aggregate_rating FLOAT NOT NULL,
    rating_color VARCHAR(50) NOT NULL,
    rating_text VARCHAR(255) NOT NULL,
    votes INT NOT NULL,
    country VARCHAR(255) NOT NULL
);
```
# Business Queries

# Easy
## 1. Total registered restaurants in the data
*Gauges the dataset coverage for analysis*
```sql
SELECT
  COUNT(*) AS nr_of_restaurants
FROM zomato;
```
![image](https://github.com/user-attachments/assets/698c8277-5aab-4347-9352-4f2b3d25be20)


## 2. Which five countries boast the highest number of restaurants listed in the data?
*Pinpoints the top restaurant locations by country.*
```sql
SELECT
  country,
  COUNT(*) AS total_restaurants
FROM zomato
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;
```
![image](https://github.com/user-attachments/assets/b899c2bd-8986-489d-9896-fe37260e4363)



# Advanced
## What are the top cuisines in each country based on the number of restaurants offering them?
*Showcase potential regional cuisine preferences*
```sql
SELECT *
FROM(
    SELECT
      country,
      cuisines,
      COUNT(*) AS total_ordered,
      RANK()OVER(
                 PARTITION BY country
                 ORDER BY COUNT(*) DESC
		)AS rnk
    FROM zomato
    GROUP BY 1, 2)
    WHERE rnk = 1;
```
![image](https://github.com/user-attachments/assets/87945ff7-bf5d-4498-909c-911f6816c807)


## Calculate the average cost for two in each price range for the United States.
*Better Understanding of pricing trends by category*
```sql
SELECT 
  CASE 
    WHEN price_range = 1 THEN '(1) Budget-Friendly'
    WHEN price_range = 2 THEN '(2) Mid-range'
    WHEN price_range = 3 THEN '(3) Upscale'
		ELSE '(4) Fine Dining'
    END AS price_range_decription,
    ROUND(AVG(average_cost_for_two), 2) AS avg_cost_for_two_usd
FROM zomato
WHERE country = 'United States'
GROUP BY 1
ORDER BY 1;
```
![image](https://github.com/user-attachments/assets/edfb4f4e-5bce-4df2-a484-5ce1ed122e19)

# Super Advanced
## For countries with table booking services available, which country has the highest average aggregate rating for restaurants offering online delivery?
*Valuable for targeting countries with high-rated, dual service restauarnts*
```sql
WITH table_bookings AS (
  SELECT *
  FROM zomato
  WHERE has_table_booking = 'Yes'
)
SELECT 
  country,
  ROUND(AVG(aggregate_rating) :: numeric ,2) AS avg_overall_rating
FROM table_bookings
WHERE has_online_delivery = 'Yes'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 1;
```
![image](https://github.com/user-attachments/assets/61d71ede-6827-4b5d-8157-d5be0260ba89)

# Expert level
## Identify cities where restaurants with table booking have significantly higher average ratings compared to those without table booking. Calculate the percentage difference for each city.
*Highlights the impact of table booking services on customer satisfaction*
```sql
WITH avg_rt_compare AS(
SELECT 
 city,
 ROUND(AVG(CASE WHEN has_table_booking = 'Yes' THEN aggregate_rating END):: numeric,2) AS avg_has_tb_rating,
 ROUND(AVG(CASE WHEN has_table_booking = 'No' THEN aggregate_rating END) :: numeric,2) AS avg_no_tb_rating
FROM zomato
GROUP BY 1
)
SELECT 
 city, 
 avg_has_tb_rating,  
 avg_no_tb_rating, 
 ROUND(((avg_has_tb_rating - avg_no_tb_rating) / avg_no_tb_rating) * 100, 2) AS percentage_difference 
FROM avg_rt_compare
WHERE avg_has_tb_rating IS NOT NULL 
 AND avg_no_tb_rating IS NOT NULL 
 AND avg_has_tb_rating > avg_no_tb_rating
ORDER BY 4 DESC;
```
![image](https://github.com/user-attachments/assets/a0079ad9-e2a3-4f97-afef-4e73d54abde2)
### 🔍 Observations 
Noida has the highest percentage difference at 56.25%, indicating that restaurants with table bookings have notably higher average ratings than those without.
