# Create table in database
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
# Understanding the data
## 1. Total registered restaurants in the data
```sql
SELECT
	COUNT(*) AS nr_of_restaurants
FROM zomato;
```
![image](https://github.com/user-attachments/assets/698c8277-5aab-4347-9352-4f2b3d25be20)
