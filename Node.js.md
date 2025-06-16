# Node.js - JavaScript Runtime

> **Quick Summary**: Node.js is a JavaScript runtime built on Chrome's V8 engine. Use it for server-side development, scripting, and building development tools.

## 🚀 Getting Started

### Installation
```bash
# Install Node.js (recommended: use Node Version Manager)
# Install nvm (macOS/Linux)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# Install nvm (Windows - use nvm-windows)
# Download from: https://github.com/coreybutler/nvm-windows

# Install latest LTS version
nvm install --lts
nvm use --lts

# Check versions
node --version
npm --version

# Run Node.js REPL
node
```

### Basic Node.js Script
```javascript
// hello.js
console.log('Hello, Node.js!');

// Run the script
// node hello.js
```

## 📁 File System Operations

### Reading Files
```javascript
const fs = require('fs');
const path = require('path');

// Synchronous file reading
try {
    const data = fs.readFileSync('file.txt', 'utf8');
    console.log(data);
} catch (error) {
    console.error('Error reading file:', error.message);
}

// Asynchronous file reading (callback)
fs.readFile('file.txt', 'utf8', (error, data) => {
    if (error) {
        console.error('Error reading file:', error.message);
        return;
    }
    console.log(data);
});

// Asynchronous file reading (promises)
const fsPromises = require('fs').promises;

async function readFileAsync() {
    try {
        const data = await fsPromises.readFile('file.txt', 'utf8');
        console.log(data);
    } catch (error) {
        console.error('Error reading file:', error.message);
    }
}

// Read JSON file
async function readJsonFile(filename) {
    try {
        const data = await fsPromises.readFile(filename, 'utf8');
        return JSON.parse(data);
    } catch (error) {
        console.error('Error reading JSON file:', error.message);
        return null;
    }
}
```

### Writing Files
```javascript
const fs = require('fs');
const fsPromises = require('fs').promises;

// Synchronous file writing
try {
    fs.writeFileSync('output.txt', 'Hello, World!', 'utf8');
    console.log('File written successfully');
} catch (error) {
    console.error('Error writing file:', error.message);
}

// Asynchronous file writing (callback)
fs.writeFile('output.txt', 'Hello, World!', 'utf8', (error) => {
    if (error) {
        console.error('Error writing file:', error.message);
        return;
    }
    console.log('File written successfully');
});

// Asynchronous file writing (promises)
async function writeFileAsync(filename, content) {
    try {
        await fsPromises.writeFile(filename, content, 'utf8');
        console.log('File written successfully');
    } catch (error) {
        console.error('Error writing file:', error.message);
    }
}

// Append to file
async function appendToFile(filename, content) {
    try {
        await fsPromises.appendFile(filename, content, 'utf8');
        console.log('Content appended successfully');
    } catch (error) {
        console.error('Error appending to file:', error.message);
    }
}

// Write JSON file
async function writeJsonFile(filename, data) {
    try {
        const jsonString = JSON.stringify(data, null, 2);
        await fsPromises.writeFile(filename, jsonString, 'utf8');
        console.log('JSON file written successfully');
    } catch (error) {
        console.error('Error writing JSON file:', error.message);
    }
}
```

### Directory Operations
```javascript
const fs = require('fs');
const path = require('path');
const fsPromises = require('fs').promises;

// Check if file/directory exists
async function checkExists(filePath) {
    try {
        await fsPromises.access(filePath);
        return true;
    } catch {
        return false;
    }
}

// Create directory
async function createDirectory(dirPath) {
    try {
        await fsPromises.mkdir(dirPath, { recursive: true });
        console.log('Directory created successfully');
    } catch (error) {
        console.error('Error creating directory:', error.message);
    }
}

// Read directory contents
async function readDirectory(dirPath) {
    try {
        const files = await fsPromises.readdir(dirPath);
        return files;
    } catch (error) {
        console.error('Error reading directory:', error.message);
        return [];
    }
}

// Get file stats
async function getFileStats(filePath) {
    try {
        const stats = await fsPromises.stat(filePath);
        return {
            isFile: stats.isFile(),
            isDirectory: stats.isDirectory(),
            size: stats.size,
            modified: stats.mtime,
            created: stats.birthtime
        };
    } catch (error) {
        console.error('Error getting file stats:', error.message);
        return null;
    }
}

// Copy file
async function copyFile(source, destination) {
    try {
        await fsPromises.copyFile(source, destination);
        console.log('File copied successfully');
    } catch (error) {
        console.error('Error copying file:', error.message);
    }
}

// Delete file
async function deleteFile(filePath) {
    try {
        await fsPromises.unlink(filePath);
        console.log('File deleted successfully');
    } catch (error) {
        console.error('Error deleting file:', error.message);
    }
}
```

