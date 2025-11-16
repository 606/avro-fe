---
{"publish":true,"aliases":"Graphql-Subscriptions Realtime-Graphql Websocket-Graphql","title":"Graphql Subscriptions","enableToc":true,"created":"2025-11-16T00:29:31.334+02:00","modified":"2025-11-16T15:31:33.383+02:00","tags":["api","graphql","subscriptions","realtime","websocket"],"cssclasses":""}
---


## Overview

GraphQL subscriptions enable real-time communication between clients and servers. Unlike queries and mutations, subscriptions maintain a persistent connection and push updates to clients as data changes.

## Subscription Basics

### Subscription Definition
```graphql
type Subscription {
  userCreated: User!
  postUpdated(id: ID!): Post!
  messageReceived(chatId: ID!): Message!
}
```

### Client Usage
```javascript
const SUBSCRIBE_TO_USER_UPDATES = gql`
  subscription OnUserCreated {
    userCreated {
      id
      name
      email
    }
  }
`;

const { data, loading, error } = useSubscription(SUBSCRIBE_TO_USER_UPDATES, {
  onSubscriptionData: ({ subscriptionData }) => {
    console.log('New user created:', subscriptionData.data.userCreated);
  }
});
```

## Transport Protocols

### WebSocket
- **Standard**: Most common transport for subscriptions
- **Connection**: Persistent full-duplex connection
- **Protocol**: graphql-ws or subscriptions-transport-ws

### Server-Sent Events (SSE)
- **HTTP-based**: Uses regular HTTP connections
- **Unidirectional**: Server to client only
- **Fallback**: Good for environments without WebSocket support

### HTTP Polling
- **Fallback**: Regular HTTP requests at intervals
- **Compatibility**: Works everywhere
- **Efficiency**: Less efficient than WebSocket

## Apollo Server Implementation

### Basic Setup
```javascript
const { ApolloServer, gql, PubSub } = require('apollo-server');

const pubsub = new PubSub();

const typeDefs = gql`
  type Subscription {
    postAdded: Post!
  }
  
  type Mutation {
    addPost(title: String!, content: String!): Post!
  }
`;

const resolvers = {
  Subscription: {
    postAdded: {
      subscribe: () => pubsub.asyncIterator(['POST_ADDED'])
    }
  },
  
  Mutation: {
    addPost: async (_, { title, content }) => {
      const post = { id: Date.now(), title, content };
      
      // Publish event
      pubsub.publish('POST_ADDED', { postAdded: post });
      
      return post;
    }
  }
};

const server = new ApolloServer({
  typeDefs,
  resolvers,
  context: ({ req, res }) => ({ req, res, pubsub })
});
```

### PubSub Implementation
```javascript
// In-memory PubSub (development)
const pubsub = new PubSub();

// Redis PubSub (production)
const { RedisPubSub } = require('graphql-redis-subscriptions');
const pubsub = new RedisPubSub({
  connection: {
    host: process.env.REDIS_HOST,
    port: process.env.REDIS_PORT,
    password: process.env.REDIS_PASSWORD
  }
});

// Custom PubSub
class CustomPubSub {
  constructor() {
    this.subscriptions = new Map();
  }
  
  async publish(triggerName, payload) {
    const subscribers = this.subscriptions.get(triggerName) || [];
    await Promise.all(
      subscribers.map(async ({ resolve, reject }) => {
        try {
          resolve(payload);
        } catch (error) {
          reject(error);
        }
      })
    );
  }
  
  async subscribe(triggerName) {
    return new Promise((resolve, reject) => {
      if (!this.subscriptions.has(triggerName)) {
        this.subscriptions.set(triggerName, []);
      }
      
      this.subscriptions.get(triggerName).push({ resolve, reject });
    });
  }
}
```

## Subscription Resolvers

### Basic Subscription Resolver
```javascript
const resolvers = {
  Subscription: {
    userCreated: {
      subscribe: (_, args, context, info) => {
        return context.pubsub.asyncIterator('USER_CREATED');
      }
    }
  }
};
```

### Filtered Subscriptions
```javascript
const resolvers = {
  Subscription: {
    postUpdated: {
      subscribe: (_, { id }, context, info) => {
        return context.pubsub.asyncIterator(`POST_UPDATED_${id}`);
      }
    },
    
    // With filtering
    userPostsUpdated: {
      subscribe: withFilter(
        (_, args, context, info) => {
          return context.pubsub.asyncIterator('POST_UPDATED');
        },
        (payload, variables, context, info) => {
          // Filter based on user ownership
          return payload.postUpdated.authorId === context.user.id;
        }
      )
    }
  }
};
```

### Subscription with Arguments
```javascript
const typeDefs = gql`
  type Subscription {
    chatMessages(chatId: ID!): Message!
    userActivity(userId: ID!): Activity!
  }
