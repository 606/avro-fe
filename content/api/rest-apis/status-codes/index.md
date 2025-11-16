---
{"publish":true,"aliases":"Http-Status-Codes Status-Codes Http-Codes","title":"Http Status Codes","enableToc":true,"created":"2025-11-16T00:28:03.512+02:00","modified":"2025-11-16T15:51:15.020+02:00","tags":["api","rest","http","status-codes"],"cssclasses":""}
---


## Overview

HTTP status codes indicate the result of an HTTP request. They are three-digit numbers grouped by the first digit, providing semantic information about the response.

## 1xx Informational

### 100 Continue
- **Description**: Server has received request headers, client should proceed with request body
- **Use Case**: Large file uploads, avoid sending data unnecessarily
- **Example**: Client sends POST with large body, server responds 100, client continues

### 101 Switching Protocols
- **Description**: Server is switching protocols as requested
- **Use Case**: Upgrading to WebSocket, HTTP/2
- **Example**: Upgrade: websocket header triggers protocol switch

### 102 Processing (WebDAV)
- **Description**: Server is processing request but no response available yet
- **Use Case**: Long-running operations
- **Rarely used in REST APIs**

## 2xx Success

### 200 OK
- **Description**: Request succeeded
- **Use Case**: GET, PUT, PATCH successful
- **Response Body**: Usually included
```javascript
GET /api/users/123
// 200 OK
{
  "id": 123,
  "name": "John Doe"
}
```

### 201 Created
- **Description**: Resource successfully created
- **Use Case**: POST requests that create resources
- **Headers**: Location header with new resource URL
```javascript
POST /api/users
// 201 Created
// Location: /api/users/123
{
  "id": 123,
  "name": "John Doe"
}
```

### 202 Accepted
- **Description**: Request accepted for processing, but not completed
- **Use Case**: Asynchronous operations, queued jobs
- **Response Body**: Optional processing status
```javascript
POST /api/reports/generate
// 202 Accepted
{
  "jobId": "abc-123",
  "status": "queued",
  "estimatedTime": "5 minutes"
}
```

### 203 Non-Authoritative Information
- **Description**: Response from cached/proxy source
- **Use Case**: Proxy responses, cached content
- **Rarely used in modern APIs**

### 204 No Content
- **Description**: Request succeeded, no content to return
- **Use Case**: DELETE operations, PUT updates
- **Response Body**: Empty
```javascript
DELETE /api/users/123
// 204 No Content
```

### 205 Reset Content
- **Description**: Reset form/view in browser
- **Use Case**: Form submissions
- **Rarely used in APIs**

### 206 Partial Content
- **Description**: Partial response for range requests
- **Use Case**: Large file downloads, resumable downloads
- **Headers**: Content-Range
```javascript
GET /api/files/large-video.mp4
Range: bytes=0-1023
// 206 Partial Content
// Content-Range: bytes 0-1023/1048576
```

### 207 Multi-Status (WebDAV)
- **Description**: Multiple status codes for batch operations
- **Use Case**: Bulk operations
- **Response Body**: XML with individual statuses

### 208 Already Reported (WebDAV)
- **Description**: Avoid duplicate bindings in multi-status
- **Use Case**: WebDAV directory listings

### 226 IM Used (HTTP Delta Encoding)
- **Description**: Response uses delta encoding
- **Use Case**: Bandwidth optimization

## 3xx Redirection

### 300 Multiple Choices
- **Description**: Multiple representations available
- **Use Case**: Content negotiation
- **Response Body**: List of available options

### 301 Moved Permanently
- **Description**: Resource moved permanently
- **Use Case**: API versioning, URL changes
- **Headers**: Location with new URL
- **Caching**: Can be cached indefinitely
```javascript
GET /api/v1/users
// 301 Moved Permanently
// Location: /api/v2/users
```

### 302 Found
- **Description**: Resource temporarily moved
- **Use Case**: Temporary redirects
- **Headers**: Location with current URL
- **Caching**: Not cached by default

### 303 See Other
- **Description**: Response can be found at different URI
- **Use Case**: POST-Redirect-GET pattern
- **Headers**: Location
- **Method Change**: Forces GET request

