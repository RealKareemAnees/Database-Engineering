# Database Engineering

this will be a very very long markdown

# ACID

so ACID is about making a database transactions reliable, ACID in general is a CS concept that databases implement

## Atomicity

for a transaction to succeed, It got to work in an atomic manner

so a transaction Is one _ATOM_ where If a part of it fails, the whole transaction has actually failed and should rollback

there is really not to much to talk about in atomicity

## Isolation

Isolation is about Isolating transactions from each others

before Isolation, thier is a problem that Isolatio n solves which is _Read Phenomina_

### Read Phenomena

#### Dirty Read

A dirty read occurs when one transaction reads updates made by another transaction before it has committed. If the second transaction rolls back or crashes, the first transaction will have read incorrect data.

![alt text](image.png)

#### Non-Repeatable Read

A non-repeatable read occurs when a transaction reads the same row twice and gets different values because another transaction has committed an update in between the reads.

![alt text](image-1.png)

#### Lost Update

A lost update happens when two transactions read the same data, modify it, and then write it back, overwriting each other's changes.

![alt text](image-2.png)

#### Phantom Read

A phantom read occurs when a transaction re-executes a query and finds additional rows that were inserted by another committed transaction.

![alt text](image-3.png)

---

### Implementing Read Phenomena

#### Dirty Read

##### **Session 1**

```sql
SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
START TRANSACTION;
UPDATE accounts SET balance = 5000 WHERE id = 1;
```

##### **Session 2**

```sql
SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
SELECT balance FROM accounts WHERE id = 1;
```

#### Non-Repeatable Read

##### **Session 1**

```sql
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
START TRANSACTION;
SELECT balance FROM accounts WHERE id = 1;
```

##### **Session 2**

```sql
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
START TRANSACTION;
UPDATE accounts SET balance = 5000 WHERE id = 1;
COMMIT;
```

##### **Back to Session 1**

```sql
SELECT balance FROM accounts WHERE id = 1;
```

#### Lost Updates

##### **Session 1**

```sql
START TRANSACTION;
SELECT balance FROM accounts WHERE id = 1;
UPDATE accounts SET balance = balance + 500 WHERE id = 1;
```

##### **Session 2**

```sql
START TRANSACTION;
SELECT balance FROM accounts WHERE id = 1;
UPDATE accounts SET balance = balance - 200 WHERE id = 1;
```

##### **Both Transactions Commit**

```sql
COMMIT;
```

#### Preventing Lost Updates

To prevent lost updates, we can use the `FOR UPDATE` statement to lock the selected rows.

```sql
SELECT balance FROM accounts WHERE id = 1 FOR UPDATE;
```

#### Phantom Read

##### **Session 1**

```sql
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
START TRANSACTION;
SELECT COUNT(*) FROM accounts WHERE balance > 1000;
```

##### **Session 2**

```sql
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
START TRANSACTION;
INSERT INTO accounts (id, balance) VALUES (10, 2000);
COMMIT;
```

##### **Back to Session 1**

```sql
SELECT COUNT(*) FROM accounts WHERE balance > 1000;
```

### Types of Isolation levels

- **READ UNCOMMITED** where there is no Isolation at all

- **READ COMMITED** each query can only see commited changes

- **REPEATABLE READ** when a query reads a row, that row will not be changed untill the transaction ends

- **SNAPSHOT** each transaction has a version of the data to work on

- **SERIALIZED** transactions will be excecuted in chain

![alt text](image-4.png)

### How Isolation is imlemented

- **Pessemistic** it locks the entity

- **Optimestic** it don't lock but track for changes

## Concistency

it is when data achieves refrential integrity and consistency in genereal

consistency in read and write

## durability

when commiting changes, theses changes got to be stored on disk, which might take a long time or may crash, so databases invents algorithms to achieve durable data operations

---

END OF ACID

# Database Internals

this chapter talks about 4 things

- each row has a unique ID, it maybe set by the DBMS or primary key like in MySQL

- each set of rows is stored in form of pages

- there are two types of databases
  - the ones thatr stores tables in form of rows
  - the othe stores data in form of colomns

## Colomns vs rows

this debate is very deep into the design of the tables but generally:

### **Row-Based DBMS (Relational Databases)**

- **Examples**: MySQL, PostgreSQL, SQL Server, Oracle
- **Best for**: Transactional (OLTP - Online Transaction Processing) workloads

#### **When to Use Row-Based DBMS:**

1. **Frequent Read and Write Transactions** – Useful for applications that require frequent inserts, updates, and deletes (e.g., banking, e-commerce, inventory management).
2. **Data Retrieval by Full Rows** – If queries often need to fetch complete records (e.g., retrieving all details about a customer or an order).
3. **ACID Compliance** – When strong consistency, atomicity, and reliability are required.
4. **Index-Based Querying** – Efficient when using indexes for looking up single or small sets of records.

---

### **Column-Based DBMS (Columnar Databases)**

- **Examples**: Apache Cassandra, Google BigQuery, ClickHouse, Amazon Redshift
- **Best for**: Analytical (OLAP - Online Analytical Processing) workloads

