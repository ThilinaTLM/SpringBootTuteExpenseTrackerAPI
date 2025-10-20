# Phase 8: File Uploads, Reports & Testing

**Duration**: 8-10 hours
**Complexity**: ⭐⭐⭐⭐⭐ Advanced

## Learning Objectives

By the end of this phase, you will:
- Implement file upload functionality
- Store and retrieve files securely
- Generate PDF and CSV reports
- Write unit tests with JUnit 5
- Write integration tests
- Use Mockito for mocking
- Add API documentation with Swagger

## Prerequisites

- ✅ Completed Phase 7
- ✅ JWT authentication working
- ✅ User-specific data implemented

---

## Part A: File Upload (Receipt Images)

### Why File Upload?

Users want to:
- Attach receipt images to expenses
- Store proof of purchase
- Review receipts later

---

## Step 1: Configure File Upload

### Update application.properties

Add file upload configuration:

```properties
# File Upload Configuration
spring.servlet.multipart.enabled=true
spring.servlet.multipart.max-file-size=5MB
spring.servlet.multipart.max-request-size=5MB

# File Storage Location
file.upload-dir=./uploads/receipts
```

---

## Step 2: Create Receipt Entity

Create `Receipt.java` in the `model` package:

```java
package com.example.expensetracker.model;

import com.fasterxml.jackson.annotation.JsonIgnore;
import jakarta.persistence.*;
import java.time.LocalDateTime;

@Entity
@Table(name = "receipts")
public class Receipt {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String filename;

    @Column(nullable = false)
    private String originalFilename;

    @Column(nullable = false)
    private String contentType;

    @Column(nullable = false)
    private Long fileSize;

    @Column(nullable = false)
    private String filePath;

    @Column(nullable = false)
    private LocalDateTime uploadedAt;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "expense_id", nullable = false)
    @JsonIgnore
    private Expense expense;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", nullable = false)
    @JsonIgnore
    private User user;

    @PrePersist
    protected void onCreate() {
        uploadedAt = LocalDateTime.now();
    }

    // Constructors
    public Receipt() {
    }

    // Getters and Setters
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getFilename() {
        return filename;
    }

    public void setFilename(String filename) {
        this.filename = filename;
    }

    public String getOriginalFilename() {
        return originalFilename;
    }

    public void setOriginalFilename(String originalFilename) {
        this.originalFilename = originalFilename;
    }

    public String getContentType() {
        return contentType;
    }

    public void setContentType(String contentType) {
        this.contentType = contentType;
    }

    public Long getFileSize() {
        return fileSize;
    }

    public void setFileSize(Long fileSize) {
        this.fileSize = fileSize;
    }

    public String getFilePath() {
        return filePath;
    }

    public void setFilePath(String filePath) {
        this.filePath = filePath;
    }

    public LocalDateTime getUploadedAt() {
        return uploadedAt;
    }

    public void setUploadedAt(LocalDateTime uploadedAt) {
        this.uploadedAt = uploadedAt;
    }

    public Expense getExpense() {
        return expense;
    }

    public void setExpense(Expense expense) {
        this.expense = expense;
    }

    public User getUser() {
        return user;
    }

    public void setUser(User user) {
        this.user = user;
    }
}
```

---

## Step 3: Create File Storage Service

### Create FileStorageService.java

Create new package: `com.example.expensetracker.service.file`

