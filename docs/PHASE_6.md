# Phase 6: MySQL Migration & Advanced Search

**Duration**: 4-5 hours
**Complexity**: ⭐⭐⭐ Intermediate

## Learning Objectives

By the end of this phase, you will:
- Install and configure MySQL
- Migrate from H2 to MySQL
- Implement advanced search with multiple filters
- Build date range queries
- Create monthly/yearly summaries
- Understand database persistence

## Prerequisites

- ✅ Completed Phase 5
- ✅ Categories and pagination working
- ✅ MySQL 8.0+ installed

---

## Why MySQL?

### H2 Limitations
- Data lost on restart (in-memory)
- Not suitable for production
- Limited to single application instance

### MySQL Benefits
- **Persistent**: Data survives restarts
- **Production-ready**: Used by millions of applications
- **Scalable**: Handles large datasets
- **Reliable**: ACID compliant, transactions
- **Industry standard**: Valuable skill

---

## Step 1: Install MySQL

### Windows
1. Download MySQL Installer: https://dev.mysql.com/downloads/installer/
2. Run installer, choose "Developer Default"
3. Set root password (remember this!)
4. Complete installation

### Mac
```bash
brew install mysql
brew services start mysql
mysql_secure_installation
```

### Linux (Ubuntu/Debian)
```bash
sudo apt update
sudo apt install mysql-server
sudo mysql_secure_installation
```

### Verify Installation
```bash
mysql --version
# Should show: mysql  Ver 8.0.x
```

---

## Step 2: Create Database

### Login to MySQL
```bash
mysql -u root -p
# Enter your root password
```

### Create Database and User
```sql
-- Create database
CREATE DATABASE expense_tracker;

-- Create user (replace 'your_password' with a strong password)
CREATE USER 'expenseapp'@'localhost' IDENTIFIED BY 'your_password';

-- Grant privileges
GRANT ALL PRIVILEGES ON expense_tracker.* TO 'expenseapp'@'localhost';

-- Apply changes
FLUSH PRIVILEGES;

-- Verify
SHOW DATABASES;

-- Exit
EXIT;
```

### Test Connection
```bash
mysql -u expenseapp -p expense_tracker
# Enter password, should connect successfully
```

---

## Step 3: Add MySQL Dependencies

### Update pom.xml

Add MySQL connector:

```xml
<!-- MySQL Connector -->
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <scope>runtime</scope>
</dependency>
```

You can remove or comment out H2:
```xml
<!-- H2 Database - No longer needed -->
<!--
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
-->
```

Refresh Maven dependencies.

---

## Step 4: Update Application Configuration

### Update application.properties

Replace H2 configuration with MySQL:

```properties
# MySQL Database Configuration
spring.datasource.url=jdbc:mysql://localhost:3306/expense_tracker
spring.datasource.username=expenseapp
spring.datasource.password=your_password
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# JPA Configuration
spring.jpa.database-platform=org.hibernate.dialect.MySQL8Dialect
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.jdbc.time_zone=UTC

# Connection Pool Configuration (optional but recommended)
spring.datasource.hikari.maximum-pool-size=10
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.connection-timeout=20000

# Remove H2 Console settings
# spring.h2.console.enabled=true
# spring.h2.console.path=/h2-console
```

### Understanding the Configuration

**spring.datasource.url**:
- `jdbc:mysql://` - MySQL protocol
- `localhost:3306` - MySQL server and port
- `expense_tracker` - Database name

**spring.jpa.hibernate.ddl-auto=update**:
- Creates tables if they don't exist
- Updates schema when entities change
- **Production**: Use `validate` or `none` + migration tools

**time_zone=UTC**:
- Stores dates in UTC
- Prevents timezone issues

---

## Step 5: Run and Verify Migration

### Start Application

Run `ExpenseTrackerApplication`.

### Check Console

You should see:
```sql
Hibernate: create table categories (id bigint not null auto_increment, description varchar(255), name varchar(50) not null, primary key (id))
Hibernate: create table expenses (amount double precision not null, date date not null, description varchar(255) not null, id bigint not null auto_increment, category_id bigint, primary key (id))
```

### Verify in MySQL

```bash
mysql -u expenseapp -p expense_tracker
```

```sql
SHOW TABLES;
# Should show: categories, expenses

DESCRIBE expenses;
# Shows table structure

SELECT * FROM expenses;
# Shows data (currently empty)
```

