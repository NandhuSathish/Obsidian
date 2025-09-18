---
tags:
  - interview
  - nodejs
  - webdevelopment
---

## Question 1: What is REST and how is it different from SOAP?

**Interviewer:** "Can you explain what REST is and how it differs from SOAP?"

**You:** "Absolutely! Think of REST like ordering food at a modern restaurant where you simply tell the waiter what you want, and they bring it to you. It's straightforward and uses simple language everyone understands.

REST stands for Representational State Transfer. It's an architectural style that uses standard HTTP methods like GET, POST, PUT, DELETE to communicate. When I build a REST API, I'm essentially creating URLs that represent resources, like `/api/users` for users or `/api/products` for products.

SOAP, on the other hand, is like sending a formal letter with a specific envelope format. It's a protocol that uses XML for everything and requires strict formatting.

Here's a practical example: If I want to get user data:

- **REST:** Simply send `GET https://api.example.com/users/123`
- **SOAP:** Send an XML envelope with headers, body, specific namespaces - much more complex

In my experience, REST wins because:

- It's lighter - we can use JSON which is much smaller than XML
- It's flexible - supports multiple formats (JSON, XML, HTML)
- It's cacheable - improves performance significantly
- It works great with modern JavaScript frameworks

I typically choose REST for web and mobile apps because it's simpler and performs better, while SOAP might still be used in enterprise systems that need strict contracts and built-in error handling."

## Question 2: Why should REST APIs be stateless?

**Interviewer:** "You mentioned REST. Why is it important for REST APIs to be stateless?"

**You:** "Great question! Statelessness is like having a conversation where each sentence stands on its own - you don't need to remember what was said before to understand the current sentence.

In practical terms, when a client makes a request to my API, that request contains everything the server needs to process it. The server doesn't store any client context between requests.

Let me give you a real-world example from a project I worked on. We had an e-commerce API:

**Bad (Stateful) approach:**

```
POST /api/login
// Server stores session
GET /api/cart
// Server looks up who you are from stored session
```

**Good (Stateless) approach:**

```
GET /api/cart
Headers: Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
// Token contains user info, server doesn't need to remember anything
```

The benefits I've seen from stateless design:

1. **Scalability** - Any server can handle any request. When we scaled from 2 to 20 servers, we didn't need sticky sessions
2. **Reliability** - If one server crashes, another can immediately take over
3. **Simplicity** - No session management headaches
4. **Performance** - Can cache requests since they're self-contained

In my last project, going stateless reduced our server memory usage by 40% and made horizontal scaling trivial. We just spun up new instances during Black Friday without worrying about session distribution."

## Question 3: How do you handle versioning in APIs?

**Interviewer:** "How do you approach API versioning in your projects?"

**You:** "API versioning is like releasing new editions of a textbook while ensuring students with older editions can still follow along. I've used three main approaches, each with its place:

**1. URL Versioning** (My usual go-to):

```
https://api.example.com/v1/users
https://api.example.com/v2/users
```

This is the clearest for developers. When I built an API for a fintech client, we used this because their partners could easily see which version they were using.

**2. Header Versioning** (For cleaner URLs):

```
GET https://api.example.com/users
Headers: API-Version: 2
```

I used this for an internal microservices architecture where we wanted clean URLs but still needed versioning control.

**3. Query Parameter Versioning** (Quick and dirty):

```
https://api.example.com/users?version=2
```

My versioning strategy includes:

- **Never breaking changes in minor versions** - only additions
- **Deprecation notices** - We give 6-12 months notice with headers like `Sunset: Sat, 31 Dec 2024 23:59:59 GMT`
- **Supporting at least 2 versions** simultaneously
- **Clear migration guides** - I always document what changed and how to upgrade

Real example: When we moved from v1 to v2 in our payment API, we:

1. Kept v1 running for 18 months
2. Added deprecation warnings 6 months before sunset
3. Provided automated migration tools
4. Monitored usage to ensure all major clients migrated

The result? Zero disruption for our 200+ API consumers."

## Question 4: What are the common HTTP methods and when should you use them?

