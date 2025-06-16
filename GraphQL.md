---
tags:
  - backend
  - graphql
  - apollo
  - schema
  - api
created: 2025-06-15
updated: 2025-06-15
---

# 🔗 GraphQL

> **Summary**: Complete guide to GraphQL including schema design, Apollo Server setup, client queries, and best practices for building flexible, efficient APIs.

## 📋 Table of Contents

- [GraphQL Fundamentals](#graphql-fundamentals)
- [Schema Design](#schema-design)
- [Apollo Server Setup](#apollo-server-setup)
- [Resolvers & Data Sources](#resolvers--data-sources)
- [Client Queries](#client-queries)
- [Apollo Client Setup](#apollo-client-setup)
- [Advanced Patterns](#advanced-patterns)
- [Performance & Optimization](#performance--optimization)

## GraphQL Fundamentals

### Core Concepts
```graphql
# Schema Definition Language (SDL)
type User {
  id: ID!
  name: String!
  email: String!
  posts: [Post!]!
  createdAt: DateTime!
}

type Post {
  id: ID!
  title: String!
  content: String!
  author: User!
  published: Boolean!
  tags: [String!]!
}

# Root Types
type Query {
  users: [User!]!
  user(id: ID!): User
  posts(published: Boolean): [Post!]!
}

type Mutation {
  createUser(input: CreateUserInput!): User!
  updateUser(id: ID!, input: UpdateUserInput!): User!
  deleteUser(id: ID!): Boolean!
}

type Subscription {
  userCreated: User!
  postPublished: Post!
}
```

### GraphQL vs REST
| Feature | GraphQL | REST |
|---------|---------|------|
| Data Fetching | Single request, exact data | Multiple requests, over-fetching |
| Versioning | Schema evolution | URL versioning |
| Caching | Complex | Simple HTTP caching |
| Learning Curve | Steeper | Gentler |
| Tooling | Rich ecosystem | Mature ecosystem |

## Schema Design

### Scalar Types
```graphql
# Built-in Scalars
scalar ID       # Unique identifier
scalar String   # UTF-8 character sequence
scalar Int      # 32-bit integer
scalar Float    # Double-precision floating-point
scalar Boolean  # true or false

# Custom Scalars
scalar DateTime
scalar Email
scalar URL
scalar JSON

# Usage
type User {
  id: ID!
  email: Email!
  website: URL
  metadata: JSON
  createdAt: DateTime!
}
```

### Object Types & Relationships
```graphql
type User {
  id: ID!
  name: String!
  email: String!
  
  # One-to-many relationship
  posts: [Post!]!
  
  # Many-to-many relationship
  followedUsers: [User!]!
  followers: [User!]!
  
  # Computed fields
  fullName: String!
  postCount: Int!
}

type Post {
  id: ID!
  title: String!
  content: String!
  
  # Many-to-one relationship
  author: User!
  
  # Many-to-many relationship
  categories: [Category!]!
  
  # Optional fields
  publishedAt: DateTime
  featuredImage: String
}

type Category {
  id: ID!
  name: String!
  posts: [Post!]!
}
```

### Input Types & Enums
```graphql
# Input types for mutations
input CreateUserInput {
  name: String!
  email: String!
  password: String!
  role: UserRole = USER
}

input UpdateUserInput {
  name: String
  email: String
  bio: String
}

input PostFilters {
  published: Boolean
  authorId: ID
  categoryIds: [ID!]
  dateRange: DateRangeInput
}

input DateRangeInput {
  from: DateTime!
  to: DateTime!
}

# Enums for predefined values
enum UserRole {
  ADMIN
  MODERATOR
  USER
  GUEST
}

enum PostStatus {
  DRAFT
  PUBLISHED
  ARCHIVED
}

enum SortOrder {
  ASC
  DESC
}
```

### Interfaces & Unions
```graphql
# Interface for common fields
interface Node {
  id: ID!
  createdAt: DateTime!
  updatedAt: DateTime!
}

# Types implementing interface
type User implements Node {
  id: ID!
  createdAt: DateTime!
  updatedAt: DateTime!
  name: String!
  email: String!
}

type Post implements Node {
  id: ID!
  createdAt: DateTime!
  updatedAt: DateTime!
  title: String!
  content: String!
}

# Union for search results
union SearchResult = User | Post | Category

type Query {
  search(query: String!): [SearchResult!]!
}
```

## Apollo Server Setup

### Basic Server Setup
```javascript
// server.js
const { ApolloServer } = require('@apollo/server');
const { startStandaloneServer } = require('@apollo/server/standalone');
const { readFileSync } = require('fs');
const { resolvers } = require('./resolvers');

// Load schema from file
const typeDefs = readFileSync('./schema.graphql', { encoding: 'utf-8' });

// Create Apollo Server
const server = new ApolloServer({
  typeDefs,
  resolvers,
  // Enable GraphQL Playground in development
  introspection: process.env.NODE_ENV !== 'production',
  playground: process.env.NODE_ENV !== 'production',
});

// Start server
const startServer = async () => {
  const { url } = await startStandaloneServer(server, {
    listen: { port: 4000 },
    context: async ({ req }) => {
      // Add authentication context
      const token = req.headers.authorization?.replace('Bearer ', '');
      const user = token ? await getUserFromToken(token) : null;
      
      return {
        user,
        dataSources: {
          userAPI: new UserAPI(),
          postAPI: new PostAPI(),
        },
      };
    },
  });
  
  console.log(`🚀 Server ready at ${url}`);
};

startServer().catch(error => {
  console.error('Error starting server:', error);
});
```

### Express Integration
```javascript
const express = require('express');
const { ApolloServer } = require('@apollo/server');
const { expressMiddleware } = require('@apollo/server/express4');
const { ApolloServerPluginDrainHttpServer } = require('@apollo/server/plugin/drainHttpServer');
const http = require('http');
const cors = require('cors');
const bodyParser = require('body-parser');

const app = express();
const httpServer = http.createServer(app);

const server = new ApolloServer({
  typeDefs,
  resolvers,
  plugins: [ApolloServerPluginDrainHttpServer({ httpServer })],
});

const startServer = async () => {
  await server.start();
  
  app.use(
    '/graphql',
    cors(),
    bodyParser.json(),
    expressMiddleware(server, {
      context: async ({ req }) => ({
        token: req.headers.authorization,
        user: await getUserFromToken(req.headers.authorization),
      }),
    }),
  );
  
  await new Promise((resolve) => httpServer.listen({ port: 4000 }, resolve));
  console.log(`🚀 Server ready at http://localhost:4000/graphql`);
};

startServer();
```

## Resolvers & Data Sources

### Basic Resolvers
```javascript
// resolvers.js
const resolvers = {
  Query: {
    users: async (parent, args, context) => {
      return await context.dataSources.userAPI.getUsers();
    },
    
    user: async (parent, { id }, context) => {
      return await context.dataSources.userAPI.getUserById(id);
    },
    
    posts: async (parent, { published }, context) => {
      return await context.dataSources.postAPI.getPosts({ published });
    },
  },
  
  Mutation: {
    createUser: async (parent, { input }, context) => {
      // Check authentication
      if (!context.user) {
        throw new Error('Authentication required');
      }
      
      return await context.dataSources.userAPI.createUser(input);
    },
    
    updateUser: async (parent, { id, input }, context) => {
      // Check authorization
      if (!context.user || (context.user.id !== id && context.user.role !== 'ADMIN')) {
        throw new Error('Not authorized');
      }
      
      return await context.dataSources.userAPI.updateUser(id, input);
    },
  },
  
  // Field resolvers
  User: {
    posts: async (parent, args, context) => {
      return await context.dataSources.postAPI.getPostsByAuthor(parent.id);
    },
    
    fullName: (parent) => {
      return `${parent.firstName} ${parent.lastName}`;
    },
    
    postCount: async (parent, args, context) => {
      const posts = await context.dataSources.postAPI.getPostsByAuthor(parent.id);
      return posts.length;
    },
  },
  
  Post: {
    author: async (parent, args, context) => {
      return await context.dataSources.userAPI.getUserById(parent.authorId);
    },
    
    categories: async (parent, args, context) => {
      return await context.dataSources.categoryAPI.getCategoriesByPost(parent.id);
    },
  },
  
  // Union resolver
  SearchResult: {
    __resolveType(obj) {
      if (obj.email) return 'User';
      if (obj.title) return 'Post';
      if (obj.name && !obj.email) return 'Category';
      return null;
    },
  },
};

module.exports = { resolvers };
```

### Data Sources
```javascript
// dataSources/UserAPI.js
const { DataSource } = require('apollo-datasource');

class UserAPI extends DataSource {
  constructor() {
    super();
  }
  
  initialize(config) {
    this.context = config.context;
  }
  
  async getUsers() {
    // Implement database query
    return await User.findAll();
  }
  
  async getUserById(id) {
    // Use DataLoader for batching
    return await this.context.loaders.user.load(id);
  }
  
  async createUser(input) {
    const user = await User.create(input);
    
    // Clear cache
    this.context.loaders.user.clear(user.id);
    
    return user;
  }
  
  async updateUser(id, input) {
    const user = await User.findByPk(id);
    if (!user) throw new Error('User not found');
    
    await user.update(input);
    
    // Clear cache
    this.context.loaders.user.clear(id);
    
    return user;
  }
}

module.exports = UserAPI;
```

### DataLoader for N+1 Problem
```javascript
// loaders.js
const DataLoader = require('dataloader');

const createUserLoader = () => new DataLoader(async (userIds) => {
  const users = await User.findAll({
    where: { id: userIds }
  });
  
  // Return users in the same order as requested IDs
  return userIds.map(id => users.find(user => user.id === id));
});

const createPostsByAuthorLoader = () => new DataLoader(async (authorIds) => {
  const posts = await Post.findAll({
    where: { authorId: authorIds }
  });
  
  // Group posts by author ID
  return authorIds.map(authorId => 
    posts.filter(post => post.authorId === authorId)
  );
});

const createLoaders = () => ({
  user: createUserLoader(),
  postsByAuthor: createPostsByAuthorLoader(),
});

module.exports = { createLoaders };
```

## Client Queries

### Basic Queries
```graphql
# Simple query
query GetUsers {
  users {
    id
    name
    email
  }
}

# Query with variables
query GetUser($userId: ID!) {
  user(id: $userId) {
    id
    name
    email
    posts {
      id
      title
      publishedAt
    }
  }
}

# Query with fragments
fragment UserInfo on User {
  id
  name
  email
  createdAt
}

query GetUsersWithPosts {
  users {
    ...UserInfo
    posts {
      id
      title
      content
    }
  }
}
```

### Mutations
```graphql
# Create mutation
mutation CreateUser($input: CreateUserInput!) {
  createUser(input: $input) {
    id
    name
    email
    createdAt
  }
}

# Update mutation
mutation UpdateUser($id: ID!, $input: UpdateUserInput!) {
  updateUser(id: $id, input: $input) {
    id
    name
    email
    updatedAt
  }
}

# Variables
{
  "input": {
    "name": "John Doe",
    "email": "john@example.com",
    "password": "securepassword"
  }
}
```

### Subscriptions
```graphql
# Real-time subscription
subscription OnUserCreated {
  userCreated {
    id
    name
    email
    createdAt
  }
}

# Filtered subscription
subscription OnPostPublished($authorId: ID) {
  postPublished(authorId: $authorId) {
    id
    title
    author {
      name
    }
    publishedAt
  }
}
```

## Apollo Client Setup

### React Setup
```javascript
// apollo-client.js
import { ApolloClient, InMemoryCache, createHttpLink } from '@apollo/client';
import { setContext } from '@apollo/client/link/context';

const httpLink = createHttpLink({
  uri: 'http://localhost:4000/graphql',
});

const authLink = setContext((_, { headers }) => {
  const token = localStorage.getItem('token');
  return {
    headers: {
      ...headers,
      authorization: token ? `Bearer ${token}` : "",
    }
  }
});

const client = new ApolloClient({
  link: authLink.concat(httpLink),
  cache: new InMemoryCache({
    typePolicies: {
      User: {
        fields: {
          posts: {
            merge(existing = [], incoming) {
              return [...existing, ...incoming];
            }
          }
        }
      }
    }
  }),
  defaultOptions: {
    watchQuery: {
      errorPolicy: 'all'
    }
  }
});

export default client;
```

### React Component Usage
```jsx
// App.js
import { ApolloProvider } from '@apollo/client';
import client from './apollo-client';
import UserList from './components/UserList';

function App() {
  return (
    <ApolloProvider client={client}>
      <div className="App">
        <UserList />
      </div>
    </ApolloProvider>
  );
}

export default App;
```

```jsx
// components/UserList.jsx
import { useQuery, useMutation } from '@apollo/client';
import { gql } from '@apollo/client';

const GET_USERS = gql`
  query GetUsers {
    users {
      id
      name
      email
      postCount
    }
  }
`;

const CREATE_USER = gql`
  mutation CreateUser($input: CreateUserInput!) {
    createUser(input: $input) {
      id
      name
      email
    }
  }
`;

function UserList() {
  const { loading, error, data, refetch } = useQuery(GET_USERS, {
    pollInterval: 30000, // Poll every 30 seconds
    notifyOnNetworkStatusChange: true
  });
  
  const [createUser, { loading: creating }] = useMutation(CREATE_USER, {
    update(cache, { data: { createUser } }) {
      // Update cache after mutation
      const { users } = cache.readQuery({ query: GET_USERS });
      cache.writeQuery({
        query: GET_USERS,
        data: { users: [...users, createUser] }
      });
    }
  });

  const handleCreateUser = async (userData) => {
    try {
      await createUser({
        variables: { input: userData }
      });
    } catch (error) {
      console.error('Error creating user:', error);
    }
  };

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <div>
      <h2>Users</h2>
      <button onClick={() => refetch()}>Refresh</button>
      
      {data.users.map(user => (
        <div key={user.id}>
          <h3>{user.name}</h3>
          <p>{user.email}</p>
          <p>Posts: {user.postCount}</p>
        </div>
      ))}
      
      <CreateUserForm 
        onSubmit={handleCreateUser} 
        loading={creating} 
      />
    </div>
  );
}
```

### Subscription Usage
```jsx
// components/LiveUserFeed.jsx
import { useSubscription } from '@apollo/client';
import { gql } from '@apollo/client';

const USER_CREATED_SUBSCRIPTION = gql`
  subscription OnUserCreated {
    userCreated {
      id
      name
      email
      createdAt
    }
  }
`;

function LiveUserFeed() {
  const { data, loading, error } = useSubscription(USER_CREATED_SUBSCRIPTION, {
    onSubscriptionData: ({ subscriptionData }) => {
      console.log('New user created:', subscriptionData.data.userCreated);
      // Show notification
      showNotification(`New user: ${subscriptionData.data.userCreated.name}`);
    }
  });

  if (loading) return <div>Connecting to live feed...</div>;
  if (error) return <div>Subscription error: {error.message}</div>;

  return (
    <div>
      <h3>Latest User</h3>
      {data?.userCreated && (
        <div>
          <p>{data.userCreated.name}</p>
          <p>{data.userCreated.email}</p>
          <p>Created: {new Date(data.userCreated.createdAt).toLocaleString()}</p>
        </div>
      )}
    </div>
  );
}
```

## Advanced Patterns

### Custom Directives
```javascript
// directives/auth.js
const { SchemaDirectiveVisitor } = require('apollo-server-express');
const { defaultFieldResolver } = require('graphql');

class AuthDirective extends SchemaDirectiveVisitor {
  visitFieldDefinition(field) {
    const { resolve = defaultFieldResolver } = field;
    const requiredRole = this.args.requires;

    field.resolve = async function (...args) {
      const [, , context] = args;
      
      if (!context.user) {
        throw new Error('Authentication required');
      }
      
      if (requiredRole && context.user.role !== requiredRole) {
        throw new Error(`Requires ${requiredRole} role`);
      }
      
      return resolve.apply(this, args);
    };
  }
}

module.exports = AuthDirective;
```

```graphql
# Schema with directive
directive @auth(requires: UserRole = USER) on FIELD_DEFINITION

type Query {
  users: [User!]! @auth
  adminUsers: [User!]! @auth(requires: ADMIN)
}
```

### Error Handling
```javascript
// Custom error classes
class AuthenticationError extends Error {
  constructor(message) {
    super(message);
    this.name = 'AuthenticationError';
    this.extensions = {
      code: 'UNAUTHENTICATED',
      http: { status: 401 }
    };
  }
}

class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = 'ValidationError';
    this.extensions = {
      code: 'BAD_USER_INPUT',
      field,
      http: { status: 400 }
    };
  }
}

// Error formatting
const server = new ApolloServer({
  typeDefs,
  resolvers,
  formatError: (error) => {
    // Log error for monitoring
    console.error('GraphQL Error:', {
      message: error.message,
      locations: error.locations,
      path: error.path,
      extensions: error.extensions
    });

    // Don't expose internal errors in production
    if (process.env.NODE_ENV === 'production' && !error.extensions?.code) {
      return new Error('Internal server error');
    }

    return error;
  }
});
```

### Schema Stitching
```javascript
// Combining multiple schemas
const { stitchSchemas } = require('@graphql-tools/stitch');

const userSchema = makeExecutableSchema({
  typeDefs: userTypeDefs,
  resolvers: userResolvers
});

const postSchema = makeExecutableSchema({
  typeDefs: postTypeDefs,
  resolvers: postResolvers
});

const stitchedSchema = stitchSchemas({
  schemas: [userSchema, postSchema],
  resolvers: {
    User: {
      posts: {
        selectionSet: '{ id }',
        resolve(user, args, context, info) {
          return delegateToSchema({
            schema: postSchema,
            operation: 'query',
            fieldName: 'postsByAuthor',
            args: { authorId: user.id },
            context,
            info
          });
        }
      }
    }
  }
});
```

## Performance & Optimization

### Query Complexity Analysis
```javascript
const depthLimit = require('graphql-depth-limit');
const costAnalysis = require('graphql-query-complexity');

const server = new ApolloServer({
  typeDefs,
  resolvers,
  validationRules: [
    depthLimit(10), // Limit query depth
    costAnalysis({
      maximumCost: 1000,
      createError: (max, actual) => {
        return new Error(`Query cost ${actual} exceeds maximum cost ${max}`);
      },
      scalarCost: 1,
      objectCost: 2,
      listFactor: 10,
      introspectionCost: 1000
    })
  ]
});
```

### Caching Strategies
```javascript
// Redis cache
const Redis = require('ioredis');
const redis = new Redis(process.env.REDIS_URL);

const resolvers = {
  Query: {
    user: async (parent, { id }, context) => {
      // Check cache first
      const cacheKey = `user:${id}`;
      const cached = await redis.get(cacheKey);
      
      if (cached) {
        return JSON.parse(cached);
      }
      
      // Fetch from database
      const user = await context.dataSources.userAPI.getUserById(id);
      
      // Cache for 5 minutes
      await redis.setex(cacheKey, 300, JSON.stringify(user));
      
      return user;
    }
  }
};

// Response caching
const responseCachePlugin = require('apollo-server-plugin-response-cache');

const server = new ApolloServer({
  typeDefs,
  resolvers,
  plugins: [
    responseCachePlugin({
      sessionId: (requestContext) => {
        return requestContext.request.http.headers.get('authorization') || null;
      }
    })
  ]
});
```

### Persisted Queries
```javascript
// Client-side
import { createPersistedQueryLink } from '@apollo/client/link/persisted-queries';
import { sha256 } from 'crypto-hash';

const persistedQueriesLink = createPersistedQueryLink({ sha256 });

const client = new ApolloClient({
  link: persistedQueriesLink.concat(httpLink),
  cache: new InMemoryCache()
});
```

## 🔗 Related Notes

- [[API Design]] - RESTful API design patterns
- [[Authentication]] - Auth strategies for GraphQL
- [[Node.js]] - Server-side JavaScript runtime
- [[React]] - Frontend framework integration
- [[Testing]] - GraphQL testing strategies

## 📚 Additional Resources

- [GraphQL Official Documentation](https://graphql.org/)
- [Apollo Server Documentation](https://www.apollographql.com/docs/apollo-server/)
- [Apollo Client Documentation](https://www.apollographql.com/docs/react/)
- [GraphQL Best Practices](https://graphql.org/learn/best-practices/)
- [GraphQL Schema Design Guide](https://www.apollographql.com/blog/graphql-schema-design-building-evolvable-schemas/)

---
*Last updated: January 27, 2025* 