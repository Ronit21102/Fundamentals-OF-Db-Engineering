# Fundamentals-OF-Db-Engineering

### Transaction?
In database engineering, a **transaction** is a sequence of operations performed as a single logical unit of work, ensuring **data integrity**.
or Collection of **queries** that is treated as single unit of work
### Key Features (ACID properties):
1. **Atomicity**: All operations in a transaction succeed or fail together.  
2. **Consistency**: The database remains in a valid state before and after the transaction.  
3. **Isolation**: Transactions are executed independently to avoid interference.  
4. **Durability**: Once committed, changes are permanently saved even in case of failure.

---

### Example:
In a **bank transfer**:
- Deduct $100 from **Account A**.  
- Add $100 to **Account B**.  

If one operation fails, the entire transaction is rolled back to maintain consistency.
![image](https://github.com/user-attachments/assets/d1ebe309-3ef0-4baa-a01e-395c6b872fd1)

###ATOMICITY
![image](https://github.com/user-attachments/assets/606af4bf-9058-492f-84ea-4b9d08f0122c)

###ISOLATION
In the context of a database, "wrote" and "commit" refer to actions taken during a transaction. Here's what they mean:

### **1. Wrote (or Write):**
- **Definition:** When a transaction "wrote" something, it means it has performed changes or updates to the database. This could involve inserting, updating, or deleting data.
- **Status:** At this point, the changes are typically held in a temporary state (e.g., in memory or a transaction log) and are **not yet permanent**.
- **Visibility:** These changes may only be visible to the transaction that made them, depending on the database's isolation level. Other transactions may not see these changes yet.

### **2. Commit:**
- **Definition:** A commit is the action that **makes the changes permanent** in the database. Once a transaction is committed:
  - The changes are saved to the database's persistent storage.
  - They become visible to other transactions.
- **Point of No Return:** After a commit, the changes cannot be undone unless a new transaction explicitly reverts them (e.g., another update or delete).

### **In Summary:**
- **"Wrote"** means the transaction has made changes but hasn't finalized them yet.
- **"Commit"** ensures that the changes are permanently applied to the database and visible to all.

If the transaction doesn’t commit (e.g., it’s rolled back or the database crashes), any writes it made are discarded, ensuring the database remains consistent.

Issue in Isolation
![image](https://github.com/user-attachments/assets/4eeee1d2-8407-474f-8d4f-2a6e7a35dde2)

Yes, in PostgreSQL, every **update** to a row creates a **new version** of that row. This behavior is due to PostgreSQL's implementation of **MVCC (Multiversion Concurrency Control)**, which ensures that transactions are isolated and consistent.

---

### **How It Works:**
1. **New Row Version:**
   - When a row is updated, PostgreSQL does not overwrite the original row. Instead, it creates a new version of the row with the updated data.
   - The original row is marked as **obsolete** (or "dead") but is retained until it is no longer needed by any active transaction.

2. **Visibility Rules:**
   - Different transactions might see different versions of the same row, depending on the isolation level and when the transaction started.
   - This ensures that transactions have a consistent view of the data.

3. **Cleanup (Vacuum Process):**
   - PostgreSQL eventually removes obsolete row versions using the **VACUUM** process, reclaiming storage space for dead tuples (old row versions) that are no longer visible to any transaction.

---

### **Example:**
1. Start with a table:
   ```sql
   CREATE TABLE accounts (
       id SERIAL PRIMARY KEY,
       balance INT
   );

   INSERT INTO accounts (balance) VALUES (100);
   ```

2. Update the balance:
   ```sql
   UPDATE accounts SET balance = 150 WHERE id = 1;
   ```

   - PostgreSQL creates a new version of the row:
     - **Old version:** `id = 1, balance = 100` (marked obsolete).
     - **New version:** `id = 1, balance = 150`.

3. Transactions and visibility:
   - If a transaction started before the `UPDATE`, it will still see the old version (`balance = 100`).
   - New transactions will see the new version (`balance = 150`).

4. After cleanup (VACUUM):
   - Once no transaction needs the old version, PostgreSQL's **VACUUM** process removes it, reclaiming disk space.

---

### **Advantages of MVCC:**
1. **Concurrency:** Multiple transactions can read and write without blocking each other.
2. **Consistency:** Each transaction gets a consistent snapshot of the data.
3. **Rollback Support:** Older versions of rows are retained, enabling rollbacks.

---

### **Downsides to Consider:**
1. **Storage Usage:** Multiple row versions increase storage requirements.
2. **Maintenance Overhead:** Regular **VACUUM** or **autovacuum** processes are necessary to reclaim space.

---

In summary, yes, PostgreSQL creates a new version of a row for every update, leveraging MVCC to provide robust transaction isolation and consistency.

![image](https://github.com/user-attachments/assets/de7c24b9-e540-43f4-88b2-85ddae6191f6)
![image](https://github.com/user-attachments/assets/8914bd38-8f7c-4e75-a2fc-f07e63578c3e)
![image](https://github.com/user-attachments/assets/865a9583-f383-4e78-9eb6-79af6ce501b8)
![image](https://github.com/user-attachments/assets/4674a8ad-b5ad-4315-9cbb-6f96018f6542)

**Isolation levels** are a key concept in database systems, defining how transaction concurrency is managed to ensure data consistency. They dictate how and when changes made by one transaction become visible to others and are a critical component of **ACID (Atomicity, Consistency, Isolation, Durability)** properties. 

---

### **Key Problems Addressed by Isolation Levels**
1. **Dirty Reads:** Reading uncommitted changes made by another transaction.
2. **Non-Repeatable Reads:** Re-reading data and finding it changed due to another committed transaction.
3. **Phantom Reads:** A query returns different rows on subsequent executions because another transaction inserted or deleted rows.

---

### **Isolation Levels (ANSI SQL Standards)**

#### 1. **Read Uncommitted**
- **Behavior:** Transactions can read uncommitted changes made by other transactions.
- **Issues:** 
  - Dirty reads are possible.
  - Non-repeatable reads and phantom reads can occur.
- **Use Case:** Rarely used due to its lack of safety. Suitable for scenarios where performance matters more than accuracy (e.g., non-critical logging systems).

**Example:**
- **Transaction A:** Updates a row but hasn’t committed.
- **Transaction B:** Reads the uncommitted value, which might later be rolled back.

---

#### 2. **Read Committed**
- **Behavior:** Transactions can only read committed changes made by other transactions. Each read sees the most recent committed value.
- **Issues:**
  - Non-repeatable reads can occur because data can change between reads.
  - Phantom reads are possible.
- **Use Case:** Default in many systems (e.g., PostgreSQL, SQL Server) for a good balance of performance and consistency.

**Example:**
- **Transaction A:** Commits a change to a row.
- **Transaction B:** Reads the row before and after Transaction A’s commit, observing different values.

---

#### 3. **Repeatable Read**
- **Behavior:** Ensures that if a transaction reads a row once, it will see the same data if it re-reads it. Prevents dirty reads and non-repeatable reads.
- **Issues:**
  - Phantom reads can still occur because new rows might be added or deleted by other transactions.
- **Use Case:** Suitable for use cases like financial applications where consistent reads are crucial.

**Example:**
- **Transaction A:** Reads a row.
- **Transaction B:** Updates the row and commits.
- **Transaction A:** Re-reads the row and still sees the original value.

---

#### 4. **Serializable**
- **Behavior:** The highest level of isolation. Transactions are executed as if they were serialized, one after the other, ensuring complete consistency.
- **Issues:**
  - High performance cost due to locking or aborting conflicting transactions.
- **Use Case:** Critical systems requiring strict consistency (e.g., banking systems).

**Example:**
- **Transaction A:** Queries for all accounts with balances > $10,000.
- **Transaction B:** Inserts a new account with a balance of $15,000.
- **Transaction A:** Will not see the new account until it completes its transaction.

---

### **Comparison Table**

| Isolation Level      | Dirty Reads | Non-Repeatable Reads | Phantom Reads   |
|-----------------------|-------------|-----------------------|-----------------|
| **Read Uncommitted** | Yes         | Yes                   | Yes             |
| **Read Committed**   | No          | Yes                   | Yes             |
| **Repeatable Read**  | No          | No                    | Yes             |
| **Serializable**     | No          | No                    | No              |

---

### **Practical Example in SQL**
Suppose we have a table `accounts`:

```sql
CREATE TABLE accounts (
    id INT PRIMARY KEY,
    balance INT
);
INSERT INTO accounts VALUES (1, 100), (2, 200);
```

#### Read Committed Example:
- **Transaction 1:**
  ```sql
  BEGIN;
  UPDATE accounts SET balance = balance - 50 WHERE id = 1;
  -- No COMMIT yet
  ```
- **Transaction 2:**
  ```sql
  BEGIN;
  SELECT balance FROM accounts WHERE id = 1; -- Will see the original value (100)
  COMMIT;
  ```
  
#### Repeatable Read Example:
- **Transaction 1:**
  ```sql
  BEGIN;
  SELECT balance FROM accounts WHERE id = 1; -- Reads balance = 100
  ```
- **Transaction 2:**
  ```sql
  BEGIN;
  UPDATE accounts SET balance = 150 WHERE id = 1;
  COMMIT;
  ```
- **Transaction 1:**
  ```sql
  SELECT balance FROM accounts WHERE id = 1; -- Still sees 100 due to Repeatable Read
  ```

---

### **Summary**
Isolation levels provide a mechanism to balance performance and consistency in a database. They are chosen based on the application's tolerance for concurrency issues like dirty reads, non-repeatable reads, or phantom reads. Understanding their trade-offs is essential for designing robust systems.
Let’s dive deeper into **Repeatable Read** and explain it step by step with a clear example to help you understand.

---

### **What is Repeatable Read?**

**Repeatable Read** ensures that if a transaction reads a row once, it will always see the same value for that row, even if other transactions modify it. This prevents **dirty reads** (reading uncommitted changes) and **non-repeatable reads** (seeing different values when re-reading the same data).

However, **phantom reads** (new rows appearing in the result set due to inserts/deletes by other transactions) can still occur.

---

### **Key Characteristics:**
1. Once a transaction reads a row, that row cannot be modified by other transactions until the first transaction finishes.
2. It provides a stable view of the rows that were read during the transaction.
3. It does NOT lock the entire table, so new rows can still be added or deleted, leading to phantom reads.

---

### **Example Step by Step**

#### Setup:
Imagine a bank system with the following `accounts` table:

| **id** | **balance** |
|--------|-------------|
| 1      | 100         |
| 2      | 200         |

---

#### Scenario: Two transactions running concurrently

**Transaction 1:**
- Starts first and is operating under the **Repeatable Read** isolation level.
- Reads the balance of `id = 1`.

**Transaction 2:**
- Runs after Transaction 1 and tries to modify the same row.

---

#### Step-by-Step Execution

1. **Transaction 1 Begins:**
   ```sql
   BEGIN TRANSACTION;
   SELECT balance FROM accounts WHERE id = 1; -- Reads balance = 100
   ```

2. **Transaction 2 Tries to Update:**
   ```sql
   BEGIN TRANSACTION;
   UPDATE accounts SET balance = 150 WHERE id = 1;
   -- Transaction 2 is BLOCKED because Transaction 1 already read the row.
   ```

   - Since Transaction 1 is operating under **Repeatable Read**, it ensures the row it read (`id = 1`) cannot be modified by another transaction until Transaction 1 completes.

3. **Transaction 1 Re-reads:**
   ```sql
   SELECT balance FROM accounts WHERE id = 1; -- Still sees balance = 100
   ```

   - The value remains **100**, ensuring **repeatable reads** for that row, even though another transaction attempted to modify it.

4. **Transaction 1 Commits:**
   ```sql
   COMMIT;
   ```

5. **Transaction 2 Proceeds:**
   - Now that Transaction 1 is finished, Transaction 2 can proceed:
     ```sql
     UPDATE accounts SET balance = 150 WHERE id = 1;
     COMMIT;
     ```

---

### **Why Use Repeatable Read?**

- **Prevents Non-Repeatable Reads:** Ensures that if you read a row multiple times in a transaction, you will always see the same value, even if another transaction tries to change it.
- **Ensures Data Consistency:** Prevents seeing inconsistent or partial updates to rows.

---

### **Limitations of Repeatable Read**
- **Phantom Reads:** 
  - If a query involves a range of rows (e.g., `SELECT * FROM accounts WHERE balance > 100`), other transactions can still insert or delete rows that match the range.
  - Example:
    - **Transaction 1:** Reads rows where `balance > 100`.
    - **Transaction 2:** Inserts a new row with `balance = 150` and commits.
    - **Transaction 1:** Re-executes the query and sees a "phantom" row.

---

### **Summary**
- In **Repeatable Read**, any data you read will remain unchanged for the duration of your transaction.
- It protects against **dirty reads** and **non-repeatable reads**, but not **phantom reads**.
- It’s ideal for scenarios where consistent reading of the same rows is critical, such as financial transactions or inventory systems.

**Eventual Consistency** is a consistency model used primarily in distributed systems to balance availability and performance with consistency. It is one of the key concepts in **CAP Theorem**, which states that in a distributed system, you can achieve at most two of the three: **Consistency**, **Availability**, and **Partition Tolerance**.

---

### **What is Eventual Consistency?**

- **Definition:** Eventual consistency guarantees that if no new updates are made to a distributed database, all replicas (copies of the data) will eventually converge to the same state. 
- It does **not** guarantee that all reads reflect the latest write immediately. Instead, it guarantees that the system will **eventually** become consistent.

---

### **Characteristics:**
1. **Temporary Inconsistency:** Reads might return outdated (stale) data shortly after a write.
2. **Eventual Convergence:** Over time, replicas synchronize, and all nodes reflect the same data if no further updates occur.
3. **Trade-off:** Sacrifices strong consistency for higher availability and partition tolerance.

---

### **When is Eventual Consistency Used?**
- Distributed databases and systems where **high availability** and **partition tolerance** are more critical than immediate consistency.
- Examples: DNS, NoSQL databases like Cassandra, DynamoDB, Riak.

---

### **How It Works?**
- When a write operation occurs, the change is applied to one replica and then propagated to other replicas asynchronously.
- During this propagation, different replicas might temporarily show different states.
- Over time, synchronization mechanisms (e.g., gossip protocols or background replication) ensure all replicas converge to the same state.

---

### **Real-World Example:**
**Use Case: Social Media Like Button**

- **Scenario:** Alice likes a post on social media.
  - The "like" action is recorded on a replica in her region (say, North America).
  - This "like" is asynchronously propagated to other replicas worldwide (e.g., Europe, Asia).
- **Result:**
  - Bob (in Europe) might see the old count of likes (stale data) for a few seconds or minutes until his replica updates.
  - Eventually, all replicas converge, and everyone sees the same like count.

---

### **Eventual Consistency in Action**
1. **Write Operation:**
   - A user updates data (e.g., adds a like or modifies a document).
   - The change is immediately acknowledged on the primary replica.
2. **Asynchronous Replication:**
   - The update propagates to other replicas in the background.
   - Until all replicas synchronize, reads from different replicas might return different values.
3. **Convergence:**
   - After all replicas synchronize, the system achieves consistency.

---

### **Advantages:**
1. **High Availability:** Reads and writes can continue even if some replicas are down.
2. **Scalability:** Suitable for systems distributed across multiple regions.
3. **Performance:** Writes and reads are fast since updates are not blocked by synchronization.

---

### **Disadvantages:**
1. **Stale Reads:** Users might see outdated data.
2. **Complexity:** Designing systems to handle eventual consistency requires careful handling of conflicts (e.g., when two replicas are updated simultaneously).
3. **Not Suitable for Strong Consistency Needs:** Applications like banking systems or inventory management might require strong consistency instead.

---

### **Comparison: Eventual Consistency vs. Strong Consistency**

| **Aspect**                | **Eventual Consistency**                            | **Strong Consistency**                            |
|---------------------------|----------------------------------------------------|--------------------------------------------------|
| **Data Synchronization**  | Asynchronous                                      | Synchronous                                      |
| **Stale Reads**           | Possible                                          | Not possible                                    |
| **Performance**           | High availability and low latency                 | Lower availability and higher latency           |
| **Use Cases**             | Social media, DNS, content delivery systems       | Banking systems, inventory management, ledgers  |

---

### **Example in DynamoDB (Eventual Consistency Default):**

- When you write to a table in DynamoDB, the write is first acknowledged by a single node.
- The update is asynchronously propagated to other nodes.
- A subsequent read might return the old value until all replicas synchronize.

**Query:**
```bash
dynamodb.getItem({ ConsistentRead: false }) 
# Reads might return stale data.
```

---

### **Summary**
- **Eventual Consistency** is ideal for applications where availability and performance are critical, and strong consistency isn't necessary.
- Examples include DNS, distributed NoSQL databases, and systems prioritizing speed and fault tolerance over immediate accuracy.
- Over time, the system guarantees that all replicas converge to the same state.

![image](https://github.com/user-attachments/assets/a968dce6-2e17-4246-9db1-1edaa867143a)

---

### **I/O and Caching:**
- Databases use **buffer pools** (memory caches) to minimize disk I/O.Understanding how databases store tables and indexes on disk involves exploring concepts like **pages**, **data structures (B-Trees, etc.)**, and **file organization**. Here’s a detailed explanation:

---

## **1. Tables and Pages**
Tables are collections of data, and databases store this data in **pages** on disk.

### **What is a Page?**
- A **page** is the smallest unit of storage in a database.
- Databases like PostgreSQL, MySQL, and SQL Server typically use a default page size (e.g., 8 KB or 16 KB).
- Pages contain rows of data for a table.

### **How Tables are Stored?**
- **Heap File:**
  - Tables are often stored as **heap files**, which are unordered collections of pages.
  - Each page contains:
    - **Row Data:** The actual records (e.g., rows of a table).
    - **Header:** Metadata about the page (e.g., free space, offsets).
    - **Slot Array:** Pointers to individual rows in the page.

- When a table grows, the database adds more pages and organizes them in **data files** on disk.

---

## **2. Indexes and B-Trees**
Indexes are created to speed up data retrieval. Most databases use **B-Trees** or **B+Trees** as the data structure for indexes.

### **What is a B-Tree?**
- A **B-Tree** is a self-balancing tree structure.
- It ensures that data is sorted and enables logarithmic search times (`O(log n)`).

### **Structure of a B-Tree Index:**
- **Root Node:** The top-level node, containing keys that divide the data.
- **Internal Nodes:** Contain keys and pointers to child nodes.
- **Leaf Nodes:** Contain pointers to the actual data (or, in some cases, the data itself).

#### **How Data is Stored in a B-Tree Index:**
1. **Keys and Pointers:**
   - Each node in the B-Tree stores keys (e.g., values from the indexed column) and pointers to other nodes or data pages.
2. **Balance:**
   - The B-Tree automatically balances itself as data is inserted or deleted, ensuring efficient search operations.
3. **Leaf Nodes:**
   - In a **B+Tree** (commonly used), the leaf nodes are linked for efficient range scans.

---

## **3. Pages in Indexes**
- Indexes also use **pages** for storage:
  - **Root Page:** Contains pointers to lower-level pages (internal nodes).
  - **Internal Pages:** Organize the keys and child pointers.
  - **Leaf Pages:** Contain the final keys and pointers to table data or actual data.

### **Clustered vs Non-Clustered Indexes**
- **Clustered Index:**
  - The table data is physically stored in the order of the index.
  - The leaf nodes contain the actual table rows.
- **Non-Clustered Index:**
  - The index is separate from the table.
  - The leaf nodes contain pointers (e.g., Row IDs) to the actual table rows.

---

## **4. Write and Read Operations**
### **Writing Data:**
1. **New Row Insert:**
   - The database finds a page with enough free space.
   - If no space is available, a new page is allocated.
2. **Index Update:**
   - If an index exists, the B-Tree structure is updated.
   - Keys are inserted into the appropriate leaf node.

### **Reading Data:**
1. **Full Table Scan:**
   - The database reads all pages sequentially.
2. **Index Scan:**
   - The database uses the B-Tree to quickly locate the relevant data pages.

---

## **5. Disk Layout**
### **Storage Files:**
- Tables and indexes are stored in **files** on disk.
- Each file corresponds to a database object (e.g., a table or an index).

- Pages are read into memory for operations, reducing the need to access the slower disk repeatedly.

---

## **6. Practical Example:**
Imagine a table with a primary key column:
- The database creates a **B+Tree index** for the primary key.
- When you insert a new row:
  1. The row is stored in a **page** within the heap file.
  2. The primary key is added to the **B+Tree index**.
  3. If the page is full, a new page is allocated, and the index adjusts accordingly.

---

### **Summary:**
- **Tables** store rows of data in **pages** on disk.
- **Indexes** use **B-Trees** or **B+Trees** for fast data retrieval.
- Pages are the fundamental unit, containing metadata, row data, or index entries.
- Databases optimize disk I/O with caching and structured organization, ensuring efficient data access.
- ![image](https://github.com/user-attachments/assets/79d196e7-e07c-466e-a1b6-47889f5f1b86)

Row-based and column-based databases differ fundamentally in how they store and access data on disk, impacting their performance and use cases. Let's explore this in depth.

---

### **1. How Data is Stored**

#### **Row-Based Storage (Row-Oriented)**
- Data is stored row by row.
- Each row contains all column values for that row, stored together.
  
**Example (Table Storage):**

| ID  | Name   | Age | City     |
|-----|--------|-----|----------|
| 1   | Alice  | 25  | New York |
| 2   | Bob    | 30  | London   |

- Row-based storage layout:
  ```
  Row 1: [1, Alice, 25, New York]
  Row 2: [2, Bob, 30, London]
  ```

#### **Column-Based Storage (Column-Oriented)**
- Data is stored column by column.
- Each column's values are stored together.

**Example (Table Storage):**

| ID  | Name   | Age | City     |
|-----|--------|-----|----------|
| 1   | Alice  | 25  | New York |
| 2   | Bob    | 30  | London   |

- Column-based storage layout:
  ```
  Column 1: [1, 2]
  Column 2: [Alice, Bob]
  Column 3: [25, 30]
  Column 4: [New York, London]
  ```

---

### **2. Key Differences in Usage**

| Feature                  | Row-Based DB                         | Column-Based DB                  |
|--------------------------|---------------------------------------|-----------------------------------|
| **Access Pattern**       | Reads entire rows.                   | Reads specific columns.          |
| **Write Performance**    | Optimized for frequent inserts/updates. | Slower for frequent writes.      |
| **Query Performance**    | Good for single-row lookups.          | Excellent for analytical queries (aggregate operations). |
| **Compression**          | Limited (row data varies widely).    | High (columnar data often has redundancy). |
| **Use Cases**            | OLTP (transactional workloads).       | OLAP (analytical workloads).     |

---

### **3. Advantages and Disadvantages**

#### **Row-Based Databases**
**Advantages:**
1. **Efficient for OLTP:**
   - Row-based storage is ideal for transactional systems where queries frequently access entire rows (e.g., `SELECT * FROM users WHERE id = 1`).
2. **Easy Updates and Inserts:**
   - Data is written and read in complete rows, making row updates and inserts straightforward.

**Disadvantages:**
1. **Inefficient for Analytical Queries:**
   - Aggregate queries (e.g., `SUM(salary)` or `AVG(age)`) require scanning unnecessary columns, leading to higher I/O costs.
2. **Less Compression:**
   - Row data is more varied, limiting compression effectiveness.

#### **Column-Based Databases**
**Advantages:**
1. **Efficient for OLAP:**
   - Analytical queries benefit from reading only the relevant columns, reducing I/O.
2. **Better Compression:**
   - Columnar data often has redundancy, allowing for efficient compression algorithms.

**Disadvantages:**
1. **Slower Writes:**
   - Writing a single row requires updating multiple column segments.
2. **Complex Updates:**
   - Row-level updates are less efficient because the database must locate and modify values across different column stores.

---

### **4. Query Execution**
#### **Row-Based Query Execution**
- For `SELECT * FROM users WHERE id = 1`:
  - The database retrieves the entire row in one disk operation.
  - Efficient for queries accessing all columns or specific rows.

#### **Column-Based Query Execution**
- For `SELECT AVG(age) FROM users`:
  - Only the `age` column is scanned.
  - This minimizes disk I/O and speeds up aggregate computations.

---

### **5. Use Cases**
#### **Row-Based DB Use Cases (OLTP):**
- **Examples: MySQL, PostgreSQL, Oracle DB**
  - Banking transactions.
  - E-commerce systems.
  - Real-time data entry applications.

#### **Column-Based DB Use Cases (OLAP):**
- **Examples: Apache Cassandra, Amazon Redshift, Google BigQuery**
  - Data warehousing.
  - Business intelligence.
  - Analytics-heavy systems.

---

### **6. Hybrid Models**
Some databases combine row and columnar storage:
- **Example:** PostgreSQL with its `cstore_fdw` extension or databases like **MariaDB ColumnStore**.
- **Hybrid Approach:** Use row storage for transactional tables and columnar storage for analytics, optimizing for both workloads.

---

### **7. Technical Insights**
#### **Storage and Compression:**
- **Row-based:** Stores varied data, limiting compression.
- **Column-based:** Similar column values enable efficient compression (e.g., Run-Length Encoding, Delta Encoding).

#### **Indexing:**
- **Row-based:** Uses traditional indexes (B-Trees, Hash Indexes).
- **Column-based:** Relies on efficient sequential scans due to its compact data storage.

#### **Parallel Processing:**
- Column-based databases enable better parallelization since operations can work independently on specific columns.

---

### **Summary**
| Aspect               | Row-Based DB                      | Column-Based DB                  |
|----------------------|------------------------------------|-----------------------------------|
| Storage Structure    | Row-oriented                     | Column-oriented                  |
| Strength             | Transactional systems (OLTP)      | Analytical systems (OLAP)        |
| Performance          | Faster for row-based operations  | Faster for columnar queries      |
| Compression          | Limited                          | High                             |

Choosing between row-based and column-based databases depends on the application's needs. **Row-based databases** excel in transactional workloads, while **column-based databases** are tailored for analytics and reporting.

Indexes in databases are powerful tools for speeding up data retrieval. Let's dive deep into **primary index**, **secondary index**, and **clustered index**, highlighting their characteristics, differences, and use cases.

---

### **1. Primary Index**

#### **Definition:**
- A **primary index** is automatically created when a table's **primary key** is defined.
- It ensures that each row in the table has a unique identifier (the primary key) and that the data is sorted based on this key.

#### **Key Characteristics:**
- **Unique:** No duplicate values are allowed.
- **Automatically Created:** Defined when the primary key is set on a table.
- **Organizes Table Data:** The table rows are stored in the order of the primary index.
- **Clustered Index:** In many database systems (like MySQL InnoDB), the primary index is implemented as a **clustered index**.

#### **Example:**
Consider a table:

| ID  | Name   | Age | City     |
|-----|--------|-----|----------|
| 1   | Alice  | 25  | New York |
| 2   | Bob    | 30  | London   |
| 3   | Carol  | 22  | Paris    |

If `ID` is the primary key, a **primary index** ensures:
- Data is sorted by `ID`.
- The index directly points to the rows in storage.

#### **Use Case:**
Efficient for queries that use the primary key (`SELECT * FROM table WHERE ID = 2`).

---

### **2. Clustered Index**

#### **Definition:**
- A **clustered index** determines the **physical order of rows** in a table.
- There can be **only one clustered index** per table because the table's data can only be sorted in one way.

#### **Key Characteristics:**
- **Data Storage:** The table's rows are stored in the same order as the clustered index.
- **Primary Key as Clustered Index:** Most databases (e.g., MySQL's InnoDB) automatically make the primary key a clustered index.
- **Efficient Range Queries:** Since rows are physically ordered, range queries (`BETWEEN`, `>`, `<`) are very fast.

#### **Example:**
If a clustered index is built on `ID`, the rows are physically stored in `ID` order:
```
Physically stored data: [1, Alice], [2, Bob], [3, Carol]
```

#### **Use Case:**
Ideal for primary key lookups and range-based queries (`WHERE ID BETWEEN 2 AND 4`).

---

### **3. Secondary Index**

#### **Definition:**
- A **secondary index** is any index created **in addition to the primary index**.
- It does not affect the physical order of rows in the table.
- Often referred to as a **non-clustered index** in some systems.

#### **Key Characteristics:**
- **Logical Order:** Points to rows in a logical way, not physical.
- **Multiple Allowed:** You can create many secondary indexes on a table.
- **Extra Storage:** Requires additional space to store the index structure.
- **Slower Writes:** Increases the cost of insert/update/delete operations because the index needs to be updated.

#### **Example:**
If a secondary index is created on `Name`:
- The database maintains a separate structure (e.g., a B+Tree) to store `Name` values and pointers to their corresponding rows.

```
Secondary Index:
[Name: Alice -> Row 1]
[Name: Bob -> Row 2]
[Name: Carol -> Row 3]
```

#### **Use Case:**
Efficient for queries on non-primary key columns (`SELECT * FROM table WHERE Name = 'Bob'`).

---

### **Comparison of Index Types**

| Aspect                | Primary Index                 | Clustered Index                 | Secondary Index                |
|-----------------------|-------------------------------|---------------------------------|--------------------------------|
| **Definition**         | Index on the primary key.     | Determines physical row order.  | Additional indexes for faster lookups. |
| **Relation to Data**   | Often a clustered index.      | Directly organizes table data.  | Points to data logically.      |
| **Uniqueness**         | Always unique.               | Often unique.                   | Can be non-unique.             |
| **Order of Rows**      | Sorts rows by primary key.    | Sorts rows by indexed column.   | Does not affect row order.     |
| **Number Allowed**     | One per table.               | One per table.                  | Many per table.                |
| **Space Usage**        | Minimal (integrated with table). | Integrated with table.          | Requires extra storage.        |
| **Use Case**           | Identifying rows uniquely.    | Range queries, primary key lookups. | Non-primary key queries.       |

---

### **Key Points to Remember**
1. **Primary Index**:
   - Often implemented as a **clustered index**.
   - Ensures unique, sorted storage.

2. **Clustered Index**:
   - Determines the physical layout of table data.
   - Only one per table.

3. **Secondary Index**:
   - Additional indexes for non-primary key columns.
   - Separate structure for faster lookups.

---

### **Practical Example**
**Table Schema:**
```sql
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY,
    Name VARCHAR(100),
    Age INT,
    DepartmentID INT,
    INDEX (Name)  -- Secondary Index
);
```

1. **Primary Index:**
   - Created on `EmployeeID`.
   - Data stored in ascending order of `EmployeeID`.

2. **Clustered Index:**
   - In most databases, `EmployeeID` is the clustered index (default for the primary key).

3. **Secondary Index:**
   - Created on `Name`.
   - Maintains a separate structure for quick lookups on `Name`.

---

### **Summary**
- **Primary Index:** Defines the main way to access rows uniquely, often tied to the primary key.
- **Clustered Index:** Physically organizes table rows for efficient retrieval; only one per table.
- **Secondary Index:** Additional indexes for improving query performance on non-primary key columns.

Each type has distinct purposes and advantages, depending on the table's usage pattern and query requirements.

---
### **What Is Indexing in a Database?**

Indexing in a database is a technique used to optimize the performance of queries by reducing the amount of data that needs to be scanned. It creates a data structure (like a B-Tree or hash table) that allows the database to locate and retrieve rows faster, based on the indexed column(s).

---

### **How Does Indexing Work?**

1. **Data Organization:**
   - An index is created for specific column(s) in a table.
   - The index stores pointers to the actual rows in the table, organized in a way that allows for fast lookups (e.g., using a B-Tree).

2. **Search Optimization:**
   - Without an index: The database performs a **full table scan**, checking each row sequentially to find the desired data.
   - With an index: The database directly uses the index to narrow down the search to the relevant rows, skipping unnecessary data.

---

### **Types of Indexes**

#### 1. **Primary Index**
   - Automatically created for the **primary key** of a table.
   - Ensures the rows are stored in sorted order based on the primary key.

#### 2. **Secondary Index**
   - Created on non-primary key columns to improve query performance.
   - Does not affect the physical order of rows in the table.
   - Useful for columns frequently used in search conditions (`WHERE`, `JOIN`, etc.).

#### 3. **Clustered Index**
   - Determines the physical order of data in a table.
   - A table can have only one clustered index because the rows can be physically sorted only one way.

#### 4. **Non-Clustered Index**
   - Does not affect the physical order of rows in the table.
   - Maintains a separate structure that points to the actual data.

#### 5. **Unique Index**
   - Ensures all values in the indexed column(s) are unique.
   - Often used for enforcing constraints like `UNIQUE`.

#### 6. **Composite Index**
   - Created on two or more columns.
   - Useful when queries involve filtering or sorting based on multiple columns.

#### 7. **Full-Text Index**
   - Used for full-text search in columns containing large text data.
   - Enables fast search of keywords, phrases, or patterns.

#### 8. **Hash Index**
   - Uses a hash table for lookups.
   - Provides fast equality searches (`=`) but is less effective for range queries.

---

### **Structure of an Index**
Indexes are typically implemented using data structures like:

1. **B+Tree (Balanced Tree):**
   - Most common structure for traditional indexes.
   - Allows fast range queries and ordered traversal.

2. **Hash Tables:**
   - Used for equality searches.
   - Does not support range queries.

3. **Bitmaps:**
   - Used in analytical databases for indexing columns with a small number of distinct values.

---

### **Advantages of Indexing**
- **Faster Queries:** Reduces the data scanned during query execution.
- **Efficient Sorting:** Optimizes `ORDER BY` and `GROUP BY` operations.
- **Speeding Up Joins:** Improves the performance of queries involving `JOIN` operations.
- **Constraint Enforcement:** Helps enforce unique constraints.

---

### **Disadvantages of Indexing**
- **Storage Overhead:** Indexes consume additional disk space.
- **Insert/Update Overhead:** Modifying data in indexed columns requires updating the index, slowing down write operations.
- **Too Many Indexes:** Excessive indexing can degrade performance instead of improving it.

---

### **When to Use Indexing**
1. **Frequently Queried Columns:**
   - Columns used in `WHERE`, `JOIN`, or `ORDER BY` clauses.
2. **Unique Columns:**
   - Columns with unique values that often require lookups.
3. **Range Queries:**
   - For queries with conditions like `BETWEEN` or `<` and `>`.

---

### **Example in SQL**
```sql
-- Creating an index on a single column
CREATE INDEX idx_customer_name ON customers(name);

-- Creating a composite index on multiple columns
CREATE INDEX idx_order_customer_date ON orders(customer_id, order_date);

-- Dropping an index
DROP INDEX idx_customer_name;
```

---

### **How Indexing Improves Performance**
1. **Query Plan Selection:**
   - When a query is executed, the database optimizer checks for available indexes and uses the most efficient one.
2. **Faster Row Lookup:**
   - The index allows the database to find rows without scanning the entire table.
3. **Range Scans:**
   - For range conditions, the index tree (e.g., B+Tree) can quickly traverse the relevant range of rows.

Indexing is a powerful tool but should be used thoughtfully to balance read and write performance in your database.

When you index a column in a database, the indexed column enables **faster query performance**, but the extent of the speed improvement depends on how the column is used in queries. Here's a breakdown of the scenarios:

---

### **1. Faster for Literal Value Matches**
   - **Example:** 
     ```sql
     SELECT * FROM employees WHERE id = 101;
     ```
   - The index is highly efficient for **exact matches** like the query above because the database can quickly locate the value in the index structure (e.g., B+Tree or hash index) without scanning the entire table.

---

### **2. Faster for Range Queries**
   - **Example:**
     ```sql
     SELECT * FROM employees WHERE salary BETWEEN 50000 AND 100000;
     ```
   - Indexing also accelerates **range queries**:
     - For a B+Tree index, the database can quickly identify the starting point (`50000`) and traverse the index to the endpoint (`100000`).
     - This avoids scanning irrelevant rows.

---

### **3. Faster for Pattern Matching (Limited Cases)**
   - **Example:**
     ```sql
     SELECT * FROM employees WHERE name LIKE 'John%';
     ```
   - Indexing works well for pattern matching **when the query starts with a fixed prefix** (`'John%'`):
     - The database uses the index to locate rows starting with `'John'`.

   - **Not Efficient for Leading Wildcards:**
     ```sql
     SELECT * FROM employees WHERE name LIKE '%John';
     ```
     - Leading wildcards (`'%John'`) prevent the database from using the index efficiently, as the search pattern isn't predictable.

---

### **4. Not Just for Literal Values**
   Indexes improve performance in various scenarios beyond exact literal matches:

   - **Sorting:**
     ```sql
     SELECT * FROM employees ORDER BY salary;
     ```
     - Indexes eliminate the need for expensive sorting operations, as the data is already logically ordered in the index.

   - **Joins:**
     ```sql
     SELECT e.name, d.name
     FROM employees e
     JOIN departments d ON e.department_id = d.id;
     ```
     - Indexes on the `department_id` column improve join efficiency by quickly locating matching rows.

   - **Grouping and Aggregations:**
     ```sql
     SELECT department_id, COUNT(*)
     FROM employees
     GROUP BY department_id;
     ```
     - Indexes can speed up grouping by organizing rows with the same key together in the index.

   - **Foreign Key Lookups:**
     - Indexes on foreign key columns make lookups in parent tables faster.

---

### **Why Literal Matches Are the Fastest**
Literal matches involve finding a single, specific value in the index. Since indexes are designed to allow quick lookups:
   - **Hash Index:** Directly maps the value to its location.
   - **B+Tree Index:** Traverses the tree to locate the exact value in `O(log n)` time.

For other scenarios like range queries or sorting, indexes still help but involve traversing more entries or combining results, which can take slightly longer than literal lookups.

---

### **Key Takeaways**
- Literal value matches benefit the most from indexing due to direct lookups.
- Indexes also improve the performance of range queries, pattern matching (with prefixes), sorting, grouping, joins, and foreign key lookups.
- Proper indexing strategies depend on the query patterns to achieve optimal performance.

### **What Happens When You Index a Column?**
When you create an index on a specific column in a database, the following takes place:

1. **Logical Ordering in the Index:**
   - **Index Structure:** The index structure (e.g., a B+Tree or a hash) maintains the values of the column in a sorted manner. This allows the database to perform quick searches, as it can traverse the tree to find the specific value or range.
   - **Physical Table:** The underlying table data does not get physically sorted. The index provides a logical map to the data, allowing the database to jump directly to the relevant rows when queried.

2. **Separate Structure:** 
   - An index is essentially a separate data structure that holds a copy of the indexed column's values and pointers to the rows in the main table. This structure is what allows for faster lookups, sorting, and filtering.

---

### **How Can You Have More Than One Index on a Table?**
You can create multiple indexes on different columns or combinations of columns in a table to optimize various query patterns.

1. **Single-Column Indexes:**
   - You can create an index on each column you query frequently.
   ```sql
   CREATE INDEX idx_employee_name ON employees(name);
   CREATE INDEX idx_employee_salary ON employees(salary);
   ```

2. **Composite (Multi-Column) Indexes:**
   - A composite index includes multiple columns and can be used to speed up queries that filter or sort by more than one column.
   ```sql
   CREATE INDEX idx_employee_department_salary ON employees(department_id, salary);
   ```
   - The order of columns in a composite index matters. The index is most effective when queries use the columns from left to right.
   - For example, `WHERE department_id = 5 AND salary > 50000` can use `idx_employee_department_salary`, but `WHERE salary > 50000` alone may not benefit as much.

3. **Unique Indexes:**
   - These enforce uniqueness for a column's values, preventing duplicate entries.
   ```sql
   CREATE UNIQUE INDEX idx_employee_email ON employees(email);
   ```

4. **Full-Text Indexes:**
   - Used for efficient text searching within larger text fields.
   ```sql
   CREATE FULLTEXT INDEX idx_employee_bio ON employees(bio);
   ```

---

### **Impact on Table Operations:**
- **Inserts/Updates/Deletes:** Maintaining indexes incurs additional overhead. When data changes, the database must update the index structures, which can slow down write operations.
- **Storage Space:** Each index takes up space on disk. More indexes mean higher storage usage and potentially more memory required for caching.

---

### **Multiple Index Use in a Query:**
- **Single-Index Use:** The database optimizer typically selects the index that offers the most efficient plan for a given query.
- **Bitmap Index Scan:** In cases where multiple conditions are involved, the database can combine results from multiple indexes using a **bitmap scan**.

### **How to Choose Which Indexes to Create:**
1. **Analyze Query Patterns:** Identify columns frequently used in `WHERE`, `JOIN`, `ORDER BY`, or `GROUP BY` clauses.
2. **Composite Indexes for Multi-Column Filters:** If your queries often filter by two or more columns, consider a composite index.
3. **Avoid Redundant Indexes:** Creating too many indexes can increase maintenance costs and storage usage without much benefit.
4. **Use Database Tools:** Most databases provide tools to analyze query performance (`EXPLAIN` in PostgreSQL/MySQL) to help identify if indexes are being used effectively.

---

### **Example Scenario:**
#### Table Definition:
```sql
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name TEXT,
    department_id INT,
    salary INT,
    email TEXT UNIQUE
);
```

#### Index Creation:
```sql
-- Single-column index
CREATE INDEX idx_employee_department ON employees(department_id);

-- Composite index for combined filtering
CREATE INDEX idx_employee_department_salary ON employees(department_id, salary);
```

#### Query Usage:
```sql
-- This query will likely use idx_employee_department
SELECT * FROM employees WHERE department_id = 10;

-- This query will use idx_employee_department_salary efficiently
SELECT * FROM employees WHERE department_id = 10 AND salary > 60000;
```

### **Key Takeaways:**
- Indexing a column creates a separate, logically sorted structure that references the main table, speeding up query performance.
- Multiple indexes can be created on a table, each optimized for different query patterns.
- Proper indexing strategy involves understanding query patterns, optimizing for read performance, and balancing write overhead.