**Interviewer:** "Walk me through the HTTP methods you use and when you'd use each one."

**You:** "I think of HTTP methods like different tools in a toolbox - each has a specific purpose. Here's how I use them with real examples from a blog API I recently built:

**GET** - Reading data (Safe and Idempotent)

```
GET /api/posts       // Get all posts
GET /api/posts/123   // Get specific post
```

I ensure GETs never modify data. They're cacheable, bookmarkable, and safe to retry.

**POST** - Creating new resources

```
POST /api/posts
Body: {"title": "REST Best Practices", "content": "..."}
Returns: 201 Created with Location header
```

Each POST creates a new resource with a server-generated ID. Not idempotent - calling twice creates two posts.

**PUT** - Full updates (Idempotent)

```
PUT /api/posts/123
Body: {"title": "Updated Title", "content": "New content", "author": "John"}
```

I use PUT when replacing entire resources. If called multiple times, result stays the same.

**PATCH** - Partial updates (More efficient)

```
PATCH /api/posts/123
Body: {"title": "Just updating the title"}
```

Perfect when you only need to change one field. Saves bandwidth.

**DELETE** - Removing resources (Idempotent)

```
DELETE /api/posts/123
Returns: 204 No Content
```

**Less common but useful:**

- **HEAD** - Like GET but only returns headers. I use it for checking if resources exist
- **OPTIONS** - Returns allowed methods. Critical for CORS preflight requests

Pro tip I learned the hard way: Always make PUT and DELETE idempotent. If someone accidentally calls DELETE twice, the second call should return 404, not crash."

## Question 5: How do you secure your REST APIs?

**Interviewer:** "Security is crucial. How do you secure your REST APIs?"

**You:** "Security is my top priority - I layer multiple defenses like a bank vault. Here's my security checklist with real implementations:

**1. Authentication & Authorization (JWT tokens)**

```javascript
// Every request includes:
Headers: Authorization: Bearer eyJhbGciOiJIUzI1NiIs...

// Token contains:
{
  "userId": "123",
  "roles": ["user", "admin"],
  "exp": 1699564800  // Expiration
}
```

I use short-lived access tokens (15 min) with refresh tokens (7 days) to balance security and user experience.

**2. HTTPS Everywhere**

- SSL/TLS for all environments, even development
- HSTS headers to force HTTPS
- Certificate pinning for mobile apps

**3. Rate Limiting**

```javascript
// Redis-based rate limiting
if (requestCount > 100 per minute) {
  return 429 Too Many Requests
  Headers: Retry-After: 60
}
```

This stopped a DDoS attempt that could've cost us thousands in cloud bills.

**4. Input Validation & Sanitization**

```javascript
// Never trust client data
const schema = Joi.object({
  email: Joi.string().email().required(),
  age: Joi.number().min(0).max(120)
});
```

**5. CORS Configuration**

```javascript
// Specific origins only, never *
Access-Control-Allow-Origin: https://myapp.com
```

**6. Security Headers**

- `X-Content-Type-Options: nosniff`
- `X-Frame-Options: DENY`
- `Content-Security-Policy: default-src 'self'`

**7. API Keys for B2B**

- Different keys for dev/staging/prod
- Rotate keys quarterly
- Monitor usage patterns

**8. Audit Logging** Every API call logged with: who, what, when, from where

**Real incident**: We detected suspicious activity because one API key suddenly made 10x normal requests at 3 AM. Turned out to be a compromised key - we revoked it immediately and prevented data breach.

My philosophy: Security isn't a feature, it's the foundation."

## Question 6: Explain how filtering and pagination work.

**Interviewer:** "How do you implement filtering and pagination in your APIs?"

**You:** "I think of filtering and pagination like a library catalog system - you need to find specific books without carrying the entire library home. Here's how I implement both:

**Pagination - Three approaches I use:**

**1. Offset-based** (Simple, common):

```
GET /api/products?page=2&limit=20
// or
GET /api/products?offset=20&limit=20

Response:
{
  "data": [...],
  "pagination": {
    "total": 500,
    "page": 2,
    "pages": 25,
    "limit": 20
  }
}
```