`;

const resolvers = {
  Subscription: {
    chatMessages: {
      subscribe: (_, { chatId }, context, info) => {
        // Validate access to chat
        if (!canAccessChat(context.user.id, chatId)) {
          throw new ForbiddenError('Access denied');
        }
        
        return context.pubsub.asyncIterator(`CHAT_MESSAGES_${chatId}`);
      }
    }
  }
};
```

## Client-Side Implementation

### Apollo Client Setup
```javascript
import { ApolloClient, InMemoryCache, split, HttpLink } from '@apollo/client';
import { WebSocketLink } from '@apollo/client/link/ws';
import { getMainDefinition } from '@apollo/client/utilities';

const httpLink = new HttpLink({
  uri: 'http://localhost:4000/graphql'
});

const wsLink = new WebSocketLink({
  uri: 'ws://localhost:4000/graphql',
  options: {
    reconnect: true,
    connectionParams: {
      authToken: localStorage.getItem('authToken')
    }
  }
});

const splitLink = split(
  ({ query }) => {
    const definition = getMainDefinition(query);
    return (
      definition.kind === 'OperationDefinition' &&
      definition.operation === 'subscription'
    );
  },
  wsLink,
  httpLink
);

const client = new ApolloClient({
  link: splitLink,
  cache: new InMemoryCache()
});
```

### React Hook Usage
```javascript
import { useSubscription } from '@apollo/client';

function UserList() {
  const { data, loading, error } = useSubscription(
    gql`
      subscription OnUserCreated {
        userCreated {
          id
          name
          email
        }
      }
    `,
    {
      onSubscriptionData: ({ subscriptionData }) => {
        // Handle real-time updates
        console.log('New user:', subscriptionData.data.userCreated);
      },
      onError: (error) => {
        console.error('Subscription error:', error);
      }
    }
  );

  if (loading) return <p>Connecting...</p>;
  if (error) return <p>Error: {error.message}</p>;

  return (
    <div>
      <h2>Real-time User Updates</h2>
      {data?.userCreated && (
        <div>New user: {data.userCreated.name}</div>
      )}
    </div>
  );
}
```

### Subscription Management
```javascript
function ChatRoom({ chatId }) {
  const [messages, setMessages] = useState([]);
  
  const { data, loading } = useSubscription(CHAT_MESSAGES, {
    variables: { chatId },
    onSubscriptionData: ({ subscriptionData }) => {
      const newMessage = subscriptionData.data.messageReceived;
      setMessages(prev => [...prev, newMessage]);
    }
  });
  
  // Cleanup handled automatically by Apollo Client
  return (
    <div>
      {messages.map(message => (
        <div key={message.id}>{message.content}</div>
      ))}
    </div>
  );
}
```

## Authentication and Authorization

### Context with Authentication
```javascript
const server = new ApolloServer({
  typeDefs,
  resolvers,
  context: async ({ req, connection }) => {
    // Handle both HTTP and WebSocket connections
    if (connection) {
      // WebSocket connection
      return {
        user: connection.context.user,
        pubsub
      };
    } else {
      // HTTP connection
      const token = req.headers.authorization;
      const user = await getUserFromToken(token);
      
      return { user, pubsub };
    }
  },
  
  subscriptions: {
    onConnect: async (connectionParams, webSocket, context) => {
      const token = connectionParams.authToken;
      const user = await getUserFromToken(token);
      
      return { user };
    },
    
    onDisconnect: async (webSocket, context) => {
      // Cleanup resources
      console.log('Client disconnected');
    }
  }
});
```

### Subscription Authorization
```javascript
const resolvers = {
  Subscription: {
    privateMessages: {
      subscribe: (_, args, { user, pubsub }) => {
        if (!user) {
          throw new AuthenticationError('Not authenticated');
        }
        
        return pubsub.asyncIterator(`PRIVATE_MESSAGES_${user.id}`);
      }
    },
    
    teamUpdates: {
      subscribe: (_, { teamId }, { user, pubsub }) => {
        if (!canAccessTeam(user.id, teamId)) {
          throw new ForbiddenError('Access denied');
        }
        
        return pubsub.asyncIterator(`TEAM_UPDATES_${teamId}`);
      }
    }
  }
};
```

## Error Handling

### Subscription Errors
```javascript
const resolvers = {
  Subscription: {
    dataUpdates: {
      subscribe: (_, args, context, info) => {
        try {
          return context.pubsub.asyncIterator('DATA_UPDATES');
        } catch (error) {
          console.error('Subscription setup failed:', error);
          throw new Error('Failed to establish subscription');
        }
      }
    }
  }
};
```

### Connection Errors
```javascript
const wsLink = new WebSocketLink({
  uri: 'ws://localhost:4000/graphql',
  options: {
    reconnect: true,
    reconnectionAttempts: 5,
    connectionParams: {
      authToken: localStorage.getItem('authToken')
    },
    onError: (error) => {
      console.error('WebSocket connection error:', error);
    },
    onReconnect: () => {
      console.log('Reconnected to WebSocket');
    }
  }
});
```

