---
tags: [backend, api, rest, design, architecture]
created: 2025-06-15
updated: 2025-06-15
---

# 🌐 API Design

> **Summary**: Comprehensive guide to designing robust, scalable APIs with RESTful conventions, proper error handling, versioning strategies, and pagination patterns.

## 📋 Table of Contents

- [RESTful Conventions](#restful-conventions)
- [HTTP Methods & Status Codes](#http-methods--status-codes)
- [URL Structure & Naming](#url-structure--naming)
- [Error Handling](#error-handling)
- [Pagination Strategies](#pagination-strategies)
- [API Versioning](#api-versioning)
- [Request/Response Patterns](#requestresponse-patterns)
- [Security Considerations](#security-considerations)

## RESTful Conventions

### Resource-Based URLs
```
✅ Good
GET /api/users          # Get all users
GET /api/users/123      # Get specific user
POST /api/users         # Create new user
PUT /api/users/123      # Update entire user
PATCH /api/users/123    # Partial update
DELETE /api/users/123   # Delete user

❌ Avoid
GET /api/getUsers
POST /api/createUser
GET /api/user-details?id=123
```

### Nested Resources
```
GET /api/users/123/posts        # Get posts by user 123
POST /api/users/123/posts       # Create post for user 123
GET /api/posts/456/comments     # Get comments for post 456
```

### Query Parameters for Filtering
```
GET /api/users?status=active&role=admin
GET /api/posts?author=123&published=true&limit=10
GET /api/products?category=electronics&price_min=100&price_max=500
```

## HTTP Methods & Status Codes

### HTTP Methods
| Method | Purpose | Idempotent | Safe |
|--------|---------|------------|------|
| GET | Retrieve data | ✅ | ✅ |
| POST | Create resource | ❌ | ❌ |
| PUT | Replace entire resource | ✅ | ❌ |
| PATCH | Partial update | ❌ | ❌ |
| DELETE | Remove resource | ✅ | ❌ |

### Status Codes
```javascript
// Success (2xx)
200 OK              // Successful GET, PUT, PATCH
201 Created         // Successful POST
204 No Content      // Successful DELETE

// Client Error (4xx)
400 Bad Request     // Invalid request data
401 Unauthorized    // Authentication required
403 Forbidden       // Access denied
404 Not Found       // Resource doesn't exist
409 Conflict        // Resource conflict
422 Unprocessable   // Validation errors

// Server Error (5xx)
500 Internal Error  // Server-side error
502 Bad Gateway     // Upstream server error
503 Service Unavailable // Temporary unavailability
```

## URL Structure & Naming

### Best Practices
```
✅ Use nouns, not verbs
/api/users (not /api/getUsers)

✅ Use plural nouns for collections
/api/products (not /api/product)

✅ Use kebab-case for multi-word resources
/api/user-profiles (not /api/userProfiles)

✅ Keep URLs short and intuitive
/api/users/123/orders/456

✅ Use consistent patterns
/api/{resource}/{id}/{sub-resource}/{sub-id}
```

### URL Hierarchy
```
/api/v1/users                    # Collection
/api/v1/users/123               # Specific resource
/api/v1/users/123/profile       # Sub-resource
/api/v1/users/123/orders        # Related collection
/api/v1/users/123/orders/456    # Specific related resource
```

## Error Handling

### Consistent Error Response Format
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "The request contains invalid data",
    "details": [
      {
        "field": "email",
        "message": "Email format is invalid"
      },
      {
        "field": "age",
        "message": "Age must be between 18 and 120"
      }
    ],
    "timestamp": "2025-01-27T10:30:00Z",
    "path": "/api/users",
    "requestId": "req_123456789"
  }
}
```

### Error Code Categories
```javascript
// Validation Errors
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": [/* field-specific errors */]
  }
}

// Authentication Errors
{
  "error": {
    "code": "AUTHENTICATION_REQUIRED",
    "message": "Valid authentication token required"
  }
}

// Authorization Errors
{
  "error": {
    "code": "INSUFFICIENT_PERMISSIONS",
    "message": "You don't have permission to access this resource"
  }
}

// Resource Errors
{
  "error": {
    "code": "RESOURCE_NOT_FOUND",
    "message": "User with ID 123 not found"
  }
}

// Rate Limiting
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests. Try again in 60 seconds",
    "retryAfter": 60
  }
}
```

### Express.js Error Handler Example
```javascript
// Custom error class
class APIError extends Error {
  constructor(message, statusCode, code, details = null) {
    super(message);
    this.statusCode = statusCode;
    this.code = code;
    this.details = details;
  }
}

// Global error handler middleware
const errorHandler = (err, req, res, next) => {
  const error = {
    code: err.code || 'INTERNAL_ERROR',
    message: err.message || 'An unexpected error occurred',
    timestamp: new Date().toISOString(),
    path: req.path,
    requestId: req.id
  };

  if (err.details) {
    error.details = err.details;
  }

  // Log error for monitoring
  console.error('API Error:', {
    ...error,
    stack: err.stack,
    userId: req.user?.id
  });

  res.status(err.statusCode || 500).json({ error });
};

// Usage in route
app.post('/api/users', async (req, res, next) => {
  try {
    const validationErrors = validateUser(req.body);
    if (validationErrors.length > 0) {
      throw new APIError(
        'Validation failed',
        422,
        'VALIDATION_ERROR',
        validationErrors
      );
    }
    
    const user = await createUser(req.body);
    res.status(201).json({ data: user });
  } catch (error) {
    next(error);
  }
});
```

## Pagination Strategies

### Offset-Based Pagination
```javascript
// Request
GET /api/users?page=2&limit=20

// Response
{
  "data": [/* users */],
  "pagination": {
    "page": 2,
    "limit": 20,
    "total": 150,
    "totalPages": 8,
    "hasNext": true,
    "hasPrev": true
  }
}

// Implementation
const getUsers = async (page = 1, limit = 20) => {
  const offset = (page - 1) * limit;
  const users = await User.findAndCountAll({
    offset,
    limit,
    order: [['createdAt', 'DESC']]
  });

  return {
    data: users.rows,
    pagination: {
      page: parseInt(page),
      limit: parseInt(limit),
      total: users.count,
      totalPages: Math.ceil(users.count / limit),
      hasNext: offset + limit < users.count,
      hasPrev: page > 1
    }
  };
};
```

### Cursor-Based Pagination (Better for Real-time Data)
```javascript
// Request
GET /api/posts?cursor=eyJpZCI6MTIzfQ&limit=20

// Response
{
  "data": [/* posts */],
  "pagination": {
    "nextCursor": "eyJpZCI6MTQ0fQ",
    "prevCursor": "eyJpZCI6MTAyfQ",
    "hasNext": true,
    "hasPrev": true,
    "limit": 20
  }
}

// Implementation
const getPosts = async (cursor = null, limit = 20) => {
  let whereClause = {};
  
  if (cursor) {
    const decodedCursor = JSON.parse(Buffer.from(cursor, 'base64').toString());
    whereClause.id = { [Op.lt]: decodedCursor.id };
  }

  const posts = await Post.findAll({
    where: whereClause,
    limit: limit + 1, // Get one extra to check if there's a next page
    order: [['id', 'DESC']]
  });

  const hasNext = posts.length > limit;
  if (hasNext) posts.pop(); // Remove the extra item

  const nextCursor = hasNext 
    ? Buffer.from(JSON.stringify({ id: posts[posts.length - 1].id })).toString('base64')
    : null;

  return {
    data: posts,
    pagination: {
      nextCursor,
      hasNext,
      limit
    }
  };
};
```

## API Versioning

### URL Versioning (Recommended)
```
/api/v1/users
/api/v2/users
/api/v3/users
```

### Header Versioning
```http
GET /api/users
Accept: application/vnd.api+json;version=1
```

### Query Parameter Versioning
```
/api/users?version=1
```

### Implementation Example
```javascript
// Express.js versioning middleware
const versionMiddleware = (req, res, next) => {
  // Extract version from URL
  const urlVersion = req.path.match(/\/v(\d+)\//)?.[1];
  
  // Extract version from header
  const headerVersion = req.headers['api-version'];
  
  // Default to latest version
  req.apiVersion = urlVersion || headerVersion || '1';
  
  next();
};

// Version-specific route handlers
const userHandlers = {
  v1: require('./handlers/v1/users'),
  v2: require('./handlers/v2/users'),
  v3: require('./handlers/v3/users')
};

app.get('/api/v:version/users', versionMiddleware, (req, res) => {
  const handler = userHandlers[`v${req.apiVersion}`];
  if (!handler) {
    return res.status(400).json({
      error: {
        code: 'UNSUPPORTED_VERSION',
        message: `API version ${req.apiVersion} is not supported`
      }
    });
  }
  
  handler.getUsers(req, res);
});
```

### Version Deprecation Strategy
```javascript
// Deprecation warning middleware
const deprecationWarning = (version, sunsetDate) => (req, res, next) => {
  res.set({
    'Deprecation': 'true',
    'Sunset': sunsetDate,
    'Link': '</api/v3/users>; rel="successor-version"'
  });
  
  console.warn(`Deprecated API v${version} accessed`, {
    path: req.path,
    userAgent: req.get('User-Agent'),
    ip: req.ip
  });
  
  next();
};

// Apply to deprecated versions
app.use('/api/v1/*', deprecationWarning('1', '2025-12-31'));
app.use('/api/v2/*', deprecationWarning('2', '2026-06-30'));
```

## Request/Response Patterns

### Consistent Response Structure
```javascript
// Success Response
{
  "data": {/* actual data */},
  "meta": {
    "timestamp": "2025-01-27T10:30:00Z",
    "version": "1.0.0",
    "requestId": "req_123456789"
  }
}

// Collection Response
{
  "data": [/* array of items */],
  "pagination": {/* pagination info */},
  "meta": {/* metadata */}
}

// Error Response
{
  "error": {/* error details */},
  "meta": {/* metadata */}
}
```

### Request Validation
```javascript
const { body, validationResult } = require('express-validator');

// Validation rules
const userValidation = [
  body('email').isEmail().normalizeEmail(),
  body('password').isLength({ min: 8 }).matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/),
  body('age').isInt({ min: 18, max: 120 }),
  body('name').trim().isLength({ min: 2, max: 50 })
];

// Validation middleware
const handleValidation = (req, res, next) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    const details = errors.array().map(error => ({
      field: error.path,
      message: error.msg,
      value: error.value
    }));
    
    return res.status(422).json({
      error: {
        code: 'VALIDATION_ERROR',
        message: 'Request validation failed',
        details
      }
    });
  }
  next();
};

