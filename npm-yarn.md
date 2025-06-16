# npm & Yarn - Package Management

> **Quick Summary**: npm and Yarn are package managers for Node.js. They handle dependencies, scripts, and project configuration for JavaScript projects.

## 📦 Package Managers Overview

### npm (Node Package Manager)
- **Built-in**: Comes with Node.js installation
- **Registry**: npmjs.com (default)
- **Lock file**: package-lock.json
- **Workspace support**: Yes (npm 7+)

### Yarn
- **Installation**: Separate installation required
- **Registry**: npmjs.com (default, configurable)
- **Lock file**: yarn.lock
- **Workspace support**: Yes (Yarn Workspaces)

## 🚀 Getting Started

### Installation
```bash
# npm comes with Node.js
node --version
npm --version

# Install Yarn
npm install -g yarn
# or
corepack enable yarn

# Check versions
yarn --version
```

### Initialize Project
```bash
# npm
npm init
npm init -y  # Skip questions, use defaults

# Yarn
yarn init
yarn init -y  # Skip questions, use defaults

# Create package.json manually
{
  "name": "my-project",
  "version": "1.0.0",
  "description": "My awesome project",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": ["javascript", "node"],
  "author": "Your Name",
  "license": "MIT"
}
```

## 📥 Installing Packages

### Basic Installation
```bash
# npm
npm install package-name
npm install express
npm install express@4.18.0  # Specific version
npm install express@latest   # Latest version

# Yarn
yarn add package-name
yarn add express
yarn add express@4.18.0     # Specific version
yarn add express@latest      # Latest version
```

### Development Dependencies
```bash
# npm
npm install --save-dev package-name
npm install -D eslint prettier

# Yarn
yarn add --dev package-name
yarn add -D eslint prettier
```

### Global Installation
```bash
# npm
npm install -g package-name
npm install -g nodemon create-react-app

# Yarn
yarn global add package-name
yarn global add nodemon create-react-app

# List global packages
npm list -g --depth=0
yarn global list
```

### Install from package.json
```bash
# npm
npm install
npm ci  # Clean install (faster, for CI/CD)

# Yarn
yarn install
yarn install --frozen-lockfile  # Don't update lock file
```

## 🗑️ Removing Packages

### Uninstall Packages
```bash
# npm
npm uninstall package-name
npm uninstall express
npm uninstall -D eslint  # Remove dev dependency

# Yarn
yarn remove package-name
yarn remove express
```

### Clean Installation
```bash
# npm
rm -rf node_modules package-lock.json
npm install

# Yarn
rm -rf node_modules yarn.lock
yarn install
```

## 📋 Package Information

### View Package Info
```bash
# npm
npm list                    # List installed packages
npm list --depth=0         # Top-level only
npm list package-name      # Specific package
npm outdated               # Check for updates
npm audit                  # Security audit
npm view express           # Package info from registry

# Yarn
yarn list                  # List installed packages
yarn list --depth=0       # Top-level only
yarn outdated             # Check for updates
yarn audit                # Security audit
yarn info express         # Package info from registry
```

### Update Packages
```bash
# npm
npm update                 # Update all packages
npm update package-name    # Update specific package
npm install package-name@latest  # Update to latest

# Yarn
yarn upgrade              # Update all packages
yarn upgrade package-name # Update specific package
yarn upgrade package-name@latest  # Update to latest
```

## 📝 Scripts

### Package.json Scripts
```json
{
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js",
    "build": "webpack --mode production",
    "test": "jest",
    "test:watch": "jest --watch",
    "lint": "eslint src/",
    "lint:fix": "eslint src/ --fix",
    "format": "prettier --write src/",
    "clean": "rm -rf dist/",
    "prebuild": "npm run clean",
    "postbuild": "echo 'Build completed!'",
    "deploy": "npm run build && npm run upload"
  }
}
```

### Running Scripts
```bash
# npm
npm run script-name
npm start              # Special script, no 'run' needed
npm test               # Special script, no 'run' needed
npm run dev
npm run build

# Yarn
yarn script-name
yarn start             # Special script
yarn test              # Special script
yarn dev
yarn build

# Pass arguments to scripts
npm run test -- --watch
yarn test --watch
```

