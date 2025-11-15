---
{"publish":true,"aliases":"content-negotiation media-types accept-headers","title":"Content Negotiation","enableToc":true,"created":"2025-11-16T00:28:03.519+02:00","modified":"2025-11-16T00:32:07.670+02:00","tags":["api","rest","content-negotiation","http"],"cssclasses":""}
---


## Overview

Content negotiation allows clients and servers to agree on the format of data exchanged. It enables APIs to serve different representations of the same resource based on client preferences.

## HTTP Headers

### Accept
- **Purpose**: Specify preferred response media types
- **Server Response**: Content-Type matching client preference
- **Quality Values**: q parameter for preference weighting

```javascript
// Single type
Accept: application/json

// Multiple types with quality
Accept: application/json; q=1.0, text/html; q=0.8

// Any type
Accept: */*

// Specific vendor types
Accept: application/vnd.api+json
```

### Content-Type
- **Purpose**: Specify request body media type
- **Client Sends**: In request body
- **Server Validates**: Ensures correct format

```javascript
// JSON request
Content-Type: application/json

// Form data
Content-Type: application/x-www-form-urlencoded

// Multipart form data
Content-Type: multipart/form-data
```

### Accept-Language
- **Purpose**: Specify preferred language for response
- **Use Case**: Internationalization, localization
- **Fallback**: Default language when preferred not available

```javascript
Accept-Language: en-US, fr-FR; q=0.5, de; q=0.3
```

### Accept-Encoding
- **Purpose**: Specify acceptable content encodings
- **Common Values**: gzip, deflate, br (Brotli)
- **Server Response**: Content-Encoding header

```javascript
Accept-Encoding: gzip, deflate, br
// Response: Content-Encoding: gzip
```

### Accept-Charset
- **Purpose**: Specify acceptable character sets
- **Default**: UTF-8
- **Rarely used**: Most APIs use UTF-8

## Media Types

### Standard Types

#### application/json
- **Description**: JSON data format
- **Usage**: Most common API format
- **Pros**: Human-readable, widely supported
- **Cons**: Verbose, no schema validation

```javascript
Content-Type: application/json
{
  "users": [
    {
      "id": 1,
      "name": "John Doe",
      "email": "john@example.com"
    }
  ]
}
```

#### application/xml
- **Description**: XML data format
- **Usage**: Legacy systems, financial APIs
- **Pros**: Schema validation (XSD), namespaces
- **Cons**: Verbose, complex parsing

```xml
Content-Type: application/xml
<users>
  <user id="1">
    <name>John Doe</name>
    <email>john@example.com</email>
  </user>
</users>
```

#### application/x-www-form-urlencoded
- **Description**: URL-encoded form data
- **Usage**: HTML forms, simple key-value data
- **Pros**: Simple, URL-safe
- **Cons**: No complex data structures

```javascript
Content-Type: application/x-www-form-urlencoded
name=John+Doe&email=john%40example.com
```

#### multipart/form-data
- **Description**: Multipart form data with boundaries
- **Usage**: File uploads, complex forms
- **Pros**: Supports files and complex data
- **Cons**: Complex parsing, larger payloads

```javascript
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary

------WebKitFormBoundary
Content-Disposition: form-data; name="name"

John Doe
------WebKitFormBoundary
Content-Disposition: form-data; name="avatar"; filename="photo.jpg"
Content-Type: image/jpeg

(binary data)
------WebKitFormBoundary--
```

### Vendor Media Types

#### JSON API (application/vnd.api+json)
- **Description**: Standardized JSON API format
- **Pros**: Consistent structure, includes relationships
- **Cons**: More verbose than plain JSON

```javascript
Content-Type: application/vnd.api+json
{
  "data": [
    {
      "type": "users",
      "id": "1",
      "attributes": {
        "name": "John Doe",
        "email": "john@example.com"
      }
    }
  ]
}
```

#### HAL (application/hal+json)
- **Description**: Hypertext Application Language
- **Pros**: Built-in hypermedia links
- **Cons**: Additional complexity

```javascript
Content-Type: application/hal+json
{
  "_links": {
    "self": { "href": "/users" },
    "next": { "href": "/users?page=2" }
  },
  "_embedded": {
    "users": [
      {
        "id": 1,
        "name": "John Doe",
        "_links": {
          "self": { "href": "/users/1" }
        }
      }
    ]
  }
}
```

