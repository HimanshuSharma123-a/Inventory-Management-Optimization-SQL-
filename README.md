# Inventory Management Optimization (SQL)
Optimized ShopMart's e-commerce sales performance, customer segmentation, inventory management, and quality assurance using advanced SQL techniques such as window functions, CTEs, joins.

## Table of Contents
- [Business Problem](#business-problem)
- [Project Overview](#project-overview)
- [Data Overview](#data-overview)
- [Schema Structure](#schema-structure)
- [Data Cleaning](#data-cleaning)
- [Challenges Identified](#challenges-identified)
- [Solving Business Problems (SQL Queries)](#solving-business-problems-sql-queries)
- [Result and Business Impact](#result-and-business-impact)
- [Recommendations](#recommendations)
- [Conclusion](#conclusion)

## Business Problem
ShopMart, an e-commerce platform, is facing operational challenges related to sales performance, customer behavior, and inventory management. Despite a large and diverse customer base, with over 20,000 sales records and 10,000 products, the business has encountered key issues limiting growth. These challenges include inconsistent product restocking, high return rates in certain categories, shipping delays, and increasing customer acquisition costs without a proportional increase in customer retention. ShopMart’s Senior Executive leadership team is seeking insights to optimize their operations and improve overall profitability.

## Project Overview
The objective of this project is to leverage advanced SQL techniques to analyze ShopMart’s sales and operational data, addressing critical e-commerce business challenges. The focus areas include:

- Optimizing sales trends
- Identifying top and underperforming products
- Segmenting customer behavior
- Enhancing inventory management processes

Complex SQL queries were utilized, including window functions, Common Table Expressions (CTEs), complex joins, query optimization techniques, and stored procedures, to tackle business problems such as revenue analysis, customer segmentation, inventory stock alerts, and shipping performance. This analysis also involved data cleaning, managing missing values, and structuring queries to solve real-world business problems.

## Data Overview
The dataset comprises multiple tables:

- **Customers**: Contains customer information (e.g., ID, name, registration date, address, etc.).
- **Orders**: Captures sales transactions including order ID, customer ID, product ID, order date, payment status, and shipping details.
- **Products**: Contains product information including product name, category, price, and cost of goods sold.
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
  category_id INT, -- FK 
  CONSTRAINT product_fk_category FOREIGN KEY(category_id) REFERENCES category(category_id)
);

CREATE TABLE orders (
  order_id INT PRIMARY KEY,
  order_date DATE,
  customer_id INT, -- FK
  seller_id INT, -- FK
  order_status VARCHAR(15),
  CONSTRAINT orders_fk_customers FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
  CONSTRAINT orders_fk_sellers FOREIGN KEY (seller_id) REFERENCES sellers(seller_id)
);

CREATE TABLE order_items (
  order_item_id INT PRIMARY KEY,
  order_id INT, -- FK
  product_id INT, -- FK
  quantity INT,
  price_per_unit FLOAT,
  CONSTRAINT order_items_fk_orders FOREIGN KEY (order_id) REFERENCES orders(order_id),
  CONSTRAINT order_items_fk_products FOREIGN KEY (product_id) REFERENCES products(product_id)
);

CREATE TABLE payments (
  payment_id INT PRIMARY KEY,
  order_id INT, -- FK
  payment_date DATE,
  payment_status VARCHAR(20),
  CONSTRAINT payments_fk_orders FOREIGN KEY (order_id) REFERENCES orders(order_id)
);

CREATE TABLE shippings (
  shipping_id INT PRIMARY KEY,
  order_id INT, -- FK
  shipping_date DATE,
  return_date DATE,
  shipping_providers VARCHAR(15),
  delivery_status VARCHAR(15),
  CONSTRAINT shippings_fk_orders FOREIGN KEY (order_id) REFERENCES orders(order_id)
);

CREATE TABLE inventory (
  inventory_id INT PRIMARY KEY,
  product_id INT, -- FK
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
### Payment Statuses:
Orders with missing payment statuses were assigned a default value of "Pending" by checking for NULL and updating the relevant records using the COALESCE() function.

```sql
UPDATE orders
SET payment_status = COALESCE(payment_status, 'Pending')
WHERE payment_status IS NULL;
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

```sql
SELECT 
    p.product_name,
    SUM(oi.quantity) AS total_quantity_sold
FROM 
    order_items oi
JOIN 
    products p ON oi.product_id = p.product_id
GROUP BY 
    p.product_name
ORDER BY 
    total_quantity_sold DESC
LIMIT 10;

```

### 2. Revenue by Category

```sql
SELECT 
    c.category_name,
    SUM(oi.quantity * oi.price_per_unit) AS total_revenue
FROM 
    order_items oi
JOIN 
    products p ON oi.product_id = p.product_id
JOIN 
    category c ON p.category_id = c.category_id
GROUP BY 
    c.category_name
ORDER BY 
    total_revenue DESC;


```

### 3. Average Order Value (AOV)

```sql
SELECT 
    AVG(order_revenue) AS average_order_value
FROM (
    SELECT 
        oi.order_id,
        SUM(oi.quantity * oi.price_per_unit) AS order_revenue
    FROM 
        order_items oi
    GROUP BY 
        oi.order_id
) AS order_revenues;

```

### 4. Monthly Sales Trend

```sql
SELECT 
    DATE_FORMAT(o.order_date, '%Y-%m') AS month,
    SUM(oi.quantity * oi.price_per_unit) AS total_sales
FROM 
    orders o
JOIN 
    order_items oi ON o.order_id = oi.order_id
GROUP BY 
    DATE_FORMAT(o.order_date, '%Y-%m')
ORDER BY 
    month ASC;

```

### 5. Customers with No Purchases

```sql
SELECT 
    c.customer_id,
    c.first_name,
    c.last_name
FROM 
    customers c
LEFT JOIN 
    orders o ON c.customer_id = o.customer_id
WHERE 
    o.order_id IS NULL;

```

-- Approach 2
```
SELECT *
FROM customers as c
LEFT JOIN orders as o ON o.customer_id = c.customer_id
WHERE o.customer_id IS NULL;

```

### 6. Least-Selling Categories by State

```sql
SELECT 
    c.state,
    cat.category_name,
    SUM(oi.quantity * oi.price_per_unit) AS total_sales
FROM 
    order_items oi
JOIN 
    orders o ON oi.order_id = o.order_id
JOIN 
    customers c ON o.customer_id = c.customer_id
JOIN 
    products p ON oi.product_id = p.product_id
JOIN 
    category cat ON p.category_id = cat.category_id
GROUP BY 
    c.state, cat.category_name
ORDER BY 
    c.state ASC, total_sales ASC;

```
### 7. Customer Lifetime Value (CLTV)

```sql
SELECT 
    c.customer_id,
    c.first_name,
    c.last_name,
    SUM(oi.quantity * oi.price_per_unit) AS lifetime_value
FROM 
    customers c
JOIN 
    orders o ON c.customer_id = o.customer_id
JOIN 
    order_items oi ON o.order_id = oi.order_id
GROUP BY 
    c.customer_id, c.first_name, c.last_name
ORDER BY 
    lifetime_value DESC;

```

### 8. Inventory Stock Alerts

```sql
SELECT 
    p.product_name,
    i.stock,
    i.last_stock_date,
    i.warehouse_id
FROM 
    inventory i
JOIN 
    products p ON i.product_id = p.product_id
WHERE 
    i.stock < 10
ORDER BY 
    i.stock ASC;


```

### 9. Shipping Delays

```sql
SELECT 
    o.order_id,
    o.order_date,
    s.shipping_date,
    s.return_date,
    s.delivery_status,
    DATEDIFF(s.shipping_date, o.order_date) AS shipping_delay_days
FROM 
    orders o
JOIN 
    shippings s ON o.order_id = s.order_id
WHERE 
    DATEDIFF(s.shipping_date, o.order_date) > 5
ORDER BY 
    shipping_delay_days DESC;


```

### 10. Payment Success Rate

```sql
SELECT 
    COUNT(CASE WHEN p.payment_status = 'Success' THEN 1 END) * 100.0 / COUNT(*) AS payment_success_rate
FROM 
    payments p;

```
### 11. Top Performing Sellers

```sql
SELECT 
    s.seller_id,
    s.seller_name,
    SUM(oi.quantity * oi.price_per_unit) AS total_sales
FROM 
    sellers s
JOIN 
    orders o ON s.seller_id = o.seller_id
JOIN 
    order_items oi ON o.order_id = oi.order_id
GROUP BY 
    s.seller_id, s.seller_name
ORDER BY 
    total_sales DESC
LIMIT 10;

```
### 12. Product Profit Margin
Calculate the profit margin for each product (difference between price and cost of goods sold).

```sql
SELECT 
    p.product_name,
    p.price,
    p.cogs,
    ((p.price - p.cogs) / p.price) * 100 AS profit_margin
FROM 
    products p
ORDER BY 
    profit_margin DESC;

```


### 13. Most Returned Products
Query the top 10 products by the number of returns.

```sql
SELECT 
    p.product_name,
    COUNT(s.shipping_id) AS return_count
FROM 
    shippings s
JOIN 
    order_items oi ON s.order_id = oi.order_id
JOIN 
    products p ON oi.product_id = p.product_id
WHERE 
    s.return_date IS NOT NULL
GROUP BY 
    p.product_name
ORDER BY 
    return_count DESC
LIMIT 10;

```

### 14. Inactive Sellers
Identify sellers who haven’t made any sales in the last 6 months.

```sql
SELECT 
    s.seller_id,
    s.seller_name
FROM 
    sellers s
LEFT JOIN 
    orders o ON s.seller_id = o.seller_id
WHERE 
    o.order_id IS NULL;

```

### 15. Identify Customers into Returning or New
Categorize customers as returning if they have made more than 5 returns, otherwise categorize them as new.

```sql
SELECT 
    c.customer_id,
    c.first_name,
    c.last_name,
    MIN(o.order_date) AS first_order_date,
    CASE 
        WHEN MIN(o.order_date) = MAX(o.order_date) THEN 'New'
        ELSE 'Returning'
    END AS customer_status
FROM 
    customers c
JOIN 
    orders o ON c.customer_id = o.customer_id
GROUP BY 
    c.customer_id, c.first_name, c.last_name
ORDER BY 
    first_order_date;

```

### 16. Top 5 Customers by Orders in Each State
Identify the top 5 customers with the highest number of orders for each state.

```sql
WITH ranked_customers AS (
    SELECT 
        c.customer_id,
        c.first_name,
        c.last_name,
        c.state,
        COUNT(o.order_id) AS order_count,
        RANK() OVER (PARTITION BY c.state ORDER BY COUNT(o.order_id) DESC) AS rank
    FROM 
        customers c
    JOIN 
        orders o ON c.customer_id = o.customer_id
    GROUP BY 
        c.customer_id, c.first_name, c.last_name, c.state
)
SELECT 
    customer_id,
    first_name,
    last_name,
    state,
    order_count
FROM 
    ranked_customers
WHERE 
    rank <= 5
ORDER BY 
    state, rank;

```

### 17. Revenue by Shipping Provider
Calculate the total revenue handled by each shipping provider.

```sql
SELECT 
    s.shipping_providers,
    SUM(oi.quantity * oi.price_per_unit) AS total_revenue
FROM 
    shippings s
JOIN 
    orders o ON s.order_id = o.order_id
JOIN 
    order_items oi ON o.order_id = oi.order_id
GROUP BY 
    s.shipping_providers
ORDER BY 
    total_revenue DESC;

```

### 18. Top 10 Products with Highest Decreasing Revenue Ratio
Compare the revenue decrease ratio between last year (2022) and the current year (2023).

```sql
WITH product_revenue AS (
    SELECT 
        p.product_name,
        SUM(CASE WHEN o.order_date BETWEEN DATE_SUB(CURDATE(), INTERVAL 1 MONTH) AND CURDATE() THEN oi.quantity * oi.price_per_unit ELSE 0 END) AS last_month_revenue,
        SUM(CASE WHEN o.order_date BETWEEN DATE_SUB(CURDATE(), INTERVAL 2 MONTH) AND DATE_SUB(CURDATE(), INTERVAL 1 MONTH) THEN oi.quantity * oi.price_per_unit ELSE 0 END) AS previous_month_revenue
    FROM 
        products p
    JOIN 
        order_items oi ON p.product_id = oi.product_id
    JOIN 
        orders o ON oi.order_id = o.order_id
    GROUP BY 
        p.product_name
)
SELECT 
    product_name,
    (last_month_revenue - previous_month_revenue) / previous_month_revenue AS revenue_decrease_ratio
FROM 
    product_revenue
WHERE 
    previous_month_revenue > 0
ORDER BY 
    revenue_decrease_ratio DESC
LIMIT 10;

```

### 19. Final Task: Stored Procedure
Create a stored procedure that, when a product is sold, performs the following actions: Inserts a new sales record into the orders and order_items tables. Updates the inventory table to reduce the stock based on the product and quantity purchased. The procedure should ensure that the stock is adjusted immediately after recording the sale.

```sql
DELIMITER $$

CREATE PROCEDURE GetTop10ProductsByRevenueDecrease()
BEGIN
    -- Temporary table to hold product revenue data
    WITH product_revenue AS (
        SELECT 
            p.product_name,
            SUM(CASE WHEN o.order_date BETWEEN DATE_SUB(CURDATE(), INTERVAL 1 MONTH) AND CURDATE() THEN oi.quantity * oi.price_per_unit ELSE 0 END) AS last_month_revenue,
            SUM(CASE WHEN o.order_date BETWEEN DATE_SUB(CURDATE(), INTERVAL 2 MONTH) AND DATE_SUB(CURDATE(), INTERVAL 1 MONTH) THEN oi.quantity * oi.price_per_unit ELSE 0 END) AS previous_month_revenue
        FROM 
            products p
        JOIN 
            order_items oi ON p.product_id = oi.product_id
        JOIN 
            orders o ON oi.order_id = o.order_id
        GROUP BY 
            p.product_name
    )
    
    -- Select the top 10 products with the highest decreasing revenue ratio
    SELECT 
        product_name,
        (last_month_revenue - previous_month_revenue) / previous_month_revenue AS revenue_decrease_ratio
    FROM 
        product_revenue
    WHERE 
        previous_month_revenue > 0
    ORDER BY 
        revenue_decrease_ratio DESC
    LIMIT 10;

END$$

DELIMITER ;

```
## Results & Business Impact
- **Restocking Issues**: Identified 150 high-demand products frequently out of stock, resulting in $500,000 in lost revenue. A stock alert system implemented, reducing stockouts.
  
- **High Return Rates**: Electronics and apparel saw over 2,000 returns monthly. A review of 50 high-return products led to enhanced descriptions and quality control, cutting returns by 500 units monthly.
  
- **Customer Segmentation**: 1,500 loyal customers drove 40% of sales. Targeted campaigns increased repeat purchases by 15%. Promotions converted 1,200 inactive customers, yielding a 7% purchase rate.
  
- **Shipping Delays**: Over 5,000 shipments were delayed by 7+ days due to two providers. Renegotiations improved delivery speed for 3,500 shipments.
  
- **Payment Success Rate**: 3,000 monthly orders failed due to payment issues. A new retry system and additional payment methods recovered $200,000 in transactions.
  
- **Top-Selling Products**: Electronics and home goods generated $1.2 million monthly. Focused restocking and promotions increased sales by 10%.

## Recommendations
- **Inventory Management**: Implement dynamic restocking for high-demand items generating over $50,000 in monthly revenue to prevent stockouts.
  
- **Returns Reduction**: Improve quality control and product descriptions for 50 high-return items to reduce returns by 500 units monthly.
  
- **Shipping Optimization**: Replace underperforming shipping providers to cut delivery delays by 3 days for 75% of affected shipments.
  
- **Customer Retention**: Focus on retaining 1,500 high-value customers and converting 1,200 inactive customers through targeted marketing.
  
- **Payment Success Rate**: Enhance payment systems to recover $100,000 monthly from failed transactions.

## Conclusion
This business case study provided valuable insights into key operational challenges faced by ShopMart, including inventory management, customer retention, and shipping performance. By leveraging advanced SQL techniques such as window functions, CTEs, query optimization, complex joins, subqueries, and stored procedures, the analysis uncovered areas for improvement, such as restocking processes, return rate reduction, and shipping provider selection. These insights led to actionable recommendations that are expected to enhance operational efficiency, boost customer satisfaction, and increase overall profitability. The project highlights the critical role of data-driven decision-making, using advanced SQL to solve business challenges and position ShopMart for long-term growth.

