# Pizza_Sales_Analysis

## Project Overview

This project focuses on analyzing pizza sales data
using MySQL to understand overall business
performance.
The analysis helps identify revenue trends, popular
pizza types, customer ordering patterns, and peak
sales periods.
By querying transactional data, meaningful insights
were derived to support data-driven business
decisions.

## Dataset Description

### Dataset Source
Pizza Sales Dataset (Kaggle)

### Dataset Type
CSV File

### Tables Used
- **[Orders](https://github.com/NishanthNalanagula/Pizza_Sales/blob/main/orders.csv)**
- **[Order Details](https://github.com/NishanthNalanagula/Pizza_Sales/blob/main/order_details.csv)**
- **[Pizzas](https://github.com/NishanthNalanagula/Pizza_Sales/blob/main/pizzas.csv)**
- **[Pizza Types](https://github.com/NishanthNalanagula/Pizza_Sales/blob/main/pizza_types.csv)**

### Dataset Details
- Total records: 48,000+
- Time period: One year of sales data
- Data type: Transactional sales data

## Data Analysis & Findings

The following SQL queries were developed to answer specific business questions:

1. **Retrieve the total number of orders placed**:
```sql
SELECT 
    COUNT(*) AS total_orders
FROM
    orders;
```

2. **Calculate the total revenue generated from pizza sales**:
```sql
SELECT 
    ROUND(SUM(od.quantity * p.price), 2) AS total_sales
FROM
    orders_details od
        JOIN
    pizzas p ON od.pizza_id = p.pizza_id;
```

3. **Identify the highest-priced pizza**:
```sql
SELECT 
    pt.name, p.price
FROM
    pizzas p
        JOIN
    pizza_types pt ON p.pizza_type_id = pt.pizza_type_id
ORDER BY price DESC
LIMIT 1;
```

4. **Identify the most common pizza size ordered**:
```sql
SELECT 
    p.size, COUNT(od.order_details_id) AS freq
FROM
    orders_details od
        JOIN
    pizzas p ON p.pizza_id = od.pizza_id
GROUP BY size
ORDER BY freq DESC;
```

5. **List the top 5 most ordered pizza types along with their quantities**:
```sql
SELECT 
    pt.name, SUM(od.quantity) AS total_quantity
FROM
    orders_details od
        JOIN
    pizzas p ON p.pizza_id = od.pizza_id
        JOIN
    pizza_types pt ON pt.pizza_type_id = p.pizza_type_id
GROUP BY pt.name
ORDER BY total_quantity DESC
LIMIT 5;
```

6. **Join the necessary tables to find the total quantity of each pizza category ordered**:
```sql
SELECT 
    pt.category, SUM(od.quantity) AS quantity
FROM
    orders_details od
        JOIN
    pizzas p ON p.pizza_id = od.pizza_id
        JOIN
    pizza_types pt ON p.pizza_type_id = pt.pizza_type_id
GROUP BY pt.category
ORDER BY od.quantity DESC;
```

7. **Determine the distribution of orders by hour of the day**:
```sql
SELECT 
    HOUR(order_time) AS hour, COUNT(order_id) AS order_count
FROM
    orders
GROUP BY HOUR(order_time);
```

8. **Join relevant tables to find the category-wise distribution of pizzas**:
```sql
SELECT 
    category, COUNT(name) AS pizza_count
FROM
    pizza_types
GROUP BY category;
```

9. **Group the orders by date and calculate the average number of pizzas ordered per day**:
```sql
WITH order_quantity AS (
SELECT 
    o.order_date, SUM(quantity) AS pizza_count
FROM
    orders o
        JOIN
    orders_details od ON od.order_id = o.order_id
GROUP BY order_date
)
SELECT 
    ROUND(AVG(pizza_count), 0) AS avg_pizzas_ordered_per_day
FROM
    order_quantity;
```

10. **Determine the top 3 most ordered pizza types based on revenue**:
```sql
SELECT 
    pt.name, SUM(price * quantity) AS revenue
FROM
    orders_details od
        JOIN
    pizzas p ON od.pizza_id = p.pizza_id
        JOIN
    pizza_types pt ON p.pizza_type_id = pt.pizza_type_id
GROUP BY pt.name
ORDER BY revenue DESC
LIMIT 3;
```

11. **Calculate the percentage contribution of each pizza type to total revenue**:
```sql
SELECT 
    pt.category,
    ROUND((SUM(price * quantity) / (SELECT 
                    ROUND(SUM(od.quantity * p.price), 2) AS total_sales
                FROM
                    orders_details od
                        JOIN
                    pizzas p ON od.pizza_id = p.pizza_id)) * 100, 2) AS revenue_pct
FROM
    orders_details od
        JOIN
    pizzas p ON od.pizza_id = p.pizza_id
        JOIN
    pizza_types pt ON pt.pizza_type_id = p.pizza_type_id
GROUP BY pt.category
ORDER BY revenue_pct DESC;
```

12. **Analyze the cumulative revenue generated over time**:
```sql
WITH sales AS (
SELECT 
    o.order_date, SUM(p.price * od.quantity) AS revenue
FROM
    orders o
        JOIN
    orders_details od ON od.order_id = o.order_id
        JOIN
    pizzas p ON p.pizza_id = od.pizza_id
GROUP BY o.order_date
)

SELECT order_date, SUM(revenue) OVER(ORDER BY order_date) AS cum_revenue
FROM sales;
```

13. **Determine the top 3 most ordered pizza types based on revenue for each pizza category**:
```sql
WITH pizza_category_revenue AS (

	WITH sales AS (
	SELECT 
    pt.category, pt.name, SUM(od.quantity * p.price) AS revenue
	FROM
    orders_details od
        JOIN
    pizzas p ON p.pizza_id = od.pizza_id
        JOIN
    pizza_types pt ON pt.pizza_type_id = p.pizza_type_id
	GROUP BY pt.category , pt.name
	)

SELECT *, RANK() OVER(PARTITION BY category ORDER BY revenue DESC) AS rn
FROM sales
)

SELECT 
    category, name, revenue
FROM
    pizza_category_revenue
WHERE
    rn <= 3;
```