### Test Data Persistence

1. Create an expense via API (Postman)
2. Restart your Spring Boot application
3. Get all expenses via API
4. **Data should still be there!** 🎉

---

## Step 6: Implement Advanced Search

### Create Search Request DTO

Create `ExpenseSearchRequest.java` in `dto` package:

```java
package com.example.expensetracker.dto;

import org.springframework.format.annotation.DateTimeFormat;
import java.time.LocalDate;

public class ExpenseSearchRequest {

    @DateTimeFormat(iso = DateTimeFormat.ISO.DATE)
    private LocalDate startDate;

    @DateTimeFormat(iso = DateTimeFormat.ISO.DATE)
    private LocalDate endDate;

    private Double minAmount;
    private Double maxAmount;
    private Long categoryId;
    private String description;

    // Getters and Setters
    public LocalDate getStartDate() {
        return startDate;
    }

    public void setStartDate(LocalDate startDate) {
        this.startDate = startDate;
    }

    public LocalDate getEndDate() {
        return endDate;
    }

    public void setEndDate(LocalDate endDate) {
        this.endDate = endDate;
    }

    public Double getMinAmount() {
        return minAmount;
    }

    public void setMinAmount(Double minAmount) {
        this.minAmount = minAmount;
    }

    public Double getMaxAmount() {
        return maxAmount;
    }

    public void setMaxAmount(Double maxAmount) {
        this.maxAmount = maxAmount;
    }

    public Long getCategoryId() {
        return categoryId;
    }

    public void setCategoryId(Long categoryId) {
        this.categoryId = categoryId;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }
}
```

### Update ExpenseRepository

Add method for advanced search using Specification:

```java
package com.example.expensetracker.repository;

import com.example.expensetracker.model.Expense;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.JpaSpecificationExecutor;
import org.springframework.data.jpa.repository.Query;
import org.springframework.stereotype.Repository;

import java.time.LocalDate;
import java.util.List;

@Repository
public interface ExpenseRepository extends JpaRepository<Expense, Long>, JpaSpecificationExecutor<Expense> {

    // Existing methods...
    List<Expense> findByDate(LocalDate date);
    List<Expense> findByDateBetween(LocalDate startDate, LocalDate endDate);
    List<Expense> findByDescriptionContainingIgnoreCase(String keyword);
    Page<Expense> findByCategoryId(Long categoryId, Pageable pageable);
    List<Expense> findByCategoryId(Long categoryId);
    Page<Expense> findByDescriptionContainingIgnoreCase(String keyword, Pageable pageable);
    Page<Expense> findByDateBetween(LocalDate startDate, LocalDate endDate, Pageable pageable);

    @Query("SELECT e FROM Expense e WHERE e.category.id = :categoryId AND e.date BETWEEN :startDate AND :endDate")
    List<Expense> findByCategoryAndDateRange(Long categoryId, LocalDate startDate, LocalDate endDate);

    // NEW: Monthly summary
    @Query("SELECT FUNCTION('MONTH', e.date) as month, SUM(e.amount) as total, COUNT(e) as count " +
           "FROM Expense e WHERE FUNCTION('YEAR', e.date) = :year " +
           "GROUP BY FUNCTION('MONTH', e.date) " +
           "ORDER BY FUNCTION('MONTH', e.date)")
    List<Object[]> getMonthlySummary(int year);

    // NEW: Category summary
    @Query("SELECT c.name, SUM(e.amount), COUNT(e) " +
           "FROM Expense e JOIN e.category c " +
           "WHERE e.date BETWEEN :startDate AND :endDate " +
           "GROUP BY c.id, c.name " +
           "ORDER BY SUM(e.amount) DESC")
    List<Object[]> getCategorySummary(LocalDate startDate, LocalDate endDate);
}
```

### Create Expense Specification

Create `ExpenseSpecification.java` in a new `specification` package:

```java
package com.example.expensetracker.specification;

import com.example.expensetracker.dto.ExpenseSearchRequest;
import com.example.expensetracker.model.Expense;
import jakarta.persistence.criteria.Predicate;
import org.springframework.data.jpa.domain.Specification;

import java.util.ArrayList;
import java.util.List;

public class ExpenseSpecification {

    public static Specification<Expense> getSearchSpecification(ExpenseSearchRequest request) {
        return (root, query, criteriaBuilder) -> {
            List<Predicate> predicates = new ArrayList<>();

            // Date range
            if (request.getStartDate() != null) {
                predicates.add(criteriaBuilder.greaterThanOrEqualTo(
                    root.get("date"), request.getStartDate()));
            }
            if (request.getEndDate() != null) {
                predicates.add(criteriaBuilder.lessThanOrEqualTo(
                    root.get("date"), request.getEndDate()));
            }

            // Amount range
            if (request.getMinAmount() != null) {
                predicates.add(criteriaBuilder.greaterThanOrEqualTo(
                    root.get("amount"), request.getMinAmount()));
            }
            if (request.getMaxAmount() != null) {
                predicates.add(criteriaBuilder.lessThanOrEqualTo(
                    root.get("amount"), request.getMaxAmount()));
            }

            // Category
            if (request.getCategoryId() != null) {
                predicates.add(criteriaBuilder.equal(
                    root.get("category").get("id"), request.getCategoryId()));
            }

            // Description
            if (request.getDescription() != null && !request.getDescription().isEmpty()) {
                predicates.add(criteriaBuilder.like(
                    criteriaBuilder.lower(root.get("description")),
                    "%" + request.getDescription().toLowerCase() + "%"));
            }

            return criteriaBuilder.and(predicates.toArray(new Predicate[0]));
        };
    }
}
```

### Update ExpenseService

Add advanced search method:

```java
package com.example.expensetracker.service;

import com.example.expensetracker.dto.ExpenseSearchRequest;
import com.example.expensetracker.exception.ResourceNotFoundException;
import com.example.expensetracker.model.Category;
import com.example.expensetracker.model.Expense;
import com.example.expensetracker.repository.ExpenseRepository;
import com.example.expensetracker.specification.ExpenseSpecification;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.domain.Specification;
import org.springframework.stereotype.Service;

import java.time.LocalDate;
import java.util.*;

@Service
public class ExpenseService {

    private final ExpenseRepository expenseRepository;
    private final CategoryService categoryService;

    @Autowired
    public ExpenseService(ExpenseRepository expenseRepository, CategoryService categoryService) {
        this.expenseRepository = expenseRepository;
        this.categoryService = categoryService;
    }

    // Existing methods...

    // NEW: Advanced search
    public Page<Expense> searchExpenses(ExpenseSearchRequest request, Pageable pageable) {
        Specification<Expense> spec = ExpenseSpecification.getSearchSpecification(request);
        return expenseRepository.findAll(spec, pageable);
    }

    // NEW: Monthly summary
    public Map<String, Object> getMonthlySummary(int year) {
        List<Object[]> results = expenseRepository.getMonthlySummary(year);

        List<Map<String, Object>> monthlyData = new ArrayList<>();
        double yearTotal = 0.0;

        for (Object[] row : results) {
            int month = (int) row[0];
            double total = row[1] != null ? ((Number) row[1]).doubleValue() : 0.0;
            long count = row[2] != null ? ((Number) row[2]).longValue() : 0;

            Map<String, Object> monthData = new HashMap<>();
            monthData.put("month", month);
            monthData.put("monthName", getMonthName(month));
            monthData.put("total", total);
            monthData.put("count", count);
            monthlyData.add(monthData);

            yearTotal += total;
        }

        Map<String, Object> summary = new HashMap<>();
        summary.put("year", year);
        summary.put("months", monthlyData);
        summary.put("yearTotal", yearTotal);

        return summary;
    }

    // NEW: Category summary
    public List<Map<String, Object>> getCategorySummary(LocalDate startDate, LocalDate endDate) {
        List<Object[]> results = expenseRepository.getCategorySummary(startDate, endDate);

        double grandTotal = results.stream()
                .mapToDouble(row -> row[1] != null ? ((Number) row[1]).doubleValue() : 0.0)
                .sum();

        List<Map<String, Object>> summaryData = new ArrayList<>();

        for (Object[] row : results) {
            String categoryName = (String) row[0];
            double total = row[1] != null ? ((Number) row[1]).doubleValue() : 0.0;
            long count = row[2] != null ? ((Number) row[2]).longValue() : 0;
            double percentage = grandTotal > 0 ? (total / grandTotal) * 100 : 0;

            Map<String, Object> categoryData = new HashMap<>();
            categoryData.put("categoryName", categoryName);
            categoryData.put("total", total);
            categoryData.put("count", count);
            categoryData.put("percentage", Math.round(percentage * 100.0) / 100.0);
            summaryData.add(categoryData);
        }

        return summaryData;
    }

    private String getMonthName(int month) {
        String[] months = {"", "January", "February", "March", "April", "May", "June",
                          "July", "August", "September", "October", "November", "December"};
        return months[month];
    }

    // All existing methods remain...
    public Expense createExpense(Expense expense, Long categoryId) {
        if (categoryId != null) {
            Category category = categoryService.getCategoryById(categoryId);
            expense.setCategory(category);
        }
        return expenseRepository.save(expense);
    }

    public Page<Expense> getAllExpenses(Pageable pageable) {
        return expenseRepository.findAll(pageable);
    }

    public List<Expense> getAllExpenses() {
        return expenseRepository.findAll();
    }

    public Expense getExpenseById(Long id) {
        return expenseRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Expense", "id", id));
    }

    public Expense updateExpense(Long id, Expense updatedExpense, Long categoryId) {
        Expense expense = getExpenseById(id);

        expense.setDescription(updatedExpense.getDescription());
        expense.setAmount(updatedExpense.getAmount());
        expense.setDate(updatedExpense.getDate());

        if (categoryId != null) {
            Category category = categoryService.getCategoryById(categoryId);
            expense.setCategory(category);
        } else {
            expense.setCategory(null);
        }

        return expenseRepository.save(expense);
    }

    public void deleteExpense(Long id) {
        Expense expense = getExpenseById(id);
        expenseRepository.delete(expense);
    }

    public Page<Expense> getExpensesByCategory(Long categoryId, Pageable pageable) {
        categoryService.getCategoryById(categoryId);
        return expenseRepository.findByCategoryId(categoryId, pageable);
    }

    public List<Expense> searchExpensesByDescription(String keyword) {
        return expenseRepository.findByDescriptionContainingIgnoreCase(keyword);
    }

    public List<Expense> getExpensesByDateRange(LocalDate startDate, LocalDate endDate) {
        return expenseRepository.findByDateBetween(startDate, endDate);
    }

    public Double getTotalExpenses() {
        return expenseRepository.findAll().stream()
                .mapToDouble(Expense::getAmount)
                .sum();
    }
}
```

