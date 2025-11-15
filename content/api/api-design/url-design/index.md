---
{"publish":true,"aliases":"url-design api-urls rest-urls","title":"URL Design Patterns","enableToc":true,"created":"2025-11-16T00:31:38.323+02:00","modified":"2025-11-16T00:32:07.701+02:00","tags":["api","design","urls","rest"],"cssclasses":""}
---


## Overview

URL design is crucial for creating intuitive, maintainable, and scalable APIs. Well-designed URLs provide clear resource hierarchies, support proper HTTP methods, and enable efficient client-side caching and linking.

## URL Structure Basics

### RESTful URL Components
```
https://api.example.com/v1/users/123/posts/456/comments
┌─────────────────────┐┌──┐┌──────┐┌───┐┌─────┐┌────────┐
│         Base URL    ││API││Resource││ID ││Sub- ││Action  │
│                     ││Ver││       ││   ││res. ││        │
└─────────────────────┘└──┘└──────┘└───┘└─────┘└────────┘
```

### URL Encoding
- **Reserved Characters**: Encode special characters (`%20` for space)
- **Unicode Support**: Use UTF-8 encoding
- **Case Sensitivity**: URLs are case-sensitive (prefer lowercase)
- **Length Limits**: Keep URLs under 2048 characters

## Resource-Based URLs

### Collection Resources
```http
# List all users
GET /users

# Create new user
POST /users

# Search users
GET /users?search=john&status=active

# Bulk operations
POST /users/batch
DELETE /users/batch
```

### Individual Resources
```http
# Get specific user
GET /users/123

# Update user
PUT /users/123
PATCH /users/123

# Delete user
DELETE /users/123

# Check existence
HEAD /users/123
```

### Sub-Resources
```http
# User's posts
GET /users/123/posts
POST /users/123/posts

# Specific post
GET /users/123/posts/456
PUT /users/123/posts/456

# Post comments
GET /users/123/posts/456/comments
POST /users/123/posts/456/comments
```

## Hierarchical URLs

### Nested Resources
```http
# Organization hierarchy
/organizations/{orgId}/departments/{deptId}/employees/{empId}

/products/{productId}/variants/{variantId}/images/{imageId}

/projects/{projectId}/issues/{issueId}/comments/{commentId}
```

### Hierarchical Guidelines
- **Depth Limit**: Keep nesting to 3 levels maximum
- **Relationship Strength**: Use for strong, stable relationships
- **Alternative Access**: Provide direct access to nested resources
- **Performance**: Consider query complexity

## Action-Based URLs

### Resource Actions
```http
# User actions
POST /users/123/activate
POST /users/123/deactivate
POST /users/123/reset-password

# Document actions
POST /documents/456/publish
POST /documents/456/archive
POST /documents/456/share
```

### Action URL Patterns
```http
# Verb-based actions
POST /users/{id}/send-verification
POST /orders/{id}/process-payment
POST /files/{id}/generate-thumbnail

# Noun-based actions (preferred)
POST /users/{id}/verification-emails
POST /orders/{id}/payments
POST /files/{id}/thumbnails
```

## Query Parameters

### Filtering
```http
# Simple filters
GET /users?status=active
GET /products?category=electronics&brand=apple

# Range filters
GET /orders?created_after=2023-01-01&created_before=2023-12-31
GET /products?price_min=100&price_max=500

# Array filters
GET /users?roles[]=admin&roles[]=moderator
GET /products?tags=wireless&tags=bluetooth
```

### Sorting
```http
# Single field sorting
GET /users?sort=name
GET /products?sort=price&order=desc

# Multiple field sorting
GET /users?sort=name,created_at&order=asc,desc
GET /products?sort=popularity desc,price asc
```

### Pagination
```http
# Offset-based
GET /users?page=2&limit=10

# Cursor-based
GET /users?cursor=abc123&limit=10

# Page-based
GET /users?page=2&per_page=10
```

### Field Selection
```http
# Sparse fieldsets
GET /users?fields=id,name,email
GET /products?fields=id,name,price&include=category

# Field expansion
GET /users/123?expand=profile,preferences
GET /orders/456?expand=items.product,shipping_address
```

## Search and Advanced Queries

### Search Parameters
```http
# Full-text search
GET /articles?search=machine%20learning

# Field-specific search
GET /users?q=name:john+email:*@example.com

# Fuzzy search
GET /products?search~="wireless headphones"
```

