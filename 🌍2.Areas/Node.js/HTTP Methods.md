---
tags:
  - nodejs
---

## 🎯 Quick Memory Framework: **CRUD-GPS**

**C**reate → POST  
**R**ead → GET  
**U**pdate → PUT/PATCH  
**D**elete → DELETE  
**G**et → GET  
**P**ost → POST  
**S**pecial → HEAD, OPTIONS, TRACE, CONNECT

---

## 📊 HTTP Methods Overview Table

|Method|Purpose|Request Body|Response Body|Idempotent|Safe|Cacheable|
|---|---|---|---|---|---|---|
|**GET**|Retrieve data|❌|✅|✅|✅|✅|
|**POST**|Create resource|✅|✅|❌|❌|⚠️ (with headers)|
|**PUT**|Replace entire resource|✅|⚠️|✅|❌|❌|
|**PATCH**|Partial update|✅|⚠️|❌*|❌|❌|
|**DELETE**|Remove resource|⚠️|⚠️|✅|❌|❌|
|**HEAD**|GET without body|❌|❌|✅|✅|✅|
|**OPTIONS**|Check allowed methods|⚠️|✅|✅|✅|❌|
|**TRACE**|Echo request|❌|✅|✅|✅|❌|
|**CONNECT**|Establish tunnel|❌|✅|❌|❌|❌|

---

## 🔍 Detailed Breakdown

### 1️⃣ **GET** - The Reader

```javascript
// ✅ Correct Usage
GET /api/users/123
GET /api/products?category=electronics&sort=price

// ❌ Never Do This
GET /api/deleteUser/123  // GET should never modify data
```

**Key Points:**

- **Idempotent**: Multiple identical requests = same result
- **Cacheable**: Browsers/CDNs can cache responses
- **No request body**: Parameters via URL/query strings
- **Safe**: Doesn't modify server state

**Interview Answer:**

> "GET retrieves data without side effects. It's idempotent and safe, meaning multiple identical requests won't change the server state. Perfect for fetching resources."

---

### 2️⃣ **POST** - The Creator

```javascript
// Creating new resource
POST /api/users
Content-Type: application/json
{
  "name": "John Doe",
  "email": "john@example.com"
}

// Response: 201 Created
Location: /api/users/124
```

**Key Points:**

- **NOT Idempotent**: Each request creates new resource
- **Has request body**: Contains data for new resource
- **Returns**: 201 (Created) with Location header
- **Use cases**: Form submissions, file uploads, creating resources

**Interview Answer:**

> "POST creates new resources and isn't idempotent - each request creates a new resource. Returns 201 with Location header pointing to the new resource."

---

### 3️⃣ **PUT** - The Full Replacer

```javascript
// Replace entire resource
PUT /api/users/123
{
  "id": 123,
  "name": "Jane Doe",
  "email": "jane@example.com",
  "age": 30
}
// Must send ALL fields, missing fields will be set to null/default
```

**Key Points:**

- **Idempotent**: Multiple identical requests = same result
- **Full replacement**: Replaces ENTIRE resource
- **Creates if doesn't exist**: Can create resource at specific URI
- **Returns**: 200 (OK) or 204 (No Content)

**Interview Answer:**

> "PUT replaces the entire resource at a specific URI. It's idempotent - sending the same PUT request multiple times won't change the result after the first request."

---

### 4️⃣ **PATCH** - The Partial Updater

```javascript
// Update only specific fields
PATCH /api/users/123
{
  "email": "newemail@example.com"
}
// Only updates email, other fields unchanged
```

**Key Points:**

- **Partial updates**: Only modify specified fields
- **Can be idempotent**: Depends on implementation
- **More efficient**: Sends only changed data
- **Returns**: 200 (OK) or 204 (No Content)

**Interview Answer:**

> "PATCH performs partial updates, modifying only the fields provided. Unlike PUT, it doesn't require the complete resource representation."

---

### 5️⃣ **DELETE** - The Remover

```javascript
// Delete a resource
DELETE /api/users/123

// Response: 204 No Content (success, no body)
// OR
// Response: 200 OK (with confirmation message)
{
  "message": "User deleted successfully"
}
```

**Key Points:**

- **Idempotent**: Deleting already deleted resource = same result
- **May have request body**: But rarely used
- **Returns**: 204 (No Content) or 200 (OK) or 404 (Not Found)

---

### 6️⃣ **HEAD** - The Inspector

```javascript
// Check if resource exists, get metadata
HEAD /api/large-file.pdf

// Response: Headers only (Content-Length, Last-Modified, etc.)
// No response body
```

**Use Cases:**

- Check if resource exists
- Get metadata without downloading
- Check if resource has been modified

---

### 7️⃣ **OPTIONS** - The Explorer

```javascript
// Check what methods are allowed
OPTIONS /api/users

// Response Headers:
Allow: GET, POST, PUT, DELETE, OPTIONS
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
```

**Key Points:**

- **CORS preflight**: Browsers use it for cross-origin requests
- **Discover API capabilities**: What methods are supported
- **Returns**: Allowed methods in headers

---

## 🎨 Real-World RESTful Examples

### User Management API

```javascript
GET    /users          // List all users
GET    /users/123      // Get specific user
POST   /users          // Create new user
PUT    /users/123      // Replace user completely
PATCH  /users/123      // Update user partially
DELETE /users/123      // Delete user
HEAD   /users/123      // Check if user exists
OPTIONS /users         // Get allowed operations
```

