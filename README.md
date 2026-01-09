--  CREATE DATABASE
CREATE DATABASE sales_dw;
USE sales_dw;

--  DROP TABLES (FOR RE-RUN SAFETY)
DROP TABLE IF EXISTS sales_fact;
DROP TABLE IF EXISTS time_dim;
DROP TABLE IF EXISTS customer_dim;
DROP TABLE IF EXISTS product_dim;
DROP TABLE IF EXISTS location_dim;

-- CREATE DIMENSION TABLES

-- TIME DIMENSION
CREATE TABLE time_dim (
    time_id INT PRIMARY KEY,
    date DATE,
    month INT,
    month_name VARCHAR(20),
    year INT
);

-- CUSTOMER DIMENSION
CREATE TABLE customer_dim (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100),
    gender VARCHAR(10),
    age INT
);

-- PRODUCT DIMENSION
CREATE TABLE product_dim (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    category VARCHAR(50),
    price DECIMAL(10,2)
);

-- LOCATION DIMENSION
CREATE TABLE location_dim (
    location_id INT PRIMARY KEY,
    city VARCHAR(50),
    region VARCHAR(50),
    country VARCHAR(50)
);

-- CREATE FACT TABLE

CREATE TABLE sales_fact (
    sales_id INT PRIMARY KEY,
    time_id INT,
    customer_id INT,
    product_id INT,
    location_id INT,
    quantity INT,
    total_amount DECIMAL(12,2),

    FOREIGN KEY (time_id) REFERENCES time_dim(time_id),
    FOREIGN KEY (customer_id) REFERENCES customer_dim(customer_id),
    FOREIGN KEY (product_id) REFERENCES product_dim(product_id),
    FOREIGN KEY (location_id) REFERENCES location_dim(location_id)
);

-- INSERT DATA INTO DIMENSIONS

INSERT INTO time_dim VALUES
(1, '2025-01-10', 1, 'January', 2025),
(2, '2025-02-15', 2, 'February', 2025),
(3, '2025-03-20', 3, 'March', 2025);

INSERT INTO customer_dim VALUES
(1, 'Ravi', 'Male', 22),
(2, 'Anita', 'Female', 25),
(3, 'Suresh', 'Male', 30);

INSERT INTO product_dim VALUES
(1, 'Laptop', 'Electronics', 50000),
(2, 'Mobile', 'Electronics', 20000),
(3, 'Headphones', 'Accessories', 3000);

INSERT INTO location_dim VALUES
(1, 'Hyderabad', 'South', 'India'),
(2, 'Delhi', 'North', 'India'),
(3, 'Mumbai', 'West', 'India');

--  INSERT DATA INTO FACT TABLE

INSERT INTO sales_fact VALUES
(1, 1, 1, 1, 1, 1, 50000),
(2, 2, 2, 2, 2, 2, 40000),
(3, 3, 3, 3, 3, 3, 9000),
(4, 1, 2, 2, 1, 1, 20000),
(5, 2, 1, 3, 2, 2, 6000);

-- 7. CREATE INDEXES (PERFORMANCE OPTIMIZATION)

CREATE INDEX idx_sales_time ON sales_fact(time_id);
CREATE INDEX idx_sales_customer ON sales_fact(customer_id);
CREATE INDEX idx_sales_product ON sales_fact(product_id);
CREATE INDEX idx_sales_location ON sales_fact(location_id);

--  ANALYTICAL QUERIES
-- REVENUE BY REGION
SELECT 
    l.region,
    SUM(s.total_amount) AS total_revenue
FROM sales_fact s
JOIN location_dim l ON s.location_id = l.location_id
GROUP BY l.region
ORDER BY total_revenue DESC;

-- REVENUE BY MONTH
SELECT 
    t.month_name,
    SUM(s.total_amount) AS total_revenue
FROM sales_fact s
JOIN time_dim t ON s.time_id = t.time_id
GROUP BY t.month_name
ORDER BY total_revenue DESC;

-- PRODUCT WISE REVENUE (EXTRA – INTERVIEW BONUS)
SELECT
    p.product_name,
    SUM(s.total_amount) AS revenue
FROM sales_fact s
JOIN product_dim p ON s.product_id = p.product_id
GROUP BY p.product_name;

-- CUSTOMER WISE SPENDING (EXTRA – INTERVIEW BONUS)
SELECT
    c.customer_name,
    SUM(s.total_amount) AS total_spent
FROM sales_fact s
JOIN customer_dim c ON s.customer_id = c.customer_id
GROUP BY c.customer_name;
