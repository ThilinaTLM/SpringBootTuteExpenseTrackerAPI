# Phase 4: Validation & Error Handling

**Duration**: 3-4 hours
**Complexity**: ⭐⭐⭐ Intermediate

## Learning Objectives

By the end of this phase, you will:
- Understand why validation is critical
- Implement Bean Validation (JSR-303)
- Create custom exceptions
- Build a global exception handler
- Return proper error responses
- Master HTTP status codes

## Prerequisites

- ✅ Completed Phase 3
- ✅ Understanding of JPA and repositories
- ✅ Basic CRUD operations working

---

## Why Validation Matters

### Without Validation

Imagine a user sends:
```json
{
  "description": "",
  "amount": -50.00,
  "date": "2025-12-31"
}
```

**Problems**:
- Empty description (meaningless expense)
- Negative amount (impossible)
- Future date (how can you spend money in the future?)

**Without validation**, this gets saved to database → Bad data → Application bugs

### With Validation

Reject invalid data immediately:
- Prevent bad data from entering database
- Provide helpful error messages
- Improve user experience
- Maintain data integrity

---

## Step 1: Add Validation Dependency

### Update pom.xml

Add inside `<dependencies>`:

```xml
<!-- Validation -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

Refresh Maven.

---

## Step 2: Add Validation Annotations to Expense

### Update Expense.java

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

    // Constructors, getters, setters (same as before)
    public Expense() {
    }

    public Expense(String description, Double amount, LocalDate date) {
        this.description = description;
        this.amount = amount;
        this.date = date;
    }

    // Getters and setters...
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

### Understanding Validation Annotations

**@NotBlank**:
- Must not be null or empty
- Trims whitespace
- Use for Strings that require content

**@NotNull**:
- Must not be null
- Use for numbers, dates, objects

**@Size(min, max)**:
- String length must be within range
- Example: `@Size(min=3, max=100)`

**@Positive**:
- Number must be greater than 0
- For amounts, quantities, etc.

**@PastOrPresent**:
- Date must be today or in the past
- Prevents future dates

**Other Common Annotations**:
- `@Email`: Valid email format
- `@Min(value)`: Minimum value
- `@Max(value)`: Maximum value
- `@Pattern(regex)`: Must match regex
- `@Future`: Must be in future
- `@Past`: Must be in past

---

## Step 3: Enable Validation in Controller

### Update ExpenseController.java

Add `@Valid` annotation to request bodies:

```java
package com.example.expensetracker.controller;

import com.example.expensetracker.model.Expense;
import com.example.expensetracker.service.ExpenseService;
import jakarta.validation.Valid;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;
import java.util.Optional;

@RestController
@RequestMapping("/api/expenses")
public class ExpenseController {

    private final ExpenseService expenseService;

    @Autowired
    public ExpenseController(ExpenseService expenseService) {
        this.expenseService = expenseService;
    }

    // CREATE - Note the @Valid annotation
    @PostMapping
    public ResponseEntity<Expense> createExpense(@Valid @RequestBody Expense expense) {
        Expense createdExpense = expenseService.createExpense(expense);
        return new ResponseEntity<>(createdExpense, HttpStatus.CREATED);
    }

    // READ - Get all
    @GetMapping
    public ResponseEntity<List<Expense>> getAllExpenses() {
        List<Expense> expenses = expenseService.getAllExpenses();
        return ResponseEntity.ok(expenses);
    }

