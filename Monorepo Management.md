
>tags: [monorepo, nx, turborepo, yarn-workspaces, architecture, tooling]
>created: 2025-01-27
>updated: 2025-01-27
>related: [[CI-CD & Deployment]], [[npm-yarn]], [[Node.js]], [[Best Practices]]
---

# 🏗️ Monorepo Management

Comprehensive guide to managing multiple packages and applications in a single repository using modern monorepo tools.

## 📋 Quick Summary

A monorepo is a software development strategy where code for many projects is stored in the same repository. Modern monorepo tools provide sophisticated dependency management, build orchestration, and development workflows that make managing multiple related packages efficient and scalable.

**Key Benefits:**
- **Shared Dependencies** - Avoid version conflicts and reduce duplication
- **Atomic Changes** - Make changes across multiple packages in a single commit
- **Simplified Tooling** - Unified linting, testing, and build processes
- **Code Sharing** - Easy to share utilities and components between projects
- **Consistent Standards** - Enforce coding standards across all packages

---

## 🎯 When to Use Monorepos

### ✅ Good Use Cases
- Multiple related applications (web app, mobile app, admin panel)
- Shared component libraries and utilities
- Microservices that need to stay in sync
- Teams that frequently make cross-cutting changes
- Organizations wanting consistent tooling and standards

### ❌ When to Avoid
- Completely unrelated projects
- Teams with very different tech stacks
- Projects with different release cycles and no shared code
- Very large codebases with performance concerns

---

## 🔧 Tool Comparison

| Feature | Nx | Turborepo | Yarn Workspaces |
|---------|----|-----------|-----------------| 
| **Setup Complexity** | Medium | Low | Low |
| **Build Caching** | Advanced | Advanced | Basic |
| **Task Orchestration** | Advanced | Advanced | Basic |
| **Code Generation** | Excellent | None | None |
| **IDE Integration** | Excellent | Good | Basic |
| **Learning Curve** | Steep | Moderate | Gentle |
| **Performance** | Excellent | Excellent | Good |
| **Ecosystem** | Large | Growing | Mature |

---

## 🚀 Nx - The Smart Monorepo

Nx is a powerful build system with first-class monorepo support and advanced features like computation caching and dependency graph analysis.

### Basic Setup

```bash
# Create new Nx workspace
npx create-nx-workspace@latest myworkspace

# Choose preset (React, Angular, Node, etc.)
# Or start with an empty workspace for maximum flexibility

# Navigate to workspace
cd myworkspace
```

### Project Structure

```
myworkspace/
├── apps/
│   ├── web/                 # React web app
│   ├── mobile/              # React Native app
│   └── api/                 # Node.js API
├── libs/
│   ├── shared/
│   │   ├── ui/              # Shared UI components
│   │   ├── utils/           # Shared utilities
│   │   └── types/           # Shared TypeScript types
│   └── feature/
│       ├── auth/            # Authentication feature
│       └── dashboard/       # Dashboard feature
├── tools/                   # Custom tools and scripts
├── nx.json                  # Nx configuration
├── workspace.json           # Workspace configuration
└── package.json
```

### Creating Applications and Libraries

```bash
# Generate a new React app
nx g @nrwl/react:app web

# Generate a new Node.js API
nx g @nrwl/node:app api

# Generate a shared UI library
nx g @nrwl/react:lib shared-ui

# Generate a utility library
nx g @nrwl/js:lib shared-utils

# Generate a feature library
nx g @nrwl/react:lib feature-auth
```

### Essential Nx Commands

```bash
# Run development server
nx serve web

# Build application
nx build web

# Run tests
nx test shared-ui

# Run linting
nx lint web

# Run all tasks for affected projects
nx affected:build
nx affected:test
nx affected:lint

# Show dependency graph
nx graph

# Run specific target for all projects
nx run-many --target=build --all

# Clear cache
nx reset
```

### Nx Configuration (`nx.json`)