#### **When to Use Column-Based DBMS:**

1. **Analytical Queries on Large Datasets** – Ideal for data warehousing, reporting, and business intelligence (BI) applications.
2. **Aggregations on Specific Columns** – Optimized for queries that involve SUM, AVG, COUNT, GROUP BY, etc., across millions or billions of rows.
3. **Data Compression & Faster Reads** – Columnar storage allows better compression and efficient scans of selected columns instead of entire rows.
4. **Appending Data** – Works well for workloads where data is primarily appended and rarely updated (e.g., event logs, IoT data, historical analysis).

---

### **Key Trade-offs**

| Feature                   | Row-Based DBMS               | Column-Based DBMS              |
| ------------------------- | ---------------------------- | ------------------------------ |
| **Best for**              | OLTP (transactions)          | OLAP (analytics)               |
| **Querying**              | Fetching full records        | Aggregations on large datasets |
| **Write Performance**     | Fast inserts/updates/deletes | Optimized for batch inserts    |
| **Read Performance**      | Good for row-based lookups   | Fast for column-based queries  |
| **Storage & Compression** | Less efficient for analytics | High compression for columns   |
| **Indexing**              | Needed for performance       | Often not needed               |

### **Choosing the Right One**

- **For a transactional web app (e.g., e-commerce, CRM, banking system)** → Use **row-based DBMS**.
- **For analytics and reporting (e.g., big data, dashboards, machine learning logs)** → Use **column-based DBMS**.
- **For hybrid workloads** → Some databases (e.g., PostgreSQL with columnar extensions, MariaDB ColumnStore) support both.

# Indexing

Indexing is building a data structure with the rows to access the data faster.

## Types of Indexes

### 1. Primary Index

- Automatically created on the **primary key** of a table.
- Ensures rows are **sorted** in order.
- Only **one** primary index per table.

