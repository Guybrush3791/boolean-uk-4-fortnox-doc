# Intro SQL

[[Intro SQL.pdf|Slides]]

## Learning Objectives
- Differentiate **relational** and **non-relational** databases and know that Postgres is relational.
- Describe SQL as the language used to **query relational databases**.
- Recognise a database as a specialised **file system** managed by a server.
- Create and drop **databases** and **tables**.
- Define **tables, rows (records), columns (fields)** and **primary keys**.
- Use SQL to **INSERT** and **SELECT** data.

## Introduction
A database acts as a *single source of truth* so every user sees consistent, up-to-date data. Applications talk to the database through SQL, while an application server enforces business rules through an API.

## Database Setup
- Postgres can run **locally** or in the **cloud**, such as AWS or GKE.
- We will practise on a **local** instance for easy access.

## Data Representation

![[Intro SQL - Datbase ER|1000]]

## Creating Tables
Define the table name, columns, data types and constraints.

```sql
-- Users table
CREATE TABLE IF NOT EXISTS users (
  id integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  name VARCHAR(20) NOT NULL,
  email VARCHAR(50) NOT NULL UNIQUE
);
-- Products table
CREATE TABLE IF NOT EXISTS products (
  id integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  name VARCHAR(50) NOT NULL,
  price DECIMAL(10,2) NOT NULL,
  discount BOOLEAN NOT NULL DEFAULT FALSE
);
-- Orders table (header for each order, linked to a user)
CREATE TABLE IF NOT EXISTS orders (
  id integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  userId INT NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  CONSTRAINT fk_user FOREIGN KEY (userId) REFERENCES users(id)
);
-- Order_items table (links orders to multiple products)
CREATE TABLE IF NOT EXISTS order_items (
  id integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  orderId INT NOT NULL,
  productId INT NOT NULL,
  quantity INT NOT NULL CHECK (quantity > 0),
  CONSTRAINT fk_order FOREIGN KEY (orderId) REFERENCES orders(id),
  CONSTRAINT fk_product FOREIGN KEY (productId) REFERENCES products(id),
  UNIQUE (orderId, productId)
);
```

Key points:

- **SERIAL** auto-increments.
- **VARCHAR(n)** limits string length.
- Constraints include **PRIMARY KEY**, **NOT NULL** and **UNIQUE**.

## Adding Data (CREATE)

```sql
-- Insert sample data into users table
INSERT INTO users (name, email)
VALUES
  ('Alice Smith', 'alice@example.com'),
  ('Bob Johnson', 'bob@example.com'),
  ('Charlie Davis', 'charlie@example.com');
-- Insert sample data into products table
INSERT INTO products (name, price, discount)
VALUES
  ('Laptop', 999.99, TRUE),
  ('Smartphone', 599.99, FALSE),
  ('Headphones', 149.99, TRUE),
  ('Keyboard', 49.99, FALSE);
-- Insert sample orders (assuming user IDs 1, 2 and 3)
INSERT INTO orders (userId)
VALUES
  (1),
  (2);
-- Insert order items (assuming order IDs 1 and 2, and product IDs 1-4)
INSERT INTO order_items (orderId, productId, quantity)
VALUES
  (1, 1, 1),
  (1, 3, 2),
  (2, 2, 1),
  (2, 4, 1);
```

## Querying Data (READ)

```sql
-- All columns and all rows
SELECT *
FROM products;
-- Only name and price
SELECT name, price
FROM products;
-- Add constraints
SELECT name, price
FROM products
WHERE price < 150
ORDER BY price ASC
LIMIT 1;
```

## Modifying Tables and Data
- **DROP TABLE users;** removes the table and all its data.
- **TRUNCATE users;** deletes the data but keeps the structure.
- **ALTER TABLE** adds, removes or modifies columns and is covered later.

## Viewing Table Metadata

```sql
SELECT table_name, column_name, data_type
FROM information_schema.columns
WHERE table_name = 'products';
```

## Relationships and JOINS
Tables connect through **foreign keys**, and SQL combines them with joins:

```sql
SELECT
  u.id AS user_id,
  u.name AS user_name,
  SUM(p.price * oi.quantity) AS total_order_price
FROM users u
JOIN orders o ON u.id = o.userId
JOIN order_items oi ON o.id = oi.orderId
JOIN products p ON oi.productId = p.id
GROUP BY u.id
ORDER BY total_order_price DESC;
```

## Practice
1. Create a new database in *DBeaver*.
2. Run `CREATE TABLE` for `users`.
3. Insert a few rows, then select them.
4. Truncate the table and verify that it is empty.
5. Drop the table and recreate it to reinforce the workflow.

---

# Links
![[Lessons/2 - Java Back-end/Day 10/__blocks/Links]]
