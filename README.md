# ParoCyber_SQL_Capstone
Project Overview: Monday Coffee, a fictional online brand selling across India since Jan 2023, plans to expand into physical stores. Acting as a Data Analyst, you will use SQL to analyze sales, customer, and city data to identify the top three Indian cities best suited for opening their new coffee shop locations.

#Technical Report: Monday Coffee Business Expansion AnalysisProject Overview
-Monday Coffee is a fictional coffee brand that has been successfully operating an online direct-to-consumer delivery service across multiple major cities in India since January 2023. As the company looks to make its first move into physical, brick-and-mortar retail storefronts, this capstone project aims to analyze backend sales, customer populations, satisfaction rankings, and local real estate overhead constraints.  Acting as the lead Data Analyst, I used PostgreSQL to query across four relational datasets to identify the top three Indian cities best suited for establishing Monday Coffee's initial physical flagship store locations

#Dataset Description & Schema Design
-The project environment utilizes a clean relational layout with strict constraints ensuring data integrity (ON DELETE CASCADE, ON DELETE SET NULL, and logical CHECK rules).  
#Entity Relationship Mappingcity: 
- Core geographical dimensions containing city names, populations, ranks, and estimated local baseline rent.  products: Catalog containing individual beverage item stock descriptions and unit base price lists.  customers: Customer profiles tracking regional identity mapping directly via city_id references.  sales: Fact table housing fine-grained transactional tracking lines connecting users, products, revenues, and ratings.

#Table Creation Scripts (DDL)
-- Drop the database if it already exists to avoid conflict errors
DROP DATABASE IF EXISTS monday_coffee;

-- Create Database 
CREATE DATABASE monday_coffee;

-- Connect to the database before creating schemas
\c monday_coffee;

-- monday_coffee schemas
DROP TABLE IF EXISTS sales;
DROP TABLE IF EXISTS customers;
DROP TABLE IF EXISTS products;
DROP TABLE IF EXISTS city;

-- 1. Create the CITY table
CREATE TABLE city (
    city_id INT PRIMARY KEY,
    city_name VARCHAR(100) NOT NULL,
    population INT NOT NULL,
    estimated_rent INT NOT NULL,
    city_rank INT
);

-- 2. Create the PRODUCTS table
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(150) NOT NULL,
    price INT NOT NULL
);

-- 3. Create the CUSTOMERS table
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(150) NOT NULL,
    city_id INT,
    CONSTRAINT fk_customer_city FOREIGN KEY (city_id) REFERENCES city(city_id) ON DELETE SET NULL
);

-- 4. Create the SALES table
CREATE TABLE sales (
    sale_id INT PRIMARY KEY,
    sale_date DATE NOT NULL,
    product_id INT NOT NULL,
    customer_id INT NOT NULL,
    total INT NOT NULL,          -- Represents units sold from the transaction
    rating INT CHECK (rating BETWEEN 1 AND 5),
    CONSTRAINT fk_sales_product FOREIGN KEY (product_id) REFERENCES products(product_id) ON DELETE CASCADE,
    CONSTRAINT fk_sales_customer FOREIGN KEY (customer_id) REFERENCES customers(customer_id) ON DELETE CASCADE
);

#Structured Data Import Rules
-To prevent foreign key dependency errors during raw data setup, data must be populated into the environment sequentially following this path:  
1. city table (Independent dimensional base)
2. products table (Independent catalog base)
3. customers table (Dependent on city)
4. sales table (Dependent on both products and customers)

#Methodology (questions and sql answers) 
-- Drop the database if it already exists to avoid conflict errors
DROP DATABASE IF EXISTS monday_coffee;

-- Create Database 
CREATE DATABASE monday_coffee;

-- monday_coffee schemas
DROP TABLE IF EXISTS sales;
DROP TABLE IF EXISTS customers;
DROP TABLE IF EXISTS products;
DROP TABLE IF EXISTS city;

