# Phase 3: H2 Database Integration

**Duration**: 4-5 hours
**Complexity**: ⭐⭐ Beginner-Intermediate

## Learning Objectives

By the end of this phase, you will:
- Understand what databases are and why they're needed
- Learn JPA (Java Persistence API) basics
- Configure H2 in-memory database
- Use Spring Data JPA repositories
- Convert your in-memory app to use a database
- Access H2 console for debugging

## Prerequisites

- ✅ Completed Phase 2
- ✅ Understand CRUD operations
- ✅ Service layer implemented

---

## Why Do We Need a Database?

### Current Problem (Phase 2)
- Data stored in ArrayList
- Data lost when application restarts
- No data persistence
- Can't scale (limited by memory)

### Solution: Database
- **Persistent storage**: Data survives restarts
- **Reliability**: Data is safely stored on disk
- **Scalability**: Can handle millions of records
- **Concurrent access**: Multiple users can access data

---

## What is JPA?

**JPA** = Java Persistence API

### The Problem JPA Solves

**Without JPA** (writing SQL manually):
```java
String sql = "INSERT INTO expenses (description, amount, date) VALUES (?, ?, ?)";
PreparedStatement stmt = connection.prepareStatement(sql);
stmt.setString(1, expense.getDescription());
stmt.setDouble(2, expense.getAmount());
stmt.setDate(3, expense.getDate());
stmt.executeUpdate();
// Plus error handling, connection management, etc.
```

**With JPA** (simple and clean):
```java
expenseRepository.save(expense);
```

### What JPA Does
- Converts Java objects to database rows (and back)
- Generates SQL automatically
- Manages database connections
- Handles transactions

### Hibernate
- Most popular JPA implementation
- Included with Spring Boot
- You write Java, Hibernate generates SQL

---

## What is H2 Database?

**H2** is a lightweight Java database that:
- Runs in memory (fast, perfect for learning)
- Requires no installation
- Includes web console for viewing data
- Can be configured to save to disk
- Will be replaced with MySQL in Phase 6

**Analogy**: H2 is like a notepad (temporary), MySQL is like a filing cabinet (permanent)

---

## Step 1: Add Dependencies

### Update pom.xml

Open `pom.xml` and add these dependencies inside `<dependencies>`:

```xml
<!-- Spring Data JPA -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<!-- H2 Database -->
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
```

### Refresh Maven
- **IntelliJ**: Click the Maven refresh icon (top right)
- **Eclipse**: Right-click project → Maven → Update Project

---

## Step 2: Configure H2 Database

### Update application.properties

Open `src/main/resources/application.properties` and add:

```properties
# H2 Database Configuration
spring.datasource.url=jdbc:h2:mem:expensedb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

# JPA Configuration
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

# H2 Console (for viewing database)
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
```

### Understanding the Configuration

**Database Connection**:
- `spring.datasource.url=jdbc:h2:mem:expensedb`
  - `jdbc`: Java Database Connectivity
  - `h2`: Using H2 database
  - `mem`: In-memory (data lost on restart)
  - `expensedb`: Database name

**JPA Settings**:
- `ddl-auto=update`: Automatically create/update tables
  - Other options: `create`, `create-drop`, `validate`, `none`
- `show-sql=true`: Print SQL queries to console (helpful for learning)
- `format_sql=true`: Pretty-print SQL

**H2 Console**:
- `enabled=true`: Enable web console
- `path=/h2-console`: Access at http://localhost:8080/h2-console

---

## Step 3: Convert Expense to JPA Entity

### Update Expense.java

Replace your `Expense` class with this JPA-enabled version:

