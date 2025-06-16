# Testing - Unit & Integration Testing

> **Quick Summary**: Testing ensures code reliability and prevents regressions. Master Jest, React Testing Library, and Vitest for comprehensive test coverage in modern web applications.

## 🧪 Testing Fundamentals

### Test Types
```javascript
// Unit Test - Test individual functions/components
function add(a, b) {
    return a + b;
}

test('adds 1 + 2 to equal 3', () => {
    expect(add(1, 2)).toBe(3);
});

// Integration Test - Test component interactions
test('user can submit form', async () => {
    render(<ContactForm />);
    
    fireEvent.change(screen.getByLabelText(/name/i), {
        target: { value: 'John Doe' }
    });
    fireEvent.change(screen.getByLabelText(/email/i), {
        target: { value: 'john@example.com' }
    });
    fireEvent.click(screen.getByRole('button', { name: /submit/i }));
    
    await waitFor(() => {
        expect(screen.getByText(/thank you/i)).toBeInTheDocument();
    });
});

// End-to-End Test - Test complete user workflows
test('user can complete checkout process', async () => {
    // Test full user journey from product selection to payment
});
```

### Test Structure (AAA Pattern)
```javascript
describe('Calculator', () => {
    test('should multiply two numbers', () => {
        // Arrange - Set up test data
        const calculator = new Calculator();
        const a = 5;
        const b = 3;
        
        // Act - Execute the function
        const result = calculator.multiply(a, b);
        
        // Assert - Verify the result
        expect(result).toBe(15);
    });
});
```

## 🃏 Jest - JavaScript Testing Framework

### Basic Setup
```json
// package.json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  },
  "devDependencies": {
    "jest": "^29.0.0",
    "@testing-library/jest-dom": "^5.16.0"
  }
}
```

```javascript
// jest.config.js
module.exports = {
  testEnvironment: 'jsdom',
  setupFilesAfterEnv: ['<rootDir>/src/setupTests.js'],
  moduleNameMapping: {
    '\\.(css|less|scss|sass)$': 'identity-obj-proxy',
    '\\.(jpg|jpeg|png|gif|svg)$': '<rootDir>/__mocks__/fileMock.js'
  },
  collectCoverageFrom: [
    'src/**/*.{js,jsx}',
    '!src/index.js',
    '!src/reportWebVitals.js'
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    }
  }
};
```

### Jest Matchers
```javascript
// Equality
expect(2 + 2).toBe(4); // Exact equality
expect({ name: 'John' }).toEqual({ name: 'John' }); // Deep equality

// Truthiness
expect(true).toBeTruthy();
expect(false).toBeFalsy();
expect(null).toBeNull();
expect(undefined).toBeUndefined();
expect('Hello').toBeDefined();

// Numbers
expect(2 + 2).toBeGreaterThan(3);
expect(Math.PI).toBeCloseTo(3.14, 2);

// Strings
expect('Hello World').toMatch(/World/);
expect('Hello World').toContain('World');

// Arrays
expect(['apple', 'banana']).toContain('apple');
expect(['a', 'b', 'c']).toHaveLength(3);

// Exceptions
expect(() => {
    throw new Error('Something went wrong');
}).toThrow('Something went wrong');

// Async
await expect(fetchUser(1)).resolves.toEqual({ id: 1, name: 'John' });
await expect(fetchUser(-1)).rejects.toThrow('User not found');
```

### Mocking
```javascript
// Mock functions
const mockCallback = jest.fn();
[1, 2, 3].forEach(mockCallback);

expect(mockCallback).toHaveBeenCalledTimes(3);
expect(mockCallback).toHaveBeenCalledWith(1);
expect(mockCallback).toHaveBeenLastCalledWith(3);

// Mock return values
const mockFn = jest.fn();
mockFn.mockReturnValue(42);
mockFn.mockReturnValueOnce(10);

// Mock modules
jest.mock('./api', () => ({
    fetchUser: jest.fn(() => Promise.resolve({ id: 1, name: 'John' }))
}));

// Mock implementation
const mockFetch = jest.fn();
mockFetch.mockImplementation((url) => {
    if (url.includes('users')) {
        return Promise.resolve({
            json: () => Promise.resolve({ users: [] })
        });
    }
});

// Spy on methods
const user = { getName: () => 'John' };
const spy = jest.spyOn(user, 'getName');
user.getName();
expect(spy).toHaveBeenCalled();
spy.mockRestore();
```

