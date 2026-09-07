# Order Management Integration using MuleSoft

A MuleSoft-based **Order Management Integration API** that validates customer orders, checks product availability and stock through an external Product API, calculates the order amount with discount rules, creates the order through an external Order Management API, and sends a customer notification through a Notification API.

The project demonstrates **API-led integration, DataWeave transformations, HTTP integrations, error handling, retry mechanisms, logging, correlation IDs, configuration management, and MUnit testing**.

---

## 📌 Project Overview

The Order Management Integration API acts as the central orchestration layer between the client application and multiple external systems.

### Business Flow

```text
                    ┌──────────────────────┐
                    │      Client/App      │
                    └──────────┬───────────┘
                               │
                               │ POST /api/orders
                               ▼
                 ┌────────────────────────────┐
                 │   Order Management API     │
                 │        MuleSoft            │
                 └─────────────┬──────────────┘
                               │
                     Validate Order Request
                               │
                               ▼
                 ┌────────────────────────────┐
                 │      Product API           │
                 │       Port 8082             │
                 └─────────────┬──────────────┘
                               │
                    Product + Price + Stock
                               │
                               ▼
                 ┌────────────────────────────┐
                 │     Business Logic         │
                 │                            │
                 │ • Validate Products         │
                 │ • Check Stock               │
                 │ • Calculate Subtotal        │
                 │ • Apply Discount            │
                 │ • Calculate Final Amount    │
                 └─────────────┬──────────────┘
                               │
                               ▼
                 ┌────────────────────────────┐
                 │    Order Management API     │
                 │       Port 8083              │
                 └─────────────┬──────────────┘
                               │
                         Order Created
                               │
                               ▼
                 ┌────────────────────────────┐
                 │     Notification API        │
                 │       Port 8084              │
                 └─────────────┬──────────────┘
                               │
                         Notification Sent
                               │
                               ▼
                    ┌──────────────────────┐
                    │    Client Response   │
                    └──────────────────────┘
```

---

# 🚀 Features

* RESTful Order Management API
* RAML-based API specification
* APIKit-based API implementation
* Customer order validation
* Product availability validation
* Product existence validation
* Stock availability validation
* Order amount calculation
* Automatic 10% discount for orders above ₹10,000
* Integration with external Product API
* Integration with external Order Management API
* Integration with Notification API
* DataWeave transformations
* Correlation ID generation
* Structured logging
* Standardized error responses
* HTTP error handling
* External service failure handling
* Retry mechanism for temporary failures
* Externalized configuration
* Idempotency support
* MUnit test cases
* Success and failure scenario testing
* Production-style integration architecture

---

# 🛠️ Technology Stack

| Technology               | Purpose                      |
| ------------------------ | ---------------------------- |
| MuleSoft Anypoint Studio | Integration development      |
| Mule Runtime             | Application runtime          |
| RAML 1.0                 | API specification            |
| APIKit                   | REST API routing             |
| DataWeave 2.0            | Data transformation          |
| HTTP Connector           | REST API integration         |
| MUnit                    | Unit and integration testing |
| YAML                     | External configuration       |
| Git/GitHub               | Version control              |

---

# 📁 Project Structure

```text
Order-Management-Integration/
│
├── Order-Management-Mule/
│   ├── src/
│   │   ├── main/
│   │   │   ├── mule/
│   │   │   │   ├── order-api.xml
│   │   │   │   ├── order-process.xml
│   │   │   │   └── global-config.xml
│   │   │   │
│   │   │   └── resources/
│   │   │       ├── order-api.raml
│   │   │       └── application.yaml
│   │   │
│   │   └── test/
│   │       └── munit/
│   │           └── order-process-test.xml
│   │
│   └── pom.xml
│
├── Product-Mock-API/
│   ├── src/
│   │   └── main/
│   │       └── mule/
│   │           └── product-api.xml
│   └── pom.xml
│
├── Order-Mock-API/
│   ├── src/
│   │   └── main/
│   │       └── mule/
│   │           └── order-mock.xml
│   └── pom.xml
│
├── Notification-Mock-API/
│   ├── src/
│   │   └── main/
│   │       └── mule/
│   │           └── notification-mock.xml
│   └── pom.xml
│
└── README.md
```

