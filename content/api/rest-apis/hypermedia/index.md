---
{"publish":true,"aliases":"hypermedia hateoas hypermedia-apis","title":"Hypermedia APIs","enableToc":true,"created":"2025-11-16T00:28:03.524+02:00","modified":"2025-11-16T00:32:07.675+02:00","tags":["api","rest","hypermedia","hateoas"],"cssclasses":""}
---


## Overview

Hypermedia APIs (also known as Hypermedia As The Engine Of Application State - HATEOAS) provide self-descriptive responses that include links to related resources and available actions.

## Core Principles

### Self-Descriptive Messages
- Responses include links to related resources
- Clients discover available actions dynamically
- No out-of-band knowledge required

### Resource State as Hypermedia
- Application state represented through links
- Transitions between states via link following
- Server guides client through application flow

### Uniform Interface
- Consistent link formats across API
- Standardized link relations
- Predictable navigation patterns

## Link Relations

### IANA Registered Relations

#### navigation
- **Purpose**: Navigate to related resources
- **Example**: `rel="next"`, `rel="prev"`, `rel="first"`, `rel="last"`

#### item
- **Purpose**: Individual item in collection
- **Example**: Collection to individual resource

#### collection
- **Purpose**: Collection containing the resource
- **Example**: Individual resource to parent collection

#### self
- **Purpose**: Link to the resource itself
- **Example**: Canonical URL, refresh link

#### edit
- **Purpose**: Link to edit the resource
- **Example**: PUT/PATCH endpoint

#### delete
- **Purpose**: Link to delete the resource
- **Example**: DELETE endpoint

### Custom Relations

#### Domain-Specific Relations
```javascript
{
  "_links": {
    "approve": { "href": "/orders/123/approve" },
    "reject": { "href": "/orders/123/reject" },
    "ship": { "href": "/orders/123/ship" }
  }
}
```

#### Action-Based Relations
```javascript
{
  "_links": {
    "cancel-subscription": { "href": "/subscriptions/456/cancel" },
    "upgrade-plan": { "href": "/subscriptions/456/upgrade" }
  }
}
```

## Hypermedia Formats

### HAL (Hypertext Application Language)

#### Structure
```javascript
{
  "_links": {
    "self": { "href": "/orders" },
    "next": { "href": "/orders?page=2" },
    "find": { "href": "/orders{?id}", "templated": true }
  },
  "_embedded": {
    "orders": [
      {
        "_links": {
          "self": { "href": "/orders/123" },
          "customer": { "href": "/customers/456" }
        },
        "id": 123,
        "total": 99.99,
        "status": "pending"
      }
    ]
  },
  "total": 50
}
```

#### Key Features
- `_links`: Navigation links
- `_embedded`: Embedded resources
- Link templates with URI templates
- Content-Type: `application/hal+json`

### JSON API

#### Structure
```javascript
{
  "data": [
    {
      "type": "articles",
      "id": "1",
      "attributes": {
        "title": "Hypermedia APIs",
        "content": "..."
      },
      "relationships": {
        "author": {
          "data": { "type": "people", "id": "9" }
        },
        "comments": {
          "data": [
            { "type": "comments", "id": "5" },
            { "type": "comments", "id": "12" }
          ]
        }
      },
      "links": {
        "self": "/articles/1"
      }
    }
  ],
  "included": [
    {
      "type": "people",
      "id": "9",
      "attributes": {
        "name": "Author Name"
      },
      "links": {
        "self": "/people/9"
      }
    }
  ],
  "links": {
    "self": "/articles",
    "next": "/articles?page=2"
  }
}
```

#### Key Features
- Resource objects with type and id
- Relationships and included resources
- Links at resource and collection level
- Content-Type: `application/vnd.api+json`

### Siren

#### Structure
```javascript
{
  "class": ["order"],
  "properties": {
    "orderNumber": 42,
    "itemCount": 3,
    "status": "pending"
  },
  "entities": [
    {
      "class": ["items", "collection"],
      "rel": ["http://x.io/rels/order-items"],
      "href": "/orders/42/items"
    }
  ],
  "actions": [
    {
      "name": "add-item",
      "title": "Add Item",
      "method": "POST",
      "href": "/orders/42/items",
      "type": "application/x-www-form-urlencoded",
      "fields": [
        { "name": "orderNumber", "type": "hidden", "value": "42" },
        { "name": "productCode", "type": "text" },
        { "name": "quantity", "type": "number" }
      ]
    }
  ],
  "links": [
    { "rel": ["self"], "href": "/orders/42" },
    { "rel": ["next"], "href": "/orders/43" }
  ]
}
```

#### Key Features
- Classes for semantic meaning
- Actions with forms
- Entities for sub-resources
- Links with relations

### Collection+JSON

