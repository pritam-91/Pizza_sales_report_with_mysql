# Pizza_sales_report_with_mysql

**1.Retrieve the total number of orders placed.**

SELECT 
    COUNT(order_id) AS TOTAL_ORDERS
FROM
    pizza.orders;

**2.Calculate the total revenue generated from pizza sales.**

SELECT 
    ROUND(SUM(quantity * price), 2) AS GROSS_REVENUE
FROM
    pizza.pizzas AS A
        INNER JOIN
    pizza.order_details AS B ON A.pizza_id = B.pizza_id;

**3.Identify the highest-priced pizza.**

 SELECT 
    name, price AS PRICE
FROM
    pizza.pizza_types AS A
        JOIN
    pizza.pizzas AS B ON A.pizza_type_id = B.pizza_type_id
ORDER BY PRICE DESC
LIMIT 1;

**4.Identify the most common pizza size ordered.**

SELECT 
    size, COUNT(size) AS COUNTS
FROM
    (SELECT 
        order_id, size
    FROM
        pizza.order_details AS A
    INNER JOIN pizza.pizzas AS B ON A.pizza_id = B.pizza_id) AS T1
GROUP BY size
ORDER BY COUNTS DESC;

**5.List the top 5 most ordered pizza types along with their quantities.**

SELECT 
    name, SUM(quantity) AS QTY
FROM
    pizza.pizza_types AS A
        JOIN
    pizza.pizzas AS B ON A.pizza_type_id = B.pizza_type_id
        JOIN
    pizza.order_details AS C ON C.pizza_id = B.pizza_id
GROUP BY name
ORDER BY QTY DESC
LIMIT 5;

**6.Join the necessary tables to find the total quantity of each pizza category ordered.**

SELECT 
    category, SUM(quantity) AS quantity
FROM
    pizza.pizza_types AS A
        JOIN
    pizza.pizzas AS B ON A.pizza_type_id = B.pizza_type_id
        JOIN
    pizza.order_details AS C ON B.pizza_id = C.pizza_id
GROUP BY category
ORDER BY quantity DESC;

**7.Determine the distribution of orders by hour of the day.**

SELECT 
    HOUR(time) AS HOURS, COUNT(order_id) AS ORDERS
FROM
    pizza.orders
GROUP BY HOURS;

**8.Join relevant tables to find the category-wise distribution of pizzas.**

SELECT 
    category, COUNT(name) AS COUNTS
FROM
    pizza.pizza_types
GROUP BY category;

**9.Group the orders by date and calculate the average number of pizzas ordered per day.**

SELECT 
    ROUND(AVG(AVG_QTY), 0) AS AVG_QTY
FROM
    (SELECT 
        date, SUM(quantity) AS AVG_QTY
    FROM
        pizza.orders AS A
    JOIN pizza.order_details AS B ON A.order_id = B.order_id
    GROUP BY date) AS T1;

**10.Determine the top 3 most ordered pizza types based on revenue.**

SELECT 
    name, SUM(price * quantity) AS REVENUE
FROM
    pizza.pizza_types AS A
        JOIN
    pizza.pizzas AS B ON A.pizza_type_id = B.pizza_type_id
        JOIN
    pizza.order_details AS C ON B.pizza_id = C.pizza_id
GROUP BY name
ORDER BY REVENUE DESC
LIMIT 3;

**11.Calculate the percentage contribution of each pizza type to total revenue.**

SELECT 
    category,
    CONCAT(ROUND(SUM(price * quantity) / (SELECT 
                            SUM(quantity * price) AS GROSS_REVENUE
                        FROM
                            pizza.pizzas AS A
                                INNER JOIN
                            pizza.order_details AS B ON A.pizza_id = B.pizza_id) * 100,
                    2),
            '%') AS PERSENT
FROM
    pizza.order_details AS A
        JOIN
    pizza.pizzas AS B ON A.pizza_id = B.pizza_id
        JOIN
    pizza.pizza_types AS C ON B.pizza_type_id = C.pizza_type_id
GROUP BY category;

**12.Analyze the cumulative revenue generated over time.**

SELECT date, ROUND(SUM(REVENUE) OVER(ORDER BY date),2) AS CUM_REVENUE 
FROM (SELECT date,SUM(quantity*price) AS REVENUE
FROM pizza.order_details AS A
JOIN pizza.pizzas AS B
ON A.pizza_id = B.pizza_id JOIN 
pizza.orders AS C
ON A.order_id = C.order_id
GROUP BY date ) AS SALES;

**13.Determine the top 3 most ordered pizza types based on revenue for each pizza category.**

SELECT name,ROUND(REVENUE,2) AS REVENUE FROM 
(SELECT category,name,REVENUE,
RANK() OVER(PARTITION BY category ORDER BY REVENUE DESC) AS RANKS 
FROM (SELECT category,name,SUM(price*quantity) AS REVENUE
FROM pizza.order_details AS A
JOIN pizza.pizzas AS B
ON A.pizza_id = B.pizza_id JOIN
pizza.pizza_types AS C
ON B.pizza_type_id = C.pizza_type_id
GROUP BY category,name) AS T1) AS T2 
WHERE RANKS <= 3;