---

# 🌐 API Endpoints

## 1. Order Management API

### Create Order

```http
POST /api/orders
```

**Port:** `8081`

### Request

```json
{
  "customerId": "CUST001",
  "customerName": "Devendiran",
  "email": "deva@example.com",
  "items": [
    {
      "productId": "P101",
      "quantity": 2
    },
    {
      "productId": "P102",
      "quantity": 1
    }
  ]
}
```

---

# 📦 Product API

### Get Product

```http
GET /products/{productId}
```

**Port:** `8082`

Example:

```http
GET http://localhost:8082/products/P101
```

### Successful Response

```json
{
  "productId": "P101",
  "productName": "Laptop",
  "price": 5000,
  "availableStock": 10
}
```

---

# 🛒 External Order Management API

### Create External Order

```http
POST /external/orders
```

**Port:** `8083`

Example request:

```json
{
  "customerId": "CUST001",
  "items": [
    {
      "productId": "P101",
      "quantity": 2,
      "unitPrice": 5000,
      "amount": 10000
    }
  ],
  "subtotal": 10000,
  "discount": 0,
  "totalAmount": 10000
}
```

Example response:

```json
{
  "orderId": "ORD1001",
  "status": "CREATED"
}
```

---

# 📧 Notification API

### Send Notification

```http
POST /notifications
```

**Port:** `8084`

Example request:

```json
{
  "customerName": "Devendiran",
  "email": "deva@example.com",
  "orderId": "ORD1001",
  "totalAmount": 10000
}
```

Example response:

```json
{
  "status": "SENT",
  "message": "Order notification sent successfully"
}
```

---

# 💰 Business Rules

## 1. Product Validation

Every product in the order must exist in the Product API.

If a product does not exist:

```text
HTTP 404
PRODUCT_NOT_FOUND
```

---

## 2. Stock Validation

The requested quantity must not exceed the available stock.

Example:

```text
Requested Quantity = 5
Available Stock    = 2
```

The order is rejected.

Response:

```text
HTTP 409
INSUFFICIENT_STOCK
```

---

## 3. Item Amount Calculation

For every item:

```text
Item Amount = Unit Price × Quantity
```

Example:

```text
Price    = ₹5,000
Quantity = 2

Item Amount = ₹5,000 × 2
            = ₹10,000
```

---

## 4. Subtotal Calculation

```text
Subtotal = Sum of all item amounts
```

Example:

```text
Laptop  = ₹10,000
Mouse   = ₹2,000

Subtotal = ₹12,000
```

---

## 5. Discount Rule

Orders above ₹10,000 receive a **10% discount**.

```text
IF subtotal > 10000
    discount = subtotal × 0.10
ELSE
    discount = 0
```

Example:

```text
Subtotal = ₹15,000

Discount = ₹15,000 × 10%
         = ₹1,500

Final Amount = ₹15,000 - ₹1,500
             = ₹13,500
```

---

# 🔄 Complete Integration Flow

The main MuleSoft application follows this sequence:

```text
POST /api/orders
       │
       ▼
Validate Request
       │
       ├── Invalid ───────► 400 Bad Request
       │
       ▼
Extract Order Items
       │
       ▼
For Each Product
       │
       ▼
GET Product API
       │
       ├── Product Not Found ─► 404
       │
       ├── API Failure ───────► 502/503
       │
       ▼
Check Available Stock
       │
       ├── Insufficient ─────► 409
       │
       ▼
Calculate Item Amount
       │
       ▼
Calculate Subtotal
       │
       ▼
Apply 10% Discount
       │
       ▼
Calculate Final Amount
       │
       ▼
POST External Order API
       │
       ├── Failure ──────────► External Service Error
       │
       ▼
Generate/Receive Order ID
       │
       ▼
POST Notification API
       │
       ├── Failure ──────────► Log + Handle Gracefully
       │
       ▼
Return Order Response
```

---

# 🔄 Data Transformation

DataWeave 2.0 is used to transform the client request into the structure required by the external APIs.

### Example

Input:

```json
{
  "productId": "P101",
  "quantity": 2
}
```

