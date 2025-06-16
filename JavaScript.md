# JavaScript (ES6+) - Modern JavaScript

> **Quick Summary**: JavaScript brings interactivity to web pages. Master ES6+ features, DOM manipulation, and asynchronous programming for modern web development.

## 🚀 ES6+ Features

### Variables and Constants
```javascript
// let and const (block-scoped)
let userName = 'John';
const API_URL = 'https://api.example.com';

// var (function-scoped, avoid in modern JS)
var oldStyle = 'avoid this';

// Temporal dead zone
console.log(userName); // ReferenceError
let userName = 'John';
```

### Arrow Functions
```javascript
// Traditional function
function add(a, b) {
    return a + b;
}

// Arrow function
const add = (a, b) => a + b;

// Multiple lines
const processUser = (user) => {
    const processed = user.name.toUpperCase();
    return { ...user, name: processed };
};

// No parameters
const getCurrentTime = () => new Date();

// Single parameter (parentheses optional)
const double = x => x * 2;

// Arrow functions don't have their own 'this'
const obj = {
    name: 'Object',
    regularFunction() {
        console.log(this.name); // 'Object'
        
        const arrowFunction = () => {
            console.log(this.name); // 'Object' (inherits from parent)
        };
        arrowFunction();
    }
};
```

### Template Literals
```javascript
const name = 'John';
const age = 30;

// String interpolation
const message = `Hello, my name is ${name} and I'm ${age} years old.`;

// Multi-line strings
const html = `
    <div class="user-card">
        <h2>${name}</h2>
        <p>Age: ${age}</p>
    </div>
`;

// Expression evaluation
const price = 19.99;
const tax = 0.08;
const total = `Total: $${(price * (1 + tax)).toFixed(2)}`;
```

### Destructuring
```javascript
// Array destructuring
const colors = ['red', 'green', 'blue'];
const [primary, secondary, tertiary] = colors;
const [first, , third] = colors; // Skip elements

// Object destructuring
const user = { name: 'John', age: 30, email: 'john@example.com' };
const { name, age } = user;
const { name: userName, age: userAge } = user; // Rename variables

// Default values
const { name = 'Anonymous', country = 'Unknown' } = user;

// Nested destructuring
const response = {
    data: {
        users: [{ name: 'John', age: 30 }]
    }
};
const { data: { users: [firstUser] } } = response;

// Function parameters
const greetUser = ({ name, age }) => {
    console.log(`Hello ${name}, you are ${age} years old`);
};
```

### Spread and Rest Operators
```javascript
// Spread operator (...)
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
const combined = [...arr1, ...arr2]; // [1, 2, 3, 4, 5, 6]

// Object spread
const user = { name: 'John', age: 30 };
const updatedUser = { ...user, age: 31, city: 'New York' };

// Rest parameters
const sum = (...numbers) => {
    return numbers.reduce((total, num) => total + num, 0);
};
sum(1, 2, 3, 4); // 10

// Rest in destructuring
const [first, second, ...rest] = [1, 2, 3, 4, 5];
// first: 1, second: 2, rest: [3, 4, 5]
```

## 🏗️ Data Types and Structures

### Primitive Types
```javascript
// String
let message = "Hello World";
let template = `Template literal`;

// Number
let integer = 42;
let float = 3.14;
let scientific = 2.5e6; // 2500000

// Boolean
let isActive = true;
let isComplete = false;

// Undefined and null
let undefinedVar; // undefined
let nullVar = null;

// Symbol (ES6)
const uniqueId = Symbol('id');
const anotherUniqueId = Symbol('id');
console.log(uniqueId === anotherUniqueId); // false

// BigInt (ES2020)
const bigNumber = 123456789012345678901234567890n;
```

### Arrays
```javascript
// Array creation
const fruits = ['apple', 'banana', 'orange'];
const numbers = new Array(1, 2, 3, 4, 5);

// Array methods
const numbers = [1, 2, 3, 4, 5];

// Map - transform each element
const doubled = numbers.map(num => num * 2); // [2, 4, 6, 8, 10]

// Filter - select elements that match condition
const evens = numbers.filter(num => num % 2 === 0); // [2, 4]

// Reduce - accumulate values
const sum = numbers.reduce((total, num) => total + num, 0); // 15

// Find - get first matching element
const found = numbers.find(num => num > 3); // 4

// Some/Every - test conditions
const hasEven = numbers.some(num => num % 2 === 0); // true
const allPositive = numbers.every(num => num > 0); // true

// ForEach - iterate without return
numbers.forEach((num, index) => {
    console.log(`Index ${index}: ${num}`);
});

// Modern array methods
const users = [
    { name: 'John', age: 30 },
    { name: 'Jane', age: 25 },
    { name: 'Bob', age: 35 }
];

// Chain methods
const youngUserNames = users
    .filter(user => user.age < 30)
    .map(user => user.name)
    .sort();
```

### Objects
```javascript
// Object creation
const user = {
    name: 'John',
    age: 30,
    greet() {
        return `Hello, I'm ${this.name}`;
    }
};