### Advanced Filtering
```http
# Complex queries
GET /orders?filter=status:pending,shipped&sort=-created_at

# JSON-based filters
GET /users?filter={"age":{"$gte":18},"country":"US"}

# Query DSL
GET /products?q=category:electronics AND price:<100
```

## Versioning in URLs

### URL Path Versioning
```http
# API versioning
/api/v1/users
/api/v2/users

# Resource versioning
/users/v1/profile
/posts/v2/content
```

### Versioning Strategies
```http
# Major version in path
/v1/users
/v2/users

# Date-based versioning
/2023-01-01/users
/2023-06-01/users

# Media type versioning
/users (with Accept header)
```

## Matrix Parameters

### Matrix Parameters Usage
```http
# Filtering with matrix params
/users;status=active;role=admin

# Multiple values
/products;category=electronics;brand=apple,samsung

# Hierarchical filtering
/organizations;country=us/departments;type=engineering/employees
```

### Matrix vs Query Parameters
```http
# Query parameters (preferred for most cases)
/users?status=active&role=admin

# Matrix parameters (for hierarchical filtering)
/users;status=active/roles;type=admin
```

## URL Templates

### RFC 6570 URI Templates
```http
# Simple expansion
/users/{id}
/posts/{id}/comments/{commentId}

# Query expansion
/users{?fields,limit,offset}
/search{?q,limit,sort}

# Fragment expansion
/users/{id}{#section}
/documents/{id}{#page}
```

### Template Examples
```javascript
// URI template expansion
const userTemplate = '/users/{id}{?fields}';
const expanded = userTemplate
  .replace('{id}', '123')
  .replace('{?fields}', '?fields=name,email');

// Result: /users/123?fields=name,email
```

## Special Resource Types

### Singleton Resources
```http
# User profile (one per user)
/users/123/profile

# Application configuration
/config

# Current user
/me
/users/current
```

### Bulk Operations
```http
# Bulk create
POST /users/bulk

# Bulk update
PATCH /users/bulk

# Bulk delete
DELETE /users/bulk
```

### Batch Operations
```http
# Batch requests
POST /batch

# With body:
{
  "requests": [
    {"method": "GET", "url": "/users/1"},
    {"method": "POST", "url": "/users", "body": {...}}
  ]
}
```

## Content Negotiation

### Format Specification
```http
# Format in extension
GET /users.json
GET /users.xml
GET /reports.pdf

# Format in query
GET /users?format=json
GET /users?format=csv

# Accept header (preferred)
GET /users
Accept: application/json
```

### API Content Types
```http
# Standard content types
Accept: application/json
Accept: application/xml
Accept: text/csv
Accept: application/pdf

# Custom content types
Accept: application/vnd.company.user.v1+json
Accept: application/vnd.company.report.v2+xml
```

## Error Handling URLs

### Error Resource URLs
```http
# Error details
GET /errors/validation/123
GET /errors/authentication/456

# Error types
GET /errors?type=validation&code=INVALID_EMAIL
```

## URL Design Best Practices

### Consistency
```http
# Consistent patterns
GET /users/{id}/posts
GET /users/{id}/comments
GET /users/{id}/followers

# Avoid mixing patterns
GET /users/{id}/posts     # Good
GET /posts?user_id={id}   # Inconsistent
```

### Clarity
```http
# Clear, descriptive names
GET /users/{id}/shipping-addresses
GET /orders/{id}/fulfillment-status

# Avoid abbreviations
GET /users/{id}/addr      # Unclear
GET /orders/{id}/status   # Ambiguous
```

### Stability
```http
# Stable URLs
/api/v1/users/{id}

/users/{id}  # Avoid if planning versioning
/v1/users/{id}  # Avoid if changing base paths
```

## SEO and Discovery

### URL-Friendly Design
```http
# Human-readable URLs
GET /articles/machine-learning-basics
GET /products/wireless-headphones

# Avoid
GET /articles/123
GET /products?category=electronics&id=456
```

### Link Relations
```http
# Link headers
Link: </users/123>; rel="self"
Link: </users/123/posts>; rel="related"
Link: </users?page=2>; rel="next"
```

## Caching Considerations

### Cache-Friendly URLs
```http
# Static resources
GET /users/123  # Cacheable
GET /users/search?q=john  # Less cacheable

# Cache-busting
GET /assets/app.js?v=1.2.3
GET /images/avatar.jpg?t=1640995200
```

### Cache Headers
```http
# Cache control
Cache-Control: max-age=300, public
ETag: "abc123"
Last-Modified: Wed, 21 Oct 2023 07:28:00 GMT
```