Product API response:

```json
{
  "productId": "P101",
  "productName": "Laptop",
  "price": 5000,
  "availableStock": 10
}
```

Transformed order item:

```json
{
  "productId": "P101",
  "productName": "Laptop",
  "quantity": 2,
  "unitPrice": 5000,
  "amount": 10000
}
```

---

# 🧮 DataWeave Discount Logic

Example DataWeave logic:

```dataweave
%dw 2.0
output application/json

var subtotal = payload.subtotal
var discount = if (subtotal > 10000)
                  subtotal * 0.10
               else
                  0

---
{
    subtotal: subtotal,
    discount: discount,
    totalAmount: subtotal - discount
}
```

---

# 🆔 Correlation ID

A correlation ID is generated for every incoming request.

Example:

```text
Correlation ID:
ORD-8f7a92c1
```

The same ID is used throughout the integration flow.

This makes it easier to trace:

```text
Client Request
      ↓
Product API
      ↓
Order API
      ↓
Notification API
```

All logs associated with the same transaction can be identified using the correlation ID.

---

# 📝 Logging

Important business and technical events are logged.

Example:

```text
INFO  Order request received
INFO  Correlation ID: ORD-8f7a92c1
INFO  Validating customer request
INFO  Checking product P101
INFO  Product P101 available
INFO  Stock validation successful
INFO  Subtotal calculated: 15000
INFO  Discount applied: 1500
INFO  Creating external order
INFO  Order created: ORD1001
INFO  Sending customer notification
INFO  Notification sent successfully
INFO  Order processing completed
```

Sensitive information such as passwords, tokens, and unnecessary customer information should not be logged.

---

# ⚠️ Error Handling

The application uses MuleSoft error handling mechanisms to handle business and system errors.

## Error Model

All errors follow a standard structure:

```json
{
  "timestamp": "2026-09-07T20:00:00Z",
  "correlationId": "ORD-8f7a92c1",
  "errorCode": "INSUFFICIENT_STOCK",
  "message": "Insufficient stock for product P101"
}
```

---

# 🚨 Error Scenarios

| Scenario                | HTTP Status | Error Code                    |
| ----------------------- | ----------: | ----------------------------- |
| Invalid request         |         400 | `INVALID_REQUEST`             |
| Product not found       |         404 | `PRODUCT_NOT_FOUND`           |
| Insufficient stock      |         409 | `INSUFFICIENT_STOCK`          |
| Product API unavailable |         503 | `PRODUCT_SERVICE_UNAVAILABLE` |
| External API failure    |         502 | `EXTERNAL_SERVICE_ERROR`      |
| Unexpected error        |         500 | `INTERNAL_SERVER_ERROR`       |

---

# 🔁 Retry Strategy

Temporary external API failures can be handled using MuleSoft's retry mechanisms.

For example:

```text
Product API
     │
     ▼
Request
     │
     ├── Success ─────► Continue
     │
     └── Temporary Failure
              │
              ▼
          Retry
              │
              ▼
          Retry Again
              │
              ▼
       Final Failure
              │
              ▼
         Error Handler
```

Retries should only be applied to appropriate transient failures and should not be used for business errors such as product-not-found or insufficient stock.

---

# 🔐 Idempotency

The API can support an `Idempotency-Key` header to prevent duplicate orders when the same request is submitted multiple times.

Example:

```http
Idempotency-Key: ORDER-CUST001-001
```

This is particularly useful when:

* Client retries a request
* Network timeout occurs
* Client does not know whether the previous request succeeded
* Payment/order systems are involved

---

# ⚙️ Externalized Configuration

Environment-specific values should not be hardcoded inside the Mule flows.

Example `application.yaml`:

```yaml
http:
  listener:
    host: "0.0.0.0"
    port: "8081"

product:
  host: "localhost"
  port: "8082"
  basePath: "/products"

order:
  host: "localhost"
  port: "8083"
  basePath: "/external/orders"

notification:
  host: "localhost"
  port: "8084"
  basePath: "/notifications"
```

This allows the same Mule application to be used across different environments.

Example:

```text
Development
localhost

Testing
test-server

Production
production-server
```