// Property access
console.log(user.name); // Dot notation
console.log(user['age']); // Bracket notation

// Dynamic property names
const propertyName = 'email';
const user2 = {
    name: 'Jane',
    [propertyName]: 'jane@example.com' // Computed property
};

// Object methods
const keys = Object.keys(user); // ['name', 'age', 'greet']
const values = Object.values(user); // ['John', 30, function]
const entries = Object.entries(user); // [['name', 'John'], ['age', 30], ...]

// Object.assign and spread
const defaults = { theme: 'light', language: 'en' };
const userPrefs = { theme: 'dark' };
const settings = Object.assign({}, defaults, userPrefs);
// or
const settings2 = { ...defaults, ...userPrefs };
```

## 🌐 DOM Manipulation

### Selecting Elements
```javascript
// Single element selectors
const element = document.getElementById('myId');
const element2 = document.querySelector('.my-class');
const element3 = document.querySelector('#myId');

// Multiple element selectors
const elements = document.getElementsByClassName('my-class');
const elements2 = document.getElementsByTagName('div');
const elements3 = document.querySelectorAll('.my-class');

// Modern approach - use querySelector/querySelectorAll
const button = document.querySelector('button[type="submit"]');
const allButtons = document.querySelectorAll('button');
```

### Modifying Elements
```javascript
const element = document.querySelector('#myElement');

// Content manipulation
element.textContent = 'New text content';
element.innerHTML = '<strong>Bold text</strong>';

// Attribute manipulation
element.setAttribute('data-id', '123');
element.getAttribute('data-id');
element.removeAttribute('data-id');
element.hasAttribute('data-id');

// Class manipulation
element.classList.add('new-class');
element.classList.remove('old-class');
element.classList.toggle('active');
element.classList.contains('active');

// Style manipulation
element.style.color = 'red';
element.style.backgroundColor = 'blue';
element.style.cssText = 'color: red; background: blue;';

// Data attributes
element.dataset.userId = '123';
console.log(element.dataset.userId);
```

### Creating and Removing Elements
```javascript
// Create elements
const newDiv = document.createElement('div');
newDiv.textContent = 'Hello World';
newDiv.className = 'my-class';

// Add to DOM
const container = document.querySelector('#container');
container.appendChild(newDiv);
container.insertBefore(newDiv, container.firstChild);

// Modern insertion methods
container.prepend(newDiv); // Add as first child
container.append(newDiv); // Add as last child
container.before(newDiv); // Add before container
container.after(newDiv); // Add after container

// Remove elements
newDiv.remove(); // Modern way
container.removeChild(newDiv); // Older way

// Clone elements
const clone = newDiv.cloneNode(true); // true for deep clone
```

### Event Handling
```javascript
const button = document.querySelector('#myButton');

// Add event listeners
button.addEventListener('click', function(event) {
    console.log('Button clicked!');
    event.preventDefault(); // Prevent default behavior
    event.stopPropagation(); // Stop event bubbling
});

// Arrow function event handler
button.addEventListener('click', (e) => {
    console.log('Clicked:', e.target);
});

// Remove event listeners
const handleClick = (e) => console.log('Clicked');
button.addEventListener('click', handleClick);
button.removeEventListener('click', handleClick);

// Event delegation
document.addEventListener('click', (e) => {
    if (e.target.matches('.dynamic-button')) {
        console.log('Dynamic button clicked');
    }
});

// Common events
element.addEventListener('click', handler);
element.addEventListener('mouseover', handler);
element.addEventListener('mouseout', handler);
element.addEventListener('keydown', handler);
element.addEventListener('submit', handler);
element.addEventListener('change', handler);
element.addEventListener('input', handler);
```

## ⚡ Asynchronous JavaScript

### Promises
```javascript
// Creating a promise
const fetchData = () => {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            const success = Math.random() > 0.5;
            if (success) {
                resolve({ data: 'Success!' });
            } else {
                reject(new Error('Failed to fetch data'));
            }
        }, 1000);
    });
};

// Using promises
fetchData()
    .then(result => {
        console.log('Success:', result);
        return result.data;
    })
    .then(data => {
        console.log('Processed:', data);
    })
    .catch(error => {
        console.error('Error:', error.message);
    })
    .finally(() => {
        console.log('Cleanup');
    });

// Promise utilities
const promise1 = Promise.resolve('First');
const promise2 = Promise.resolve('Second');
const promise3 = Promise.resolve('Third');

// Wait for all promises
Promise.all([promise1, promise2, promise3])
    .then(results => console.log(results)); // ['First', 'Second', 'Third']

// Wait for first resolved promise
Promise.race([promise1, promise2, promise3])
    .then(result => console.log(result)); // 'First'

// Wait for all promises (including rejected ones)
Promise.allSettled([promise1, promise2, promise3])
    .then(results => console.log(results));