```json
{
  "npmScope": "myworkspace",
  "affected": {
    "defaultBase": "main"
  },
  "cli": {
    "defaultCollection": "@nrwl/react"
  },
  "defaultProject": "web",
  "generators": {
    "@nrwl/react": {
      "application": {
        "babel": true,
        "style": "styled-components",
        "linter": "eslint"
      },
      "component": {
        "style": "styled-components"
      },
      "library": {
        "linter": "eslint"
      }
    }
  },
  "targetDefaults": {
    "build": {
      "dependsOn": ["^build"],
      "inputs": ["production", "^production"]
    },
    "test": {
      "inputs": ["default", "^production", "{workspaceRoot}/jest.preset.js"]
    }
  },
  "namedInputs": {
    "default": ["{projectRoot}/**/*", "sharedGlobals"],
    "production": [
      "default",
      "!{projectRoot}/**/?(*.)+(spec|test).[jt]s?(x)?(.snap)",
      "!{projectRoot}/tsconfig.spec.json",
      "!{projectRoot}/jest.config.[jt]s",
      "!{projectRoot}/.eslintrc.json"
    ],
    "sharedGlobals": []
  }
}
```

### Project Configuration (`project.json`)

```json
{
  "name": "web",
  "$schema": "../../node_modules/nx/schemas/project-schema.json",
  "sourceRoot": "apps/web/src",
  "projectType": "application",
  "targets": {
    "build": {
      "executor": "@nrwl/webpack:webpack",
      "outputs": ["{options.outputPath}"],
      "defaultConfiguration": "production",
      "options": {
        "compiler": "babel",
        "outputPath": "dist/apps/web",
        "index": "apps/web/src/index.html",
        "baseHref": "/",
        "main": "apps/web/src/main.tsx",
        "polyfills": "apps/web/src/polyfills.ts",
        "tsConfig": "apps/web/tsconfig.app.json",
        "assets": ["apps/web/src/favicon.ico", "apps/web/src/assets"],
        "styles": ["apps/web/src/styles.css"],
        "scripts": [],
        "webpackConfig": "@nrwl/react/plugins/webpack"
      },
      "configurations": {
        "development": {
          "extractLicenses": false,
          "optimization": false,
          "sourceMap": true,
          "vendorChunk": true
        },
        "production": {
          "fileReplacements": [
            {
              "replace": "apps/web/src/environments/environment.ts",
              "with": "apps/web/src/environments/environment.prod.ts"
            }
          ],
          "optimization": true,
          "outputHashing": "all",
          "sourceMap": false,
          "namedChunks": false,
          "extractLicenses": true,
          "vendorChunk": false
        }
      }
    },
    "serve": {
      "executor": "@nrwl/webpack:dev-server",
      "defaultConfiguration": "development",
      "options": {
        "buildTarget": "web:build",
        "hmr": true
      },
      "configurations": {
        "development": {
          "buildTarget": "web:build:development"
        },
        "production": {
          "buildTarget": "web:build:production",
          "hmr": false
        }
      }
    }
  },
  "tags": ["scope:web", "type:app"]
}
```

### Advanced Nx Features

```bash
# Custom executors for specialized tasks
nx g @nrwl/workspace:executor my-executor

# Custom generators for code scaffolding
nx g @nrwl/workspace:generator my-generator

# Workspace generators for organization-specific patterns
nx g @nrwl/workspace:workspace-generator component

# Module boundary rules (enforce architecture)
# In .eslintrc.json
{
  "rules": {
    "@nrwl/nx/enforce-module-boundaries": [
      "error",
      {
        "enforceBuildableLibDependency": true,
        "allow": [],
        "depConstraints": [
          {
            "sourceTag": "scope:web",
            "onlyDependOnLibsWithTags": ["scope:web", "scope:shared"]
          },
          {
            "sourceTag": "type:feature",
            "onlyDependOnLibsWithTags": ["type:feature", "type:ui", "type:util"]
          }
        ]
      }
    ]
  }
}
```

---

## ⚡ Turborepo - High-Performance Build System

Turborepo is a high-performance build system for JavaScript and TypeScript codebases, optimized for monorepos.

### Basic Setup

```bash
# Create new Turborepo
npx create-turbo@latest

# Or add to existing monorepo
npm install turbo --save-dev

# Initialize turbo.json
npx turbo gen config
```