    // READ - Get by ID
    @GetMapping("/{id}")
    public ResponseEntity<Expense> getExpenseById(@PathVariable Long id) {
        Optional<Expense> expense = expenseService.getExpenseById(id);
        return expense.map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    // UPDATE - Note the @Valid annotation
    @PutMapping("/{id}")
    public ResponseEntity<Expense> updateExpense(
            @PathVariable Long id,
            @Valid @RequestBody Expense expense) {

        Optional<Expense> updatedExpense = expenseService.updateExpense(id, expense);
        return updatedExpense.map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    // DELETE
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteExpense(@PathVariable Long id) {
        boolean deleted = expenseService.deleteExpense(id);
        return deleted
                ? ResponseEntity.noContent().build()
                : ResponseEntity.notFound().build();
    }
}
```

**@Valid annotation**:
- Triggers validation
- Spring checks all validation annotations
- If validation fails, throws `MethodArgumentNotValidException`

---

## Step 4: Create Custom Exceptions

Create package: `com.example.expensetracker.exception`

### Create ResourceNotFoundException.java

```java
package com.example.expensetracker.exception;

public class ResourceNotFoundException extends RuntimeException {

    public ResourceNotFoundException(String message) {
        super(message);
    }

    public ResourceNotFoundException(String resourceName, String fieldName, Object fieldValue) {
        super(String.format("%s not found with %s: '%s'", resourceName, fieldName, fieldValue));
    }
}
```

### Create InvalidDataException.java

```java
package com.example.expensetracker.exception;

public class InvalidDataException extends RuntimeException {

    public InvalidDataException(String message) {
        super(message);
    }
}
```

---

## Step 5: Create Error Response DTO

Create `ErrorResponse.java` in the `dto` package:

```java
package com.example.expensetracker.dto;

import java.time.LocalDateTime;
import java.util.Map;

public class ErrorResponse {

    private LocalDateTime timestamp;
    private int status;
    private String error;
    private String message;
    private String path;
    private Map<String, String> validationErrors;

    public ErrorResponse() {
        this.timestamp = LocalDateTime.now();
    }

    public ErrorResponse(int status, String error, String message, String path) {
        this.timestamp = LocalDateTime.now();
        this.status = status;
        this.error = error;
        this.message = message;
        this.path = path;
    }

    // Getters and Setters
    public LocalDateTime getTimestamp() {
        return timestamp;
    }

    public void setTimestamp(LocalDateTime timestamp) {
        this.timestamp = timestamp;
    }

    public int getStatus() {
        return status;
    }

    public void setStatus(int status) {
        this.status = status;
    }

    public String getError() {
        return error;
    }

    public void setError(String error) {
        this.error = error;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    public String getPath() {
        return path;
    }

    public void setPath(String path) {
        this.path = path;
    }

    public Map<String, String> getValidationErrors() {
        return validationErrors;
    }

    public void setValidationErrors(Map<String, String> validationErrors) {
        this.validationErrors = validationErrors;
    }
}
```

---

## Step 6: Create Global Exception Handler

Create `GlobalExceptionHandler.java` in the `exception` package:

```java
package com.example.expensetracker.exception;

import com.example.expensetracker.dto.ErrorResponse;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.FieldError;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;
import org.springframework.web.context.request.WebRequest;

import java.util.HashMap;
import java.util.Map;

@RestControllerAdvice
public class GlobalExceptionHandler {

    // Handle validation errors
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidationExceptions(
            MethodArgumentNotValidException ex,
            WebRequest request) {

        Map<String, String> errors = new HashMap<>();

        ex.getBindingResult().getAllErrors().forEach((error) -> {
            String fieldName = ((FieldError) error).getField();
            String errorMessage = error.getDefaultMessage();
            errors.put(fieldName, errorMessage);
        });

        ErrorResponse errorResponse = new ErrorResponse(
                HttpStatus.BAD_REQUEST.value(),
                "Validation Failed",
                "Input validation failed",
                request.getDescription(false).replace("uri=", "")
        );
        errorResponse.setValidationErrors(errors);

        return new ResponseEntity<>(errorResponse, HttpStatus.BAD_REQUEST);
    }

    // Handle resource not found
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFoundException(
            ResourceNotFoundException ex,
            WebRequest request) {

        ErrorResponse errorResponse = new ErrorResponse(
                HttpStatus.NOT_FOUND.value(),
                "Not Found",
                ex.getMessage(),
                request.getDescription(false).replace("uri=", "")
        );

        return new ResponseEntity<>(errorResponse, HttpStatus.NOT_FOUND);
    }

    // Handle invalid data
    @ExceptionHandler(InvalidDataException.class)
    public ResponseEntity<ErrorResponse> handleInvalidDataException(
            InvalidDataException ex,
            WebRequest request) {

        ErrorResponse errorResponse = new ErrorResponse(
                HttpStatus.BAD_REQUEST.value(),
                "Bad Request",
                ex.getMessage(),
                request.getDescription(false).replace("uri=", "")
        );

        return new ResponseEntity<>(errorResponse, HttpStatus.BAD_REQUEST);
    }

    // Handle all other exceptions
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGlobalException(
            Exception ex,
            WebRequest request) {

        ErrorResponse errorResponse = new ErrorResponse(
                HttpStatus.INTERNAL_SERVER_ERROR.value(),
                "Internal Server Error",
                "An unexpected error occurred: " + ex.getMessage(),
                request.getDescription(false).replace("uri=", "")
        );

        return new ResponseEntity<>(errorResponse, HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```

### Understanding @RestControllerAdvice

**@RestControllerAdvice**:
- Global exception handler
- Applies to all controllers
- Catches exceptions and converts to proper responses

**@ExceptionHandler**:
- Handles specific exception types
- Returns ResponseEntity with error details

**How it works**:
1. Exception thrown in controller
2. Spring catches it
3. Looks for matching @ExceptionHandler
4. Executes handler method
5. Returns error response to client

---

## Step 7: Update Service to Throw Custom Exceptions

### Update ExpenseService.java

```java
package com.example.expensetracker.service;

import com.example.expensetracker.exception.ResourceNotFoundException;
import com.example.expensetracker.model.Expense;
import com.example.expensetracker.repository.ExpenseRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.time.LocalDate;
import java.util.List;

@Service
public class ExpenseService {

    private final ExpenseRepository expenseRepository;

    @Autowired
    public ExpenseService(ExpenseRepository expenseRepository) {
        this.expenseRepository = expenseRepository;
    }

    public Expense createExpense(Expense expense) {
        return expenseRepository.save(expense);
    }

    public List<Expense> getAllExpenses() {
        return expenseRepository.findAll();
    }

    public Expense getExpenseById(Long id) {
        return expenseRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Expense", "id", id));
    }

    public Expense updateExpense(Long id, Expense updatedExpense) {
        Expense expense = getExpenseById(id); // Throws if not found

        expense.setDescription(updatedExpense.getDescription());
        expense.setAmount(updatedExpense.getAmount());
        expense.setDate(updatedExpense.getDate());

        return expenseRepository.save(expense);
    }

    public void deleteExpense(Long id) {
        Expense expense = getExpenseById(id); // Throws if not found
        expenseRepository.delete(expense);
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

### Update ExpenseController.java

Simplify controller since service now handles exceptions:

```java
@RestController
@RequestMapping("/api/expenses")
public class ExpenseController {

    private final ExpenseService expenseService;

    @Autowired
    public ExpenseController(ExpenseService expenseService) {
        this.expenseService = expenseService;
    }

    @PostMapping
    public ResponseEntity<Expense> createExpense(@Valid @RequestBody Expense expense) {
        Expense createdExpense = expenseService.createExpense(expense);
        return new ResponseEntity<>(createdExpense, HttpStatus.CREATED);
    }

    @GetMapping
    public ResponseEntity<List<Expense>> getAllExpenses() {
        List<Expense> expenses = expenseService.getAllExpenses();
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
            @Valid @RequestBody Expense expense) {
        Expense updatedExpense = expenseService.updateExpense(id, expense);
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

## Step 8: Test Validation

### Test 1: Empty Description

**Request**:
```json
POST /api/expenses
{
  "description": "",
  "amount": 50.00,
  "date": "2024-10-20"
}
```

**Response** (400 Bad Request):
```json
{
  "timestamp": "2024-10-20T10:30:00",
  "status": 400,
  "error": "Validation Failed",
  "message": "Input validation failed",
  "path": "/api/expenses",
  "validationErrors": {
    "description": "Description is required"
  }
}
```

### Test 2: Negative Amount

**Request**:
```json
{
  "description": "Test",
  "amount": -50.00,
  "date": "2024-10-20"
}
```

**Response** (400 Bad Request):
```json
{
  "validationErrors": {
    "amount": "Amount must be positive"
  }
}
```

### Test 3: Future Date

**Request**:
```json
{
  "description": "Test",
  "amount": 50.00,
  "date": "2025-12-31"
}
```

**Response** (400 Bad Request):
```json
{
  "validationErrors": {
    "date": "Date cannot be in the future"
  }
}
```

### Test 4: Multiple Validation Errors

**Request**:
```json
{
  "description": "",
  "amount": -50.00,
  "date": "2025-12-31"
}
```

**Response** (400 Bad Request):
```json
{
  "validationErrors": {
    "description": "Description is required",
    "amount": "Amount must be positive",
    "date": "Date cannot be in the future"
  }
}
```

### Test 5: Resource Not Found

**Request**: `GET /api/expenses/999`

**Response** (404 Not Found):
```json
{
  "timestamp": "2024-10-20T10:30:00",
  "status": 404,
  "error": "Not Found",
  "message": "Expense not found with id: '999'",
  "path": "/api/expenses/999"
}
```

---

## HTTP Status Codes Reference

### Success Codes (2xx)
- **200 OK**: Request successful
- **201 Created**: Resource created
- **204 No Content**: Successful but no data to return

### Client Error Codes (4xx)
- **400 Bad Request**: Invalid input, validation failed
- **401 Unauthorized**: Not authenticated
- **403 Forbidden**: Authenticated but not authorized
- **404 Not Found**: Resource doesn't exist
- **409 Conflict**: Resource already exists

### Server Error Codes (5xx)
- **500 Internal Server Error**: Unexpected server error
- **503 Service Unavailable**: Server temporarily unavailable

---

## Checkpoint Exercises

### Exercise 1: Add Amount Range Validation

Limit expense amounts between $0.01 and $10,000.

<details>
<summary>Show Solution</summary>

```java
@DecimalMin(value = "0.01", message = "Amount must be at least $0.01")
@DecimalMax(value = "10000.00", message = "Amount cannot exceed $10,000")
private Double amount;
```
</details>

### Exercise 2: Custom Validation for Description

Description should not contain special characters like `<`, `>`, `&`.

<details>
<summary>Show Solution</summary>

```java
@Pattern(
    regexp = "^[a-zA-Z0-9\\s,.!?-]+$",
    message = "Description contains invalid characters"
)
private String description;
```
</details>

### Exercise 3: Add Category Field

Add optional category field (max 50 characters).

<details>
<summary>Show Solution</summary>

```java
@Size(max = 50, message = "Category cannot exceed 50 characters")
@Column(length = 50)
private String category;
```
</details>

---

## Key Takeaways

✅ **Bean Validation**: Use annotations to validate input
✅ **@Valid**: Triggers validation in controllers
✅ **Custom Exceptions**: Create meaningful exception classes
✅ **@RestControllerAdvice**: Global exception handler
✅ **@ExceptionHandler**: Handles specific exceptions
✅ **ErrorResponse**: Standardized error format
✅ **HTTP Status Codes**: Use appropriate codes for different scenarios

---

## What's Next?

In **Phase 5**, you'll learn:
- Entity relationships (One-to-Many)
- Category management
- Pagination and sorting
- Advanced JPA queries
- JPQL (Java Persistence Query Language)

---

**Next**: [Phase 5: Categories & Pagination](PHASE_5.md)
