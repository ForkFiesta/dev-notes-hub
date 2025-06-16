---
tags:
  - testing
  - mocking
  - fixtures
  - msw
  - faker
created: 2025-06-15
updated: 2025-06-15
---

# Mocking & Fixtures

Comprehensive guide to mocking APIs and creating test fixtures for reliable, maintainable tests.

## Overview

Mocking and fixtures are essential for creating isolated, fast, and reliable tests. This guide covers modern approaches using MSW (Mock Service Worker) and Faker.js.

## Mock Service Worker (MSW)

### Installation & Setup

```bash
npm install --save-dev msw
# For Node.js testing
npx msw init public/ --save
```

### Basic API Mocking

```javascript
// src/mocks/handlers.js
import { rest } from 'msw'

export const handlers = [
  // GET request
  rest.get('/api/users', (req, res, ctx) => {
    return res(
      ctx.status(200),
      ctx.json([
        { id: 1, name: 'John Doe', email: 'john@example.com' },
        { id: 2, name: 'Jane Smith', email: 'jane@example.com' }
      ])
    )
  }),

  // POST request with validation
  rest.post('/api/users', async (req, res, ctx) => {
    const newUser = await req.json()
    
    if (!newUser.email) {
      return res(
        ctx.status(400),
        ctx.json({ error: 'Email is required' })
      )
    }

    return res(
      ctx.status(201),
      ctx.json({ id: 3, ...newUser })
    )
  }),

  // Dynamic route parameters
  rest.get('/api/users/:id', (req, res, ctx) => {
    const { id } = req.params
    
    return res(
      ctx.status(200),
      ctx.json({ id: Number(id), name: `User ${id}` })
    )
  })
]
```

### Test Setup

```javascript
// src/mocks/server.js
import { setupServer } from 'msw/node'
import { handlers } from './handlers'

export const server = setupServer(...handlers)

// src/setupTests.js
import { server } from './mocks/server'

beforeAll(() => server.listen())
afterEach(() => server.resetHandlers())
afterAll(() => server.close())
```

### Advanced MSW Patterns

```javascript
// Dynamic responses based on request
rest.get('/api/search', (req, res, ctx) => {
  const query = req.url.searchParams.get('q')
  const page = req.url.searchParams.get('page') || '1'
  
  if (!query) {
    return res(ctx.status(400), ctx.json({ error: 'Query required' }))
  }

  const results = mockSearchResults.filter(item => 
    item.title.toLowerCase().includes(query.toLowerCase())
  )

  return res(
    ctx.status(200),
    ctx.json({
      results: results.slice((page - 1) * 10, page * 10),
      total: results.length,
      page: Number(page)
    })
  )
})

// Simulating network delays
rest.get('/api/slow-endpoint', (req, res, ctx) => {
  return res(
    ctx.delay(2000), // 2 second delay
    ctx.status(200),
    ctx.json({ message: 'This was slow!' })
  )
})

// Error scenarios
rest.get('/api/flaky-endpoint', (req, res, ctx) => {
  if (Math.random() < 0.3) {
    return res(ctx.status(500), ctx.json({ error: 'Server error' }))
  }
  return res(ctx.status(200), ctx.json({ success: true }))
})
```

### Test-Specific Overrides

```javascript
// In your test file
import { server } from '../mocks/server'
import { rest } from 'msw'

test('handles API error', async () => {
  // Override handler for this test
  server.use(
    rest.get('/api/users', (req, res, ctx) => {
      return res(ctx.status(500), ctx.json({ error: 'Server error' }))
    })
  )

  // Your test code here
  render(<UserList />)
  expect(await screen.findByText('Error loading users')).toBeInTheDocument()
})
```

## Faker.js for Test Data

### Installation & Basic Usage

```bash
npm install --save-dev @faker-js/faker
```

```javascript
import { faker } from '@faker-js/faker'

// Basic data generation
const user = {
  id: faker.datatype.uuid(),
  name: faker.name.fullName(),
  email: faker.internet.email(),
  avatar: faker.image.avatar(),
  birthDate: faker.date.birthdate(),
  address: {
    street: faker.address.streetAddress(),
    city: faker.address.city(),
    country: faker.address.country()
  }
}
```

### Factory Functions

```javascript
// User factory
export const createUser = (overrides = {}) => ({
  id: faker.datatype.uuid(),
  name: faker.name.fullName(),
  email: faker.internet.email(),
  role: faker.helpers.arrayElement(['user', 'admin', 'moderator']),
  createdAt: faker.date.recent(),
  isActive: faker.datatype.boolean(),
  ...overrides
})

// Product factory
export const createProduct = (overrides = {}) => ({
  id: faker.datatype.uuid(),
  name: faker.commerce.productName(),
  price: parseFloat(faker.commerce.price()),
  description: faker.commerce.productDescription(),
  category: faker.commerce.department(),
  inStock: faker.datatype.number({ min: 0, max: 100 }),
  ...overrides
})

// Usage in tests
const testUser = createUser({ role: 'admin' })
const products = Array.from({ length: 5 }, () => createProduct())
```

### Seeded Data for Consistency

```javascript
// For reproducible test data
faker.seed(123)

// Or create a seeded instance
import { Faker, en } from '@faker-js/faker'
const seededFaker = new Faker({ locale: [en] })
seededFaker.seed(123)

const consistentUser = {
  name: seededFaker.name.fullName(), // Always generates same name
  email: seededFaker.internet.email()
}
```

## Test Data Strategies

### 1. Fixture Files