### Project Structure

```
my-turborepo/
├── apps/
│   ├── web/                 # Next.js app
│   └── docs/                # Documentation site
├── packages/
│   ├── ui/                  # Shared UI components
│   ├── eslint-config-custom/# Shared ESLint config
│   └── tsconfig/            # Shared TypeScript configs
├── turbo.json               # Turborepo configuration
└── package.json
```

### Turborepo Configuration (`turbo.json`)

```json
{
  "$schema": "https://turbo.build/schema.json",
  "globalDependencies": ["**/.env.*local"],
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": [".next/**", "!.next/cache/**", "dist/**"]
    },
    "test": {
      "dependsOn": ["build"],
      "inputs": ["src/**/*.tsx", "src/**/*.ts", "test/**/*.ts", "test/**/*.tsx"]
    },
    "lint": {
      "outputs": []
    },
    "dev": {
      "cache": false,
      "persistent": true
    },
    "clean": {
      "cache": false
    }
  }
}
```

### Package.json Scripts

```json
{
  "scripts": {
    "build": "turbo run build",
    "dev": "turbo run dev",
    "lint": "turbo run lint",
    "test": "turbo run test",
    "clean": "turbo run clean",
    "format": "prettier --write \"**/*.{ts,tsx,md}\"",
    "changeset": "changeset",
    "version-packages": "changeset version",
    "release": "turbo run build --filter=!docs && changeset publish"
  }
}
```

### Essential Turborepo Commands

```bash
# Run build for all packages
turbo run build

# Run dev servers (with parallelization)
turbo run dev

# Run build for specific package and its dependencies
turbo run build --filter=web

# Run tasks in parallel with custom concurrency
turbo run test --concurrency=4

# Force rebuild (ignore cache)
turbo run build --force

# Show what would be run (dry run)
turbo run build --dry-run

# Generate dependency graph
turbo run build --graph

# Prune workspace for deployment
turbo prune --scope=web --docker
```

### Advanced Turborepo Features

```json
// turbo.json - Advanced configuration
{
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**", ".next/**"],
      "env": ["NODE_ENV", "DATABASE_URL"],
      "inputs": ["src/**/*.tsx", "src/**/*.ts", "package.json"]
    },
    "test": {
      "dependsOn": ["build"],
      "inputs": ["src/**/*.tsx", "src/**/*.ts", "test/**/*.ts"],
      "outputs": ["coverage/**"]
    },
    "deploy": {
      "dependsOn": ["build", "test"],
      "cache": false
    }
  },
  "globalEnv": ["NODE_ENV"],
  "globalPassThroughEnv": ["AWS_*", "VERCEL_*"]
}
```

### Docker Integration

```dockerfile
# Dockerfile with Turborepo pruning
FROM node:18-alpine AS base

FROM base AS builder
RUN apk add --no-cache libc6-compat
RUN apk update
WORKDIR /app
RUN npm install -g turbo
COPY . .
RUN turbo prune --scope=web --docker

FROM base AS installer
RUN apk add --no-cache libc6-compat
RUN apk update
WORKDIR /app

COPY .gitignore .gitignore
COPY --from=builder /app/out/json/ .
COPY --from=builder /app/out/package-lock.json ./package-lock.json
RUN npm ci

COPY --from=builder /app/out/full/ .
COPY turbo.json turbo.json
RUN npx turbo run build --filter=web...

FROM base AS runner
WORKDIR /app
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs
USER nextjs

COPY --from=installer /app/apps/web/next.config.js .
COPY --from=installer /app/apps/web/package.json .
COPY --from=installer --chown=nextjs:nodejs /app/apps/web/.next/standalone ./
COPY --from=installer --chown=nextjs:nodejs /app/apps/web/.next/static ./apps/web/.next/static

EXPOSE 3000
ENV PORT 3000
CMD ["node", "apps/web/server.js"]
```

---

## 📦 Yarn Workspaces - Simple and Effective

Yarn Workspaces provide basic monorepo functionality with excellent package management and dependency hoisting.

### Basic Setup

```bash
# Initialize workspace
mkdir my-workspace && cd my-workspace
npm init -y

# Enable workspaces in package.json
```