```java
package com.example.expensetracker.model;

import jakarta.persistence.*;
import java.time.LocalDate;

@Entity
@Table(name = "expenses")
public class Expense {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String description;

    @Column(nullable = false)
    private Double amount;

    @Column(nullable = false)
    private LocalDate date;

    // Default constructor (required by JPA)
    public Expense() {
    }

    // Constructor without ID (for creating new expenses)
    public Expense(String description, Double amount, LocalDate date) {
        this.description = description;
        this.amount = amount;
        this.date = date;
    }

    // Getters and Setters (same as before)
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public Double getAmount() {
        return amount;
    }

    public void setAmount(Double amount) {
        this.amount = amount;
    }

    public LocalDate getDate() {
        return date;
    }

    public void setDate(LocalDate date) {
        this.date = date;
    }

    @Override
    public String toString() {
        return "Expense{" +
                "id=" + id +
                ", description='" + description + '\'' +
                ", amount=" + amount +
                ", date=" + date +
                '}';
    }
}
```

### Understanding JPA Annotations

**@Entity**:
- Marks this class as a database table
- JPA will create a table for this class

**@Table(name = "expenses")**:
- Specifies table name in database
- Without this, table name would be "Expense" (class name)

**@Id**:
- Marks this field as the primary key
- Every table needs a primary key (unique identifier)

**@GeneratedValue(strategy = GenerationType.IDENTITY)**:
- Database automatically generates ID values
- `IDENTITY`: Uses database's auto-increment feature
- We no longer need AtomicLong counter

**@Column(nullable = false)**:
- Specifies column constraints
- `nullable = false`: Field is required (NOT NULL in SQL)
- Other options: `length`, `unique`, etc.

---

## Step 4: Create Repository Interface

The **Repository** is your interface to the database.

### Create Repository Package

Create package: `com.example.expensetracker.repository`

### Create ExpenseRepository Interface

Create `ExpenseRepository.java`:

```java
package com.example.expensetracker.repository;

import com.example.expensetracker.model.Expense;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.time.LocalDate;
import java.util.List;

@Repository
public interface ExpenseRepository extends JpaRepository<Expense, Long> {

    // Custom query methods (Spring generates SQL automatically)

    // Find expenses by date
    List<Expense> findByDate(LocalDate date);

    // Find expenses between dates
    List<Expense> findByDateBetween(LocalDate startDate, LocalDate endDate);

    // Find expenses by description containing keyword
    List<Expense> findByDescriptionContainingIgnoreCase(String keyword);

    // Find expenses greater than amount
    List<Expense> findByAmountGreaterThan(Double amount);
}
```

### Understanding the Repository

**JpaRepository<Expense, Long>**:
- `Expense`: The entity type
- `Long`: The type of the primary key (ID)

**Built-in Methods** (no implementation needed!):
- `save(expense)`: Insert or update
- `findById(id)`: Find by ID
- `findAll()`: Get all records
- `deleteById(id)`: Delete by ID
- `count()`: Count records
- And many more!

**Custom Query Methods**:
- Spring generates SQL based on method names
- `findByDate` → `SELECT * FROM expenses WHERE date = ?`
- `findByDescriptionContainingIgnoreCase` → `SELECT * FROM expenses WHERE LOWER(description) LIKE LOWER(?)`

**How does this work?**
- Spring Data JPA reads the method name
- Generates appropriate SQL query
- You just call the method!

---

## Step 5: Update ExpenseService

### Replace ExpenseService.java