**2. Cursor-based** (Better for real-time data):

```
GET /api/feed?cursor=eyJpZCI6MTIzfQ&limit=20

Response:
{
  "data": [...],
  "nextCursor": "eyJpZCI6MTQzfQ",
  "hasMore": true
}
```

I used this for a social media feed where new posts constantly appear. Prevents duplicates when data changes.

**3. Link Headers** (RESTful approach):

```
Link: <https://api.example.com/products?page=3>; rel="next",
      <https://api.example.com/products?page=1>; rel="prev",
      <https://api.example.com/products?page=10>; rel="last"
```

**Filtering - Multiple strategies:**

**1. Query Parameters** (Most common):

```
GET /api/products?category=electronics&price_min=100&price_max=500&brand=apple,samsung
```

**2. Advanced Filtering** (For complex queries):

```
GET /api/products?filter={"price":{"$gte":100,"$lte":500},"brand":{"$in":["apple","samsung"]}}
```

**3. Search with filters**:

```
GET /api/products?q=laptop&category=electronics&sort=-price
// '-price' means descending order
```

**Real-world example from an e-commerce project:**

```
GET /api/products?
  category=laptops&
  price_min=500&
  price_max=1500&
  brand=dell,hp,lenovo&
  features=ssd,touchscreen&
  in_stock=true&
  sort=-popularity,price&
  page=1&
  limit=24

Response:
{
  "products": [...],
  "facets": {
    "brands": [
      {"name": "Dell", "count": 45},
      {"name": "HP", "count": 32}
    ],
    "priceRanges": [...]
  },
  "totalResults": 156,
  "page": 1,
  "totalPages": 7
}
```

**Performance tips I've learned:**

- Always use database indexes on filtered fields
- Limit maximum page size (I cap at 100)
- For large datasets, discourage deep pagination (page 1000+)
- Include filter counts to show users how many results match"

## Question 7: How do you handle errors and return appropriate status codes?

**Interviewer:** "Tell me about your approach to error handling and HTTP status codes."

**You:** "Error handling is like giving directions - you need to be clear about what went wrong and how to fix it. I follow a consistent pattern across all my APIs:

**My Status Code Categories:**

**2xx - Success**

- `200 OK` - Standard success
- `201 Created` - After POST creating resource
- `204 No Content` - Successful DELETE
- `202 Accepted` - Async operation started

**4xx - Client Errors** (You did something wrong)

- `400 Bad Request` - Invalid syntax/parameters
- `401 Unauthorized` - Missing/invalid authentication
- `403 Forbidden` - Authenticated but not allowed
- `404 Not Found` - Resource doesn't exist
- `409 Conflict` - State conflict (duplicate email)
- `422 Unprocessable Entity` - Validation errors
- `429 Too Many Requests` - Rate limit hit

**5xx - Server Errors** (We did something wrong)

- `500 Internal Server Error` - Generic server error
- `502 Bad Gateway` - Upstream service issue
- `503 Service Unavailable` - Temporary overload
- `504 Gateway Timeout` - Upstream timeout

**My Error Response Format:**

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed for the request",
    "details": [
      {
        "field": "email",
        "message": "Email format is invalid",
        "code": "INVALID_FORMAT"
      },
      {
        "field": "age",
        "message": "Age must be between 18 and 120",
        "code": "OUT_OF_RANGE"
      }
    ],
    "timestamp": "2024-01-15T10:30:00Z",
    "traceId": "abc-123-def",
    "documentation": "https://api.example.com/docs/errors/VALIDATION_ERROR"
  }
}
```

**Real-world example - Payment API error handling:**

```javascript
// Client error (422)
{
  "error": {
    "code": "INSUFFICIENT_FUNDS",
    "message": "Payment cannot be processed due to insufficient funds",
    "details": {
      "required": 150.00,
      "available": 100.00,
      "currency": "USD"
    },
    "suggestedAction": "Please add funds or use a different payment method",
    "documentation": "https://api.example.com/docs/payments#insufficient-funds"
  }
}

