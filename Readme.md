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