#### Example:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100) UNIQUE
);
```

### 2. Unique Index

- Ensures the column values are **unique**.
- Prevents duplicate data entries.

#### Example:

```sql
CREATE UNIQUE INDEX unique_email ON users(email);
```

### 3. Clustered Index

- Defines the **physical order** of rows in a table.
- Only **one** clustered index per table.

#### Example:

```sql
CREATE CLUSTERED INDEX idx_user_id ON users(id);
```

### 4. Non-Clustered Index

- Stores a separate structure that points to the actual data rows.
- Multiple non-clustered indexes can exist per table.

#### Example:

```sql
CREATE INDEX idx_user_name ON users(name);
```

### 5. Composite Index (Multi-Column Index)

- Indexes multiple columns for optimizing multi-column searches.

#### Example:

```sql
CREATE INDEX idx_user_name_email ON users(name, email);
```

### 6. Full-Text Index

- Used for **text search** on large textual data.

#### Example (MySQL):

```sql
CREATE FULLTEXT INDEX idx_user_bio ON users(bio);
```

### 7. Bitmap Index

- Efficient for **low-cardinality columns** (few unique values like `status`).
- Used in **data warehouses**.

### 8. Hash Index

- Optimized for **exact match lookups**.
- Not suitable for **range queries**.

#### Example (PostgreSQL):

```sql
CREATE INDEX idx_user_email_hash ON users USING HASH(email);
```

### 9. Partial Index

- Indexes **only a subset** of rows to save space and improve performance.

#### Example:

```sql
CREATE INDEX idx_active_users ON users(email) WHERE status = 'Active';
```

### 10. Covering Index

- Includes all columns needed for a query to avoid reading the table.

#### Example:

```sql
CREATE INDEX idx_user_email_name ON users(email, name);
```

## Indexing Algorithms

Indexing has several algorithms based on the type of the data and operations expected on these data.

### 1. B-Tree Index

- A self-balancing tree structure where nodes maintain sorted data.
- Efficient for **range queries** and **sorted data retrieval**.
- Used in most relational databases.

### 2. B+ Tree Index

- A variation of B-Trees where **only leaf nodes store actual data**.
- Internal nodes contain only keys for faster traversal.
- Supports **sequential scans** efficiently.

### 3. Hash Index

- Uses **hashing functions** to map keys to memory locations.
- Best for **exact match lookups**, but **not efficient** for range queries.

### 4. Bitmap Index

- Uses **bitmaps** (arrays of 0s and 1s) to represent data efficiently.
- Best suited for **low-cardinality columns** (few unique values, e.g., gender, status).
- Commonly used in **data warehouses**.

### 5. Trie (Prefix Tree)

- A tree data structure where each node represents a character in a key.
- Best for **prefix searches** (e.g., autocomplete, dictionary lookups).
- Used in **full-text search engines**.

### 6. Skip List Index

- A linked list-based index with multiple levels for fast traversal.
- Provides **O(log N) search time**, similar to B-Trees.
- Used in databases requiring **fast inserts and reads**.

### 7. R-Tree Index (Spatial Indexing)

- Used for **geospatial indexing** (e.g., maps, geographic locations).
- Efficient for **bounding box searches** and **2D/3D spatial data**.
- Commonly found in **PostGIS, Oracle Spatial**.

### Choosing the Right Indexing Algorithm

| Algorithm            | Best for                                | Drawbacks                               |
| -------------------- | --------------------------------------- | --------------------------------------- |
| **B-Tree / B+ Tree** | General-purpose indexing, range queries | Slower for large writes                 |
| **Hash Index**       | Exact match queries (`WHERE email = ?`) | No range queries (`<, >, BETWEEN`)      |
| **Bitmap Index**     | Low-cardinality columns (e.g., status)  | Large storage for high-cardinality data |
| **Trie**             | Prefix searches (e.g., autocomplete)    | High memory usage                       |
| **Skip List**        | Fast inserts and reads                  | More memory overhead                    |
| **R-Tree**           | Geospatial queries                      | Complex to maintain                     |

# Partitioning

partitioning is spliting a table into multiple tables, thats it

![alt text](image-5.png)

## Horizontal vs vertical partition

I think this this table is the best ever demonstration of the topic, thank you chat-gpt

| Feature                | **Horizontal Partitioning**                                                                                       | **Vertical Partitioning**                                                                                            |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| **Concept**            | Divides a table into multiple rows based on a condition (e.g., range, hash, or list).                             | Splits a table into multiple columns across different tables.                                                        |
| **Structure**          | Each partition has the same schema but different subsets of data.                                                 | Each partition has different columns but shares a common primary key.                                                |
| **Use Case**           | Useful when dealing with a large number of rows that need to be distributed across multiple servers or databases. | Useful when a table has too many columns, and some are rarely used together.                                         |
| **Performance Impact** | Improves query performance by reducing the number of rows scanned.                                                | Improves query performance by reducing the number of columns read.                                                   |
| **Scalability**        | Good for scaling out (sharding across multiple servers).                                                          | Helps optimize storage and memory usage.                                                                             |
| **Example**            | A customer table is split by country, with each partition containing customers from a specific country.           | A user profile table is split into `user_basic_info` (ID, Name, Email) and `user_preferences` (ID, Theme, Language). |
| **Complexity**         | Requires partitioning logic and proper indexing to optimize queries.                                              | Requires joining tables in queries, which can increase complexity.                                                   |

### **When to Use Each?**

- **Use Horizontal Partitioning** if you have a massive dataset and need to distribute the load across multiple servers (e.g., sharding in NoSQL databases).
- **Use Vertical Partitioning** if a table has too many columns and queries only need a subset of them, reducing read latency and memory consumption.

## Types of partitioning

either by range or by list or by hash

here is yet another aawesome chat-gpt table comparing between types of partitioning types

| Feature               | **Range Partitioning**                                      | **List Partitioning**                                                        | **Hash Partitioning**                                    |
| --------------------- | ----------------------------------------------------------- | ---------------------------------------------------------------------------- | -------------------------------------------------------- |
| **Concept**           | Divides data based on a range of values.                    | Divides data based on predefined discrete values.                            | Distributes data evenly using a hash function.           |
| **Use Case**          | When data has a natural order (e.g., dates, numerical IDs). | When data fits into specific, known categories (e.g., regions, departments). | When uniform distribution is needed to avoid hotspots.   |
| **Partitioning Key**  | A column with sequential values (e.g., `date`, `ID`).       | A column with a fixed set of values (e.g., `region`, `category`).            | A column that can be hashed (e.g., `user_id`).           |
| **Query Performance** | Efficient for range-based queries.                          | Efficient for queries targeting specific categories.                         | Helps distribute load but requires hash computation.     |
| **Scalability**       | Easy to add new partitions as new ranges emerge.            | Requires updating partitions if new categories are added.                    | Good for large-scale distributed databases.              |
| **Example Use Case**  | Sales data partitioned by month.                            | Customer data partitioned by country.                                        | User data evenly distributed across multiple partitions. |

---

### **SQL Examples**

#### **1. Range Partitioning**

Used when data can be divided into sequential ranges (e.g., dates, IDs).

```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    order_date DATE NOT NULL
) PARTITION BY RANGE (order_date);

CREATE TABLE orders_2023 PARTITION OF orders
    FOR VALUES FROM ('2023-01-01') TO ('2023-12-31');

CREATE TABLE orders_2024 PARTITION OF orders
    FOR VALUES FROM ('2024-01-01') TO ('2024-12-31');
```

- New orders are automatically placed in the correct partition.
- Optimized for date-based queries (e.g., **`WHERE order_date BETWEEN '2023-01-01' AND '2023-12-31'`**).

---

#### **2. List Partitioning**

Used when data falls into distinct categories.

```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    country TEXT NOT NULL
) PARTITION BY LIST (country);

CREATE TABLE customers_egypt PARTITION OF customers
    FOR VALUES IN ('Egypt');

CREATE TABLE customers_usa PARTITION OF customers
    FOR VALUES IN ('USA');

CREATE TABLE customers_other PARTITION OF customers
    DEFAULT;