// Usage
app.post('/api/users', userValidation, handleValidation, createUser);
```

## Security Considerations

### Input Sanitization
```javascript
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');
const mongoSanitize = require('express-mongo-sanitize');

// Security middleware
app.use(helmet()); // Security headers
app.use(mongoSanitize()); // Prevent NoSQL injection
app.use(express.json({ limit: '10mb' })); // Limit payload size

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Limit each IP to 100 requests per windowMs
  message: {
    error: {
      code: 'RATE_LIMIT_EXCEEDED',
      message: 'Too many requests from this IP'
    }
  }
});

app.use('/api/', limiter);
```

### Authentication & Authorization
```javascript
// JWT middleware
const authenticateToken = (req, res, next) => {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1];

  if (!token) {
    return res.status(401).json({
      error: {
        code: 'AUTHENTICATION_REQUIRED',
        message: 'Access token is required'
      }
    });
  }

  jwt.verify(token, process.env.JWT_SECRET, (err, user) => {
    if (err) {
      return res.status(403).json({
        error: {
          code: 'INVALID_TOKEN',
          message: 'Invalid or expired token'
        }
      });
    }
    req.user = user;
    next();
  });
};

// Role-based authorization
const requireRole = (roles) => (req, res, next) => {
  if (!roles.includes(req.user.role)) {
    return res.status(403).json({
      error: {
        code: 'INSUFFICIENT_PERMISSIONS',
        message: 'You do not have permission to access this resource'
      }
    });
  }
  next();
};

// Usage
app.get('/api/admin/users', authenticateToken, requireRole(['admin']), getUsers);
```

## 🔗 Related Notes

- [[GraphQL]] - Alternative API design approach
- [[Authentication]] - Detailed auth patterns and implementations
- [[Node.js]] - Server-side JavaScript runtime
- [[Testing]] - API testing strategies
- [[CI-CD & Deployment]] - API deployment patterns

## 📚 Additional Resources

- [REST API Design Best Practices](https://restfulapi.net/)
- [HTTP Status Codes Reference](https://httpstatuses.com/)
- [API Security Checklist](https://github.com/shieldfy/API-Security-Checklist)
- [OpenAPI Specification](https://swagger.io/specification/)

---
*Last updated: January 27, 2025* 