### Script Hooks
```json
{
  "scripts": {
    "prebuild": "echo 'Before build'",
    "build": "webpack",
    "postbuild": "echo 'After build'",
    "prestart": "npm run build",
    "start": "node server.js"
  }
}
```

## 🔧 Configuration

### npm Configuration
```bash
# View configuration
npm config list
npm config get registry

# Set configuration
npm config set registry https://registry.npmjs.org/
npm config set init-author-name "Your Name"
npm config set init-author-email "your.email@example.com"
npm config set init-license "MIT"

# Delete configuration
npm config delete key-name

# Edit config file
npm config edit

# .npmrc file (project-specific)
registry=https://registry.npmjs.org/
save-exact=true
engine-strict=true
```

### Yarn Configuration
```bash
# View configuration
yarn config list
yarn config get registry

# Set configuration
yarn config set registry https://registry.npmjs.org/
yarn config set init-author-name "Your Name"
yarn config set init-author-email "your.email@example.com"
yarn config set init-license "MIT"

# Delete configuration
yarn config delete key-name

# .yarnrc.yml file (Yarn 2+)
nodeLinker: node-modules
yarnPath: .yarn/releases/yarn-3.2.0.cjs
```

## 🔒 Security

### Security Auditing
```bash
# npm
npm audit                  # Check for vulnerabilities
npm audit fix             # Fix vulnerabilities automatically
npm audit fix --force     # Force fix (may break changes)
npm audit --audit-level high  # Only high severity

# Yarn
yarn audit                # Check for vulnerabilities
yarn audit --level high   # Only high severity
```

### Package Verification
```bash
# npm
npm pack                  # Create tarball
npm publish --dry-run     # Test publish without actually publishing

# Yarn
yarn pack                 # Create tarball
```

## 📊 Version Management

### Semantic Versioning (SemVer)
```bash
# Version format: MAJOR.MINOR.PATCH
# 1.2.3
# ^1.2.3  - Compatible within major version (>=1.2.3 <2.0.0)
# ~1.2.3  - Compatible within minor version (>=1.2.3 <1.3.0)
# 1.2.3   - Exact version

# Update version
npm version patch    # 1.0.0 -> 1.0.1
npm version minor    # 1.0.0 -> 1.1.0
npm version major    # 1.0.0 -> 2.0.0

yarn version --patch
yarn version --minor
yarn version --major
```

### Lock Files
```bash
# package-lock.json (npm)
# - Locks exact versions of all dependencies
# - Should be committed to version control
# - Ensures consistent installs across environments

# yarn.lock (Yarn)
# - Locks exact versions of all dependencies
# - Should be committed to version control
# - Ensures consistent installs across environments

# Regenerate lock files
npm install --package-lock-only
yarn install --mode update-lockfile
```

## 🏗️ Workspaces

### npm Workspaces
```json
// package.json (root)
{
  "name": "my-monorepo",
  "workspaces": [
    "packages/*",
    "apps/*"
  ]
}

// packages/utils/package.json
{
  "name": "@my-org/utils",
  "version": "1.0.0"
}

// packages/app/package.json
{
  "name": "@my-org/app",
  "dependencies": {
    "@my-org/utils": "1.0.0"
  }
}
```

```bash
# npm workspace commands
npm install                           # Install all workspace dependencies
npm run test --workspaces            # Run test in all workspaces
npm run build --workspace=@my-org/app # Run build in specific workspace
npm install lodash --workspace=@my-org/app # Install package in specific workspace
```

### Yarn Workspaces
```json
// package.json (root)
{
  "name": "my-monorepo",
  "private": true,
  "workspaces": [
    "packages/*",
    "apps/*"
  ]
}
```

```bash
# Yarn workspace commands
yarn install                         # Install all workspace dependencies
yarn workspaces run test            # Run test in all workspaces
yarn workspace @my-org/app run build # Run build in specific workspace
yarn workspace @my-org/app add lodash # Install package in specific workspace
```

