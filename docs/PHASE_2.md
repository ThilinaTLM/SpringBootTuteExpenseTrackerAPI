# Phase 2: In-Memory CRUD Operations

**Duration**: 3-4 hours
**Complexity**: ⭐⭐ Beginner

## Learning Objectives

By the end of this phase, you will:
- Understand HTTP methods (GET, POST, PUT, DELETE)
- Know what CRUD means and why it's important
- Implement create, read, update, and delete operations
- Learn the Controller-Service pattern
- Work with request bodies and path variables
- Understand HTTP status codes

## Prerequisites

- ✅ Completed Phase 1
- ✅ Spring Boot application running
- ✅ Postman installed (for testing)

---

## What is CRUD?

**CRUD** = **C**reate, **R**ead, **U**pdate, **D**elete

These are the four basic operations you can do with data:
- **Create**: Add new data (like adding a new expense)
- **Read**: View data (like viewing your expenses)
- **Update**: Modify existing data (like editing an expense amount)
- **Delete**: Remove data (like deleting an expense)

**Real-world example**: Your phone contacts app
- **Create**: Add a new contact
- **Read**: View your contacts list
- **Update**: Change someone's phone number
- **Delete**: Remove a contact

---

## HTTP Methods Explained

### The Four Main Methods

| Method | CRUD | Purpose | Example |
|--------|------|---------|---------|
| **POST** | Create | Add new data | Add new expense |
| **GET** | Read | Retrieve data | View expenses |
| **PUT** | Update | Modify data | Edit expense |
| **DELETE** | Delete | Remove data | Delete expense |

### Restaurant Analogy (Revisited)

- **POST** (Create): "I'll have a burger" - You're ordering (creating) a new order
- **GET** (Read): "What's on my bill?" - You're asking to see (read) your order
- **PUT** (Update): "Change my burger to well-done" - You're modifying your order
- **DELETE** (Delete): "Cancel the fries" - You're removing an item

---

## The Expense Model

First, let's define what an "expense" looks like.

### Step 1: Create the Model Package

Create a new package: `com.example.expensetracker.model`

### Step 2: Create the Expense Class

Create `Expense.java` in the `model` package:

```java
package com.example.expensetracker.model;

import java.time.LocalDate;

public class Expense {
    private Long id;
    private String description;
    private Double amount;
    private LocalDate date;

    // Default constructor (required for JSON conversion)
    public Expense() {
    }

    // Constructor with parameters
    public Expense(Long id, String description, Double amount, LocalDate date) {
        this.id = id;
        this.description = description;
        this.amount = amount;
        this.date = date;
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

    // toString for debugging
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

### Understanding the Model

**Fields**:
- `id`: Unique identifier for each expense
- `description`: What the expense was for
- `amount`: How much it cost
- `date`: When it happened

**Why getters/setters?**
- Java convention for accessing private fields
- Allows controlled access to data
- Required for JSON conversion

**Pro tip**: Most IDEs can generate getters/setters automatically:
- IntelliJ: Alt+Insert (Windows) or Cmd+N (Mac) → Getters and Setters
- Eclipse: Right-click → Source → Generate Getters and Setters

---

## The Service Layer

The **Service** layer contains business logic. It's the "brain" of your application.

### Why Separate Controller and Service?

**Bad approach** (everything in controller):
```
Controller: Receive request → Process data → Store data → Send response
```

**Good approach** (separation of concerns):
```
Controller: Receive request → Call service → Send response
Service: Process data → Store data → Return result
```

**Benefits**:
- ✅ Easier to test
- ✅ Reusable logic
- ✅ Cleaner code
- ✅ Easier to maintain

### Step 1: Create Service Package

Create package: `com.example.expensetracker.service`

### Step 2: Create ExpenseService Class

Create `ExpenseService.java`:

```java
package com.example.expensetracker.service;

import com.example.expensetracker.model.Expense;
import org.springframework.stereotype.Service;

import java.util.ArrayList;
import java.util.List;
import java.util.Optional;
import java.util.concurrent.atomic.AtomicLong;

@Service
public class ExpenseService {

    // In-memory storage (temporary, will be replaced with database in Phase 3)
    private final List<Expense> expenses = new ArrayList<>();

    // ID generator (simulates database auto-increment)
    private final AtomicLong idCounter = new AtomicLong(1);

    // Create expense
    public Expense createExpense(Expense expense) {
        expense.setId(idCounter.getAndIncrement());
        expenses.add(expense);
        return expense;
    }

    // Get all expenses
    public List<Expense> getAllExpenses() {
        return new ArrayList<>(expenses); // Return copy to prevent modification
    }