```

- Queries like **`WHERE country = 'Egypt'`** are optimized.
- If a country doesn’t match any partition, it goes to the **default** partition.

---

#### **3. Hash Partitioning**

Used to distribute data evenly across multiple partitions.

```sql
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    username TEXT NOT NULL
) PARTITION BY HASH (user_id);

CREATE TABLE users_p0 PARTITION OF users
    FOR VALUES WITH (MODULUS 4, REMAINDER 0);

CREATE TABLE users_p1 PARTITION OF users
    FOR VALUES WITH (MODULUS 4, REMAINDER 1);

CREATE TABLE users_p2 PARTITION OF users
    FOR VALUES WITH (MODULUS 4, REMAINDER 2);

CREATE TABLE users_p3 PARTITION OF users
    FOR VALUES WITH (MODULUS 4, REMAINDER 3);
```

- A **hash function** ensures even data distribution.
- Works well for **high-traffic applications** like user authentication systems.

---

### **When to Use Each?**

- **Use Range Partitioning** if your queries filter by ordered values (e.g., dates).
- **Use List Partitioning** if your data naturally falls into predefined groups.
- **Use Hash Partitioning** if you need even load distribution across multiple partitions.

## Attaching partitions

### **What Does "Attach Partition" Mean?**

- Imagine you already have **separate tables** storing data.
- Later, you decide that instead of keeping them separate, you want to **group them together as partitions** in a new table.
- Instead of moving data manually, you can **attach** these tables as partitions.

---

### **Example: Attaching Existing Tables as Partitions**

#### **Scenario**

You have two tables, `orders_2023` and `orders_2024`, storing order data for different years. Initially, they were just **normal tables**.

Now, you decide to **create a main `orders` table** that will organize these by **year-based partitions**.

#### **Step 1: Create the Partitioned Table**

```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    order_date DATE NOT NULL
) PARTITION BY RANGE (order_date);
```

✅ This is an **empty partitioned table**, ready to accept partitions.

#### **Step 2: Create Separate Tables (if not already existing)**

```sql
CREATE TABLE orders_2023 (
    order_id INT PRIMARY KEY,
    order_date DATE NOT NULL
);

CREATE TABLE orders_2024 (
    order_id INT PRIMARY KEY,
    order_date DATE NOT NULL;
);
```

✅ These tables already **exist** and contain data.

#### **Step 3: Attach Existing Tables as Partitions**

```sql
ALTER TABLE orders ATTACH PARTITION orders_2023
FOR VALUES FROM ('2023-01-01') TO ('2023-12-31');

ALTER TABLE orders ATTACH PARTITION orders_2024
FOR VALUES FROM ('2024-01-01') TO ('2024-12-31');
```

✅ Now, `orders_2023` and `orders_2024` **become partitions of `orders`**, without copying or modifying their data.

---

### **🔹 Why is This Useful?**

1. **No Data Movement** → The data stays in place; we just tell the database to treat these tables as partitions.
2. **Fast Query Performance** → Queries on `orders` now automatically route to the correct partition.
3. **Easy Migration** → If you had old, separate tables, you can integrate them into a partitioned system without rebuilding everything.

---

### **What Happens After Attaching a Partition?**

- If you run:

```sql
SELECT * FROM orders WHERE order_date = '2023-05-01';
```

✅ It will automatically **fetch data only from `orders_2023`**, improving performance.

- If you insert:

```sql
INSERT INTO orders (order_id, order_date) VALUES (101, '2024-03-15');
```

✅ The row **goes directly into `orders_2024`** without needing manual placement.

---

## Indexing with Partitioning

### Key Differences in Indexing Partitioned Tables

1. **Indexes are not inherited**
   - If you create an index on the **parent partitioned table**, it won’t automatically apply to child partitions.
   - You need to **create indexes on each partition separately**.
2. **Global vs. Local Indexes**
   - **Global Indexes**: One index covering all partitions (not supported in PostgreSQL).
   - **Local Indexes**: Separate indexes on each partition (used in PostgreSQL).

---

### Example: Creating Indexes in a Partitioned Table

#### Step 1: Create a Partitioned Table

```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    order_date DATE NOT NULL,
    customer_id INT NOT NULL
) PARTITION BY RANGE (order_date);
```

This is the **main partitioned table**.

---

#### Step 2: Create Partitions

```sql
CREATE TABLE orders_2023 PARTITION OF orders
    FOR VALUES FROM ('2023-01-01') TO ('2023-12-31');

CREATE TABLE orders_2024 PARTITION OF orders
    FOR VALUES FROM ('2024-01-01') TO ('2024-12-31');
