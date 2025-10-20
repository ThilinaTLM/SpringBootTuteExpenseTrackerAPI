# Expense Tracker REST API - API Specification

Complete REST API endpoint reference for all phases of the Expense Tracker project.

**Base URL**: `http://localhost:8080`

---

## Table of Contents
- [Phase 1: Hello World](#phase-1-hello-world)
- [Phase 2-6: Expenses](#expenses-api)
- [Phase 5-6: Categories](#categories-api)
- [Phase 6: Search & Summaries](#search--summaries-api)
- [Phase 7: Authentication](#authentication-api)
- [Phase 7: Users](#users-api)
- [Phase 8: Receipts](#receipts-api)
- [Phase 8: Reports](#reports-api)
- [Common Response Codes](#common-response-codes)
- [Error Response Format](#error-response-format)

---

## Phase 1: Hello World

### Get Welcome Message
Returns a welcome message to verify the API is running.

**Endpoint**: `GET /api/hello`

**Response**: `200 OK`
```json
{
  "message": "Welcome to Expense Tracker API!"
}
```

---

## Expenses API
Available from Phase 2 onwards

### Create Expense
Create a new expense record.

**Endpoint**: `POST /api/expenses`

**Headers**:
- `Content-Type: application/json`
- `Authorization: Bearer <token>` (Phase 7+)

**Request Body**:
```json
{
  "description": "Grocery shopping",
  "amount": 45.50,
  "date": "2024-10-20",
  "categoryId": 1
}
```

**Validation Rules** (Phase 4+):
- `description`: Required, not blank, max 255 characters
- `amount`: Required, must be positive (> 0)
- `date`: Required, cannot be in the future
- `categoryId`: Optional (Phase 5+)

**Response**: `201 Created`
```json
{
  "id": 1,
  "description": "Grocery shopping",
  "amount": 45.50,
  "date": "2024-10-20",
  "categoryId": 1,
  "categoryName": "Food & Dining",
  "createdAt": "2024-10-20T10:30:00",
  "updatedAt": "2024-10-20T10:30:00"
}
```

---

### Get All Expenses
Retrieve all expenses with optional pagination and filtering.

**Endpoint**: `GET /api/expenses`

**Headers**:
- `Authorization: Bearer <token>` (Phase 7+)

**Query Parameters** (Phase 5+):
- `page` (optional): Page number, default 0
- `size` (optional): Page size, default 20
- `sort` (optional): Sort field and direction, e.g., `date,desc` or `amount,asc`
- `category` (optional, Phase 5+): Filter by category ID

**Examples**:
```
GET /api/expenses
GET /api/expenses?page=0&size=10
GET /api/expenses?sort=date,desc
GET /api/expenses?category=1
GET /api/expenses?page=0&size=10&sort=amount,desc&category=2
```

**Response**: `200 OK`

**Phase 2-4** (Simple list):
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
  }
]
```

**Phase 5+** (Paginated):
```json
{
  "content": [
    {
      "id": 1,
      "description": "Grocery shopping",
      "amount": 45.50,
      "date": "2024-10-20",
      "categoryId": 1,
      "categoryName": "Food & Dining"
    }
  ],
  "pageable": {
    "pageNumber": 0,
    "pageSize": 10,
    "sort": {
      "sorted": true,
      "unsorted": false
    }
  },
  "totalElements": 50,
  "totalPages": 5,
  "last": false,
  "first": true
}
```

---

### Get Expense by ID
Retrieve a specific expense by its ID.

**Endpoint**: `GET /api/expenses/{id}`

**Headers**:
- `Authorization: Bearer <token>` (Phase 7+)

**Path Parameters**:
- `id`: Expense ID (integer)

**Response**: `200 OK`
```json
{
  "id": 1,
  "description": "Grocery shopping",
  "amount": 45.50,
  "date": "2024-10-20",
  "categoryId": 1,
  "categoryName": "Food & Dining",
  "createdAt": "2024-10-20T10:30:00",
  "updatedAt": "2024-10-20T10:30:00"
}
```

**Error Response**: `404 Not Found`
```json
{
  "timestamp": "2024-10-20T10:30:00",
  "status": 404,
  "error": "Not Found",
  "message": "Expense not found with id: 999",
  "path": "/api/expenses/999"
}
```

---

### Update Expense
Update an existing expense.

**Endpoint**: `PUT /api/expenses/{id}`

**Headers**:
- `Content-Type: application/json`
- `Authorization: Bearer <token>` (Phase 7+)

**Path Parameters**:
- `id`: Expense ID (integer)

**Request Body**:
```json
{
  "description": "Grocery shopping - updated",
  "amount": 50.00,
  "date": "2024-10-20",
  "categoryId": 1
}
```

**Response**: `200 OK`
```json
{
  "id": 1,
  "description": "Grocery shopping - updated",
  "amount": 50.00,
  "date": "2024-10-20",
  "categoryId": 1,
  "categoryName": "Food & Dining",
  "updatedAt": "2024-10-20T11:00:00"
}
```

**Error Response**: `404 Not Found` (if expense doesn't exist)

---

### Delete Expense
Delete an expense by ID.

**Endpoint**: `DELETE /api/expenses/{id}`

**Headers**:
- `Authorization: Bearer <token>` (Phase 7+)

**Path Parameters**:
- `id`: Expense ID (integer)

**Response**: `204 No Content`

**Error Response**: `404 Not Found` (if expense doesn't exist)

---

## Categories API
Available from Phase 5 onwards

### Create Category
Create a new expense category.

**Endpoint**: `POST /api/categories`

**Headers**:
- `Content-Type: application/json`
- `Authorization: Bearer <token>` (Phase 7+)

**Request Body**:
```json
{
  "name": "Food & Dining",
  "description": "Expenses related to food and restaurants"
}
```

**Validation Rules**:
- `name`: Required, not blank, max 100 characters, unique
- `description`: Optional, max 500 characters

**Response**: `201 Created`
```json
{
  "id": 1,
  "name": "Food & Dining",
  "description": "Expenses related to food and restaurants",
  "createdAt": "2024-10-20T10:00:00"
}
```

---

### Get All Categories
Retrieve all categories.

**Endpoint**: `GET /api/categories`

**Headers**:
- `Authorization: Bearer <token>` (Phase 7+)

**Response**: `200 OK`
```json
[
  {
    "id": 1,
    "name": "Food & Dining",
    "description": "Expenses related to food and restaurants",
    "expenseCount": 25
  },
  {
    "id": 2,
    "name": "Transportation",
    "description": "Gas, public transport, etc.",
    "expenseCount": 10
  }
]
```

---

### Get Category by ID
Retrieve a specific category.

**Endpoint**: `GET /api/categories/{id}`

**Headers**:
- `Authorization: Bearer <token>` (Phase 7+)

**Response**: `200 OK`
```json
{
  "id": 1,
  "name": "Food & Dining",
  "description": "Expenses related to food and restaurants",
  "expenseCount": 25,
  "totalAmount": 1250.50
}
```

---

### Update Category
Update an existing category.

**Endpoint**: `PUT /api/categories/{id}`

**Headers**:
- `Content-Type: application/json`
- `Authorization: Bearer <token>` (Phase 7+)

**Request Body**:
```json
{
  "name": "Food & Dining - Updated",
  "description": "Updated description"
}
```

**Response**: `200 OK`

---

### Delete Category
Delete a category. Cannot delete if it has expenses.

**Endpoint**: `DELETE /api/categories/{id}`

**Headers**:
- `Authorization: Bearer <token>` (Phase 7+)

**Response**: `204 No Content`

**Error Response**: `400 Bad Request` (if category has expenses)
```json
{
  "timestamp": "2024-10-20T10:30:00",
  "status": 400,
  "error": "Bad Request",
  "message": "Cannot delete category with existing expenses",
  "path": "/api/categories/1"
}
```

---

### Get Expenses in Category
Retrieve all expenses for a specific category.

**Endpoint**: `GET /api/categories/{id}/expenses`

**Headers**:
- `Authorization: Bearer <token>` (Phase 7+)

**Query Parameters**:
- `page`, `size`, `sort`: Same as Get All Expenses

**Response**: `200 OK`
```json
{
  "content": [
    {
      "id": 1,
      "description": "Grocery shopping",
      "amount": 45.50,
      "date": "2024-10-20"
    }
  ],
  "totalElements": 25,
  "totalPages": 3
}
```

---

## Search & Summaries API
Available from Phase 6 onwards

### Advanced Expense Search
Search expenses with multiple filters.

**Endpoint**: `GET /api/expenses/search`

**Headers**:
- `Authorization: Bearer <token>` (Phase 7+)

**Query Parameters**:
- `startDate` (optional): Start date (YYYY-MM-DD)
- `endDate` (optional): End date (YYYY-MM-DD)
- `minAmount` (optional): Minimum amount
- `maxAmount` (optional): Maximum amount
- `categoryId` (optional): Category ID
- `description` (optional): Search keyword in description
- `page`, `size`, `sort`: Pagination parameters

**Example**:
```
GET /api/expenses/search?startDate=2024-01-01&endDate=2024-12-31&minAmount=10&maxAmount=100&categoryId=1&description=lunch
```

**Response**: `200 OK`
```json
{
  "content": [
    {
      "id": 5,
      "description": "Lunch with team",
      "amount": 45.50,
      "date": "2024-10-15",
      "categoryId": 1,
      "categoryName": "Food & Dining"
    }
  ],
  "totalElements": 3,
  "totalPages": 1
}
```

---

### Get Statistics
Get expense statistics grouped by category.

**Endpoint**: `GET /api/expenses/statistics`

**Headers**:
- `Authorization: Bearer <token>` (Phase 7+)

**Query Parameters**:
- `startDate` (optional): Start date
- `endDate` (optional): End date

**Response**: `200 OK`
```json
{
  "totalExpenses": 150,
  "totalAmount": 5420.75,
  "categories": [
    {
      "categoryId": 1,
      "categoryName": "Food & Dining",
      "count": 45,
      "totalAmount": 2150.50,
      "percentage": 39.7
    },
    {
      "categoryId": 2,
      "categoryName": "Transportation",
      "count": 30,
      "totalAmount": 1800.00,
      "percentage": 33.2
    }
  ]
}
```

---

### Get Monthly Summary
Get expense summary by month.

**Endpoint**: `GET /api/expenses/summary/monthly`

**Headers**:
- `Authorization: Bearer <token>` (Phase 7+)

**Query Parameters**:
- `year`: Year (required)

**Example**: `GET /api/expenses/summary/monthly?year=2024`

**Response**: `200 OK`
```json
{
  "year": 2024,
  "months": [
    {
      "month": 1,
      "monthName": "January",
      "count": 25,
      "totalAmount": 850.50
    },
    {
      "month": 2,
      "monthName": "February",
      "count": 20,
      "totalAmount": 720.00
    }
  ],
  "yearTotal": 10250.50
}
```

---

### Get Yearly Summary
Get expense summary by year.

**Endpoint**: `GET /api/expenses/summary/yearly`

**Headers**:
- `Authorization: Bearer <token>` (Phase 7+)

**Query Parameters**:
- `startYear` (optional): Start year
- `endYear` (optional): End year

**Response**: `200 OK`
```json
[
  {
    "year": 2024,
    "count": 300,
    "totalAmount": 10250.50
  },
  {
    "year": 2023,
    "count": 280,
    "totalAmount": 9800.00
  }
]
```

---

## Authentication API
Available from Phase 7 onwards

### Register User
Create a new user account.

**Endpoint**: `POST /api/auth/register`

**Headers**:
- `Content-Type: application/json`

**Request Body**:
```json
{
  "username": "john_doe",
  "email": "john@example.com",
  "password": "SecurePass123!"
}
```

**Validation Rules**:
- `username`: Required, 3-50 characters, unique
- `email`: Required, valid email format, unique
- `password`: Required, min 8 characters, must contain uppercase, lowercase, and number

**Response**: `201 Created`
```json
{
  "id": 1,
  "username": "john_doe",
  "email": "john@example.com",
  "createdAt": "2024-10-20T10:00:00"
}
```

**Error Response**: `400 Bad Request` (if username/email already exists)
```json
{
  "timestamp": "2024-10-20T10:30:00",
  "status": 400,
  "error": "Bad Request",
  "message": "Username already exists",
  "path": "/api/auth/register"
}
```

---

### Login
Authenticate and receive JWT token.

**Endpoint**: `POST /api/auth/login`

**Headers**:
- `Content-Type: application/json`

**Request Body**:
```json
{
  "username": "john_doe",
  "password": "SecurePass123!"
}
```

**Response**: `200 OK`
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "type": "Bearer",
  "expiresIn": 86400,
  "user": {
    "id": 1,
    "username": "john_doe",
    "email": "john@example.com"
  }
}
```

**Error Response**: `401 Unauthorized` (invalid credentials)
```json
{
  "timestamp": "2024-10-20T10:30:00",
  "status": 401,
  "error": "Unauthorized",
  "message": "Invalid username or password",
  "path": "/api/auth/login"
}
```

---

## Users API
Available from Phase 7 onwards

### Get Current User Profile
Get the profile of the authenticated user.

**Endpoint**: `GET /api/users/me`

**Headers**:
- `Authorization: Bearer <token>`

**Response**: `200 OK`
```json
{
  "id": 1,
  "username": "john_doe",
  "email": "john@example.com",
  "createdAt": "2024-10-20T10:00:00",
  "totalExpenses": 45,
  "totalAmount": 2150.50
}
```

---

### Update User Profile
Update the current user's profile.

**Endpoint**: `PUT /api/users/me`

**Headers**:
- `Content-Type: application/json`
- `Authorization: Bearer <token>`

**Request Body**:
```json
{
  "email": "newemail@example.com"
}
```

**Response**: `200 OK`

---

### Change Password
Change the current user's password.

**Endpoint**: `PUT /api/users/me/password`

**Headers**:
- `Content-Type: application/json`
- `Authorization: Bearer <token>`

**Request Body**:
```json
{
  "currentPassword": "OldPass123!",
  "newPassword": "NewPass456!"
}
```

**Response**: `200 OK`

**Error Response**: `400 Bad Request` (if current password is incorrect)

---

## Receipts API
Available from Phase 8 onwards

### Upload Receipt
Upload a receipt image for an expense.

**Endpoint**: `POST /api/expenses/{id}/receipts`

**Headers**:
- `Content-Type: multipart/form-data`
- `Authorization: Bearer <token>`

**Path Parameters**:
- `id`: Expense ID

**Form Data**:
- `file`: Image file (JPG, PNG, PDF)

**Validation**:
- Max file size: 5MB
- Allowed types: image/jpeg, image/png, application/pdf

**Response**: `201 Created`
```json
{
  "id": 1,
  "filename": "receipt_550e8400-e29b-41d4-a716-446655440000.jpg",
  "originalFilename": "receipt.jpg",
  "contentType": "image/jpeg",
  "size": 245678,
  "uploadedAt": "2024-10-20T10:30:00",
  "expenseId": 1
}
```

---

### Get Receipts for Expense
Get all receipts for a specific expense.

**Endpoint**: `GET /api/expenses/{id}/receipts`

**Headers**:
- `Authorization: Bearer <token>`

**Response**: `200 OK`
```json
[
  {
    "id": 1,
    "filename": "receipt_550e8400-e29b-41d4-a716-446655440000.jpg",
    "originalFilename": "receipt.jpg",
    "contentType": "image/jpeg",
    "size": 245678,
    "uploadedAt": "2024-10-20T10:30:00",
    "downloadUrl": "/api/receipts/1"
  }
]
```

---

### Download Receipt
Download a receipt file.

**Endpoint**: `GET /api/receipts/{id}`

**Headers**:
- `Authorization: Bearer <token>`

**Response**: `200 OK`
- Content-Type: image/jpeg (or appropriate type)
- Binary file content

---

### Delete Receipt
Delete a receipt file.

**Endpoint**: `DELETE /api/receipts/{id}`

**Headers**:
- `Authorization: Bearer <token>`

**Response**: `204 No Content`

---

## Reports API
Available from Phase 8 onwards

### Generate Expense Report
Generate a formatted expense report.

**Endpoint**: `GET /api/reports/expenses`

**Headers**:
- `Authorization: Bearer <token>`

**Query Parameters**:
- `format`: Report format (pdf or csv)
- `startDate`: Start date (YYYY-MM-DD)
- `endDate`: End date (YYYY-MM-DD)
- `categoryId` (optional): Filter by category

**Examples**:
```
GET /api/reports/expenses?format=pdf&startDate=2024-01-01&endDate=2024-12-31
GET /api/reports/expenses?format=csv&startDate=2024-10-01&endDate=2024-10-31&categoryId=1
```

**Response**: `200 OK`
- **PDF**: Content-Type: application/pdf (binary PDF file)
- **CSV**: Content-Type: text/csv (CSV text)

**CSV Format**:
```csv
Date,Description,Amount,Category
2024-10-20,Grocery shopping,45.50,Food & Dining
2024-10-19,Gas,60.00,Transportation
```

---

### Generate Monthly Summary Report
Generate monthly summary report.

**Endpoint**: `GET /api/reports/summary/monthly`

**Headers**:
- `Authorization: Bearer <token>`

**Query Parameters**:
- `year`: Year
- `format`: pdf or csv

**Response**: `200 OK` (PDF or CSV file)

---

## Common Response Codes

### Success Codes
- `200 OK`: Request successful
- `201 Created`: Resource created successfully
- `204 No Content`: Request successful, no content to return

### Client Error Codes
- `400 Bad Request`: Invalid request data or validation error
- `401 Unauthorized`: Missing or invalid authentication token
- `403 Forbidden`: Authenticated but not authorized for this resource
- `404 Not Found`: Resource not found
- `409 Conflict`: Resource conflict (e.g., duplicate username)

### Server Error Codes
- `500 Internal Server Error`: Unexpected server error

---

## Error Response Format

All error responses follow this standard format:

```json
{
  "timestamp": "2024-10-20T10:30:00",
  "status": 400,
  "error": "Bad Request",
  "message": "Detailed error message",
  "path": "/api/expenses",
  "errors": [
    {
      "field": "amount",
      "message": "Amount must be positive"
    },
    {
      "field": "description",
      "message": "Description cannot be blank"
    }
  ]
}
```

**Fields**:
- `timestamp`: ISO 8601 timestamp
- `status`: HTTP status code
- `error`: HTTP status text
- `message`: Human-readable error description
- `path`: Request path that caused the error
- `errors`: Array of field-specific validation errors (for validation errors)

---

## Authentication

### Using JWT Tokens (Phase 7+)

1. **Obtain token**: Call `/api/auth/login` with credentials
2. **Include in requests**: Add `Authorization` header
   ```
   Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
   ```
3. **Token expiration**: Tokens expire after 24 hours (configurable)
4. **Refresh**: Login again to get a new token

### Public Endpoints (No authentication required)
- `POST /api/auth/register`
- `POST /api/auth/login`
- `GET /api/hello` (Phase 1 only)

### Protected Endpoints (Authentication required)
All other endpoints require a valid JWT token from Phase 7 onwards.

---

## Testing the API

### Using cURL

**Create expense**:
```bash
curl -X POST http://localhost:8080/api/expenses \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d '{
    "description": "Lunch",
    "amount": 15.50,
    "date": "2024-10-20",
    "categoryId": 1
  }'
```

**Get all expenses**:
```bash
curl -X GET http://localhost:8080/api/expenses \
  -H "Authorization: Bearer YOUR_TOKEN"
```

### Using Postman

1. Create a new collection for "Expense Tracker API"
2. Set base URL variable: `{{baseUrl}}` = `http://localhost:8080`
3. For authenticated requests, add Authorization header with Bearer token
4. Import this specification to auto-generate requests

---

## Versioning

This API uses URL path versioning:
- Current version: v1 (implied in `/api/` prefix)
- Future versions: `/api/v2/...` (if needed)

---

## Rate Limiting (Optional - Phase 8 extension)

Not implemented in basic phases. Can be added as an extension:
- 100 requests per minute per user
- Returns `429 Too Many Requests` if exceeded

---

## CORS Configuration (Phase 8)

By default, CORS is configured to allow:
- Origins: `http://localhost:3000` (for frontend development)
- Methods: GET, POST, PUT, DELETE
- Headers: Content-Type, Authorization

Configure in `application.properties` for production.

---

## Additional Resources

- **Swagger UI** (Phase 8): `http://localhost:8080/swagger-ui.html`
- **API Docs** (Phase 8): `http://localhost:8080/v3/api-docs`
- **H2 Console** (Phases 3-5): `http://localhost:8080/h2-console`

---

**Last Updated**: 2024-10-20
**API Version**: 1.0
**Spring Boot Version**: 3.x
