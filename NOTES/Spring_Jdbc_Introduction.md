# Spring JDBC

**Spring JDBC** = *Spring Java Database Connectivity*
---

## Overview
In Java, we can access data from a database using the **JDBC API**.  
However, there are several problems in building applications using the plain JDBC API.


# Difference Between Checked and Unchecked Exceptions

---

## Overview

- Unlike Java, other programming languages (like **C, C++**) terminate the program **abnormally** when any error occurs during program execution.  
- **Exception Handling** in Java helps to handle failures encountered during program execution and provides an **alternate path of execution** without terminating the program abnormally.

---

## Types of Exceptions in Java

1. **Checked Exceptions**  
   - Checked exceptions are **recoverable**.  
   - They have an **alternate path of execution**.

2. **Unchecked Exceptions**  
   - Unchecked exceptions are **non-recoverable**.  
   - They may **not have an alternate path** of execution.

---

## General Example

### Problem
- **Fever**

### Solution
- No need to terminate — there is an **alternate solution** to mitigate (handle) the problem.

---

### Problem
- **Cancer**

### Solution
- **Non-recoverable** — cannot recover from the failure and eventually has to terminate.

---

## Application Program Example

- **Example 1:**  
  Program = There is no file found to read.  
  - Alternate path = You can read another file instead.  
  - This means the failure is **recoverable**, and the code can continue executing.

- **Example 2:**  
  Object has not been instantiated → `NullPointerException`.  
  - The lines of code after that **cannot execute** because of the exception.  
  - There is **no alternate path of execution**, so the program terminates.

---

## Compiler Enforcement

- When a line of code may throw a **recoverable (checked)** exception,  
  → The **Java compiler enforces** the developer to write a `try/catch` block because there is an alternate path.

- When a line of code may throw a **non-recoverable (unchecked)** exception,  
  → The **Java compiler does not enforce** writing a `try/catch` block because the program will terminate due to no alternate path.  
  → For unchecked exceptions, writing a `try/catch` block is **optional** — it’s left to the developer’s choice.

---

## Execution Behavior

- For **unchecked exceptions**, control moves to the **catch block**, and the catch block executes.  
- After that, depending on the type of failure, the program **may or may not terminate**.

# Java Database Connectivity (JDBC) API Problems

---

## Problem 1: All Exceptions in JDBC API are Checked Exceptions

- In the **JDBC API**, **all exceptions are checked exceptions**.  
- This means the programmer is forced to write **unnecessary `try/catch` blocks** in the application.  
- Even after catching these exceptions, the program often ends up **terminating** because there’s no valid alternate path of execution.

---

### Example: Pseudo Code (Java JDBC API)

```java
public Product findProduct(int productNo)
{
    try
    {
        Class.forName("com.mysql.cj.jdbc.Driver");
        
        // While fetching data from the database, 
        // if the database is down, there is no alternate path of execution.
        Connection con = DriverManager.getConnection(url, userName, password);

        Statement stmt = con.createStatement();
        ResultSet rs = stmt.executeQuery("select * from product where product_id = 1");
    }
    catch (ClassNotFoundException | SQLException e)
    {
        // Here we cannot provide an alternate path of execution because the database is down.
        // Still, the Java compiler enforces writing the try/catch block
        // since all JDBC exceptions are checked exceptions.
    }
}
````
✅ **Summary:**
Even though JDBC forces handling checked exceptions, in real-world cases (like database unavailability),
developers **cannot always recover**, making the enforced try/catch blocks **redundant** and **cluttered**.

# 2. Resource Management in JDBC

---

## Problem: Resource Management

- The **Connection**, **Statement**, and **ResultSet** objects created in a JDBC program **must be closed** at the end of their usage.  
- Any **mismanagement or failure** to close these resources properly can lead to **resource leakage issues**.  
- No matter how carefully we handle the code for closing resources, **JDBC programs are still prone to leaks** due to their verbose and error-prone nature.

---

### Example: Pseudo Code (Java JDBC API)

```java
public Product findProduct(int productNo)
{
    try
    {
        Class.forName("com.mysql.cj.jdbc.Driver");
        Connection con = DriverManager.getConnection(url, userName, password);

        Statement stmt = con.createStatement();
        ResultSet rs = stmt.executeQuery("select * from product where product_id = 1");
    }
    catch (ClassNotFoundException | SQLException e)
    {
        // Handle exception
    }
    finally
    {
        try {
            if (rs != null) {
                rs.close();
            }
        } catch (SQLException e) {
            // Handle exception during resource close
        }

        try {
            if (stmt != null) {
                stmt.close();
            }
        } catch (SQLException e) {
            // Handle exception during resource close
        }

        try {
            if (con != null) {
                con.close();
            }
        } catch (SQLException e) {
            // Handle exception during resource close
        }
    }
}
````

---

### Key Observation

* The `close()` method itself **throws `SQLException`**, which again needs to be handled inside **nested try-catch blocks**.
* This leads to **verbose**, **hard-to-maintain**, and **error-prone** code.

---

✅ **Summary:**
Manual resource management in JDBC is **complex and unreliable**.
Even small mistakes can cause **connection leaks** or **database performance issues**, making the code less maintainable and less robust.


# 3. Boilerplate Logic in JDBC Programs

---

## Problem: Repetitive Code

- In JDBC programs, **boilerplate logic** such as opening and closing the **Connection**, **Statement**, and **ResultSet** must be written **in every program**.  
- This repetitive code looks **identical across multiple programs**, adding **unnecessary complexity** and **increasing development time and effort**.

---