### Update ExpenseController

Add search and summary endpoints:

```java
package com.example.expensetracker.controller;

import com.example.expensetracker.dto.ExpenseSearchRequest;
import com.example.expensetracker.model.Expense;
import com.example.expensetracker.service.ExpenseService;
import jakarta.validation.Valid;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;
import org.springframework.format.annotation.DateTimeFormat;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.time.LocalDate;
import java.util.List;
import java.util.Map;

@RestController
@RequestMapping("/api/expenses")
public class ExpenseController {

    private final ExpenseService expenseService;

    @Autowired
    public ExpenseController(ExpenseService expenseService) {
        this.expenseService = expenseService;
    }

    // Existing endpoints...

    @PostMapping
    public ResponseEntity<Expense> createExpense(
            @Valid @RequestBody Expense expense,
            @RequestParam(required = false) Long categoryId) {
        Expense createdExpense = expenseService.createExpense(expense, categoryId);
        return new ResponseEntity<>(createdExpense, HttpStatus.CREATED);
    }

    @GetMapping
    public ResponseEntity<Page<Expense>> getAllExpenses(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "20") int size,
            @RequestParam(defaultValue = "date") String sortBy,
            @RequestParam(defaultValue = "desc") String sortDir) {

        Sort.Direction direction = sortDir.equalsIgnoreCase("asc") ? Sort.Direction.ASC : Sort.Direction.DESC;
        Pageable pageable = PageRequest.of(page, size, Sort.by(direction, sortBy));

        Page<Expense> expenses = expenseService.getAllExpenses(pageable);
        return ResponseEntity.ok(expenses);
    }

    @GetMapping("/{id}")
    public ResponseEntity<Expense> getExpenseById(@PathVariable Long id) {
        Expense expense = expenseService.getExpenseById(id);
        return ResponseEntity.ok(expense);
    }

    @PutMapping("/{id}")
    public ResponseEntity<Expense> updateExpense(
            @PathVariable Long id,
            @Valid @RequestBody Expense expense,
            @RequestParam(required = false) Long categoryId) {
        Expense updatedExpense = expenseService.updateExpense(id, expense, categoryId);
        return ResponseEntity.ok(updatedExpense);
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteExpense(@PathVariable Long id) {
        expenseService.deleteExpense(id);
        return ResponseEntity.noContent().build();
    }

    // NEW: Advanced search
    @GetMapping("/search")
    public ResponseEntity<Page<Expense>> searchExpenses(
            @RequestParam(required = false) @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate startDate,
            @RequestParam(required = false) @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate endDate,
            @RequestParam(required = false) Double minAmount,
            @RequestParam(required = false) Double maxAmount,
            @RequestParam(required = false) Long categoryId,
            @RequestParam(required = false) String description,
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "20") int size,
            @RequestParam(defaultValue = "date") String sortBy,
            @RequestParam(defaultValue = "desc") String sortDir) {

        ExpenseSearchRequest searchRequest = new ExpenseSearchRequest();
        searchRequest.setStartDate(startDate);
        searchRequest.setEndDate(endDate);
        searchRequest.setMinAmount(minAmount);
        searchRequest.setMaxAmount(maxAmount);
        searchRequest.setCategoryId(categoryId);
        searchRequest.setDescription(description);

        Sort.Direction direction = sortDir.equalsIgnoreCase("asc") ? Sort.Direction.ASC : Sort.Direction.DESC;
        Pageable pageable = PageRequest.of(page, size, Sort.by(direction, sortBy));

        Page<Expense> expenses = expenseService.searchExpenses(searchRequest, pageable);
        return ResponseEntity.ok(expenses);
    }

    // NEW: Monthly summary
    @GetMapping("/summary/monthly")
    public ResponseEntity<Map<String, Object>> getMonthlySummary(
            @RequestParam int year) {
        Map<String, Object> summary = expenseService.getMonthlySummary(year);
        return ResponseEntity.ok(summary);
    }

    // NEW: Category summary
    @GetMapping("/summary/category")
    public ResponseEntity<List<Map<String, Object>>> getCategorySummary(
            @RequestParam @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate startDate,
            @RequestParam @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate endDate) {
        List<Map<String, Object>> summary = expenseService.getCategorySummary(startDate, endDate);
        return ResponseEntity.ok(summary);
    }
}
```