#### Siren (application/vnd.siren+json)
- **Description**: Hypermedia format with actions
- **Pros**: Includes available actions
- **Cons**: Complex specification

## Content Negotiation Strategies

### Server-Driven Negotiation
- **Approach**: Server chooses best representation
- **Headers**: Accept, Accept-Language, Accept-Encoding
- **Pros**: Simple implementation
- **Cons**: Server makes assumptions

```javascript
// Client request
GET /api/users
Accept: application/json, application/xml; q=0.5

// Server response (chooses JSON)
Content-Type: application/json
```

### Agent-Driven Negotiation
- **Approach**: Client chooses from options
- **Status Code**: 300 Multiple Choices or 406 Not Acceptable
- **Pros**: Client has full control
- **Cons**: Multiple round trips

```javascript
// Server offers choices
HTTP/1.1 300 Multiple Choices
Content-Type: application/json

{
  "choices": [
    {
      "type": "application/json",
      "href": "/api/users?format=json"
    },
    {
      "type": "application/xml",
      "href": "/api/users?format=xml"
    }
  ]
}
```

### Transparent Negotiation
- **Approach**: Proxy handles negotiation
- **Headers**: Negotiate, TCN, Variant-Vary
- **Rarely used**: Complex implementation

## Implementation Patterns

### Express.js Content Negotiation
```javascript
const express = require('express');
const app = express();

// Accept header parsing
app.get('/api/users', (req, res) => {
  const accepts = req.accepts(['json', 'xml', 'html']);
  
  if (accepts === 'json') {
    res.json({ users: [] });
  } else if (accepts === 'xml') {
    res.type('xml').send('<users></users>');
  } else {
    res.status(406).send('Not Acceptable');
  }
});
```

### ASP.NET Core Content Negotiation
```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    [HttpGet]
    public IActionResult GetUsers()
    {
        var users = _userService.GetUsers();
        
        // Automatic content negotiation
        return Ok(users);
    }
}
```

### Custom Content Negotiation
```javascript
function negotiateContent(req, res, options) {
  const accept = req.headers.accept;
  
  for (const option of options) {
    if (accept.includes(option.type)) {
      res.type(option.type);
      return option.handler();
    }
  }
  
  res.status(406).json({ error: 'Not Acceptable' });
}
```

## Quality Values (q-values)

### Syntax
```
type/subtype; q=value
```

### Examples
```javascript
// Prefer JSON, accept XML as fallback
Accept: application/json; q=1.0, application/xml; q=0.5

// Prefer English, accept French
Accept-Language: en-US; q=1.0, fr-FR; q=0.8

// Prefer gzip, accept deflate
Accept-Encoding: gzip; q=1.0, deflate; q=0.8
```

### Processing Rules
1. Sort by quality value (highest first)
2. Use first supported type
3. Default q=1.0 if not specified
4. q=0 means "not acceptable"

## Common Issues

### Missing Accept Header
- **Problem**: Clients don't specify preferences
- **Solution**: Default to JSON or configurable default
- **Best Practice**: Document default behavior

### Conflicting Quality Values
- **Problem**: Multiple types with same q-value
- **Solution**: Server-defined priority order
- **RFC 7231**: Server chooses first in list

### Vendor Type Conflicts
- **Problem**: application/vnd.company+json vs standard types
- **Solution**: Clear versioning strategy
- **Best Practice**: Use structured versioning

## Best Practices

### API Design
- **Document Supported Types**: Clear documentation
- **Consistent Defaults**: Same default across endpoints
- **Version in Media Types**: application/vnd.api.v1+json

### Implementation
- **Validate Content-Type**: Reject unsupported formats
- **Quality Value Support**: Honor client preferences
- **Caching Considerations**: Vary header for caching

### Testing
- **Test All Formats**: Validate each supported type
- **Edge Cases**: Invalid Accept headers, quality values
- **Client Compatibility**: Test with various clients

## Related
- [[api/rest-apis/http-methods/index\|HTTP Methods]]
- [[api/rest-apis/status-codes/index\|HTTP Status Codes]]
- [[../api-design/index\|API Design]]
- [[../api-documentation/index\|API Documentation]]