--Create Tables

-- 1. Create the CITY table
CREATE TABLE city (
    city_id INT PRIMARY KEY,
    city_name VARCHAR(100) NOT NULL,
    population INT NOT NULL,
    estimated_rent INT NOT NULL,
    city_rank INT
);

SELECT* FROM city;

-- 2. Create the PRODUCTS table
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(150) NOT NULL,
    price INT NOT NULL
);

SELECT* FROM products
LIMIT 10;

-- 3. Create the CUSTOMERS table
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(150) NOT NULL,
    city_id INT,
    CONSTRAINT fk_customer_city FOREIGN KEY (city_id) REFERENCES city(city_id) ON DELETE SET NULL
);

SELECT* FROM customers
LIMIT 10;

-- 4. Create the SALES table
CREATE TABLE sales (
    sale_id INT PRIMARY KEY,
    sale_date DATE NOT NULL,
    product_id INT NOT NULL,
    customer_id INT NOT NULL,
    total INT NOT NULL,          -- Represents revenue from the sale
    rating INT CHECK (rating BETWEEN 1 AND 5),
    CONSTRAINT fk_sales_product FOREIGN KEY (product_id) REFERENCES products(product_id) ON DELETE CASCADE,
    CONSTRAINT fk_sales_customer FOREIGN KEY (customer_id) REFERENCES customers(customer_id) ON DELETE CASCADE
);

SELECT* FROM sales
LIMIT 10;

--Import rules
--1st import to city
--2nd import to products
--3rd import to customers
--4th import to sales

/*Question 1: Coffee Consumer Estimate 
Assuming 25% of each city's population drinks coffee, calculate the estimated number of coffee 
consumers (in millions) per city. Order results from highest to lowest.*/

SELECT 
    city_name,
    ROUND((population * 0.25) / 1000000.0, 2) AS estimated_coffee_consumers_millions
FROM 
    city
ORDER BY 
    estimated_coffee_consumers_millions DESC;

/*Question 2: Total Revenue - Q4 2023 
What is the total revenue generated from coffee sales across all cities during the last quarter of 
2023 (October-December)? Show results per city, ordered by revenue descending.*/
-- Joining sales, products, and customers to calculate Q4 2023 revenue by city
SELECT 
    ci.city_name,
    SUM(s.total * p.price) AS total_revenue
FROM 
    sales AS s
JOIN 
    products AS p ON s.product_id = p.product_id
JOIN 
    customers AS cu ON s.customer_id = cu.customer_id
JOIN 
    city AS ci ON cu.city_id = ci.city_id
WHERE 
    s.sale_date BETWEEN '2023-10-01' AND '2023-12-31'
GROUP BY 
    ci.city_name
ORDER BY 
    total_revenue DESC
LIMIT 10;

/*Question 3: Sales Volume by Product 
How many units of each coffee product have been sold in total? Rank products from best-selling 
to least-selling.*/

-- Aggregating total units sold per product and ranking them descending
SELECT 
    p.product_name,
    SUM(s.total) AS total_units_sold
FROM 
    sales AS s
JOIN 
    products AS p ON s.product_id = p.product_id
GROUP BY 
    p.product_name
ORDER BY 
    total_units_sold DESC
LIMIT 10;

/*Question 4: Average Sales per Customer by City 
What is the average total sales amount per unique customer in each city? Include total revenue 
and customer count alongside the average. Order by total revenue descending.*/

-- Calculating total revenue, unique customers, and average spend per customer by city
SELECT 
    ci.city_name,
    SUM(s.total * p.price) AS total_revenue,
    COUNT(DISTINCT s.customer_id) AS unique_customer_count,
    SUM(s.total * p.price) / COUNT(DISTINCT s.customer_id) AS avg_sales_per_customer
FROM 
    sales AS s
JOIN 
    products AS p ON s.product_id = p.product_id
JOIN 
    customers AS cu ON s.customer_id = cu.customer_id
