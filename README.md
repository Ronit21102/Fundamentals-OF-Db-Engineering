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