    // Get expense by ID
    public Optional<Expense> getExpenseById(Long id) {
        return expenses.stream()
                .filter(expense -> expense.getId().equals(id))
                .findFirst();
    }

    // Update expense
    public Optional<Expense> updateExpense(Long id, Expense updatedExpense) {
        Optional<Expense> existingExpense = getExpenseById(id);

        if (existingExpense.isPresent()) {
            Expense expense = existingExpense.get();
            expense.setDescription(updatedExpense.getDescription());
            expense.setAmount(updatedExpense.getAmount());
            expense.setDate(updatedExpense.getDate());
            return Optional.of(expense);
        }

        return Optional.empty();
    }

    // Delete expense
    public boolean deleteExpense(Long id) {
        return expenses.removeIf(expense -> expense.getId().equals(id));
    }
}
```

### Understanding the Service

**@Service annotation**:
- Marks this class as a service component
- Spring automatically creates an instance
- Allows injection into controllers

**In-memory storage**:
- `ArrayList<Expense>`: Stores expenses temporarily
- Data is lost when application restarts
- Will be replaced with database in Phase 3

**AtomicLong for IDs**:
- Thread-safe counter
- Simulates auto-incrementing database IDs
- Starts at 1, increments for each new expense

**Optional<T>**:
- Java class that may or may not contain a value
- Prevents null pointer exceptions
- `.isPresent()`: Checks if value exists
- `.get()`: Retrieves the value

---

## The Expense Controller

Now let's create the REST endpoints.

### Create ExpenseController

Create `ExpenseController.java` in the `controller` package:

```java
package com.example.expensetracker.controller;

import com.example.expensetracker.model.Expense;
import com.example.expensetracker.service.ExpenseService;
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

    // Constructor injection (recommended way)
    @Autowired
    public ExpenseController(ExpenseService expenseService) {
        this.expenseService = expenseService;
    }

    // CREATE: POST /api/expenses
    @PostMapping
    public ResponseEntity<Expense> createExpense(@RequestBody Expense expense) {
        Expense createdExpense = expenseService.createExpense(expense);
        return new ResponseEntity<>(createdExpense, HttpStatus.CREATED);
    }

    // READ: GET /api/expenses (get all)
    @GetMapping
    public ResponseEntity<List<Expense>> getAllExpenses() {
        List<Expense> expenses = expenseService.getAllExpenses();
        return ResponseEntity.ok(expenses);
    }

    // READ: GET /api/expenses/{id} (get one)
    @GetMapping("/{id}")
    public ResponseEntity<Expense> getExpenseById(@PathVariable Long id) {
        Optional<Expense> expense = expenseService.getExpenseById(id);

        if (expense.isPresent()) {
            return ResponseEntity.ok(expense.get());
        } else {
            return ResponseEntity.notFound().build();
        }
    }

    // UPDATE: PUT /api/expenses/{id}
    @PutMapping("/{id}")
    public ResponseEntity<Expense> updateExpense(
            @PathVariable Long id,
            @RequestBody Expense expense) {

        Optional<Expense> updatedExpense = expenseService.updateExpense(id, expense);

        if (updatedExpense.isPresent()) {
            return ResponseEntity.ok(updatedExpense.get());
        } else {
            return ResponseEntity.notFound().build();
        }
    }

    // DELETE: DELETE /api/expenses/{id}
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteExpense(@PathVariable Long id) {
        boolean deleted = expenseService.deleteExpense(id);

        if (deleted) {
            return ResponseEntity.noContent().build();
        } else {
            return ResponseEntity.notFound().build();
        }
    }
}
```

### Understanding the Controller

**@RequestMapping("/api/expenses")**:
- Base path for all endpoints in this controller
- All URLs start with `/api/expenses`

**@Autowired**:
- Spring automatically provides the `ExpenseService` instance
- This is called **Dependency Injection**

**@RequestBody**:
- Tells Spring to convert JSON from request body to `Expense` object
- Example: `{"description": "Lunch", "amount": 15.50, "date": "2024-10-20"}`

**@PathVariable**:
- Extracts value from URL path
- Example: `/api/expenses/5` → `id = 5`

**ResponseEntity<T>**:
- Allows customizing HTTP response (status code, headers, body)
- Better than just returning the object

**HTTP Status Codes**:
- `201 CREATED`: Resource successfully created
- `200 OK`: Request successful
- `204 NO CONTENT`: Successful deletion (no body to return)
- `404 NOT FOUND`: Resource doesn't exist

---

## Testing Your API

Now let's test all CRUD operations!

### Setup: Start Your Application

1. Run `ExpenseTrackerApplication`
2. Wait for "Started ExpenseTrackerApplication..."
3. Open Postman

### Test 1: Create an Expense (POST)

**Request**:
- Method: `POST`
- URL: `http://localhost:8080/api/expenses`
- Headers: `Content-Type: application/json`
- Body (raw JSON):
```json
{
  "description": "Grocery shopping",
  "amount": 45.50,
  "date": "2024-10-20"
}
```