```

### Async/Await
```javascript
// Async function
async function fetchUserData(userId) {
    try {
        const response = await fetch(`/api/users/${userId}`);
        
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        const userData = await response.json();
        return userData;
    } catch (error) {
        console.error('Error fetching user:', error);
        throw error; // Re-throw if needed
    }
}

// Using async function
async function displayUser(userId) {
    try {
        const user = await fetchUserData(userId);
        console.log('User:', user);
    } catch (error) {
        console.log('Failed to display user');
    }
}

// Async arrow function
const fetchAndProcess = async (url) => {
    const response = await fetch(url);
    const data = await response.json();
    return data.map(item => item.name);
};

// Parallel async operations
async function fetchMultipleUsers() {
    try {
        // Sequential (slower)
        const user1 = await fetchUserData(1);
        const user2 = await fetchUserData(2);
        
        // Parallel (faster)
        const [user3, user4] = await Promise.all([
            fetchUserData(3),
            fetchUserData(4)
        ]);
        
        return [user1, user2, user3, user4];
    } catch (error) {
        console.error('Error fetching users:', error);
    }
}
```

### Fetch API
```javascript
// Basic GET request
fetch('/api/data')
    .then(response => response.json())
    .then(data => console.log(data))
    .catch(error => console.error('Error:', error));

// POST request with JSON
fetch('/api/users', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer token123'
    },
    body: JSON.stringify({
        name: 'John Doe',
        email: 'john@example.com'
    })
})
.then(response => {
    if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
    }
    return response.json();
})
.then(data => console.log('Success:', data));

// Async/await with fetch
async function createUser(userData) {
    try {
        const response = await fetch('/api/users', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify(userData)
        });
        
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        const newUser = await response.json();
        return newUser;
    } catch (error) {
        console.error('Failed to create user:', error);
        throw error;
    }
}
```

## 📦 Modules

### ES6 Modules
```javascript
// math.js - Named exports
export const PI = 3.14159;
export const add = (a, b) => a + b;
export const multiply = (a, b) => a * b;

// Default export
export default function subtract(a, b) {
    return a - b;
}

// utils.js - Multiple exports
export { formatDate, formatCurrency } from './formatters.js';
export * as validators from './validators.js';

// main.js - Importing
import subtract, { PI, add, multiply } from './math.js';
import { formatDate } from './utils.js';
import * as math from './math.js';

// Dynamic imports
async function loadModule() {
    const { default: subtract, add } = await import('./math.js');
    console.log(add(5, 3));
}

// Conditional imports
if (condition) {
    import('./feature.js').then(module => {
        module.initialize();
    });
}
```

## 🛠️ Common Patterns

### Error Handling
```javascript
// Try-catch with async/await
async function safeApiCall() {
    try {
        const response = await fetch('/api/data');
        const data = await response.json();
        return data;
    } catch (error) {
        console.error('API call failed:', error);
        return null; // or default value
    }
}

// Custom error classes
class ValidationError extends Error {
    constructor(message, field) {
        super(message);
        this.name = 'ValidationError';
        this.field = field;
    }
}

function validateEmail(email) {
    if (!email.includes('@')) {
        throw new ValidationError('Invalid email format', 'email');
    }
}

// Error boundaries for promises
const handleErrors = (promise) => {
    return promise.catch(error => {
        console.error('Promise rejected:', error);
        return null; // or default value
    });
};
```

### Debouncing and Throttling
```javascript
// Debounce - delay execution until after calls have stopped
function debounce(func, delay) {
    let timeoutId;
    return function (...args) {
        clearTimeout(timeoutId);
        timeoutId = setTimeout(() => func.apply(this, args), delay);
    };
}

// Usage
const searchInput = document.querySelector('#search');
const debouncedSearch = debounce((query) => {
    console.log('Searching for:', query);
}, 300);

searchInput.addEventListener('input', (e) => {
    debouncedSearch(e.target.value);
});

// Throttle - limit execution frequency
function throttle(func, limit) {
    let inThrottle;
    return function (...args) {
        if (!inThrottle) {
            func.apply(this, args);
            inThrottle = true;
            setTimeout(() => inThrottle = false, limit);
        }
    };
}

// Usage
const throttledScroll = throttle(() => {
    console.log('Scroll event');
}, 100);

window.addEventListener('scroll', throttledScroll);
```

## ✅ Quick Checklist

- [ ] Use `const` and `let` instead of `var`
- [ ] Prefer arrow functions for callbacks
- [ ] Use template literals for string interpolation
- [ ] Implement proper error handling with try-catch
- [ ] Use async/await for cleaner asynchronous code
- [ ] Leverage array methods (map, filter, reduce)
- [ ] Use destructuring for cleaner code
- [ ] Implement event delegation for dynamic content
- [ ] Use modules to organize code
- [ ] Handle edge cases and validate inputs

## 🔗 Related Topics
- [[HTML]] - Structure for JavaScript to manipulate
- [[React]] - JavaScript library for building UIs
- [[Node.js]] - JavaScript runtime for server-side development

---
*Remember: Modern JavaScript is powerful and expressive. Use ES6+ features to write cleaner, more maintainable code!* 