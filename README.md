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
