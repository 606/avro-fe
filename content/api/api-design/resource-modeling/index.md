---
{"publish":true,"aliases":"Resource-Modeling Api-Resources Domain-Modeling","title":"Resource Modeling","enableToc":true,"created":"2025-11-16T00:31:22.212+02:00","modified":"2025-11-16T15:31:33.382+02:00","tags":["api","design","resources","modeling"],"cssclasses":""}
---


## Overview

Resource modeling is the process of identifying and defining the entities, relationships, and operations that form the foundation of your API. Effective resource modeling creates intuitive, maintainable, and scalable APIs.

## Domain Analysis

### Business Domain Understanding
- **Stakeholder Interviews**: Understand business requirements and user needs
- **Domain Experts**: Collaborate with subject matter experts
- **Use Cases**: Document primary use cases and workflows
- **Data Flow**: Map data movement through the system

### Entity Identification
```javascript
// Domain entities in an e-commerce system
class User {
  id: string;
  email: string;
  profile: UserProfile;
  orders: Order[];
}

class Product {
  id: string;
  name: string;
  description: string;
  price: Money;
  category: Category;
  inventory: Inventory;
}

class Order {
  id: string;
  customer: User;
  items: OrderItem[];
  status: OrderStatus;
  total: Money;
}
```

### Relationship Mapping
- **One-to-One**: User has one profile
- **One-to-Many**: User has many orders
- **Many-to-Many**: Products belong to multiple categories
- **Hierarchical**: Categories have parent-child relationships

## Resource Design Principles

### Noun-Based Resources
```http
# Good - noun-based
GET /users
GET /users/123
POST /orders

# Avoid - verb-based
GET /getUsers
POST /createOrder
```

### Resource Hierarchy
```http
# Hierarchical resources
/organizations/{orgId}/users/{userId}
/users/{userId}/orders/{orderId}
/products/{productId}/reviews/{reviewId}
```

### Resource Lifecycle
- **Creation**: POST to collection
- **Retrieval**: GET from collection or individual
- **Update**: PUT/PATCH to individual
- **Deletion**: DELETE from individual

## Resource Representations

### JSON Schema Design
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "id": {
      "type": "string",
      "format": "uuid"
    },
    "name": {
      "type": "string",
      "minLength": 1,
      "maxLength": 100
    },
    "email": {
      "type": "string",
      "format": "email"
    },
    "createdAt": {
      "type": "string",
      "format": "date-time"
    },
    "updatedAt": {
      "type": "string",
      "format": "date-time"
    }
  },
  "required": ["id", "name", "email"]
}
```

### Field Naming Conventions
```javascript
// Consistent naming patterns
{
  // Use camelCase for JavaScript clients
  "userId": "123",
  "firstName": "John",
  "lastName": "Doe",
  "emailAddress": "john@example.com",
  
  // Or snake_case for consistency
  "user_id": "123",
  "first_name": "John",
  "last_name": "Doe",
  "email_address": "john@example.com"
}
```

### Metadata Inclusion
```json
{
  "data": {
    "id": "123",
    "name": "John Doe",
    "email": "john@example.com"
  },
  "meta": {
    "createdAt": "2023-01-01T00:00:00Z",
    "updatedAt": "2023-01-02T00:00:00Z",
    "version": "1.0"
  },
  "links": {
    "self": "/users/123",
    "orders": "/users/123/orders"
  }
}
```

## Relationship Handling

### Embedded Resources
```json
// Embedded relationships
{
  "id": "123",
  "name": "John Doe",
  "profile": {
    "bio": "Software developer",
    "avatar": "https://example.com/avatar.jpg"
  },
  "recentOrders": [
    {
      "id": "456",
      "total": 99.99,
      "status": "delivered"
    }
  ]
}
```

### Linked Resources
```json
// Linked relationships (HATEOAS)
{
  "id": "123",
  "name": "John Doe",
  "_links": {
    "self": { "href": "/users/123" },
    "profile": { "href": "/users/123/profile" },
    "orders": { "href": "/users/123/orders" },
    "recentOrders": {
      "href": "/users/123/orders{?limit,offset}",
      "templated": true
    }
  }
}
```

### Reference IDs
```json
// Simple ID references
{
  "id": "123",
  "name": "John Doe",
  "profileId": "456",
  "orderIds": ["789", "012"]
}
```

## Sub-Resource Design

### Nested Resources
```http
# User posts
GET /users/123/posts
POST /users/123/posts
GET /users/123/posts/456
PUT /users/123/posts/456
DELETE /users/123/posts/456

# Product reviews
GET /products/789/reviews
POST /products/789/reviews
```

### Sub-Resource Guidelines
- Use when relationship is strong and stable
- Keep nesting shallow (max 2-3 levels)
- Consider performance implications
- Provide direct access when needed

## Collection Resources

### Collection Operations
```http
# List all users
GET /users

# Create new user
POST /users

# Bulk operations
POST /users/batch
DELETE /users/batch
```

### Collection Metadata
```json
{
  "data": [
    { "id": "1", "name": "John" },
    { "id": "2", "name": "Jane" }
  ],
  "meta": {
    "total": 100,
    "page": 1,
    "limit": 10,
    "totalPages": 10
  },
  "links": {
    "self": "/users?page=1&limit=10",
    "next": "/users?page=2&limit=10",
    "prev": null,
    "first": "/users?page=1&limit=10",
    "last": "/users?page=10&limit=10"
  }
}
```

## Action Resources

### Resource Actions
```http
# User actions
POST /users/123/activate
POST /users/123/deactivate
POST /users/123/reset-password

