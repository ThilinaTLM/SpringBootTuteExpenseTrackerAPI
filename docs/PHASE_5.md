# Phase 5: Categories & Pagination

**Duration**: 5-6 hours
**Complexity**: ⭐⭐⭐ Intermediate

## Learning Objectives

By the end of this phase, you will:
- Understand JPA entity relationships
- Implement One-to-Many relationships
- Create and manage categories
- Add pagination support
- Implement sorting
- Write custom JPQL queries
- Generate category statistics

## Prerequisites

- ✅ Completed Phase 4
- ✅ Understanding of validation
- ✅ Exception handling in place

---

## Why Categories?

**Problem**: All expenses in one big list
- Hard to organize
- Can't group by type
- No spending insights by category

**Solution**: Categories
- Group related expenses (Food, Transport, etc.)
- Track spending by category
- Better organization and insights

---

## Entity Relationships Explained

### One-to-Many Relationship

**One Category** can have **Many Expenses**

Example:
- Category: "Food & Dining"
  - Expense: "Grocery shopping" - $45.50
  - Expense: "Restaurant lunch" - $25.00
  - Expense: "Coffee" - $5.50

### Visual Representation

```
Category (One)           Expenses (Many)
┌─────────────┐         ┌──────────────────┐
│ ID: 1       │◄────────│ ID: 1            │
│ Name: Food  │         │ Description: ... │
└─────────────┘         │ Category_ID: 1   │
                        └──────────────────┘
                        ┌──────────────────┐
                        │ ID: 2            │
                        │ Description: ... │
                        │ Category_ID: 1   │
                        └──────────────────┘
```

---

## Step 1: Create Category Entity

Create `Category.java` in the `model` package:

```java
package com.example.expensetracker.model;

import com.fasterxml.jackson.annotation.JsonIgnore;
import jakarta.persistence.*;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;
import java.util.ArrayList;
import java.util.List;

@Entity
@Table(name = "categories")
public class Category {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotBlank(message = "Category name is required")
    @Size(min = 2, max = 50, message = "Category name must be between 2 and 50 characters")
    @Column(nullable = false, unique = true, length = 50)
    private String name;

    @Size(max = 255, message = "Description cannot exceed 255 characters")
    @Column(length = 255)
    private String description;

    @OneToMany(mappedBy = "category", cascade = CascadeType.ALL, orphanRemoval = true)
    @JsonIgnore // Prevent infinite recursion in JSON
    private List<Expense> expenses = new ArrayList<>();

    // Constructors
    public Category() {
    }

    public Category(String name, String description) {
        this.name = name;
        this.description = description;
    }

    // Getters and Setters
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public List<Expense> getExpenses() {
        return expenses;
    }

    public void setExpenses(List<Expense> expenses) {
        this.expenses = expenses;
    }

    // Helper methods
    public void addExpense(Expense expense) {
        expenses.add(expense);
        expense.setCategory(this);
    }

    public void removeExpense(Expense expense) {
        expenses.remove(expense);
        expense.setCategory(null);
    }

    @Override
    public String toString() {
        return "Category{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", description='" + description + '\'' +
                '}';
    }
}
```

### Understanding Annotations

**@OneToMany**:
- One category has many expenses
- `mappedBy = "category"`: Links to `category` field in Expense
- `cascade = CascadeType.ALL`: Operations cascade to children
- `orphanRemoval = true`: Delete expenses when removed from list

**@JsonIgnore**:
- Prevents infinite loop when converting to JSON
- Without it: Category → Expenses → Category → Expenses → ...

**unique = true**:
- Category name must be unique
- Database enforces this constraint

---

## Step 2: Update Expense Entity

### Modify Expense.java

Add the category relationship:

```java
package com.example.expensetracker.model;

import jakarta.persistence.*;
import jakarta.validation.constraints.*;
import java.time.LocalDate;

@Entity
@Table(name = "expenses")
public class Expense {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotBlank(message = "Description is required")
    @Size(min = 3, max = 255, message = "Description must be between 3 and 255 characters")
    @Column(nullable = false)
    private String description;

    @NotNull(message = "Amount is required")
    @Positive(message = "Amount must be positive")
    @Column(nullable = false)
    private Double amount;

    @NotNull(message = "Date is required")
    @PastOrPresent(message = "Date cannot be in the future")
    @Column(nullable = false)
    private LocalDate date;

    // NEW: Many expenses can belong to one category
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "category_id")
    private Category category;

    // Constructors
    public Expense() {
    }

    public Expense(String description, Double amount, LocalDate date) {
        this.description = description;
        this.amount = amount;
        this.date = date;
    }

    // Getters and Setters (including new category field)
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

    // NEW: Category getter and setter
    public Category getCategory() {
        return category;
    }

    public void setCategory(Category category) {
        this.category = category;
    }

    @Override
    public String toString() {
        return "Expense{" +
                "id=" + id +
                ", description='" + description + '\'' +
                ", amount=" + amount +
                ", date=" + date +
                ", category=" + (category != null ? category.getName() : "None") +
                '}';
    }
}
```

### Understanding @ManyToOne

**@ManyToOne**:
- Many expenses belong to one category
- Opposite side of @OneToMany

**@JoinColumn(name = "category_id")**:
- Creates foreign key column in expenses table
- Links to category's ID

**FetchType.LAZY**:
- Category loaded only when accessed
- Improves performance
- Alternative: EAGER (loads immediately)

---

## Step 3: Create DTOs

Create DTOs for better API responses.

### Create ExpenseDTO.java

```java
package com.example.expensetracker.dto;

import java.time.LocalDate;

public class ExpenseDTO {
    private Long id;
    private String description;
    private Double amount;
    private LocalDate date;
    private Long categoryId;
    private String categoryName;

    // Constructors
    public ExpenseDTO() {
    }

    // Getters and Setters
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

    public Long getCategoryId() {
        return categoryId;
    }

    public void setCategoryId(Long categoryId) {
        this.categoryId = categoryId;
    }

    public String getCategoryName() {
        return categoryName;
    }

    public void setCategoryName(String categoryName) {
        this.categoryName = categoryName;
    }
}
```

### Create CategoryDTO.java

```java
package com.example.expensetracker.dto;

public class CategoryDTO {
    private Long id;
    private String name;
    private String description;
    private Integer expenseCount;
    private Double totalAmount;

    // Constructors
    public CategoryDTO() {
    }

    // Getters and Setters
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public Integer getExpenseCount() {
        return expenseCount;
    }

    public void setExpenseCount(Integer expenseCount) {
        this.expenseCount = expenseCount;
    }

    public Double getTotalAmount() {
        return totalAmount;
    }

    public void setTotalAmount(Double totalAmount) {
        this.totalAmount = totalAmount;
    }
}
```

---

## Step 4: Create Repositories

### Create CategoryRepository.java

```java
package com.example.expensetracker.repository;

import com.example.expensetracker.model.Category;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.stereotype.Repository;

import java.util.Optional;

@Repository
public interface CategoryRepository extends JpaRepository<Category, Long> {

    Optional<Category> findByName(String name);

    boolean existsByName(String name);

    @Query("SELECT COUNT(e) FROM Expense e WHERE e.category.id = :categoryId")
    Long countExpensesByCategoryId(Long categoryId);

    @Query("SELECT SUM(e.amount) FROM Expense e WHERE e.category.id = :categoryId")
    Double sumAmountByCategoryId(Long categoryId);
}
```

### Update ExpenseRepository.java

Add category-related queries:

```java
package com.example.expensetracker.repository;

import com.example.expensetracker.model.Expense;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.stereotype.Repository;

import java.time.LocalDate;
import java.util.List;

@Repository
public interface ExpenseRepository extends JpaRepository<Expense, Long> {

    // Existing methods
    List<Expense> findByDate(LocalDate date);

    List<Expense> findByDateBetween(LocalDate startDate, LocalDate endDate);

    List<Expense> findByDescriptionContainingIgnoreCase(String keyword);

    // NEW: Category-related queries
    Page<Expense> findByCategoryId(Long categoryId, Pageable pageable);

    List<Expense> findByCategoryId(Long categoryId);

    // NEW: Pagination support
    Page<Expense> findByDescriptionContainingIgnoreCase(String keyword, Pageable pageable);

    Page<Expense> findByDateBetween(LocalDate startDate, LocalDate endDate, Pageable pageable);

    // NEW: JPQL query
    @Query("SELECT e FROM Expense e WHERE e.category.id = :categoryId AND e.date BETWEEN :startDate AND :endDate")
    List<Expense> findByCategoryAndDateRange(Long categoryId, LocalDate startDate, LocalDate endDate);
}
```

