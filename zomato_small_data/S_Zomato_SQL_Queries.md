# Set up Database
```sql
CREATE DATABASE zomato_db;
```
## Create table in database
DATASET: path = kagglehub.dataset_download("shrutimehta/zomato-restaurants-data")
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
## 1. Total registered restaurants in the data
```sql
SELECT
  COUNT(*) AS nr_of_restaurants
FROM zomato;
```
![image](https://github.com/user-attachments/assets/698c8277-5aab-4347-9352-4f2b3d25be20)


## 2. Which five countries boast the highest number of restaurants listed in the Zomato database?
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


## 3. Which city in each country has the highest number of restaurants listed in the Zomato database?
```sql
SELECT 
  country,
  city,
  total_restaurants
FROM (
  SELECT
    country,
    city,
    COUNT(*) AS total_restaurants,
    RANK()OVER(
               PARTITION BY country
               ORDER BY COUNT(*) DESC
		) AS rnk
  FROM zomato
  GROUP BY 1, 2)
WHERE rnk = 1;
```
![image](https://github.com/user-attachments/assets/687119e2-cf06-4bbc-9e74-b93f97647cc9)


## 4. Identify the most popular cuisine in each Country.
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


## 5. Calculate the average cost for two in each price range.
```sql
SELECT 
  price_range,
  ROUND(AVG(average_cost_for_two), 2) AS avg_cost_for_two
FROM zomato
GROUP BY 1
ORDER BY 1;
```