```java
package com.example.expensetracker.service.file;

import com.example.expensetracker.exception.InvalidDataException;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;
import org.springframework.util.StringUtils;
import org.springframework.web.multipart.MultipartFile;

import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.nio.file.StandardCopyOption;
import java.util.UUID;

@Service
public class FileStorageService {

    private final Path fileStorageLocation;

    public FileStorageService(@Value("${file.upload-dir}") String uploadDir) {
        this.fileStorageLocation = Paths.get(uploadDir).toAbsolutePath().normalize();

        try {
            Files.createDirectories(this.fileStorageLocation);
        } catch (Exception ex) {
            throw new RuntimeException("Could not create upload directory", ex);
        }
    }

    public String storeFile(MultipartFile file) {
        // Validate file
        validateFile(file);

        // Generate unique filename
        String originalFilename = StringUtils.cleanPath(file.getOriginalFilename());
        String fileExtension = getFileExtension(originalFilename);
        String filename = UUID.randomUUID().toString() + fileExtension;

        try {
            // Copy file to storage location
            Path targetLocation = this.fileStorageLocation.resolve(filename);
            Files.copy(file.getInputStream(), targetLocation, StandardCopyOption.REPLACE_EXISTING);

            return filename;
        } catch (IOException ex) {
            throw new RuntimeException("Could not store file " + filename, ex);
        }
    }

    public Path loadFile(String filename) {
        return fileStorageLocation.resolve(filename).normalize();
    }

    public void deleteFile(String filename) {
        try {
            Path filePath = loadFile(filename);
            Files.deleteIfExists(filePath);
        } catch (IOException ex) {
            throw new RuntimeException("Could not delete file " + filename, ex);
        }
    }

    private void validateFile(MultipartFile file) {
        if (file.isEmpty()) {
            throw new InvalidDataException("File is empty");
        }

        // Check file size (5MB max)
        if (file.getSize() > 5 * 1024 * 1024) {
            throw new InvalidDataException("File size exceeds maximum limit of 5MB");
        }

        // Check file type
        String contentType = file.getContentType();
        if (contentType == null || !isValidContentType(contentType)) {
            throw new InvalidDataException("Invalid file type. Only JPG, PNG, and PDF allowed");
        }
    }

    private boolean isValidContentType(String contentType) {
        return contentType.equals("image/jpeg") ||
               contentType.equals("image/png") ||
               contentType.equals("application/pdf");
    }

    private String getFileExtension(String filename) {
        int lastDotIndex = filename.lastIndexOf('.');
        if (lastDotIndex == -1) {
            return "";
        }
        return filename.substring(lastDotIndex);
    }
}
```

---

## Step 4: Create Receipt Repository and Service

### Create ReceiptRepository.java

```java
package com.example.expensetracker.repository;

import com.example.expensetracker.model.Receipt;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
public interface ReceiptRepository extends JpaRepository<Receipt, Long> {

    List<Receipt> findByExpenseId(Long expenseId);

    List<Receipt> findByUserId(Long userId);

    void deleteByExpenseId(Long expenseId);
}
```

### Create ReceiptService.java

```java
package com.example.expensetracker.service;

import com.example.expensetracker.exception.ResourceNotFoundException;
import com.example.expensetracker.model.Expense;
import com.example.expensetracker.model.Receipt;
import com.example.expensetracker.model.User;
import com.example.expensetracker.repository.ReceiptRepository;
import com.example.expensetracker.service.file.FileStorageService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.web.multipart.MultipartFile;

import java.util.List;

@Service
public class ReceiptService {

    private final ReceiptRepository receiptRepository;
    private final FileStorageService fileStorageService;
    private final ExpenseService expenseService;
    private final UserService userService;

    @Autowired
    public ReceiptService(ReceiptRepository receiptRepository,
                          FileStorageService fileStorageService,
                          ExpenseService expenseService,
                          UserService userService) {
        this.receiptRepository = receiptRepository;
        this.fileStorageService = fileStorageService;
        this.expenseService = expenseService;
        this.userService = userService;
    }

    @Transactional
    public Receipt uploadReceipt(Long expenseId, MultipartFile file) {
        Long userId = getCurrentUserId();

        // Verify expense exists and belongs to user
        Expense expense = expenseService.getExpenseById(expenseId);
        if (!expense.getUser().getId().equals(userId)) {
            throw new ResourceNotFoundException("Expense not found");
        }

        // Store file
        String filename = fileStorageService.storeFile(file);

        // Create receipt record
        Receipt receipt = new Receipt();
        receipt.setFilename(filename);
        receipt.setOriginalFilename(file.getOriginalFilename());
        receipt.setContentType(file.getContentType());
        receipt.setFileSize(file.getSize());
        receipt.setFilePath("/api/receipts/" + filename);
        receipt.setExpense(expense);
        receipt.setUser(userService.getUserById(userId));

        return receiptRepository.save(receipt);
    }

    public List<Receipt> getReceiptsByExpense(Long expenseId) {
        Long userId = getCurrentUserId();

        // Verify expense belongs to user
        Expense expense = expenseService.getExpenseById(expenseId);
        if (!expense.getUser().getId().equals(userId)) {
            throw new ResourceNotFoundException("Expense not found");
        }

        return receiptRepository.findByExpenseId(expenseId);
    }

    public Receipt getReceiptById(Long id) {
        Long userId = getCurrentUserId();

        Receipt receipt = receiptRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Receipt", "id", id));

        if (!receipt.getUser().getId().equals(userId)) {
            throw new ResourceNotFoundException("Receipt not found");
        }

        return receipt;
    }

    @Transactional
    public void deleteReceipt(Long id) {
        Receipt receipt = getReceiptById(id);

        // Delete file from storage
        fileStorageService.deleteFile(receipt.getFilename());

        // Delete database record
        receiptRepository.delete(receipt);
    }

    private Long getCurrentUserId() {
        return (Long) SecurityContextHolder.getContext().getAuthentication().getPrincipal();
    }
}
```