## 🛠️ Path Operations
```javascript
const path = require('path');

// Path manipulation
const filePath = '/users/john/documents/file.txt';

console.log(path.dirname(filePath));   // /users/john/documents
console.log(path.basename(filePath));  // file.txt
console.log(path.extname(filePath));   // .txt
console.log(path.parse(filePath));     // Object with path components

// Join paths
const fullPath = path.join('/users', 'john', 'documents', 'file.txt');
console.log(fullPath); // /users/john/documents/file.txt

// Resolve paths
const absolutePath = path.resolve('documents', 'file.txt');
console.log(absolutePath); // Absolute path from current directory

// Check if path is absolute
console.log(path.isAbsolute('/users/john')); // true
console.log(path.isAbsolute('documents'));   // false

// Get relative path
const relativePath = path.relative('/users/john', '/users/john/documents/file.txt');
console.log(relativePath); // documents/file.txt

// Cross-platform path separators
const crossPlatformPath = path.join('users', 'john', 'documents');
// Uses correct separator for current OS
```

## 🌐 HTTP Server

### Basic HTTP Server
```javascript
const http = require('http');
const url = require('url');

// Create basic server
const server = http.createServer((req, res) => {
    // Set response headers
    res.setHeader('Content-Type', 'text/html');
    res.setHeader('Access-Control-Allow-Origin', '*');
    
    // Parse URL
    const parsedUrl = url.parse(req.url, true);
    const pathname = parsedUrl.pathname;
    const query = parsedUrl.query;
    
    // Handle different routes
    if (pathname === '/') {
        res.statusCode = 200;
        res.end('<h1>Welcome to Node.js Server!</h1>');
    } else if (pathname === '/api/users') {
        res.statusCode = 200;
        res.setHeader('Content-Type', 'application/json');
        res.end(JSON.stringify({ users: ['John', 'Jane'] }));
    } else if (pathname === '/api/user' && query.id) {
        res.statusCode = 200;
        res.setHeader('Content-Type', 'application/json');
        res.end(JSON.stringify({ id: query.id, name: 'User ' + query.id }));
    } else {
        res.statusCode = 404;
        res.end('<h1>404 - Page Not Found</h1>');
    }
});

// Start server
const PORT = process.env.PORT || 3000;
server.listen(PORT, () => {
    console.log(`Server running on http://localhost:${PORT}`);
});
```

### Express.js Server
```javascript
// First install: npm install express
const express = require('express');
const app = express();

// Middleware
app.use(express.json()); // Parse JSON bodies
app.use(express.static('public')); // Serve static files

// Routes
app.get('/', (req, res) => {
    res.send('<h1>Welcome to Express!</h1>');
});

app.get('/api/users', (req, res) => {
    const users = [
        { id: 1, name: 'John Doe', email: 'john@example.com' },
        { id: 2, name: 'Jane Smith', email: 'jane@example.com' }
    ];
    res.json(users);
});

app.get('/api/users/:id', (req, res) => {
    const userId = parseInt(req.params.id);
    const user = { id: userId, name: `User ${userId}` };
    res.json(user);
});

app.post('/api/users', (req, res) => {
    const { name, email } = req.body;
    const newUser = {
        id: Date.now(),
        name,
        email
    };
    res.status(201).json(newUser);
});

// Error handling middleware
app.use((error, req, res, next) => {
    console.error(error.stack);
    res.status(500).json({ error: 'Something went wrong!' });
});

// 404 handler
app.use((req, res) => {
    res.status(404).json({ error: 'Route not found' });
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
    console.log(`Express server running on http://localhost:${PORT}`);
});
```

## 📦 Working with Modules

### CommonJS Modules
```javascript
// math.js - Export functions
function add(a, b) {
    return a + b;
}

function subtract(a, b) {
    return a - b;
}