## ⚛️ React Testing Library

### Setup
```javascript
// setupTests.js
import '@testing-library/jest-dom';

// Test utilities
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
```

### Component Testing
```javascript
import { render, screen, fireEvent } from '@testing-library/react';
import Counter from './Counter';

// Basic component test
test('renders counter with initial value', () => {
    render(<Counter initialValue={0} />);
    expect(screen.getByText('Count: 0')).toBeInTheDocument();
});

// Testing user interactions
test('increments counter when button is clicked', async () => {
    const user = userEvent.setup();
    render(<Counter initialValue={0} />);
    
    const incrementButton = screen.getByRole('button', { name: /increment/i });
    await user.click(incrementButton);
    
    expect(screen.getByText('Count: 1')).toBeInTheDocument();
});

// Testing forms
test('submits form with correct data', async () => {
    const mockSubmit = jest.fn();
    const user = userEvent.setup();
    
    render(<ContactForm onSubmit={mockSubmit} />);
    
    await user.type(screen.getByLabelText(/name/i), 'John Doe');
    await user.type(screen.getByLabelText(/email/i), 'john@example.com');
    await user.click(screen.getByRole('button', { name: /submit/i }));
    
    expect(mockSubmit).toHaveBeenCalledWith({
        name: 'John Doe',
        email: 'john@example.com'
    });
});
```

### Queries and Assertions
```javascript
// Query methods (in order of preference)
screen.getByRole('button', { name: /submit/i }); // Accessible
screen.getByLabelText(/email/i); // Form labels
screen.getByPlaceholderText(/enter email/i); // Placeholders
screen.getByText(/welcome/i); // Text content
screen.getByDisplayValue('John'); // Form values
screen.getByAltText(/profile picture/i); // Image alt text
screen.getByTitle(/tooltip/i); // Title attributes
screen.getByTestId('submit-button'); // Last resort

// Query variants
screen.getBy... // Throws error if not found
screen.queryBy... // Returns null if not found
screen.findBy... // Async, waits for element

// Multiple elements
screen.getAllByRole('listitem');
screen.queryAllByText(/item/i);
screen.findAllByTestId('product-card');

// Custom matchers
expect(screen.getByRole('button')).toBeEnabled();
expect(screen.getByRole('button')).toBeDisabled();
expect(screen.getByRole('textbox')).toHaveValue('John');
expect(screen.getByRole('checkbox')).toBeChecked();
```

### Testing Async Components
```javascript
// Testing loading states
test('shows loading spinner while fetching data', async () => {
    render(<UserProfile userId={1} />);
    
    expect(screen.getByText(/loading/i)).toBeInTheDocument();
    
    await waitFor(() => {
        expect(screen.queryByText(/loading/i)).not.toBeInTheDocument();
    });
    
    expect(screen.getByText('John Doe')).toBeInTheDocument();
});

// Testing error states
test('displays error message when fetch fails', async () => {
    // Mock API to reject
    jest.spyOn(global, 'fetch').mockRejectedValueOnce(
        new Error('API Error')
    );
    
    render(<UserProfile userId={1} />);
    
    await waitFor(() => {
        expect(screen.getByText(/error loading user/i)).toBeInTheDocument();
    });
});
```

### Testing Context and Providers
```javascript
// Custom render with providers
const renderWithProviders = (ui, options = {}) => {
    const { initialState = {}, ...renderOptions } = options;
    
    const Wrapper = ({ children }) => (
        <ThemeProvider theme="light">
            <UserProvider initialState={initialState}>
                {children}
            </UserProvider>
        </ThemeProvider>
    );
    
    return render(ui, { wrapper: Wrapper, ...renderOptions });
};

// Usage
test('displays user name from context', () => {
    renderWithProviders(<UserProfile />, {
        initialState: { user: { name: 'John Doe' } }
    });
    
    expect(screen.getByText('John Doe')).toBeInTheDocument();
});
```

