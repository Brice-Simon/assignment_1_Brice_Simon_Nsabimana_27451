# PL/SQL Assignment 1 – Sunrise Supermarket Report

**Student Name:** Brice Simon Nsabimana
**Student ID:** 27451
**Database Used:** Oracle Database 21c
**SQL Tool:** Oracle SQL Developer 26.2.0

---

## 1. About the Business

Sunrise Supermarket sells everyday products to customers. Customers place orders containing different products. The purpose of this assignment is to help management understand customer purchases, spending habits, and sales trends using SQL queries.

## 2. Database Setup

I created four tables: `customers`, `products`, `orders`, and `order_items`.

I inserted 6 customers, 8 products, 15 orders, and 25 order items. The sample data is spread across different dates and product categories to help demonstrate the queries.

The following screenshot shows my database setup.

<img width="1083" height="899" alt="Database setup" src="https://github.com/user-attachments/assets/7e6dbacd-516f-4326-856a-233d36e733ec" />

---

## 3. SQL Queries and Results

### Question 1: INNER JOIN – Orders and Customers

**What it does:** Displays each order with the customer's name, city, and order date.

**Why I used it:** The JOIN combines order information with the details of the customer who placed each order.

<img width="1685" height="876" alt="Question 1 results" src="https://github.com/user-attachments/assets/7f3316cb-aa1a-4d5d-9d5a-97b0cd4b4067" />

```sql
SELECT
    o.order_id,
    c.customer_name,
    c.city,
    o.order_date
FROM orders o
JOIN customers c
    ON o.customer_id = c.customer_id
ORDER BY o.order_id;
```

### Question 2: JOIN – Order Items and Products

**What it does:** Shows the products included in each order, together with the product name, category, price, and quantity.

**Why I used it:** The JOIN connects order item records with product details, making it easier to see what customers purchased.

<img width="1287" height="743" alt="Question 2 results" src="https://github.com/user-attachments/assets/efaa40be-6c1d-4c89-be22-69f4bb92d32e" />

```sql
SELECT
    oi.order_item_id,
    oi.order_id,
    p.product_name,
    p.category,
    p.price,
    oi.quantity
FROM order_items oi
JOIN products p
    ON oi.product_id = p.product_id
ORDER BY oi.order_item_id;
```

### Question 3: LEFT JOIN – Customers and Orders

**What it does:** Lists all customers, including those who have never placed an order.

**Why I used it:** A LEFT JOIN keeps every customer in the results, even when there is no matching order. This helps identify customers who have not purchased anything.

<img width="1711" height="914" alt="Question 3 results" src="https://github.com/user-attachments/assets/e6197502-cd55-46db-9017-ae2ef40d65d9" />

```sql
SELECT
    c.customer_id,
    c.customer_name,
    c.city,
    o.order_id,
    o.order_date
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id
ORDER BY c.customer_id, o.order_date;
```

### Question 4: CTE – Customers Spending Above Average

**What it does:** Calculates the total amount spent by each customer and returns only those whose spending is above the average spending of all customers.

**Why I used it:** The CTE calculates each customer's spending first. The main query then compares those totals with the average.

<img width="1273" height="596" alt="Question 4 results" src="https://github.com/user-attachments/assets/ccf00064-e2a0-47f1-b36b-ac3f004b402b" />

```sql
WITH customer_spending AS (
    SELECT
        c.customer_id,
        c.customer_name,
        NVL(SUM(oi.quantity * p.price), 0) AS total_spent
    FROM customers c
    LEFT JOIN orders o
        ON c.customer_id = o.customer_id
    LEFT JOIN order_items oi
        ON o.order_id = oi.order_id
    LEFT JOIN products p
        ON oi.product_id = p.product_id
    GROUP BY c.customer_id, c.customer_name
)
SELECT
    customer_id,
    customer_name,
    total_spent
FROM customer_spending
WHERE total_spent > (
    SELECT AVG(total_spent)
    FROM customer_spending
)
ORDER BY total_spent DESC;
```

### Question 5: DENSE_RANK – Rank Customers by Spending

**What it does:** Ranks customers according to their total spending, from highest to lowest.

**Why I used it:** `DENSE_RANK()` assigns the same rank to customers with equal spending, without skipping the next rank.

<img width="1477" height="827" alt="Question 5 results" src="https://github.com/user-attachments/assets/034e01e2-5ffd-4ce9-b25d-c32527310e90" />

```sql
WITH customer_spending AS (
    SELECT
        c.customer_id,
        c.customer_name,
        NVL(SUM(oi.quantity * p.price), 0) AS total_spent
    FROM customers c
    LEFT JOIN orders o
        ON c.customer_id = o.customer_id
    LEFT JOIN order_items oi
        ON o.order_id = oi.order_id
    LEFT JOIN products p
        ON oi.product_id = p.product_id
    GROUP BY c.customer_id, c.customer_name
)
SELECT
    customer_id,
    customer_name,
    total_spent,
    DENSE_RANK() OVER (
        ORDER BY total_spent DESC
    ) AS spend_rank
FROM customer_spending
ORDER BY total_spent DESC, customer_id;
```

