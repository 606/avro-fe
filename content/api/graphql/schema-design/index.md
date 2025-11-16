---
{"publish":true,"aliases":"Graphql-Schema Schema-Design Graphql-Types","title":"Graphql Schema Design","enableToc":true,"created":"2025-11-16T00:29:31.267+02:00","modified":"2025-11-16T15:31:33.383+02:00","tags":["api","graphql","schema","design"],"cssclasses":""}
---


## Overview

GraphQL schema design is the foundation of any GraphQL API. A well-designed schema provides a clear contract between client and server, enables efficient data fetching, and supports future evolution.

## Schema Definition Language (SDL)

### Basic Schema Structure
```graphql
type Query {
  # Entry points for data fetching
}

type Mutation {
  # Entry points for data modification
}

type Subscription {
  # Entry points for real-time data
}

# Custom types
type User {
  # Fields and relationships
}
```

### Type System

#### Scalar Types
```graphql
scalar DateTime
scalar Email
scalar URL
scalar JSON

type User {
  id: ID!
  email: Email!
  createdAt: DateTime!
  metadata: JSON
}
```

#### Object Types
```graphql
type User {
  id: ID!
  name: String!
  email: String!
  posts: [Post!]!
  profile: Profile
}

type Post {
  id: ID!
  title: String!
  content: String!
  author: User!
  publishedAt: DateTime!
}
```

#### Interface Types
```graphql
interface Node {
  id: ID!
}

interface Commentable {
  comments: [Comment!]!
}

type Post implements Node & Commentable {
  id: ID!
  title: String!
  content: String!
  comments: [Comment!]!
}

type Video implements Node & Commentable {
  id: ID!
  title: String!
  url: URL!
  comments: [Comment!]!
}
```

#### Union Types
```graphql
union SearchResult = User | Post | Comment

type Query {
  search(query: String!): [SearchResult!]!
}
```

#### Enum Types
```graphql
enum UserRole {
  ADMIN
  MODERATOR
  USER
}

enum PostStatus {
  DRAFT
  PUBLISHED
  ARCHIVED
}

type User {
  role: UserRole!
}

type Post {
  status: PostStatus!
}
```

#### Input Types
```graphql
input CreateUserInput {
  name: String!
  email: String!
  password: String!
}

input UpdateUserInput {
  name: String
  email: String
}

type Mutation {
  createUser(input: CreateUserInput!): User!
  updateUser(id: ID!, input: UpdateUserInput!): User!
}
```

## Schema Design Principles

### Start with Requirements
- **Domain Analysis**: Understand business domain
- **Client Needs**: What data do clients need?
- **Use Cases**: How will the API be used?

### Design for Clients
- **Client-Centric**: Design from client perspective
- **Data Requirements**: What fields are needed together?
- **Performance**: Consider N+1 query problems

### Schema Evolution
- **Backward Compatibility**: Don't break existing clients
- **Deprecation**: Use @deprecated directive
- **Versioning**: Schema versioning strategies

## Field Design

### Naming Conventions
```graphql
# Good: camelCase for fields
type User {
  firstName: String!
  lastName: String!
  emailAddress: String!
  createdAt: DateTime!
}

# Avoid: abbreviations
type User {
  fname: String!  # Bad
  lname: String!  # Bad
  email: String!  # OK if standard
}
```

### Field Arguments
```graphql
type Query {
  # Simple argument
  user(id: ID!): User
  
  # Multiple arguments
  users(limit: Int = 10, offset: Int = 0): [User!]!
  
  # Complex arguments
  posts(
    authorId: ID
    status: PostStatus
    publishedAfter: DateTime
    sortBy: PostSort = CREATED_AT
    sortOrder: SortOrder = DESC
  ): [Post!]!
}

enum PostSort {
  CREATED_AT
  UPDATED_AT
  TITLE
}

enum SortOrder {
  ASC
  DESC
}
```

### Field Nullability
```graphql
type User {
  # Required fields
  id: ID!
  name: String!
  
  # Optional fields
  email: String
  phone: String
  
  # Required list, optional items
  posts: [Post!]!
  
  # Optional list, required items
  comments: [Comment!]
}
```

## Relationship Design

### One-to-One Relationships
```graphql
type User {
  id: ID!
  name: String!
  profile: Profile
}

type Profile {
  id: ID!
  bio: String
  avatar: URL
  user: User!
}
```

### One-to-Many Relationships
```graphql
type User {
  id: ID!
  name: String!
  posts: [Post!]!
}

type Post {
  id: ID!
  title: String!
  author: User!
}
```