---

## Step 7: Test Advanced Features

### Test 1: Advanced Search

```bash
GET /api/expenses/search?
    startDate=2024-01-01&
    endDate=2024-12-31&
    minAmount=10&
    maxAmount=100&
    categoryId=1&
    description=lunch&
    page=0&
    size=10
```

### Test 2: Monthly Summary

```bash
GET /api/expenses/summary/monthly?year=2024
```

Response:
```json
{
  "year": 2024,
  "months": [
    {
      "month": 1,
      "monthName": "January",
      "total": 450.50,
      "count": 15
    },
    ...
  ],
  "yearTotal": 5420.75
}
```

### Test 3: Category Summary

```bash
GET /api/expenses/summary/category?startDate=2024-01-01&endDate=2024-12-31
```

---

## Common MySQL Issues

### Issue 1: Access Denied

**Error**: `Access denied for user 'expenseapp'@'localhost'`

**Solution**:
- Check username/password in `application.properties`
- Verify user exists: `SELECT User FROM mysql.user;`
- Grant privileges again

### Issue 2: Database Not Found

**Error**: `Unknown database 'expense_tracker'`

**Solution**:
- Create database: `CREATE DATABASE expense_tracker;`

### Issue 3: Connection Timeout

**Error**: `Communications link failure`

**Solution**:
- Check MySQL is running: `sudo service mysql status`
- Verify port 3306 is open
- Check firewall settings

---

## Key Takeaways

✅ **MySQL**: Production-ready relational database
✅ **Data Persistence**: Survives application restarts
✅ **Specification Pattern**: Dynamic query building
✅ **JpaSpecificationExecutor**: Advanced filtering
✅ **JPQL Queries**: Custom queries with aggregations
✅ **Monthly/Category Summaries**: Business intelligence

---

**Next**: [Phase 7: JWT Authentication & Security](PHASE_7.md)
