# SQL Concepts

## Table of Content

- [Create Table Statement](#create-table-statement)
- [SELECT Statement](#select-statement)
- [DISTINCT Keyword](#distinct-keyword)
- [WHERE Clause](#where-clause)
- [ORDER BY Clause](#order-by-clause)
- [AND, OR, NOT Operators](#and-or-not-operators)
- [NULL Values](#null-values)
- [INSERT Statement](#insert-statement)
- [UPDATE Statement](#update-statement)
- [DELETE Statement](#delete-statement)
- [Aggregate Functions](#aggregate-functions)
  - [MIN](#min) · [MAX](#max) · [AVG](#avg) · [SUM](#sum) · [COUNT](#count)
- [LIKE Operator](#like-operator)
- [ALTER TABLE](#alter-table)
  - [DROP Column](#drop-column) · [DROP Table](#drop-table) · [DELETE Row](#delete-row)
- [JOIN Operations](#join-operations)
  - [INNER JOIN Example](#inner-join-example)
- [Scenario 1: Insurance Company](#scenario-1-insurance-company)
  - [Policies Table](#policies-table)
  - [Claims Table](#claims-table)
  - [Question 1: Find total claim amount and total premium collected per region for 2023 (LEFT JOIN)](#question-1-find-total-claim-amount-and-total-premium-collected-per-region-for-2023-left-join)
  - [Question 2: Identify customers with approved claims and their total claim amounts](#question-2-identify-customers-with-approved-claims-and-their-total-claim-amounts)
  - [Question 3: Find the top 2 policies with highest total claim amount per region](#question-3-find-the-top-2-policies-with-highest-total-claim-amount-per-region)
- [Scenario 2: E-commerce Store](#scenario-2-e-commerce-store)
  - [Tables](#tables)
  - [Question 1: Find the customer who spent the most money](#question-1-find-the-customer-who-spent-the-most-money)
  - [Question 2: List all customers who have not made any orders](#question-2-list-all-customers-who-have-not-made-any-orders)
  - [Question 3: Find the top 3 customers by total spending](#question-3-find-the-top-3-customers-by-total-spending)
- [Scenario 3: Movie Streaming Platform](#scenario-3-movie-streaming-platform)
  - [Users Table](#users-table)
  - [Watch History Table](#watch-history-table)
  - [Question 1: Find users who watched more than 5 movies](#question-1-find-users-who-watched-more-than-5-movies)
  - [Question 2: Show total movies watched per user](#question-2-show-total-movies-watched-per-user)
  - [Question 3: Find users who never watched any movie](#question-3-find-users-who-never-watched-any-movie)
  - [Question 4: Get the most active user](#question-4-get-the-most-active-user)
  - [Question 5: Count users by subscription type](#question-5-count-users-by-subscription-type)
- [SQL Practice Questions and Solutions](#sql-practice-questions-and-solutions)
  - [1. Second Highest Salary](#1-second-highest-salary)
  - [2. Duplicate Emails](#2-duplicate-emails)
  - [3. Customers Without Orders](#3-customers-without-orders)
  - [4. Total Sales Per Product](#4-total-sales-per-product)
  - [5. Rank Employees by Salary](#5-rank-employees-by-salary)
  - [6. Monthly Sales Report](#6-monthly-sales-report)

---

## Create Table Statement

```sql
CREATE TABLE Customers (
    CustomerID int,
    CustomerName varchar(255),
    City varchar(255),
    Country varchar(255),
    Age int
);
```

Creates a table named `Customers` with the columns:

- `CustomerID`
- `CustomerName`
- `City`
- `Country`
- `Age`

---

## SELECT Statement

```sql
SELECT * FROM Customers;
```

Shows the entire `Customers` table.

```sql
SELECT CustomerName FROM Customers;
```

Shows only the `CustomerName` column.

```sql
SELECT CustomerName, CustomerAddress FROM Customers;
```

Shows both `CustomerName` and `CustomerAddress` columns.

---

## DISTINCT Keyword

```sql
SELECT DISTINCT CustomerName FROM Customers;
```

Shows all unique customer names.

```sql
SELECT COUNT(DISTINCT City)
FROM Customers;
```

Shows the number of unique cities.

---

## WHERE Clause

```sql
SELECT * FROM Customers
WHERE Country = 'Mexico';
```

Shows all customers from Mexico.

```sql
SELECT * FROM Products
WHERE Price >= 30;
```

Shows all products with price greater than or equal to 30.

```sql
SELECT * FROM Customers
WHERE City IN ('Paris', 'London');
```

Shows all customers located in Paris or London.

---

## ORDER BY Clause

```sql
SELECT * FROM Products
ORDER BY Price;
```

Sorts products by price in ascending order.

```sql
SELECT * FROM Products
ORDER BY Price ASC;
```

Ascending order by price.

```sql
SELECT * FROM Products
ORDER BY Price DESC;
```

Descending order by price.

---

## AND, OR, NOT Operators

```sql
SELECT * FROM Customers
WHERE Country = 'Spain' AND CustomerName LIKE 'G%';
```

Shows customers from Spain whose names start with `G`.

```sql
SELECT * FROM Customers
WHERE Country = 'Spain' OR Country = 'Germany';
```

Shows customers from Spain or Germany.

```sql
SELECT * FROM Customers
WHERE NOT Country = 'Spain';
```

Shows customers not from Spain.

---

## NULL Values

```sql
SELECT * FROM Customers
WHERE Address IS NULL;
```

Shows rows where `Address` is NULL.

```sql
SELECT * FROM Customers
WHERE Address IS NOT NULL;
```

Shows rows where `Address` is not NULL.

---

## INSERT Statement

```sql
INSERT INTO Customers (CustomerName, City, Country, Age)
VALUES ('John', 'Toronto', 'Canada', 26);
```

Adds a new customer row.

---

## UPDATE Statement

```sql
UPDATE Customers
SET CustomerName = 'John Cena', City = 'LA'
WHERE CustomerID = 1;
```

Updates the customer name and city where `CustomerID = 1`.

---

## DELETE Statement

```sql
DELETE FROM Customers
WHERE CustomerID = 1;
```

Deletes the row where `CustomerID = 1`.

---

## Aggregate Functions

### MIN

```sql
SELECT MIN(Price)
FROM Products;
```

Returns the minimum price.

### MAX

```sql
SELECT MAX(Price)
FROM Products;
```

Returns the maximum price.

### AVG

```sql
SELECT AVG(Price)
FROM Products;
```

Returns the average price.

### SUM

```sql
SELECT SUM(Price)
FROM Products;
```

Returns the total sum of prices.

### COUNT

```sql
SELECT COUNT(Price)
FROM Products;
```

Returns the total number of prices.

---

## LIKE Operator

```sql
SELECT * FROM Customers
WHERE CustomerName LIKE 'a%';
```

Names starting with `a`.

```sql
SELECT * FROM Customers
WHERE CustomerName LIKE '%a'
   OR CustomerName LIKE '%e';
```

Names ending with `a` or `e`.

```sql
SELECT * FROM Customers
WHERE City LIKE '%on%';
```

Cities containing `on`.

---

## ALTER TABLE

### DROP Column

```sql
ALTER TABLE Customers
DROP COLUMN Age;
```

Deletes the `Age` column.

### DROP Table

```sql
DROP TABLE Customers;
```

Deletes the entire `Customers` table.

### DELETE Row

```sql
DELETE FROM Customers
WHERE CustomerID = 1;
```

Deletes the row where `CustomerID = 1`.

---

## JOIN Operations

- **INNER JOIN** → Only matching rows
- **LEFT JOIN** → All left table rows + matching rows
- **RIGHT JOIN** → All right table rows + matching rows
- **FULL JOIN** → All rows from both tables

### INNER JOIN Example

```sql
SELECT Orders.OrderID,
       Customers.CustomerName,
       Orders.OrderDate
FROM Orders
INNER JOIN Customers
ON Orders.CustomerID = Customers.CustomerID;
```

Shows matching order and customer information.

---

## Scenario 1: Insurance Company

### Policies Table

| policy_id | customer_id | policy_start_date | premium_amount | region |
|-----------|-------------|-------------------|----------------|--------|
| 101       | C1          | 2023-01-15        | 1000           | North  |
| 102       | C2          | 2023-02-20        | 1200           | North  |
| 103       | C3          | 2023-06-10        | 1500           | South  |
| 104       | C4          | 2022-12-01        | 800            | South  |

### Claims Table

| claim_id | policy_id | claim_amount | claim_status |
|----------|-----------|--------------|--------------|
| 5001     | 101       | 200          | Approved     |
| 5002     | 101       | 150          | Approved     |
| 5003     | 103       | 400          | Pending      |
| 5004     | 999       | 100          | Approved     |

### Question 1: Find total claim amount and total premium collected per region for 2023 (LEFT JOIN)

```sql
SELECT policies.region,
       SUM(policies.premium_amount) AS total_premium,
       SUM(claims.claim_amount) AS total_claims
FROM policies
LEFT JOIN claims ON policies.policy_id = claims.policy_id
WHERE policies.policy_start_date
BETWEEN '2023-01-01' AND '2023-12-31'
GROUP BY policies.region;
```

**Output:**

| region | total_premium | total_claims |
|--------|--------------:|-------------:|
| North  |          2200 |          350 |
| South  |          1500 |          400 |

### Question 2: Identify customers with approved claims and their total claim amounts

```sql
SELECT policies.customer_id,
       SUM(claims.claim_amount) AS total_claims
FROM policies
LEFT JOIN claims ON policies.policy_id = claims.policy_id
WHERE claims.claim_status = 'Approved'
GROUP BY policies.customer_id;
```

**Output:**

| customer_id | total_claims |
|-------------|--------------|
| C1          | 350          |

### Question 3: Find the top 2 policies with highest total claim amount per region

```sql
SELECT policies.region,
       policies.policy_id,
       SUM(claims.claim_amount) AS total_claims
FROM policies
LEFT JOIN claims ON policies.policy_id = claims.policy_id
GROUP BY policies.region, policies.policy_id
ORDER BY policies.region, total_claims DESC;
```

**Output:**

| region | policy_id | total_claims |
|--------|-----------|--------------|
| North  | 101       | 350          |
| North  | 102       | NULL         |
| South  | 103       | 400          |
| South  | 104       | NULL         |

---

## Scenario 2: E-commerce Store

### Tables

**Customers**

- customer_id
- name
- city

**Orders**

- order_id
- customer_id
- order_date
- amount

### Question 1: Find the customer who spent the most money

```sql
SELECT customers.name,
       SUM(orders.amount) AS total_spent
FROM customers
JOIN orders ON customers.customer_id = orders.customer_id
GROUP BY customers.name
ORDER BY total_spent DESC
LIMIT 1;
```

### Question 2: List all customers who have not made any orders

```sql
SELECT customers.name
FROM customers
LEFT JOIN orders ON customers.customer_id = orders.customer_id
WHERE orders.customer_id IS NULL;
```

### Question 3: Find the top 3 customers by total spending

```sql
SELECT customers.name,
       SUM(orders.amount) AS total_spent
FROM customers
JOIN orders ON customers.customer_id = orders.customer_id
GROUP BY customers.name
ORDER BY total_spent DESC
LIMIT 3;
```

---

## Scenario 3: Movie Streaming Platform

### Users Table

- user_id
- name
- subscription_type

### Watch History Table

- user_id
- movie_id
- watch_date

### Question 1: Find users who watched more than 5 movies

```sql
SELECT users.name,
       COUNT(watch_history.movie_id) AS movies_watched
FROM users
JOIN watch_history ON users.user_id = watch_history.user_id
GROUP BY users.name
HAVING COUNT(watch_history.movie_id) > 5;
```

### Question 2: Show total movies watched per user

```sql
SELECT users.name,
       COUNT(watch_history.movie_id) AS total_movies_watched
FROM users
JOIN watch_history ON users.user_id = watch_history.user_id
GROUP BY users.name;
```

### Question 3: Find users who never watched any movie

```sql
SELECT users.name
FROM users
LEFT JOIN watch_history ON users.user_id = watch_history.user_id
WHERE watch_history.user_id IS NULL;
```

### Question 4: Get the most active user

```sql
SELECT users.name,
       COUNT(watch_history.movie_id) AS movies_watched
FROM users
JOIN watch_history ON users.user_id = watch_history.user_id
GROUP BY users.name
ORDER BY movies_watched DESC
LIMIT 1;
```

### Question 5: Count users by subscription type

```sql
SELECT subscription_type,
       COUNT(user_id) AS user_count
FROM users
GROUP BY subscription_type;
```

---

# SQL Practice Questions and Solutions

## 1. Second Highest Salary

### Table: employees

| employee_id | name    | salary |
|-------------|---------|--------|
| 1           | Alice   | 80000  |
| 2           | Bob     | 60000  |
| 3           | Charlie | 90000  |
| 4           | David   | 50000  |

### Question

Find the second highest salary.

### Solution

```sql
SELECT MAX(salary) AS second_highest_salary
FROM employees
WHERE salary < (
    SELECT MAX(salary)
    FROM employees
);
```

### Explanation

- First, the subquery finds the highest salary.
- Then the outer query finds the maximum salary smaller than the highest salary.

---

## 2. Duplicate Emails

### Table: users

| user_id | email      |
|---------|------------|
| 1       | a@test.com |
| 2       | b@test.com |
| 3       | a@test.com |
| 4       | c@test.com |

### Question

Find duplicate email addresses.

### Solution

```sql
SELECT 
    email,
    COUNT(*) AS duplicate_count
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
```

### Explanation

- `GROUP BY` groups rows with the same email.
- `COUNT(*)` counts occurrences.
- `HAVING COUNT(*) > 1` filters duplicate emails.

---

## 3. Customers Without Orders

### Table: customers

| customer_id | customer_name |
|-------------|---------------|
| 1           | John          |
| 2           | Sarah         |
| 3           | Mike          |

### Table: orders

| order_id | customer_id | amount |
|----------|-------------|--------|
| 101      | 1           | 200    |
| 102      | 2           | 150    |

### Question

Find customers who never placed an order.

### Solution

```sql
SELECT 
    customers.customer_name
FROM customers
LEFT JOIN orders ON customers.customer_id = orders.customer_id
WHERE orders.order_id IS NULL;
```

### Explanation

- `LEFT JOIN` keeps all customers.
- Customers without matching orders will have `NULL` values in the orders table.
- The query filters those NULL rows.

---

## 4. Total Sales Per Product

### Table: products

| product_id | product_name |
|------------|--------------|
| 1          | Laptop       |
| 2          | Mouse        |

### Table: sales

| sale_id | product_id | quantity | price |
|---------|------------|----------|-------|
| 1       | 1          | 2        | 1000  |
| 2       | 2          | 5        | 20    |
| 3       | 1          | 1        | 1000  |

### Question

Calculate the total sales amount for each product.

### Solution

```sql
SELECT 
    products.product_name,
    SUM(sales.quantity * sales.price) AS total_sales
FROM sales
JOIN products ON sales.product_id = products.product_id
GROUP BY products.product_name;
```

### Explanation

- `quantity * price` calculates total amount per sale.
- `SUM()` adds all sales amounts for each product.
- `GROUP BY` groups results by product.

---

## 5. Rank Employees by Salary

### Table: employees

| employee_id | name    | salary |
|-------------|---------|--------|
| 1           | Alice   | 80000  |
| 2           | Bob     | 60000  |
| 3           | Charlie | 90000  |
| 4           | David   | 80000  |

### Question

Rank employees based on salary.

### Solution

```sql
SELECT
    name,
    salary,
    RANK() OVER (
        ORDER BY salary DESC
    ) AS salary_rank
FROM employees;
```

### Explanation

- `RANK()` assigns rankings based on salary.
- `ORDER BY salary DESC` ranks highest salary first.
- Employees with equal salaries receive the same rank.

---

## 6. Monthly Sales Report

### Table: orders

| order_id | order_date | amount |
|----------|------------|--------|
| 1        | 2025-01-10 | 100    |
| 2        | 2025-01-15 | 200    |
| 3        | 2025-02-01 | 300    |

### Question

Generate total sales by month.

### MySQL Solution

```sql
SELECT
    DATE_FORMAT(order_date, '%Y-%m') AS month,
    SUM(amount) AS total_sales
FROM orders
GROUP BY DATE_FORMAT(order_date, '%Y-%m')
ORDER BY month;
```

### PostgreSQL Solution

```sql
SELECT
    TO_CHAR(order_date, 'YYYY-MM') AS month,
    SUM(amount) AS total_sales
FROM orders
GROUP BY TO_CHAR(order_date, 'YYYY-MM')
ORDER BY month;
```

### Explanation

- The date is formatted into `YYYY-MM`.
- `SUM(amount)` calculates total monthly sales.
- `GROUP BY` combines rows for the same month.