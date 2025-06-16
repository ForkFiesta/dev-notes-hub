---
tags: [tooling, eslint, prettier, code-quality, formatting, linting]
created: 2025-06-15
updated: 2025-06-15
---

# 🔧 ESLint & Prettier

> **Summary**: Complete setup guide for ESLint and Prettier including custom rules, team configurations, and integration with popular frameworks and editors.

## 📋 Table of Contents

- [ESLint Setup](#eslint-setup)
- [Prettier Setup](#prettier-setup)
- [Integration & Configuration](#integration--configuration)
- [Custom Rules & Plugins](#custom-rules--plugins)
- [Team Configurations](#team-configurations)
- [Editor Integration](#editor-integration)
- [Troubleshooting](#troubleshooting)

## ESLint Setup

### Basic Installation
```bash
# Install ESLint
npm install --save-dev eslint

# Initialize configuration
npx eslint --init

# Or use create-config for interactive setup
npm create @eslint/config
```

### Basic Configuration
```javascript
// .eslintrc.js
module.exports = {
  env: {
    browser: true,
    es2021: true,
    node: true,
  },
  extends: [
    'eslint:recommended',
    '@typescript-eslint/recommended',
    'plugin:react/recommended',
    'plugin:react-hooks/recommended',
  ],
  parser: '@typescript-eslint/parser',
  parserOptions: {
    ecmaFeatures: {
      jsx: true,
    },
    ecmaVersion: 'latest',
    sourceType: 'module',
  },
  plugins: [
    'react',
    '@typescript-eslint',
    'react-hooks',
  ],
  rules: {
    // Possible Errors
    'no-console': 'warn',
    'no-debugger': 'error',
    'no-duplicate-case': 'error',
    'no-empty': 'error',
    'no-extra-semi': 'error',
    
    // Best Practices
    'curly': 'error',
    'eqeqeq': 'error',
    'no-eval': 'error',
    'no-implied-eval': 'error',
    'no-var': 'error',
    'prefer-const': 'error',
    
    // React specific
    'react/prop-types': 'off', // Using TypeScript
    'react/react-in-jsx-scope': 'off', // React 17+
    'react-hooks/rules-of-hooks': 'error',
    'react-hooks/exhaustive-deps': 'warn',
    
    // TypeScript specific
    '@typescript-eslint/no-unused-vars': 'error',
    '@typescript-eslint/explicit-function-return-type': 'off',
    '@typescript-eslint/explicit-module-boundary-types': 'off',
  },
  settings: {
    react: {
      version: 'detect',
    },
  },
};
```

### Framework-Specific Configurations

#### React + TypeScript
```javascript
// .eslintrc.js
module.exports = {
  extends: [
    'eslint:recommended',
    '@typescript-eslint/recommended',
    'plugin:react/recommended',
    'plugin:react-hooks/recommended',
    'plugin:jsx-a11y/recommended',
  ],
  plugins: [
    'react',
    '@typescript-eslint',
    'react-hooks',
    'jsx-a11y',
  ],
  rules: {
    // React
    'react/jsx-uses-react': 'off',
    'react/react-in-jsx-scope': 'off',
    'react/jsx-props-no-spreading': 'warn',
    'react/jsx-filename-extension': [1, { extensions: ['.tsx'] }],
    
    // TypeScript
    '@typescript-eslint/no-explicit-any': 'warn',
    '@typescript-eslint/prefer-nullish-coalescing': 'error',
    '@typescript-eslint/prefer-optional-chain': 'error',
    
    // Accessibility
    'jsx-a11y/anchor-is-valid': 'error',
    'jsx-a11y/alt-text': 'error',
  },
};
```

#### Next.js
```javascript
// .eslintrc.js
module.exports = {
  extends: [
    'next/core-web-vitals',
    '@typescript-eslint/recommended',
  ],
  rules: {
    '@next/next/no-img-element': 'error',
    '@next/next/no-html-link-for-pages': 'error',
    '@next/next/no-sync-scripts': 'error',
  },
};
```

#### Vue.js
```javascript
// .eslintrc.js
module.exports = {
  extends: [
    'eslint:recommended',
    '@vue/typescript/recommended',
    'plugin:vue/vue3-recommended',
  ],
  plugins: ['vue'],
  rules: {
    'vue/multi-word-component-names': 'off',
    'vue/no-unused-vars': 'error',
    'vue/component-definition-name-casing': ['error', 'PascalCase'],
  },
};
```

## Prettier Setup

### Installation
```bash
# Install Prettier
npm install --save-dev prettier

# Install ESLint integration
npm install --save-dev eslint-config-prettier eslint-plugin-prettier
```

### Basic Configuration
```json
// .prettierrc
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 80,
  "tabWidth": 2,
  "useTabs": false,
  "bracketSpacing": true,
  "bracketSameLine": false,
  "arrowParens": "avoid",
  "endOfLine": "lf",
  "quoteProps": "as-needed",
  "jsxSingleQuote": true,
  "proseWrap": "preserve"
}
```

### Prettier Ignore
```bash
# .prettierignore
node_modules
dist
build
coverage
.next
.nuxt
.vuepress/dist
*.min.js
*.min.css
package-lock.json
yarn.lock
```

### Advanced Prettier Configuration
```javascript
// prettier.config.js
module.exports = {
  semi: true,
  trailingComma: 'es5',
  singleQuote: true,
  printWidth: 80,
  tabWidth: 2,
  useTabs: false,
  
  // Override for specific file types
  overrides: [
    {
      files: '*.json',
      options: {
        printWidth: 200,
      },
    },
    {
      files: '*.md',
      options: {
        proseWrap: 'always',
        printWidth: 70,
      },
    },
    {
      files: ['*.yml', '*.yaml'],
      options: {
        tabWidth: 2,
      },
    },
  ],
};
```

## Integration & Configuration

### ESLint + Prettier Integration
```javascript
// .eslintrc.js
module.exports = {
  extends: [
    'eslint:recommended',
    '@typescript-eslint/recommended',
    'plugin:react/recommended',
    'prettier', // Must be last to override other configs
  ],
  plugins: [
    'prettier',
  ],
  rules: {
    'prettier/prettier': 'error',
  },
};
```

### Package.json Scripts
```json
{
  "scripts": {
    "lint": "eslint . --ext .js,.jsx,.ts,.tsx",
    "lint:fix": "eslint . --ext .js,.jsx,.ts,.tsx --fix",
    "format": "prettier --write .",
    "format:check": "prettier --check .",
    "type-check": "tsc --noEmit",
    "check-all": "npm run type-check && npm run lint && npm run format:check"
  }
}
```

### Lint-staged Configuration
```json
// package.json
{
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

## Custom Rules & Plugins

### Custom ESLint Rules
```javascript
// .eslintrc.js
module.exports = {
  rules: {
    // Enforce consistent naming
    'camelcase': ['error', { properties: 'never' }],
    
    // Prevent specific imports
    'no-restricted-imports': ['error', {
      patterns: ['../*', './*/index'],
      paths: [{
        name: 'lodash',
        message: 'Use lodash-es for better tree shaking'
      }]
    }],
    
    // Custom complexity rules
    'complexity': ['warn', 10],
    'max-depth': ['warn', 4],
    'max-lines-per-function': ['warn', 50],
    
    // Enforce consistent file naming
    'unicorn/filename-case': ['error', {
      case: 'kebabCase',
      ignore: ['^[A-Z].*\\.tsx?$'] // Allow PascalCase for components
    }],
  },
};
```

### Popular ESLint Plugins
```bash
# Install useful plugins
npm install --save-dev \
  eslint-plugin-import \
  eslint-plugin-unicorn \
  eslint-plugin-security \
  eslint-plugin-sonarjs \
  eslint-plugin-promise
```

```javascript
// .eslintrc.js with plugins
module.exports = {
  extends: [
    'plugin:import/recommended',
    'plugin:import/typescript',
    'plugin:unicorn/recommended',
    'plugin:security/recommended',
    'plugin:sonarjs/recommended',
    'plugin:promise/recommended',
  ],
  plugins: [
    'import',
    'unicorn',
    'security',
    'sonarjs',
    'promise',
  ],
  rules: {
    // Import rules
    'import/order': ['error', {
      groups: [
        'builtin',
        'external',
        'internal',
        'parent',
        'sibling',
        'index',
      ],
      'newlines-between': 'always',
    }],
    'import/no-unresolved': 'error',
    'import/no-cycle': 'error',
    
    // Unicorn rules (opinionated)
    'unicorn/prevent-abbreviations': 'off',
    'unicorn/no-null': 'off',
    
    // Security rules
    'security/detect-object-injection': 'warn',
    
    // SonarJS rules
    'sonarjs/cognitive-complexity': ['error', 15],
    'sonarjs/no-duplicate-string': ['error', 3],
  },
};
```

## Team Configurations

### Shared Configuration Package
```bash
# Create shared config package
mkdir eslint-config-company
cd eslint-config-company
npm init -y
```

```javascript
// eslint-config-company/index.js
module.exports = {
  extends: [
    'eslint:recommended',
    '@typescript-eslint/recommended',
    'plugin:react/recommended',
    'plugin:react-hooks/recommended',
    'prettier',
  ],
  plugins: [
    '@typescript-eslint',
    'react',
    'react-hooks',
    'import',
  ],
  rules: {
    // Company-specific rules
    'no-console': 'error',
    'prefer-const': 'error',
    'import/order': ['error', {
      groups: [
        'builtin',
        'external',
        'internal',
        ['parent', 'sibling'],
        'index',
      ],
      'newlines-between': 'always',
      alphabetize: { order: 'asc' },
    }],
    
    // React rules
    'react/prop-types': 'off',
    'react/react-in-jsx-scope': 'off',
    'react-hooks/exhaustive-deps': 'error',
    
    // TypeScript rules
    '@typescript-eslint/no-unused-vars': 'error',
    '@typescript-eslint/explicit-function-return-type': 'off',
  },
  settings: {
    react: { version: 'detect' },
    'import/resolver': {
      typescript: {},
    },
  },
};
```

### Environment-Specific Configurations
```javascript
// .eslintrc.js
module.exports = {
  extends: ['@company/eslint-config'],
  
  overrides: [
    // Test files
    {
      files: ['**/*.test.{js,jsx,ts,tsx}', '**/__tests__/**'],
      extends: ['plugin:testing-library/react'],
      env: { jest: true },
      rules: {
        'testing-library/await-async-query': 'error',
        'testing-library/no-await-sync-query': 'error',
      },
    },
    
    // Configuration files
    {
      files: ['*.config.{js,ts}', '.eslintrc.js'],
      env: { node: true },
      rules: {
        'no-console': 'off',
        '@typescript-eslint/no-var-requires': 'off',
      },
    },
    
    // Stories files
    {
      files: ['*.stories.{js,jsx,ts,tsx}'],
      rules: {
        'import/no-anonymous-default-export': 'off',
      },
    },
  ],
};
```

### Monorepo Configuration
```javascript
// Root .eslintrc.js
module.exports = {
  root: true,
  extends: ['@company/eslint-config'],
  
  overrides: [
    {
      files: ['packages/ui/**/*'],
      extends: ['plugin:storybook/recommended'],
    },
    {
      files: ['packages/api/**/*'],
      env: { node: true, browser: false },
      rules: {
        'no-console': 'off',
      },
    },
  ],
};
```

## Editor Integration

### VS Code Configuration
```json
// .vscode/settings.json
{
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true,
    "source.organizeImports": true
  },
  "eslint.validate": [
    "javascript",
    "javascriptreact",
    "typescript",
    "typescriptreact"
  ],
  "prettier.requireConfig": true,
  "typescript.preferences.organizeImportsIgnoreCase": false
}
```

### VS Code Extensions
```json
// .vscode/extensions.json
{
  "recommendations": [
    "esbenp.prettier-vscode",
    "dbaeumer.vscode-eslint",
    "bradlc.vscode-tailwindcss",
    "ms-vscode.vscode-typescript-next"
  ]
}
```

### WebStorm Configuration
```xml
<!-- .idea/codeStyles/Project.xml -->
<component name="ProjectCodeStyleConfiguration">
  <code_scheme name="Project" version="173">
    <option name="LINE_SEPARATOR" value="&#10;" />
    <JSCodeStyleSettings version="0">
      <option name="USE_SEMICOLON_AFTER_STATEMENT" value="true" />
      <option name="USE_DOUBLE_QUOTES" value="false" />
      <option name="SPACES_WITHIN_OBJECT_LITERAL_BRACES" value="true" />
    </JSCodeStyleSettings>
  </code_scheme>
</component>
```

## Troubleshooting

### Common Issues & Solutions

#### ESLint and Prettier Conflicts
```javascript
// Problem: ESLint and Prettier fighting over formatting
// Solution: Use eslint-config-prettier
module.exports = {
  extends: [
    'eslint:recommended',
    'prettier', // Must be last
  ],
  rules: {
    // Remove conflicting rules
    'quotes': 'off',
    'semi': 'off',
    'indent': 'off',
  },
};
```

#### Performance Issues
```javascript
// .eslintrc.js - Performance optimizations
module.exports = {
  // Cache results
  cache: true,
  cacheLocation: '.eslintcache',
  
  // Ignore patterns
  ignorePatterns: [
    'node_modules/',
    'dist/',
    'build/',
    '*.min.js',
  ],
  
  // Parallel processing
  parserOptions: {
    project: './tsconfig.json',
    tsconfigRootDir: __dirname,
  },
};
```

#### TypeScript Path Mapping
```javascript
// .eslintrc.js
module.exports = {
  settings: {
    'import/resolver': {
      typescript: {
        alwaysTryTypes: true,
        project: './tsconfig.json',
      },
    },
  },
};
```

### Debugging Configuration
```bash
# Debug ESLint configuration
npx eslint --print-config src/index.ts

# Debug Prettier configuration
npx prettier --find-config-path src/index.ts

# Check what files are being processed
npx eslint --debug src/

# Test specific rule
npx eslint --rule 'no-console: error' src/index.ts
```

### Migration Scripts
```javascript
// migrate-eslint.js - Automated migration helper
const fs = require('fs');
const path = require('path');

const oldConfig = {
  extends: ['react-app'],
  rules: {
    'no-console': 'warn',
  },
};

const newConfig = {
  extends: [
    'eslint:recommended',
    '@typescript-eslint/recommended',
    'plugin:react/recommended',
    'prettier',
  ],
  plugins: ['@typescript-eslint', 'react'],
  rules: {
    'no-console': 'error',
    'react/react-in-jsx-scope': 'off',
  },
};

fs.writeFileSync('.eslintrc.js', `module.exports = ${JSON.stringify(newConfig, null, 2)}`);
console.log('ESLint configuration migrated!');
```

## 🔗 Related Notes

- [[Vite Webpack ESBuild]] - Build tool integration
- [[Husky + Lint-Staged]] - Pre-commit hooks setup
- [[TypeScript]] - TypeScript-specific linting rules
- [[Testing]] - Testing-specific ESLint configurations

## 📚 Additional Resources

- [ESLint Documentation](https://eslint.org/docs/)
- [Prettier Documentation](https://prettier.io/docs/)
- [ESLint Rules Reference](https://eslint.org/docs/rules/)
- [Awesome ESLint](https://github.com/dustinspecker/awesome-eslint)
- [Prettier Options](https://prettier.io/docs/en/options.html)

---
*Last updated: January 27, 2025* 