---

# 🧪 MUnit Testing

MUnit is used to validate both successful and failure scenarios.

## Test Cases

### TC01 — Successful Order

```text
Given:
Valid customer
Valid products
Sufficient stock

Expected:
201 Created
Order ID generated
Notification sent
```

---

### TC02 — Invalid Request

```text
Given:
Missing customerId

Expected:
400 Bad Request
INVALID_REQUEST
```

---

### TC03 — Product Not Found

```text
Given:
Product ID = P999

Expected:
404 Not Found
PRODUCT_NOT_FOUND
```

---

### TC04 — Insufficient Stock

```text
Given:
Requested quantity > available stock

Expected:
409 Conflict
INSUFFICIENT_STOCK
```

---

### TC05 — Product API Failure

```text
Given:
Product API unavailable

Expected:
502/503
PRODUCT_SERVICE_UNAVAILABLE
```

---

### TC06 — Order API Failure

```text
Given:
Order Management API fails

Expected:
External service error handled correctly
```

---

### TC07 — Discount Applied

```text
Given:
Subtotal = ₹15,000

Expected:
Discount = ₹1,500
Total = ₹13,500
```

---

### TC08 — No Discount

```text
Given:
Subtotal = ₹8,000

Expected:
Discount = ₹0
Total = ₹8,000
```

---

### TC09 — Notification Failure

```text
Given:
Order successfully created
Notification API fails

Expected:
Order remains CREATED
Notification failure is handled/logged
```

---

# 🧪 Testing with Postman

## Start Applications

Run the applications in the following order:

```text
1. Product-Mock-API
2. Order-Mock-API
3. Notification-Mock-API
4. Order-Management-Mule
```

---

## Send Order Request

```http
POST http://localhost:8081/api/orders
```

Headers:

```http
Content-Type: application/json
```

Body:

```json
{
  "customerId": "CUST001",
  "customerName": "Devendiran",
  "email": "deva@example.com",
  "items": [
    {
      "productId": "P101",
      "quantity": 2
    },
    {
      "productId": "P102",
      "quantity": 1
    }
  ]
}
```

---

# ✅ Expected Successful Response

```json
{
  "orderId": "ORD1001",
  "status": "CREATED",
  "subtotal": 15000,
  "discount": 1500,
  "totalAmount": 13500,
  "notificationStatus": "SENT"
}
```

---

# 📊 Sample Product Data

| Product ID | Product         |  Price | Stock |
| ---------- | --------------- | -----: | ----: |
| P101       | Laptop          | ₹5,000 |    10 |
| P102       | Mouse           | ₹2,000 |    20 |
| P103       | Keyboard        | ₹3,000 |     5 |
| P104       | Monitor         | ₹8,000 |     8 |
| P999       | Unknown Product |      - |     - |

Special test products:

```text
P999 → Product Not Found
P500 → Simulated Product API Failure
```

---

# 🏗️ API Architecture

The solution follows an API-led integration approach.

```text
                 Experience Layer
                       │
                       ▼
             Order Management API
                       │
                 Process Layer
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
     Product API   Order API   Notification API
          │            │            │
          ▼            ▼            ▼
       Product      Order System  Notification
        System        System        System
```

For the hackathon, the external systems are represented by MuleSoft mock APIs.

---

# 🔒 Security Considerations

For a production deployment, the following security mechanisms can be added:

* Client ID Enforcement
* OAuth 2.0
* HTTPS
* Secure properties
* Encrypted credentials
* API Manager policies
* Rate limiting
* Input validation
* Sensitive-data masking

Credentials and secrets should never be committed to Git.

---

# 📈 Production Enhancements

The following features can be added when moving from the hackathon implementation toward production:

### API Management

* API Manager
* Client ID Enforcement
* OAuth 2.0
* Rate limiting
* SLA-based policies

### Observability

* Centralized logging
* Anypoint Monitoring
* Application metrics
* Distributed tracing
* Correlation IDs

### Reliability

* Retry
* Until Successful
* Circuit-breaker pattern
* Timeout handling
* Dead-letter processing

### Scalability

* CloudHub deployment
* Horizontal scaling
* Queue-based asynchronous processing
* Object Store for distributed state

