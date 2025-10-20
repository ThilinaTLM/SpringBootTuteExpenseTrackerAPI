# Phase 1: Hello World REST API

**Duration**: 1-2 hours
**Complexity**: ⭐ Beginner

## Learning Objectives

By the end of this phase, you will:
- Understand what a REST API is and why it's useful
- Know how to set up a Spring Boot project
- Create your first REST endpoint
- Run and test a Spring Boot application
- Understand the basic structure of a Spring Boot project

## Prerequisites

Before starting, ensure you have:
- ✅ JDK 17 or higher installed
- ✅ Maven 3.6+ installed
- ✅ An IDE (IntelliJ IDEA recommended, or Eclipse)
- ✅ Basic Java knowledge (classes, methods, objects)

---

## What is a REST API?

### The Restaurant Analogy

Think of a REST API like a restaurant:
- **You (Client)**: A customer who wants food
- **Menu**: Available actions (API endpoints)
- **Waiter**: The API that takes your order and brings food
- **Kitchen**: Your application's business logic
- **Food**: The data you requested

You don't go into the kitchen and cook yourself. Instead, you:
1. Look at the menu (API documentation)
2. Tell the waiter what you want (make a request)
3. Wait for the food (processing)
4. Receive your order (get response)

### Technical Definition

**REST** = **RE**presentational **S**tate **T**ransfer

A REST API is a way for programs to talk to each other over the internet using HTTP (the same protocol your web browser uses).

**Key concepts**:
- **Client**: The program making requests (web browser, mobile app, etc.)
- **Server**: Your Spring Boot application that handles requests
- **Endpoint**: A specific URL that does something (like `/api/hello`)
- **Request**: Asking the server to do something
- **Response**: The server's answer

---

## What is Spring Boot?

**Spring Boot** is a framework that makes it easy to create web applications in Java.

### Why Spring Boot?

Without Spring Boot, creating a web server requires:
- Setting up a web server (Tomcat)
- Configuring how requests are handled
- Writing lots of configuration files
- Managing dependencies

With Spring Boot:
- ✨ Everything is pre-configured
- ✨ Web server included (you just run your app)
- ✨ Easy dependency management
- ✨ Focus on writing business logic, not configuration

Think of it as **Java with superpowers for web development**.

---

## Step 1: Create Your Spring Boot Project

### Option A: Using Spring Initializr (Recommended)

1. **Visit**: https://start.spring.io/

2. **Configure your project**:
   - **Project**: Maven
   - **Language**: Java
   - **Spring Boot**: 3.2.0 (or latest stable)
   - **Project Metadata**:
     - Group: `com.example`
     - Artifact: `expense-tracker`
     - Name: `expense-tracker`
     - Description: `Expense Tracker REST API`
     - Package name: `com.example.expensetracker`
     - Packaging: Jar
     - Java: 17

3. **Add Dependencies** (click "ADD DEPENDENCIES"):
   - **Spring Web**: For creating REST APIs

4. **Click "GENERATE"**: Downloads a ZIP file

5. **Extract the ZIP** to your desired location (e.g., Desktop)

6. **Open in your IDE**:
   - IntelliJ: File → Open → Select the extracted folder
   - Eclipse: File → Import → Existing Maven Project

### Option B: Using IntelliJ IDEA

1. File → New → Project
2. Select "Spring Initializr"
3. Fill in the same details as Option A
4. Add "Spring Web" dependency
5. Click Create

---

## Step 2: Understand the Project Structure

After creating the project, you'll see this structure:

```
expense-tracker/
├── src/
│   ├── main/
│   │   ├── java/com/example/expensetracker/
│   │   │   └── ExpenseTrackerApplication.java    ← Main application file
│   │   └── resources/
│   │       └── application.properties             ← Configuration file
│   └── test/                                      ← Tests (we'll use later)
├── pom.xml                                        ← Maven dependencies
└── README.md
```

### Key Files Explained

