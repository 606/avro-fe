---
{"publish":true,"aliases":"http-methods http-verbs rest-methods","title":"HTTP Methods Deep Dive","enableToc":true,"created":"2025-11-16T00:28:03.500+02:00","modified":"2025-11-16T00:32:07.663+02:00","tags":["api","rest","http","methods"],"cssclasses":""}
---


## Overview

HTTP methods (also called HTTP verbs) define the action to be performed on a resource. Each method has specific semantics and expected behavior.

## GET Method

### Purpose
- Retrieve resource representation
- Safe operation (doesn't modify data)
- Idempotent (multiple calls = same result)

### Characteristics
- **Cacheable**: Can be cached by browsers and proxies
- **Safe**: No side effects on server state
- **Idempotent**: Multiple identical requests have same effect

### Best Practices
```javascript
// Good: Use query parameters for filtering
GET /api/users?status=active&limit=10

// Good: Use path parameters for resource identification
GET /api/users/123

// Avoid: Using GET for sensitive operations
GET /api/users/delete/123  // Wrong!
```

### Headers
```
Accept: application/json
Accept-Language: en-US
If-Modified-Since: Wed, 21 Oct 2015 07:28:00 GMT
If-None-Match: "etag-value"
```

## POST Method

### Purpose
- Create new resources
- Process data (non-idempotent operations)
- Submit form data

### Characteristics
- **Not Safe**: Modifies server state
- **Not Idempotent**: Multiple calls create multiple resources
- **Not Cacheable**: Responses typically not cached

### Use Cases
```javascript
// Creating resources
POST /api/users
{
  "name": "John Doe",
  "email": "john@example.com"
}

// Processing operations
POST /api/payments/process
{
  "amount": 100.00,
  "currency": "USD"
}
```

### Response Patterns
- **201 Created**: Resource successfully created
- **Location header**: URL of newly created resource
- **Response body**: Created resource representation

## PUT Method

### Purpose
- Update/replace entire resource
- Create resource at known URI (less common)

### Characteristics
- **Not Safe**: Modifies server state
- **Idempotent**: Multiple calls have same effect
- **Not Cacheable**: Typically not cached

### Idempotency
```javascript
// First call: Creates user with ID 123
PUT /api/users/123
{
  "name": "John Doe",
  "email": "john@example.com"
}

// Second call: Updates existing user (same result)
PUT /api/users/123
{
  "name": "John Doe",
  "email": "john@example.com"
}
```

### Partial Updates
- PUT replaces entire resource
- Use PATCH for partial updates
- Document behavior clearly

## PATCH Method

### Purpose
- Apply partial modifications to resource
- Update specific fields without replacing entire resource

### Characteristics
- **Not Safe**: Modifies server state
- **Not Idempotent**: May or may not be (depends on implementation)
- **Not Cacheable**: Typically not cached

### Patch Formats

#### JSON Patch (RFC 6902)
```json
PATCH /api/users/123
Content-Type: application/json-patch+json

[
  { "op": "replace", "path": "/name", "value": "Jane Doe" },
  { "op": "add", "path": "/phone", "value": "+1-555-0123" },
  { "op": "remove", "path": "/oldField" }
]
```

#### Merge Patch (RFC 7396)
```json
PATCH /api/users/123
Content-Type: application/merge-patch+json

{
  "name": "Jane Doe",
  "phone": "+1-555-0123",
  "department": null  // Remove field
}
```

## DELETE Method

### Purpose
- Remove resource from server
- Logical deletion (soft delete) or physical deletion

### Characteristics
- **Not Safe**: Modifies server state
- **Idempotent**: Multiple calls have same effect (resource gone)
- **Not Cacheable**: Typically not cached

### Deletion Patterns
```javascript
// Hard delete
DELETE /api/users/123
// Returns 204 No Content

// Soft delete (logical)
DELETE /api/users/123
// Returns 200 OK with updated resource
{
  "id": 123,
  "deleted": true,
  "deletedAt": "2025-11-16T10:00:00Z"
}
```

### Cascade Deletion
- Consider relationships
- Document cascade behavior
- Use transactions for consistency

## HEAD Method

### Purpose
- Retrieve headers without response body
- Check resource existence and metadata
- Bandwidth optimization

### Characteristics
- **Safe**: No side effects
- **Idempotent**: Same as GET without body
- **Cacheable**: Same caching rules as GET

### Use Cases
```javascript
// Check if resource exists
HEAD /api/users/123
// Returns 200 OK or 404 Not Found

// Get resource metadata
HEAD /api/documents/large-report.pdf
// Returns Content-Length, Last-Modified, etc.
```

## OPTIONS Method

### Purpose
- Describe communication options for target resource
- CORS preflight requests
- API discovery

### Characteristics
- **Safe**: No side effects
- **Idempotent**: Always same result
- **Not Cacheable**: Fresh information needed

### CORS Example
```javascript
OPTIONS /api/users
Access-Control-Request-Method: POST
Access-Control-Request-Headers: content-type,authorization

// Response
HTTP/1.1 200 OK
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS
Access-Control-Allow-Headers: content-type,authorization
Access-Control-Max-Age: 86400
```

## Less Common Methods

### TRACE
- Echoes request back to client
- Debugging proxy configurations
- Security risk (request disclosure)

### CONNECT
- Establish tunnel to server
- Used by proxies for HTTPS
- Not typically used in REST APIs

## Method Override

### X-HTTP-Method-Override Header
```javascript
// Browser limitation workaround
POST /api/users/123
X-HTTP-Method-Override: DELETE
```

### Query Parameter Override
```javascript
POST /api/users/123?_method=DELETE
```

## Security Considerations

### Method-Specific Risks
- **GET**: Parameter tampering, sensitive data in URLs
- **POST**: CSRF attacks, request body manipulation
- **PUT/PATCH**: Authorization bypass, partial update vulnerabilities
- **DELETE**: Accidental deletion, cascade issues

### Best Practices
- Validate method permissions
- Implement proper authorization
- Use HTTPS for all methods
- Log method usage for auditing

## Implementation Patterns

### Method Dispatching
```javascript
// Node.js/Express example
app.route('/api/users/:id')
  .get((req, res) => { /* GET logic */ })
  .post((req, res) => { /* POST logic */ })
  .put((req, res) => { /* PUT logic */ })
  .patch((req, res) => { /* PATCH logic */ })
  .delete((req, res) => { /* DELETE logic */ });
```

### Method Validation
```javascript
const allowedMethods = ['GET', 'POST', 'PUT', 'DELETE'];

if (!allowedMethods.includes(req.method)) {
  res.status(405).json({ error: 'Method not allowed' });
}
```

## Related
- [[api/rest-apis/status-codes/index\|HTTP Status Codes]]
- [[api/rest-apis/content-negotiation/index\|Content Negotiation]]
- [[../api-security/index\|API Security]]
- [[../rest-apis/index\|REST APIs]]