### Graceful Degradation
```javascript
function useSubscriptionWithFallback(query, options) {
  const subscription = useSubscription(query, options);
  
  // Fallback to polling if subscription fails
  const [pollingData, setPollingData] = useState(null);
  
  useEffect(() => {
    if (subscription.error) {
      const interval = setInterval(async () => {
        try {
          const result = await client.query({ query });
          setPollingData(result.data);
        } catch (error) {
          console.error('Polling failed:', error);
        }
      }, 5000);
      
      return () => clearInterval(interval);
    }
  }, [subscription.error]);
  
  return subscription.error ? 
    { ...subscription, data: pollingData } : 
    subscription;
}
```

## Performance Optimization

### Connection Pooling
```javascript
// Server-side connection limits
const server = new ApolloServer({
  subscriptions: {
    path: '/graphql',
    keepAlive: 30000, // 30 seconds
    onConnect: (connectionParams) => {
      // Limit concurrent connections per user
      return checkConnectionLimit(connectionParams.userId);
    }
  }
});
```

### Payload Optimization
```javascript
const resolvers = {
  Subscription: {
    userUpdated: {
      subscribe: withFilter(
        (_, args, context) => {
          return context.pubsub.asyncIterator('USER_UPDATED');
        },
        (payload, variables, context) => {
          // Only send updates relevant to the subscriber
          return payload.userUpdated.id === variables.userId;
        }
      )
    }
  }
};
```

### Batching Updates
```javascript
class BatchedPubSub {
  constructor(pubsub, batchDelay = 100) {
    this.pubsub = pubsub;
    this.batchDelay = batchDelay;
    this.batches = new Map();
  }
  
  async publish(triggerName, payload) {
    if (!this.batches.has(triggerName)) {
      this.batches.set(triggerName, []);
      
      setTimeout(() => {
        const batch = this.batches.get(triggerName);
        this.batches.delete(triggerName);
        
        // Publish batched updates
        this.pubsub.publish(triggerName, { updates: batch });
      }, this.batchDelay);
    }
    
    this.batches.get(triggerName).push(payload);
  }
}
```

## Testing Subscriptions

### Unit Testing
```javascript
const { createTestClient } = require('apollo-server-testing');

describe('Subscriptions', () => {
  let pubsub;
  
  beforeEach(() => {
    pubsub = new PubSub();
  });
  
  it('should publish user created events', async () => {
    const { subscribe } = createTestClient(server);
    
    const subscription = subscribe({
      query: `
        subscription {
          userCreated {
            id
            name
          }
        }
      `
    });
    
    // Trigger mutation that publishes event
    await createUser({ name: 'John Doe' });
    
    const result = await subscription.next();
    expect(result.value.data.userCreated.name).toBe('John Doe');
  });
});
```

### Integration Testing
```javascript
const WebSocket = require('ws');
const { SubscriptionClient } = require('subscriptions-transport-ws');

describe('WebSocket Subscriptions', () => {
  let client;
  
  beforeAll(() => {
    client = new SubscriptionClient(
      'ws://localhost:4000/graphql',
      {},
      WebSocket
    );
  });
  
  afterAll(() => {
    client.close();
  });
  
  it('should receive real-time updates', (done) => {
    client.request({
      query: `
        subscription {
          postAdded {
            id
            title
          }
        }
      `
    }).subscribe({
      next: (result) => {
        expect(result.data.postAdded.title).toBeDefined();
        done();
      }
    });
    
    // Trigger the event
    createPost({ title: 'Test Post' });
  });
});
```

## Production Considerations

### Scaling
- **Load Balancing**: Distribute WebSocket connections
- **Redis**: Use Redis for cross-server pub/sub
- **Clustering**: Handle subscriptions across multiple server instances

### Monitoring
- **Connection Count**: Monitor active connections
- **Message Rate**: Track subscription message frequency
- **Error Rates**: Monitor subscription failures

### Security
- **Rate Limiting**: Limit subscription frequency
- **Connection Limits**: Restrict connections per user/IP
- **Timeout**: Set connection timeouts

## Common Patterns

### Real-time Chat
```graphql
type Subscription {
  messageReceived(chatId: ID!): Message!
  userJoined(chatId: ID!): User!
  userLeft(chatId: ID!): User!
}
```

### Live Data Updates
```graphql
type Subscription {
  stockPriceUpdated(symbol: String!): StockPrice!
  weatherUpdated(location: String!): Weather!
  sensorDataUpdated(sensorId: ID!): SensorData!
}
```

### Collaborative Editing
```graphql
type Subscription {
  documentUpdated(documentId: ID!): Document!
  userCursorMoved(documentId: ID!, userId: ID!): CursorPosition!
  userSelectionChanged(documentId: ID!, userId: ID!): Selection!
}
```

## Related
- [[api/graphql/schema-design/index\|Schema Design]]
- [[api/graphql/resolvers/index\|Resolvers]]
- [[api/graphql/federation/index\|Federation]]
- [[../../realtime-apis/index\|Real-time APIs]]
