# Optimizing Inventory Management (SQL)
Improved ShopMart's online sales, customer groups, inventory management, and quality checks using advanced SQL techniques like window functions, Common Table Expressions (CTEs), and joins.

## Contents
- [Problem](#problem)
- [Project Summary](#project-summary)
- [Data Summary](#data-summary)
- [Database Structure](#database-structure)
- [Data Cleanup](#data-cleanup)
- [Challenges Found](#challenges-found)
- [Solving Problems with SQL](#solving-problems-with-sql)
- [Results and Impact](#results-and-impact)
- [Suggestions](#suggestions)
- [End](#end)

## Problem
ShopMart, an online store, faces problems with sales, customer behavior, and inventory management. Despite having many customers and products, they face key issues that limit growth. These problems include irregular restocking, high return rates in some categories, shipping delays, and rising customer acquisition costs without a corresponding increase in customer retention. ShopMart's leadership team is looking for insights to improve their operations and profitability.

## Project Summary
The goal of this project is to use advanced SQL techniques to analyze ShopMart's sales and operational data to address major business challenges. The main areas of focus are:

- Improving sales trends
- Identifying best and worst-selling products
- Grouping customers based on their behavior
- Enhancing inventory management

Complex SQL queries were used, including window functions, Common Table Expressions (CTEs), complex joins, query optimization techniques, and stored procedures, to tackle problems such as revenue analysis, customer grouping, inventory alerts, and shipping performance. The analysis also involved cleaning data, managing missing values, and structuring queries to solve real business problems.

## Data Summary
The dataset includes several tables:

- **Customers**: Contains customer details ( ID, name, registration date, address, etc.).
- **Orders**: Captures sales transactions including order ID, customer ID, product ID, order date, payment status, and shipping details.
- **Products**: Contains product details including product name, category, price, and cost of goods sold.
- **Inventory**: Tracks stock levels and restock dates for each product.
- **Returns**: Contains details on product returns including return dates and reasons.
- **Shipping Providers**: Includes information about the shipping provider used for each order, along with their average delivery times.

### Schema Structure

```sql
CREATE TABLE category (
  category_id INT PRIMARY KEY,
  category_name VARCHAR(20)
);

CREATE TABLE customers (
  customer_id INT PRIMARY KEY,
  first_name VARCHAR(20),
  last_name VARCHAR(20),
  state VARCHAR(20),
  address VARCHAR(5) DEFAULT ('xxxx')
);

CREATE TABLE sellers (
  seller_id INT PRIMARY KEY,
  seller_name VARCHAR(25),
  origin VARCHAR(15)
);

CREATE TABLE products (
  product_id INT PRIMARY KEY,
  product_name VARCHAR(50),
  price FLOAT,
  cogs FLOAT,
  category_id INT,  
  CONSTRAINT product_fk_category FOREIGN KEY(category_id) REFERENCES category(category_id)
);

CREATE TABLE orders (
  order_id INT PRIMARY KEY,
  order_date DATE,
  customer_id INT, 
  seller_id INT, 
  order_status VARCHAR(15),
  CONSTRAINT orders_fk_customers FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
  CONSTRAINT orders_fk_sellers FOREIGN KEY (seller_id) REFERENCES sellers(seller_id)
);

CREATE TABLE order_items (
  order_item_id INT PRIMARY KEY,
  order_id INT, 
  product_id INT, 
  quantity INT,
  price_per_unit FLOAT,
  CONSTRAINT order_items_fk_orders FOREIGN KEY (order_id) REFERENCES orders(order_id),
  CONSTRAINT order_items_fk_products FOREIGN KEY (product_id) REFERENCES products(product_id)
);

CREATE TABLE payments (
  payment_id INT PRIMARY KEY,
  order_id INT, 
  payment_date DATE,
  payment_status VARCHAR(20),
  CONSTRAINT payments_fk_orders FOREIGN KEY (order_id) REFERENCES orders(order_id)
);

CREATE TABLE shippings (
  shipping_id INT PRIMARY KEY,
  order_id INT, 
  shipping_date DATE,
  return_date DATE,
  shipping_providers VARCHAR(15),
  delivery_status VARCHAR(15),
  CONSTRAINT shippings_fk_orders FOREIGN KEY (order_id) REFERENCES orders(order_id)
);

CREATE TABLE inventory (
  inventory_id INT PRIMARY KEY,
  product_id INT, 
  stock INT,
  warehouse_id INT,
  last_stock_date DATE,
  CONSTRAINT inventory_fk_products FOREIGN KEY (product_id) REFERENCES products(product_id)
);
```
## Data Cleaning
The dataset was cleaned using a series of SQL operations aimed at ensuring data integrity and consistency:

### Removing Duplicates:
Duplicate records in both the customer and order tables were identified using a combination of `ROW_NUMBER()` window functions and `PARTITION BY` clauses. Rows with duplicate `customer_id` and `order_id` were flagged, and only the first occurrence of each was retained, while the rest were deleted using `DELETE` with a CTE for efficient removal.

```sql
WITH cte_duplicates AS (
    SELECT 
        customer_id,
        ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY customer_id) AS row_num
    FROM customers
)
DELETE FROM customers
WHERE customer_id IN (
    SELECT customer_id
    FROM cte_duplicates
    WHERE row_num > 1
);
```

## Handling Null Values
Null values were handled contextually using SQL functions and logic:

### Customer Addresses:
For records where the address was missing, `COALESCE()` was used to replace null values with a default placeholder ("Unknown Address") while ensuring that legitimate non-null values remained unchanged.

```sql
UPDATE customers
SET address = COALESCE(address, 'Unknown Address')
WHERE address IS NULL;

```
### Shipping Information:
For null return_date fields, no updates were made as nulls were valid entries indicating unreturned shipments. This was handled by allowing null values to remain unchanged unless explicitly required for reporting.

## Challenges Identified
Through preliminary data exploration and discussions with ShopMart’s leadership team, the following business challenges were identified:

- **Low Product Availability**: Inconsistent restocking is causing frequent stockouts in key product categories, leading to lost sales opportunities.
- **High Return Rates**: Certain product categories have disproportionately high return rates, which negatively impacts revenue.
- **Shipping Delays**: There are significant delays in shipments, leading to poor customer satisfaction and increased churn.
- **Low Customer Retention**: High customer acquisition costs are not translating into long-term customer loyalty, with retention rates remaining low despite marketing efforts.
- **Payment Success Rate**: A large number of orders have failed payment statuses, contributing to missed revenue opportunities.

## Solving Business Problems

### 1. Top Selling Products:


![Dashboard Screenshot](https://github.com/HimanshuSharma123-a/Inventory-Management-Optimization-SQL-/blob/main/a.png)

### 2. Revenue by Category


![Dashboard Screenshot](https://github.com/HimanshuSharma123-a/Inventory-Management-Optimization-SQL-/blob/main/B.png)

### 3. Average Order Value (AOV)


![Dashboard Screenshot](https://github.com/HimanshuSharma123-a/Inventory-Management-Optimization-SQL-/blob/main/C.png)

### 4. Monthly Sales Trend


![Dashboard Screenshot](https://github.com/HimanshuSharma123-a/Inventory-Management-Optimization-SQL-/blob/main/D.png)

### 5. Customers with No Purchases


![Dashboard Screenshot](https://github.com/HimanshuSharma123-a/Inventory-Management-Optimization-SQL-/blob/main/E.png)

### 6. Least-Selling Categories by State


![Dashboard Screenshot](https://github.com/HimanshuSharma123-a/Inventory-Management-Optimization-SQL-/blob/main/F.png)

### 7. Customer Lifetime Value (CLTV)


![Dashboard Screenshot](https://github.com/HimanshuSharma123-a/Inventory-Management-Optimization-SQL-/blob/main/G.png)

### 8. Inventory Stock Alerts


![Dashboard Screenshot](https://github.com/HimanshuSharma123-a/Inventory-Management-Optimization-SQL-/blob/main/H.png)

### 9. Shipping Delays


![Dashboard Screenshot](https://github.com/HimanshuSharma123-a/Inventory-Management-Optimization-SQL-/blob/main/I.png)

### 10. Payment Success Rate


![Dashboard Screenshot](https://github.com/HimanshuSharma123-a/Inventory-Management-Optimization-SQL-/blob/main/J.png)

### 11. Top Performing Sellers


![Dashboard Screenshot](https://github.com/HimanshuSharma123-a/Inventory-Management-Optimization-SQL-/blob/main/K.png)

### 12. Product Profit Margin
Calculate the profit margin for each product (difference between price and cost of goods sold).


![Dashboard Screenshot](https://github.com/HimanshuSharma123-a/Inventory-Management-Optimization-SQL-/blob/main/L.png)

### 13. Most Returned Products
Query the top 10 products by the number of returns.


![Dashboard Screenshot](https://github.com/HimanshuSharma123-a/Inventory-Management-Optimization-SQL-/blob/main/M.png)