// Server error (503) with retry information
{
  "error": {
    "code": "SERVICE_TEMPORARILY_UNAVAILABLE",
    "message": "Payment service is under maintenance",
    "retryAfter": 300,  // seconds
    "alternativeEndpoint": "https://api-backup.example.com/payments"
  }
}
```

**Key principles I follow:**

1. **Never expose internal details** in production (no stack traces!)
2. **Be consistent** - Same error format everywhere
3. **Be helpful** - Include what went wrong and how to fix it
4. **Log everything** server-side but return safe messages to clients
5. **Include trace IDs** for debugging customer issues

This approach reduced our support tickets by 30% because developers could self-diagnose issues."

## Question 8: What is HATEOAS and why is it important?

**Interviewer:** "Can you explain HATEOAS and its importance?"

**You:** "HATEOAS - Hypermedia as the Engine of Application State - sounds complex, but it's actually like a GPS navigation system. Instead of memorizing all possible routes, the GPS tells you the next possible turns based on where you are.

In REST APIs, HATEOAS means the API responses include links to related actions you can take. The client doesn't need to hardcode URLs or know the API structure in advance.

**Without HATEOAS** (Most common):

```json
{
  "orderId": 123,
  "status": "pending",
  "total": 99.99
}
// Client needs to know to call POST /orders/123/cancel to cancel
```

**With HATEOAS** (Self-descriptive):

```json
{
  "orderId": 123,
  "status": "pending",
  "total": 99.99,
  "_links": {
    "self": {
      "href": "/orders/123"
    },
    "cancel": {
      "href": "/orders/123/cancel",
      "method": "POST"
    },
    "payment": {
      "href": "/orders/123/payment",
      "method": "POST"
    },
    "items": {
      "href": "/orders/123/items"
    }
  }
}
```

**Real example from a workflow API I built:**

```json
{
  "documentId": "doc-456",
  "status": "draft",
  "title": "Q4 Report",
  "_links": {
    "self": { "href": "/documents/doc-456" },
    "edit": { 
      "href": "/documents/doc-456/edit",
      "method": "PUT"
    },
    "submit": {
      "href": "/documents/doc-456/submit",
      "method": "POST",
      "available": true
    },
    "approve": {
      "href": "/documents/doc-456/approve",
      "method": "POST",
      "available": false,
      "reason": "Document must be submitted first"
    },
    "history": {
      "href": "/documents/doc-456/history"
    }
  }
}
```

**Benefits I've experienced:**

1. **API evolution** - Changed URLs without breaking clients
2. **Self-documenting** - New developers understand available actions
3. **State-driven UI** - Frontend automatically shows/hides buttons based on available links
4. **Reduced coupling** - Clients don't hardcode business logic

**Honest perspective:** While HATEOAS is RESTful ideal, I typically implement a pragmatic version:

- Include links for critical workflows
- Use it for complex state machines
- Skip it for simple CRUD if it adds no value

In my document workflow system, HATEOAS reduced frontend bugs by 40% because the UI automatically adapted to what actions were available based on document state."

## Bonus Questions

### Question 9: How do you handle API documentation?

**Interviewer:** "How do you approach API documentation?"

**You:** "API documentation is like a user manual - if it's not good, nobody will use your product correctly. I use OpenAPI/Swagger as my foundation:

**My documentation stack:**

1. **OpenAPI 3.0 Specification** - Single source of truth
2. **Swagger UI** - Interactive documentation
3. **Postman Collections** - Ready-to-run examples
4. **README with quickstart** - Get developers running in 5 minutes

**Example OpenAPI snippet:**

```yaml
paths:
  /users/{id}:
    get:
      summary: Get user by ID
      description: Returns a single user with detailed information
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: string
      responses:
        200:
          description: User found
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
              example:
                id: "123"
                name: "John Doe"
                email: "john@example.com"
