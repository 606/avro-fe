---
{"publish":true,"aliases":"graphql-federation schema-federation apollo-federation","title":"GraphQL Schema Federation","enableToc":true,"created":"2025-11-16T00:29:31.324+02:00","modified":"2025-11-16T00:32:07.691+02:00","tags":["api","graphql","federation","microservices"],"cssclasses":""}
---


## Overview

Schema federation allows multiple GraphQL services to compose into a single unified schema. It's particularly useful in microservices architectures where different teams or services own different parts of the schema.

## Federation Concepts

### Subgraphs
- **Definition**: Individual GraphQL services with their own schemas
- **Ownership**: Each subgraph is owned by a specific team or service
- **Independence**: Subgraphs can be developed and deployed independently

### Gateway
- **Definition**: Central GraphQL server that composes subgraphs
- **Composition**: Combines subgraph schemas into a unified schema
- **Routing**: Routes queries to appropriate subgraphs

### Entities
- **Definition**: Types that can be extended across subgraphs
- **Keys**: Unique identifiers for cross-subgraph references
- **Extensions**: Adding fields to entities from other subgraphs

## Federation Directives

### @key
- **Purpose**: Defines entity keys for cross-subgraph references
- **Usage**: Marks fields that uniquely identify an entity
- **Multiple Keys**: Entities can have multiple keys

```graphql
type User @key(fields: "id") @key(fields: "email") {
  id: ID!
  email: String!
  name: String!
}
```

### @extends
- **Purpose**: Extends an entity defined in another subgraph
- **Usage**: Adds fields to entities from other subgraphs
- **Requirements**: Must include @key directive

```graphql
type User @extends @key(fields: "id") {
  id: ID! @external
  posts: [Post!]!
}
```

### @external
- **Purpose**: Marks fields that are defined in other subgraphs
- **Usage**: References fields from extended entities
- **Resolution**: Fields are resolved by the originating subgraph

```graphql
type User @extends @key(fields: "id") {
  id: ID! @external
  email: String! @external
  profile: Profile
}
```

### @provides
- **Purpose**: Indicates fields provided by this subgraph
- **Usage**: Optimizes query planning
- **Context**: Used with entity extensions

```graphql
type User @extends @key(fields: "id") {
  id: ID! @external
  posts: [Post!]! @provides(fields: "author")
}
```

### @requires
- **Purpose**: Indicates fields needed from other subgraphs
- **Usage**: Access fields from extended entities
- **Dependencies**: Creates subgraph dependencies

```graphql
type Review @key(fields: "id") {
  id: ID!
  product: Product! @provides(fields: "reviews")
  rating: Int! @requires(fields: "product { category }")
}
```

## Subgraph Architecture

### Service Definition
```javascript
// users subgraph
const { ApolloServer } = require('apollo-server');
const { buildSubgraphSchema } = require('@apollo/subgraph');

const typeDefs = gql`
  type User @key(fields: "id") {
    id: ID!
    name: String!
    email: String!
  }
`;

const resolvers = {
  User: {
    __resolveReference(reference) {
      return getUserById(reference.id);
    }
  }
};

const server = new ApolloServer({
  schema: buildSubgraphSchema({ typeDefs, resolvers })
});
```

### Entity Resolution
```javascript
const resolvers = {
  User: {
    __resolveReference(reference, context, info) {
      // Resolve entity by key
      if (reference.id) {
        return getUserById(reference.id);
      }
      if (reference.email) {
        return getUserByEmail(reference.email);
      }
      
      throw new Error('Unable to resolve User reference');
    },
    
    // Regular field resolvers
    posts: (parent, args, context, info) => {
      return getPostsByUserId(parent.id);
    }
  }
};
```

### Cross-Subgraph References
```javascript
// posts subgraph
const typeDefs = gql`
  type Post @key(fields: "id") {
    id: ID!
    title: String!
    content: String!
    authorId: ID!
    author: User!
  }
  
  extend type User @key(fields: "id") {
    id: ID! @external
    posts: [Post!]!
  }
`;

const resolvers = {
  Post: {
    author: (parent, args, context, info) => {
      // Reference resolution handled by gateway
      return { __typename: 'User', id: parent.authorId };
    }
  },
  
  User: {
    posts: (parent, args, context, info) => {
      return getPostsByUserId(parent.id);
    }
  }
};
```

## Gateway Configuration

### Apollo Gateway Setup
```javascript
const { ApolloGateway, RemoteGraphQLDataSource } = require('@apollo/gateway');

const gateway = new ApolloGateway({
  serviceList: [
    { name: 'users', url: 'http://localhost:4001' },
    { name: 'posts', url: 'http://localhost:4002' },
    { name: 'reviews', url: 'http://localhost:4003' }
  ],
  
  buildService({ name, url }) {
    return new RemoteGraphQLDataSource({
      url,
      willSendRequest({ request, context }) {
        // Forward authentication headers
        request.http.headers.set('authorization', context.authToken);
      }
    });
  }
});

const server = new ApolloServer({
  gateway,
  subscriptions: false
});
```

### Schema Composition
```javascript
// Automatic composition
const gateway = new ApolloGateway({
  serviceList: serviceList,
  // Composition happens automatically
});

// Manual composition (for advanced cases)
const { composeServices } = require('@apollo/composition');

const composedSchema = composeServices([
  { name: 'users', typeDefs: usersSchema },
  { name: 'posts', typeDefs: postsSchema }
]);
```

## Advanced Patterns

### Value Types
- **Definition**: Types that don't need @key directive
- **Usage**: Simple data structures shared across subgraphs
- **Composition**: Included in all subgraphs that use them