const PI = 3.14159;

// Export individual functions
exports.add = add;
exports.subtract = subtract;
exports.PI = PI;

// Or export as object
module.exports = {
    add,
    subtract,
    PI
};

// app.js - Import and use
const math = require('./math');
const { add, subtract } = require('./math');

console.log(math.add(5, 3)); // 8
console.log(add(5, 3)); // 8
```

### ES6 Modules (with .mjs or "type": "module")
```javascript
// math.mjs - Export functions
export function add(a, b) {
    return a + b;
}

export function subtract(a, b) {
    return a - b;
}

export const PI = 3.14159;

// Default export
export default function multiply(a, b) {
    return a * b;
}

// app.mjs - Import and use
import multiply, { add, subtract, PI } from './math.mjs';
import * as math from './math.mjs';

console.log(add(5, 3)); // 8
console.log(multiply(5, 3)); // 15
```

## 🔧 Environment and Process

### Environment Variables
```javascript
// Access environment variables
const port = process.env.PORT || 3000;
const dbUrl = process.env.DATABASE_URL || 'localhost:5432';
const nodeEnv = process.env.NODE_ENV || 'development';

// Set environment variables in .env file
// Install: npm install dotenv
require('dotenv').config();

// .env file
/*
PORT=3000
DATABASE_URL=postgresql://localhost:5432/mydb
API_KEY=your-secret-key
NODE_ENV=development
*/

// Use environment variables
const config = {
    port: process.env.PORT,
    databaseUrl: process.env.DATABASE_URL,
    apiKey: process.env.API_KEY,
    isDevelopment: process.env.NODE_ENV === 'development'
};
```

### Process Information
```javascript
// Process information
console.log('Node.js version:', process.version);
console.log('Platform:', process.platform);
console.log('Architecture:', process.arch);
console.log('Current working directory:', process.cwd());
console.log('Process ID:', process.pid);
console.log('Command line arguments:', process.argv);

// Handle process events
process.on('exit', (code) => {
    console.log(`Process exiting with code: ${code}`);
});

process.on('uncaughtException', (error) => {
    console.error('Uncaught Exception:', error);
    process.exit(1);
});

process.on('unhandledRejection', (reason, promise) => {
    console.error('Unhandled Rejection at:', promise, 'reason:', reason);
});

// Graceful shutdown
process.on('SIGINT', () => {
    console.log('Received SIGINT. Graceful shutdown...');
    // Cleanup code here
    process.exit(0);
});

process.on('SIGTERM', () => {
    console.log('Received SIGTERM. Graceful shutdown...');
    // Cleanup code here
    process.exit(0);
});
```

## 🛠️ Utilities and Built-in Modules

### URL Module
```javascript
const url = require('url');

// Parse URL
const myUrl = 'https://example.com:8080/path/to/resource?name=john&age=30#section';
const parsedUrl = url.parse(myUrl, true);

console.log(parsedUrl.protocol); // https:
console.log(parsedUrl.hostname); // example.com
console.log(parsedUrl.port);     // 8080
console.log(parsedUrl.pathname); // /path/to/resource
console.log(parsedUrl.query);    // { name: 'john', age: '30' }
console.log(parsedUrl.hash);     // #section

// URL constructor (modern approach)
const myURL = new URL('https://example.com:8080/path?name=john');
console.log(myURL.hostname); // example.com
console.log(myURL.searchParams.get('name')); // john

// Build URL
const builtUrl = url.format({
    protocol: 'https:',
    hostname: 'example.com',
    pathname: '/api/users',
    query: { page: 1, limit: 10 }
});
console.log(builtUrl); // https://example.com/api/users?page=1&limit=10
```

### Crypto Module
```javascript
const crypto = require('crypto');

// Generate random bytes
const randomBytes = crypto.randomBytes(16).toString('hex');
console.log('Random bytes:', randomBytes);

// Hash data
const hash = crypto.createHash('sha256');
hash.update('Hello, World!');
const hashedData = hash.digest('hex');
console.log('SHA256 hash:', hashedData);

// HMAC
const hmac = crypto.createHmac('sha256', 'secret-key');
hmac.update('Hello, World!');
const hmacDigest = hmac.digest('hex');
console.log('HMAC:', hmacDigest);