JOIN 
    city AS ci ON cu.city_id = ci.city_id
GROUP BY 
    ci.city_name
ORDER BY 
    total_revenue DESC
LIMIT 10;

/*Question 5: Current Customers vs. Estimated Coffee Consumers 
For each city, show both the estimated coffee-drinking population (25% of city population, in 
millions) and the actual number of unique customers from the sales data. Use a CTE.*/

-- Using a CTE to compile market potential vs. actual active customer counts per city
WITH city_market_cte AS (
    SELECT 
        city_id,
        city_name,
        ROUND((population * 0.25) / 1000000, 3) AS estimated_consumers_millions
    FROM 
        city
)
SELECT 
    cm.city_name,
    cm.estimated_consumers_millions,
    COUNT(DISTINCT s.customer_id) AS actual_unique_customers
FROM 
    sales AS s
JOIN 
    customers AS cu ON s.customer_id = cu.customer_id
JOIN 
    city_market_cte AS cm ON cu.city_id = cm.city_id
GROUP BY 
    cm.city_name, 
    cm.estimated_consumers_millions
ORDER BY 
    actual_unique_customers DESC
LIMIT 5;

/*Question 6: Top 3 Products per City 
What are the top 3 best-selling coffee products in each city, based on number of orders? Use a 
window function to rank products within each city.*/
-- Ranking product popularity within each city using DENSE_RANK() to find the top 3
WITH ranked_products_cte AS (
    SELECT 
        ci.city_name,
        p.product_name,
        COUNT(s.sale_id) AS total_orders,
        DENSE_RANK() OVER (PARTITION BY ci.city_name ORDER BY COUNT(s.sale_id) DESC) AS product_rank
    FROM 
        sales AS s
    JOIN 
        products AS p ON s.product_id = p.product_id
    JOIN 
        customers AS cu ON s.customer_id = cu.customer_id
    JOIN 
        city AS ci ON cu.city_id = ci.city_id
    GROUP BY 
        ci.city_name, 
        p.product_name
)
SELECT 
    city_name,
    product_name,
    total_orders,
    product_rank
FROM 
    ranked_products_cte
WHERE 
    product_rank <= 3
LIMIT 5;

/*Question 7: Unique Customers per City 
How many unique customers in each city have made at least one coffee purchase? Order by 
customer count descending.*/

-- Counting distinct customer IDs associated with transactions in each city
SELECT 
    ci.city_name,
    COUNT(DISTINCT cu.customer_id) AS unique_customer_count
FROM 
    customers AS cu
JOIN 
    city AS ci ON cu.city_id = ci.city_id
JOIN 
    sales AS s ON cu.customer_id = s.customer_id
GROUP BY 
    ci.city_name
ORDER BY 
    unique_customer_count DESC
LIMIT 10;

/*Question 8: Average Sale vs. Average Rent per Customer 
For each city, compare the average sale amount per customer against the average rent cost per 
customer (estimated_rent divided by number of customers). This helps evaluate cost efficiency.*/

-- Evaluating rent cost-efficiency against revenue metrics per unique customer
SELECT 
    ci.city_name,
    SUM(s.total * p.price) / COUNT(DISTINCT s.customer_id) AS avg_sale_per_customer,
    ci.estimated_rent / COUNT(DISTINCT s.customer_id) AS avg_rent_per_customer
FROM 
    sales AS s
JOIN 
    products AS p ON s.product_id = p.product_id
JOIN 
    customers AS cu ON s.customer_id = cu.customer_id
JOIN 
    city AS ci ON cu.city_id = ci.city_id
GROUP BY 
    ci.city_name, 
    ci.estimated_rent
ORDER BY 
    avg_sale_per_customer DESC
LIMIT 10;

/*Question 9: Month-on-Month Sales Growth 
Calculate the month-on-month percentage change in total sales for each city. Use a window 
function (LAG) to compare each month's sales to the previous month. Show only rows where a 
prior month exists.*/

