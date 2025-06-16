---
tags: [testing, e2e, playwright, cypress, automation, integration]
created: 2025-06-15
updated: 2025-06-15
---

# 🎭 End-to-End Testing

> **Summary**: Comprehensive guide to E2E testing with Playwright and Cypress, including setup, test structure, best practices, and framework comparison.

## 📋 Table of Contents

- [Playwright vs Cypress](#playwright-vs-cypress)
- [Playwright Setup](#playwright-setup)
- [Cypress Setup](#cypress-setup)
- [Test Structure & Patterns](#test-structure--patterns)
- [Advanced Testing Techniques](#advanced-testing-techniques)
- [CI/CD Integration](#cicd-integration)
- [Best Practices](#best-practices)

## Playwright vs Cypress

### Feature Comparison

| Feature | Playwright | Cypress |
|---------|------------|---------|
| **Browser Support** | Chrome, Firefox, Safari, Edge | Chrome, Firefox, Edge |
| **Language Support** | JavaScript, TypeScript, Python, C#, Java | JavaScript, TypeScript |
| **Parallel Testing** | ✅ Built-in | ✅ Paid feature |
| **Mobile Testing** | ✅ Device emulation | ✅ Viewport simulation |
| **Network Interception** | ✅ Full control | ✅ Limited |
| **Multiple Tabs/Windows** | ✅ Native support | ❌ Limited |
| **Real Browser** | ✅ Yes | ✅ Yes |
| **Speed** | ⚡⚡⚡ Very fast | ⚡⚡ Fast |
| **Debugging** | ⚡⚡ Good | ⚡⚡⚡ Excellent |
| **Learning Curve** | ⚡⚡ Moderate | ⚡⚡⚡ Easy |

### When to Choose Each

**Choose Playwright when:**
- Cross-browser testing is critical
- You need mobile device testing
- Performance and speed are priorities
- You want built-in parallel execution
- Multiple tabs/windows testing required

**Choose Cypress when:**
- Developer experience is priority
- Team is new to E2E testing
- You need excellent debugging tools
- Real-time test runner is important
- Strong community support needed

## Playwright Setup

### Installation
```bash
# Install Playwright
npm init playwright@latest

# Or add to existing project
npm install --save-dev @playwright/test

# Install browsers
npx playwright install
```

### Basic Configuration
```javascript
// playwright.config.js
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },

  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
    {
      name: 'Mobile Safari',
      use: { ...devices['iPhone 12'] },
    },
  ],

  webServer: {
    command: 'npm run start',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

### Basic Playwright Test
```javascript
// tests/example.spec.js
import { test, expect } from '@playwright/test';

test('homepage has title and get started link', async ({ page }) => {
  await page.goto('/');

  // Expect a title "to contain" a substring.
  await expect(page).toHaveTitle(/Playwright/);

  // Create a locator for the get started link.
  const getStarted = page.getByRole('link', { name: 'Get started' });

  // Expect an attribute "to be strictly equal" to the value.
  await expect(getStarted).toHaveAttribute('href', '/intro');

  // Click the get started link.
  await getStarted.click();

  // Expects the URL to contain intro.
  await expect(page).toHaveURL(/.*intro/);
});
```

### Advanced Playwright Features
```javascript
// tests/advanced.spec.js
import { test, expect } from '@playwright/test';

test.describe('User Authentication', () => {
  test.beforeEach(async ({ page }) => {
    // Setup before each test
    await page.goto('/login');
  });

  test('should login successfully', async ({ page }) => {
    // Fill login form
    await page.fill('[data-testid="email"]', 'user@example.com');
    await page.fill('[data-testid="password"]', 'password123');
    
    // Submit form
    await page.click('[data-testid="login-button"]');
    
    // Wait for navigation
    await page.waitForURL('/dashboard');
    
    // Verify login success
    await expect(page.getByText('Welcome back')).toBeVisible();
  });

  test('should handle login errors', async ({ page }) => {
    await page.fill('[data-testid="email"]', 'invalid@example.com');
    await page.fill('[data-testid="password"]', 'wrongpassword');
    
    await page.click('[data-testid="login-button"]');
    
    // Check for error message
    await expect(page.getByText('Invalid credentials')).toBeVisible();
  });
});

test.describe('API Integration', () => {
  test('should intercept and mock API calls', async ({ page }) => {
    // Mock API response
    await page.route('**/api/users', async route => {
      const json = [
        { id: 1, name: 'John Doe', email: 'john@example.com' },
        { id: 2, name: 'Jane Smith', email: 'jane@example.com' }
      ];
      await route.fulfill({ json });
    });

    await page.goto('/users');
    
    // Verify mocked data is displayed
    await expect(page.getByText('John Doe')).toBeVisible();
    await expect(page.getByText('Jane Smith')).toBeVisible();
  });

  test('should test file upload', async ({ page }) => {
    await page.goto('/upload');
    
    // Upload file
    const fileChooserPromise = page.waitForEvent('filechooser');
    await page.getByText('Upload file').click();
    const fileChooser = await fileChooserPromise;
    await fileChooser.setFiles('tests/fixtures/sample.pdf');
    
    // Verify upload success
    await expect(page.getByText('File uploaded successfully')).toBeVisible();
  });
});
```

## Cypress Setup

### Installation
```bash
# Install Cypress
npm install --save-dev cypress

# Open Cypress for first time setup
npx cypress open
```

### Basic Configuration
```javascript
// cypress.config.js
import { defineConfig } from 'cypress'

export default defineConfig({
  e2e: {
    baseUrl: 'http://localhost:3000',
    viewportWidth: 1280,
    viewportHeight: 720,
    video: true,
    screenshotOnRunFailure: true,
    
    setupNodeEvents(on, config) {
      // implement node event listeners here
    },
    
    env: {
      apiUrl: 'http://localhost:8000/api',
    },
    
    retries: {
      runMode: 2,
      openMode: 0
    }
  },
  
  component: {
    devServer: {
      framework: 'react',
      bundler: 'vite',
    },
  },
})
```

### Basic Cypress Test
```javascript
// cypress/e2e/example.cy.js
describe('Homepage', () => {
  beforeEach(() => {
    cy.visit('/');
  });

  it('should display the homepage title', () => {
    cy.title().should('contain', 'My App');
    cy.get('[data-cy="welcome-message"]').should('be.visible');
  });

  it('should navigate to about page', () => {
    cy.get('[data-cy="nav-about"]').click();
    cy.url().should('include', '/about');
    cy.get('h1').should('contain', 'About Us');
  });
});
```

### Advanced Cypress Features
```javascript
// cypress/e2e/user-flow.cy.js
describe('User Registration Flow', () => {
  beforeEach(() => {
    // Reset database state
    cy.task('db:seed');
  });

  it('should complete user registration', () => {
    cy.visit('/register');
    
    // Fill registration form
    cy.get('[data-cy="first-name"]').type('John');
    cy.get('[data-cy="last-name"]').type('Doe');
    cy.get('[data-cy="email"]').type('john.doe@example.com');
    cy.get('[data-cy="password"]').type('SecurePassword123!');
    cy.get('[data-cy="confirm-password"]').type('SecurePassword123!');
    
    // Submit form
    cy.get('[data-cy="register-button"]').click();
    
    // Verify success
    cy.url().should('include', '/welcome');
    cy.get('[data-cy="success-message"]')
      .should('contain', 'Registration successful');
  });

  it('should handle form validation', () => {
    cy.visit('/register');
    
    // Try to submit empty form
    cy.get('[data-cy="register-button"]').click();
    
    // Check validation messages
    cy.get('[data-cy="first-name-error"]')
      .should('contain', 'First name is required');
    cy.get('[data-cy="email-error"]')
      .should('contain', 'Email is required');
  });
});

// Custom commands
// cypress/support/commands.js
Cypress.Commands.add('login', (email, password) => {
  cy.session([email, password], () => {
    cy.visit('/login');
    cy.get('[data-cy="email"]').type(email);
    cy.get('[data-cy="password"]').type(password);
    cy.get('[data-cy="login-button"]').click();
    cy.url().should('include', '/dashboard');
  });
});

Cypress.Commands.add('createUser', (userData) => {
  cy.request({
    method: 'POST',
    url: `${Cypress.env('apiUrl')}/users`,
    body: userData,
  });
});

// Usage in tests
describe('Dashboard', () => {
  beforeEach(() => {
    cy.login('user@example.com', 'password123');
    cy.visit('/dashboard');
  });

  it('should display user dashboard', () => {
    cy.get('[data-cy="user-name"]').should('contain', 'John Doe');
    cy.get('[data-cy="dashboard-stats"]').should('be.visible');
  });
});
```

## Test Structure & Patterns

### Page Object Model (Playwright)
```javascript
// pages/LoginPage.js
export class LoginPage {
  constructor(page) {
    this.page = page;
    this.emailInput = page.getByTestId('email');
    this.passwordInput = page.getByTestId('password');
    this.loginButton = page.getByTestId('login-button');
    this.errorMessage = page.getByTestId('error-message');
  }

  async goto() {
    await this.page.goto('/login');
  }

  async login(email, password) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.loginButton.click();
  }

  async getErrorMessage() {
    return await this.errorMessage.textContent();
  }
}

// tests/login.spec.js
import { test, expect } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';

test('user can login', async ({ page }) => {
  const loginPage = new LoginPage(page);
  
  await loginPage.goto();
  await loginPage.login('user@example.com', 'password123');
  
  await expect(page).toHaveURL('/dashboard');
});
```

### Page Object Model (Cypress)
```javascript
// cypress/pages/LoginPage.js
export class LoginPage {
  visit() {
    cy.visit('/login');
  }

  fillEmail(email) {
    cy.get('[data-cy="email"]').type(email);
    return this;
  }

  fillPassword(password) {
    cy.get('[data-cy="password"]').type(password);
    return this;
  }

  submit() {
    cy.get('[data-cy="login-button"]').click();
    return this;
  }

  getErrorMessage() {
    return cy.get('[data-cy="error-message"]');
  }
}

// cypress/e2e/login.cy.js
import { LoginPage } from '../pages/LoginPage';

describe('Login', () => {
  const loginPage = new LoginPage();

  it('should login successfully', () => {
    loginPage
      .visit()
      .fillEmail('user@example.com')
      .fillPassword('password123')
      .submit();
    
    cy.url().should('include', '/dashboard');
  });
});
```

### Test Data Management
```javascript
// fixtures/users.json
{
  "validUser": {
    "email": "user@example.com",
    "password": "password123",
    "firstName": "John",
    "lastName": "Doe"
  },
  "adminUser": {
    "email": "admin@example.com",
    "password": "admin123",
    "role": "admin"
  }
}

// Playwright usage
import userData from '../fixtures/users.json';

test('admin can access admin panel', async ({ page }) => {
  const loginPage = new LoginPage(page);
  await loginPage.goto();
  await loginPage.login(userData.adminUser.email, userData.adminUser.password);
  
  await expect(page.getByText('Admin Panel')).toBeVisible();
});

// Cypress usage
describe('User Management', () => {
  beforeEach(() => {
    cy.fixture('users').as('userData');
  });

  it('should create new user', function() {
    cy.login(this.userData.adminUser.email, this.userData.adminUser.password);
    // Test implementation
  });
});
```

## Advanced Testing Techniques

### Visual Testing (Playwright)
```javascript
// tests/visual.spec.js
import { test, expect } from '@playwright/test';

test('homepage visual regression', async ({ page }) => {
  await page.goto('/');
  
  // Full page screenshot
  await expect(page).toHaveScreenshot('homepage.png');
  
  // Element screenshot
  await expect(page.getByTestId('hero-section')).toHaveScreenshot('hero.png');
});

test('responsive design', async ({ page }) => {
  await page.goto('/');
  
  // Test different viewports
  await page.setViewportSize({ width: 375, height: 667 }); // Mobile
  await expect(page).toHaveScreenshot('mobile-homepage.png');
  
  await page.setViewportSize({ width: 1024, height: 768 }); // Tablet
  await expect(page).toHaveScreenshot('tablet-homepage.png');
});
```

### API Testing Integration
```javascript
// Playwright API testing
import { test, expect } from '@playwright/test';

test('API and UI integration', async ({ page, request }) => {
  // Create user via API
  const response = await request.post('/api/users', {
    data: {
      name: 'Test User',
      email: 'test@example.com'
    }
  });
  
  expect(response.ok()).toBeTruthy();
  const user = await response.json();
  
  // Verify user appears in UI
  await page.goto('/users');
  await expect(page.getByText(user.name)).toBeVisible();
});

// Cypress API testing
describe('API Integration', () => {
  it('should sync API and UI state', () => {
    // Create user via API
    cy.request('POST', '/api/users', {
      name: 'Test User',
      email: 'test@example.com'
    }).then((response) => {
      expect(response.status).to.eq(201);
      
      // Visit UI and verify user
      cy.visit('/users');
      cy.contains(response.body.name).should('be.visible');
    });
  });
});
```

### Performance Testing
```javascript
// Playwright performance testing
import { test, expect } from '@playwright/test';

test('page load performance', async ({ page }) => {
  const startTime = Date.now();
  
  await page.goto('/');
  
  // Wait for page to be fully loaded
  await page.waitForLoadState('networkidle');
  
  const loadTime = Date.now() - startTime;
  expect(loadTime).toBeLessThan(3000); // 3 seconds
  
  // Check Core Web Vitals
  const metrics = await page.evaluate(() => {
    return new Promise((resolve) => {
      new PerformanceObserver((list) => {
        const entries = list.getEntries();
        resolve(entries);
      }).observe({ entryTypes: ['navigation', 'paint'] });
    });
  });
  
  console.log('Performance metrics:', metrics);
});
```

## CI/CD Integration

### GitHub Actions (Playwright)
```yaml
# .github/workflows/playwright.yml
name: Playwright Tests

on:
  push:
    branches: [ main, master ]
  pull_request:
    branches: [ main, master ]

jobs:
  test:
    timeout-minutes: 60
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - uses: actions/setup-node@v3
      with:
        node-version: 18
        
    - name: Install dependencies
      run: npm ci
      
    - name: Install Playwright Browsers
      run: npx playwright install --with-deps
      
    - name: Run Playwright tests
      run: npx playwright test
      
    - uses: actions/upload-artifact@v3
      if: always()
      with:
        name: playwright-report
        path: playwright-report/
        retention-days: 30
```

### GitHub Actions (Cypress)
```yaml
# .github/workflows/cypress.yml
name: Cypress Tests

on:
  push:
    branches: [ main, master ]
  pull_request:
    branches: [ main, master ]

jobs:
  cypress-run:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
        
      - name: Cypress run
        uses: cypress-io/github-action@v5
        with:
          build: npm run build
          start: npm start
          wait-on: 'http://localhost:3000'
          
      - name: Upload screenshots
        uses: actions/upload-artifact@v3
        if: failure()
        with:
          name: cypress-screenshots
          path: cypress/screenshots
```

### Docker Integration
```dockerfile
# Dockerfile.e2e
FROM mcr.microsoft.com/playwright:v1.40.0-focal

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .

CMD ["npx", "playwright", "test"]
```

```yaml
# docker-compose.e2e.yml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=test
      
  e2e-tests:
    build:
      context: .
      dockerfile: Dockerfile.e2e
    depends_on:
      - app
    environment:
      - BASE_URL=http://app:3000
    volumes:
      - ./test-results:/app/test-results
```

## Best Practices

### Test Organization
```javascript
// Good: Descriptive test names
test('user can successfully complete checkout with valid payment information', async ({ page }) => {
  // Test implementation
});

// Bad: Vague test names
test('checkout works', async ({ page }) => {
  // Test implementation
});

// Good: Grouped related tests
test.describe('User Authentication', () => {
  test.describe('Login', () => {
    test('with valid credentials');
    test('with invalid credentials');
    test('with missing fields');
  });
  
  test.describe('Registration', () => {
    test('with valid data');
    test('with duplicate email');
  });
});
```

### Reliable Selectors
```javascript
// Good: Use data attributes
await page.click('[data-testid="submit-button"]');
cy.get('[data-cy="submit-button"]').click();

// Good: Use semantic selectors
await page.getByRole('button', { name: 'Submit' }).click();
cy.get('button').contains('Submit').click();

// Avoid: Fragile CSS selectors
await page.click('.btn.btn-primary.submit-btn'); // Bad
cy.get('#content > div:nth-child(2) > button'); // Bad
```

### Wait Strategies
```javascript
// Playwright - Good waiting
await page.waitForSelector('[data-testid="results"]');
await page.waitForLoadState('networkidle');
await expect(page.getByText('Loading...')).toBeHidden();

// Cypress - Good waiting
cy.get('[data-cy="results"]').should('be.visible');
cy.get('[data-cy="loading"]').should('not.exist');

// Avoid: Fixed waits
await page.waitForTimeout(5000); // Bad
cy.wait(5000); // Bad
```

### Test Data Isolation
```javascript
// Playwright - Database cleanup
test.beforeEach(async ({ page }) => {
  await page.request.post('/api/test/reset-db');
});

// Cypress - Database seeding
beforeEach(() => {
  cy.task('db:seed');
});

// Use unique test data
const testUser = {
  email: `test-${Date.now()}@example.com`,
  name: `Test User ${Math.random()}`
};
```

## 🔗 Related Notes

- [[Testing]] - General testing strategies and unit testing
- [[Mocking & Fixtures]] - Test data management and API mocking
- [[CI-CD & Deployment]] - Automated testing in deployment pipelines
- [[Performance Optimization]] - Performance testing strategies

## 📚 Additional Resources

- [Playwright Documentation](https://playwright.dev/)
- [Cypress Documentation](https://docs.cypress.io/)
- [Testing Best Practices](https://github.com/goldbergyoni/javascript-testing-best-practices)
- [E2E Testing Strategies](https://martinfowler.com/articles/practical-test-pyramid.html)

---
*Last updated: January 27, 2025* 