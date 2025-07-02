# Zomato-SQL-Project
This project performs a comprehensive analysis of Zomato's food delivery data using **MySQL**. The focus is on customer behavior, restaurant performance, rider efficiency, and overall business KPIs — answered through 20 real-world business queries.

---

## 🧰 Tools Used

- 🐬 **MySQL** – for data extraction, analysis, and business intelligence
- 📂 **CSV Datasets** – Customers, Orders, Restaurants, Riders, Deliveries

---
## 📁 Dataset Overview

The project is based on five relational tables:
- **Customers** – customer details and registration dates  
- **Orders** – order metadata including items, time, status, amount  
- **Restaurants** – name, city, and operating hours  
- **Deliveries** – delivery status, time, and rider info  
- **Riders** – delivery agents and sign-up details  

---

## 🎯 Project Objectives

- Analyze customer spending, churn, and segmentation  
- Track order trends, cancellations, and most popular dishes  
- Evaluate restaurant performance by city, revenue, and growth  
- Assess rider performance, earnings, and efficiency  
- Identify sales trends and seasonal demand

---

## CREATING DATABASE
```sql
CREATE DATABASE zomato_db;
USE zomato_db;
```
## TABLE CREATION

```sql
CREATE TABLE Customers
    (
   customer_id INT PRIMARY KEY,
	customer_name VARCHAR (30),
	reg_date DATE
    );

CREATE TABLE Restaurants
	(
    restaurant_id INT PRIMARY KEY,
	restaurant_name VARCHAR (50),
	city VARCHAR (25),
    opening_hours VARCHAR (55)
    );

CREATE TABLE Orders
	(
    order_id INT PRIMARY KEY,	
    customer_id INT,
	restaurant_id INT,
	order_item VARCHAR (55),
	order_date DATE,
	order_time TIME,
	order_status VARCHAR (30),
	total_amount FLOAT,
    CONSTRAINT fk_customers FOREIGN KEY (customer_id) REFERENCES customers (customer_id),
    CONSTRAINT fk_restaurants FOREIGN KEY (restaurant_id) REFERENCES restaurants (restaurant_id)
    );
    
CREATE TABLE Riders
	(
    rider_id INT PRIMARY KEY,
	rider_name VARCHAR (30),
	sign_up DATE
    );
    
CREATE TABLE Deliveries
	(
    delivery_id INT PRIMARY KEY,
	order_id INT,
	delivery_status VARCHAR (30),
	delivery_time TIME,
	rider_id INT,
    CONSTRAINT fk_orders FOREIGN KEY (order_id) REFERENCES Orders (order_id),
    CONSTRAINT fk_riders FOREIGN KEY (rider_id) REFERENCES riders (rider_id)
    );
```

## DATA IMPORT

## DATA CLEANING AND HANDLING NULL VALUES
```sql
SELECT * FROM customers
WHERE customer_name IS NULL 
OR reg_date IS NULL;

SELECT * FROM restaurants
WHERE restaurant_name IS NULL 
OR city IS NULL 
OR opening_hours IS NULL;

SELECT * FROM orders
WHERE customer_id IS NULL	
OR restaurant_id IS NULL	
OR order_item IS NULL	
OR order_date IS NULL	
OR order_time IS NULL	
OR order_status IS NULL	
OR total_amount IS NULL;

SELECT * FROM riders
WHERE rider_name IS NULL	
OR sign_up IS NULL;

SELECT * FROM deliveries
WHERE order_id IS NULL	
OR delivery_status IS NULL	
OR delivery_time IS NULL	
OR rider_id IS NULL;
```
## BUSINESS PROBLEMS 