// Generate UUID
function generateUUID() {
    return crypto.randomUUID();
}
console.log('UUID:', generateUUID());
```

### OS Module
```javascript
const os = require('os');

// System information
console.log('Platform:', os.platform());
console.log('Architecture:', os.arch());
console.log('CPU cores:', os.cpus().length);
console.log('Total memory:', Math.round(os.totalmem() / 1024 / 1024 / 1024) + ' GB');
console.log('Free memory:', Math.round(os.freemem() / 1024 / 1024 / 1024) + ' GB');
console.log('Home directory:', os.homedir());
console.log('Temp directory:', os.tmpdir());
console.log('Hostname:', os.hostname());
console.log('Username:', os.userInfo().username);

// Network interfaces
const networkInterfaces = os.networkInterfaces();
console.log('Network interfaces:', Object.keys(networkInterfaces));
```

## 📝 Scripting Examples

### File Processing Script
```javascript
#!/usr/bin/env node

const fs = require('fs').promises;
const path = require('path');

async function processFiles(directory) {
    try {
        const files = await fs.readdir(directory);
        
        for (const file of files) {
            const filePath = path.join(directory, file);
            const stats = await fs.stat(filePath);
            
            if (stats.isFile() && path.extname(file) === '.txt') {
                console.log(`Processing: ${file}`);
                
                // Read file content
                const content = await fs.readFile(filePath, 'utf8');
                
                // Process content (example: convert to uppercase)
                const processedContent = content.toUpperCase();
                
                // Write processed content to new file
                const outputPath = path.join(directory, `processed_${file}`);
                await fs.writeFile(outputPath, processedContent, 'utf8');
                
                console.log(`Created: processed_${file}`);
            }
        }
    } catch (error) {
        console.error('Error processing files:', error.message);
    }
}

// Get directory from command line argument
const directory = process.argv[2] || './';
processFiles(directory);
```

### Command Line Tool
```javascript
#!/usr/bin/env node

const fs = require('fs').promises;
const path = require('path');

// Simple command line argument parsing
const args = process.argv.slice(2);
const command = args[0];

async function createFile(filename, content = '') {
    try {
        await fs.writeFile(filename, content, 'utf8');
        console.log(`✅ Created file: ${filename}`);
    } catch (error) {
        console.error(`❌ Error creating file: ${error.message}`);
    }
}

async function deleteFile(filename) {
    try {
        await fs.unlink(filename);
        console.log(`✅ Deleted file: ${filename}`);
    } catch (error) {
        console.error(`❌ Error deleting file: ${error.message}`);
    }
}

async function listFiles(directory = './') {
    try {
        const files = await fs.readdir(directory);
        console.log(`📁 Files in ${directory}:`);
        files.forEach(file => console.log(`  - ${file}`));
    } catch (error) {
        console.error(`❌ Error listing files: ${error.message}`);
    }
}

// Command handling
switch (command) {
    case 'create':
        if (args[1]) {
            createFile(args[1], args[2] || '');
        } else {
            console.log('Usage: node script.js create <filename> [content]');
        }
        break;
    
    case 'delete':
        if (args[1]) {
            deleteFile(args[1]);
        } else {
            console.log('Usage: node script.js delete <filename>');
        }
        break;
    
    case 'list':
        listFiles(args[1]);
        break;
    
    default:
        console.log('Available commands:');
        console.log('  create <filename> [content] - Create a new file');
        console.log('  delete <filename>           - Delete a file');
        console.log('  list [directory]            - List files in directory');
}
```

## ✅ Quick Checklist

- [ ] Use LTS version of Node.js
- [ ] Handle errors properly with try-catch
- [ ] Use asynchronous operations for I/O
- [ ] Set up environment variables with .env
- [ ] Implement graceful shutdown handling
- [ ] Use path module for cross-platform compatibility
- [ ] Validate input and handle edge cases
- [ ] Use appropriate HTTP status codes
- [ ] Implement proper logging
- [ ] Follow security best practices

## 🔗 Related Topics
- [[npm-yarn|npm & Yarn]] - Package management for Node.js
- [[JavaScript]] - Core language features
- [[Git]] - Version control for Node.js projects

---
*Remember: Node.js excels at I/O-intensive applications. Use its asynchronous nature and rich ecosystem to build scalable applications!* 