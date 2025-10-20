# Expense Tracker REST API

A comprehensive learning project designed to teach Spring Boot REST API development through progressive, hands-on implementation.

## 🎯 Project Goals

This project is designed for students who:
- Know Java fundamentals but have **no experience with REST APIs or Spring Boot**
- Want to learn backend development through practical, incremental steps
- Prefer building real-world applications over theoretical concepts

## 📚 What You'll Learn

By completing this project, you will gain practical experience with:

- **Core Spring Boot concepts**: Controllers, Services, Repositories
- **REST API design**: HTTP methods, status codes, request/response handling
- **Database integration**: H2 (in-memory) → MySQL migration
- **Spring Data JPA**: Entity mapping, repositories, queries
- **Validation & Error Handling**: Input validation, custom exceptions
- **Security**: JWT authentication, Spring Security basics
- **Advanced features**: File uploads, pagination, filtering, reporting
- **Testing**: Unit tests, integration tests, API testing

## 🚀 Learning Approach

This project uses a **phased learning approach** with 8 progressive phases. Each phase:

1. Introduces **one or two new concepts** at a time
2. Builds on what you learned in previous phases
3. Includes step-by-step implementation guides
4. Provides checkpoint exercises to verify understanding
5. Explains common pitfalls and how to avoid them

### Phase Overview

| Phase | Focus | Key Concepts |
|-------|-------|--------------|
| **Phase 1** | Hello World API | Basic REST endpoint, Spring Boot setup |
| **Phase 2** | In-Memory CRUD | HTTP methods, Controller, Service pattern |
| **Phase 3** | Database Basics | H2, JPA entities, repositories |
| **Phase 4** | Validation & Errors | Input validation, exception handling |
| **Phase 5** | Advanced Queries | Categories, filtering, pagination |
| **Phase 6** | Production DB | MySQL setup, date ranges, search |
| **Phase 7** | Security | JWT authentication, Spring Security |
| **Phase 8** | Production Ready | File uploads, reporting, testing |

**Recommended approach**: Complete phases in order. Don't skip ahead—each phase builds essential knowledge for the next.

## 📋 Prerequisites

### Required Knowledge
- Java basics (classes, objects, methods, collections)
- Basic understanding of how web applications work (client-server model)

### Required Software
- **JDK 17 or higher** ([Download](https://adoptium.net/))
- **Maven 3.6+** ([Download](https://maven.apache.org/download.cgi))
- **IDE**: IntelliJ IDEA (recommended) or Eclipse
- **MySQL 8.0+** (required from Phase 6 onwards)
- **Postman** or **cURL** for API testing

### Optional but Helpful
- Git for version control
- MySQL Workbench for database management
- Basic SQL knowledge (you'll learn as you go)

## 🛠️ Getting Started

### Initial Setup (Phase 1)

1. **Clone or download this repository**
   ```bash
   git clone <repository-url>
   cd expense-tracker-api
   ```

2. **Follow the Phase 1 guide**
   - Read `docs/PHASE_1.md` for detailed setup instructions
   - Create your first Spring Boot project
   - Run your first REST API endpoint

3. **Verify your setup**
   - Ensure you can build and run the application
   - Test the API using your browser or Postman

### Phase-by-Phase Progression

After completing Phase 1 setup:

1. Read the phase documentation (`docs/PHASE_*.md`)
2. Implement the features described
3. Test your implementation
4. Complete the checkpoint exercises
5. Move to the next phase

## 📖 Documentation

### Main Documents
- **[FEATURES.md](FEATURES.md)** - Complete feature list organized by phase
- **[docs/API_SPECIFICATION.md](docs/API_SPECIFICATION.md)** - REST API endpoint reference

### Phase Guides
- **[Phase 1: Hello World REST API](docs/PHASE_1.md)** - Your first Spring Boot application
- **[Phase 2: In-Memory CRUD Operations](docs/PHASE_2.md)** - Basic create, read, update, delete
- **[Phase 3: H2 Database Integration](docs/PHASE_3.md)** - Persistent storage with JPA
- **[Phase 4: Validation & Error Handling](docs/PHASE_4.md)** - Robust input validation
- **[Phase 5: Categories & Pagination](docs/PHASE_5.md)** - Advanced queries
- **[Phase 6: MySQL Migration](docs/PHASE_6.md)** - Production database setup
- **[Phase 7: JWT Security](docs/PHASE_7.md)** - User authentication
- **[Phase 8: Production Features](docs/PHASE_8.md)** - Files, reports, testing

## 🎓 Learning Tips

### For Success
- ✅ **Code along**: Type the code yourself rather than copy-pasting
- ✅ **Test frequently**: Verify each feature works before moving on
- ✅ **Read error messages**: They often tell you exactly what's wrong
- ✅ **Use the debugger**: Step through code to understand flow
- ✅ **Experiment**: Try variations of the examples

### Common Mistakes to Avoid
- ❌ Skipping phases to "get to the good stuff"
- ❌ Not testing after each change
- ❌ Copy-pasting without understanding
- ❌ Moving forward when confused (re-read the section instead)

## 🧪 Testing Your API

### Using Postman
1. Download Postman (free)
2. Import the API specification
3. Test each endpoint as you build it

### Using cURL
Each phase guide includes cURL examples for testing endpoints from the command line.

### Using Browser
Simple GET requests can be tested directly in your browser.

## 🤔 Getting Help

### When You're Stuck
1. **Re-read the current section** - The answer is often there
2. **Check the "Common Pitfalls" section** in each phase guide
3. **Review previous phases** - Make sure earlier concepts are solid
4. **Google the error message** - You'll find helpful solutions
5. **Ask for help** - Describe what you tried and what error you got

### Understanding Concepts
- Each phase includes "Why?" sections explaining the reasoning
- Links to official Spring documentation for deeper learning
- Checkpoint exercises to verify your understanding

## 📈 Project Structure (Final)

By the end of Phase 8, your project will have this structure:

```
expense-tracker-api/
├── src/
│   ├── main/
│   │   ├── java/com/example/expensetracker/
│   │   │   ├── controller/      # REST endpoints
│   │   │   ├── service/          # Business logic
│   │   │   ├── repository/       # Database access
│   │   │   ├── model/            # Entity classes
│   │   │   ├── dto/              # Data transfer objects
│   │   │   ├── exception/        # Custom exceptions
│   │   │   ├── security/         # Security config
│   │   │   └── config/           # Application config
│   │   └── resources/
│   │       ├── application.properties
│   │       └── static/           # Uploaded files
│   └── test/                     # Unit & integration tests
├── docs/                         # Phase guides
├── pom.xml                       # Maven dependencies
└── README.md                     # This file
```

## 🌟 What You'll Build

An **Expense Tracker REST API** that allows users to:

- ✨ Create, view, update, and delete expenses
- 📁 Organize expenses by categories
- 🔍 Search and filter expenses by date, amount, category
- 👤 Register and authenticate users
- 📎 Upload receipt images
- 📊 Generate expense reports and summaries
- 📄 Export data in various formats

## 📝 License

This is a learning project. Feel free to use it for educational purposes.

## 🚦 Ready to Start?

**Begin with Phase 1**: Open `docs/PHASE_1.md` and start building your first REST API!

Remember: Learning to code is like learning a musical instrument. Consistent practice with gradual progression beats cramming. Take your time with each phase.

Happy coding! 🎉
