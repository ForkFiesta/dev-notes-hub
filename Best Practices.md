---
tags: [best practices, development, coding, architecture, patterns]
created: 2025-06-15
updated: 2025-06-15
---

# Best Practices - Development Guidelines

> **Quick Summary**: Essential best practices for writing maintainable, scalable, and high-quality code across all technologies.

## 🏗️ General Development Principles

### Code Quality
- **Write self-documenting code** - Use clear, descriptive names
- **Keep functions small** - Single responsibility principle
- **Avoid deep nesting** - Use early returns and guard clauses
- **Be consistent** - Follow established patterns and conventions
- **Comment why, not what** - Explain business logic and decisions

### Project Organization
```
project/
├── src/
│   ├── components/
│   ├── utils/
│   ├── hooks/
│   ├── services/
│   └── styles/
├── public/
├── tests/
├── docs/
├── .env.example
├── .gitignore
├── README.md
└── package.json
```

### Version Control
- **Commit early and often** - Small, focused commits
- **Write meaningful commit messages** - Use conventional commits
- **Use feature branches** - Keep main branch stable
- **Review code** - Use pull requests for all changes
- **Tag releases** - Use semantic versioning

## 🌐 HTML Best Practices

### Semantic Structure
```html
<!-- Good: Semantic and accessible -->
<article>
    <header>
        <h1>Article Title</h1>
        <time datetime="2023-12-01">December 1, 2023</time>
    </header>
    <main>
        <p>Article content...</p>
    </main>
    <footer>
        <p>Author: John Doe</p>
    </footer>
</article>

<!-- Avoid: Non-semantic divs -->
<div class="article">
    <div class="title">Article Title</div>
    <div class="content">Article content...</div>
</div>
```

### Accessibility
- **Always include alt text** for images
- **Use proper heading hierarchy** (h1 → h2 → h3)
- **Label form inputs** with `<label>` elements
- **Provide keyboard navigation** for interactive elements
- **Use ARIA attributes** when semantic HTML isn't enough

### Performance
- **Optimize images** - Use appropriate formats and sizes
- **Minimize HTTP requests** - Combine files when possible
- **Use lazy loading** for images below the fold
- **Validate HTML** - Use W3C validator

## 🎨 CSS Best Practices

### Architecture
```css
/* Use BEM methodology for naming */
.card { }
.card__header { }
.card__title { }
.card--featured { }

/* Organize CSS logically */
/* 1. Base styles */
/* 2. Layout */
/* 3. Components */
/* 4. Utilities */
/* 5. Media queries */
```

### Performance
- **Minimize CSS** - Remove unused styles
- **Use efficient selectors** - Avoid overly specific selectors
- **Leverage CSS custom properties** - For theming and consistency
- **Optimize for critical rendering path** - Inline critical CSS

### Responsive Design
```css
/* Mobile-first approach */
.component {
    /* Mobile styles */
    font-size: 14px;
}

@media (min-width: 768px) {
    .component {
        /* Tablet styles */
        font-size: 16px;
    }
}

@media (min-width: 1024px) {
    .component {
        /* Desktop styles */
        font-size: 18px;
    }
}
```

### Maintainability
- **Use consistent naming conventions**
- **Group related properties**
- **Avoid !important** - Use specificity instead
- **Comment complex calculations** and browser hacks

## ⚡ JavaScript Best Practices

### Code Quality
```javascript
// Good: Clear, descriptive names
function calculateTotalPrice(items, taxRate) {
    return items.reduce((total, item) => {
        return total + (item.price * item.quantity);
    }, 0) * (1 + taxRate);
}

// Avoid: Unclear names and logic
function calc(arr, rate) {
    let t = 0;
    for (let i = 0; i < arr.length; i++) {
        t += arr[i].p * arr[i].q;
    }
    return t * (1 + rate);
}
```

### Error Handling
```javascript
// Always handle errors gracefully
async function fetchUserData(userId) {
    try {
        const response = await fetch(`/api/users/${userId}`);
        
        if (!response.ok) {
            throw new Error(`Failed to fetch user: ${response.status}`);
        }
        
        return await response.json();
    } catch (error) {
        console.error('Error fetching user data:', error);
        // Return default value or re-throw based on context
        return null;
    }
}
```

### Performance
- **Use const and let** instead of var
- **Prefer array methods** over for loops
- **Debounce expensive operations** like API calls
- **Use async/await** for cleaner asynchronous code
- **Avoid memory leaks** - Clean up event listeners and timers

### Security
```javascript
// Validate and sanitize user input
function sanitizeInput(input) {
    return input.trim().replace(/[<>]/g, '');
}

// Use environment variables for sensitive data
const API_KEY = process.env.API_KEY;

// Validate data on both client and server
function validateEmail(email) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email);
}
```

