---
{"publish":true,"aliases":"graphql-resolvers resolvers field-resolution","title":"GraphQL Resolvers","enableToc":true,"created":"2025-11-16T00:29:31.320+02:00","modified":"2025-11-16T00:32:07.686+02:00","tags":["api","graphql","resolvers","backend"],"cssclasses":""}
---


## Overview

Resolvers are the functions that populate the data for each field in a GraphQL schema. They connect the GraphQL schema to your data sources and business logic.

## Resolver Function Signature

### Basic Resolver
```javascript
const resolvers = {
  Query: {
    user: (parent, args, context, info) => {
      // Resolver implementation
      return getUserById(args.id);
    }
  }
};
```

### Resolver Parameters

#### parent (root)
- **Description**: Result of the parent field's resolver
- **Type**: Any (result of parent resolver)
- **Use Case**: Accessing parent data in nested resolvers

```javascript
const resolvers = {
  User: {
    posts: (parent, args, context, info) => {
      // parent is the User object
      return getPostsByUserId(parent.id);
    }
  }
};
```

#### args
- **Description**: Arguments passed to the field
- **Type**: Object with argument values
- **Use Case**: Filtering, pagination, search parameters

```javascript
const resolvers = {
  Query: {
    users: (parent, args, context, info) => {
      const { limit = 10, offset = 0, status } = args;
      return getUsers({ limit, offset, status });
    }
  }
};
```

#### context
- **Description**: Shared context across all resolvers in a query
- **Type**: Object (mutable across resolvers)
- **Use Case**: Database connections, authentication, caching

```javascript
const resolvers = {
  Query: {
    user: (parent, args, context, info) => {
      // Check authentication
      if (!context.user) {
        throw new AuthenticationError('Not authenticated');
      }
      
      // Use database from context
      return context.db.users.findById(args.id);
    }
  }
};
```

#### info
- **Description**: AST representation of the query
- **Type**: GraphQLResolveInfo
- **Use Case**: Advanced introspection, query analysis

```javascript
const resolvers = {
  Query: {
    users: (parent, args, context, info) => {
      // Get selected fields
      const selectedFields = info.fieldNodes[0].selectionSet.selections
        .map(selection => selection.name.value);
      
      return getUsersWithFields(args, selectedFields);
    }
  }
};
```

## Resolver Types

### Scalar Resolvers
```javascript
const resolvers = {
  DateTime: {
    // Custom scalar resolver
    parseValue(value) {
      return new Date(value);
    },
    serialize(value) {
      return value.toISOString();
    },
    parseLiteral(ast) {
      if (ast.kind === Kind.STRING) {
        return new Date(ast.value);
      }
      return null;
    }
  }
};
```

### Object Type Resolvers
```javascript
const resolvers = {
  User: {
    fullName: (parent, args, context, info) => {
      return `${parent.firstName} ${parent.lastName}`;
    },
    
    posts: async (parent, args, context, info) => {
      return context.dataLoaders.posts.load(parent.id);
    },
    
    friends: (parent, args, context, info) => {
      return getFriends(parent.id, args.limit);
    }
  }
};
```

### Interface Resolvers
```javascript
const resolvers = {
  Node: {
    __resolveType(obj, context, info) {
      if (obj.__typename) {
        return obj.__typename;
      }
      
      // Type resolution logic
      if (obj.email) {
        return 'User';
      }
      if (obj.title) {
        return 'Post';
      }
      
      return null; // GraphQL error
    }
  }
};
```

### Union Resolvers
```javascript
const resolvers = {
  SearchResult: {
    __resolveType(obj, context, info) {
      if (obj.__typename) {
        return obj.__typename;
      }
      
      // Union type resolution
      if (obj.userId) {
        return 'User';
      }
      if (obj.postId) {
        return 'Post';
      }
      
      return null;
    }
  }
};
```

## Data Loading Patterns

### Direct Database Access
```javascript
const resolvers = {
  Query: {
    user: async (parent, args, context, info) => {
      const user = await context.db.users.findById(args.id);
      return user;
    }
  },
  
  User: {
    posts: async (parent, args, context, info) => {
      const posts = await context.db.posts.findByAuthor(parent.id);
      return posts;
    }
  }
};
```

### DataLoader (Batch Loading)
```javascript
import DataLoader from 'dataloader';

const createDataLoaders = (db) => ({
  users: new DataLoader(async (ids) => {
    const users = await db.users.findByIds(ids);
    return ids.map(id => users.find(user => user.id === id));
  }),
  
  posts: new DataLoader(async (userIds) => {
    const posts = await db.posts.findByAuthorIds(userIds);
    return userIds.map(userId => 
      posts.filter(post => post.authorId === userId)
    );
  })
});

const resolvers = {
  User: {
    posts: (parent, args, context, info) => {
      return context.dataLoaders.posts.load(parent.id);
    }
  }
};
```

### Cached Resolvers
```javascript
const resolvers = {
  Query: {
    user: async (parent, args, context, info) => {
      const cacheKey = `user:${args.id}`;
      let user = await context.cache.get(cacheKey);
      
      if (!user) {
        user = await context.db.users.findById(args.id);
        await context.cache.set(cacheKey, user, { ttl: 300 });
      }
      
      return user;
    }
  }
};
```

## Error Handling

### Resolver Errors
```javascript
const resolvers = {
  Query: {
    user: async (parent, args, context, info) => {
      try {
        const user = await context.db.users.findById(args.id);
        
        if (!user) {
          throw new UserNotFoundError(`User ${args.id} not found`);
        }
        
        return user;
      } catch (error) {
        if (error instanceof UserNotFoundError) {
          throw new GraphQLError(error.message, {
            extensions: {
              code: 'USER_NOT_FOUND',
              argumentName: 'id'
            }
          });
        }
        
        throw error; // Re-throw other errors
      }
    }
  }
};
```

