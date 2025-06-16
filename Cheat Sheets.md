---
tags: [cheat sheets, quick reference, commands, patterns]
created: 2025-06-15
updated: 2025-06-15
---

# Cheat Sheets - Quick Command References

> **Quick Summary**: Fast reference for common commands and patterns across all technologies in this vault.

## 🌐 HTML Quick Reference

### Essential Elements
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Page Title</title>
</head>
<body>
    <header><nav></nav></header>
    <main><article></article></main>
    <footer></footer>
</body>
</html>
```

### Form Elements
```html
<form action="/submit" method="POST">
    <input type="text" name="name" required>
    <input type="email" name="email" required>
    <select name="country">
        <option value="us">United States</option>
    </select>
    <textarea name="message"></textarea>
    <button type="submit">Submit</button>
</form>
```

## 🎨 CSS Quick Reference

### Flexbox
```css
.container {
    display: flex;
    justify-content: center;    /* horizontal alignment */
    align-items: center;        /* vertical alignment */
    flex-direction: row;        /* row | column */
    flex-wrap: wrap;           /* wrap | nowrap */
    gap: 20px;                 /* space between items */
}

.item {
    flex: 1;                   /* grow to fill space */
    flex-shrink: 0;           /* don't shrink */
}
```

### Grid
```css
.grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    grid-template-rows: auto;
    gap: 20px;
}

.item {
    grid-column: 1 / 3;       /* span columns */
    grid-row: 2 / 4;          /* span rows */
}
```

### Media Queries
```css
/* Mobile first */
.element { font-size: 14px; }

@media (min-width: 768px) {   /* Tablet */
    .element { font-size: 16px; }
}

@media (min-width: 1024px) {  /* Desktop */
    .element { font-size: 18px; }
}
```

## ⚡ JavaScript Quick Reference

### Array Methods
```javascript
const arr = [1, 2, 3, 4, 5];

arr.map(x => x * 2)           // Transform: [2, 4, 6, 8, 10]
arr.filter(x => x > 2)        // Filter: [3, 4, 5]
arr.reduce((sum, x) => sum + x, 0)  // Reduce: 15
arr.find(x => x > 2)          // Find first: 3
arr.some(x => x > 3)          // Test some: true
arr.every(x => x > 0)         // Test all: true
```

### Async/Await
```javascript
// Async function
async function fetchData() {
    try {
        const response = await fetch('/api/data');
        const data = await response.json();
        return data;
    } catch (error) {
        console.error('Error:', error);
    }
}

// Promise.all for parallel requests
const [users, posts] = await Promise.all([
    fetch('/api/users').then(r => r.json()),
    fetch('/api/posts').then(r => r.json())
]);
```

### DOM Manipulation
```javascript
// Select elements
const el = document.querySelector('.class');
const els = document.querySelectorAll('.class');

// Modify elements
el.textContent = 'New text';
el.innerHTML = '<strong>Bold</strong>';
el.classList.add('active');
el.classList.toggle('visible');
el.style.color = 'red';

// Events
el.addEventListener('click', (e) => {
    e.preventDefault();
    console.log('Clicked!');
});
```

## ⚛️ React Quick Reference

### Component Patterns
```jsx
// Function component with hooks
function MyComponent({ prop1, prop2 }) {
    const [state, setState] = useState(initialValue);
    
    useEffect(() => {
        // Side effect
        return () => {
            // Cleanup
        };
    }, [dependency]);
    
    return <div>{state}</div>;
}

// Conditional rendering
{condition && <Component />}
{condition ? <ComponentA /> : <ComponentB />}

// Lists
{items.map(item => (
    <Item key={item.id} data={item} />
))}
```

### Hooks
```jsx
// State
const [count, setCount] = useState(0);
const [user, setUser] = useState({ name: '', email: '' });

// Effect
useEffect(() => {
    // Effect logic
}, []); // Dependencies

// Context
const value = useContext(MyContext);

// Reducer
const [state, dispatch] = useReducer(reducer, initialState);
```

## 🎨 Tailwind CSS Quick Reference

### Layout
```html
<!-- Flexbox -->
<div class="flex justify-center items-center">
<div class="flex flex-col space-y-4">

<!-- Grid -->
<div class="grid grid-cols-3 gap-4">
<div class="col-span-2">

<!-- Spacing -->
<div class="p-4 m-2">        <!-- padding: 1rem, margin: 0.5rem -->
<div class="px-6 py-3">      <!-- padding-x: 1.5rem, padding-y: 0.75rem -->
```

### Typography & Colors
```html
<!-- Text -->
<p class="text-lg font-bold text-center text-blue-600">
<h1 class="text-3xl font-extrabold text-gray-900">

<!-- Background -->
<div class="bg-blue-500 bg-opacity-50">
<div class="bg-gradient-to-r from-blue-500 to-purple-600">
```

### Responsive Design
```html
<!-- Mobile first -->
<div class="text-sm md:text-base lg:text-lg">
<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3">
<div class="hidden md:block">  <!-- Hide on mobile -->
```

## 📦 Git Quick Reference

### Basic Commands
```bash
# Setup
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Repository
git init                      # Initialize repo
git clone <url>              # Clone repo
git status                   # Check status
git add .                    # Stage all files
git commit -m "message"      # Commit changes
git push origin main         # Push to remote
git pull origin main         # Pull from remote