### Root Package.json Configuration

```json
{
  "name": "my-workspace",
  "private": true,
  "workspaces": [
    "packages/*",
    "apps/*"
  ],
  "scripts": {
    "build": "yarn workspaces foreach -A run build",
    "test": "yarn workspaces foreach -A run test",
    "lint": "yarn workspaces foreach -A run lint",
    "dev": "yarn workspaces foreach -pi run dev",
    "clean": "yarn workspaces foreach -A run clean",
    "typecheck": "yarn workspaces foreach -A run typecheck"
  },
  "devDependencies": {
    "@types/node": "^18.0.0",
    "typescript": "^4.9.0",
    "prettier": "^2.8.0",
    "eslint": "^8.0.0"
  }
}
```

### Project Structure

```
my-workspace/
├── apps/
│   ├── web/
│   │   ├── package.json
│   │   └── src/
│   └── api/
│       ├── package.json
│       └── src/
├── packages/
│   ├── shared-ui/
│   │   ├── package.json
│   │   └── src/
│   ├── shared-utils/
│   │   ├── package.json
│   │   └── src/
│   └── eslint-config/
│       └── package.json
├── package.json
└── yarn.lock
```

### Package Configuration

```json
// packages/shared-ui/package.json
{
  "name": "@myworkspace/shared-ui",
  "version": "1.0.0",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "scripts": {
    "build": "tsc",
    "dev": "tsc --watch",
    "clean": "rm -rf dist",
    "typecheck": "tsc --noEmit"
  },
  "dependencies": {
    "react": "^18.0.0"
  },
  "devDependencies": {
    "@types/react": "^18.0.0",
    "typescript": "^4.9.0"
  },
  "peerDependencies": {
    "react": ">=16.8.0"
  }
}
```

```json
// apps/web/package.json
{
  "name": "@myworkspace/web",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "typecheck": "tsc --noEmit"
  },
  "dependencies": {
    "@myworkspace/shared-ui": "workspace:*",
    "@myworkspace/shared-utils": "workspace:*",
    "next": "^13.0.0",
    "react": "^18.0.0",
    "react-dom": "^18.0.0"
  },
  "devDependencies": {
    "@myworkspace/eslint-config": "workspace:*",
    "@types/node": "^18.0.0",
    "@types/react": "^18.0.0",
    "eslint": "^8.0.0",
    "typescript": "^4.9.0"
  }
}
```

### Essential Yarn Workspaces Commands

```bash
# Install dependencies for all workspaces
yarn install

# Add dependency to specific workspace
yarn workspace @myworkspace/web add lodash

# Add dev dependency to root
yarn add -D -W prettier

# Run script in specific workspace
yarn workspace @myworkspace/web run dev

# Run script in all workspaces
yarn workspaces foreach run build

# Run script in all workspaces in parallel
yarn workspaces foreach -pi run dev

# List all workspaces
yarn workspaces list

# Show workspace dependency tree
yarn workspaces list --tree

# Remove node_modules from all workspaces
yarn workspaces foreach run clean
rm -rf node_modules
yarn install
```

### Advanced Yarn Workspaces Features

```json
// .yarnrc.yml - Yarn configuration
nodeLinker: node-modules
yarnPath: .yarn/releases/yarn-3.6.0.cjs

plugins:
  - path: .yarn/plugins/@yarnpkg/plugin-workspace-tools.cjs
    spec: "@yarnpkg/plugin-workspace-tools"

enableGlobalCache: true
compressionLevel: mixed

packageExtensions:
  "react-scripts@*":
    peerDependencies:
      typescript: "*"
```

### TypeScript Configuration

```json
// tsconfig.json (root)
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["dom", "dom.iterable", "es6"],
    "allowJs": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "baseUrl": ".",
    "paths": {
      "@myworkspace/*": ["packages/*/src"]
    }
  },
  "include": [
    "packages/*/src",
    "apps/*/src"
  ],
  "exclude": [
    "node_modules",
    "dist",
    ".next"
  ]
}
```

---

## 🔄 Version Management and Publishing

### Using Changesets (Recommended)