### Question 6: ROW_NUMBER – Number Each Customer's Orders

**What it does:** Numbers each customer's orders starting from 1, based on the order date.

**Why I used it:** `ROW_NUMBER()` gives each order a sequence number within its customer's orders.

<img width="1379" height="816" alt="Question 6 results" src="https://github.com/user-attachments/assets/786fecb7-6c81-46a6-bef9-c92c098050e9" />

```sql
SELECT
    c.customer_name,
    o.order_id,
    o.order_date,
    ROW_NUMBER() OVER (
        PARTITION BY o.customer_id
        ORDER BY o.order_date, o.order_id
    ) AS order_sequence_number
FROM orders o
JOIN customers c
    ON o.customer_id = c.customer_id
ORDER BY c.customer_name, order_sequence_number;
```

### Question 7: Running Total of Revenue Over Time

**What it does:** Calculates the revenue for each order and shows the cumulative revenue from the first order through the current order.

**Why I used it:** The running total helps show how sales accumulate over time. Orders are arranged by date and order ID.

<img width="1415" height="858" alt="Question 7 results" src="https://github.com/user-attachments/assets/b869f772-488d-4d04-a9e9-cd57cf2e184e" />

```sql
WITH daily_order_revenue AS (
    SELECT
        o.order_id,
        o.order_date,
        SUM(oi.quantity * p.price) AS order_revenue
    FROM orders o
    JOIN order_items oi
        ON o.order_id = oi.order_id
    JOIN products p
        ON oi.product_id = p.product_id
    GROUP BY o.order_id, o.order_date
)
SELECT
    order_id,
    order_date,
    order_revenue,
    SUM(order_revenue) OVER (
        ORDER BY order_date, order_id
        ROWS BETWEEN UNBOUNDED PRECEDING
        AND CURRENT ROW
    ) AS running_total_revenue
FROM daily_order_revenue
ORDER BY order_date, order_id;
```

### Question 8: LAG – Days Between Orders

**What it does:** Shows the time between orders for customers who have placed more than one order.

**Why I used it:** `LAG()` retrieves the previous order date, allowing me to calculate how many days passed before the next order.

<img width="1509" height="861" alt="Question 8 results" src="https://github.com/user-attachments/assets/76e22830-30ba-4ed6-8931-ceb92373ebaa" />

```sql
WITH customer_order_gaps AS (
    SELECT
        o.customer_id,
        c.customer_name,
        o.order_id,
        o.order_date,
        LAG(o.order_date) OVER (
            PARTITION BY o.customer_id
            ORDER BY o.order_date, o.order_id
        ) AS previous_order_date,
        COUNT(o.order_id) OVER (
            PARTITION BY o.customer_id
        ) AS total_customer_orders
    FROM orders o
    JOIN customers c
        ON o.customer_id = c.customer_id
)
SELECT
    customer_id,
    customer_name,
    order_id,
    order_date,
    previous_order_date,
    (order_date - previous_order_date) AS days_since_last_order
FROM customer_order_gaps
WHERE total_customer_orders > 1
ORDER BY customer_id, order_date, order_id;
```

---

## 4. Business Interpretation

The queries can help Sunrise Supermarket understand customer activity and sales.

* **Customer spending:** The spending totals and ranking show how much each customer contributes to sales. Management can use this information to identify customers with higher spending.
* **Inactive customers:** The LEFT JOIN in Question 3 helps identify customers who have not placed any orders. Management could consider ways to encourage them to shop.
* **Buying patterns:** The results of Question 8 show the number of days between repeat orders. This information can help management understand how often customers return.
* **Sales over time:** The running revenue in Question 7 shows how total sales accumulate as orders are placed.

---

## 5. Problems Faced and Solutions

**Problem:** When I ran the setup script more than once, I encountered errors such as `ORA-00942` and `ORA-00001`.

**Solution:** I added `DROP TABLE ... CASCADE CONSTRAINTS` statements before creating the tables. This allowed me to remove the existing tables and their constraints before running the setup script again.

---

## 6. How to Run the Code

1. Open Oracle SQL Developer and connect to the Oracle Database 21c database.
2. Open a SQL worksheet.
3. Run the database setup script to create the tables and insert the sample data.
4. Run any of the eight query blocks to view the results.
5. Check the screenshots in this README to see the outputs.

**Note:** The setup script should be run before the query blocks.


This assignment gave me practice using SQL JOINs, a CTE, and window functions. I used them to combine customer and order information, calculate spending, rank customers, number orders, and examine sales over time.