# Order actions
POST /orders/456/cancel
POST /orders/456/ship
POST /orders/456/refund
```

### Action Design Guidelines
- Use for operations that don't fit CRUD
- Return appropriate status codes
- Document action parameters clearly
- Consider using job resources for async operations

## Versioning Resources

### Resource Evolution
```javascript
// Version 1
{
  "id": "123",
  "name": "John Doe",
  "email": "john@example.com"
}

// Version 2 (additive changes)
{
  "id": "123",
  "name": "John Doe",
  "email": "john@example.com",
  "phone": "+1-555-0123"
}

// Version 3 (breaking changes - use versioning)
{
  "id": "123",
  "fullName": "John Doe",  // renamed field
  "contact": {
    "email": "john@example.com",
    "phone": "+1-555-0123"
  }
}
```

### Backward Compatibility
- Add optional fields
- Avoid removing fields
- Deprecate gradually
- Use versioning for breaking changes

## Validation and Constraints

### Input Validation
```javascript
// Input validation rules
const userSchema = {
  name: {
    type: 'string',
    required: true,
    minLength: 1,
    maxLength: 100
  },
  email: {
    type: 'string',
    required: true,
    format: 'email',
    unique: true
  },
  age: {
    type: 'number',
    minimum: 0,
    maximum: 150
  }
};
```

### Business Rules
```javascript
// Business rule validation
function validateOrder(order) {
  // User must exist
  if (!userExists(order.userId)) {
    throw new ValidationError('Invalid user');
  }
  
  // Sufficient inventory
  for (const item of order.items) {
    if (!hasInventory(item.productId, item.quantity)) {
      throw new ValidationError('Insufficient inventory');
    }
  }
  
  // Valid payment method
  if (!isValidPayment(order.payment)) {
    throw new ValidationError('Invalid payment method');
  }
}
```

## Performance Considerations

### Resource Granularity
```javascript
// Fine-grained resources (flexible but chatty)
GET /users/123
GET /users/123/profile
GET /users/123/preferences

// Coarse-grained resources (efficient but less flexible)
GET /users/123?include=profile,preferences
```

### Caching Strategies
```http
# Cache headers for resources
GET /users/123
Cache-Control: max-age=300
ETag: "abc123"

GET /products/popular
Cache-Control: max-age=60
```

### Lazy Loading
```json
// Lazy-loaded relationships
{
  "id": "123",
  "name": "John Doe",
  "orders": {
    "href": "/users/123/orders",
    "count": 5
  }
}
```

## Documentation

### Resource Documentation
```yaml
# OpenAPI resource definition
User:
  type: object
  properties:
    id:
      type: string
      format: uuid
      description: Unique user identifier
    name:
      type: string
      minLength: 1
      maxLength: 100
      description: User's full name
    email:
      type: string
      format: email
      description: User's email address
  required:
    - id
    - name
    - email
```

### API Examples
```javascript
// Client usage examples
// Create user
const user = await api.post('/users', {
  name: 'John Doe',
  email: 'john@example.com'
});

// Get user with relationships
const user = await api.get('/users/123?include=profile,orders');

// Update user
await api.patch('/users/123', {
  name: 'Jane Doe'
});
```

## Common Patterns

### Content Management
```http
# Articles and content
GET /articles
GET /articles/123
GET /articles/123/comments
POST /articles/123/publish
```

### E-commerce
```http
# Products and orders
GET /products?category=electronics
POST /cart/items
POST /orders
GET /orders/123/fulfillment
```

### Social Media
```http
# Users and content
GET /users/123/followers
POST /posts/456/like
GET /feed?algorithm=chronological
```

## Anti-Patterns

### Over-Nesting
```http
# Avoid deep nesting
GET /organizations/1/departments/2/teams/3/members/4/tasks/5/comments

# Prefer flatter structures
GET /comments?task=5
```

### Inconsistent Naming
```http
# Inconsistent resource names
GET /users
GET /CustomerDetails  # Different naming convention
GET /order-items      # Inconsistent pluralization
```

### Tight Coupling
```http
# Avoid client-specific resources
GET /mobile-app-users  # Client-specific
GET /web-users         # Client-specific

# Use consistent resources
GET /users?client=mobile
```

## Testing Resource Models

### Contract Testing
```javascript
// Test resource contracts
describe('User Resource', () => {
  it('should return user with required fields', async () => {
    const response = await request(app)
      .get('/users/123')
      .expect(200);
    
    expect(response.body).toHaveProperty('id');
    expect(response.body).toHaveProperty('name');
    expect(response.body).toHaveProperty('email');
  });
  
  it('should create user with valid input', async () => {
    const response = await request(app)
      .post('/users')
      .send({
        name: 'John Doe',
        email: 'john@example.com'
      })
      .expect(201);
    
    expect(response.body).toHaveProperty('id');
  });
});
```

### Integration Testing
```javascript
// Test resource relationships
describe('User-Order Relationship', () => {
  it('should link user to orders', async () => {
    // Create user
    const user = await createUser();
    
    // Create order for user
    const order = await createOrder({ userId: user.id });
    
    // Verify relationship
    const userOrders = await getUserOrders(user.id);
    expect(userOrders).toContain(order);
  });
});
```

## Related
- [[api/api-design/index\|API Design]]
- [[api/api-design/url-design/index\|URL Design Patterns]]
- [[../versioning/index\|API Versioning]]
- [[../pagination/index\|Pagination Strategies]]