---

## Step 5: Create Receipt Controller

### Create ReceiptController.java

```java
package com.example.expensetracker.controller;

import com.example.expensetracker.model.Receipt;
import com.example.expensetracker.service.ReceiptService;
import com.example.expensetracker.service.file.FileStorageService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.io.Resource;
import org.springframework.core.io.UrlResource;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

import java.nio.file.Path;
import java.util.List;

@RestController
@RequestMapping("/api")
public class ReceiptController {

    private final ReceiptService receiptService;
    private final FileStorageService fileStorageService;

    @Autowired
    public ReceiptController(ReceiptService receiptService, FileStorageService fileStorageService) {
        this.receiptService = receiptService;
        this.fileStorageService = fileStorageService;
    }

    @PostMapping("/expenses/{expenseId}/receipts")
    public ResponseEntity<Receipt> uploadReceipt(
            @PathVariable Long expenseId,
            @RequestParam("file") MultipartFile file) {

        Receipt receipt = receiptService.uploadReceipt(expenseId, file);
        return new ResponseEntity<>(receipt, HttpStatus.CREATED);
    }

    @GetMapping("/expenses/{expenseId}/receipts")
    public ResponseEntity<List<Receipt>> getReceiptsByExpense(@PathVariable Long expenseId) {
        List<Receipt> receipts = receiptService.getReceiptsByExpense(expenseId);
        return ResponseEntity.ok(receipts);
    }

    @GetMapping("/receipts/{id}")
    public ResponseEntity<Resource> downloadReceipt(@PathVariable Long id) {
        Receipt receipt = receiptService.getReceiptById(id);

        try {
            Path filePath = fileStorageService.loadFile(receipt.getFilename());
            Resource resource = new UrlResource(filePath.toUri());

            if (resource.exists()) {
                return ResponseEntity.ok()
                        .contentType(MediaType.parseMediaType(receipt.getContentType()))
                        .header(HttpHeaders.CONTENT_DISPOSITION,
                                "attachment; filename=\"" + receipt.getOriginalFilename() + "\"")
                        .body(resource);
            } else {
                return ResponseEntity.notFound().build();
            }
        } catch (Exception ex) {
            return ResponseEntity.internalServerError().build();
        }
    }

    @DeleteMapping("/receipts/{id}")
    public ResponseEntity<Void> deleteReceipt(@PathVariable Long id) {
        receiptService.deleteReceipt(id);
        return ResponseEntity.noContent().build();
    }
}
```

---

## Part B: Report Generation

### Step 6: Add Dependencies for Reports

### Update pom.xml

Add CSV generation library:

```xml
<!-- Apache Commons CSV -->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-csv</artifactId>
    <version>1.10.0</version>
</dependency>
```

---

## Step 7: Create Report Service

### Create ReportService.java