**1. `ExpenseTrackerApplication.java`**
This is your application's entry point:
```java
package com.example.expensetracker;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ExpenseTrackerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ExpenseTrackerApplication.class, args);
    }
}
```

**What's happening**:
- `@SpringBootApplication`: Magic annotation that enables Spring Boot
- `main()`: Entry point, starts the application
- `SpringApplication.run()`: Starts the web server

**2. `pom.xml`**
Lists all the libraries (dependencies) your project needs. Spring Initializr already added Spring Web for you.

**3. `application.properties`**
Configuration file (empty for now, we'll add config later)

---

## Step 3: Create Your First REST Controller

A **Controller** is a class that handles HTTP requests.

### Create the Controller Class

1. **Create a new package** (for organization):
   - Right-click on `com.example.expensetracker`
   - New → Package
   - Name it: `controller`

2. **Create a new class**:
   - Right-click on the `controller` package
   - New → Java Class
   - Name it: `HelloController`

3. **Write the code**:

```java
package com.example.expensetracker.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api")
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "Welcome to Expense Tracker API!";
    }
}
```

### Understanding the Code

Let's break down each part:

```java
@RestController
```
- Tells Spring: "This class handles web requests"
- Automatically converts return values to JSON (or text)

```java
@RequestMapping("/api")
```
- Sets the base URL for all endpoints in this class
- All endpoints in this controller start with `/api`

```java
@GetMapping("/hello")
```
- This method handles GET requests to `/api/hello`
- GET = retrieve data (like viewing a webpage)

```java
public String hello()
```
- The method that runs when someone visits `/api/hello`
- Returns a simple string

**Full URL**: `http://localhost:8080/api/hello`
- `localhost` = your computer
- `8080` = default Spring Boot port
- `/api/hello` = your endpoint

---

## Step 4: Run Your Application

### Using IntelliJ IDEA
1. Open `ExpenseTrackerApplication.java`
2. Click the green play button ▶️ next to the class name
3. Or: Right-click → Run 'ExpenseTrackerApplication'

### Using Eclipse
1. Right-click on the project
2. Run As → Java Application
3. Select `ExpenseTrackerApplication`

### Using Command Line
```bash
cd expense-tracker
mvn spring-boot:run
```

### What to Expect

You should see output like this:
```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v3.2.0)

2024-10-20 10:30:00.000  INFO ... : Starting ExpenseTrackerApplication
2024-10-20 10:30:01.234  INFO ... : Tomcat started on port(s): 8080 (http)
2024-10-20 10:30:01.245  INFO ... : Started ExpenseTrackerApplication in 2.5 seconds
```

**Key line**: `Tomcat started on port(s): 8080`
- Your application is now running!
- It's listening for requests on port 8080

---

## Step 5: Test Your API

### Method 1: Web Browser (Easiest)

1. Open your web browser
2. Go to: `http://localhost:8080/api/hello`
3. You should see: `Welcome to Expense Tracker API!`

🎉 **Congratulations! You created your first REST API!**

### Method 2: Using cURL (Command Line)

Open a terminal and run:
```bash
curl http://localhost:8080/api/hello
```

Output:
```
Welcome to Expense Tracker API!
```

### Method 3: Using Postman (Professional Tool)

1. **Download Postman** (free): https://www.postman.com/downloads/
2. **Install and open** Postman
3. **Create a new request**:
   - Method: GET
   - URL: `http://localhost:8080/api/hello`
   - Click "Send"
4. **See the response** in the bottom panel

**Why use Postman?**
- Browser only works for GET requests
- Later, you'll need POST, PUT, DELETE (browser can't do these easily)
- Postman makes testing APIs much easier

---

## Step 6: Make It More Interesting

Let's return JSON instead of plain text.

### Create a Response Class

1. **Create a new package** called `dto` (Data Transfer Object)
2. **Create a class** called `MessageResponse`:

```java
package com.example.expensetracker.dto;

public class MessageResponse {
    private String message;
    private String timestamp;

    // Constructor
    public MessageResponse(String message, String timestamp) {
        this.message = message;
        this.timestamp = timestamp;
    }

    // Getters and Setters
    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    public String getTimestamp() {
        return timestamp;
    }

    public void setTimestamp(String timestamp) {
        this.timestamp = timestamp;
    }
}
```

### Update Your Controller

Modify `HelloController.java`:

```java
package com.example.expensetracker.controller;

import com.example.expensetracker.dto.MessageResponse;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

@RestController
@RequestMapping("/api")
public class HelloController {

    @GetMapping("/hello")
    public MessageResponse hello() {
        String timestamp = LocalDateTime.now()
            .format(DateTimeFormatter.ISO_LOCAL_DATE_TIME);

        return new MessageResponse(
            "Welcome to Expense Tracker API!",
            timestamp
        );
    }
}
```

### Test Again

Visit: `http://localhost:8080/api/hello`

**Now you get JSON**:
```json
{
  "message": "Welcome to Expense Tracker API!",
  "timestamp": "2024-10-20T10:30:00"
}
```

**What changed?**
- Spring Boot automatically converted your Java object to JSON
- This is called **serialization**
- JSON is the standard format for REST APIs

---

## Understanding HTTP and REST

### What Just Happened?

1. **You made a request**: Browser sent `GET http://localhost:8080/api/hello`
2. **Spring Boot received it**: Looked for a method with `@GetMapping("/hello")`
3. **Your method ran**: `hello()` created a `MessageResponse`
4. **Spring converted to JSON**: Your object → JSON
5. **Response sent back**: Browser displayed the JSON

### HTTP Request Structure

```
GET /api/hello HTTP/1.1
Host: localhost:8080
Accept: application/json
```

- **GET**: The HTTP method (asking to retrieve data)
- **/api/hello**: The path
- **Host**: Where to send the request

### HTTP Response Structure

```
HTTP/1.1 200 OK
Content-Type: application/json

{
  "message": "Welcome to Expense Tracker API!",
  "timestamp": "2024-10-20T10:30:00"
}
```

- **200 OK**: Status code (success)
- **Content-Type**: What kind of data (JSON)
- **Body**: The actual data

---

## Common Issues and Solutions

### Issue 1: Port 8080 Already in Use

**Error**: `Port 8080 was already in use`

**Solution**: Change the port in `application.properties`:
```properties
server.port=8081
```
Then use `http://localhost:8081/api/hello`

### Issue 2: 404 Not Found

**Error**: Visiting URL shows 404

**Checklist**:
- ✅ Is the application running? Check console
- ✅ Is the URL correct? Should be `/api/hello`
- ✅ Did you save the file? Save and restart
- ✅ Is the class in the correct package?

### Issue 3: Application Won't Start

**Error**: Various startup errors

**Solutions**:
- Check Java version: `java -version` (should be 17+)
- Check Maven: `mvn -version`
- Reimport Maven project: Right-click → Maven → Reload Project
- Check for typos in annotations

### Issue 4: Changes Not Showing

**Problem**: Modified code but seeing old response

**Solution**:
1. Stop the application (red square button)
2. Restart it (green play button)
3. Refresh your browser

---

## Checkpoint Exercises

Test your understanding by completing these exercises:

### Exercise 1: Add a New Endpoint
Create a `/api/status` endpoint that returns:
```json
{
  "status": "running",
  "version": "1.0.0"
}
```

<details>
<summary>Show Solution</summary>

```java
@GetMapping("/status")
public Map<String, String> status() {
    Map<String, String> response = new HashMap<>();
    response.put("status", "running");
    response.put("version", "1.0.0");
    return response;
}
```

Add this import:
```java
import java.util.HashMap;
import java.util.Map;
```
</details>

### Exercise 2: Parameterized Endpoint
Create a `/api/greet/{name}` endpoint that returns:
```json
{
  "greeting": "Hello, John!",
  "timestamp": "2024-10-20T10:30:00"
}
```