---

## 🔥 Interview Questions & Answers

### Q1: "What's the difference between PUT and PATCH?"

**Perfect Answer:**

```
PUT replaces the ENTIRE resource - you must send all fields.
PATCH updates PARTIALLY - you send only the fields to change.

Example:
User = { id: 1, name: "John", email: "john@example.com", age: 25 }

PUT /users/1 { name: "Jane", email: "jane@example.com" }
Result: { id: 1, name: "Jane", email: "jane@example.com", age: null } ❌ Lost age!

PATCH /users/1 { name: "Jane" }
Result: { id: 1, name: "Jane", email: "john@example.com", age: 25 } ✅ Only name changed!
```

### Q2: "What does idempotent mean?"

**Perfect Answer:**

```
Idempotent means multiple identical requests produce the same result.

✅ Idempotent:
- GET /users/1 (called 5 times) = Same user returned
- PUT /users/1 {same data} (called 5 times) = Same result
- DELETE /users/1 (called 5 times) = User stays deleted

❌ NOT Idempotent:
- POST /users {same data} (called 5 times) = 5 different users created!
```

### Q3: "When should I use POST vs PUT?"

**Perfect Answer:**

```
POST: When the server decides the resource ID
- POST /users (Server creates /users/124)
- Creating new resources
- Non-idempotent operations

PUT: When the client knows the resource ID
- PUT /users/124 (Client specifies exact location)
- Replacing existing resources
- Idempotent operations
```

### Q4: "Can GET requests have a body?"

**Perfect Answer:**

```
Technically YES by HTTP spec, but practically NO.

- HTTP/1.1 spec doesn't forbid it
- BUT most servers/frameworks ignore GET body
- Use query parameters instead: GET /search?q=javascript
- If you need body, use POST for complex searches
```

### Q5: "What's a safe method?"

**Perfect Answer:**

```
Safe methods don't modify server state - only retrieve data.

Safe Methods: GET, HEAD, OPTIONS, TRACE
- Read-only operations
- Can be cached
- No side effects

Unsafe Methods: POST, PUT, PATCH, DELETE
- Modify server state
- Should not be cached
- Have side effects
```

---

## 💡 Pro Tips for Interviews

### 1. **Status Codes to Remember**

```
GET:    200 (OK), 404 (Not Found)
POST:   201 (Created), 400 (Bad Request)
PUT:    200 (OK), 201 (Created), 204 (No Content)
PATCH:  200 (OK), 204 (No Content)
DELETE: 200 (OK), 204 (No Content), 404 (Not Found)
```

### 2. **Common Mistakes to Avoid**

```javascript
❌ GET /api/deleteUser/123     // GET shouldn't modify
❌ POST /api/getUsers           // POST shouldn't just retrieve
❌ PUT without all fields       // Use PATCH for partial updates
❌ Non-idempotent DELETE        // DELETE should be idempotent
```

### 3. **Best Practices**

```javascript
✅ Use nouns for resources:     /users (not /getUsers)
✅ Use HTTP methods for actions: DELETE /users/123 (not /deleteUser/123)
✅ Return proper status codes:   201 for POST, 204 for DELETE
✅ Use query params for filters: GET /users?role=admin
```

---

## 🧠 Memory Tricks

### **CRUDL** Pattern

- **C**reate → POST
- **R**ead → GET
- **U**pdate → PUT/PATCH
- **D**elete → DELETE
- **L**ist → GET (collection)

### **SIG** (Safe, Idempotent, GET-like)

- **Safe**: GET, HEAD, OPTIONS, TRACE
- **Idempotent**: GET, PUT, DELETE, HEAD, OPTIONS, TRACE
- **GET-like**: GET, HEAD (retrieve without modification)

### **Response Code Pattern**

- **2xx**: Success (200 OK, 201 Created, 204 No Content)
- **4xx**: Client Error (400 Bad Request, 404 Not Found)
- **5xx**: Server Error (500 Internal Server Error)

---

## 🎬 Quick Interview Script

**Interviewer**: "Explain HTTP methods"

**You**: "HTTP methods define the action to perform on resources. The main ones follow CRUD operations:

- **GET** reads data (safe, idempotent, cacheable)
- **POST** creates new resources (not idempotent)
- **PUT** replaces entire resources (idempotent)
- **PATCH** partially updates resources
- **DELETE** removes resources (idempotent)

Special methods include HEAD for metadata, OPTIONS for CORS and capability checking. The key concepts are idempotency - whether repeated requests have the same effect, and safety - whether the method modifies server state."

---

## 🔗 Related Topics to Review

- REST principles
- HTTP status codes
- CORS (Cross-Origin Resource Sharing)
- Request/Response headers
- API versioning strategies
- Authentication methods (Bearer, OAuth)
- Rate limiting
- GraphQL vs REST

---

## ✅ Final Checklist

Before your interview, ensure you can:

- [ ] Explain each HTTP method in one sentence
- [ ] Give real-world example for each method
- [ ] Explain idempotency with examples
- [ ] Describe PUT vs PATCH difference
- [ ] Know which methods are safe
- [ ] List proper status codes for each method
- [ ] Explain when to use each method
- [ ] Understand CORS preflight (OPTIONS)

---

**Remember**: Interviewers love practical examples. Always relate HTTP methods to real-world API design scenarios like user management, e-commerce, or social media platforms.