## ⚡ Vitest - Fast Unit Testing

### Setup
```javascript
// vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: './src/setupTests.js',
    css: true
  }
});
```

```json
// package.json
{
  "scripts": {
    "test": "vitest",
    "test:ui": "vitest --ui",
    "test:coverage": "vitest --coverage"
  },
  "devDependencies": {
    "vitest": "^0.34.0",
    "@vitest/ui": "^0.34.0",
    "jsdom": "^22.0.0"
  }
}
```

### Vitest Features
```javascript
// Hot Module Replacement for tests
import { test, expect, vi } from 'vitest';

// Snapshot testing
test('component renders correctly', () => {
    const { container } = render(<Button>Click me</Button>);
    expect(container.firstChild).toMatchSnapshot();
});

// Inline snapshots
test('formats currency', () => {
    expect(formatCurrency(1234.56)).toMatchInlineSnapshot('"$1,234.56"');
});

// Benchmarking
import { bench, describe } from 'vitest';

describe('sorting algorithms', () => {
    bench('bubble sort', () => {
        bubbleSort([3, 1, 4, 1, 5, 9, 2, 6]);
    });
    
    bench('quick sort', () => {
        quickSort([3, 1, 4, 1, 5, 9, 2, 6]);
    });
});
```

## 🎭 Mocking Strategies

### API Mocking
```javascript
// Mock Service Worker (MSW)
import { rest } from 'msw';
import { setupServer } from 'msw/node';

const server = setupServer(
    rest.get('/api/users/:id', (req, res, ctx) => {
        const { id } = req.params;
        return res(
            ctx.json({
                id: parseInt(id),
                name: 'John Doe',
                email: 'john@example.com'
            })
        );
    })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

// Test using mocked API
test('fetches and displays user', async () => {
    render(<UserProfile userId={1} />);
    
    await waitFor(() => {
        expect(screen.getByText('John Doe')).toBeInTheDocument();
    });
});
```

### Module Mocking
```javascript
// Mock external dependencies
jest.mock('axios', () => ({
    get: jest.fn(() => Promise.resolve({ data: {} })),
    post: jest.fn(() => Promise.resolve({ data: {} }))
}));

// Mock React Router
jest.mock('react-router-dom', () => ({
    ...jest.requireActual('react-router-dom'),
    useNavigate: () => jest.fn(),
    useParams: () => ({ id: '1' })
}));

// Mock localStorage
const localStorageMock = {
    getItem: jest.fn(),
    setItem: jest.fn(),
    removeItem: jest.fn(),
    clear: jest.fn()
};
global.localStorage = localStorageMock;
```

## 📊 Test Coverage and Reporting

### Coverage Configuration
```javascript
// jest.config.js
module.exports = {
    collectCoverageFrom: [
        'src/**/*.{js,jsx,ts,tsx}',
        '!src/**/*.d.ts',
        '!src/index.js',
        '!src/serviceWorker.js'
    ],
    coverageThreshold: {
        global: {
            branches: 80,
            functions: 80,
            lines: 80,
            statements: 80
        },
        './src/components/': {
            branches: 90,
            functions: 90,
            lines: 90,
            statements: 90
        }
    },
    coverageReporters: ['text', 'lcov', 'html']
};
```

### Test Organization
```javascript
// Group related tests
describe('UserService', () => {
    describe('when user exists', () => {
        test('should return user data', () => {
            // Test implementation
        });
        
        test('should cache user data', () => {
            // Test implementation
        });
    });
    
    describe('when user does not exist', () => {
        test('should throw UserNotFoundError', () => {
            // Test implementation
        });
    });
});

// Setup and teardown
describe('Database tests', () => {
    beforeAll(async () => {
        await database.connect();
    });
    
    afterAll(async () => {
        await database.disconnect();
    });
    
    beforeEach(async () => {
        await database.clear();
        await database.seed();
    });
    
    afterEach(async () => {
        await database.clear();
    });
});
```