### 304 Not Modified
- **Description**: Resource not modified since last request
- **Use Case**: Conditional requests, caching
- **Headers**: ETag, Last-Modified
- **Response Body**: Empty
```javascript
GET /api/users
If-None-Match: "etag-123"
// 304 Not Modified
```

### 305 Use Proxy
- **Description**: Request must use proxy
- **Deprecated**: Security concerns

### 307 Temporary Redirect
- **Description**: Temporary redirect, same method
- **Use Case**: Temporary API endpoint changes
- **Headers**: Location
- **Method Preservation**: Keeps original method

### 308 Permanent Redirect
- **Description**: Permanent redirect, same method
- **Use Case**: Permanent API migrations
- **Headers**: Location
- **Method Preservation**: Keeps original method

## 4xx Client Error

### 400 Bad Request
- **Description**: Invalid request syntax or parameters
- **Use Case**: Validation errors, malformed JSON
- **Response Body**: Error details
```javascript
POST /api/users
{
  "name": "",
  "email": "invalid-email"
}
// 400 Bad Request
{
  "error": "Validation failed",
  "details": [
    {"field": "name", "message": "Name is required"},
    {"field": "email", "message": "Invalid email format"}
  ]
}
```

### 401 Unauthorized
- **Description**: Authentication required or failed
- **Use Case**: Missing/invalid credentials
- **Headers**: WWW-Authenticate
```javascript
GET /api/users
// 401 Unauthorized
// WWW-Authenticate: Bearer
{
  "error": "Authentication required"
}
```

### 402 Payment Required
- **Description**: Payment required for resource access
- **Use Case**: Premium API features
- **Rarely used**

### 403 Forbidden
- **Description**: Access denied despite authentication
- **Use Case**: Insufficient permissions, account suspended
```javascript
GET /api/admin/users
// 403 Forbidden
{
  "error": "Insufficient permissions"
}
```

### 404 Not Found
- **Description**: Resource does not exist
- **Use Case**: Invalid resource ID, wrong URL
```javascript
GET /api/users/999
// 404 Not Found
{
  "error": "User not found"
}
```

### 405 Method Not Allowed
- **Description**: HTTP method not supported for resource
- **Use Case**: Invalid method for endpoint
- **Headers**: Allow with supported methods
```javascript
PATCH /api/users
// 405 Method Not Allowed
// Allow: GET, POST
{
  "error": "Method not allowed"
}
```

### 406 Not Acceptable
- **Description**: Content type not acceptable
- **Use Case**: Content negotiation failures
- **Headers**: Accept with supported types

### 407 Proxy Authentication Required
- **Description**: Proxy authentication required
- **Use Case**: Corporate proxies

### 408 Request Timeout
- **Description**: Server timed out waiting for request
- **Use Case**: Slow clients, network issues

### 409 Conflict
- **Description**: Request conflicts with current state
- **Use Case**: Resource already exists, version conflicts
```javascript
POST /api/users
// 409 Conflict
{
  "error": "User with this email already exists"
}
```

### 410 Gone
- **Description**: Resource permanently removed
- **Use Case**: Deleted resources that won't return
- **Difference from 404**: Intentional permanent removal

### 411 Length Required
- **Description**: Content-Length header required
- **Use Case**: Servers requiring explicit length

### 412 Precondition Failed
- **Description**: Preconditions in headers not met
- **Use Case**: Conditional requests, optimistic locking

### 413 Payload Too Large
- **Description**: Request entity too large
- **Use Case**: File upload limits
- **Headers**: Retry-After for retry timing

### 414 URI Too Long
- **Description**: URI longer than server can handle
- **Use Case**: Excessive query parameters

### 415 Unsupported Media Type
- **Description**: Media type not supported
- **Use Case**: Wrong Content-Type header
```javascript
POST /api/users
Content-Type: text/plain
// 415 Unsupported Media Type
{
  "error": "Content-Type must be application/json"
}
```

### 416 Range Not Satisfiable
- **Description**: Range request cannot be fulfilled
- **Use Case**: Invalid byte ranges