### Data

* Persistent database
* Order history
* Inventory management
* Customer management

---

# 👥 Team Responsibilities

The project can be divided among five team members.

| Member   | Responsibility                               |
| -------- | -------------------------------------------- |
| Member 1 | Main Order Management API + RAML + APIKit    |
| Member 2 | Product API + Product validation + Stock     |
| Member 3 | Order Management Mock API + Notification API |
| Member 4 | DataWeave + Business Logic + Error Handling  |
| Member 5 | MUnit + Integration Testing + Documentation  |

All members should integrate and test the complete solution together before final submission.

---

# 📅 Development Plan

## Day 1

### Phase 1 — API Design

* Create RAML specification
* Define request/response types
* Define HTTP status codes
* Define error model

### Phase 2 — Main API

* Create MuleSoft application
* Import RAML
* Generate APIKit flows
* Configure HTTP Listener

### Phase 3 — External APIs

* Build Product Mock API
* Build Order Mock API
* Build Notification Mock API

### Phase 4 — Happy Path

Implement:

```text
Order Request
     ↓
Product Validation
     ↓
Stock Validation
     ↓
Amount Calculation
     ↓
Discount
     ↓
Create Order
     ↓
Notification
     ↓
Response
```

---

# 📅 Day 2

### Phase 5 — Error Handling

Implement:

* Invalid request
* Product not found
* Insufficient stock
* Product API failure
* Order API failure
* Notification failure
* Unexpected errors

### Phase 6 — Production Features

Implement:

* Correlation ID
* Structured logging
* Externalized configuration
* Retry
* Idempotency

### Phase 7 — Testing

Create MUnit tests for:

* Happy path
* Validation failures
* Product failures
* Stock failures
* External API failures
* Discount scenarios

### Phase 8 — Final Integration

Perform end-to-end testing:

```text
Client
 ↓
Order API
 ↓
Product API
 ↓
Business Logic
 ↓
Order API
 ↓
Notification API
 ↓
Client
```

---

# 📌 Example End-to-End Scenario

Customer:

```text
Customer ID: CUST001
Name: Devendiran
```

Orders:

```text
P101 → 2 × ₹5,000 = ₹10,000
P102 → 1 × ₹5,000 = ₹5,000
```

Subtotal:

```text
₹10,000 + ₹5,000 = ₹15,000
```

Discount:

```text
₹15,000 × 10% = ₹1,500
```

Final amount:

```text
₹15,000 - ₹1,500 = ₹13,500
```

Integration:

```text
POST /api/orders
        ↓
Validate request
        ↓
GET /products/P101
        ↓
Stock OK
        ↓
GET /products/P102
        ↓
Stock OK
        ↓
Subtotal = ₹15,000
        ↓
Discount = ₹1,500
        ↓
Total = ₹13,500
        ↓
POST /external/orders
        ↓
Order ID = ORD1001
        ↓
POST /notifications
        ↓
Notification SENT
        ↓
201 CREATED
```

---

# 🎯 Project Objective

The primary objective of this project is to demonstrate how MuleSoft can be used as an **integration and orchestration platform** to connect multiple independent systems and expose a unified Order Management API.

The solution demonstrates:

```text
API Design
    +
API Implementation
    +
System Integration
    +
Data Transformation
    +
Business Logic
    +
Error Handling
    +
Resilience
    +
Testing
    +
Observability
```

---

# 🏆 Hackathon Value

This project demonstrates practical knowledge of:

* MuleSoft Anypoint Studio
* APIKit
* RAML
* HTTP Connector
* HTTP Listener
* HTTP Request
* DataWeave
* Choice Router
* For Each
* Set Variable
* Set Payload
* Transform Message
* Error Handling
* On Error Continue
* On Error Propagate
* Retry mechanisms
* Logging
* Correlation IDs
* Configuration properties
* MUnit
* API-led connectivity

---

# 📜 License

This project was developed as part of a MuleSoft integration hackathon/academic project.

---

## 👨‍💻 Project Team

**Order Management Integration using MuleSoft**

Built using MuleSoft Anypoint Platform and Anypoint Studio.