## 🚀 Best Practices

### Writing Good Tests
```javascript
// ✅ Good: Descriptive test names
test('should display error message when email is invalid', () => {
    // Test implementation
});

// ❌ Bad: Vague test names
test('email test', () => {
    // Test implementation
});

// ✅ Good: Test behavior, not implementation
test('shows welcome message after successful login', async () => {
    render(<LoginForm />);
    
    await userEvent.type(screen.getByLabelText(/email/i), 'user@example.com');
    await userEvent.type(screen.getByLabelText(/password/i), 'password123');
    await userEvent.click(screen.getByRole('button', { name: /login/i }));
    
    expect(screen.getByText(/welcome/i)).toBeInTheDocument();
});

// ❌ Bad: Testing implementation details
test('calls setState with correct value', () => {
    const wrapper = shallow(<Counter />);
    wrapper.instance().increment();
    expect(wrapper.state('count')).toBe(1);
});
```

### Test Data Management
```javascript
// Factory functions for test data
const createUser = (overrides = {}) => ({
    id: 1,
    name: 'John Doe',
    email: 'john@example.com',
    role: 'user',
    ...overrides
});

const createProduct = (overrides = {}) => ({
    id: 1,
    name: 'Test Product',
    price: 99.99,
    inStock: true,
    ...overrides
});

// Usage
test('displays out of stock message', () => {
    const product = createProduct({ inStock: false });
    render(<ProductCard product={product} />);
    
    expect(screen.getByText(/out of stock/i)).toBeInTheDocument();
});
```

### Performance Testing
```javascript
// Test component performance
test('renders large list efficiently', () => {
    const items = Array.from({ length: 1000 }, (_, i) => ({ id: i, name: `Item ${i}` }));
    
    const startTime = performance.now();
    render(<ItemList items={items} />);
    const endTime = performance.now();
    
    expect(endTime - startTime).toBeLessThan(100); // Should render in under 100ms
});

// Memory leak detection
test('cleans up event listeners', () => {
    const { unmount } = render(<ComponentWithListeners />);
    
    const initialListeners = getEventListeners(window).length;
    unmount();
    const finalListeners = getEventListeners(window).length;
    
    expect(finalListeners).toBe(initialListeners);
});
```

## 🔧 Quick Reference

### Common Test Patterns
```javascript
// Testing custom hooks
import { renderHook, act } from '@testing-library/react';

test('useCounter hook', () => {
    const { result } = renderHook(() => useCounter(0));
    
    expect(result.current.count).toBe(0);
    
    act(() => {
        result.current.increment();
    });
    
    expect(result.current.count).toBe(1);
});

// Testing error boundaries
test('error boundary catches errors', () => {
    const ThrowError = () => {
        throw new Error('Test error');
    };
    
    render(
        <ErrorBoundary>
            <ThrowError />
        </ErrorBoundary>
    );
    
    expect(screen.getByText(/something went wrong/i)).toBeInTheDocument();
});

// Testing with timers
test('debounced search', async () => {
    jest.useFakeTimers();
    
    render(<SearchInput onSearch={mockSearch} />);
    
    await userEvent.type(screen.getByRole('textbox'), 'test query');
    
    // Fast-forward time
    act(() => {
        jest.advanceTimersByTime(500);
    });
    
    expect(mockSearch).toHaveBeenCalledWith('test query');
    
    jest.useRealTimers();
});
```

### Test Commands
```bash
# Run all tests
npm test

# Run tests in watch mode
npm run test:watch

# Run tests with coverage
npm run test:coverage

# Run specific test file
npm test -- UserProfile.test.js

# Run tests matching pattern
npm test -- --testNamePattern="should render"

# Update snapshots
npm test -- --updateSnapshot

# Run tests in CI mode
npm test -- --ci --coverage --watchAll=false
```

---

## 🔗 Related Topics
- [[JavaScript]] - Core language features for testing
- [[React]] - Component testing strategies
- [[Node.js]] - Testing Node.js applications
- [[Best Practices]] - General development guidelines
- [[CI/CD & Deployment]] - Automated testing in pipelines 