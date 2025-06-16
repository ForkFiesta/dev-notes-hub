---
tags:
  - process
  - code-review
  - team
  - best-practices
  - collaboration
created: 2025-06-15
updated: 2025-06-15
---

# Code Review Guidelines

Comprehensive guide to effective code reviews that improve code quality, share knowledge, and build better teams.

## Overview

Code reviews are one of the most effective ways to catch bugs, share knowledge, and maintain code quality. This guide covers best practices for both reviewers and authors.

## Code Review Process

### 1. Pre-Review Checklist (Author)

**Before Submitting:**
- [ ] Code compiles without warnings
- [ ] All tests pass locally
- [ ] Linting rules are satisfied
- [ ] Self-review completed
- [ ] Meaningful commit messages
- [ ] PR description explains the change
- [ ] Screenshots/demos for UI changes

**PR Description Template:**
```markdown
## What
Brief description of what this PR does

## Why
Context and reasoning behind the change

## How
Technical approach and key decisions

## Testing
- [ ] Unit tests added/updated
- [ ] Manual testing completed
- [ ] Edge cases considered

## Screenshots/Demo
(For UI changes)

## Checklist
- [ ] Breaking changes documented
- [ ] Database migrations included
- [ ] Environment variables updated
```

### 2. Review Checklist (Reviewer)

**Functionality:**
- [ ] Code does what it's supposed to do
- [ ] Edge cases are handled
- [ ] Error handling is appropriate
- [ ] No obvious bugs or logic errors

**Code Quality:**
- [ ] Code is readable and well-structured
- [ ] Functions/classes have single responsibility
- [ ] Naming is clear and consistent
- [ ] No code duplication
- [ ] Appropriate abstractions

**Testing:**
- [ ] Adequate test coverage
- [ ] Tests are meaningful and not brittle
- [ ] Test names clearly describe what they test
- [ ] Mock usage is appropriate

**Security:**
- [ ] No sensitive data exposed
- [ ] Input validation present
- [ ] Authentication/authorization correct
- [ ] SQL injection prevention
- [ ] XSS prevention measures

**Performance:**
- [ ] No obvious performance issues
- [ ] Database queries are efficient
- [ ] Caching used appropriately
- [ ] Memory leaks avoided

## Review Categories

### 1. Critical Issues (Must Fix)

**Security Vulnerabilities:**
```javascript
// ❌ Bad: SQL injection vulnerability
const query = `SELECT * FROM users WHERE id = ${userId}`

// ✅ Good: Parameterized query
const query = 'SELECT * FROM users WHERE id = ?'
db.query(query, [userId])
```

**Logic Errors:**
```javascript
// ❌ Bad: Off-by-one error
for (let i = 0; i <= array.length; i++) {
  console.log(array[i]) // Will throw error on last iteration
}

// ✅ Good: Correct bounds
for (let i = 0; i < array.length; i++) {
  console.log(array[i])
}
```

**Resource Leaks:**
```javascript
// ❌ Bad: Memory leak
const intervals = []
function startTimer() {
  intervals.push(setInterval(() => {}, 1000))
  // Never cleared!
}

// ✅ Good: Proper cleanup
function startTimer() {
  const interval = setInterval(() => {}, 1000)
  return () => clearInterval(interval)
}
```

### 2. Major Issues (Should Fix)

**Poor Error Handling:**
```javascript
// ❌ Bad: Silent failure
try {
  await api.updateUser(user)
} catch (error) {
  // Silently ignored
}

// ✅ Good: Proper error handling
try {
  await api.updateUser(user)
} catch (error) {
  logger.error('Failed to update user', { error, userId: user.id })
  throw new UserUpdateError('Failed to update user', { cause: error })
}
```

**Performance Issues:**
```javascript
// ❌ Bad: N+1 query problem
const users = await User.findAll()
for (const user of users) {
  user.posts = await Post.findByUserId(user.id) // N queries
}

// ✅ Good: Single query with joins
const users = await User.findAll({
  include: [{ model: Post }]
})
```

### 3. Minor Issues (Nice to Fix)

**Code Style:**
```javascript
// ❌ Inconsistent naming
const user_name = getUserName()
const userAge = getUserAge()

// ✅ Consistent naming
const userName = getUserName()
const userAge = getUserAge()
```

**Readability:**
```javascript
// ❌ Hard to read
if (user && user.profile && user.profile.settings && user.profile.settings.notifications) {
  // ...
}

// ✅ More readable
const hasNotificationSettings = user?.profile?.settings?.notifications
if (hasNotificationSettings) {
  // ...
}
```

