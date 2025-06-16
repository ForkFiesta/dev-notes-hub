---
tags: [tooling, husky, lint-staged, git-hooks, pre-commit, automation]
created: 2025-01-27
updated: 2025-01-27
---

# 🐕 Husky + Lint-Staged

> **Summary**: Complete setup guide for Husky and lint-staged to automate code quality checks with Git hooks, ensuring consistent code standards across your team.

## 📋 Table of Contents

- [Installation & Setup](#installation--setup)
- [Basic Configuration](#basic-configuration)
- [Advanced Configurations](#advanced-configurations)
- [Common Use Cases](#common-use-cases)
- [Troubleshooting](#troubleshooting)
- [Team Best Practices](#team-best-practices)

## Installation & Setup

### Basic Installation
```bash
# Install Husky and lint-staged
npm install --save-dev husky lint-staged

# Initialize Husky
npx husky install

# Add prepare script to package.json
npm pkg set scripts.prepare="husky install"
```

### Package.json Configuration
```json
{
  "scripts": {
    "prepare": "husky install",
    "lint": "eslint . --ext .js,.jsx,.ts,.tsx",
    "lint:fix": "eslint . --ext .js,.jsx,.ts,.tsx --fix",
    "format": "prettier --write .",
    "type-check": "tsc --noEmit"
  },
  "lint-staged": {
    "*.{js,jsx,ts,tsx}": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.{json,md,yml,yaml}": [
      "prettier --write"
    ]
  }
}
```

### Create Pre-commit Hook
```bash
# Create pre-commit hook
npx husky add .husky/pre-commit "npx lint-staged"

# Make it executable (if needed)
chmod +x .husky/pre-commit
```

## Basic Configuration

### Simple Pre-commit Setup
```bash
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

# Run lint-staged
npx lint-staged
```

### Lint-staged Configuration Options
```javascript
// lint-staged.config.js
module.exports = {
  // JavaScript/TypeScript files
  '*.{js,jsx,ts,tsx}': [
    'eslint --fix',
    'prettier --write',
    'git add'
  ],
  
  // Style files
  '*.{css,scss,less}': [
    'stylelint --fix',
    'prettier --write'
  ],
  
  // Markdown and config files
  '*.{md,json,yml,yaml}': [
    'prettier --write'
  ],
  
  // Package.json
  'package.json': [
    'sort-package-json',
    'prettier --write'
  ]
};
```

### Alternative Configuration in Package.json
```json
{
  "lint-staged": {
    "*.{js,jsx,ts,tsx}": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.{css,scss}": [
      "stylelint --fix",
      "prettier --write"
    ],
    "*.{md,json}": [
      "prettier --write"
    ]
  }
}
```

## Advanced Configurations

### Multiple Git Hooks
```bash
# Pre-commit hook
npx husky add .husky/pre-commit "npx lint-staged"

# Pre-push hook
npx husky add .husky/pre-push "npm run type-check && npm test"

# Commit message hook
npx husky add .husky/commit-msg "npx commitlint --edit $1"

# Post-merge hook
npx husky add .husky/post-merge "npm install"
```

### Complex Lint-staged Configuration
```javascript
// lint-staged.config.js
const path = require('path');

module.exports = {
  // Run different commands based on file location
  'src/**/*.{js,jsx,ts,tsx}': [
    'eslint --fix',
    'prettier --write',
    () => 'npm run type-check'
  ],
  
  // Test files
  '**/*.test.{js,jsx,ts,tsx}': [
    'eslint --fix',
    'prettier --write',
    'jest --bail --findRelatedTests'
  ],
  
  // Style files with different configs
  'src/**/*.{css,scss}': [
    'stylelint --fix --config .stylelintrc.json',
    'prettier --write'
  ],
  
  // Documentation
  'docs/**/*.md': [
    'markdownlint --fix',
    'prettier --write'
  ],
  
  // Package files
  'package.json': [
    'sort-package-json',
    'prettier --write'
  ],
  
  // Custom function for complex logic
  '*.{js,jsx,ts,tsx}': (filenames) => {
    const commands = [];
    
    // Run ESLint
    commands.push(`eslint --fix ${filenames.join(' ')}`);
    
    // Run Prettier
    commands.push(`prettier --write ${filenames.join(' ')}`);
    
    // Run tests for changed files
    if (filenames.some(f => f.includes('.test.'))) {
      commands.push('npm test -- --passWithNoTests');
    }
    
    return commands;
  }
};
```

### Conditional Execution
```javascript
// lint-staged.config.js
module.exports = {
  '*.{js,jsx,ts,tsx}': (filenames) => {
    const commands = [];
    
    // Always run linting
    commands.push(`eslint --fix ${filenames.join(' ')}`);
    commands.push(`prettier --write ${filenames.join(' ')}`);
    
    // Only run type checking if TypeScript files changed
    const tsFiles = filenames.filter(f => f.endsWith('.ts') || f.endsWith('.tsx'));
    if (tsFiles.length > 0) {
      commands.push('tsc --noEmit');
    }
    
    // Run tests for component files
    const componentFiles = filenames.filter(f => 
      f.includes('/components/') && !f.includes('.test.')
    );
    if (componentFiles.length > 0) {
      commands.push('npm test -- --passWithNoTests');
    }
    
    return commands;
  }
};
```

## Common Use Cases

### React + TypeScript Project
```json
{
  "lint-staged": {
    "*.{js,jsx,ts,tsx}": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.{css,scss,module.css,module.scss}": [
      "stylelint --fix",
      "prettier --write"
    ],
    "*.{json,md}": [
      "prettier --write"
    ]
  }
}
```

### Monorepo Configuration
```javascript
// lint-staged.config.js
module.exports = {
  // Frontend package
  'packages/frontend/**/*.{js,jsx,ts,tsx}': [
    'eslint --fix --config packages/frontend/.eslintrc.js',
    'prettier --write'
  ],
  
  // Backend package
  'packages/backend/**/*.{js,ts}': [
    'eslint --fix --config packages/backend/.eslintrc.js',
    'prettier --write'
  ],
  
  // Shared package
  'packages/shared/**/*.{js,ts}': [
    'eslint --fix --config packages/shared/.eslintrc.js',
    'prettier --write',
    'npm run build --workspace=packages/shared'
  ],
  
  // Root level files
  '*.{js,json,md}': [
    'prettier --write'
  ]
};
```

### Next.js Project
```json
{
  "lint-staged": {
    "*.{js,jsx,ts,tsx}": [
      "next lint --fix",
      "prettier --write"
    ],
    "*.{css,scss}": [
      "stylelint --fix",
      "prettier --write"
    ],
    "*.{json,md}": [
      "prettier --write"
    ]
  }
}
```

### Full-Stack Project with Testing
```javascript
// lint-staged.config.js
module.exports = {
  // Frontend files
  'client/**/*.{js,jsx,ts,tsx}': [
    'eslint --fix',
    'prettier --write',
    'jest --bail --findRelatedTests --passWithNoTests'
  ],
  
  // Backend files
  'server/**/*.{js,ts}': [
    'eslint --fix',
    'prettier --write',
    'jest --bail --findRelatedTests --passWithNoTests'
  ],
  
  // Database migrations
  'server/migrations/*.sql': [
    'sql-formatter --fix'
  ],
  
  // API documentation
  'docs/api/**/*.md': [
    'markdownlint --fix',
    'prettier --write'
  ],
  
  // Configuration files
  '*.{json,yml,yaml}': [
    'prettier --write'
  ]
};
```

## Troubleshooting

### Common Issues & Solutions

#### Husky Not Working After Clone
```bash
# Solution: Reinstall Husky hooks
npm run prepare

# Or manually
npx husky install
```

#### Permission Issues
```bash
# Make hooks executable
chmod +x .husky/*

# Or specific hook
chmod +x .husky/pre-commit
```

#### Lint-staged Hanging
```javascript
// lint-staged.config.js - Add timeout
module.exports = {
  '*.{js,jsx,ts,tsx}': [
    'eslint --fix --max-warnings=0',
    'prettier --write'
  ]
};

// Or in package.json
{
  "lint-staged": {
    "*.{js,jsx,ts,tsx}": [
      "timeout 30s eslint --fix",
      "prettier --write"
    ]
  }
}
```

#### Skipping Hooks (Emergency)
```bash
# Skip pre-commit hook
git commit --no-verify -m "Emergency fix"

# Skip pre-push hook
git push --no-verify
```

#### Performance Issues
```javascript
// lint-staged.config.js - Optimize for performance
module.exports = {
  '*.{js,jsx,ts,tsx}': [
    // Run ESLint only on changed files
    'eslint --fix --cache',
    'prettier --write'
  ],
  
  // Avoid running expensive operations
  '*.{js,jsx,ts,tsx}': (filenames) => {
    // Only run type check if more than 5 files changed
    if (filenames.length > 5) {
      return [
        `eslint --fix ${filenames.join(' ')}`,
        `prettier --write ${filenames.join(' ')}`,
        'tsc --noEmit'
      ];
    }
    
    return [
      `eslint --fix ${filenames.join(' ')}`,
      `prettier --write ${filenames.join(' ')}`
    ];
  }
};
```

### Debugging Configuration
```bash
# Debug lint-staged
npx lint-staged --debug

# Test specific files
npx lint-staged --diff="HEAD~1"

# Dry run
npx lint-staged --dry-run
```

### Custom Hook Scripts
```bash
#!/usr/bin/env sh
# .husky/pre-commit

. "$(dirname -- "$0")/_/husky.sh"

echo "🔍 Running pre-commit checks..."

# Run lint-staged
npx lint-staged

# Check if there are any staged files after linting
if git diff --cached --quiet; then
  echo "❌ No staged files found after linting. Please stage your changes."
  exit 1
fi

echo "✅ Pre-commit checks passed!"
```

## Team Best Practices

### Onboarding New Developers
```bash
# .husky/post-checkout
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

# Check if switching to a different branch
if [ $3 = 1 ]; then
  echo "📦 Installing dependencies..."
  npm install
  
  echo "🔧 Setting up development environment..."
  npm run setup
fi
```

### Commit Message Validation
```bash
# Install commitlint
npm install --save-dev @commitlint/cli @commitlint/config-conventional

# Create commitlint config
echo "module.exports = {extends: ['@commitlint/config-conventional']}" > commitlint.config.js

# Add commit-msg hook
npx husky add .husky/commit-msg "npx commitlint --edit $1"
```

### Branch Protection
```bash
#!/usr/bin/env sh
# .husky/pre-push

. "$(dirname -- "$0")/_/husky.sh"

# Prevent pushing to main/master
branch=$(git rev-parse --abbrev-ref HEAD)

if [ "$branch" = "main" ] || [ "$branch" = "master" ]; then
  echo "❌ Direct push to $branch is not allowed"
  exit 1
fi

# Run tests before push
echo "🧪 Running tests..."
npm test

echo "✅ Pre-push checks passed!"
```

### Configuration Templates
```javascript
// scripts/setup-hooks.js
const fs = require('fs');
const path = require('path');

const hooks = {
  'pre-commit': `#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

echo "🔍 Running pre-commit checks..."
npx lint-staged
echo "✅ Pre-commit checks passed!"`,

  'pre-push': `#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

echo "🧪 Running tests..."
npm test
echo "✅ Tests passed!"`,

  'commit-msg': `#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

npx commitlint --edit $1`
};

// Create hooks
Object.entries(hooks).forEach(([name, content]) => {
  const hookPath = path.join('.husky', name);
  fs.writeFileSync(hookPath, content);
  fs.chmodSync(hookPath, '755');
  console.log(`✅ Created ${name} hook`);
});
```

### CI/CD Integration
```yaml
# .github/workflows/ci.yml
name: CI

on: [push, pull_request]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - run: npm ci
      
      # Run the same checks as pre-commit hook
      - run: npm run lint
      - run: npm run format:check
      - run: npm run type-check
      - run: npm test
```

### Documentation
```markdown
# Development Setup

## Git Hooks

This project uses Husky and lint-staged to ensure code quality:

- **pre-commit**: Runs ESLint and Prettier on staged files
- **pre-push**: Runs type checking and tests
- **commit-msg**: Validates commit message format

## Setup

```bash
npm install
npm run prepare
```

## Bypassing Hooks (Emergency Only)

```bash
git commit --no-verify -m "Emergency fix"
```

## Troubleshooting

If hooks aren't working:
1. Run `npm run prepare`
2. Check hook permissions: `ls -la .husky/`
3. Make executable: `chmod +x .husky/*`
```

## 🔗 Related Notes

- [[ESLint & Prettier]] - Code quality tools configuration
- [[Git]] - Version control workflows
- [[CI-CD & Deployment]] - Automated testing and deployment
- [[Testing]] - Test automation strategies

## 📚 Additional Resources

- [Husky Documentation](https://typicode.github.io/husky/)
- [lint-staged Documentation](https://github.com/okonet/lint-staged)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [Git Hooks Documentation](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks)

---
*Last updated: January 27, 2025* 