# Expense Tracker REST API - Features

This document outlines all features organized by learning phase. Each phase introduces new concepts progressively to avoid overwhelming beginners.

---

## Phase 1: Hello World REST API
**Goal**: Get familiar with Spring Boot and create your first REST endpoint

**Duration**: 1-2 hours
**Complexity**: ⭐ Beginner

### Features
- ✅ Spring Boot project setup using Spring Initializr
- ✅ Single GET endpoint that returns a welcome message
- ✅ Understanding project structure (src/main, pom.xml)
- ✅ Running the application
- ✅ Testing with browser/Postman

### New Concepts
- What is Spring Boot?
- What is a REST API?
- Controllers and `@RestController`
- `@GetMapping` annotation
- HTTP GET method
- Running Spring Boot applications

### Endpoints
```
GET /api/hello - Returns welcome message
```

### No Database Yet
Data is hardcoded in the controller

---

## Phase 2: In-Memory CRUD Operations
**Goal**: Learn HTTP methods and implement basic CRUD without a database

**Duration**: 3-4 hours
**Complexity**: ⭐⭐ Beginner

### Features
- ✅ Expense model class (id, description, amount, date)
- ✅ Create new expense (POST)
- ✅ Get all expenses (GET)
- ✅ Get single expense by ID (GET)
- ✅ Update expense (PUT)
- ✅ Delete expense (DELETE)
- ✅ Service layer introduction
- ✅ In-memory storage using ArrayList

### New Concepts
- HTTP methods: GET, POST, PUT, DELETE
- `@PostMapping`, `@PutMapping`, `@DeleteMapping`
- `@PathVariable` for URL parameters
- `@RequestBody` for JSON input
- Controller vs Service separation
- Model/Entity classes
- HTTP status codes (200, 201, 404)
- `ResponseEntity` for custom responses

### Endpoints
```
POST   /api/expenses           - Create expense
GET    /api/expenses           - Get all expenses
GET    /api/expenses/{id}      - Get expense by ID
PUT    /api/expenses/{id}      - Update expense
DELETE /api/expenses/{id}      - Delete expense
```

### Data Storage
In-memory ArrayList (data lost on restart)

---

## Phase 3: H2 Database Integration
**Goal**: Persist data to a database using Spring Data JPA

**Duration**: 4-5 hours
**Complexity**: ⭐⭐ Beginner-Intermediate

### Features
- ✅ H2 in-memory database setup
- ✅ JPA Entity configuration
- ✅ Repository interface (Spring Data JPA)
- ✅ H2 console access
- ✅ Database operations replace ArrayList
- ✅ Auto-generated IDs
- ✅ Data persists during session (lost on app restart)

### New Concepts
- What is JPA/Hibernate?
- `@Entity`, `@Id`, `@GeneratedValue`
- `@Table`, `@Column` annotations
- Repository pattern
- `JpaRepository` interface
- H2 database and console
- `application.properties` configuration
- SQL basics (automatic table creation)

### Endpoints
Same as Phase 2, but now data is stored in H2 database

### Data Storage
H2 in-memory database (data lost on app restart)

---

## Phase 4: Validation & Error Handling
**Goal**: Make the API robust with proper validation and error messages

**Duration**: 3-4 hours
**Complexity**: ⭐⭐⭐ Intermediate

### Features
- ✅ Input validation (amount > 0, description not empty, etc.)
- ✅ Bean Validation annotations (`@NotNull`, `@Positive`, etc.)
- ✅ Custom exception classes
- ✅ Global exception handler (`@ControllerAdvice`)
- ✅ Proper HTTP status codes (400, 404, 500)
- ✅ JSON error response format
- ✅ Validation error messages

### New Concepts
- Bean Validation (JSR-303)
- `@Valid` annotation
- `@NotNull`, `@NotBlank`, `@Positive`, `@PastOrPresent`
- Custom exceptions
- `@ControllerAdvice` and `@ExceptionHandler`
- Proper error response structure
- HTTP status codes in depth

### Validation Rules
- **Description**: Required, not blank, max 255 characters
- **Amount**: Required, must be positive
- **Date**: Required, cannot be in future
- **Category**: Optional at this phase

### Error Response Format
```json
{
  "timestamp": "2024-10-20T10:15:30",
  "status": 400,
  "error": "Bad Request",
  "message": "Amount must be positive",
  "path": "/api/expenses"
}
```

### Endpoints
Same as Phase 3, but with validation

---