```java
package com.example.expensetracker.service;

import com.example.expensetracker.model.Expense;
import org.apache.commons.csv.CSVFormat;
import org.apache.commons.csv.CSVPrinter;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.stereotype.Service;

import java.io.ByteArrayOutputStream;
import java.io.OutputStreamWriter;
import java.time.LocalDate;
import java.util.List;

@Service
public class ReportService {

    private final ExpenseService expenseService;

    @Autowired
    public ReportService(ExpenseService expenseService) {
        this.expenseService = expenseService;
    }

    public byte[] generateCsvReport(LocalDate startDate, LocalDate endDate, Long categoryId) {
        Long userId = getCurrentUserId();

        // Get expenses
        List<Expense> expenses = expenseService.getExpensesByDateRange(startDate, endDate);

        // Filter by category if specified
        if (categoryId != null) {
            expenses = expenses.stream()
                    .filter(e -> e.getCategory() != null && e.getCategory().getId().equals(categoryId))
                    .toList();
        }

        try (ByteArrayOutputStream out = new ByteArrayOutputStream();
             OutputStreamWriter writer = new OutputStreamWriter(out);
             CSVPrinter csvPrinter = new CSVPrinter(writer, CSVFormat.DEFAULT
                     .withHeader("Date", "Description", "Amount", "Category"))) {

            double total = 0.0;

            for (Expense expense : expenses) {
                csvPrinter.printRecord(
                        expense.getDate(),
                        expense.getDescription(),
                        expense.getAmount(),
                        expense.getCategory() != null ? expense.getCategory().getName() : "Uncategorized"
                );
                total += expense.getAmount();
            }

            // Add total row
            csvPrinter.println();
            csvPrinter.printRecord("", "TOTAL", total, "");

            csvPrinter.flush();
            return out.toByteArray();

        } catch (Exception e) {
            throw new RuntimeException("Error generating CSV report", e);
        }
    }

    private Long getCurrentUserId() {
        return (Long) SecurityContextHolder.getContext().getAuthentication().getPrincipal();
    }
}
```

---

## Step 8: Create Report Controller

### Create ReportController.java

```java
package com.example.expensetracker.controller;

import com.example.expensetracker.service.ReportService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.format.annotation.DateTimeFormat;
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.time.LocalDate;

@RestController
@RequestMapping("/api/reports")
public class ReportController {

    private final ReportService reportService;

    @Autowired
    public ReportController(ReportService reportService) {
        this.reportService = reportService;
    }

    @GetMapping("/expenses/csv")
    public ResponseEntity<byte[]> generateCsvReport(
            @RequestParam @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate startDate,
            @RequestParam @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate endDate,
            @RequestParam(required = false) Long categoryId) {

        byte[] csvData = reportService.generateCsvReport(startDate, endDate, categoryId);

        return ResponseEntity.ok()
                .header(HttpHeaders.CONTENT_DISPOSITION,
                        "attachment; filename=\"expense_report_" + LocalDate.now() + ".csv\"")
                .contentType(MediaType.parseMediaType("text/csv"))
                .body(csvData);
    }
}
```

---

## Part C: Testing

### Step 9: Add Testing Dependencies

Already included in Spring Boot Starter Test:
- JUnit 5
- Mockito
- Spring Test
- AssertJ

---

## Step 10: Write Unit Tests

### Create ExpenseServiceTest.java

Create in `src/test/java/com/example/expensetracker/service`:

