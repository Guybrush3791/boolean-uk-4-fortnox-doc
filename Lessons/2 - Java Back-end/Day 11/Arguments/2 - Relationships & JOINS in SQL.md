# Relationships & JOINS in SQL

## Learning Objectives
- Define a **relationship** in a relational database.
- Distinguish **one-to-one (1 : 1)**, **one-to-many (1 : N)** and **many-to-many (N : M)** relationships.
- Create tables that model 1 : N and N : M relationships with **foreign keys**.
- Use **INNER JOIN**, **LEFT JOIN** and **JOIN chains** to retrieve related data.

## 1. What is a Relationship?
A *relationship* connects rows in one table with rows in another so the database can store data **once** yet reference it **many times**.

Types of relationships:

| Type | Meaning | Typical implementation |
| --- | --- | --- |
| 1 : 1 | Every row in A relates to **exactly one** row in B | Unique foreign key or the same shared primary key |
| 1 : N | One row in A relates to **many** rows in B | B gets a **foreign key** referencing A |
| N : M | Many rows in A relate to **many** rows in B | A **junction table** holds two foreign keys |

## 2. Example Schema

![[Relationships, joins and migrations - Example DB|1000]]

We will model a blogging platform:

1. **Authors** can write many **Posts** (1 : N).
2. **Posts** can have many **Tags**, and tags can belong to many posts (N : M).

```sql
-- Authors (1 side)
CREATE TABLE authors (
  id SERIAL PRIMARY KEY,
  name VARCHAR(50) NOT NULL
);
-- Posts (N side of 1:N)
CREATE TABLE posts (
  id SERIAL PRIMARY KEY,
  author_id SERIAL NOT NULL REFERENCES authors(id),
  title VARCHAR(100) NOT NULL,
  body TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);
-- Tags
CREATE TABLE tags (
  id SERIAL PRIMARY KEY,
  name VARCHAR(30) UNIQUE NOT NULL
);
-- Junction table post_tags (N:M)
CREATE TABLE post_tags (
  post_id SERIAL NOT NULL REFERENCES posts(id),
  tag_id SERIAL NOT NULL REFERENCES tags(id),
  PRIMARY KEY (post_id, tag_id)
);
```

## 3. Inserting Sample Data

```sql
-- Insert authors
INSERT INTO authors (name) VALUES ('Alice'), ('Bob');
-- Insert posts
INSERT INTO posts (author_id, title, body)
VALUES
  (1, 'Intro to SQL', '...'),
  (1, 'Advanced JOINS', '...'),
  (2, 'JSON vs Relational', '...');
-- Insert tags
INSERT INTO tags (name) VALUES ('sql'), ('joins'), ('database');
-- Insert post-tag relation
INSERT INTO post_tags (post_id, tag_id)
VALUES
  (1, 1), (2, 1), (2, 2), (3, 1), (3, 3);
```

## 4. Joining Tables

### 4.1 INNER JOIN (1 : N)
Get every post with its author’s name:

```sql
SELECT p.id, p.title, a.name AS author
FROM posts p
INNER JOIN authors a ON a.id = p.author_id;
```

### 4.2 Filtering with JOIN
Posts written by **Alice** only:

```sql
SELECT p.title
FROM posts p
INNER JOIN authors a ON a.id = p.author_id
WHERE a.name = 'Alice';
```

### 4.3 LEFT JOIN
List all authors, including those without posts yet:

```sql
SELECT a.name, p.title
FROM authors a
LEFT JOIN posts p ON p.author_id = a.id
ORDER BY a.name;
```

### 4.4 Chaining JOINS (N : M)
Return each post with every attached tag:

```sql
SELECT p.title, t.name AS tag
FROM posts p
JOIN post_tags pt ON pt.post_id = p.id
JOIN tags t ON t.id = pt.tag_id
ORDER BY p.title;
```

### 4.5 Aggregating After JOIN
Show how many posts use each tag:

```sql
SELECT t.name, COUNT(*) AS post_count
FROM tags t
JOIN post_tags pt ON pt.tag_id = t.id
GROUP BY t.name
ORDER BY post_count DESC;
```

## 5. Practice
1. Add a `categories` table and link posts to categories with a 1 : N relationship.
2. Write a JOIN that lists every post title, author name and category.
3. Modify a query to return authors **without** posts tagged `"joins"` (hint: `LEFT JOIN` plus `WHERE tag IS NULL`).

---

# Links
![[Lessons/2 - Java Back-end/Day 11/__blocks/Links]]