-- Using LAG to pull previous month's revenue and calculating MoM growth percentage
WITH monthly_sales_cte AS (
    SELECT 
        ci.city_name,
        EXTRACT(MONTH FROM s.sale_date) AS sale_month,
        EXTRACT(YEAR FROM s.sale_date) AS sale_year,
        SUM(s.total * p.price) AS total_sales
    FROM 
        sales AS s
    JOIN 
        products AS p ON s.product_id = p.product_id
    JOIN 
        customers AS cu ON s.customer_id = cu.customer_id
    JOIN 
        city AS ci ON cu.city_id = ci.city_id
    GROUP BY 
        ci.city_name, 
        EXTRACT(YEAR FROM s.sale_date), 
        EXTRACT(MONTH FROM s.sale_date)
),
lagged_sales_cte AS (
    SELECT 
        city_name,
        sale_year,
        sale_month,
        total_sales,
        LAG(total_sales) OVER(PARTITION BY city_name ORDER BY sale_year, sale_month) AS previous_month_sales
    FROM 
        monthly_sales_cte
)
SELECT 
    city_name,
    sale_year,
    sale_month,
    total_sales,
    previous_month_sales,
    ROUND(((total_sales - previous_month_sales) / previous_month_sales) * 100, 2) AS mom_growth_percentage
FROM 
    lagged_sales_cte
WHERE 
    previous_month_sales IS NOT NULL
LIMIT 10;

/*Question 10: Market Potential Summary 
Produce a full market potential table for each city, showing: total revenue, estimated rent, total 
customers, estimated coffee consumers (millions), average sale per customer, and average rent 
per customer. Order by total revenue descending.*/

-- Creating a comprehensive market analysis profile for final expansions
SELECT 
    ci.city_name,
    SUM(s.total * p.price) AS total_revenue,
    ci.estimated_rent,
    COUNT(DISTINCT s.customer_id) AS total_customers,
    ROUND((ci.population * 0.25) / 1000000, 3) AS estimated_coffee_consumers_millions,
    SUM(s.total * p.price) / COUNT(DISTINCT s.customer_id) AS avg_sale_per_customer,
    ci.estimated_rent / COUNT(DISTINCT s.customer_id) AS avg_rent_per_customer
FROM 
    sales AS s
JOIN 
    products AS p ON s.product_id = p.product_id
JOIN 
    customers AS cu ON s.customer_id = cu.customer_id
JOIN 
    city AS ci ON cu.city_id = ci.city_id
GROUP BY 
    ci.city_name, 
    ci.estimated_rent, 
    ci.population
ORDER BY 
    total_revenue DESC
LIMIT 10;

/*Bonus Task: Design Your Own Questions 
Now that you've explored the Monday Coffee dataset, it's your turn to think like an analyst. 
Come up with three original business questions that can be answered using SQL on this 
dataset. For each question: 
1. Write the business question clearly (what insight are you trying to surface?). 
2. Write the SQL query that answers it. 
3. Write a one-sentence interpretation of what the result tells Monday Coffee.*/

/*Question 1: Customer Retention / Purchase Frequency
Business Question: Which cities have the most loyal customer bases 
measured by the average number of orders placed per unique customer?
This identifies where coffee habits are most deeply formed*/

-- Finding the average number of orders placed per unique customer in each city
SELECT 
    ci.city_name,
    COUNT(s.sale_id) AS total_orders,
    COUNT(DISTINCT s.customer_id) AS unique_customers,
    ROUND(COUNT(s.sale_id)::NUMERIC / COUNT(DISTINCT s.customer_id), 2) AS avg_orders_per_customer
FROM 
    sales AS s
JOIN 
    customers AS cu ON s.customer_id = cu.customer_id
JOIN 
    city AS ci ON cu.city_id = ci.city_id
GROUP BY 
    ci.city_name
ORDER BY 
    avg_orders_per_customer DESC