```java
package com.example.expensetracker.service;

import com.example.expensetracker.exception.ResourceNotFoundException;
import com.example.expensetracker.model.Expense;
import com.example.expensetracker.model.User;
import com.example.expensetracker.repository.ExpenseRepository;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import java.time.LocalDate;
import java.util.Optional;

import static org.assertj.core.api.Assertions.assertThat;
import static org.assertj.core.api.Assertions.assertThatThrownBy;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
class ExpenseServiceTest {

    @Mock
    private ExpenseRepository expenseRepository;

    @Mock
    private CategoryService categoryService;

    @Mock
    private UserService userService;

    @InjectMocks
    private ExpenseService expenseService;

    private Expense testExpense;
    private User testUser;

    @BeforeEach
    void setUp() {
        testUser = new User("testuser", "test@example.com", "password");
        testUser.setId(1L);

        testExpense = new Expense("Test Expense", 50.0, LocalDate.now());
        testExpense.setId(1L);
        testExpense.setUser(testUser);
    }

    @Test
    void getExpenseById_ShouldReturnExpense_WhenExpenseExists() {
        // Given
        when(expenseRepository.findById(1L)).thenReturn(Optional.of(testExpense));

        // When
        Expense result = expenseService.getExpenseById(1L);

        // Then
        assertThat(result).isNotNull();
        assertThat(result.getId()).isEqualTo(1L);
        assertThat(result.getDescription()).isEqualTo("Test Expense");
        verify(expenseRepository, times(1)).findById(1L);
    }

    @Test
    void getExpenseById_ShouldThrowException_WhenExpenseNotFound() {
        // Given
        when(expenseRepository.findById(999L)).thenReturn(Optional.empty());

        // When & Then
        assertThatThrownBy(() -> expenseService.getExpenseById(999L))
                .isInstanceOf(ResourceNotFoundException.class)
                .hasMessageContaining("Expense not found");

        verify(expenseRepository, times(1)).findById(999L);
    }

    @Test
    void deleteExpense_ShouldDeleteExpense_WhenExpenseExists() {
        // Given
        when(expenseRepository.findById(1L)).thenReturn(Optional.of(testExpense));
        doNothing().when(expenseRepository).delete(testExpense);

        // When
        expenseService.deleteExpense(1L);

        // Then
        verify(expenseRepository, times(1)).findById(1L);
        verify(expenseRepository, times(1)).delete(testExpense);
    }
}
```

---

## Step 11: Write Integration Tests

### Create ExpenseControllerIntegrationTest.java

Create in `src/test/java/com/example/expensetracker/controller`:

```java
package com.example.expensetracker.controller;

import com.example.expensetracker.model.Expense;
import com.example.expensetracker.model.User;
import com.example.expensetracker.repository.ExpenseRepository;
import com.example.expensetracker.repository.UserRepository;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.transaction.annotation.Transactional;

import java.time.LocalDate;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@SpringBootTest
@AutoConfigureMockMvc
@Transactional
class ExpenseControllerIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;

    @Autowired
    private ExpenseRepository expenseRepository;

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private PasswordEncoder passwordEncoder;

    private User testUser;
    private String jwtToken;

    @BeforeEach
    void setUp() throws Exception {
        // Create test user
        testUser = new User("testuser", "test@example.com", passwordEncoder.encode("password123"));
        testUser = userRepository.save(testUser);

        // Login and get JWT token
        String loginJson = "{\"username\":\"testuser\",\"password\":\"password123\"}";

        String response = mockMvc.perform(post("/api/auth/login")
                .contentType(MediaType.APPLICATION_JSON)
                .content(loginJson))
                .andExpect(status().isOk())
                .andReturn()
                .getResponse()
                .getContentAsString();

        jwtToken = objectMapper.readTree(response).get("token").asText();
    }

    @Test
    void createExpense_ShouldReturnCreated() throws Exception {
        // Given
        Expense expense = new Expense("Test Expense", 50.0, LocalDate.now());

        // When & Then
        mockMvc.perform(post("/api/expenses")
                .header("Authorization", "Bearer " + jwtToken)
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(expense)))
                .andExpect(status().isCreated())
                .andExpect(jsonPath("$.description").value("Test Expense"))
                .andExpect(jsonPath("$.amount").value(50.0));
    }

    @Test
    void getExpenses_ShouldReturnPagedExpenses() throws Exception {
        // Given
        Expense expense1 = new Expense("Expense 1", 30.0, LocalDate.now());
        expense1.setUser(testUser);
        expenseRepository.save(expense1);

        Expense expense2 = new Expense("Expense 2", 40.0, LocalDate.now());
        expense2.setUser(testUser);
        expenseRepository.save(expense2);

        // When & Then
        mockMvc.perform(get("/api/expenses")
                .header("Authorization", "Bearer " + jwtToken))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.content").isArray())
                .andExpect(jsonPath("$.content.length()").value(2));
    }

    @Test
    void createExpense_WithoutAuth_ShouldReturnUnauthorized() throws Exception {
        // Given
        Expense expense = new Expense("Test Expense", 50.0, LocalDate.now());

        // When & Then
        mockMvc.perform(post("/api/expenses")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(expense)))
                .andExpect(status().isUnauthorized());
    }
}
```