**Expected Response** (Status: 201 Created):
```json
{
  "id": 1,
  "description": "Grocery shopping",
  "amount": 45.50,
  "date": "2024-10-20"
}
```

**Create a few more expenses**:
```json
{
  "description": "Gas",
  "amount": 60.00,
  "date": "2024-10-19"
}
```

```json
{
  "description": "Coffee",
  "amount": 5.50,
  "date": "2024-10-20"
}
```

### Test 2: Get All Expenses (GET)

**Request**:
- Method: `GET`
- URL: `http://localhost:8080/api/expenses`

**Expected Response** (Status: 200 OK):
```json
[
  {
    "id": 1,
    "description": "Grocery shopping",
    "amount": 45.50,
    "date": "2024-10-20"
  },
  {
    "id": 2,
    "description": "Gas",
    "amount": 60.00,
    "date": "2024-10-19"
  },
  {
    "id": 3,
    "description": "Coffee",
    "amount": 5.50,
    "date": "2024-10-20"
  }
]
```

### Test 3: Get Expense by ID (GET)

**Request**:
- Method: `GET`
- URL: `http://localhost:8080/api/expenses/1`

**Expected Response** (Status: 200 OK):
```json
{
  "id": 1,
  "description": "Grocery shopping",
  "amount": 45.50,
  "date": "2024-10-20"
}
```

**Test with invalid ID**:
- URL: `http://localhost:8080/api/expenses/999`
- Expected: `404 Not Found`

### Test 4: Update an Expense (PUT)

**Request**:
- Method: `PUT`
- URL: `http://localhost:8080/api/expenses/1`
- Body:
```json
{
  "description": "Grocery shopping - updated",
  "amount": 50.00,
  "date": "2024-10-20"
}
```

**Expected Response** (Status: 200 OK):
```json
{
  "id": 1,
  "description": "Grocery shopping - updated",
  "amount": 50.00,
  "date": "2024-10-20"
}
```

Verify by getting all expenses again.

### Test 5: Delete an Expense (DELETE)

**Request**:
- Method: `DELETE`
- URL: `http://localhost:8080/api/expenses/3`

**Expected Response** (Status: 204 No Content):
- No body, just status code

Verify by getting all expenses - ID 3 should be gone.

---

## Testing with cURL

If you prefer command line:

### Create
```bash
curl -X POST http://localhost:8080/api/expenses \
  -H "Content-Type: application/json" \
  -d '{"description":"Lunch","amount":15.50,"date":"2024-10-20"}'
```

### Get All
```bash
curl http://localhost:8080/api/expenses
```

### Get by ID
```bash
curl http://localhost:8080/api/expenses/1
```

### Update
```bash
curl -X PUT http://localhost:8080/api/expenses/1 \
  -H "Content-Type: application/json" \
  -d '{"description":"Updated lunch","amount":20.00,"date":"2024-10-20"}'
```

### Delete
```bash
curl -X DELETE http://localhost:8080/api/expenses/1
```

---

## Common Issues and Solutions

### Issue 1: 405 Method Not Allowed

**Problem**: Using wrong HTTP method

**Solution**: Check:
- POST for create
- GET for read
- PUT for update
- DELETE for delete

### Issue 2: 400 Bad Request

**Problem**: Invalid JSON in request body

**Solutions**:
- Check for missing commas
- Ensure proper quote marks
- Validate JSON using online validator

### Issue 3: 404 Not Found on POST/PUT

**Problem**: Wrong URL or missing slash

**Correct**: `/api/expenses`
**Incorrect**: `/api/expense` (no 's')

### Issue 4: ID Not Auto-Incrementing

**Problem**: Setting ID in request body

**Solution**: Don't include `id` in POST request. The service generates it.

### Issue 5: Data Lost After Restart

**This is expected!** Data is in memory. It will be saved in Phase 3 with a database.

---

## Understanding the Flow

Let's trace a complete request:

### Creating an Expense (POST)

1. **Client sends request**: POST /api/expenses with JSON body
2. **Spring receives request**: Routes to `ExpenseController.createExpense()`
3. **@RequestBody conversion**: JSON → Expense object
4. **Controller calls service**: `expenseService.createExpense(expense)`
5. **Service processes**: Generates ID, adds to ArrayList
6. **Service returns**: The created expense
7. **Controller wraps response**: ResponseEntity with 201 status
8. **Spring converts to JSON**: Expense object → JSON
9. **Client receives response**: JSON with ID included