---

## Step 5: Create Services

### Create CategoryService.java

```java
package com.example.expensetracker.service;

import com.example.expensetracker.dto.CategoryDTO;
import com.example.expensetracker.exception.InvalidDataException;
import com.example.expensetracker.exception.ResourceNotFoundException;
import com.example.expensetracker.model.Category;
import com.example.expensetracker.repository.CategoryRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.stream.Collectors;

@Service
public class CategoryService {

    private final CategoryRepository categoryRepository;

    @Autowired
    public CategoryService(CategoryRepository categoryRepository) {
        this.categoryRepository = categoryRepository;
    }

    public Category createCategory(Category category) {
        if (categoryRepository.existsByName(category.getName())) {
            throw new InvalidDataException("Category with name '" + category.getName() + "' already exists");
        }
        return categoryRepository.save(category);
    }

    public List<CategoryDTO> getAllCategories() {
        return categoryRepository.findAll().stream()
                .map(this::convertToDTO)
                .collect(Collectors.toList());
    }

    public Category getCategoryById(Long id) {
        return categoryRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Category", "id", id));
    }

    public Category updateCategory(Long id, Category updatedCategory) {
        Category category = getCategoryById(id);

        // Check if new name conflicts with another category
        if (!category.getName().equals(updatedCategory.getName()) &&
                categoryRepository.existsByName(updatedCategory.getName())) {
            throw new InvalidDataException("Category with name '" + updatedCategory.getName() + "' already exists");
        }

        category.setName(updatedCategory.getName());
        category.setDescription(updatedCategory.getDescription());

        return categoryRepository.save(category);
    }

    public void deleteCategory(Long id) {
        Category category = getCategoryById(id);

        Long expenseCount = categoryRepository.countExpensesByCategoryId(id);
        if (expenseCount > 0) {
            throw new InvalidDataException("Cannot delete category with existing expenses. " +
                    "Please delete or reassign " + expenseCount + " expense(s) first.");
        }

        categoryRepository.delete(category);
    }

    // Convert entity to DTO
    private CategoryDTO convertToDTO(Category category) {
        CategoryDTO dto = new CategoryDTO();
        dto.setId(category.getId());
        dto.setName(category.getName());
        dto.setDescription(category.getDescription());

        // Calculate statistics
        Long count = categoryRepository.countExpensesByCategoryId(category.getId());
        Double total = categoryRepository.sumAmountByCategoryId(category.getId());

        dto.setExpenseCount(count != null ? count.intValue() : 0);
        dto.setTotalAmount(total != null ? total : 0.0);

        return dto;
    }
}
```

### Update ExpenseService.java

Add pagination and category support:

```java
package com.example.expensetracker.service;

import com.example.expensetracker.exception.ResourceNotFoundException;
import com.example.expensetracker.model.Category;
import com.example.expensetracker.model.Expense;
import com.example.expensetracker.repository.ExpenseRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.stereotype.Service;

import java.time.LocalDate;
import java.util.List;

@Service
public class ExpenseService {

    private final ExpenseRepository expenseRepository;
    private final CategoryService categoryService;

    @Autowired
    public ExpenseService(ExpenseRepository expenseRepository, CategoryService categoryService) {
        this.expenseRepository = expenseRepository;
        this.categoryService = categoryService;
    }

    public Expense createExpense(Expense expense, Long categoryId) {
        if (categoryId != null) {
            Category category = categoryService.getCategoryById(categoryId);
            expense.setCategory(category);
        }
        return expenseRepository.save(expense);
    }

    // NEW: Paginated get all
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

    // NEW: Get expenses by category with pagination
    public Page<Expense> getExpensesByCategory(Long categoryId, Pageable pageable) {
        categoryService.getCategoryById(categoryId); // Verify category exists
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

---

## Step 6: Create Controllers

### Create CategoryController.java

```java
package com.example.expensetracker.controller;

