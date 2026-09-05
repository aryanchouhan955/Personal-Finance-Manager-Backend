# 💰 Personal Finance Manager API

[![Build & Test](https://img.shields.io/badge/Build-Passing-brightgreen?style=for-the-badge&logo=github-actions)](https://github.com/aryanchouhan955/Personal-Finance-Manager-Backend)
[![Java](https://img.shields.io/badge/Java-17%2B-orange?style=for-the-badge&logo=openjdk)](https://www.oracle.com/java/)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.4.13-green?style=for-the-badge&logo=springboot)](https://spring.io/projects/spring-boot)
[![Code Coverage](https://img.shields.io/badge/Coverage-80%25%2B-blue?style=for-the-badge&logo=jacoco)](https://www.jacoco.org/jacoco/)
[![Docker](https://img.shields.io/badge/Docker-Ready-2496ED?style=for-the-badge&logo=docker)](https://www.docker.com/)

An enterprise-grade **Personal Finance Manager RESTful Backend API** built with **Java 17** and **Spring Boot 3**. The application allows users to manage income, track expenses, customize budget categories, set and monitor savings goals, and analyze monthly/yearly financial performance with data isolation and session-based security.

---

## ✨ Key Features

### 🔒 User Management & Authentication
* **User Registration & Validation**: Mandatory email-formatted username, full name, phone number, and password validation.
* **Session-Based Authentication**: Secure cookie management (`JSESSIONID`) with Spring Security protection across endpoints.
* **Data Isolation**: Strict multi-tenant data isolation ensuring users only access their own records.

### 💸 Transaction Management
* **Full CRUD Capabilities**: Create, view, update, and delete income/expense transactions.
* **Smart Filtering**: Query transactions by date range (`startDate`, `endDate`), `categoryId`, and transaction type (`INCOME`/`EXPENSE`).
* **Validation Guards**: Prevents future-dated transactions and negative amount values.

### 🏷️ Category Management
* **Built-in System Categories**: Pre-seeded immutable categories:
  * `INCOME`: `Salary`
  * `EXPENSE`: `Food`, `Rent`, `Transportation`, `Entertainment`, `Healthcare`, `Utilities`
* **Custom Categories**: User-defined unique categories tied to income or expense types.
* **Referential Integrity**: Categories currently referenced by transactions are protected from deletion.

### 🎯 Savings Goals
* **Goal Setup**: Define target financial goals with target amount and target date constraints.
* **Real-time Progress Calculation**: Automatically calculates progress as `Total Income - Total Expenses` recorded since the goal's start date.
* **Metrics**: Provides completed percentage and remaining target amount in real time.

### 📊 Reports & Analytics
* **Monthly Financial Breakdown**: Total income & expense grouped by category + net savings calculation for any selected month/year.
* **Yearly Overview**: Aggregated 12-month summary for long-term financial planning.

---

## 🛠️ Technology Stack & Architecture

| Component | Technology / Library |
| :--- | :--- |
| **Language** | Java 17 |
| **Framework** | Spring Boot 3.4.13 (`spring-boot-starter-web`) |
| **Security** | Spring Security 6 (`spring-boot-starter-security`) |
| **Validation** | Jakarta Bean Validation (`spring-boot-starter-validation`) |
| **Monitoring** | Spring Boot Actuator (`spring-boot-starter-actuator`) |
| **Testing** | JUnit 5, Mockito, Spring Security Test |
| **Code Coverage** | JaCoCo Maven Plugin (80%+ mandatory threshold) |
| **Build & Deploy** | Maven, Docker, Render PaaS |

---

## 🏗️ System Architecture

The project adheres to **Clean Layered Architecture** with strict boundary separation:

```
    ┌─────────────────────────────────────────┐
    │          Client App / HTTP API          │
    └────────────────────┬────────────────────┘
                         │ (Cookies / REST)
    ┌────────────────────▼────────────────────┐
    │       Controller Layer (REST DTOs)      │
    └────────────────────┬────────────────────┘
                         │
    ┌────────────────────▼────────────────────┐
    │      Service Layer (Business Logic)     │
    └────────────────────┬────────────────────┘
                         │
    ┌────────────────────▼────────────────────┐
    │    Repository / Data Layer (Entities)   │
    └─────────────────────────────────────────┘
```

* **Controller Layer**: Handles REST contracts, Request/Response DTO mapping, input validation, and HTTP status codes.
* **Service Layer**: Houses core business rules, transactional boundaries, data isolation enforcement, and report aggregation math.
* **Repository Layer**: Manages persistent domain models and query logic.
* **Global Exception Handler**: Intercepts domain errors to construct uniform RFC 7807 error responses with proper HTTP status codes.

---

## 📖 API Specification

### 1. User Authentication

#### Register User
```http
POST /api/auth/register
```
* **Request Body:**
  ```json
  {
    "username": "user@example.com",
    "password": "password123",
    "fullName": "John Doe",
    "phoneNumber": "+1234567890"
  }
  ```
* **Response (201 Created):**
  ```json
  {
    "message": "User registered successfully",
    "userId": 1
  }
  ```

#### Login
```http
POST /api/auth/login
```
* **Request Body:**
  ```json
  {
    "username": "user@example.com",
    "password": "password123"
  }
  ```
* **Response (200 OK):**
  ```json
  {
    "message": "Login successful"
  }
  ```
  *(Sets HttpOnly Session Cookie)*

#### Logout
```http
POST /api/auth/logout
```
* **Response (200 OK):**
  ```json
  {
    "message": "Logout successful"
  }
  ```

---

### 2. Transaction Management

#### Create Transaction
```http
POST /api/transactions
```
* **Request Body:**
  ```json
  {
    "amount": 50000.00,
    "date": "2026-09-01",
    "category": "Salary",
    "description": "September Monthly Salary"
  }
  ```
* **Response (201 Created):**
  ```json
  {
    "id": 1,
    "amount": 50000.00,
    "date": "2026-09-01",
    "category": "Salary",
    "description": "September Monthly Salary",
    "type": "INCOME"
  }
  ```

#### Get Transactions
```http
GET /api/transactions?startDate=2026-09-01&endDate=2026-09-30&categoryId=1
```
* **Response (200 OK):**
  ```json
  {
    "transactions": [
      {
        "id": 1,
        "amount": 50000.00,
        "date": "2026-09-01",
        "category": "Salary",
        "description": "September Monthly Salary",
        "type": "INCOME"
      }
    ]
  }
  ```

#### Update Transaction
```http
PUT /api/transactions/{id}
```
* **Request Body:**
  ```json
  {
    "amount": 55000.00,
    "description": "Updated September Salary"
  }
  ```

#### Delete Transaction
```http
DELETE /api/transactions/{id}
```

---

### 3. Category Management

#### Get All Categories
```http
GET /api/categories
```
* **Response (200 OK):**
  ```json
  {
    "categories": [
      { "name": "Salary", "type": "INCOME", "isCustom": false },
      { "name": "Food", "type": "EXPENSE", "isCustom": false },
      { "name": "Freelance", "type": "INCOME", "isCustom": true }
    ]
  }
  ```

#### Create Custom Category
```http
POST /api/categories
```
* **Request Body:**
  ```json
  {
    "name": "SideBusinessIncome",
    "type": "INCOME"
  }
  ```

#### Delete Custom Category
```http
DELETE /api/categories/{name}
```

---

### 4. Savings Goals

#### Create Goal
```http
POST /api/goals
```
* **Request Body:**
  ```json
  {
    "goalName": "Emergency Fund",
    "targetAmount": 5000.00,
    "targetDate": "2027-01-01",
    "startDate": "2026-09-01"
  }
  ```
* **Response (201 Created):**
  ```json
  {
    "id": 1,
    "goalName": "Emergency Fund",
    "targetAmount": 5000.00,
    "targetDate": "2027-01-01",
    "startDate": "2026-09-01",
    "currentProgress": 1000.00,
    "progressPercentage": 20.0,
    "remainingAmount": 4000.00
  }
  ```

#### Get All Goals
```http
GET /api/goals
```

---

### 5. Reports & Analytics

#### Monthly Financial Report
```http
GET /api/reports/monthly/{year}/{month}
```
* **Response (200 OK):**
  ```json
  {
    "month": 9,
    "year": 2026,
    "totalIncome": {
      "Salary": 50000.00
    },
    "totalExpenses": {
      "Food": 400.00,
      "Rent": 12000.00
    },
    "netSavings": 37600.00
  }
  ```

#### Yearly Financial Report
```http
GET /api/reports/yearly/{year}
```

---

## 🚦 Error Handling

The API standardizes error responses with descriptive diagnostic messages:

| HTTP Status | Scenario / Description |
| :--- | :--- |
| `400 Bad Request` | Validation failure (e.g. negative amounts, malformed JSON, future dates) |
| `401 Unauthorized` | Invalid credentials, missing or expired session cookie |
| `403 Forbidden` | Data isolation guard violation (attempting to access another user's resources) |
| `404 Not Found` | Requested resource ID does not exist |
| `409 Conflict` | Conflict constraint (e.g. duplicate category names) |

---

## 🚀 Getting Started

### Prerequisites
* **Java**: JDK 17 or higher installed (`java -version`)
* **Maven**: Version 3.8+ (`mvn -version`)
* **Docker** *(Optional)*: For containerized testing

### Local Installation & Run

1. **Clone the Repository:**
   ```bash
   git clone https://github.com/aryanchouhan955/Personal-Finance-Manager-Backend.git
   cd Personal-Finance-Manager-Backend
   ```

2. **Build the Project:**
   ```bash
   mvn clean package
   ```

3. **Run Application:**
   ```bash
   mvn spring-boot:run
   ```
   The backend API will start at: `http://localhost:8080`

### Docker Setup

Build and execute via Docker:

```bash
# Build Docker image
docker build -t personal-finance-manager .

# Run container
docker run -p 8080:8080 personal-finance-manager
```

---

## 🧪 Testing & Quality Assurance

### Unit & Integration Testing
Execute the JUnit test suite:
```bash
mvn test
```

### Code Coverage Verification
JaCoCo enforces an **80% minimum code coverage threshold**:
```bash
mvn verify
```
View HTML coverage report at: `target/site/jacoco/index.html`

### End-to-End Automated Test Script
To run the automated test suite against a running server instance:
```bash
bash financial_manager_tests.sh http://localhost:8080/api
```

---

## 🌐 Deployment

This application is configured for continuous deployment on **Render PaaS** via `render.yaml` and `Dockerfile`.

* **Render Service Name**: `personal-finance-manager`
* **Health Check Endpoint**: `/actuator/health`

---

## 📄 License

Distributed under the MIT License. See `LICENSE` for details.