### Diagram

```
Client (Postman)
    ↓ POST /api/expenses + JSON
Controller (ExpenseController)
    ↓ calls createExpense()
Service (ExpenseService)
    ↓ adds to ArrayList
In-Memory Storage
    ↓ returns Expense with ID
Controller
    ↓ wraps in ResponseEntity
Client ← JSON response
```

---

## Checkpoint Exercises

### Exercise 1: Add Total Calculation

Add an endpoint that returns the total of all expenses.

**Endpoint**: `GET /api/expenses/total`

**Expected response**:
```json
{
  "total": 111.00,
  "count": 3
}
```

<details>
<summary>Show Solution</summary>

**In ExpenseService.java**:
```java
public Double getTotalExpenses() {
    return expenses.stream()
            .mapToDouble(Expense::getAmount)
            .sum();
}

public int getExpenseCount() {
    return expenses.size();
}
```

**Create DTO** (`TotalResponse.java` in `dto` package):
```java
public class TotalResponse {
    private Double total;
    private int count;

    public TotalResponse(Double total, int count) {
        this.total = total;
        this.count = count;
    }

    // Getters and setters...
}
```

**In ExpenseController.java**:
```java
@GetMapping("/total")
public ResponseEntity<TotalResponse> getTotal() {
    Double total = expenseService.getTotalExpenses();
    int count = expenseService.getExpenseCount();
    TotalResponse response = new TotalResponse(total, count);
    return ResponseEntity.ok(response);
}
```
</details>

### Exercise 2: Filter by Description

Add a search endpoint that filters by keyword in description.

**Endpoint**: `GET /api/expenses/search?keyword=coffee`

<details>
<summary>Show Solution</summary>

**In ExpenseService.java**:
```java
public List<Expense> searchByDescription(String keyword) {
    return expenses.stream()
            .filter(expense -> expense.getDescription()
                    .toLowerCase()
                    .contains(keyword.toLowerCase()))
            .collect(Collectors.toList());
}
```

**In ExpenseController.java**:
```java
@GetMapping("/search")
public ResponseEntity<List<Expense>> searchExpenses(
        @RequestParam String keyword) {
    List<Expense> results = expenseService.searchByDescription(keyword);
    return ResponseEntity.ok(results);
}
```

Add import: `import org.springframework.web.bind.annotation.RequestParam;`
</details>

### Exercise 3: Add Date Range Filter

Get expenses between two dates.

**Endpoint**: `GET /api/expenses/date-range?start=2024-10-01&end=2024-10-31`

<details>
<summary>Show Solution</summary>

**In ExpenseService.java**:
```java
public List<Expense> getExpensesByDateRange(LocalDate start, LocalDate end) {
    return expenses.stream()
            .filter(expense -> !expense.getDate().isBefore(start)
                    && !expense.getDate().isAfter(end))
            .collect(Collectors.toList());
}
```

**In ExpenseController.java**:
```java
@GetMapping("/date-range")
public ResponseEntity<List<Expense>> getExpensesByDateRange(
        @RequestParam @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate start,
        @RequestParam @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate end) {
    List<Expense> results = expenseService.getExpensesByDateRange(start, end);
    return ResponseEntity.ok(results);
}
```

Add import: `import org.springframework.format.annotation.DateTimeFormat;`
</details>

---

## Key Takeaways

✅ **CRUD**: Create, Read, Update, Delete - the foundation of data operations
✅ **HTTP Methods**: POST (create), GET (read), PUT (update), DELETE (delete)
✅ **Separation of Concerns**: Controller handles requests, Service handles logic
✅ **@RequestBody**: Converts JSON to Java objects
✅ **@PathVariable**: Extracts values from URL
✅ **ResponseEntity**: Controls HTTP response details
✅ **Status Codes**: 200 OK, 201 Created, 204 No Content, 404 Not Found

---

## What's Next?

In **Phase 3**, you'll learn:
- Persistent storage with H2 database
- JPA (Java Persistence API)
- Repository pattern
- Automatic table creation
- SQL basics

The big change: Your data will survive application restarts!

---

## Additional Resources

### Documentation
- [Spring Web Annotations](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller.html)
- [HTTP Status Codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)

### Practice
- Implement more filtering options (by amount, by date)
- Add sorting (by date, amount, description)
- Implement pagination (return 10 expenses at a time)

---

**Next**: [Phase 3: H2 Database Integration](PHASE_3.md)