## ⚛️ React Best Practices

### Component Design
```jsx
// Good: Small, focused component
function UserCard({ user, onEdit, onDelete }) {
    return (
        <div className="user-card">
            <img src={user.avatar} alt={`${user.name}'s avatar`} />
            <h3>{user.name}</h3>
            <p>{user.email}</p>
            <div className="user-actions">
                <button onClick={() => onEdit(user.id)}>Edit</button>
                <button onClick={() => onDelete(user.id)}>Delete</button>
            </div>
        </div>
    );
}

// Avoid: Large, multi-purpose component
function UserManagement() {
    // 200+ lines of mixed concerns
}
```

### State Management
```jsx
// Good: Lift state up when needed
function App() {
    const [users, setUsers] = useState([]);
    
    return (
        <div>
            <UserList users={users} />
            <UserForm onAddUser={(user) => setUsers([...users, user])} />
        </div>
    );
}

// Use useReducer for complex state
function useUserManager() {
    const [state, dispatch] = useReducer(userReducer, initialState);
    
    const addUser = (user) => dispatch({ type: 'ADD_USER', payload: user });
    const deleteUser = (id) => dispatch({ type: 'DELETE_USER', payload: id });
    
    return { users: state.users, addUser, deleteUser };
}
```

### Performance Optimization
```jsx
// Memoize expensive calculations
const expensiveValue = useMemo(() => {
    return items.reduce((sum, item) => sum + item.value, 0);
}, [items]);

// Memoize callback functions
const handleClick = useCallback((id) => {
    onItemClick(id);
}, [onItemClick]);

// Memoize components
const ExpensiveComponent = React.memo(({ data }) => {
    return <div>{/* Expensive rendering */}</div>;
});
```

### Hooks Best Practices
- **Use custom hooks** for reusable logic
- **Follow hooks rules** - Only call at top level
- **Optimize dependencies** in useEffect
- **Clean up effects** to prevent memory leaks

## 🎨 Tailwind CSS Best Practices

### Component Organization
```jsx
// Good: Extract component classes
const buttonClasses = {
    base: "px-4 py-2 rounded font-medium transition-colors",
    primary: "bg-blue-500 text-white hover:bg-blue-600",
    secondary: "bg-gray-200 text-gray-800 hover:bg-gray-300"
};

function Button({ variant = 'primary', children, ...props }) {
    const classes = `${buttonClasses.base} ${buttonClasses[variant]}`;
    return <button className={classes} {...props}>{children}</button>;
}
```

### Responsive Design
```jsx
// Use mobile-first approach
<div className="
    grid grid-cols-1 gap-4
    sm:grid-cols-2 sm:gap-6
    lg:grid-cols-3 lg:gap-8
">
    {/* Grid items */}
</div>
```

### Customization
```javascript
// tailwind.config.js
module.exports = {
    theme: {
        extend: {
            colors: {
                brand: {
                    50: '#eff6ff',
                    500: '#3b82f6',
                    900: '#1e3a8a'
                }
            },
            spacing: {
                '18': '4.5rem',
                '88': '22rem'
            }
        }
    }
}
```

## 📦 Git Best Practices

### Commit Messages
```bash
# Good: Clear, descriptive commits
git commit -m "feat: add user authentication with JWT"
git commit -m "fix: resolve memory leak in image carousel"
git commit -m "docs: update API documentation for v2.0"

# Conventional Commits format
# type(scope): description
# 
# Types: feat, fix, docs, style, refactor, test, chore
```

### Branching Strategy
```bash
# Feature branch workflow
git checkout main
git pull origin main
git checkout -b feature/user-profile
# Work on feature
git add .
git commit -m "feat: implement user profile page"
git push -u origin feature/user-profile
# Create pull request
```

### Repository Management
- **Use .gitignore** - Exclude unnecessary files
- **Keep commits atomic** - One logical change per commit
- **Write good PR descriptions** - Explain what and why
- **Review code thoroughly** - Check for bugs and style
- **Squash merge** feature branches to keep history clean

## 🟢 Node.js Best Practices

### Project Structure
```
api/
├── src/
│   ├── controllers/
│   ├── middleware/
│   ├── models/
│   ├── routes/
│   ├── services/
│   └── utils/
├── tests/
├── config/
├── .env.example
└── server.js
```

### Error Handling
```javascript
// Global error handler
app.use((err, req, res, next) => {
    console.error(err.stack);
    
    if (err.name === 'ValidationError') {
        return res.status(400).json({ error: err.message });
    }
    
    if (err.name === 'UnauthorizedError') {
        return res.status(401).json({ error: 'Unauthorized' });
    }
    
    res.status(500).json({ 
        error: 'Internal server error',
        ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
    });
});