```

Now we have **two partitions**.

---

#### Step 3: Create Indexes on Each Partition

```sql
CREATE INDEX orders_2023_idx ON orders_2023 (customer_id);
CREATE INDEX orders_2024_idx ON orders_2024 (customer_id);
```

Each partition now has **its own index**.

---

### Query Execution with Indexed Partitioning

If you run:

```sql
SELECT * FROM orders WHERE order_date = '2023-06-01' AND customer_id = 5;
```

#### How It Works:

1. **Partition Pruning**: The query planner **only scans `orders_2023`**.
2. **Index Usage**: The index on `orders_2023 (customer_id)` speeds up the query.

---

### Alternative: Creating an Index on the Parent Table

You can **define an index on the parent** that automatically applies to new partitions:

```sql
CREATE INDEX ON ONLY orders (customer_id);
```

#### Important Notes:

- This does **not** index existing partitions.
- It acts as a **template** for new partitions.
- If you **want indexes on all existing partitions**, you must create them **manually**.

---

### Summary

- **Partitioned tables don’t share indexes with child partitions.**
- **Indexes must be created separately on each partition.**
- **Global indexes (across all partitions) are not supported in PostgreSQL.**
- **Use "ON ONLY" to create an index template for new partitions.**

# Sharding

sharding is distriputing data over multiple databases over multiple locations

let us say that odd IDs are stored on a server and even IDs are stored on another server

then an application level sharding would be like this

```js
const mysql = require("mysql2");

// Define connections for shards
const shard1 = mysql.createConnection({
  host: "shard1_host",
  user: "root",
  database: "db1",
});
const shard2 = mysql.createConnection({
  host: "shard2_host",
  user: "root",
  database: "db2",
});

function getShard(userId) {
  return userId % 2 === 0 ? shard1 : shard2; // just picks the desired server's pointer
}

