# SQL Project: Data Analysis for Zomato - A Food Delivery Company

## Overview

This project demonstrates my SQL problem-solving skills through the analysis of data for Zomato, a popular food delivery company in India. The project involves setting up the database, importing data, handling null values, and solving a variety of business problems using complex SQL queries.

## Project Structure

- **Database Setup:** Creation of the `zomato_db` database and the required tables.
- **Data Import:** Inserting sample data into the tables.
- **Data Cleaning:** Handling null values and ensuring data integrity.
- **Business Problems:** Solving 20 specific business problems using SQL queries.

## Database Setup
```sql
CREATE DATABASE zomato_db;
```
### 1. Create Tables
#### Create Dimension(child) table : customers
```sql
CREATE TABLE customers(
			 customer_id INT PRIMARY KEY,
			 customer_name VARCHAR(25),
			 reg_date DATE
);
```
#### Create Dimension(child) table : restaurants
```sql
CREATE TABLE restaurants(
			 restaurant_id INT PRIMARY KEY,
			 restaurant_name VARCHAR(50),
			 city VARCHAR(15),
			 opening_hours VARCHAR(55)
);
```
#### Create Dimension(child) table : riders
```sql
CREATE TABLE riders(
			 rider_id INT PRIMARY KEY,
			 rider_name	VARCHAR(55),
			 sign_up DATE
);
```
#### Create Fact(parent) table : orders
```sql
CREATE TABLE orders(
     		 order_id INT PRIMARY KEY,
			 customer_id INT,
			 restaurant_id INT,
			 order_item VARCHAR(55),
			 order_date DATE,
			 order_time TIME,
			 order_status VARCHAR(25),
			 total_amount DECIMAL(10,2),
			 FOREIGN KEY(customer_id) REFERENCES customers(customer_id),
			 FOREIGN KEY(restaurant_id) REFERENCES restaurants(restaurant_id)
);
```
#### Create Fact(parent) table : deliveries
```sql
CREATE TABLE deliveries(
    		 delivery_id INT PRIMARY KEY,
			 order_id INT,
			 delivery_status VARCHAR(35),
			 delivery_time TIME,
			 rider_id INT,
			 FOREIGN KEY(order_id) REFERENCES orders(order_id),
			 FOREIGN KEY(rider_id) REFERENCES riders(rider_id)
);
```

## Data Import

## Data Cleaning and Handling Null Values
### Check for Null values
Missing Values can impact the results of the Queries
```sql
SELECT 
	COUNT(*)
FROM customers
WHERE reg_date IS NULL 
	OR customer_name IS NULL;
```
```sql
SELECT 
	COUNT(*)
FROM restaurants
WHERE restaurant_name IS NULL 
	OR city IS NULL
	OR opening_hours IS NULL;
```
```sql
SELECT 
	COUNT(*)
FROM orders
WHERE order_item IS NULL 
	OR order_date IS NULL
	OR order_time IS NULL
	OR order_status IS NULL
	OR total_amount IS NULL;
```
```sql
SELECT 
	 COUNT(*)
FROM riders
WHERE rider_name IS NULL 
	OR sign_up IS NULL;
```
```sql
SELECT *
FROM deliveries
WHERE delivery_status IS NULL 
	OR delivery_time IS NULL;
```
❗There are 797 missing delivery time values out of 9750 records (approx 8%) :Delivery time is NULL when order has not been delivered or when order has just been placed. We do not have to handle these missing values as they contribute to the data.