# Branching
git branch                   # List branches
git checkout -b feature      # Create and switch branch
git merge feature           # Merge branch
git branch -d feature       # Delete branch
```

### Advanced Commands
```bash
# Stashing
git stash                    # Stash changes
git stash pop               # Apply and remove stash
git stash list              # List stashes

# Undoing
git reset HEAD~1            # Undo last commit (keep changes)
git reset --hard HEAD~1     # Undo last commit (discard changes)
git revert <commit>         # Revert specific commit

# History
git log --oneline           # Compact log
git log --graph             # Visual graph
git diff                    # Show changes
```

## 🟢 Node.js Quick Reference

### File Operations
```javascript
const fs = require('fs').promises;

// Read file
const data = await fs.readFile('file.txt', 'utf8');

// Write file
await fs.writeFile('file.txt', 'content', 'utf8');

// Check if exists
try {
    await fs.access('file.txt');
    console.log('File exists');
} catch {
    console.log('File does not exist');
}

// Directory operations
await fs.mkdir('directory', { recursive: true });
const files = await fs.readdir('directory');
```

### HTTP Server
```javascript
const http = require('http');

const server = http.createServer((req, res) => {
    res.setHeader('Content-Type', 'application/json');
    res.statusCode = 200;
    res.end(JSON.stringify({ message: 'Hello World' }));
});

server.listen(3000, () => {
    console.log('Server running on port 3000');
});
```

## 📦 npm/Yarn Quick Reference

### Package Management
```bash
# npm
npm init -y                  # Initialize project
npm install package          # Install package
npm install -D package       # Install dev dependency
npm install -g package       # Install globally
npm uninstall package        # Remove package
npm update                   # Update packages
npm audit                    # Security audit

# Yarn
yarn init -y                 # Initialize project
yarn add package             # Install package
yarn add -D package          # Install dev dependency
yarn global add package      # Install globally
yarn remove package          # Remove package
yarn upgrade                 # Update packages
yarn audit                   # Security audit
```

### Scripts
```bash
# Run scripts
npm run script-name
yarn script-name

# Common scripts
npm start                    # Start application
npm test                     # Run tests
npm run build               # Build for production
npm run dev                 # Development mode
```

## 🔧 VS Code Shortcuts

### Essential Shortcuts
```
Ctrl/Cmd + P                 # Quick file open
Ctrl/Cmd + Shift + P         # Command palette
Ctrl/Cmd + `                 # Toggle terminal
Ctrl/Cmd + /                 # Toggle comment
Ctrl/Cmd + D                 # Select next occurrence
Alt + Up/Down                # Move line up/down
Shift + Alt + Up/Down        # Copy line up/down
Ctrl/Cmd + Shift + K         # Delete line
```

### Multi-cursor
```
Alt + Click                  # Add cursor
Ctrl/Cmd + Alt + Up/Down     # Add cursor above/below
Ctrl/Cmd + Shift + L         # Select all occurrences
```

## 🌐 Browser DevTools

### Console Commands
```javascript
console.log('message')       // Log message
console.error('error')       // Log error
console.table(array)         // Display as table
console.time('label')        // Start timer
console.timeEnd('label')     // End timer

// Inspect element
$0                          // Selected element
$$('selector')              // Query all elements
```

### Network Tab
- **Throttling**: Simulate slow connections
- **Preserve log**: Keep logs across page reloads
- **Disable cache**: Force fresh requests

## 🔍 Debugging Tips

### JavaScript Debugging
```javascript
// Breakpoints
debugger;                   // Programmatic breakpoint

// Console methods
console.assert(condition, 'message');
console.trace();            // Stack trace
console.count('label');     // Count calls

// Error handling
try {
    // Code that might throw
} catch (error) {
    console.error('Error:', error.message);
    console.error('Stack:', error.stack);
}
```

### React Debugging
```jsx
// React DevTools
// Install React Developer Tools browser extension

// Debug renders
console.log('Component rendered', props, state);

// Debug hooks
useEffect(() => {
    console.log('Effect ran', dependency);
}, [dependency]);
```

## 📱 Responsive Breakpoints

### Common Breakpoints
```css
/* Mobile first approach */
/* Default: Mobile (0px+) */

@media (min-width: 480px) {  /* Small mobile */}
@media (min-width: 768px) {  /* Tablet */}
@media (min-width: 1024px) { /* Desktop */}
@media (min-width: 1200px) { /* Large desktop */}
@media (min-width: 1440px) { /* Extra large */}
```

### Tailwind Breakpoints
```
sm: 640px
md: 768px
lg: 1024px
xl: 1280px
2xl: 1536px
```

## 🔗 Related Topics
- [[HTML]] - Complete HTML reference
- [[CSS]] - Complete CSS reference
- [[JavaScript]] - Complete JavaScript reference
- [[React]] - Complete React reference
- [[Git]] - Complete Git reference

---
*Keep this cheat sheet handy for quick reference while coding!* 