### TOP 5 MOST FREQUENTLY ORDERED DISHES BY ARJUN MEHTA
-- 1. Write a query to find the top 5 most frequently ordered dishes by the customer "Arjun Mehta" in the last 2 year.
```sql
WITH Top5dish AS (
SELECT c.customer_name, o.order_item, COUNT(*) AS Total_Orders,
DENSE_RANK() OVER (ORDER BY COUNT(*) DESC) AS rnk
FROM orders AS o
JOIN customers AS c ON o.customer_id = c.customer_id
WHERE c.customer_name = 'Arjun Mehta' 
                AND
o.order_date >= CURDATE() - INTERVAL 2 YEAR
GROUP BY c.customer_name, o.order_item)
SELECT  customer_name, order_item, Total_Orders
FROM Top5dish
WHERE rnk <= 5;
```
### POPULAR TIME SLOTS
-- 2. Identify the time slots during which the most orders are placed, based on 2-hour intervals.
```sql
SELECT
     CASE 
        WHEN HOUR(order_time) BETWEEN 0 AND 1 THEN '12:00 AM - 2:00 AM'
        WHEN HOUR(order_time) BETWEEN 2 AND 3 THEN '2:00 AM - 4:00 AM'
        WHEN HOUR(order_time) BETWEEN 4 AND 5 THEN '4:00 AM - 6:00 AM'
        WHEN HOUR(order_time) BETWEEN 6 AND 7 THEN '6:00 AM - 8:00 AM'
        WHEN HOUR(order_time) BETWEEN 8 AND 9 THEN '8:00 AM - 10:00 AM'
        WHEN HOUR(order_time) BETWEEN 10 AND 11 THEN '10:00 AM - 12:00 PM'
        WHEN HOUR(order_time) BETWEEN 12 AND 13 THEN '12:00 PM - 2:00 PM'
        WHEN HOUR(order_time) BETWEEN 14 AND 15 THEN '2:00 PM - 4:00 PM'
        WHEN HOUR(order_time) BETWEEN 16 AND 17 THEN '4:00 PM - 6:00 PM'
        WHEN HOUR(order_time) BETWEEN 18 AND 19 THEN '6:00 PM - 8:00 PM'
        WHEN HOUR(order_time) BETWEEN 20 AND 21 THEN '8:00 PM - 10:00 PM'
        WHEN HOUR(order_time) BETWEEN 22 AND 23 THEN '10:00 PM - 12:00 AM'
     END AS Time_slot,
     COUNT(order_id) AS Total_order
FROM orders
GROUP BY Time_slot
ORDER BY Total_order DESC
LIMIT 1;
```
### ORDER VALUE ANALYSIS
-- 3. Find the average order value per customer who has placed more than 750 orders. Return: customer_name, aov (average order value).
```sql
SELECT c.customer_id,
       c.customer_name,
       ROUND(AVG(o.total_amount),2) AS AOV,
       COUNT(o.order_id) AS Total_orders
FROM orders AS o
JOIN customers AS c
ON c.customer_id = o.customer_id
GROUP BY c.customer_id, customer_name               
HAVING Total_orders > 750;
```
### HIGH-VALUE CUSTOMERS
-- 4. List the customers who have spent more than 100K in total on food orders. return customer_name, and customer_id
```sql
SELECT c.customer_id, 
       c.customer_name, 
       ROUND(SUM(o.total_amount)) AS Total_amt
FROM customers AS c
JOIN orders AS o
ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name
HAVING Total_amt > 100000
ORDER BY Total_amt DESC;
```
### ORDERS WITHOUT DELIVERY
-- 5. Write a query to find orders that were placed but not delivered. 
-- Return each restuarant name, city and number of not delivered orders
```sql
SELECT r.restaurant_id, r.restaurant_name, r.city , SUM(CASE WHEN delivery_status="not delivered" THEN 1 ELSE 0 END ) AS Not_delivered_orders
FROM orders AS o JOIN restaurants AS r
ON o.restaurant_id = r.restaurant_id
JOIN deliveries AS d
ON d.order_id = o.order_id
WHERE order_status = "Completed" AND delivery_status="Not Delivered"
GROUP BY r.restaurant_id, r.restaurant_name, r.city
ORDER BY Not_delivered_orders DESC;

CREATE INDEX idx_orderstatus ON orders(order_status);
CREATE INDEX idx_deliverystatus ON deliveries(delivery_status);
```
### RESTAURANT REVENUE RANKING
-- 6. Rank restaurants by their total revenue from the last 2 year. 
-- Return: restaurant_name, total_revenue, and their rank within their city.
```sql
SELECT r.restaurant_id, r.restaurant_name, r.city, SUM(total_amount) AS Revenue,
DENSE_RANK() OVER( PARTITION BY city ORDER BY SUM(total_amount)  DESC ) AS rnk
FROM orders AS o JOIN restaurants AS r
ON o.restaurant_id = r.restaurant_id
WHERE o.order_date >= CURDATE() - INTERVAL 2 YEAR
GROUP BY r.restaurant_id, r.restaurant_name, r.city;

CREATE INDEX idx_orderdate ON orders(order_date);
```
### MOST POPULAR DISH BY CITY
-- 7. Identify the most popular dish in each city based on the number of orders.
```sql
WITH DishPopularity AS (
  SELECT 
    r.city,
    o.order_item,
    COUNT(*) AS Total_orders,
    DENSE_RANK() OVER (PARTITION BY r.city ORDER BY COUNT(*) DESC) AS rnk
  FROM orders AS o
  JOIN restaurants AS r ON o.restaurant_id = r.restaurant_id
  GROUP BY r.city, o.order_item
)
SELECT 
  city,
  order_item AS Most_Popular_Dish,
  Total_orders
FROM DishPopularity
WHERE rnk = 1
ORDER BY city;
```
### CUSTOMER CHURN
-- 8. Find customers who haven’t placed an order in 2024 but did in 2023.
```sql
SELECT c.customer_id,
       c.customer_name
FROM customers AS c
JOIN orders AS o 
ON c.customer_id = o.customer_id
WHERE YEAR(o.order_date) = 2023
  AND c.customer_id NOT IN (
        SELECT customer_id
        FROM orders
        WHERE YEAR(order_date) = 2024
        )
GROUP BY c.customer_id, c.customer_name;
```
### CANCELLATION RATE PER RESTAURANT 
-- 9. Calculate the cancellation rate for each restaurant between the 2023 and 2024.
```sql
WITH cte AS (
  SELECT r.restaurant_id, r.restaurant_name, COUNT(o.order_id) AS cancellation_count
  FROM orders o
  INNER JOIN restaurants r 
  ON r.restaurant_id = o.restaurant_id
  WHERE order_status = 'Not Fulfilled' 
    AND YEAR(order_date) BETWEEN 2023 AND 2024
  GROUP BY r.restaurant_id, r.restaurant_name
),
total_count AS (
  SELECT r.restaurant_id, r.restaurant_name, COUNT(o.order_id) AS total_counts
  FROM orders o
  INNER JOIN restaurants r 
  ON r.restaurant_id = o.restaurant_id
  WHERE YEAR(order_date) BETWEEN 2023 AND 2024
  GROUP BY r.restaurant_id, r.restaurant_name
)
SELECT 
  cte.restaurant_id,
  cte.restaurant_name,
  CONCAT(ROUND(cte.cancellation_count * 100 / total_count.total_counts, 2), ' %') AS cancellation_rate
FROM cte 
INNER JOIN total_count 
ON cte.restaurant_id = total_count.restaurant_id
ORDER BY cancellation_rate DESC;
```
### RIDER AVERAGE DELIVERY TIME
-- 10. Determine each rider's average delivery time.
```sql
WITH cte AS (
  SELECT 
    riders.rider_id,
    rider_name,
    CASE 
      WHEN delivery_time > order_time 
        THEN ROUND(TIMESTAMPDIFF(MINUTE, order_time, delivery_time), 2)
     ELSE ROUND(TIMESTAMPDIFF(MINUTE, order_time, delivery_time + INTERVAL 1 DAY), 2)
    END AS delivery_min
  FROM riders 
  LEFT JOIN deliveries ON riders.rider_id = deliveries.rider_id
  INNER JOIN orders ON orders.order_id = deliveries.order_id
)

SELECT 
  rider_id,
  rider_name,
  ROUND(AVG(delivery_min), 2) AS Avg_delivery_time_min
FROM cte
GROUP BY rider_id, rider_name
ORDER BY avg_delivery_time_min DESC;
```
### MONTHLY RESTAURANT GROWTH RATIO
-- 11. Calculate each restaurant's growth ratio for monthly, based on the total number of delivered orders since its joining.
```sql
WITH cte AS (
  SELECT 
    r.restaurant_id,
    r.restaurant_name,
    DATE_FORMAT(o.order_date, "%m/%Y") AS Month_no,
    COUNT(o.order_id) AS total_orders,
    LAG(COUNT(o.order_id), 1) OVER (
      PARTITION BY r.restaurant_id 
      ORDER BY DATE_FORMAT(o.order_date, "%m/%Y")
    ) AS previous_month_order
  FROM restaurants r
  LEFT JOIN orders o ON r.restaurant_id = o.restaurant_id
  LEFT JOIN deliveries d ON d.order_id = o.order_id
  WHERE d.delivery_status = 'Delivered'
  GROUP BY r.restaurant_id, r.restaurant_name, Month_no
)

SELECT 
  cte.restaurant_id,
  cte.restaurant_name,
  cte.Month_no,
  cte.total_orders,
  cte.previous_month_order,
  ROUND(
    (cte.total_orders - cte.previous_month_order) / NULLIF(cte.previous_month_order, 0) * 100, 2
  ) AS monthly_growth_ratio
FROM cte
ORDER BY cte.restaurant_id, cte.Month_no;
```
## CUSTOMER SEGMENTATION
-- 12. Segment customers into 'Gold' or 'Silver' groups based on their total spending compared to the average order value (AOV). 
-- If a customer's total spending exceeds the AOV, -- label them as 'Gold'; 
-- otherwise, label them as 'Silver'. -- Return: The total number of orders and total revenue for each segment.
```sql
WITH customer_spending AS (
  SELECT 
    c.customer_id,
    c.customer_name,
    COUNT(o.order_id) AS total_orders,
    SUM(o.total_amount) AS total_spent
  FROM customers c
  JOIN orders o ON c.customer_id = o.customer_id
  GROUP BY c.customer_id, c.customer_name
),

aov_cte AS (
  SELECT AVG(total_amount) AS aov FROM orders
),

segmented_customers AS (
  SELECT 
    cs.customer_id,
    CASE 
      WHEN cs.total_spent > a.aov THEN 'Gold'
      ELSE 'Silver'
    END AS customer_segmentation,
    cs.total_orders,
    cs.total_spent
  FROM customer_spending cs
  CROSS JOIN aov_cte a
)

SELECT 
  customer_segmentation,
  SUM(total_orders) AS Total_orders,
  ROUND(SUM(total_spent), 2) AS Total_revenue
FROM segmented_customers
GROUP BY customer_segmentation;
```
### RIDER MONTHLY EARNINGS
-- 13. Calculate each rider's total monthly earnings, assuming they earn 8% of the order amount.
```sql
SELECT 
  riders.rider_id,
  rider_name, 
  DATE_FORMAT(order_date, "%Y-%m") AS Month_no,
  ROUND(SUM(total_amount) * 0.08, 2) AS Monthly_Earnings
FROM riders 
LEFT JOIN deliveries ON riders.rider_id = deliveries.rider_id
LEFT JOIN orders ON orders.order_id = deliveries.order_id
WHERE order_status = "Completed" AND delivery_status = "Delivered"
GROUP BY riders.rider_id, rider_name, Month_no
ORDER BY riders.rider_id, Month_no;
```
### RIDER RATING ANALYSIS
-- 14. Find the number of 5-star, 4-star, and 3-star ratings each rider has. Riders receive 
-- ratings based on delivery time: 
-- ● 5-star: Delivered in less than 30 minutes 
-- ● 4-star: Delivered between 30 and 45 minutes 
-- ● 3-star: Delivered after 45 minutes
```sql
WITH cte AS (
SELECT riders.rider_id, rider_name,
 CASE WHEN delivery_time > order_time THEN ROUND(TIMESTAMPDIFF(MINUTE,order_time,delivery_time),2) 
      ELSE ROUND(TIMESTAMPDIFF(MINUTE,order_time,delivery_time + INTERVAL 1 DAY),2)  END AS Delivery_min
FROM riders LEFT JOIN deliveries
ON riders.rider_id=deliveries.rider_id
LEFT JOIN orders
ON orders.order_id=deliveries.order_id
WHERE Order_status = "completed" AND delivery_status = "delivered"
)
SELECT rider_id, rider_name,
SUM(CASE WHEN delivery_min < 30 THEN 1 ELSE 0 END) AS Five_star,
SUM(CASE WHEN delivery_min BETWEEN 30 AND 45 THEN 1 ELSE 0 END) AS Four_star,
SUM(CASE WHEN delivery_min > 45 THEN 1 ELSE 0 END) AS Three_star
FROM cte
GROUP BY rider_id, rider_name
ORDER BY rider_id;
```
### ORDER FREQUENCY BY DAY
-- 15. Analyze order frequency per day of the week and identify the peak day for each restaurant.
```sql
WITH cte AS (
    SELECT 
        r.restaurant_id,
        r.restaurant_name,
        COUNT(o.order_id) AS total_orders,
        DAYNAME(o.order_date) AS day_of_week
    FROM orders o
    INNER JOIN restaurants r ON o.restaurant_id = r.restaurant_id
    WHERE o.order_status = "Completed"
    GROUP BY r.restaurant_id, r.restaurant_name, day_of_week),
highest_order_day AS (
    SELECT 
        restaurant_id,
        restaurant_name,
        day_of_week,
        total_orders,
        DENSE_RANK() OVER (PARTITION BY restaurant_id ORDER BY total_orders DESC) AS rnk
    FROM cte)
SELECT 
    restaurant_id, 
    restaurant_name, 
    day_of_week AS peak_day, 
    total_orders
FROM highest_order_day
WHERE rnk = 1
ORDER BY restaurant_id;
```
### CUSTOMER LIFETIME VALUE
-- 16. Calculate the total revenue generated by each customer over all their orders.
```sql
SELECT 
  customers.customer_id,
  customer_name, 
  SUM(total_amount) AS CLV
FROM customers 
LEFT JOIN orders
  ON customers.customer_id = orders.customer_id
GROUP BY customers.customer_id, customer_name;
```
### MONTHLY SALES TREND
-- 17. Identify sales trends by comparing each month's total sales to the previous month.
```sql
WITH cte AS (
SELECT DATE_FORMAT(order_date,"%Y") AS Year_no, DATE_FORMAT(order_date,"%m") AS month_no,
SUM(total_amount) AS Total_sales, 
LAG(SUM(total_amount),1) OVER ( ORDER BY DATE_FORMAT(order_date,"%Y"), DATE_FORMAT(order_date,"%m")) AS Previous_month_sales
FROM orders
GROUP BY year_no,month_no
ORDER BY year_no,month_no)

SELECT Year_no,Month_no,Total_sales, Previous_month_sales,CONCAT(ROUND((total_sales-previous_month_sales)/previous_month_sales*100,2)," ","%") AS Growth_ratio
FROM cte;
```
### RIDER EFFICIENCY
-- 18. Evaluate rider efficiency by determining average delivery times and identifying those with the lowest and highest averages.
```sql
WITH delivery_times AS (
  SELECT 
    r.rider_id,
    r.rider_name,
    CASE 
      WHEN d.delivery_time > o.order_time 
      THEN TIMESTAMPDIFF(MINUTE, o.order_time, d.delivery_time)
      ELSE TIMESTAMPDIFF(MINUTE, o.order_time, d.delivery_time + INTERVAL 1 DAY)
    END AS delivery_minutes
  FROM riders r
  JOIN deliveries d ON r.rider_id = d.rider_id
  JOIN orders o ON o.order_id = d.order_id
  WHERE o.order_status = 'Completed' AND d.delivery_status = 'Delivered'),

avg_times AS (
  SELECT 
    rider_id,
    rider_name,
    ROUND(AVG(delivery_minutes), 2) AS avg_delivery_time
  FROM delivery_times
  GROUP BY rider_id, rider_name)

SELECT 
  *,
  (SELECT MIN(avg_delivery_time) FROM avg_times) AS Min_avg_time,
  (SELECT MAX(avg_delivery_time) FROM avg_times) AS Max_avg_time
FROM avg_times
ORDER BY avg_delivery_time;
```
### ORDER ITEM POPULARITY
-- 19. Track the popularity of specific order items over time and identify seasonal demand spikes.
```sql
WITH cte AS(
 SELECT order_item, COUNT(order_id) AS Total_orders,
    CASE 
       WHEN MONTH(order_date) BETWEEN 3 AND 5 THEN "Spring"
       WHEN MONTH(order_date) BETWEEN 6 AND 8 THEN "Summer"
       WHEN MONTH(Order_date) BETWEEN 9 AND 11 THEN "Autumn"
       ELSE "Winter"
       END AS Seasons
FROM orders 
WHERE order_status = "completed"
GROUP BY order_item, seasons),

Trending_item AS (
SELECT  order_item, Total_orders, seasons, DENSE_RANK() OVER(PARTITION BY order_item ORDER BY total_orders DESC) AS rnk
FROM cte)

SELECT order_item, Total_orders, seasons FROM trending_item WHERE rnk = 1;
```
### CITY REVENUE RANKING
-- 20. Rank each city based on the total revenue for the year 2023.
```sql
SELECT 
    r.city,
    SUM(o.total_amount) AS total_revenue,
    DENSE_RANK() OVER (ORDER BY SUM(o.total_amount) DESC) AS city_rank
FROM restaurants r
JOIN orders o ON r.restaurant_id = o.restaurant_id
WHERE YEAR(o.order_date) = 2023
GROUP BY r.city
ORDER BY city_rank;
```


