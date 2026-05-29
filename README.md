# Job_Portal_App

# Spring Job Portal — REST API

A full-featured **Job Portal REST API** built with Spring Boot 3, demonstrating enterprise-grade patterns including Spring Security, Spring Data JPA, and Aspect-Oriented Programming (AOP). Designed as a backend service for a React or any other frontend application.

---

## Table of Contents

- [Overview](#overview)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Features](#features)
- [Architecture](#architecture)
- [API Endpoints](#api-endpoints)
- [Security](#security)
- [AOP — Cross-Cutting Concerns](#aop--cross-cutting-concerns)
- [Database Schema](#database-schema)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Database Setup](#database-setup)
  - [Running the Application](#running-the-application)
- [Configuration](#configuration)
- [Sample Data](#sample-data)
- [Future Improvements](#future-improvements)

---

## Overview

The Spring Job Portal is a RESTful backend application that allows users to register, authenticate, and manage job postings. It supports full CRUD operations for job posts and a keyword-based search across job profiles and descriptions.

The project highlights several real-world Spring Boot patterns:

- Stateless REST API secured with **HTTP Basic Auth** (JWT-ready)
- **BCrypt** password hashing for secure user registration
- **AOP** for logging, performance monitoring, and input validation
- **Spring Data JPA** with PostgreSQL for data persistence
- **CORS** configuration for React frontend integration

---

## Tech Stack

| Layer | Technology |
|---|---|
| Language | Java 17 |
| Framework | Spring Boot 3.3.1 |
| Security | Spring Security 6 |
| ORM | Spring Data JPA / Hibernate |
| Database | PostgreSQL |
| Build Tool | Maven |
| Boilerplate Reduction | Lombok |
| AOP | Spring AOP |
| Testing | Spring Boot Test, Spring Security Test |

---

## Project Structure

```
src/
└── main/
    ├── java/com/shivam/SpringJobPortalRevision/
    │   ├── SpringJobPortalRevisionApplication.java   # Entry point
    │   ├── aop/
    │   │   ├── LoggingAspect.java                    # Before/After/AfterThrowing/AfterReturning advice
    │   │   ├── PerformanceMonitorAspect.java          # Around advice — method execution timing
    │   │   └── ValidationAspect.java                 # Around advice — input sanitization
    │   ├── config/
    │   │   └── SecurityConfig.java                   # Spring Security filter chain, auth provider
    │   ├── controllers/
    │   │   ├── JobRestController.java                # Job CRUD + search endpoints
    │   │   └── UserController.java                   # User registration endpoint
    │   ├── model/
    │   │   ├── JobPost.java                          # Job post entity
    │   │   ├── User.java                             # User entity
    │   │   └── UserPrinciple.java                    # UserDetails wrapper for Spring Security
    │   ├── repo/
    │   │   ├── JobRepo.java                          # JPA repository for JobPost
    │   │   └── UserRepo.java                         # JPA repository for User
    │   └── service/
    │       ├── JobService.java                       # Business logic for job operations
    │       ├── MyUserDetailsService.java             # UserDetailsService impl for auth
    │       └── UserService.java                      # User registration with BCrypt encoding
    └── resources/
        └── application.properties                    # App, DB, JPA, and server config
```

---

## Features

- **User Registration** — Register with a username and password; password is BCrypt-encoded at strength 12 before persistence.
- **Secure Authentication** — All API endpoints protected via HTTP Basic Authentication backed by a PostgreSQL user store.
- **Job Post CRUD** — Create, read, update, and delete job postings with full entity persistence.
- **Keyword Search** — Full-text search across job profile name and job description fields simultaneously.
- **Sample Data Loader** — Single endpoint to seed the database with 5 realistic job postings for quick testing.
- **AOP Logging** — Automatic structured logging of service method calls, return values, and exceptions — zero boilerplate in business logic.
- **Performance Monitoring** — Around-advice transparently measures and logs execution time for all job service methods.
- **Input Validation** — AOP-based sanitization intercepts `getJob(postId)` and auto-corrects negative IDs before they reach the database.
- **CORS Ready** — Whitelists `http://localhost:3000` for seamless React frontend development.

---

## Architecture

```
HTTP Request
     │
     ▼
┌─────────────────────────────────┐
│        Spring Security          │  ← Authenticates every request (HTTP Basic)
│  (SecurityFilterChain)          │
└────────────────┬────────────────┘
                 │
                 ▼
┌─────────────────────────────────┐
│           AOP Layer             │  ← Logging, Performance, Validation (transparent)
└────────────────┬────────────────┘
                 │
                 ▼
┌─────────────────────────────────┐
│         Controller Layer        │  ← Routes HTTP verbs to service calls
│  JobRestController / UserCtrl   │
└────────────────┬────────────────┘
                 │
                 ▼
┌─────────────────────────────────┐
│          Service Layer          │  ← Business logic and orchestration
│    JobService / UserService     │
└────────────────┬────────────────┘
                 │
                 ▼
┌─────────────────────────────────┐
│        Repository Layer         │  ← Spring Data JPA (auto-generated queries)
│      JobRepo / UserRepo         │
└────────────────┬────────────────┘
                 │
                 ▼
┌─────────────────────────────────┐
│          PostgreSQL DB          │
└─────────────────────────────────┘
```

---

## API Endpoints

All endpoints require HTTP Basic Authentication unless otherwise noted.

### Job Endpoints

| Method | URL | Description | Auth Required |
|---|---|---|---|
| `GET` | `/load` | Seed DB with 5 sample job posts | Yes |
| `GET` | `/jobPosts` | Get all job posts | Yes |
| `GET` | `/jobPost/{postId}` | Get a single job post by ID | Yes |
| `GET` | `/jobPosts/keyword/{keyword}` | Search jobs by keyword (profile + description) | Yes |
| `POST` | `/jobPost` | Create a new job post | Yes |
| `PUT` | `/jobPost` | Update an existing job post | Yes |
| `DELETE` | `/jobPost/{postId}` | Delete a job post by ID | Yes |

### User Endpoints

| Method | URL | Description | Auth Required |
|---|---|---|---|
| `POST` | `/register` | Register a new user | Yes* |

> *The `/register` endpoint currently requires authentication. This is a known limitation — see [Future Improvements](#future-improvements).

### Request / Response Examples

**POST /jobPost** — Create a job post

```json
// Request Body
{
  "postProfile": "Backend Developer",
  "postDesc": "Looking for an experienced backend developer with Spring Boot skills",
  "reqExperience": 3,
  "postTechStack": ["Java", "Spring Boot", "PostgreSQL", "Docker"]
}

// Response: 200 OK — returns persisted JobPost with assigned ID
{
  "postId": 6,
  "postProfile": "Backend Developer",
  "postDesc": "Looking for an experienced backend developer with Spring Boot skills",
  "reqExperience": 3,
  "postTechStack": ["Java", "Spring Boot", "PostgreSQL", "Docker"]
}
```

**GET /jobPosts/keyword/java** — Keyword search

```json
// Response: 200 OK — array of matching job posts
[
  {
    "postId": 1,
    "postProfile": "Java Developer",
    "postDesc": "Develop and maintain Java-based applications...",
    "reqExperience": 2,
    "postTechStack": ["Core Java", "J2EE", "Spring Boot", "Hibernate"]
  }
]
```

---

## Security

### Authentication Flow

```
POST /register  →  UserService.saveUser()  →  BCrypt(strength=12)  →  users table
GET  /jobPosts  →  SecurityFilter  →  DaoAuthenticationProvider
                       │
                       ▼
               MyUserDetailsService.loadUserByUsername()
                       │
                       ▼
               Compares BCrypt hash  →  grants UserPrinciple authority "USER"
```

### Security Configuration Highlights

- **Password Encoding** — BCryptPasswordEncoder with cost factor 12 (computationally expensive to brute-force).
- **CSRF Disabled** — Appropriate for stateless REST APIs; the client manages auth headers, not cookies.
- **Stateless Sessions** — `SessionCreationPolicy.STATELESS` ensures no server-side session is created or maintained.
- **HTTP Basic** — Credentials are sent as Base64-encoded `Authorization` header on every request.
- **All Endpoints Authenticated** — `anyRequest().authenticated()` enforces auth globally.

> **Note on Passwords in Config:** The current `application.properties` contains database credentials in plaintext. In a production setup, use environment variables or a secrets manager (e.g., AWS Secrets Manager, HashiCorp Vault).

---

## AOP — Cross-Cutting Concerns

Three aspects keep business logic clean by handling observability and validation transparently:

### LoggingAspect

Pointcut targets all methods in `JobService`.

| Advice Type | Trigger | Action |
|---|---|---|
| `@Before` | Any `JobService` method | Logs method name before execution |
| `@After` | `getJob()` or `updateJob()` | Logs method name after execution |
| `@AfterThrowing` | Any exception in `JobService` | Logs the exception message |
| `@AfterReturning` | Successful return from any method | Logs the returned value |

### PerformanceMonitorAspect

`@Around` advice wraps every `JobService` method call, records `System.currentTimeMillis()` before and after, and logs execution duration in milliseconds. No changes to service code needed.

### ValidationAspect

`@Around` advice intercepts `JobService.getJob(postId)`. If the caller passes a **negative** `postId`, the aspect converts it to its absolute value before the actual service method executes — preventing invalid DB lookups silently.

---

## Database Schema

Hibernate auto-generates and updates the schema on startup (`ddl-auto=update`).

### `job_post` table

| Column | Type | Constraints |
|---|---|---|
| `post_id` | INTEGER | PRIMARY KEY, auto-increment |
| `post_profile` | VARCHAR | NOT NULL |
| `post_desc` | VARCHAR | — |
| `req_experience` | INTEGER | — |
| `post_tech_stack` | VARCHAR[] / TEXT | — |

### `users` table

| Column | Type | Constraints |
|---|---|---|
| `id` | INTEGER | PRIMARY KEY, auto-increment |
| `username` | VARCHAR | UNIQUE |
| `password` | VARCHAR | BCrypt hash |

---

## Getting Started

### Prerequisites

- **Java 17+** — [Download JDK](https://adoptium.net/)
- **Maven 3.8+** — [Download Maven](https://maven.apache.org/download.cgi)
- **PostgreSQL 12+** — [Download PostgreSQL](https://www.postgresql.org/download/)

### Database Setup

1. Connect to PostgreSQL as a superuser:
   ```sql
   psql -U postgres
   ```

2. Create the database:
   ```sql
   CREATE DATABASE shivam;
   ```

3. Verify the database exists:
   ```sql
   \l
   ```

> Hibernate will auto-create the required tables on first run (`ddl-auto=update`).

### Running the Application

1. **Clone the repository:**
   ```bash
   git clone https://github.com/your-username/SpringJobPortalRevision.git
   cd SpringJobPortalRevision
   ```

2. **Update database credentials** in [src/main/resources/application.properties](src/main/resources/application.properties):
   ```properties
   spring.datasource.url=jdbc:postgresql://localhost:5432/shivam
   spring.datasource.username=postgres
   spring.datasource.password=YOUR_PASSWORD
   ```

3. **Build and run:**
   ```bash
   mvn spring-boot:run
   ```

4. **Verify the server is running:**
   ```
   Server started on port 8081
   ```

5. **Test with curl:**
   ```bash
   # Register a user
   curl -X POST http://localhost:8081/register \
     -H "Content-Type: application/json" \
     -d '{"username": "admin", "password": "secret"}'

   # Load sample data (requires auth)
   curl -u admin:secret http://localhost:8081/load

   # Get all jobs
   curl -u admin:secret http://localhost:8081/jobPosts
   ```

---

## Configuration

Key properties in [src/main/resources/application.properties](src/main/resources/application.properties):

```properties
spring.application.name=SpringJobPortalRevision

# Server
server.port=8081

# Database
spring.datasource.url=jdbc:postgresql://localhost:5432/shivam
spring.datasource.username=postgres
spring.datasource.password=0000

# JPA / Hibernate
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true

# Session cookie policy
server.servlet.session.cookie.same-site=strict
```

---

## Sample Data

Calling `GET /load` seeds the database with these five job postings:

| Profile | Experience | Tech Stack |
|---|---|---|
| Java Developer | 2 years | Core Java, J2EE, Spring Boot, Hibernate |
| Frontend Developer | 3 years | HTML, CSS, JavaScript, React |
| Data Scientist | 4 years | Python, ML, TensorFlow, Deep Learning |
| Network Engineer | 5 years | Networking, Security, Routing, Switching |
| Mobile Developer | 3 years | Android, iOS, React Native, Flutter |

---

## Future Improvements

- [ ] **JWT Authentication** — Infrastructure exists (`/login` endpoint is stubbed); replace HTTP Basic with a JWT token flow for true statelessness.
- [ ] **Open `/register` Endpoint** — Remove the authentication requirement from `/register` so new users can self-onboard.
- [ ] **Input Validation** — Add `@Valid` + Bean Validation (`@NotBlank`, `@Min`) on request bodies.
- [ ] **Pagination** — Add `Pageable` support to `GET /jobPosts` for large result sets.
- [ ] **Role-Based Access Control** — Introduce `ADMIN` and `USER` roles; restrict destructive operations to admins.
- [ ] **Externalize Secrets** — Move database credentials to environment variables or a secrets manager.
- [ ] **Docker Compose** — Add a `docker-compose.yml` to spin up the API and PostgreSQL together with a single command.
- [ ] **Swagger / OpenAPI** — Add `springdoc-openapi` for interactive API documentation at `/swagger-ui.html`.
- [ ] **Unit & Integration Tests** — Expand test coverage for service and controller layers.

---

## Author

**Shivam Jain**

---

## License

This project is open-source and available under the [MIT License](LICENSE).