## Phase 5: Categories & Pagination
**Goal**: Add categories, filtering, and pagination for better data management

**Duration**: 5-6 hours
**Complexity**: ⭐⭐⭐ Intermediate

### Features
- ✅ Category entity (id, name, description)
- ✅ One-to-Many relationship (Category → Expenses)
- ✅ Category CRUD operations
- ✅ Assign category to expense
- ✅ Filter expenses by category
- ✅ Pagination support (page, size)
- ✅ Sorting support (by date, amount)
- ✅ Get category statistics (total expenses per category)

### New Concepts
- Entity relationships (`@ManyToOne`, `@OneToMany`)
- Foreign keys
- JPA join operations
- `Pageable` interface
- `Page` response type
- Custom repository queries
- `@Query` annotation
- JPQL (Java Persistence Query Language)

### New Endpoints
```
# Categories
POST   /api/categories              - Create category
GET    /api/categories              - Get all categories
GET    /api/categories/{id}         - Get category by ID
PUT    /api/categories/{id}         - Update category
DELETE /api/categories/{id}         - Delete category
GET    /api/categories/{id}/expenses - Get expenses in category

# Expenses (enhanced)
GET    /api/expenses?page=0&size=10&sort=date,desc
GET    /api/expenses?category={categoryId}
GET    /api/expenses/statistics     - Summary by category
```

### Category Examples
- Food & Dining
- Transportation
- Entertainment
- Healthcare
- Utilities
- Shopping
- Others

---

## Phase 6: MySQL Migration & Advanced Queries
**Goal**: Migrate to production database and add complex search features

**Duration**: 4-5 hours
**Complexity**: ⭐⭐⭐ Intermediate

### Features
- ✅ MySQL database setup
- ✅ Database migration from H2 to MySQL
- ✅ Date range filtering
- ✅ Amount range filtering
- ✅ Text search in descriptions
- ✅ Combined filters
- ✅ Monthly/yearly summaries
- ✅ Database persistence across restarts

### New Concepts
- MySQL installation and configuration
- Database connection properties
- `spring.datasource` configuration
- Date range queries
- Between queries
- LIKE queries for text search
- Complex WHERE clauses
- Aggregate functions (SUM, COUNT, AVG)
- GROUP BY queries

### Enhanced Endpoints
```
GET /api/expenses/search?
    startDate=2024-01-01&
    endDate=2024-12-31&
    minAmount=10&
    maxAmount=100&
    category=1&
    description=lunch

GET /api/expenses/summary/monthly?year=2024
GET /api/expenses/summary/yearly?year=2024
GET /api/expenses/summary/category?startDate=...&endDate=...
```

### Search Capabilities
- Date range (start date, end date)
- Amount range (min, max)
- Category filter
- Description keyword search
- Combination of all above

---

## Phase 7: User Authentication & Security
**Goal**: Add user accounts and secure the API with JWT authentication

**Duration**: 6-8 hours
**Complexity**: ⭐⭐⭐⭐ Advanced

### Features
- ✅ User entity (username, email, password)
- ✅ User registration
- ✅ User login (JWT token generation)
- ✅ Password hashing (BCrypt)
- ✅ JWT token validation
- ✅ Secured endpoints (require authentication)
- ✅ User-specific expenses (users only see their own data)
- ✅ User profile management

### New Concepts
- Spring Security basics
- Authentication vs Authorization
- JWT (JSON Web Tokens)
- Password hashing
- `BCryptPasswordEncoder`
- Security filters
- `@PreAuthorize` annotation
- SecurityContext
- User principal

### New Endpoints
```
# Authentication (public)
POST /api/auth/register    - User registration
POST /api/auth/login       - User login (returns JWT)

# User (authenticated)
GET  /api/users/me         - Get current user profile
PUT  /api/users/me         - Update profile
PUT  /api/users/me/password - Change password
```

### Security Rules
- Public endpoints: `/api/auth/**`
- Protected endpoints: All `/api/expenses/**`, `/api/categories/**`, `/api/users/**`
- Users can only access their own expenses
- Admin role (optional stretch goal)

### JWT Flow
1. User registers → Password hashed and stored
2. User logs in → JWT token returned
3. Client includes token in `Authorization: Bearer <token>` header
4. Server validates token for each request
5. User identity extracted from token

---

## Phase 8: Production Features
**Goal**: Add file uploads, reporting, and comprehensive testing

**Duration**: 8-10 hours
**Complexity**: ⭐⭐⭐⭐⭐ Advanced