## 🚀 Publishing Packages

### Prepare for Publishing
```json
{
  "name": "my-awesome-package",
  "version": "1.0.0",
  "description": "An awesome package",
  "main": "index.js",
  "files": [
    "lib/",
    "README.md",
    "LICENSE"
  ],
  "keywords": ["awesome", "package"],
  "author": "Your Name <your.email@example.com>",
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "https://github.com/username/my-awesome-package.git"
  },
  "bugs": {
    "url": "https://github.com/username/my-awesome-package/issues"
  },
  "homepage": "https://github.com/username/my-awesome-package#readme"
}
```

### Publishing
```bash
# npm
npm login                    # Login to npm registry
npm whoami                   # Check logged in user
npm publish                  # Publish package
npm publish --access public  # Publish scoped package as public
npm unpublish package-name@version # Unpublish (within 72 hours)

# Yarn
yarn login                   # Login to npm registry
yarn publish                 # Publish package
yarn publish --access public # Publish scoped package as public
```

### Pre-publish Scripts
```json
{
  "scripts": {
    "prepublishOnly": "npm run test && npm run build",
    "prepack": "npm run build",
    "postpack": "echo 'Package created successfully'"
  }
}
```

## 🛠️ Development Tools

### Useful Packages
```bash
# Development tools
npm install -D nodemon          # Auto-restart server
npm install -D eslint           # Code linting
npm install -D prettier         # Code formatting
npm install -D jest             # Testing framework
npm install -D webpack          # Module bundler
npm install -D typescript       # TypeScript support

# Utility libraries
npm install lodash              # Utility functions
npm install axios               # HTTP client
npm install express             # Web framework
npm install dotenv              # Environment variables
npm install cors                # CORS middleware
```

### Package.json Best Practices
```json
{
  "name": "my-project",
  "version": "1.0.0",
  "description": "Clear, concise description",
  "main": "index.js",
  "engines": {
    "node": ">=14.0.0",
    "npm": ">=6.0.0"
  },
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js",
    "test": "jest",
    "lint": "eslint .",
    "format": "prettier --write ."
  },
  "dependencies": {
    "express": "^4.18.0"
  },
  "devDependencies": {
    "nodemon": "^2.0.0",
    "jest": "^28.0.0"
  },
  "keywords": ["web", "server", "api"],
  "author": "Your Name <email@example.com>",
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "https://github.com/username/repo.git"
  }
}
```

## 🔍 Troubleshooting

### Common Issues
```bash
# Clear npm cache
npm cache clean --force

# Clear Yarn cache
yarn cache clean

# Fix permission issues (macOS/Linux)
sudo chown -R $(whoami) ~/.npm
sudo chown -R $(whoami) ~/.config/yarn

# Reinstall node_modules
rm -rf node_modules package-lock.json
npm install

# Check for duplicate packages
npm ls --depth=0
yarn list --depth=0

# Fix peer dependency warnings
npm install --legacy-peer-deps

# Update npm itself
npm install -g npm@latest

# Update Yarn
yarn set version latest
```

### Performance Tips
```bash
# Use npm ci in CI/CD (faster, reliable)
npm ci

# Use Yarn with frozen lockfile
yarn install --frozen-lockfile

# Enable parallel installs
npm config set maxsockets 15

# Use local registry/cache
npm config set registry http://localhost:4873/
```

## ✅ Quick Checklist

- [ ] Use package-lock.json or yarn.lock in version control
- [ ] Specify Node.js version in engines field
- [ ] Use semantic versioning for releases
- [ ] Include meaningful scripts in package.json
- [ ] Audit packages regularly for security
- [ ] Use exact versions for critical dependencies
- [ ] Document installation and usage in README
- [ ] Test packages before publishing
- [ ] Use .npmignore or files field to control published content
- [ ] Keep dependencies up to date

## 🔗 Related Topics
- [[Node.js]] - JavaScript runtime for package execution
- [[Git]] - Version control for package development
- [[JavaScript]] - Core language for packages

---
*Remember: Good package management is crucial for project maintainability. Keep dependencies updated and use lock files for consistency!* 