## Business Problems Solved
### 1. Customer Specific Information : Find the Top 5 most frequently ordered dishes by customer called "Arjun Mehta" in the last year
```sql
SELECT 
	order_item AS dishes,
	total_orders
FROM (
	SELECT 
		o.order_item,
		COUNT(*) AS total_orders,
		DENSE_RANK()OVER(
			ORDER BY COUNT(*) DESC
		) AS rnk
	FROM orders as o
	JOIN customers as c
		ON o.customer_id = c.customer_id
	WHERE customer_name ='Arjun Mehta'
		AND o.order_date >= CURRENT_DATE - INTERVAL '1 Year'
	GROUP BY 1
	) AS t1
	WHERE rnk <= 5;
```
![image](https://github.com/user-attachments/assets/b49362f1-d215-4381-8730-a2cdc63da944)

### 2. Popular Time slots : Identify the time slots during which the most orders are placed. Time slots are defined by  2-hour intervals
```sql
SELECT 
	CASE 
		WHEN EXTRACT(HOUR FROM order_time) BETWEEN 0 AND 1 THEN '0:00 - 1:59 AM'
		WHEN EXTRACT(HOUR FROM order_time) BETWEEN 2 AND 3 THEN '2:00 - 3:59 AM'
		WHEN EXTRACT(HOUR FROM order_time) BETWEEN 4 AND 5 THEN '4:00 - 5:59 AM'
		WHEN EXTRACT(HOUR FROM order_time) BETWEEN 6 AND 7 THEN '6:00 - 7:59 AM'
		WHEN EXTRACT(HOUR FROM order_time) BETWEEN 8 AND 9 THEN '8:00 - 9:59 AM'
		WHEN EXTRACT(HOUR FROM order_time) BETWEEN 10 AND 11 THEN '10:00 - 11:59 AM'
		WHEN EXTRACT(HOUR FROM order_time) BETWEEN 12 AND 13 THEN '12:00 - 13:59 PM'
		WHEN EXTRACT(HOUR FROM order_time) BETWEEN 14 AND 15 THEN '14:00 - 15:59 PM'
		WHEN EXTRACT(HOUR FROM order_time) BETWEEN 16 AND 17 THEN '16:00 - 17:59 PM'
		WHEN EXTRACT(HOUR FROM order_time) BETWEEN 18 AND 19 THEN '18:00 - 19:59 PM'
		WHEN EXTRACT(HOUR FROM order_time) BETWEEN 20 AND 21 THEN '20:00 - 21:59 PM'
		WHEN EXTRACT(HOUR FROM order_time) BETWEEN 22 AND 23 THEN '22:00 - 23:59 PM'
		ELSE '0:00 - 1:59 AM'
	END AS time_slots,
	COUNT(*) AS orders_placed
FROM orders
GROUP BY 1
ORDER BY 2 DESC;
```
![image](https://github.com/user-attachments/assets/126fe457-7f0a-4f1b-8fd4-d1687b8bae7f)

### 3. Order value Analysis :Find the average order value per customer who has placed more than 750 orders. 
Return customer_name, and aov (average order value)
```sql
SELECT 
	c.customer_name,
	ROUND(AVG(o.total_amount),2) AS avg_order_value
FROM orders AS o
JOIN customers AS c
	ON o.customer_id = c.customer_id
GROUP BY 1
HAVING COUNT(*) > 750
ORDER BY 2 DESC;
```
![image](https://github.com/user-attachments/assets/dba6eca5-07fa-4408-a5d1-6f6b3e0b6886)

### 4. High Value Customers : List the customers who have spent more than 100k in total on food orders.
Return customer_name and customer_id.
```sql
SELECT 
    c.customer_name,
	o.customer_id
FROM orders AS o
JOIN customers AS c
	ON o.customer_id = c.customer_id
GROUP BY 1, 2
HAVING SUM(o.total_amount) > 100000.00
ORDER BY 2 ASC;
```
![image](https://github.com/user-attachments/assets/dc712d94-c81d-42ec-bf06-c1dbf9681037)

### 5. Orders without Delivery : Find the restaurants which had orders placed but not delivered.
Return each restaurant name, city and number of not delivered orders.
```sql
SELECT 
	r.restaurant_name,
	r.city,
	COUNT(*) AS cnt_not_delivered
FROM orders AS o
LEFT JOIN deliveries AS d
	ON o.order_id = d.order_id
JOIN restaurants AS r
	ON o.restaurant_id = r.restaurant_id
WHERE delivery_id IS NULL
GROUP BY 1, 2
ORDER BY 3 DESC;
```
![image](https://github.com/user-attachments/assets/f831ca27-51fc-4580-9c9f-f618c6de8729)

### 6. Restaurant Revenue Ranking : Which restaurants in each city generated the highest total revenue within the last year?
* Rank restaurants by their Total revenue from the last year.
* Return the best ranked restaurant name, city and total revenue for each city.
```sql
WITH restaurants_ranked AS(
	SELECT 
		r.restaurant_name,
		r.city,
		SUM(total_amount) AS total_revenue,
		RANK()OVER(
				PARTITION BY city
				ORDER BY SUM(total_amount) DESC
		) AS city_rank
	FROM orders AS o
	JOIN restaurants AS r
		ON o.restaurant_id = r.restaurant_id
	WHERE order_date >= CURRENT_DATE - INTERVAL '1 Year'
	GROUP BY 1, 2)
SELECT 
	restaurant_name,
	city,
	total_revenue
FROM restaurants_ranked
WHERE city_rank = 1;
```
![image](https://github.com/user-attachments/assets/414d9bbd-cc82-40e9-8c97-8ed300a78406)

### 7. Most Popular Dish by City : What is the most popular dish in each city based on the number of orders?
```sql
WITH dish_cty_ranks AS(
	SELECT 
		city,
		order_item,
		COUNT(*) AS cnt_orders,
		RANK()OVER(
			PARTITION BY city
			ORDER BY COUNT(*) DESC
		) AS rnk
	FROM orders AS o
	JOIN restaurants AS r
		ON o.restaurant_id = r.restaurant_id
	GROUP BY 1, 2
)
SELECT 
	city,
	order_item AS dish
FROM dish_cty_ranks
WHERE rnk = 1;
```
![image](https://github.com/user-attachments/assets/73cc2672-9e31-4d1d-a5a4-5dfb00c12cc2)

### 8. Customer Churn: Find the customers who have not placed an order in 2024 but did in 2023.
* Return customer_id and customer_name

```sql
SELECT 
	DISTINCT(o.customer_id),
	c.customer_name
FROM orders AS o
JOIN customers AS c
	ON o.customer_id = c.customer_id
WHERE EXTRACT(Year from o.order_date) = 2023
	AND o.customer_id NOT IN (
		SELECT
			DISTINCT(customer_id)
		FROM orders
		WHERE EXTRACT(Year from order_date) = 2024
);
```
![image](https://github.com/user-attachments/assets/858797bc-bf83-4648-a396-25a6a02ea34e)

### 9. Cancelation Rate Comparison : Calculate and compare the order cancellation rate for each restaurant for the past year.
* Return restaurant_name, restaurant_id , total number of orders, total number of cancellations and the cancellation rate.
```sql
WITH current_year_cancellations AS (
	SELECT
		o.restaurant_id,
		COUNT(CASE WHEN d.delivery_id IS NULL THEN 1 END) AS total_cancellations,
		COUNT(o.order_id) AS total_orders,
		ROUND((COUNT(CASE WHEN d.delivery_id IS NULL THEN 1 END)::numeric / COUNT(o.order_id)::numeric) * 100,2) AS cancellation_rate
	FROM orders AS o
	LEFT JOIN deliveries AS d
		ON o.order_id = d.order_id
	WHERE EXTRACT(Year FROM order_date) = EXTRACT(Year FROM CURRENT_DATE)
	GROUP BY 1
),
previous_year_cancellations AS (
SELECT
	o.restaurant_id,
	COUNT(CASE WHEN d.delivery_id IS NULL THEN 1 END) AS total_cancellations,
	COUNT(o.order_id) AS total_orders,
	ROUND((COUNT(CASE WHEN d.delivery_id IS NULL THEN 1 END)::numeric / COUNT(o.order_id)::numeric) * 100,2) AS cancellation_rate
FROM orders AS o
LEFT JOIN deliveries AS d
	ON o.order_id = d.order_id
WHERE EXTRACT(Year FROM order_date) = EXTRACT(Year FROM CURRENT_DATE - INTERVAL '1 Year')
GROUP BY 1
)

SELECT 
	c.restaurant_id,
	r.restaurant_name,
	c.cancellation_rate AS cancellation_rate_2023,
	p.cancellation_rate AS cancellation_rate_2024
FROM current_year_cancellations AS c
JOIN previous_year_cancellations AS p
	ON c.restaurant_id = p.restaurant_id
JOIN restaurants AS r
	ON c.restaurant_id = r.restaurant_id;
```
![image](https://github.com/user-attachments/assets/4a47ffd3-a668-49b4-bf62-05227bda1287)

### 10. Rider Average Delivery Time: Determine the average time it takes each rider to deliver an order.
* Return the rider_id, rider_name and their average delivery time.
```sql
SELECT
	d.rider_id,
	r.rider_name,
	AVG((d.delivery_time - o.order_time)::TIME) AS avg_delivery_time
FROM orders AS o
LEFT JOIN deliveries AS d
    ON o.order_id = d.order_id
JOIN riders AS r
	ON d.rider_id = r.rider_id
WHERE d.delivery_status = 'Delivered'
GROUP BY 1, 2
ORDER BY 1;
```
![image](https://github.com/user-attachments/assets/7862ab89-5b54-4008-9400-751d6e3897ea)

### 11. Monthly Restaurant Growth Rate: Calculate each restaurant's growth rate based on the total number of delivered orders since its joining.
```sql
WITH restaurant_monthly_orders AS(
	SELECT 
		o.restaurant_id,
		TO_CHAR(o.order_date, 'mm-yy') AS month_date,
		COUNT(*) AS current_monthly_orders
	FROM orders AS o
	JOIN deliveries AS d
		ON o.order_id = d.order_id
	WHERE d.delivery_status = 'Delivered'
	GROUP BY 1, 2
	ORDER BY 1, 2
),
restaurant_performance_over_time AS(
	SELECT 
		restaurant_id,
		TO_DATE(month_date, 'mm_yy')AS monthly_date,
		current_monthly_orders,
		LAG(current_monthly_orders, 1)OVER(
					PARTITION BY restaurant_id
					ORDER BY TO_DATE(month_date, 'mm_yy')
		)AS previous_monthly_orders
	FROM restaurant_monthly_orders
	ORDER BY 1, 2
)
SELECT 
	r1.restaurant_id,
	r2.restaurant_name,
	r1.monthly_date,
	r1.previous_monthly_orders,
	r1.current_monthly_orders,
	ROUND((r1.current_monthly_orders::numeric - r1.previous_monthly_orders::numeric) / r1.previous_monthly_orders::numeric * 100, 2) AS growth_rate
FROM restaurant_performance_over_time AS r1
JOIN restaurants AS r2
	ON r1.restaurant_id = r2.restaurant_id;
```
![image](https://github.com/user-attachments/assets/872a937d-d177-4db4-8ebd-6eb28bf78c2d)

### 12. Customer Segmentation: Segment customers into 'Gold' or 'Silver' groups based on their total spending compared to the average order value (AOV). 
* If a customer's total spending exceeds the AOV, label them as 'Gold'; otherwise label them as 'Silver'.
* Write a SQL query to determine each segment's total number of orders and total revenue.

```sql
WITH segmented_customers AS
(
	SELECT 
		customer_id,
		SUM(o.total_amount) AS total_spending,
		CASE
			WHEN SUM(o.total_amount) > AVG(o.total_amount) THEN 'Gold'
			ELSE 'Silver'
		END AS customer_segment 
	FROM orders AS o
	GROUP BY 1
	ORDER BY 1
)
SELECT 
	s.customer_segment,
	COUNT(o.*) AS total_orders,
	SUM(o.total_amount) AS total_revenue
FROM segmented_customers AS s
JOIN orders AS o
	ON s.customer_id = o.customer_id
GROUP BY 1;
```
![image](https://github.com/user-attachments/assets/34ac4654-0c3c-4eb8-849c-462bdcf692f2)

### 13. Rider Monthly Earnings : Calculate each rider's total monthly earnings,assuming they earn 8% of the order amount.
```sql
SELECT 
	r.rider_id,
	r.rider_name,
	TO_DATE(TO_CHAR(o.order_date, 'mm_yy'), 'mm-yy') AS month,
	SUM(o.total_amount) AS total_revuenue,
	SUM(o.total_amount) * 0.08 AS rider_monthly_earnings
FROM orders AS o
LEFT JOIN deliveries AS d
	ON o.order_id = d.order_id
JOIN riders AS r
	ON d.rider_id = r.rider_id
WHERE d.delivery_status = 'Delivered'
GROUP BY 1, 2, 3
ORDER BY 1, 3;
```
![image](https://github.com/user-attachments/assets/4b042875-d894-49cc-b3dd-5df8e39d76c5)

### 14. Rider Ratings Analysis:  Find the number of 5-star, 4-star and 3-star  ratings each rider has. Riders receive these ratings based on delivery time.
* If order is  delivered less than 15 min of order received time : 5-star
* If order is  delivered between 15 and 20 min of order received time: 4-star
* If order is delivered after 20 min : 3-star

```sql
WITH rider_ratings AS (
	SELECT
		r.rider_id,
		r.rider_name,
		o.order_time,
		d.delivery_time,
		(d.delivery_time - o.order_time) :: TIME as delivery_duration,
		CASE
			WHEN (d.delivery_time - o.order_time) :: TIME < '00:15:00' THEN '5-star'
			WHEN (d.delivery_time - o.order_time) :: TIME BETWEEN '00:15:00' AND '00:20:00' THEN '4-star'
			ELSE '3-star'
		END AS rider_rating
	FROM orders AS o
	LEFT JOIN deliveries AS d
		ON o.order_id = d.order_id
	JOIN riders AS r
		ON d.rider_id = r.rider_id
	WHERE d.delivery_status = 'Delivered'
	ORDER BY 1
)
SELECT 
	rider_id,
	rider_name,
	COUNT(CASE WHEN rider_rating = '5-star' THEN 1 END) AS nr_5_star_ratings,
	COUNT(CASE WHEN rider_rating = '4-star' THEN 1 END) AS nr_4_star_ratings,
	COUNT(CASE WHEN rider_rating = '3-star' THEN 1 END) AS nr_3_star_ratings
FROM rider_ratings
GROUP BY 1, 2
ORDER BY 1;
```
![image](https://github.com/user-attachments/assets/cc5e1711-b079-40e7-b4d5-bfcfe321c1d8)

### 15. Order Frequency by Day: Analyze order frequency per day of the week and identify the peak day for each restaurant.
```sql
SELECT 
	t1.restaurant_id,
	t1.restaurant_name,
	t1.weekday AS peak_day,
	t1.order_frequency AS cnt_orders
FROM (
	SELECT 
		r.restaurant_id,
		r.restaurant_name,
		TO_CHAR(o.order_date, 'Day') AS weekday,
		COUNT(*) AS order_frequency,
		DENSE_RANK()OVER(
			PARTITION BY r.restaurant_name
			ORDER BY COUNT(*) DESC
		) AS rnk
	FROM orders AS o
	JOIN restaurants AS r
		ON o.restaurant_id = r.restaurant_id
	GROUP BY 1, 2, 3
	ORDER BY 1, 5) AS t1
WHERE rnk = 1;
```
![image](https://github.com/user-attachments/assets/12bf61e5-c66f-48b9-b8c3-d00b462b2cc3)

### 16. Customer Lifetime Value (CLV): Calculate the Total Revenue generated by each customer over all their orders.
```sql
SELECT
	c.customer_id,
	c.customer_name,
	SUM(o.total_amount) AS clv
FROM orders AS o
JOIN customers AS c
	ON o.customer_id = c.customer_id
GROUP BY 1, 2
ORDER BY 1;
```
![image](https://github.com/user-attachments/assets/14072031-a88e-456c-8025-e7a571c8f21f)

### 17. Monthly Sales Trends : Identify sales trends by comparing each months total sales by the previous month.
```sql
WITH monthly_sales AS (
	SELECT
		TO_DATE(TO_CHAR(order_date, 'mm-yy'), 'mm-yy') AS month_year,
		SUM(total_amount) AS current_month_sales,
		LAG(SUM(total_amount), 1)OVER(
			ORDER BY TO_DATE(TO_CHAR(order_date, 'mm-yy'), 'mm-yy') 
		) AS previous_month_sales
	FROM orders
	GROUP BY 1
)
SELECT 
	*,
	ROUND((current_month_sales - previous_month_sales )/ previous_month_sales * 100 ,2) AS sales_growth_rate
FROM monthly_sales;
```
![image](https://github.com/user-attachments/assets/b3cb41ba-ab06-4b54-822c-d65ca2b14a78)

### 18. Rider Efficiency : Evaluate the rider efficiency by determining the average delivery times and identifying those riders with the highest and lowest average.
```sql
WITH rider_delivery_times AS(
	SELECT 
		r.rider_id,
		r.rider_name,
		AVG((d.delivery_time - o.order_time)::TIME) AS avg_delivery
	FROM orders AS o 
	LEFT JOIN deliveries AS d USING(order_id)
	JOIN riders AS r USING(rider_id)
	WHERE d.delivery_status = 'Delivered'
	GROUP BY 1, 2
	ORDER BY 3 DESC
)
(SELECT *
FROM rider_delivery_times
WHERE avg_delivery = (SELECT MAX(avg_delivery) FROM rider_delivery_times))
UNION
(SELECT *
FROM rider_delivery_times
WHERE avg_delivery = (SELECT MIN(avg_delivery) FROM rider_delivery_times));
```
![image](https://github.com/user-attachments/assets/5f1198dc-57e0-4a8d-833e-6e3cb6eecfb6)

### 19. Order Item Popularity: Track the popularity of specific order items over time and identify seasonal demand spikes.
```sql
SELECT 
	CASE
		WHEN EXTRACT (Month FROM order_date) IN (3,4,5) THEN 'Summer'
		WHEN EXTRACT (Month FROM order_date) IN (6,7,8,9,10) THEN 'Monsoon'
		WHEN EXTRACT (Month FROM order_date) IN (11,12,1,2) THEN 'Winter'
	END AS Season,
	order_item,
	COUNT(order_item) AS cnt
FROM orders 
GROUP BY 1, 2
ORDER BY 2;
```
![image](https://github.com/user-attachments/assets/c1c01894-6f6f-4cc3-8e93-26654b3b38cb)

### 20. Rank each City based on their total revenue for 2023
```sql
SELECT
	r.city,
	SUM(o.total_amount) AS total_revenue,
	RANK()OVER(
		ORDER BY SUM(o.total_amount) DESC
	) AS city_rank
FROM orders AS o
JOIN restaurants AS r USING (restaurant_id)
WHERE EXTRACT(Year FROM o.order_date) = '2023'
GROUP BY 1;
```
![image](https://github.com/user-attachments/assets/6490328c-9d72-45f5-b1af-5692cb8aa18f)