#### Structure
```javascript
{
  "collection": {
    "version": "1.0",
    "href": "/orders",
    "links": [
      { "rel": "feed", "href": "/orders" }
    ],
    "items": [
      {
        "href": "/orders/123",
        "data": [
          { "name": "orderNumber", "value": "123" },
          { "name": "total", "value": "99.99" }
        ],
        "links": [
          { "rel": "customer", "href": "/customers/456" }
        ]
      }
    ],
    "queries": [
      {
        "rel": "search",
        "href": "/orders/search",
        "data": [
          { "name": "status", "value": "" }
        ]
      }
    ],
    "template": {
      "data": [
        { "name": "customerId", "value": "" },
        { "name": "items", "value": [] }
      ]
    }
  }
}
```

#### Key Features
- Collection-centric design
- Queries for searching
- Templates for creating resources
- Writeable collections

## Implementation Patterns

### Link Generation
```javascript
class OrderResource {
  toHal() {
    return {
      _links: {
        self: { href: `/orders/${this.id}` },
        customer: { href: `/customers/${this.customerId}` },
        approve: this.canApprove() ? { href: `/orders/${this.id}/approve` } : undefined,
        ship: this.canShip() ? { href: `/orders/${this.id}/ship` } : undefined
      },
      id: this.id,
      total: this.total,
      status: this.status
    };
  }
}
```

### Conditional Links
```javascript
function addConditionalLinks(resource, user) {
  const links = {
    self: { href: `/resources/${resource.id}` }
  };
  
  if (user.canEdit(resource)) {
    links.edit = { href: `/resources/${resource.id}/edit` };
  }
  
  if (user.canDelete(resource)) {
    links.delete = { href: `/resources/${resource.id}`, method: 'DELETE' };
  }
  
  return links;
}
```

### Link Templates
```javascript
// RFC 6570 URI Templates
{
  "_links": {
    "search": {
      "href": "/users{?name,email}",
      "templated": true
    },
    "filter": {
      "href": "/users{?status,role}",
      "templated": true
    }
  }
}

// Usage
GET /users?name=john&email=john@example.com
GET /users?status=active&role=admin
```

## Benefits

### API Evolution
- **Backward Compatibility**: New links don't break old clients
- **Graceful Degradation**: Clients can ignore unknown links
- **Versioning**: Links can point to new versions

### Client Flexibility
- **Dynamic Discovery**: No hardcoded URLs
- **State Management**: Server controls application flow
- **Reduced Coupling**: Clients don't need API knowledge

### Developer Experience
- **Self-Documenting**: API teaches itself
- **Explorable**: Follow links to discover functionality
- **Testable**: Links provide test scenarios

## Challenges

### Complexity
- **Learning Curve**: More complex than simple REST
- **Implementation**: Requires careful design
- **Testing**: More scenarios to test

### Performance
- **Payload Size**: Links increase response size
- **Database Queries**: Generating links may require joins
- **Caching**: Dynamic links harder to cache

### Tooling
- **Limited Support**: Fewer tools than REST
- **Client Libraries**: Less mature ecosystems
- **Documentation**: Harder to document hypermedia

## Best Practices

### Link Design
- **Consistent Relations**: Use standard relations when possible
- **Descriptive URIs**: Self-explanatory URLs
- **Method Hints**: Include method for non-GET links

### Resource State
- **Current State**: Include current resource state
- **Available Actions**: Show what client can do next
- **Business Rules**: Reflect domain constraints

### Error Handling
- **Link Errors**: Handle broken or invalid links
- **Fallbacks**: Provide fallbacks for missing links
- **Validation**: Validate link generation

## Real-World Examples

### GitHub API (Partial HATEOAS)
```javascript
{
  "id": 1296269,
  "name": "Hello-World",
  "full_name": "octocat/Hello-World",
  "owner": {
    "login": "octocat",
    "url": "https://api.github.com/users/octocat"
  },
  "url": "https://api.github.com/repos/octocat/Hello-World",
  "html_url": "https://github.com/octocat/Hello-World",
  "clone_url": "https://github.com/octocat/Hello-World.git"
}
```

### PayPal API
```javascript
{
  "id": "PAY-1234567890",
  "create_time": "2017-09-11T22:32:05Z",
  "update_time": "2017-09-11T22:32:05Z",
  "state": "created",
  "links": [
    {
      "href": "https://api.paypal.com/v1/payments/payment/PAY-1234567890",
      "rel": "self",
      "method": "GET"
    },
    {
      "href": "https://api.paypal.com/v1/payments/payment/PAY-1234567890/execute",
      "rel": "execute",
      "method": "POST"
    }
  ]
}
```

## Tools & Libraries

### Server-Side
- **HAL**: hal/hal for PHP
- **JsonApi**: json-api-dotnet for .NET
- **Siren**: siren4j for Java
- **Collection+JSON**: Various implementations

### Client-Side
- **Traverson**: JavaScript hypermedia client
- **Hyperagent**: Ruby hypermedia client
- **SirenJS**: JavaScript Siren client

## Related
- [[../rest-apis/index\|REST APIs]]
- [[../api-design/index\|API Design]]
- [[../graphql/index\|GraphQL]]
- [[../api-documentation/index\|API Documentation]]
