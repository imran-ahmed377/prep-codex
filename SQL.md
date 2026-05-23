# SQL Cheat Sheet

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

# JOIN Operations

- **INNER JOIN** → Only matching rows
- **LEFT JOIN** → All left table rows + matching rows
- **RIGHT JOIN** → All right table rows + matching rows
- **FULL JOIN** → All rows from both tables

---

## INNER JOIN Example

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

# Scenario 1: Insurance Company

## Policies Table

| policy_id | customer_id | policy_start_date | premium_amount | region |
|-----------|-------------|-------------------|----------------|--------|
| 101 | C1 | 2023-01-15 | 1000 | North |
| 102 | C2 | 2023-02-20 | 1200 | North |
| 103 | C3 | 2023-06-10 | 1500 | South |
| 104 | C4 | 2022-12-01 | 800 | South |

## Claims Table

| claim_id | policy_id | claim_amount | claim_status |
|----------|-----------|--------------|--------------|
| 5001 | 101 | 200 | Approved |
| 5002 | 101 | 150 | Approved |
| 5003 | 103 | 400 | Pending |
| 5004 | 999 | 100 | Approved |

---

## Question 1

### Find total claim amount and total premium collected per region for 2023

```sql
SELECT policies.region,
       SUM(policies.premium_amount) AS total_premium,
       SUM(claims.claim_amount) AS total_claims
FROM policies
LEFT JOIN claims
ON policies.policy_id = claims.policy_id
WHERE policies.policy_start_date
BETWEEN '2023-01-01' AND '2023-12-31'
GROUP BY policies.region;
```

---

## Question 2

### Identify customers with approved claims and their total claim amounts

```sql
SELECT policies.customer_id,
       SUM(claims.claim_amount) AS total_claims
FROM policies
LEFT JOIN claims
ON policies.policy_id = claims.policy_id
WHERE claims.claim_status = 'Approved'
GROUP BY policies.customer_id;
```

---

## Question 3

### Find the top 2 policies with highest total claim amount per region

```sql
SELECT policies.region,
       policies.policy_id,
       SUM(claims.claim_amount) AS total_claims
FROM policies
LEFT JOIN claims
ON policies.policy_id = claims.policy_id
GROUP BY policies.region, policies.policy_id
ORDER BY policies.region, total_claims DESC;
```

---

# Scenario 2: E-commerce Store

## Tables

### Customers
- customer_id
- name
- city

### Orders
- order_id
- customer_id
- order_date
- amount

---

## Question 1

### Find the customer who spent the most money

```sql
SELECT customers.name,
       SUM(orders.amount) AS total_spent
FROM customers
JOIN orders
ON customers.customer_id = orders.customer_id
GROUP BY customers.name
ORDER BY total_spent DESC
LIMIT 1;
```

---

## Question 2

### List all customers who have not made any orders

```sql
SELECT customers.name
FROM customers
LEFT JOIN orders
ON customers.customer_id = orders.customer_id
WHERE orders.customer_id IS NULL;
```

---

## Question 3

### Find the top 3 customers by total spending

```sql
SELECT customers.name,
       SUM(orders.amount) AS total_spent
FROM customers
JOIN orders
ON customers.customer_id = orders.customer_id
GROUP BY customers.name
ORDER BY total_spent DESC
LIMIT 3;
```

---

# Scenario 3: Movie Streaming Platform

## Users Table
- user_id
- name
- subscription_type

## Watch History Table
- user_id
- movie_id
- watch_date

---

## Question 1

### Find users who watched more than 5 movies

```sql
SELECT users.name,
       COUNT(watch_history.movie_id) AS movies_watched
FROM users
JOIN watch_history
ON users.user_id = watch_history.user_id
GROUP BY users.name
HAVING COUNT(watch_history.movie_id) > 5;
```

---

## Question 2

### Show total movies watched per user

```sql
SELECT users.name,
       COUNT(watch_history.movie_id) AS total_movies_watched
FROM users
JOIN watch_history
ON users.user_id = watch_history.user_id
GROUP BY users.name;
```

---

## Question 3

### Find users who never watched any movie

```sql
SELECT users.name
FROM users
LEFT JOIN watch_history
ON users.user_id = watch_history.user_id
WHERE watch_history.user_id IS NULL;
```

---

## Question 4

### Get the most active user

```sql
SELECT users.name,
       COUNT(watch_history.movie_id) AS movies_watched
FROM users
JOIN watch_history
ON users.user_id = watch_history.user_id
GROUP BY users.name
ORDER BY movies_watched DESC
LIMIT 1;
```

---

## Question 5

### Count users by subscription type

```sql
SELECT subscription_type,
       COUNT(user_id) AS user_count
FROM users
GROUP BY subscription_type;
```

---

# ALTER TABLE

## Drop Column

```sql
ALTER TABLE Customers
DROP COLUMN Age;
```

Deletes the `Age` column.

---

# DROP TABLE

```sql
DROP TABLE Customers;
```

Deletes the entire `Customers` table.

---

# DELETE Row Example

```sql
DELETE FROM Customers
WHERE CustomerID = 1;
```

Deletes the row where `CustomerID = 1`.

---