import com.example.expensetracker.dto.CategoryDTO;
import com.example.expensetracker.model.Category;
import com.example.expensetracker.model.Expense;
import com.example.expensetracker.service.CategoryService;
import com.example.expensetracker.service.ExpenseService;
import jakarta.validation.Valid;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/categories")
public class CategoryController {

    private final CategoryService categoryService;
    private final ExpenseService expenseService;

    @Autowired
    public CategoryController(CategoryService categoryService, ExpenseService expenseService) {
        this.categoryService = categoryService;
        this.expenseService = expenseService;
    }

    @PostMapping
    public ResponseEntity<Category> createCategory(@Valid @RequestBody Category category) {
        Category createdCategory = categoryService.createCategory(category);
        return new ResponseEntity<>(createdCategory, HttpStatus.CREATED);
    }

    @GetMapping
    public ResponseEntity<List<CategoryDTO>> getAllCategories() {
        List<CategoryDTO> categories = categoryService.getAllCategories();
        return ResponseEntity.ok(categories);
    }

    @GetMapping("/{id}")
    public ResponseEntity<Category> getCategoryById(@PathVariable Long id) {
        Category category = categoryService.getCategoryById(id);
        return ResponseEntity.ok(category);
    }

    @PutMapping("/{id}")
    public ResponseEntity<Category> updateCategory(
            @PathVariable Long id,
            @Valid @RequestBody Category category) {
        Category updatedCategory = categoryService.updateCategory(id, category);
        return ResponseEntity.ok(updatedCategory);
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteCategory(@PathVariable Long id) {
        categoryService.deleteCategory(id);
        return ResponseEntity.noContent().build();
    }

    @GetMapping("/{id}/expenses")
    public ResponseEntity<Page<Expense>> getExpensesByCategory(
            @PathVariable Long id,
            Pageable pageable) {
        Page<Expense> expenses = expenseService.getExpensesByCategory(id, pageable);
        return ResponseEntity.ok(expenses);
    }
}
```

### Update ExpenseController.java

Add pagination and category parameter:

```java
package com.example.expensetracker.controller;

import com.example.expensetracker.model.Expense;
import com.example.expensetracker.service.ExpenseService;
import jakarta.validation.Valid;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/expenses")
public class ExpenseController {

    private final ExpenseService expenseService;

    @Autowired
    public ExpenseController(ExpenseService expenseService) {
        this.expenseService = expenseService;
    }

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
}
```

---

## Step 7: Test the API

### Test 1: Create Categories

```bash
POST /api/categories
{
  "name": "Food & Dining",
  "description": "Restaurants, groceries, coffee"
}

POST /api/categories
{
  "name": "Transportation",
  "description": "Gas, public transport, parking"
}

POST /api/categories
{
  "name": "Entertainment",
  "description": "Movies, games, hobbies"
}
```

### Test 2: Get All Categories

```bash
GET /api/categories
```

Response shows expense count and total amount per category.

### Test 3: Create Expense with Category

```bash
POST /api/expenses?categoryId=1
{
  "description": "Grocery shopping",
  "amount": 45.50,
  "date": "2024-10-20"
}
```

### Test 4: Get Expenses with Pagination

```bash
GET /api/expenses?page=0&size=10&sortBy=amount&sortDir=desc
```

### Test 5: Get Expenses by Category

```bash
GET /api/categories/1/expenses?page=0&size=10
```

---

## Understanding Pagination

**Pageable** parameters:
- `page`: Page number (0-indexed)
- `size`: Items per page
- `sortBy`: Field to sort by
- `sortDir`: Sort direction (asc/desc)

**Page response** contains:
- `content`: Array of items
- `totalElements`: Total count
- `totalPages`: Number of pages
- `number`: Current page
- `size`: Page size
- `first`: Is first page?
- `last`: Is last page?

---

## Key Takeaways

✅ **@OneToMany / @ManyToOne**: Entity relationships
✅ **@JoinColumn**: Foreign key configuration
✅ **Pageable**: Built-in pagination support
✅ **Page<T>**: Paginated response type
✅ **JPQL**: Custom queries using Java entities
✅ **DTO Pattern**: Separate API responses from entities

---

**Next**: [Phase 6: MySQL Migration & Advanced Search](PHASE_6.md)