### 417 Expectation Failed
- **Description**: Expect header cannot be met
- **Use Case**: 100-Continue expectations

### 418 I'm a Teapot (RFC 2324)
- **Description**: Server refuses to brew coffee
- **Use Case**: April Fools, testing

### 421 Misdirected Request
- **Description**: Request directed at wrong server
- **Use Case**: Domain fronting, server configuration

### 422 Unprocessable Entity
- **Description**: Request well-formed but semantically invalid
- **Use Case**: Business rule violations
```javascript
POST /api/transfers
{
  "fromAccount": "123",
  "toAccount": "123",
  "amount": 100
}
// 422 Unprocessable Entity
{
  "error": "Cannot transfer to same account"
}
```

### 423 Locked (WebDAV)
- **Description**: Resource is locked
- **Use Case**: Concurrent access control

### 424 Failed Dependency (WebDAV)
- **Description**: Request failed due to failed dependency

### 425 Too Early
- **Description**: Server unwilling to risk replay attack

### 426 Upgrade Required
- **Description**: Client must upgrade protocol
- **Use Case**: HTTP to HTTPS, HTTP/1.1 to HTTP/2

### 428 Precondition Required
- **Description**: Request must include preconditions
- **Use Case**: Conditional requests required

### 429 Too Many Requests
- **Description**: Rate limit exceeded
- **Use Case**: API rate limiting
- **Headers**: Retry-After, X-RateLimit-*
```javascript
GET /api/users
// 429 Too Many Requests
// Retry-After: 60
// X-RateLimit-Remaining: 0
{
  "error": "Rate limit exceeded"
}
```

### 431 Request Header Fields Too Large
- **Description**: Header fields too large
- **Use Case**: Oversized headers

### 451 Unavailable For Legal Reasons
- **Description**: Resource unavailable due to legal reasons
- **Use Case**: DMCA takedowns, censorship

## 5xx Server Error

### 500 Internal Server Error
- **Description**: Unexpected server error
- **Use Case**: Bugs, configuration issues
- **Logging**: Always log these errors
```javascript
GET /api/users
// 500 Internal Server Error
{
  "error": "Internal server error",
  "requestId": "abc-123-def"
}
```

### 501 Not Implemented
- **Description**: Server doesn't support requested functionality
- **Use Case**: Unsupported HTTP methods, features

### 502 Bad Gateway
- **Description**: Invalid response from upstream server
- **Use Case**: Proxy/load balancer issues

### 503 Service Unavailable
- **Description**: Server temporarily unable to handle request
- **Use Case**: Maintenance, overload
- **Headers**: Retry-After
```javascript
GET /api/users
// 503 Service Unavailable
// Retry-After: 300
{
  "error": "Service temporarily unavailable"
}
```

### 504 Gateway Timeout
- **Description**: Gateway timeout waiting for upstream response
- **Use Case**: Slow upstream services

### 505 HTTP Version Not Supported
- **Description**: HTTP version not supported
- **Use Case**: Legacy clients

### 506 Variant Also Negotiates
- **Description**: Content negotiation configuration error

### 507 Insufficient Storage (WebDAV)
- **Description**: Server cannot store representation

### 508 Loop Detected (WebDAV)
- **Description**: Infinite loop in request processing

### 510 Not Extended
- **Description**: Further extensions required

### 511 Network Authentication Required
- **Description**: Network authentication required
- **Use Case**: Captive portals

## Best Practices

### Consistent Error Responses
```javascript
// Standard error format
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format",
        "code": "INVALID_FORMAT"
      }
    ],
    "requestId": "abc-123-def",
    "timestamp": "2025-11-16T10:00:00Z"
  }
}
```

### Status Code Selection
- Use most specific code available
- Prefer 4xx for client errors, 5xx for server errors
- Document your API's status code usage

### Monitoring & Alerting
- Track status code distribution
- Alert on high 5xx rates
- Monitor 4xx rates for API issues

## Related
- [[api/rest-apis/http-methods/index\|Http Methods]]
- [[../api-design/index\|Api Design]]
- [[../api-testing/index\|Api Testing]]
- [[../rest-apis/index\|Rest Apis]]