```

Every endpoint includes:

- Clear description
- Request/response examples
- Error scenarios
- Rate limits
- Authentication requirements

This reduced onboarding time for new API consumers from days to hours."

### Question 10: How do you handle file uploads in REST APIs?

**Interviewer:** "What's your approach to handling file uploads?"

**You:** "File uploads require special consideration. I use different strategies based on file size and requirements:

**For small files (< 10MB):**

```
POST /api/upload
Content-Type: multipart/form-data
Body: [file data + metadata]
```

**For large files (> 10MB):**

1. **Pre-signed URLs** (My preference):

```javascript
// 1. Client requests upload URL
POST /api/uploads/request
Body: {"filename": "video.mp4", "size": 104857600}

// 2. API returns pre-signed S3 URL
Response: {
  "uploadUrl": "https://s3.amazon.com/...",
  "fileId": "file-789"
}

// 3. Client uploads directly to S3
// 4. Client confirms completion
POST /api/uploads/file-789/complete
```

**Benefits:**

- Reduces server load
- Enables progress tracking
- Supports resume on failure
- Scales infinitely

In our video platform, this approach handled 10TB daily uploads without stressing our API servers."

### Question 11: How do you design APIs for mobile apps vs web apps?

**Interviewer:** "Do you design APIs differently for mobile versus web clients?"

**You:** "Great question! Mobile clients have unique constraints I always consider:

**Mobile-specific optimizations:**

1. **Minimize payload size:**

```javascript
// Web response
{
  "user": {
    "profile": {...},
    "settings": {...},
    "preferences": {...},
    "history": [...]
  }
}

// Mobile response with field selection
GET /api/users/123?fields=id,name,avatar
{
  "id": 123,
  "name": "John",
  "avatar": "..."
}
```

2. **Reduce number of calls** (Battery/bandwidth):

```javascript
// GraphQL-like aggregation endpoint
GET /api/mobile/home
Returns: {
  "user": {...},
  "notifications": [...],
  "recentActivity": [...]
}
```

3. **Implement sync endpoints** for offline support:

```javascript
POST /api/sync
Body: {
  "lastSync": "2024-01-15T10:00:00Z",
  "localChanges": [...]
}
Response: {
  "serverChanges": [...],
  "conflicts": [...],
  "newSyncToken": "..."
}
```

4. **Version handling:**

- Longer deprecation cycles (app store approval delays)
- Force-update mechanisms for critical changes
- Backend for Frontend (BFF) pattern when needed

This strategy reduced our mobile app's data usage by 60% and improved battery life noticeably."

### Question 12: How do you monitor API performance and health?

**Interviewer:** "How do you monitor your APIs in production?"

**You:** "Monitoring is like having a health tracker for your API. Here's my monitoring setup:

**Key metrics I track:**

1. **Response times** - P50, P95, P99 percentiles
2. **Error rates** - 4xx vs 5xx trends
3. **Throughput** - Requests per second
4. **Availability** - Uptime percentage

**My monitoring stack:**

- **APM**: New Relic/DataDog for application performance
- **Logging**: ELK stack (Elasticsearch, Logstash, Kibana)
- **Synthetic monitoring**: Pingdom for endpoint health
- **Custom dashboards**: Grafana for business metrics

**Alert thresholds:**

```javascript
// Critical alerts (wake me up)
- Error rate > 5% for 5 minutes
- P99 latency > 3 seconds
- 5xx errors > 10 per minute

// Warning alerts (business hours)
- P95 latency > 1 second
- 4xx errors increasing 50% over baseline
```

**Real example:** Our monitoring caught a memory leak that only appeared after 10 hours of runtime. The gradual latency increase triggered alerts before customers noticed.

Pro tip: I always include correlation IDs in logs to trace requests across services. Saved countless debugging hours."

## Final Thoughts

**Interviewer:** "Any final best practices you'd like to share?"

**You:** "Absolutely! Here are my golden rules:

1. **Design API-first** - Not as an afterthought
2. **Version from day one** - Even if you think you won't need it
3. **Be consistent** - Same patterns everywhere
4. **Think about developer experience** - They're your users
5. **Plan for scale** - But don't over-engineer
6. **Document everything** - Your future self will thank you
7. **Monitor religiously** - You can't fix what you can't see