---

## Step 12: Run Tests

### Run All Tests

```bash
# Using Maven
mvn test

# Using IDE
Right-click on test directory → Run Tests
```

### Check Test Coverage

```bash
mvn test jacoco:report
# Report generated in target/site/jacoco/index.html
```

---

## Part D: API Documentation with Swagger

### Step 13: Add Swagger Dependencies

### Update pom.xml

```xml
<!-- Swagger/OpenAPI -->
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.2.0</version>
</dependency>
```

---

## Step 14: Configure Swagger

### Update application.properties

```properties
# Swagger Configuration
springdoc.api-docs.path=/v3/api-docs
springdoc.swagger-ui.path=/swagger-ui.html
springdoc.swagger-ui.operationsSorter=method
```

### Update SecurityConfig.java

Allow access to Swagger UI:

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
            .csrf(csrf -> csrf.disable())
            .sessionManagement(session ->
                    session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                    .requestMatchers("/api/auth/**").permitAll()
                    .requestMatchers("/swagger-ui/**", "/v3/api-docs/**").permitAll() // NEW
                    .anyRequest().authenticated()
            )
            .addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class);

    return http.build();
}
```

---

## Step 15: Access Swagger UI

Start your application and visit:
```
http://localhost:8080/swagger-ui.html
```

You'll see:
- All API endpoints
- Request/response schemas
- Try out functionality
- Authentication support

---

## Testing the Complete Application

### Test Workflow

1. **Register User**
   ```bash
   POST /api/auth/register
   ```

2. **Login**
   ```bash
   POST /api/auth/login
   ```

3. **Create Categories**
   ```bash
   POST /api/categories
   Authorization: Bearer <token>
   ```

4. **Create Expenses**
   ```bash
   POST /api/expenses?categoryId=1
   Authorization: Bearer <token>
   ```

5. **Upload Receipt**
   ```bash
   POST /api/expenses/1/receipts
   Authorization: Bearer <token>
   Content-Type: multipart/form-data
   ```

6. **Generate Report**
   ```bash
   GET /api/reports/expenses/csv?startDate=2024-01-01&endDate=2024-12-31
   Authorization: Bearer <token>
   ```

---

## Key Takeaways

✅ **File Upload**: MultipartFile handling and validation
✅ **File Storage**: Secure file storage with UUID naming
✅ **Report Generation**: CSV export with Apache Commons CSV
✅ **Unit Testing**: Mockito for mocking dependencies
✅ **Integration Testing**: End-to-end API testing
✅ **API Documentation**: Swagger/OpenAPI for interactive docs
✅ **Production Ready**: Complete, tested, documented API

---

## Congratulations! 🎉

You've completed all 8 phases and built a **production-ready REST API** with:
- ✅ Complete CRUD operations
- ✅ MySQL database persistence
- ✅ JWT authentication & authorization
- ✅ File upload functionality
- ✅ Report generation
- ✅ Comprehensive testing
- ✅ API documentation

## Next Steps

### Optional Enhancements
1. **Dockerization**: Containerize your application
2. **CI/CD**: Set up automated testing and deployment
3. **Cloud Deployment**: Deploy to AWS, Heroku, or Azure
4. **Frontend**: Build React or Angular frontend
5. **Advanced Features**:
   - Email notifications
   - Budget tracking
   - Recurring expenses
   - Data visualization
   - Multi-currency support

### Continue Learning
- Microservices architecture
- Message queues (RabbitMQ, Kafka)
- Caching (Redis)
- GraphQL
- WebSockets for real-time updates

---

**Thank you for completing this learning journey!**

You now have the skills to build production-ready Spring Boot REST APIs. Keep practicing and building!