## Red Flags to Watch For

### 1. Code Smells

**Long Functions:**
```javascript
// ❌ Red flag: 100+ line function
function processOrder(order) {
  // Validation logic (20 lines)
  // Payment processing (30 lines)
  // Inventory updates (25 lines)
  // Email notifications (20 lines)
  // Logging and analytics (15 lines)
}

// ✅ Better: Extracted functions
function processOrder(order) {
  validateOrder(order)
  processPayment(order)
  updateInventory(order)
  sendNotifications(order)
  logOrderProcessing(order)
}
```

**Deep Nesting:**
```javascript
// ❌ Red flag: Deep nesting
if (user) {
  if (user.isActive) {
    if (user.hasPermission) {
      if (user.subscription) {
        if (user.subscription.isValid) {
          // Finally do something
        }
      }
    }
  }
}

// ✅ Better: Early returns
if (!user || !user.isActive) return
if (!user.hasPermission) return
if (!user.subscription?.isValid) return

// Do something
```

**Magic Numbers/Strings:**
```javascript
// ❌ Red flag: Magic numbers
setTimeout(callback, 86400000) // What is this?
if (status === 'PENDING_APPROVAL_STAGE_2') // Unclear

// ✅ Better: Named constants
const ONE_DAY_MS = 24 * 60 * 60 * 1000
const ORDER_STATUS = {
  PENDING_APPROVAL_STAGE_2: 'PENDING_APPROVAL_STAGE_2'
}

setTimeout(callback, ONE_DAY_MS)
if (status === ORDER_STATUS.PENDING_APPROVAL_STAGE_2)
```

### 2. Architecture Issues

**Tight Coupling:**
```javascript
// ❌ Red flag: Tight coupling
class OrderService {
  processOrder(order) {
    // Direct database access
    const user = database.users.findById(order.userId)
    
    // Direct email service usage
    emailService.send(user.email, 'Order confirmed')
    
    // Direct payment processor
    paymentProcessor.charge(order.amount)
  }
}

// ✅ Better: Dependency injection
class OrderService {
  constructor(userRepository, emailService, paymentService) {
    this.userRepository = userRepository
    this.emailService = emailService
    this.paymentService = paymentService
  }
  
  async processOrder(order) {
    const user = await this.userRepository.findById(order.userId)
    await this.emailService.sendOrderConfirmation(user.email, order)
    await this.paymentService.processPayment(order.amount)
  }
}
```

**Missing Abstractions:**
```javascript
// ❌ Red flag: Repeated patterns
function getUserFromDatabase(id) {
  return fetch(`/api/users/${id}`).then(r => r.json())
}

function getProductFromDatabase(id) {
  return fetch(`/api/products/${id}`).then(r => r.json())
}

function getOrderFromDatabase(id) {
  return fetch(`/api/orders/${id}`).then(r => r.json())
}

// ✅ Better: Generic abstraction
class ApiClient {
  async get(endpoint) {
    const response = await fetch(`/api/${endpoint}`)
    return response.json()
  }
}

const api = new ApiClient()
const getUser = (id) => api.get(`users/${id}`)
const getProduct = (id) => api.get(`products/${id}`)
const getOrder = (id) => api.get(`orders/${id}`)
```

## Effective Feedback

### 1. Constructive Comments

**Good Feedback Examples:**
```
✅ "Consider extracting this logic into a separate function for better testability"
✅ "This could cause a race condition if multiple users access simultaneously. Consider adding a lock or using atomic operations"
✅ "Great use of the builder pattern here! This makes the code much more readable"
✅ "Minor: This variable name could be more descriptive. Maybe 'userAuthToken' instead of 'token'?"
```

**Poor Feedback Examples:**
```
❌ "This is wrong"
❌ "Bad code"
❌ "I don't like this"
❌ "This won't work"
```

### 2. Feedback Categories

**Use Prefixes for Clarity:**
- `Critical:` Must be fixed before merge
- `Major:` Should be addressed
- `Minor:` Nice to have
- `Nit:` Style/preference issue
- `Question:` Seeking clarification
- `Suggestion:` Alternative approach
- `Praise:` Positive feedback

**Examples:**
```
Critical: This SQL query is vulnerable to injection attacks
Major: This function is doing too many things and should be split
Minor: Consider using const instead of let here
Nit: Extra whitespace on line 42
Question: Why did you choose this approach over X?
Suggestion: You might consider using the Strategy pattern here
Praise: Excellent error handling throughout this module!
```

### 3. Tone and Language