```graphql
# Shared value type
type Address {
  street: String!
  city: String!
  country: String!
}

type User @key(fields: "id") {
  id: ID!
  name: String!
  address: Address!
}
```

### Interface Extensions
```graphql
# Base interface
interface Node {
  id: ID!
}

# Extending interface
type User implements Node @key(fields: "id") {
  id: ID!
  name: String!
}

type Post implements Node @key(fields: "id") {
  id: ID!
  title: String!
}
```

### Federation Versioning
```graphql
# Versioned entities
type User_v1 @key(fields: "id") {
  id: ID!
  name: String!
}

type User_v2 @key(fields: "id") {
  id: ID!
  name: String!
  email: String!
}
```

## Query Planning

### Query Decomposition
- **Analysis**: Gateway analyzes incoming query
- **Decomposition**: Breaks query into subgraph operations
- **Execution**: Runs operations in parallel
- **Composition**: Combines results into final response

### Execution Flow
```
Client Query
    ↓
Gateway Analysis
    ↓
Query Planning
    ↓
Parallel Execution
    ↓
Result Composition
    ↓
Client Response
```

### Optimization
- **Batching**: Combine multiple requests to same subgraph
- **Caching**: Cache entity references
- **Prefetching**: Preload commonly accessed entities

## Error Handling

### Subgraph Errors
```javascript
const resolvers = {
  User: {
    posts: async (parent, args, context, info) => {
      try {
        return await getPostsByUserId(parent.id);
      } catch (error) {
        // Log error but don't fail entire query
        console.error('Failed to load posts:', error);
        return []; // Return empty array
      }
    }
  }
};
```

### Partial Results
- **Graceful Degradation**: Return partial results when possible
- **Error Propagation**: Include subgraph errors in response
- **Fallback Values**: Provide default values for failed fields

## Testing Federation

### Subgraph Testing
```javascript
const { createTestClient } = require('apollo-server-testing');

describe('Users Subgraph', () => {
  const { query } = createTestClient(usersServer);
  
  it('should resolve user reference', async () => {
    const result = await query({
      query: `
        query($representations: [_Any!]!) {
          _entities(representations: $representations) {
            ...on User {
              id
              name
            }
          }
        }
      `,
      variables: {
        representations: [{ __typename: 'User', id: '1' }]
      }
    });
    
    expect(result.data._entities[0].name).toBe('John Doe');
  });
});
```

### Federation Testing
```javascript
const { ApolloServer } = require('apollo-server');
const { ApolloGateway } = require('@apollo/gateway');

describe('Federated Schema', () => {
  let gateway;
  
  beforeAll(async () => {
    gateway = new ApolloGateway({
      serviceList: [
        { name: 'users', url: 'http://localhost:4001' },
        { name: 'posts', url: 'http://localhost:4002' }
      ]
    });
    
    await gateway.load();
  });
  
  it('should compose schemas successfully', () => {
    expect(gateway.schema).toBeDefined();
  });
});
```

## Performance Considerations

### N+1 Query Problem
- **Cause**: Multiple round trips for entity resolution
- **Solution**: Use DataLoader in subgraphs
- **Optimization**: Batch entity resolution

### Network Latency
- **Impact**: Cross-subgraph calls add latency
- **Mitigation**: Colocate related subgraphs
- **Caching**: Cache entity references

### Schema Complexity
- **Composition Time**: Large schemas take longer to compose
- **Query Planning**: Complex queries require more planning
- **Optimization**: Keep subgraph schemas focused

## Deployment Strategies

### Independent Deployment
- **Subgraphs**: Deploy independently
- **Gateway**: Update service list
- **Compatibility**: Maintain backward compatibility

### Blue-Green Deployment
- **Subgraph Updates**: Deploy new version alongside old
- **Traffic Shifting**: Gradually move traffic
- **Rollback**: Quick reversion capability

### Schema Registry
- **Storage**: Store composed schemas
- **Validation**: Validate changes against registry
- **Approval**: Require approval for breaking changes

## Tooling

### Apollo Studio
- **Schema Registry**: Store and version schemas
- **Composition**: Automatic schema composition
- **Analytics**: Query performance and usage
- **Linting**: Schema validation and best practices

### Rover CLI
```bash
# Publish subgraph schema
rover subgraph publish my-graph@current \
  --schema ./subgraph-schema.graphql \
  --name users \
  --routing-url http://users-service.com

# Check composition
rover subgraph check my-graph@current \
  --schema ./subgraph-schema.graphql \
  --name users
```

### GraphQL Mesh
- **Alternative**: Schema stitching approach
- **Sources**: Multiple data sources (REST, GraphQL, etc.)
- **Transformation**: Transform and merge schemas

## Best Practices

### Subgraph Design
- **Domain Boundaries**: Align subgraphs with domain boundaries
- **Team Ownership**: One team per subgraph
- **Size**: Keep subgraphs manageable
- **Independence**: Minimize cross-subgraph dependencies

### Entity Design
- **Keys**: Choose stable, unique keys
- **Extensions**: Use @extends for cross-cutting concerns
- **Ownership**: Clear ownership of entities

### Performance
- **DataLoader**: Use in all subgraphs
- **Caching**: Implement appropriate caching
- **Monitoring**: Monitor subgraph performance

### Governance
- **Review Process**: Review schema changes
- **Breaking Changes**: Plan for breaking changes
- **Documentation**: Document subgraph responsibilities

## Related
- [[api/graphql/schema-design/index\|Schema Design]]
- [[api/graphql/resolvers/index\|Resolvers]]
- [[api/graphql/subscriptions/index\|Subscriptions]]
- [[api/microservices/index\|Microservices]]