<details>
<summary>Show Solution</summary>

```java
@GetMapping("/greet/{name}")
public Map<String, String> greet(@PathVariable String name) {
    Map<String, String> response = new HashMap<>();
    response.put("greeting", "Hello, " + name + "!");
    response.put("timestamp", LocalDateTime.now()
        .format(DateTimeFormatter.ISO_LOCAL_DATE_TIME));
    return response;
}
```

Add import:
```java
import org.springframework.web.bind.annotation.PathVariable;
```

Test: `http://localhost:8080/api/greet/John`
</details>

### Exercise 3: Info Endpoint
Create a `/api/info` endpoint that returns project information:
```json
{
  "projectName": "Expense Tracker API",
  "description": "A REST API for tracking expenses",
  "author": "Your Name"
}
```

<details>
<summary>Show Solution</summary>

Create a `ProjectInfo` class in the `dto` package:
```java
package com.example.expensetracker.dto;

public class ProjectInfo {
    private String projectName;
    private String description;
    private String author;

    public ProjectInfo(String projectName, String description, String author) {
        this.projectName = projectName;
        this.description = description;
        this.author = author;
    }

    // Getters and setters...
}
```

Add to controller:
```java
@GetMapping("/info")
public ProjectInfo info() {
    return new ProjectInfo(
        "Expense Tracker API",
        "A REST API for tracking expenses",
        "Your Name"
    );
}
```
</details>

---

## Key Takeaways

✅ **REST API**: A way for programs to communicate over HTTP
✅ **Spring Boot**: Framework that simplifies Java web development
✅ **@RestController**: Marks a class as a REST endpoint handler
✅ **@GetMapping**: Handles HTTP GET requests
✅ **Endpoints**: URLs that perform specific actions
✅ **JSON**: Standard data format for REST APIs
✅ **localhost:8080**: Default location for Spring Boot apps

---

## What's Next?

In **Phase 2**, you'll learn:
- Other HTTP methods (POST, PUT, DELETE)
- Creating, updating, and deleting data
- Separating concerns (Controller vs Service)
- Building a real CRUD API for expenses

---

## Additional Resources

### Official Documentation
- [Spring Boot Getting Started](https://spring.io/guides/gs/spring-boot/)
- [Building REST Services](https://spring.io/guides/tutorials/rest/)

### Video Tutorials
- Search YouTube: "Spring Boot REST API tutorial for beginners"
- Look for: "Spring Boot Hello World"

### Practice
- Try creating more endpoints
- Experiment with returning different data
- Break things and fix them (best way to learn!)

---

## Glossary

- **API**: Application Programming Interface - a way for programs to talk to each other
- **REST**: REpresentational State Transfer - a style of API design
- **HTTP**: HyperText Transfer Protocol - how web browsers and servers communicate
- **JSON**: JavaScript Object Notation - a data format
- **Controller**: A class that handles web requests
- **Endpoint**: A specific URL in your API
- **Maven**: A tool for managing Java project dependencies
- **Dependency**: A library your project needs
- **Port**: A number that identifies a specific service (8080 for Spring Boot)
- **localhost**: Your own computer
- **Annotation**: Java feature using `@` that adds behavior to code

---

## Troubleshooting Tips

1. **Read error messages carefully** - They usually tell you what's wrong
2. **Google error messages** - Add "Spring Boot" to your search
3. **Check for typos** - Java is case-sensitive
4. **Make sure you saved files** - Ctrl+S (or Cmd+S on Mac)
5. **Restart the application** after changes
6. **Use the debugger** - Set breakpoints to see what's happening

---

**Congratulations on completing Phase 1!** 🎉

You've created your first REST API and understand the basics of Spring Boot. Take a break, then move on to Phase 2 when you're ready to build something more complex.

**Next**: [Phase 2: In-Memory CRUD Operations](PHASE_2.md)