```java
package com.example.expensetracker.service;

import com.example.expensetracker.model.Expense;
import com.example.expensetracker.repository.ExpenseRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.time.LocalDate;
import java.util.List;
import java.util.Optional;

@Service
public class ExpenseService {

    private final ExpenseRepository expenseRepository;

    @Autowired
    public ExpenseService(ExpenseRepository expenseRepository) {
        this.expenseRepository = expenseRepository;
    }

    // Create expense
    public Expense createExpense(Expense expense) {
        return expenseRepository.save(expense);
    }

    // Get all expenses
    public List<Expense> getAllExpenses() {
        return expenseRepository.findAll();
    }

    // Get expense by ID
    public Optional<Expense> getExpenseById(Long id) {
        return expenseRepository.findById(id);
    }

    // Update expense
    public Optional<Expense> updateExpense(Long id, Expense updatedExpense) {
        Optional<Expense> existingExpense = expenseRepository.findById(id);

        if (existingExpense.isPresent()) {
            Expense expense = existingExpense.get();
            expense.setDescription(updatedExpense.getDescription());
            expense.setAmount(updatedExpense.getAmount());
            expense.setDate(updatedExpense.getDate());

            Expense savedExpense = expenseRepository.save(expense);
            return Optional.of(savedExpense);
        }

        return Optional.empty();
    }

    // Delete expense
    public boolean deleteExpense(Long id) {
        if (expenseRepository.existsById(id)) {
            expenseRepository.deleteById(id);
            return true;
        }
        return false;
    }

    // Additional methods using custom queries

    public List<Expense> getExpensesByDate(LocalDate date) {
        return expenseRepository.findByDate(date);
    }

    public List<Expense> getExpensesByDateRange(LocalDate startDate, LocalDate endDate) {
        return expenseRepository.findByDateBetween(startDate, endDate);
    }

    public List<Expense> searchExpensesByDescription(String keyword) {
        return expenseRepository.findByDescriptionContainingIgnoreCase(keyword);
    }

    public Double getTotalExpenses() {
        return expenseRepository.findAll().stream()
                .mapToDouble(Expense::getAmount)
                .sum();
    }
}
```

### What Changed?

**Before** (Phase 2):
- ArrayList for storage
- AtomicLong for ID generation
- Manual filtering with streams

**Now** (Phase 3):
- Repository for database access
- Database generates IDs
- Repository methods for queries

**Controller stays the same!** (separation of concerns pays off)

---

## Step 6: Test Your Application

### Start the Application

Run `ExpenseTrackerApplication` and watch the console.

### Look for SQL Output

You should see:
```sql
Hibernate: create table expenses (id bigint generated by default as identity, amount double not null, date date not null, description varchar(255) not null, primary key (id))
```

This means Hibernate created the table automatically!

### Test CRUD Operations

Use Postman to test all endpoints (same as Phase 2):

**1. Create Expense**:
```
POST http://localhost:8080/api/expenses
Body: {"description":"Grocery","amount":45.50,"date":"2024-10-20"}
```

**2. Get All Expenses**:
```
GET http://localhost:8080/api/expenses
```

**3. Restart Application**:
- Stop and start the app
- Get all expenses again
- **Data is gone!** (because H2 is in-memory)

**4. This is expected** - we'll fix it in Phase 6 with MySQL

---

## Step 7: Access H2 Console

The H2 console lets you view database tables and run SQL.

### Open H2 Console

1. Go to: `http://localhost:8080/h2-console`
2. You'll see a login page

### Login Settings
- **JDBC URL**: `jdbc:h2:mem:expensedb` (must match `application.properties`)
- **User Name**: `sa`
- **Password**: (leave empty)
- Click "Connect"

### Explore Your Database

**View Table Structure**:
- Left panel: Click `EXPENSES` table
- See columns: ID, DESCRIPTION, AMOUNT, DATE

**Run SQL Query**:
```sql
SELECT * FROM EXPENSES;
```

**Add Data via SQL** (for testing):
```sql
INSERT INTO EXPENSES (DESCRIPTION, AMOUNT, DATE)
VALUES ('Test expense', 25.00, '2024-10-20');
```

**Count Records**:
```sql
SELECT COUNT(*) FROM EXPENSES;
```

---

## Understanding What Happens Behind the Scenes

### Creating an Expense

**Your code**:
```java
expenseRepository.save(expense);
```

**What JPA does**:
1. Converts `Expense` object to SQL:
```sql
INSERT INTO expenses (description, amount, date)
VALUES ('Grocery', 45.50, '2024-10-20');
```
2. Executes SQL
3. Gets generated ID from database
4. Sets ID on expense object
5. Returns the expense

### Getting All Expenses

**Your code**:
```java
expenseRepository.findAll();
```

**What JPA does**:
1. Generates SQL:
```sql
SELECT * FROM expenses;
```
2. Executes query
3. Converts rows to `Expense` objects
4. Returns List<Expense>

---

## Common Issues and Solutions

### Issue 1: Table Not Found