## Internationalization

### Localized URLs
```http
# Language in path
/en/users
/fr/utilisateurs
/de/benutzer

# Language in subdomain
en.api.example.com/users
fr.api.example.com/utilisateurs
```

### URL Encoding
```http
# Unicode in URLs
GET /users/M%C3%A9lanie  # Mélanie
GET /products/caf%C3%A9  # café

# Avoid non-ASCII in URLs when possible
GET /users/melanie
GET /products/cafe
```

## URL Rewriting and Proxies

### Reverse Proxy Configuration
```nginx
# URL rewriting
location /api/v1/ {
  proxy_pass http://backend:8080/;
  rewrite ^/api/v1/(.*) /$1 break;
}

# Path stripping
location /api/ {
  proxy_pass http://api-server:3000/;
  proxy_set_header X-Forwarded-Prefix /api;
}
```

### Load Balancer Setup
```haproxy
# Path-based routing
acl is_api path_beg /api/
use_backend api_backend if is_api

# Host-based routing
acl is_mobile hdr(host) -i mobile.api.example.com
use_backend mobile_backend if is_mobile
```

## Testing URL Patterns

### URL Validation
```javascript
// URL pattern testing
describe('URL Patterns', () => {
  const validUrls = [
    '/users',
    '/users/123',
    '/users/123/posts',
    '/users?status=active',
    '/users?page=2&limit=10'
  ];
  
  const invalidUrls = [
    '/Users',        // Wrong case
    '/users/123/',   // Trailing slash
    '/usr/123',      // Wrong resource name
  ];
  
  validUrls.forEach(url => {
    it(`should accept ${url}`, () => {
      expect(isValidUrl(url)).toBe(true);
    });
  });
});
```

### Route Testing
```javascript
// Route handler testing
describe('User Routes', () => {
  it('should handle user collection', async () => {
    const response = await request(app)
      .get('/users')
      .expect(200);
    
    expect(Array.isArray(response.body)).toBe(true);
  });
  
  it('should handle individual user', async () => {
    const response = await request(app)
      .get('/users/123')
      .expect(200);
    
    expect(response.body).toHaveProperty('id', '123');
  });
});
```

## Common URL Patterns

### E-commerce APIs
```http
# Product catalog
GET /products
GET /products/{id}
GET /categories/{id}/products
GET /products?category=electronics&brand=apple

# Shopping cart
GET /cart
POST /cart/items
PUT /cart/items/{id}
DELETE /cart/items/{id}

# Orders
GET /orders
POST /orders
GET /orders/{id}
POST /orders/{id}/cancel
```

### Social Media APIs
```http
# Users and relationships
GET /users/{id}/followers
GET /users/{id}/following
POST /users/{id}/follow
DELETE /users/{id}/follow

# Content
GET /posts
GET /posts/{id}
GET /users/{id}/posts
POST /posts/{id}/like
POST /posts/{id}/comments
```

### Content Management
```http
# Articles and content
GET /articles
GET /articles/{slug}
GET /articles?category=tech&published=true
POST /articles/{id}/publish
POST /articles/{id}/unpublish

# Media management
GET /media
POST /media/upload
GET /media/{id}/download
DELETE /media/{id}
```

## Anti-Patterns

### Verb-Based URLs
```http
# Avoid verb-based URLs
GET /getUsers
POST /createUser
PUT /updateUser
DELETE /removeUser

# Use resource-based
GET /users
POST /users
PUT /users/{id}
DELETE /users/{id}
```

### Inconsistent Naming
```http
# Inconsistent pluralization
GET /user/123     # Singular
GET /posts        # Plural

# Inconsistent naming
GET /users
GET /Customer     # Different convention
GET /order_items  # Different separator
```

### Query Parameter Abuse
```http
# Avoid complex logic in query params
GET /users?action=delete&id=1,2,3,4,5

# Use proper resources
DELETE /users/batch
// with body: { "ids": [1,2,3,4,5] }
```

### Deep Nesting
```http
# Avoid deep nesting
GET /orgs/1/depts/2/teams/3/projects/4/tasks/5/comments/6

# Provide direct access
GET /comments/6
GET /comments?task=5
```

## Related
- [[api/api-design/index\|API Design]]
- [[api/api-design/resource-modeling/index\|Resource Modeling]]
- [[../versioning/index\|API Versioning]]
- [[../pagination/index\|Pagination Strategies]]
