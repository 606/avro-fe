---
{"publish":true,"aliases":"Graphql Graphql Graph-Ql","title":"Graphql","enableToc":true,"created":"2025-11-16T00:25:07.546+02:00","modified":"2025-11-16T15:31:33.383+02:00","tags":["api","graphql","query-language"],"cssclasses":""}
---


## Overview

GraphQL is a query language for APIs that allows clients to request exactly the data they need, making APIs more efficient and flexible.

## Core Concepts

### Schema
- Defines available data and operations
- Strongly typed with SDL (Schema Definition Language)
- Self-documenting API

### Queries
- Read operations to fetch data
- Declarative data fetching
- Nested and related data in single request

### Mutations
- Write operations to modify data
- Similar to queries but for changes
- Return modified data

### Subscriptions
- Real-time data updates
- WebSocket-based communication
- Push-based data flow

## Schema Definition

### Types
```graphql
type User {
  id: ID!
  name: String!
  email: String!
  posts: [Post!]!
}

type Post {
  id: ID!
  title: String!
  content: String!
  author: User!
  publishedAt: DateTime!
}
```

### Queries
```graphql
type Query {
  users: [User!]!
  user(id: ID!): User
  posts(limit: Int): [Post!]!
}
```

### Mutations
```graphql
type Mutation {
  createUser(input: CreateUserInput!): User!
  updateUser(id: ID!, input: UpdateUserInput!): User!
  deleteUser(id: ID!): Boolean!
}
```

## Query Examples

### Basic Query
```graphql
query {
  user(id: "123") {
    name
    email
  }
}
```

### Nested Query
```graphql
query {
  user(id: "123") {
    name
    posts {
      title
      publishedAt
    }
  }
}
```

### Query with Arguments
```graphql
query {
  posts(limit: 10) {
    title
    author {
      name
    }
  }
}
```

### Mutation
```graphql
mutation {
  createUser(input: {
    name: "John Doe"
    email: "john@example.com"
  }) {
    id
    name
    email
  }
}
```

## Advantages over REST

### Precise Data Fetching
- No over-fetching: Get exactly what's needed
- No under-fetching: Single request for related data
- Reduced network overhead

### Single Endpoint
- All operations through `/graphql`
- Versioning through schema evolution
- Simplified client-server communication

### Strong Typing
- Compile-time query validation
- Better IDE support and tooling
- Self-documenting APIs

### Real-time Capabilities
- Built-in subscriptions for live data
- WebSocket transport for real-time updates

## Implementation

### Popular Libraries
- **Apollo Server** (Node.js)
- **GraphQL Yoga** (Node.js)
- **Strawberry** (Python)
- **Graphene** (Python)
- **Hot Chocolate** (.NET)

### Schema Stitching
- Combine multiple GraphQL schemas
- Federation for microservices
- Schema composition

## Best Practices

### Schema Design
- Use descriptive field names
- Implement proper pagination
- Handle errors consistently
- Version schema evolution

### Security
- Query complexity limits
- Depth limiting
- Authentication and authorization
- Rate limiting

### Performance
- Dataloader for batching
- Caching strategies
- Query optimization
- Monitoring and metrics

## Detailed Topics

### Schema Design
- [[api/graphql/schema-design/index\|Schema Design]] - Designing effective GraphQL schemas with types, interfaces, unions, and best practices

### Resolvers
- [[api/graphql/resolvers/index\|Resolvers]] - Implementing resolver functions, data loading patterns, error handling, and performance optimization

### Federation
- [[api/graphql/federation/index\|Federation]] - Schema federation for microservices, entity composition, and cross-service data management

### Subscriptions
- [[api/graphql/subscriptions/index\|Subscriptions]] - Real-time GraphQL with WebSocket connections, pub/sub patterns, and live data updates

## Related
- [[api/rest-apis/index\|REST APIs]]
- [[api/api-design/index\|API Design]]
- [[api/microservices/index\|Microservices]]
- [[api/api-testing/index\|API Testing]]