### Many-to-Many Relationships
```graphql
type User {
  id: ID!
  name: String!
  following: [User!]!
  followers: [User!]!
}

type Post {
  id: ID!
  title: String!
  tags: [Tag!]!
}

type Tag {
  id: ID!
  name: String!
  posts: [Post!]!
}
```

### Connection Pattern (Relay)
```graphql
type User {
  id: ID!
  name: String!
  posts(first: Int, after: String): PostConnection!
}

type PostConnection {
  edges: [PostEdge!]!
  pageInfo: PageInfo!
}

type PostEdge {
  node: Post!
  cursor: String!
}

type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}
```

## Schema Organization

### Modular Schemas
```graphql
# schema.graphql
schema {
  query: Query
  mutation: Mutation
  subscription: Subscription
}

# Separate files
# types/
#   - User.graphql
#   - Post.graphql
#   - Comment.graphql
# - queries.graphql
# - mutations.graphql
```

### Schema Stitching
```javascript
// Combine multiple schemas
import { makeExecutableSchema } from '@graphql-tools/schema';
import userSchema from './user.graphql';
import postSchema from './post.graphql';

const schema = makeExecutableSchema({
  typeDefs: [userSchema, postSchema],
  resolvers: mergeResolvers([userResolvers, postResolvers])
});
```

## Advanced Patterns

### Generic Types
```graphql
# Generic pagination
type PaginatedResult<T> {
  items: [T!]!
  totalCount: Int!
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
}

# Usage (not valid GraphQL, but conceptual)
type Query {
  users(page: Int, limit: Int): PaginatedResult<User>!
}
```

### Custom Directives
```graphql
directive @auth(requires: Role!) on FIELD_DEFINITION
directive @deprecated(reason: String) on FIELD_DEFINITION | ENUM_VALUE

type Query {
  adminUsers: [User!]! @auth(requires: ADMIN)
  oldField: String @deprecated(reason: "Use newField instead")
}
```

### Schema Extensions
```graphql
# Base schema
type User {
  id: ID!
  name: String!
}

# Extension
extend type User {
  email: String
  posts: [Post!]!
}
```

## Schema Validation

### Type Validation
- **Strong Typing**: All fields have explicit types
- **Non-Null**: Use ! for required fields
- **Lists**: Proper list type definitions

### Schema Linting
```javascript
// GraphQL ESLint rules
{
  "@graphql-eslint/no-unused-fields": "error",
  "@graphql-eslint/require-description": ["error", { types: true }],
  "@graphql-eslint/strict-id-in-types": "error"
}
```

### Schema Testing
```javascript
const { makeExecutableSchema } = require('@graphql-tools/schema');

describe('Schema', () => {
  it('should be valid', () => {
    expect(() => makeExecutableSchema({ typeDefs, resolvers })).not.toThrow();
  });
});
```

## Performance Considerations

### Query Complexity
- **Field Costs**: Assign complexity scores
- **Depth Limits**: Prevent deeply nested queries
- **Rate Limiting**: Limit query execution time

### Schema Optimization
- **Batch Loading**: Use DataLoader for N+1 problems
- **Caching**: Cache expensive field resolutions
- **Indexing**: Ensure database indexes for queried fields

## Documentation

### Field Descriptions
```graphql
type User {
  "Unique identifier for the user"
  id: ID!
  
  "Full name of the user"
  name: String!
  
  "Email address (only visible to user)"
  email: String @auth(requires: OWNER)
}
```

### Type Documentation
```graphql
"""
Represents a user in the system
"""
type User {
  id: ID!
  name: String!
  email: String
}
```

## Tooling

### Schema Development Tools
- **GraphQL Playground**: Interactive schema exploration
- **GraphiQL**: In-browser IDE
- **Apollo Studio**: Schema management and analytics
- **GraphQL Voyager**: Visual schema exploration

### Code Generation
- **GraphQL Code Generator**: Generate types and resolvers
- **TypeScript**: Type-safe GraphQL operations
- **Apollo Client**: Generate queries and mutations

## Best Practices

### Schema Design
- **Client-First**: Design for client needs
- **Consistent Naming**: Follow conventions
- **Progressive Disclosure**: Expose complexity gradually

### Evolution
- **Additive Changes**: Prefer adding over removing
- **Deprecation**: Mark deprecated fields
- **Versioning**: Plan for breaking changes

### Performance
- **Query Analysis**: Monitor query patterns
- **Optimization**: Address N+1 query problems
- **Caching**: Implement appropriate caching strategies

## Related
- [[api/graphql/resolvers/index\|GraphQL Resolvers]]
- [[api/graphql/federation/index\|Schema Federation]]
- [[api/graphql/subscriptions/index\|GraphQL Subscriptions]]
- [[../api-design/index\|API Design]]