**Error**: `Table "EXPENSES" not found`

**Solutions**:
- Check `@Entity` annotation on Expense class
- Verify `spring.jpa.hibernate.ddl-auto=update`
- Restart application

### Issue 2: Can't Access H2 Console

**Error**: 404 when visiting `/h2-console`

**Solutions**:
- Check `spring.h2.console.enabled=true`
- Verify path: `spring.h2.console.path=/h2-console`
- Restart application

### Issue 3: Wrong JDBC URL in H2 Console

**Error**: Database "expensedb" not found

**Solution**: JDBC URL must exactly match `application.properties`:
- Correct: `jdbc:h2:mem:expensedb`
- Incorrect: `jdbc:h2:mem:testdb`

### Issue 4: Import jakarta.persistence vs javax.persistence

**Error**: Cannot resolve symbol 'Entity'

**Solution**: Use `jakarta.persistence` (newer):
```java
import jakarta.persistence.*;
```

Not `javax.persistence` (older, deprecated in Spring Boot 3+)

---

## Checkpoint Exercises

### Exercise 1: Add Created/Updated Timestamps

Add automatic timestamps for when expense is created and updated.

<details>
<summary>Show Solution</summary>

**Update Expense.java**:
```java
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

@Entity
@Table(name = "expenses")
@EntityListeners(AuditingEntityListener.class)
public class Expense {
    // ... existing fields ...

    @CreatedDate
    @Column(nullable = false, updatable = false)
    private LocalDateTime createdAt;

    @LastModifiedDate
    @Column(nullable = false)
    private LocalDateTime updatedAt;

    // Add getters
}
```

**Enable JPA Auditing** - Add to main application class:
```java
@SpringBootApplication
@EnableJpaAuditing
public class ExpenseTrackerApplication {
    // ...
}
```

Add import: `import org.springframework.data.jpa.repository.config.EnableJpaAuditing;`
</details>

### Exercise 2: Find Expensive Items

Add a method to find expenses above a certain amount.

<details>
<summary>Show Solution</summary>

**In ExpenseRepository.java**:
```java
List<Expense> findByAmountGreaterThanEqual(Double minAmount);
```

**In ExpenseService.java**:
```java
public List<Expense> getExpensiveExpenses(Double minAmount) {
    return expenseRepository.findByAmountGreaterThanEqual(minAmount);
}
```

**In ExpenseController.java**:
```java
@GetMapping("/expensive")
public ResponseEntity<List<Expense>> getExpensiveExpenses(
        @RequestParam Double minAmount) {
    List<Expense> expenses = expenseService.getExpensiveExpenses(minAmount);
    return ResponseEntity.ok(expenses);
}
```

Test: `GET /api/expenses/expensive?minAmount=50`
</details>

### Exercise 3: Count Expenses

Add an endpoint to count total expenses.

<details>
<summary>Show Solution</summary>

**In ExpenseService.java**:
```java
public long countExpenses() {
    return expenseRepository.count();
}
```

**In ExpenseController.java**:
```java
@GetMapping("/count")
public ResponseEntity<Map<String, Long>> countExpenses() {
    long count = expenseService.countExpenses();
    return ResponseEntity.ok(Map.of("count", count));
}
```

Test: `GET /api/expenses/count`
</details>

---

## Key Takeaways

✅ **JPA**: Java Persistence API - converts Java objects to database rows
✅ **@Entity**: Marks a class as a database table
✅ **@Id, @GeneratedValue**: Define primary key with auto-increment
✅ **JpaRepository**: Interface providing database operations
✅ **H2**: In-memory database perfect for development/learning
✅ **H2 Console**: Web interface to view and query database
✅ **spring.jpa.hibernate.ddl-auto=update**: Auto-create tables

---

## What's Next?

In **Phase 4**, you'll learn:
- Input validation
- Custom exception handling
- Proper error responses
- Bean Validation annotations
- Global exception handlers

Your API will become more robust and production-ready!

---

**Next**: [Phase 4: Validation & Error Handling](PHASE_4.md)