// Graceful shutdown
process.on('SIGTERM', () => {
    console.log('SIGTERM received. Shutting down gracefully...');
    server.close(() => {
        console.log('Process terminated');
    });
});
```

### Security
```javascript
// Use security middleware
app.use(helmet()); // Security headers
app.use(cors({ origin: process.env.ALLOWED_ORIGINS })); // CORS
app.use(express.json({ limit: '10mb' })); // Limit payload size

// Validate input
const { body, validationResult } = require('express-validator');

app.post('/users', [
    body('email').isEmail().normalizeEmail(),
    body('password').isLength({ min: 8 })
], (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
        return res.status(400).json({ errors: errors.array() });
    }
    // Process valid data
});
```

## 📦 Package Management Best Practices

### Dependencies
```json
{
  "engines": {
    "node": ">=16.0.0",
    "npm": ">=8.0.0"
  },
  "scripts": {
    "preinstall": "npx check-node-version --node $(cat .nvmrc)",
    "start": "node server.js",
    "dev": "nodemon server.js",
    "test": "jest",
    "lint": "eslint .",
    "format": "prettier --write ."
  }
}
```

### Security
- **Audit dependencies regularly** - `npm audit`
- **Use exact versions** for critical dependencies
- **Keep dependencies updated** - Use tools like Dependabot
- **Review package contents** before installing

## 🧪 Testing Best Practices

### Test Structure
```javascript
// Good: Clear test structure
describe('UserService', () => {
    describe('createUser', () => {
        it('should create a user with valid data', async () => {
            // Arrange
            const userData = { name: 'John', email: 'john@example.com' };
            
            // Act
            const user = await UserService.createUser(userData);
            
            // Assert
            expect(user).toHaveProperty('id');
            expect(user.name).toBe('John');
            expect(user.email).toBe('john@example.com');
        });
        
        it('should throw error with invalid email', async () => {
            // Arrange
            const userData = { name: 'John', email: 'invalid-email' };
            
            // Act & Assert
            await expect(UserService.createUser(userData))
                .rejects.toThrow('Invalid email format');
        });
    });
});
```

### Test Coverage
- **Aim for 80%+ coverage** but focus on critical paths
- **Test edge cases** and error conditions
- **Use integration tests** for API endpoints
- **Mock external dependencies** in unit tests

## 🚀 Performance Best Practices

### Frontend Performance
- **Optimize images** - Use WebP, lazy loading
- **Minimize bundle size** - Code splitting, tree shaking
- **Use CDN** for static assets
- **Implement caching** strategies
- **Monitor Core Web Vitals**

### Backend Performance
- **Use connection pooling** for databases
- **Implement caching** (Redis, Memcached)
- **Optimize database queries** - Use indexes
- **Use compression** for responses
- **Monitor application metrics**

## 🔒 Security Best Practices

### Authentication & Authorization
```javascript
// Use secure session configuration
app.use(session({
    secret: process.env.SESSION_SECRET,
    resave: false,
    saveUninitialized: false,
    cookie: {
        secure: process.env.NODE_ENV === 'production',
        httpOnly: true,
        maxAge: 24 * 60 * 60 * 1000 // 24 hours
    }
}));

// Implement rate limiting
const rateLimit = require('express-rate-limit');
const limiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100 // limit each IP to 100 requests per windowMs
});
app.use('/api/', limiter);
```

### Data Protection
- **Validate all inputs** on both client and server
- **Use HTTPS** in production
- **Sanitize user data** before storing
- **Implement proper error handling** - Don't expose sensitive info
- **Use environment variables** for secrets

## ✅ Development Workflow Checklist

### Before Starting
- [ ] Set up development environment
- [ ] Configure linting and formatting
- [ ] Set up version control
- [ ] Create project documentation
- [ ] Plan architecture and structure

### During Development
- [ ] Write tests for new features
- [ ] Follow coding standards
- [ ] Commit changes frequently
- [ ] Review code before merging
- [ ] Update documentation

### Before Deployment
- [ ] Run all tests
- [ ] Check for security vulnerabilities
- [ ] Optimize for production
- [ ] Test in staging environment
- [ ] Update version numbers

### After Deployment
- [ ] Monitor application performance
- [ ] Check error logs
- [ ] Verify functionality
- [ ] Update team on changes
- [ ] Plan next iteration

## 🔗 Related Topics
- [[Code Snippets]] - Reusable code patterns
- [[Troubleshooting]] - Common issues and solutions
- [[Cheat Sheets]] - Quick reference guides

---
*Following these best practices will help you write better code, avoid common pitfalls, and build maintainable applications!* 