```javascript
// fixtures/users.json
{
  "users": [
    {
      "id": "1",
      "name": "John Doe",
      "email": "john@example.com",
      "role": "admin"
    },
    {
      "id": "2",
      "name": "Jane Smith",
      "email": "jane@example.com",
      "role": "user"
    }
  ]
}

// Loading fixtures
import usersFixture from '../fixtures/users.json'

export const mockUsers = usersFixture.users
```

### 2. Builder Pattern

```javascript
class UserBuilder {
  constructor() {
    this.user = {
      id: faker.datatype.uuid(),
      name: faker.name.fullName(),
      email: faker.internet.email(),
      role: 'user',
      isActive: true
    }
  }

  withRole(role) {
    this.user.role = role
    return this
  }

  inactive() {
    this.user.isActive = false
    return this
  }

  withEmail(email) {
    this.user.email = email
    return this
  }

  build() {
    return { ...this.user }
  }
}

// Usage
const adminUser = new UserBuilder()
  .withRole('admin')
  .withEmail('admin@company.com')
  .build()
```

### 3. Database Seeding

```javascript
// For integration tests
export const seedDatabase = async () => {
  const users = Array.from({ length: 10 }, () => createUser())
  const products = Array.from({ length: 20 }, () => createProduct())
  
  await db.users.createMany({ data: users })
  await db.products.createMany({ data: products })
  
  return { users, products }
}

// In test setup
beforeEach(async () => {
  await db.clearAll()
  await seedDatabase()
})
```

## Integration with Testing Frameworks

### Jest + React Testing Library

```javascript
// __tests__/UserList.test.js
import { render, screen, waitFor } from '@testing-library/react'
import { server } from '../mocks/server'
import { createUser } from '../factories/userFactory'
import UserList from '../components/UserList'

describe('UserList', () => {
  test('displays users from API', async () => {
    const mockUsers = Array.from({ length: 3 }, () => createUser())
    
    server.use(
      rest.get('/api/users', (req, res, ctx) => {
        return res(ctx.json(mockUsers))
      })
    )

    render(<UserList />)
    
    await waitFor(() => {
      mockUsers.forEach(user => {
        expect(screen.getByText(user.name)).toBeInTheDocument()
      })
    })
  })
})
```

### Cypress Integration

```javascript
// cypress/support/commands.js
Cypress.Commands.add('seedUsers', (count = 5) => {
  const users = Array.from({ length: count }, () => createUser())
  
  cy.request('POST', '/api/test/seed-users', { users })
  return cy.wrap(users)
})

// In test
cy.seedUsers(10).then((users) => {
  cy.visit('/users')
  cy.contains(users[0].name).should('be.visible')
})
```

## Best Practices

### 1. Realistic Data

```javascript
// Good: Realistic data
const user = createUser({
  email: 'john.doe@company.com',
  name: 'John Doe',
  department: 'Engineering'
})

// Avoid: Unrealistic test data
const user = {
  email: 'test@test.com',
  name: 'Test User',
  department: 'Test'
}
```

### 2. Boundary Testing

```javascript
// Test edge cases
const edgeCaseUsers = [
  createUser({ name: '' }), // Empty name
  createUser({ name: 'A'.repeat(255) }), // Max length
  createUser({ email: 'invalid-email' }), // Invalid email
  createUser({ age: -1 }), // Invalid age
  createUser({ age: 150 }) // Extreme age
]
```

### 3. Consistent Test Data

```javascript
// Use constants for important test values
export const TEST_USER_ID = '123e4567-e89b-12d3-a456-426614174000'
export const TEST_ADMIN_EMAIL = 'admin@testcompany.com'

// Reference in multiple tests
const testUser = createUser({ 
  id: TEST_USER_ID,
  email: TEST_ADMIN_EMAIL 
})
```

### 4. Clean Up

```javascript
// Reset mocks between tests
afterEach(() => {
  server.resetHandlers()
  faker.seed() // Reset faker seed
})

// Clear database in integration tests
afterEach(async () => {
  await db.clearAll()
})
```

## Common Patterns

### API Response Mocking

```javascript
// Paginated responses
const createPaginatedResponse = (data, page = 1, limit = 10) => ({
  data: data.slice((page - 1) * limit, page * limit),
  pagination: {
    page,
    limit,
    total: data.length,
    totalPages: Math.ceil(data.length / limit)
  }
})

// Error responses
const createErrorResponse = (message, code = 'GENERIC_ERROR') => ({
  error: {
    message,
    code,
    timestamp: new Date().toISOString()
  }
})
```

### State-Based Mocking

```javascript
// Simulate stateful API
let mockUsers = []

const handlers = [
  rest.get('/api/users', (req, res, ctx) => {
    return res(ctx.json(mockUsers))
  }),
  
  rest.post('/api/users', async (req, res, ctx) => {
    const newUser = await req.json()
    const user = { id: faker.datatype.uuid(), ...newUser }
    mockUsers.push(user)
    return res(ctx.status(201), ctx.json(user))
  }),
  
  rest.delete('/api/users/:id', (req, res, ctx) => {
    const { id } = req.params
    mockUsers = mockUsers.filter(user => user.id !== id)
    return res(ctx.status(204))
  })
]
```

## Related Notes

- [[Testing]] - General testing strategies
- [[Unit Testing]] - Unit testing best practices
- [[End-to-End Testing]] - E2E testing with Playwright/Cypress
- [[API Design]] - API design patterns for better mocking
- [[React Testing]] - React-specific testing approaches

## Resources

- [MSW Documentation](https://mswjs.io/)
- [Faker.js Documentation](https://fakerjs.dev/)
- [Testing Library Best Practices](https://testing-library.com/docs/guiding-principles)
- [Test Data Builders](https://www.natpryce.com/articles/000714.html) 