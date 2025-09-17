
A REST API (Representational State Transfer API) is a widely used architectural style that enables communication between client and server applications over HTTP. A well-designed REST API is predictable, scalable, and maintainable. This note covers all key principles, best practices, and interview-ready concepts.

---

## ✅ What is REST API?
- A set of guidelines and conventions to build scalable and efficient web services.
- Uses standard [[HTTP Methods]] (`GET`, `POST`, `PUT`, `DELETE`, etc.).
- Stateless communication between client and server.
- Resources are represented as URLs and accessed using standard operations.

---

## ✅ Core REST API Principles

### 1️⃣ **Statelessness**
- The server does not store any client session data.
- Each request must contain all necessary information.
- Helps in scalability and fault tolerance.

### 2️⃣ **Resource-Based Design**
- Everything is treated as a resource (users, products, orders, etc.).
- Resources are accessed via URLs (endpoints).
  
**Example:**
```

GET /users/123 → Get user with ID 123  
POST /orders → Create a new order

```

### 3️⃣ **Use of HTTP Methods**
- `GET` → Retrieve resource(s)
- `POST` → Create new resource
- `PUT` → Update entire resource
- `PATCH` → Update part of resource
- `DELETE` → Remove resource

### 4️⃣ **Use of HTTP Status Codes**
- `200 OK` → Success
- `201 Created` → Resource created
- `204 No Content` → Successful deletion or update with no body
- `400 Bad Request` → Invalid request parameters
- `401 Unauthorized` → Authentication required
- `403 Forbidden` → Access denied
- `404 Not Found` → Resource not found
- `500 Internal Server Error` → Server error

### 5️⃣ **Consistent Naming Conventions**
- Use plural nouns for resources.
```

/users, /products, /orders

```
- Avoid verbs in endpoint names.
```

✅ /users, ❌ /getUsers

```

### 6️⃣ **Versioning**
- APIs evolve, so versioning is critical.
- Use URI versioning or header-based versioning.

**Example:**
```

/api/v1/users

```

### 7️⃣ **Filtering, Sorting, and Pagination**
- Allow querying resources with parameters.

**Example:**
```

GET /products?category=books&sort=price_asc&page=2&limit=10

```

### 8️⃣ **Hypermedia as the Engine of Application State (HATEOAS)**
- Provide links to related resources within responses.
- Makes APIs easier to navigate and self-descriptive.

---

## ✅ Best Practices

### ✅ Clear and Concise API Documentation
- Explain endpoints, request parameters, responses, error codes.
- Tools: Swagger/OpenAPI, Postman.

### ✅ Authentication & Authorization
- Use JWT, OAuth, or API keys.
- Secure endpoints based on user roles and permissions.

### ✅ Data Validation & Error Handling
- Validate incoming data.
- Provide meaningful error messages with status codes.

### ✅ Rate Limiting & Throttling
- Protect APIs from abuse and overload.

### ✅ Use of Content-Type
- Always specify request/response formats.
```

Content-Type: application/json  
Accept: application/json

```

### ✅ Secure Data Transfer
- Always use HTTPS to encrypt communication.

---

## ✅ Common Interview Questions

1. What is REST and how is it different from SOAP?
2. Why should REST APIs be stateless?
3. How do you handle versioning in APIs?
4. What are the common HTTP methods and when should you use them?
5. How do you secure your REST APIs?
6. Explain how filtering and pagination work.
7. How do you handle errors and return appropriate status codes?
8. What is HATEOAS and why is it important?

---

## ✅ Tools and Resources to Practice
- **Postman** – API testing
- **Swagger/OpenAPI** – API documentation
- **JWT.io** – Authentication handling
- **curl** – Command-line API interaction
- **GitHub** – Explore open-source APIs

---

## ✅ Summary

A modern REST API:
✔ Uses standard HTTP methods  
✔ Is stateless and scalable  
✔ Has well-defined resources and URLs  
✔ Handles errors with proper status codes  
✔ Is versioned and documented clearly  
✔ Is secure, validated, and efficient  



