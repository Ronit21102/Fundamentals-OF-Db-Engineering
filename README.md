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