### Features
- ✅ Receipt image upload
- ✅ File storage (local or cloud)
- ✅ Multiple receipts per expense
- ✅ File validation (size, type)
- ✅ Generate expense reports (PDF/CSV)
- ✅ Data export functionality
- ✅ Unit tests for services
- ✅ Integration tests for controllers
- ✅ API documentation (Swagger/OpenAPI)

### New Concepts
- File upload handling
- `MultipartFile`
- File storage strategies
- File validation
- Report generation libraries (JasperReports, iText)
- CSV generation
- Unit testing with JUnit 5
- Mockito for mocking
- `@WebMvcTest` for controller tests
- `@SpringBootTest` for integration tests
- Test data setup
- API documentation with Swagger

### New Endpoints
```
# File uploads
POST   /api/expenses/{id}/receipts      - Upload receipt
GET    /api/expenses/{id}/receipts      - Get all receipts
GET    /api/receipts/{id}                - Download receipt
DELETE /api/receipts/{id}                - Delete receipt

# Reports
GET    /api/reports/expenses?format=pdf&startDate=...&endDate=...
GET    /api/reports/expenses?format=csv&startDate=...&endDate=...
GET    /api/reports/summary/monthly?year=2024&format=pdf
```

### File Upload Features
- Support image formats: JPG, PNG, PDF
- Max file size: 5MB
- Multiple receipts per expense
- Secure file access (users can only access their receipts)
- File naming strategy (UUID)

### Reporting Features
- **PDF Reports**: Formatted expense reports with totals
- **CSV Export**: Spreadsheet-compatible export
- **Date range selection**: Filter report data
- **Category breakdown**: Expenses grouped by category
- **Monthly summaries**: Visual monthly summaries

### Testing Coverage
- **Unit Tests**: Service layer logic
- **Integration Tests**: API endpoints
- **Test scenarios**:
  - Happy path (successful operations)
  - Validation errors
  - Authentication failures
  - Not found errors
  - Authorization checks

---

## Feature Summary by Phase

| Phase | Features | Database | Security | Testing |
|-------|----------|----------|----------|---------|
| 1 | Hello World | None | None | Manual |
| 2 | CRUD | ArrayList | None | Manual |
| 3 | CRUD | H2 | None | Manual |
| 4 | CRUD + Validation | H2 | None | Manual |
| 5 | Categories + Pagination | H2 | None | Manual |
| 6 | Advanced Search | MySQL | None | Manual |
| 7 | Multi-user | MySQL | JWT | Manual |
| 8 | Files + Reports | MySQL | JWT | Automated |

---

## Optional Extensions (Beyond Phase 8)

For students who want to go further:

### Advanced Features
- Budget tracking and alerts
- Recurring expenses
- Multiple currencies
- Shared expenses (between users)
- Email notifications
- Mobile app integration (expose API)
- Real-time updates (WebSockets)
- Docker containerization
- Cloud deployment (AWS, Heroku)

### Advanced Security
- OAuth2 integration (Google/Facebook login)
- Refresh tokens
- Rate limiting
- CORS configuration

### Advanced Database
- Database migrations (Flyway/Liquibase)
- Redis caching
- Database indexing optimization
- Soft deletes

---

## Learning Path Recommendation

### Minimum Viable Product (MVP)
Complete **Phases 1-4** for a basic working API

### Production-Like Application
Complete **Phases 1-7** for a secure, multi-user API

### Portfolio-Ready Project
Complete **Phases 1-8** with comprehensive testing and documentation

### Time Estimates
- **Part-time** (10 hrs/week): 6-8 weeks for all phases
- **Full-time** (40 hrs/week): 1.5-2 weeks for all phases
- **Intensive** (focused learning): 3-5 days for all phases

---

## Success Criteria

After completing each phase, you should be able to:

**Phase 1**: Explain what a REST API is and run a Spring Boot app
**Phase 2**: Implement CRUD operations and understand HTTP methods
**Phase 3**: Configure a database and use JPA repositories
**Phase 4**: Implement validation and handle errors properly
**Phase 5**: Work with relationships and implement pagination
**Phase 6**: Perform complex queries and use production databases
**Phase 7**: Implement authentication and secure endpoints
**Phase 8**: Add file handling, generate reports, write tests

---

## Next Steps

Ready to begin? Start with **[Phase 1: Hello World REST API](docs/PHASE_1.md)**

Each phase builds on the previous one, so follow them in order for the best learning experience.