### Validation Errors
```javascript
import { UserInputError } from 'apollo-server';

const resolvers = {
  Mutation: {
    createUser: async (parent, args, context, info) => {
      const { input } = args;
      
      // Validation
      if (input.password.length < 8) {
        throw new UserInputError('Password too short', {
          argumentName: 'input.password',
          validationErrors: ['Password must be at least 8 characters']
        });
      }
      
      return context.db.users.create(input);
    }
  }
};
```

### Authentication Errors
```javascript
import { AuthenticationError, ForbiddenError } from 'apollo-server';

const resolvers = {
  Query: {
    adminUsers: (parent, args, context, info) => {
      if (!context.user) {
        throw new AuthenticationError('Not authenticated');
      }
      
      if (context.user.role !== 'ADMIN') {
        throw new ForbiddenError('Insufficient permissions');
      }
      
      return context.db.users.findAdmins();
    }
  }
};
```

## Performance Optimization

### Query Complexity Analysis
```javascript
const resolvers = {
  Query: {
    users: (parent, args, context, info) => {
      // Calculate complexity
      const complexity = calculateQueryComplexity(info);
      
      if (complexity > context.maxComplexity) {
        throw new GraphQLError('Query too complex');
      }
      
      return getUsers(args);
    }
  }
};
```

### Field-Level Caching
```javascript
const resolvers = {
  User: {
    avatarUrl: async (parent, args, context, info) => {
      const cacheKey = `user:${parent.id}:avatar`;
      let avatarUrl = await context.cache.get(cacheKey);
      
      if (!avatarUrl) {
        avatarUrl = await generateAvatarUrl(parent);
        await context.cache.set(cacheKey, avatarUrl, { ttl: 3600 });
      }
      
      return avatarUrl;
    }
  }
};
```

### Selective Field Resolution
```javascript
const resolvers = {
  User: {
    profile: (parent, args, context, info) => {
      // Only fetch profile if requested
      const requestedFields = info.fieldNodes[0].selectionSet.selections
        .map(selection => selection.name.value);
      
      if (requestedFields.includes('bio') || requestedFields.includes('website')) {
        return context.db.profiles.findByUserId(parent.id);
      }
      
      return null;
    }
  }
};
```

## Testing Resolvers

### Unit Testing
```javascript
const { makeExecutableSchema } = require('@graphql-tools/schema');
const { graphql } = require('graphql');

describe('User Resolvers', () => {
  const mockContext = {
    db: {
      users: {
        findById: jest.fn()
      }
    }
  };
  
  it('should resolve user', async () => {
    mockContext.db.users.findById.mockResolvedValue({
      id: '1',
      name: 'John Doe'
    });
    
    const result = await graphql({
      schema,
      source: '{ user(id: "1") { id name } }',
      contextValue: mockContext
    });
    
    expect(result.data.user.name).toBe('John Doe');
  });
});
```

### Integration Testing
```javascript
const { createTestClient } = require('apollo-server-testing');

describe('User API', () => {
  const { query, mutate } = createTestClient(server);
  
  it('should get user', async () => {
    const GET_USER = `
      query GetUser($id: ID!) {
        user(id: $id) {
          id
          name
        }
      }
    `;
    
    const res = await query({
      query: GET_USER,
      variables: { id: '1' }
    });
    
    expect(res.data.user.name).toBe('John Doe');
  });
});
```

## Resolver Composition

### Resolver Middleware
```javascript
const authMiddleware = (resolver) => {
  return (parent, args, context, info) => {
    if (!context.user) {
      throw new AuthenticationError('Not authenticated');
    }
    
    return resolver(parent, args, context, info);
  };
};

const resolvers = {
  Query: {
    users: authMiddleware((parent, args, context, info) => {
      return context.db.users.findAll();
    })
  }
};
```

### Resolver Factories
```javascript
const createCRUDResolvers = (model) => ({
  Query: {
    [`${model.name.toLowerCase()}`]: (parent, args, context, info) => {
      return context.db[model.name].findById(args.id);
    },
    
    [`${model.name.toLowerCase()}s`]: (parent, args, context, info) => {
      return context.db[model.name].findAll(args);
    }
  },
  
  Mutation: {
    [`create${model.name}`]: (parent, args, context, info) => {
      return context.db[model.name].create(args.input);
    },
    
    [`update${model.name}`]: (parent, args, context, info) => {
      return context.db[model.name].update(args.id, args.input);
    },
    
    [`delete${model.name}`]: (parent, args, context, info) => {
      return context.db[model.name].delete(args.id);
    }
  }
});
```

## Best Practices

### Error Handling
- **Consistent Errors**: Use standard error formats
- **Appropriate HTTP Codes**: Map GraphQL errors to HTTP codes
- **Detailed Messages**: Provide helpful error information

### Performance
- **Batch Loading**: Use DataLoader for N+1 problems
- **Caching**: Implement appropriate caching strategies
- **Limits**: Set query complexity and depth limits

### Security
- **Authentication**: Check user permissions
- **Authorization**: Validate access to resources
- **Input Validation**: Sanitize and validate inputs

### Maintainability
- **Separation of Concerns**: Keep resolvers focused
- **Testing**: Write comprehensive tests
- **Documentation**: Document resolver behavior

## Related
- [[api/graphql/schema-design/index\|Schema Design]]
- [[api/graphql/federation/index\|Schema Federation]]
- [[api/graphql/subscriptions/index\|Subscriptions]]
- [[../api-testing/index\|API Testing]]