```bash
# Install changesets
yarn add -D -W @changesets/cli

# Initialize changesets
yarn changeset init

# Create a changeset
yarn changeset

# Version packages
yarn changeset version

# Publish packages
yarn changeset publish
```

### Changeset Configuration (`.changeset/config.json`)

```json
{
  "$schema": "https://unpkg.com/@changesets/config@2.3.0/schema.json",
  "changelog": "@changesets/cli/changelog",
  "commit": false,
  "fixed": [],
  "linked": [],
  "access": "restricted",
  "baseBranch": "main",
  "updateInternalDependencies": "patch",
  "ignore": ["@myworkspace/docs"]
}
```

### GitHub Actions for Automated Publishing

```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    branches:
      - main

concurrency: ${{ github.workflow }}-${{ github.ref }}

jobs:
  release:
    name: Release
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repo
        uses: actions/checkout@v3

      - name: Setup Node.js 18.x
        uses: actions/setup-node@v3
        with:
          node-version: 18.x

      - name: Install Dependencies
        run: yarn

      - name: Build packages
        run: yarn build

      - name: Create Release Pull Request or Publish to npm
        id: changesets
        uses: changesets/action@v1
        with:
          publish: yarn changeset publish
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

---

## 🎯 Best Practices

### 1. **Consistent Tooling**
```json
// Shared configurations across workspace
{
  "devDependencies": {
    "@myworkspace/eslint-config": "workspace:*",
    "@myworkspace/prettier-config": "workspace:*",
    "@myworkspace/tsconfig": "workspace:*"
  }
}
```

### 2. **Clear Dependency Management**
```bash
# Use workspace protocol for internal dependencies
"@myworkspace/shared-ui": "workspace:*"

# Pin external dependencies at root level
# Let workspaces inherit versions
```

### 3. **Effective Caching Strategies**
```json
// Cache outputs and inputs properly
{
  "pipeline": {
    "build": {
      "outputs": ["dist/**", ".next/**"],
      "inputs": ["src/**", "package.json", "tsconfig.json"]
    }
  }
}
```

### 4. **Proper Package Boundaries**
```typescript
// Use barrel exports for clean APIs
// packages/shared-ui/src/index.ts
export { Button } from './Button';
export { Input } from './Input';
export type { ButtonProps, InputProps } from './types';
```

### 5. **Incremental Builds**
```bash
# Only build what changed
nx affected:build
turbo run build --filter=[HEAD^1]
```

---

## 🚨 Common Pitfalls and Solutions

### 1. **Dependency Hell**
```bash
# Problem: Version conflicts between workspaces
# Solution: Use resolutions in root package.json
{
  "resolutions": {
    "react": "^18.0.0",
    "@types/react": "^18.0.0"
  }
}
```

### 2. **Circular Dependencies**
```bash
# Problem: Package A depends on Package B which depends on Package A
# Solution: Extract shared code to a third package
packages/
├── feature-a/     # Depends on shared
├── feature-b/     # Depends on shared  
└── shared/        # No dependencies on features
```

### 3. **Build Order Issues**
```json
// Problem: Building packages in wrong order
// Solution: Properly define dependencies
{
  "pipeline": {
    "build": {
      "dependsOn": ["^build"]  // Build dependencies first
    }
  }
}
```

### 4. **Cache Invalidation**
```bash
# Problem: Stale cache causing issues
# Solution: Clear cache and rebuild
nx reset
turbo run build --force
yarn workspaces foreach run clean
```

---

## 🔗 Related Topics

- [[CI-CD & Deployment]] - Deploying monorepo applications
- [[npm-yarn]] - Package management fundamentals
- [[Node.js]] - Node.js development in monorepos
- [[Best Practices]] - General development best practices

---

## 📚 Additional Resources

- [Nx Documentation](https://nx.dev/)
- [Turborepo Documentation](https://turbo.build/)
- [Yarn Workspaces](https://yarnpkg.com/features/workspaces)
- [Changesets](https://github.com/changesets/changesets)
- [Monorepo Tools Comparison](https://monorepo.tools/)

---

*Last updated: January 27, 2025* 