LIMIT 10;


/*This shows us where customers keep coming back.A high purchase frequency implies strong brand stickiness, 
reducing the marketing spend required to sustain a physical store.*/

/*Question 2: Rent-to-Revenue Efficiency Ratio
Business Question: What percentage of online sales revenue would be consumed by estimated rent costs in each city?
Lower percentages signify a safer and more profitable retail footprint.*/

-- Calculating estimated rent as a percentage of total revenue to evaluate margin safety
SELECT 
        ci.city_name,
        ci.estimated_rent,
        SUM(s.total) AS total_revenue,
        ROUND((ci.estimated_rent::NUMERIC / SUM(s.total)) * 100, 2) AS rent_to_revenue_percentage
    FROM 
        sales AS s
    JOIN 
        customers AS cu ON s.customer_id = cu.customer_id
    JOIN 
        city AS ci ON cu.city_id = ci.city_id
    GROUP BY 
        ci.city_name, 
        ci.estimated_rent
    ORDER BY 
        rent_to_revenue_percentage ASC
LIMIT 10;
/*Cities like Pune and Jaipur are highly efficient because rent consumes less than 1.5% of their current online revenue pool, 
whereas cities like Mumbai or Hyderabad consume over 13%, making physical expansion riskier there.*/

/*Question 3: Customer Satisfaction (Average Rating) by City
Business Question: What is the average product rating given by customers in each city? 
This helps gauge product-market fit before building brick-and-mortar stores.*/

-- Calculating the average customer experience rating per city
SELECT 
    ci.city_name,
    ROUND(AVG(s.rating), 2) AS average_customer_rating,
    COUNT(s.sale_id) AS total_ratings_received
FROM 
    sales AS s
JOIN 
    customers AS cu ON s.customer_id = cu.customer_id
JOIN 
    city AS ci ON cu.city_id = ci.city_id
GROUP BY 
    ci.city_name
ORDER BY 
    average_customer_rating DESC
LIMIT 10;

/*Chennai, Bangalore, and Pune stand out significantly with stellar average ratings above 4.4, 
indicating exceptionally strong affinity and satisfaction with Monday Coffee's products.*/

# Key insights and recommendations 
/*Final task: Recommendation 
Based on your SQL analysis above, write your business recommendation. Identify the three 
cities you would recommend for Monday Coffee's first physical store locations and justify each 
choice using specific metrics from your queries (e.g., revenue, customer count, rent efficiency, 
consumer potential).*/

--1. **Pune (Top Pick)**: Highest revenue (1,258,290) and excellent rent efficiency (only 15,300/month). 
--Backed by a highly satisfied customer base (4.47 rating), it offers an unrivaled risk-to-reward ratio.

--2. **Chennai (High-Value Market)**: Features premium spenders with the highest average ticket size 22,479 per customer 
--and top-tier customer satisfaction 4.52 rating on (944,120) in revenue.

--3. **Jaipur (Growth Hub)**: Boasts the largest community of unique active customers (69). 
--Paired with an incredibly low rent of (10,800/month), it is a highly scalable, low-risk market.

### SQL Concepts Used
*   **Data Definition Language (DDL):** `CREATE TABLE`, `DROP TABLE IF EXISTS`, `PRIMARY KEY`, `FOREIGN KEY` constraints
*   **Aggregations & Grouping:** `SUM()`, `COUNT(DISTINCT)`, `AVG()`, `GROUP BY`, `ORDER BY`
*   **Advanced Analytical Functions:** `DENSE_RANK() OVER (PARTITION BY ...)` and `LAG() OVER (...)` window calculations
*   **Subqueries & Common Table Expressions (CTEs):** Isolated business logical matrices using multi-layered `WITH` blocks
*   **Data Type Casting & Functions:** `::NUMERIC` ratio conversions, `ROUND()` manipulation, and `EXTRACT(MONTH/YEAR FROM ...)` chronological calculations



