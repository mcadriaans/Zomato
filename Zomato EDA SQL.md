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