function insertUser(userId, name) {
  const shard = getShard(userId);
  shard.query(
    "INSERT INTO users (id, name) VALUES (?, ?)",
    [userId, name],
    (err) => {
      if (err) console.error(err);
    }
  );
}
```

# Concurrency control

Pessimistic and optimistic concurrency control are two different approaches to managing concurrent access to a database in **Database Management Systems (DBMSs)**.

## **1. Pessimistic Concurrency Control**

- **Concept**: Assumes conflicts **will happen**, so it prevents them by **locking** resources before making changes.
- **Mechanism**: Uses **locks (exclusive locks, shared locks)** to ensure that only one transaction can modify a resource at a time.
- **Common Usage**: When there is a high likelihood of conflicts, such as in high-write environments like financial systems.

### **How to Achieve Pessimistic Concurrency Control**

- **Row-Level Locking**: `SELECT ... FOR UPDATE` (Locks the row until the transaction is committed or rolled back).
  ```sql
  BEGIN;
  SELECT * FROM accounts WHERE id = 1 FOR UPDATE;
  UPDATE accounts SET balance = balance - 100 WHERE id = 1;
  COMMIT;
  ```
- **Table-Level Locking**: `LOCK TABLE table_name IN EXCLUSIVE MODE;`
- **Explicit Locking in Application Code**: Using `SELECT ... LOCK IN SHARE MODE` for read consistency.

---

## **2. Optimistic Concurrency Control**

- **Concept**: Assumes conflicts **are rare**, so it allows multiple transactions to proceed without locking but **checks for conflicts before committing**.
- **Mechanism**: Uses **versioning (timestamps, row versions) or "Compare and Swap" techniques**.
- **Common Usage**: When there are more reads than writes, such as in web applications and analytics systems.

### **How to Achieve Optimistic Concurrency Control**

- **Using Version Numbers**

  ```sql
  -- Read data along with its version
  SELECT id, balance, version FROM accounts WHERE id = 1;

  -- Attempt update only if version has not changed
  UPDATE accounts
  SET balance = balance - 100, version = version + 1
  WHERE id = 1 AND version = 5;
  ```

  If the update affects **zero rows**, another transaction modified the row, and you must retry.

- **Using Timestamps**

  ```sql
  -- Read row with timestamp
  SELECT id, balance, last_updated FROM accounts WHERE id = 1;

  -- Update only if timestamp is unchanged
  UPDATE accounts
  SET balance = balance - 100, last_updated = CURRENT_TIMESTAMP
  WHERE id = 1 AND last_updated = '2025-03-26 12:34:56';
  ```

  If no rows were updated, retry the transaction.

---

## **Comparison Table**

| Feature                | Pessimistic Concurrency Control                  | Optimistic Concurrency Control          |
| ---------------------- | ------------------------------------------------ | --------------------------------------- |
| **Assumption**         | Conflicts **will** happen                        | Conflicts **are rare**                  |
| **How it works**       | Locks resources before updates                   | Detects conflicts at commit             |
| **Performance Impact** | Slower due to locking                            | Faster due to no locking                |
| **Best for**           | High-write systems (banking, transactions)       | High-read systems (web apps, analytics) |
| **Drawbacks**          | Can cause **deadlocks**, slows down transactions | Requires retry logic, can fail updates  |

If your system is **write-heavy and needs strict consistency**, go with **pessimistic control**. If your system is **read-heavy with fewer conflicts**, go with **optimistic control**.

# Replication

unlike sharding; replication is making multiple copies of the same database

## types of replication:

### Master/backup replication

one master node that writes data; another many nodes that reads theses data

![alt text](image-6.png)

![alt text](image-7.png)

this is best for many reads and few writes

### Multi-master replication

many node can write

this is complex as it needs conflict management

## Synchronous vs Asynchronous writes

- **Synchronous** Each write to the master will wait for existing writes to finish and is not considered successful until the master finishes writing to the replicas

- **ASynchronous** write is considered successfull once the change is submitted to the master and in the other hand changes are then applied asynchronously

## Replication with MySQL

Replication is quite hard to implement and it's syntax differes from DBMS to the other

# Database Engines

simply, a database engine has the idea of a library that can be injected into the DBMS to do the CRUD operations implementations

## MyISAM

it is a storage engine for MySQL with the following properties:

1. **Table-Level Locking** – Locks the entire table during insert, update, or delete operations, making it less suitable for high-concurrency applications.
2. **No Foreign Key Constraints** – MyISAM does not support foreign keys, making it less suitable for maintaining strict relational integrity.
3. **No Transactions** – It lacks support for ACID transactions (Atomicity, Consistency, Isolation, Durability), making it less reliable for critical applications.
4. **Fast Reads, Slow Writes** – Optimized for read-heavy operations but slower for writes due to table-level locking.
5. **Full-Text Search** – Supports full-text indexing, making it useful for search-heavy applications.
6. **Crash Recovery Issues** – Susceptible to corruption and lacks built-in crash recovery mechanisms like InnoDB.
7. **Compact Storage** – MyISAM tables generally take up less disk space compared to InnoDB.
8. **Manual Repair Needed** – If a table gets corrupted, `REPAIR TABLE` must be used to fix it manually.
9. **No Row-Level Locking** – Entire table locks make concurrent updates inefficient.
10. **Uses `.MYD`, `.MYI`, and `.frm` Files** – Stores data in `.MYD` files, indexes in `.MYI`, and table definitions in `.frm`.

Due to its limitations, MyISAM has been largely replaced by **InnoDB**, which offers better concurrency, transactions, and crash recovery.

## **InnoDB**

it is the default storage engine for MySQL, offering strong reliability, performance, and data integrity. Here are its key properties:

1. **Row-Level Locking** – Supports concurrent writes by locking only the affected rows instead of the entire table.
2. **ACID Compliance** – Supports transactions with **Atomicity, Consistency, Isolation, and Durability (ACID)**, ensuring data integrity.
3. **Foreign Key Support** – Enforces **referential integrity** through foreign key constraints.
4. **Crash Recovery** – Uses a **redo log** and automatic recovery mechanism to restore data after a crash.
5. **MVCC (Multi-Version Concurrency Control)** – Allows non-blocking reads, improving performance in concurrent environments.
6. **Supports Transactions** – Implements **commit, rollback, and savepoint** features.
7. **Uses a Clustered Index** – Data is stored in a **B+Tree structure**, optimizing primary key lookups.
8. **Better Write Performance** – Optimized for mixed read/write operations due to row-level locking.
9. **Larger Storage Overhead** – Requires more disk space than MyISAM due to transactional logs and indexing.
10. **Uses `.ibd` and `.frm` Files** – Stores data and indexes in a single `.ibd` file (if **file-per-table** is enabled) and table definitions in `.frm`.

### **When to Use InnoDB?**

- When **transactions** are required.
- When ensuring **data integrity** via foreign keys.
- For **high-concurrency applications** with frequent writes.

### **Comparison with MyISAM**

| Feature           | InnoDB            | MyISAM           |
| ----------------- | ----------------- | ---------------- |
| Locking Mechanism | Row-level         | Table-level      |
| Transactions      | ✅ Yes            | ❌ No            |
| Foreign Keys      | ✅ Yes            | ❌ No            |
| Performance       | Better for writes | Better for reads |
| Crash Recovery    | ✅ Yes            | ❌ No            |
| Storage           | Larger footprint  | Compact          |

InnoDB is the **recommended** choice for most modern MySQL applications due to its reliability and performance advantages.

## SQLite

it is a lightweight, self-contained database engine designed for embedded systems, mobile apps, and small-scale applications. Here are its key properties:

### **1. Serverless & Self-Contained**

- Unlike MySQL or PostgreSQL, **SQLite is serverless**—it doesn’t run as a separate process but as a library linked within an application.
- The entire database is stored in a **single file** on disk.

### **2. Zero Configuration**

- No need for setup, user authentication, or configuration files.
- Easy to deploy; just include the SQLite library.

### **3. ACID Compliance**

- Supports **Atomicity, Consistency, Isolation, and Durability (ACID)** via rollback journals or Write-Ahead Logging (WAL).

### **4. Lightweight & Fast**

- Designed for **small to medium-sized** applications with low resource consumption.
- Read operations are fast, but write performance is limited by single-writer concurrency.

### **5. Single-File Storage**

- The entire database (tables, indexes, and data) is stored in a single `.sqlite` file.
- This makes it easy to copy, backup, and distribute.

### **6. No User Management**

- Unlike MySQL or PostgreSQL, **SQLite has no built-in user authentication** or role-based access control (RBAC).
- Security must be handled at the application level.

### **7. Limited Concurrency**

- **Single-writer, multiple-reader model** – multiple processes can read, but only one can write at a time.
- Suitable for **low-concurrency applications**, but not ideal for high-write workloads.

### **8. No Native Network Access**

- SQLite doesn’t have a built-in client-server model.
- To access SQLite remotely, you need an external wrapper like **LiteFS** or use it with a web API.

### **9. No Strict Type Enforcements**

- SQLite uses **dynamic typing**, meaning a column can store any type of data regardless of its declared type.
- This provides flexibility but can lead to **data inconsistency** if not managed properly.

### **10. Built-in Full-Text Search (FTS)**

- Supports **full-text search (FTS5)**, making it great for local search applications.

---

### **When to Use SQLite?**

✅ **Mobile Apps** – Used in **Android & iOS** (via Room, Core Data, etc.).  
✅ **Embedded Systems** – Ideal for IoT devices, sensors, and offline applications.  
✅ **Small Websites & Local Storage** – Suitable for blogs, caching, and config storage.  
✅ **Prototyping & Testing** – Quick setup for testing database-driven applications.

### **Comparison with MySQL & InnoDB**

| Feature        | SQLite                     | MySQL (InnoDB)                  |
| -------------- | -------------------------- | ------------------------------- |
| Architecture   | Serverless                 | Client-Server                   |
| Concurrency    | Limited                    | High                            |
| Transactions   | ✅ Yes                     | ✅ Yes                          |
| Foreign Keys   | ✅ Yes (since 3.6.19)      | ✅ Yes                          |
| Performance    | Fast reads, limited writes | Better for high concurrency     |
| Storage        | Single-file `.sqlite`      | Multiple files (`.ibd`, `.frm`) |
| Scalability    | Limited                    | Highly scalable                 |
| Network Access | ❌ No                      | ✅ Yes                          |

**Bottom Line:**  
SQLite is **simple, lightweight, and great for embedded or local applications** but lacks the scalability and concurrency of MySQL/InnoDB.

## LevelDB

it is a fast key-value storage engine developed by **Google**. It is optimized for **fast reads and writes** using an **LSM (Log-Structured Merge) tree** architecture. Unlike traditional relational databases, **LevelDB is not SQL-based** and does not support tables, indexes, or transactions.

---

### **Key Properties of LevelDB**

1. **Key-Value Storage**

   - Stores data as **key-value pairs**, similar to **Redis** or **RocksDB**.
   - Keys and values are arbitrary **byte arrays**, meaning you can store anything (numbers, strings, binary data).

2. **LSM-Tree Based**

   - Uses **Log-Structured Merge (LSM) trees** instead of B-Trees (used in MySQL/InnoDB).
   - Optimized for **high write throughput** and minimizes disk writes.

3. **Ordered Storage**

   - Keys are stored **in sorted order**, making **range queries efficient**.
   - Supports **prefix scans** and **iterators** to traverse data efficiently.

4. **High Write Performance**

   - Writes are **batched** and stored in an **append-only log (memtable)** before being merged into disk (SSTable).
   - Ideal for applications with **high write workloads**.

5. **No SQL, No Schema**

   - Unlike MySQL or SQLite, **LevelDB is not a relational database**.
   - No tables, no columns—just raw key-value storage.

6. **Atomic Writes & Snapshots**

   - Supports **atomic batch writes** but not full ACID transactions.
   - Snapshots allow **point-in-time consistent reads**.

7. **Compression Support**

   - Uses **Snappy compression** to reduce storage space and improve performance.

8. **Limited Concurrency**

   - **Single-process access** (only one process can access a LevelDB database at a time).
   - Supports multiple **concurrent reads** but only **one writer** at a time.

9. **No Native Replication or Sharding**

   - Unlike MySQL, LevelDB does not have **built-in replication** or **sharding**.
   - Can be scaled using external tools like **RocksDB** (a more advanced fork).

10. **File-Based Storage**

- Stores data in multiple **SSTable** (Sorted String Table) files on disk.
- Uses a **WAL (Write-Ahead Log)** for crash recovery.

---

### **When to Use LevelDB?**

✅ **Embedded Databases** – Great for local storage in browsers, mobile apps, and desktop applications.  
✅ **Caching** – Used in **Chrome, Firefox, and Android** for caching.  
✅ **Blockchain & Distributed Systems** – Used in **Bitcoin Core** and **Ethereum** for fast key-value lookups.  
✅ **High Write Applications** – Ideal for logging systems and time-series data.  
✅ **Key-Value Stores** – Good for applications that don’t need relational data.

---

### **Comparison with Other Databases**

| Feature      | LevelDB                     | SQLite                    | MySQL (InnoDB)           |
| ------------ | --------------------------- | ------------------------- | ------------------------ |
| Data Model   | Key-Value Store             | Relational (SQL)          | Relational (SQL)         |
| Architecture | Embedded                    | Embedded                  | Client-Server            |
| Concurrency  | Single-writer, multi-reader | Limited write concurrency | High concurrency         |
| Transactions | Atomic batch writes         | ACID transactions         | ACID transactions        |
| Performance  | High writes, fast reads     | Moderate                  | Optimized for high loads |
| Storage      | LSM-Tree (SSTables)         | Single-file `.sqlite`     | B+Tree (Clustered Index) |
| Scalability  | Limited (single process)    | Limited                   | High scalability         |

---

### **Bottom Line:**

**LevelDB is a fast, embedded key-value store optimized for high write performance** but lacks SQL features, multi-process concurrency, and built-in replication. If you need a lightweight, fast database for **caching, blockchain, or embedded storage**, LevelDB is a great choice!

## RocksDB

it is an advanced key-value storage engine developed by **Facebook**, built on top of **LevelDB** but optimized for better performance, scalability, and efficiency. It is designed for high-throughput, low-latency applications and is widely used in databases, distributed systems, and real-time analytics.

---

### **Key Properties of RocksDB**

1. **Key-Value Storage**

   - Stores **key-value pairs** where both keys and values are **byte arrays**.
   - Supports **arbitrary data types** (strings, numbers, JSON, binary blobs).

2. **LSM-Tree (Log-Structured Merge Tree) Architecture**

   - Optimized for write-heavy workloads using an **append-only** log-based structure.
   - Reduces disk I/O by **batching writes** and flushing them in compacted **Sorted String Table (SSTable) files**.

3. **High Write & Read Performance**

   - Uses **multi-threaded compaction** for efficient data merging.
   - Supports **block-based caching** to speed up reads.
   - Optimized for **SSD storage** and large datasets.

4. **Better Concurrency Than LevelDB**

   - Supports **multiple concurrent writers**, unlike LevelDB (which allows only one).
   - Uses **fine-grained locking** to improve parallelism.

5. **Column Families**

   - Supports multiple **column families**, similar to **tables in relational databases**.
   - Allows **isolation of different data types** within a single database.

6. **Transactions & Atomic Writes**

   - Supports **ACID-compliant transactions** (unlike LevelDB, which only supports atomic batch writes).
   - Provides **snapshot isolation** for consistent reads.

7. **Compression & Space Efficiency**

   - Uses **Snappy, ZSTD, or LZ4 compression** to reduce storage costs.
   - Supports **prefix compression** to speed up range queries.

8. **Efficient Range Queries & Iteration**

   - Data is stored in **sorted order**, allowing **fast range queries**.
   - Optimized iterators for **prefix and range scans**.

9. **Replication & Sharding Support**

   - Used in distributed databases like **TiDB, CockroachDB, YugabyteDB**.
   - Supports **log shipping** and **custom replication strategies**.

10. **Persistent & Embedded Storage**

- Can be used as an **embedded database** (similar to SQLite).
- Also serves as **persistent storage for distributed databases**.

---

### **When to Use RocksDB?**

- **High-Performance Databases** – Used in NoSQL databases (e.g., **TiDB, Apache Cassandra, CockroachDB**).
- **Streaming & Logging** – Great for real-time logging, analytics, and event processing.
- **Blockchain & Cryptocurrency** – Used in **Bitcoin, Ethereum, Hyperledger** for storing blockchain data.
- **Embedded Databases** – Used in **Kafka Streams, Flink, and Elasticsearch** for key-value storage.
- **Cache & SSD-Based Storage** – Optimized for fast access in **SSD-backed** applications.

---

### **Comparison: RocksDB vs LevelDB vs SQLite vs MySQL**

| Feature           | RocksDB                       | LevelDB         | SQLite           | MySQL (InnoDB)   |
| ----------------- | ----------------------------- | --------------- | ---------------- | ---------------- |
| Data Model        | Key-Value Store               | Key-Value Store | Relational (SQL) | Relational (SQL) |
| Architecture      | Embedded                      | Embedded        | Embedded         | Client-Server    |
| Write Performance | High                          | Moderate        | Low-Medium       | Medium           |
| Read Performance  | High                          | High            | Medium           | High             |
| Transactions      | Yes (ACID)                    | No              | Yes              | Yes              |
| Concurrency       | Multi-threaded                | Single-writer   | Limited          | High             |
| Compression       | Yes (Snappy, ZSTD)            | Yes (Snappy)    | No               | Yes              |
| Scalability       | High (Replication & Sharding) | Low             | Low              | High             |

---

### **Bottom Line**

RocksDB is a high-performance, embeddable key-value store optimized for fast writes, high concurrency, and efficient storage.  
It is an improved version of LevelDB, designed for use in distributed databases, real-time analytics, and big data applications.

## DBMS by indexing DS

just this image

![alt text](image-8.png)

# Security

- use SSL communication encryption

- each table/partition has many users where each user can only do one very scoped task

# homophoric encryption

it is still relatively new and has huge performance trade-offs, it is about encryption algorithm that can do operations without the need for decryption as long as the arguments are encrypted using the same algorithms and keys

---

---

---

---

---

---

---

---

---

---

# THE END

that's it, thanks for reaching so far
