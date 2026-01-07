/* ================================
   1. CREATE TABLES
================================ */

CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100),
    email VARCHAR(100),
    city VARCHAR(50),
    state VARCHAR(10),
    registration_date DATE
);

CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    category VARCHAR(50),
    price INT
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    order_status VARCHAR(20),
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

CREATE TABLE order_items (
    order_item_id INT PRIMARY KEY,
    order_id INT,
    product_id INT,
    quantity INT,
    price INT,
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);

CREATE TABLE payments (
    payment_id INT PRIMARY KEY,
    order_id INT,
    payment_method VARCHAR(30),
    payment_status VARCHAR(20),
    FOREIGN KEY (order_id) REFERENCES orders(order_id)
);

/* ================================
   2. INSERT DATA
================================ */

INSERT INTO customers VALUES
(1,'Aarav','aarav@gmail.com','Bangalore','KA','2023-01-10'),
(2,'Priya','priya@gmail.com','Hyderabad','TS','2023-02-15'),
(3,'Rohit','rohit@gmail.com','Delhi','DL','2023-03-20'),
(4,'Sneha','sneha@gmail.com','Chennai','TN','2023-04-05'),
(5,'Vikram','vikram@gmail.com','Mumbai','MH','2023-05-01');

INSERT INTO products VALUES
(101,'Laptop','Electronics',60000),
(102,'Smartphone','Electronics',30000),
(103,'Headphones','Accessories',2000),
(104,'Keyboard','Accessories',1500),
(105,'Mouse','Accessories',800);

INSERT INTO orders VALUES
(1001,1,'2024-01-05','Delivered'),
(1002,2,'2024-01-10','Delivered'),
(1003,3,'2024-02-02','Cancelled'),
(1004,1,'2024-02-20','Delivered'),
(1005,4,'2024-03-01','Delivered');

INSERT INTO order_items VALUES
(1,1001,101,1,60000),
(2,1001,105,2,800),
(3,1002,102,1,30000),
(4,1003,103,1,2000),
(5,1004,104,1,1500),
(6,1005,101,1,60000);

INSERT INTO payments VALUES
(5001,1001,'Credit Card','Success'),
(5002,1002,'UPI','Success'),
(5003,1003,'Debit Card','Failed'),
(5004,1004,'UPI','Success'),
(5005,1005,'Net Banking','Success');


-- Total Revenue
SELECT SUM(quantity * price) AS total_revenue
FROM order_items;

-- Total Orders
SELECT COUNT(*) AS total_orders
FROM orders;

-- Average Order Value
SELECT 
    SUM(quantity * price) / COUNT(DISTINCT order_id) AS avg_order_value
FROM order_items;

-- Top Selling Products
SELECT 
    p.product_name,
    SUM(oi.quantity) AS total_sold
FROM order_items oi
JOIN products p ON oi.product_id = p.product_id
GROUP BY p.product_name
ORDER BY total_sold DESC;

-- Category-wise Revenue
SELECT 
    p.category,
    SUM(oi.quantity * oi.price) AS category_revenue
FROM order_items oi
JOIN products p ON oi.product_id = p.product_id
GROUP BY p.category;

-- Top Customers by Spending
SELECT 
    c.customer_name,
    SUM(oi.quantity * oi.price) AS total_spent
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
GROUP BY c.customer_name
ORDER BY total_spent DESC;

-- Monthly Sales Trend
SELECT 
    YEAR(o.order_date) AS year,
    MONTH(o.order_date) AS month,
    SUM(oi.quantity * oi.price) AS monthly_sales
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
GROUP BY year, month
ORDER BY year, month;

-- Payment Method Usage
SELECT 
    payment_method,
    COUNT(*) AS usage_count
FROM payments
GROUP BY payment_method
ORDER BY usage_count DESC;


-- Product Revenue Ranking (Window Function)
SELECT 
    p.product_name,
    SUM(oi.quantity * oi.price) AS revenue,
    RANK() OVER (ORDER BY SUM(oi.quantity * oi.price) DESC) AS rank_no
FROM order_items oi
JOIN products p ON oi.product_id = p.product_id
GROUP BY p.product_name;

-- Customers with More Than One Order
SELECT 
    customer_id,
    COUNT(order_id) AS order_count
FROM orders
GROUP BY customer_id
HAVING COUNT(order_id) > 1;