### Example of Boilerplate Code (Typical in JDBC)
```java
Class.forName("com.mysql.cj.jdbc.Driver");
Connection con = DriverManager.getConnection(url, userName, password);
Statement stmt = con.createStatement();
ResultSet rs = stmt.executeQuery("SELECT * FROM product");
````

* The same setup code must be written repeatedly across different JDBC programs.
* This leads to **code duplication**, **maintenance difficulties**, and **reduced productivity**.

---

✅ **Summary:**
Boilerplate JDBC code makes applications **verbose**, **hard to maintain**, and **time-consuming to develop**.

---

# 4. Poor Transaction Management in JDBC

---

## Problem: Limited Transaction Support

* The **JDBC API** provides **basic transaction management** features, such as:

  * `setAutoCommit(false)`
  * `commit()`
  * `rollback()`

* However, it supports **only local transactions**, meaning it can manage transactions **within a single database connection**.

---

### Limitation

* JDBC **does not support distributed transactions** (i.e., transactions across multiple databases or systems).
* Managing complex transactional workflows manually becomes **error-prone** and **difficult to maintain**.

---

✅ **Summary:**
The JDBC API is **poor at transaction management**, as it supports only **local transactions**, making it unsuitable for **enterprise-level or multi-database applications**.

# How Spring JDBC Overcomes the Challenges of Traditional JDBC

---

## 1. Simplified Exception Handling

- In **Spring JDBC**, all the exceptions are **unchecked exceptions**.  
- This means developers **don’t need to write unnecessary `try/catch` blocks** unless it’s truly required.

- **Spring JDBC** provides a **rich exception hierarchy** to represent different types of failures reported by the JDBC API or the database itself.

- Spring wraps **Java’s checked JDBC exceptions** inside its own **unchecked exceptions** and **rethrows** them to the application.  
  This makes error handling **cleaner, simpler, and more developer-friendly**.

---

### ✅ Example
Instead of handling exceptions like:
```java
catch (SQLException e) { ... }
````

Spring JDBC throws exceptions such as:

* `DataAccessException`
* `BadSqlGrammarException`
* `DuplicateKeyException`
* `DataIntegrityViolationException`

All of these are **unchecked exceptions** (subclasses of `RuntimeException`).

## Pictorial Respresentation:

<img width="1778" height="488" alt="Screenshot 2025-11-12 141635" src="https://github.com/user-attachments/assets/6142f5ff-6f29-4667-a924-a2dcc590918a" />

---


## 2. Automatic Resource Management

* **Spring JDBC automatically handles resource management** — it takes care of **opening** and **closing** resources such as:

  * `Connection`
  * `Statement`
  * `ResultSet`

* Developers **don’t need to write boilerplate code** to close resources manually.

* As a result, **resource leakage issues** are completely avoided.

---

✅ **Benefit:**

* Cleaner and more maintainable code.
* No redundant `try/finally` blocks for closing connections.

---

## 3. Integrated Transaction Management

* **Spring JDBC integrates seamlessly with Spring’s Transaction Management framework.**

* This allows developers to **manage transactions declaratively**, without writing manual transaction-handling code.

* With Spring JDBC:

  * You don’t need to write code for `commit()` or `rollback()`.
  * Transaction boundaries can be managed using **annotations** or **XML configuration**.

* Spring JDBC supports **both local and global transactions**, making it suitable for **enterprise-level applications**.

---

✅ **Summary:**

| Problem in JDBC                             | Solution in Spring JDBC                                         |
| ------------------------------------------- | --------------------------------------------------------------- |
| Checked exceptions forcing try/catch blocks | All exceptions are unchecked (`RuntimeException`)               |
| Manual resource management prone to leaks   | Automatic resource handling by Spring                           |
| Poor transaction management (local only)    | Integrated, declarative transaction management (local + global) |

---

🚀 **Conclusion:**
Spring JDBC provides a **simpler**, **cleaner**, and **more reliable** approach to database access by addressing the key pain points of traditional JDBC.

Here is your text converted into **Markdown (.md)** format:

# How to work with SpringJdbc?

In Java JDBC there are 3 ways of accessing the data from the database:

1. **Statement**
2. **PreparedStatement**
3. **CallableStatement**

Whereas in SpringJdbc there are multiple ways of accessing the data from the database:

1. **JdbcTemplate** — (Acts similar to `Statement` in Java JDBC)
2. **NamedParameterJdbcTemplate** — (Acts similar to `PreparedStatement` in Java JDBC)

`JdbcTemplate` is the core object of SpringJdbc using which we can perform database operations.

# What are all the things I need to do to access the data from a database and what Spring JDBC provides?

1. **Declare the connection parameters**  
2. **Load the driver and create the connection**  
3. **Prepare the SQL query for accessing the data from the table**  
4. Create `PreparedStatement`  
5. **Supply parameters to SQL**  
6. Substitute parameter values in `PreparedStatement`  
7. Execute the `PreparedStatement`  
8. Gather `ResultSet` and iterate  
9. **Wrap the data into objects**  
10. Close resources

# What are the SQL operations we perform on the database?

- **DDL (Data Definition Language)**  
  `create table`, `drop table`, `alter table`  
  *Not applicable for Java applications*

- **DML (Data Manipulation Language)**  
  `insert`, `update`, `delete`

- **DCL (Data Control Language)**  
  `grant`, `revoke`  
  *Not applicable for Java applications*

- **DQL (Data Query Language)**  
  `select`

- **DTL (Data Transaction Language)**  
  `commit`, `savepoint`, `rollback`  
  *Not applicable for Java applications because Spring uses transaction management*

---

## DML

- insert  
- update  
- delete  

---

## DQL (Possible outcomes for SELECT query)

- **select**
  - aggregate query (`count`, `sum`, `avg`, `min`, `max`)
  - one column one row  
  - one row of data  
  - multiple rows  
  - multiple tables multiple rows