**Positive Framing:**
```
✅ "What do you think about extracting this into a helper function?"
✅ "I wonder if we could simplify this by..."
✅ "This is a great start! One thing to consider..."

❌ "You should extract this"
❌ "This is too complicated"
❌ "Wrong approach"
```

**Be Specific:**
```
✅ "This function has 15 parameters, which makes it hard to use. Consider using an options object or breaking it into smaller functions"

❌ "Too many parameters"
```

## Review Tools and Automation

### 1. Automated Checks

**Pre-commit Hooks:**
```json
// package.json
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged",
      "pre-push": "npm test"
    }
  },
  "lint-staged": {
    "*.{js,ts}": ["eslint --fix", "prettier --write"],
    "*.{json,md}": ["prettier --write"]
  }
}
```

**CI/CD Pipeline:**
```yaml
# .github/workflows/pr-checks.yml
name: PR Checks
on: [pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Install dependencies
        run: npm ci
      - name: Run linting
        run: npm run lint
      - name: Run tests
        run: npm test
      - name: Check coverage
        run: npm run test:coverage
      - name: Security audit
        run: npm audit
```

### 2. Review Tools

**GitHub/GitLab Features:**
- Required reviewers
- Status checks
- Draft PRs
- Review templates
- Code owners

**Third-party Tools:**
- SonarQube for code quality
- CodeClimate for maintainability
- Snyk for security scanning
- Codecov for coverage tracking

## Team Practices

### 1. Review Assignment

**Strategies:**
- Round-robin assignment
- Expertise-based assignment
- Pair programming follow-up
- Junior/senior pairing

**Code Owners:**
```
# CODEOWNERS file
# Global owners
* @team-leads

# Frontend
/src/components/ @frontend-team
/src/styles/ @frontend-team

# Backend
/src/api/ @backend-team
/src/database/ @backend-team @dba-team

# Infrastructure
/docker/ @devops-team
/.github/ @devops-team
```

### 2. Review Standards

**Team Agreement:**
- Maximum PR size (e.g., 400 lines)
- Response time expectations (e.g., 24 hours)
- Number of required approvals
- When to request re-review
- Escalation process for disagreements

**Review Metrics:**
- Time to first review
- Time to merge
- Number of review cycles
- Defect escape rate

### 3. Knowledge Sharing

**Learning Opportunities:**
- Junior developers review senior code
- Cross-team reviews for knowledge sharing
- Architecture decision documentation
- Review retrospectives

**Documentation:**
```markdown
## Architecture Decision Record (ADR)

### Status
Accepted

### Context
We need to choose a state management solution for our React app

### Decision
We will use Redux Toolkit with RTK Query

### Consequences
- Pros: Standardized patterns, good DevTools, handles caching
- Cons: Learning curve, boilerplate for simple cases
```

## Common Pitfalls

### 1. Review Fatigue

**Symptoms:**
- Rubber-stamp approvals
- Delayed reviews
- Superficial feedback
- Avoiding difficult reviews

**Solutions:**
- Limit PR size
- Rotate reviewers
- Use automated checks
- Prioritize critical changes

### 2. Bikeshedding

**Example:**
```javascript
// Don't spend 30 minutes debating this:
const userCount = users.length  // vs users.size vs numberOfUsers
```

**Solutions:**
- Use automated formatting
- Establish style guides
- Focus on logic over style
- Time-box style discussions

### 3. Personal Attacks

**Avoid:**
- "You always do this wrong"
- "This is terrible code"
- "You don't understand X"

**Instead:**
- Focus on the code, not the person
- Explain the reasoning
- Suggest improvements
- Ask questions

## Measuring Success

### Key Metrics

**Quality Metrics:**
- Defect density
- Time to fix bugs
- Customer satisfaction
- Code coverage

**Process Metrics:**
- Review participation rate
- Average review time
- Review thoroughness score
- Knowledge sharing index

**Team Metrics:**
- Developer satisfaction
- Learning velocity
- Code ownership distribution
- Review quality feedback

## Related Notes

- [[Git Workflow]] - Git branching and workflow strategies
- [[Testing]] - Testing strategies and best practices
- [[Documentation]] - Code documentation practices
- [[Team Collaboration]] - General team collaboration practices
- [[Refactoring]] - Code refactoring techniques

## Resources

- [Google's Code Review Guidelines](https://google.github.io/eng-practices/review/)
- [Best Practices for Code Review](https://smartbear.com/learn/code-review/best-practices-for-peer-code-review/)
- [The Art of Readable Code](https://www.oreilly.com/library/view/the-art-of/9781449318482/)
- [Clean Code](https://www.oreilly.com/library/view/clean-code-a/9780136083238/) 