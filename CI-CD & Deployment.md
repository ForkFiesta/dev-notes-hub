# CI/CD & Deployment - Automated Workflows

> **Quick Summary**: CI/CD automates testing, building, and deployment processes. Master GitHub Actions, Netlify/Vercel workflows, environment management, and deployment strategies for reliable software delivery.

## 🔄 GitHub Actions

### Basic Workflow Setup
```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        node-version: [16.x, 18.x, 20.x]
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
      
    - name: Setup Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v4
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'
        
    - name: Install dependencies
      run: npm ci
      
    - name: Run linting
      run: npm run lint
      
    - name: Run tests
      run: npm run test:ci
      
    - name: Run build
      run: npm run build
      
    - name: Upload coverage reports
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage/lcov.info
```

### Advanced Workflow Features
```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [ main ]
    tags: [ 'v*' ]

env:
  NODE_VERSION: '18.x'
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    
    outputs:
      version: ${{ steps.version.outputs.version }}
      
    steps:
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0 # Full history for semantic versioning
        
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: ${{ env.NODE_VERSION }}
        cache: 'npm'
        
    - name: Install dependencies
      run: npm ci
      
    - name: Run tests with coverage
      run: npm run test:coverage
      
    - name: Build application
      run: npm run build
      env:
        NODE_ENV: production
        REACT_APP_API_URL: ${{ secrets.PROD_API_URL }}
        
    - name: Generate version
      id: version
      run: |
        if [[ $GITHUB_REF == refs/tags/* ]]; then
          echo "version=${GITHUB_REF#refs/tags/}" >> $GITHUB_OUTPUT
        else
          echo "version=latest" >> $GITHUB_OUTPUT
        fi
        
    - name: Upload build artifacts
      uses: actions/upload-artifact@v3
      with:
        name: build-files
        path: build/
        retention-days: 30

  security-scan:
    runs-on: ubuntu-latest
    needs: build-and-test
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Run security audit
      run: npm audit --audit-level high
      
    - name: Run Snyk security scan
      uses: snyk/actions/node@master
      env:
        SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}

  deploy-staging:
    runs-on: ubuntu-latest
    needs: [build-and-test, security-scan]
    if: github.ref == 'refs/heads/main'
    
    environment:
      name: staging
      url: https://staging.example.com
      
    steps:
    - name: Download build artifacts
      uses: actions/download-artifact@v3
      with:
        name: build-files
        path: build/
        
    - name: Deploy to staging
      run: |
        # Deploy to staging environment
        echo "Deploying to staging..."
        
    - name: Run smoke tests
      run: |
        # Run basic smoke tests against staging
        curl -f https://staging.example.com/health || exit 1

  deploy-production:
    runs-on: ubuntu-latest
    needs: [deploy-staging]
    if: startsWith(github.ref, 'refs/tags/')
    
    environment:
      name: production
      url: https://example.com
      
    steps:
    - name: Download build artifacts
      uses: actions/download-artifact@v3
      with:
        name: build-files
        path: build/
        
    - name: Deploy to production
      run: |
        # Deploy to production environment
        echo "Deploying to production..."
        
    - name: Create GitHub release
      uses: actions/create-release@v1
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      with:
        tag_name: ${{ needs.build-and-test.outputs.version }}
        release_name: Release ${{ needs.build-and-test.outputs.version }}
        draft: false
        prerelease: false
```

### Reusable Workflows
```yaml
# .github/workflows/reusable-test.yml
name: Reusable Test Workflow

on:
  workflow_call:
    inputs:
      node-version:
        required: false
        type: string
        default: '18.x'
      environment:
        required: false
        type: string
        default: 'test'
    secrets:
      API_KEY:
        required: true

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: ${{ inputs.node-version }}
        cache: 'npm'
        
    - name: Install dependencies
      run: npm ci
      
    - name: Run tests
      run: npm test
      env:
        NODE_ENV: ${{ inputs.environment }}
        API_KEY: ${{ secrets.API_KEY }}

# Usage in another workflow
# .github/workflows/main.yml
name: Main Workflow

on: [push, pull_request]

jobs:
  call-reusable-workflow:
    uses: ./.github/workflows/reusable-test.yml
    with:
      node-version: '20.x'
      environment: 'staging'
    secrets:
      API_KEY: ${{ secrets.STAGING_API_KEY }}
```

## 🚀 Netlify Deployment

### Netlify Configuration
```toml
# netlify.toml
[build]
  publish = "build"
  command = "npm run build"
  
[build.environment]
  NODE_VERSION = "18"
  NPM_FLAGS = "--prefix=/dev/null"

# Branch-specific builds
[context.production]
  command = "npm run build:prod"
  
[context.deploy-preview]
  command = "npm run build:preview"
  
[context.branch-deploy]
  command = "npm run build:dev"

# Redirects and rewrites
[[redirects]]
  from = "/api/*"
  to = "https://api.example.com/:splat"
  status = 200
  
[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200

# Headers
[[headers]]
  for = "/*"
  [headers.values]
    X-Frame-Options = "DENY"
    X-XSS-Protection = "1; mode=block"
    X-Content-Type-Options = "nosniff"
    Referrer-Policy = "strict-origin-when-cross-origin"

[[headers]]
  for = "/static/*"
  [headers.values]
    Cache-Control = "public, max-age=31536000, immutable"

# Functions
[functions]
  directory = "netlify/functions"
  
# Edge functions
[edge_functions]
  directory = "netlify/edge-functions"
```

### Netlify Functions
```javascript
// netlify/functions/api.js
exports.handler = async (event, context) => {
    const { httpMethod, path, queryStringParameters, body } = event;
    
    // CORS headers
    const headers = {
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Headers': 'Content-Type',
        'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE'
    };
    
    // Handle preflight requests
    if (httpMethod === 'OPTIONS') {
        return {
            statusCode: 200,
            headers,
            body: ''
        };
    }
    
    try {
        switch (httpMethod) {
            case 'GET':
                return {
                    statusCode: 200,
                    headers,
                    body: JSON.stringify({ message: 'Hello from Netlify Functions!' })
                };
                
            case 'POST':
                const data = JSON.parse(body);
                // Process data
                return {
                    statusCode: 201,
                    headers,
                    body: JSON.stringify({ success: true, data })
                };
                
            default:
                return {
                    statusCode: 405,
                    headers,
                    body: JSON.stringify({ error: 'Method not allowed' })
                };
        }
    } catch (error) {
        return {
            statusCode: 500,
            headers,
            body: JSON.stringify({ error: error.message })
        };
    }
};

// netlify/edge-functions/auth.js
export default async (request, context) => {
    const url = new URL(request.url);
    
    // Check authentication
    const authHeader = request.headers.get('authorization');
    if (!authHeader) {
        return new Response('Unauthorized', { status: 401 });
    }
    
    // Continue to origin
    return context.next();
};

export const config = {
    path: "/protected/*"
};
```

## ⚡ Vercel Deployment

### Vercel Configuration
```json
// vercel.json
{
  "version": 2,
  "builds": [
    {
      "src": "package.json",
      "use": "@vercel/static-build",
      "config": {
        "distDir": "build"
      }
    }
  ],
  "routes": [
    {
      "src": "/api/(.*)",
      "dest": "/api/$1"
    },
    {
      "src": "/(.*)",
      "dest": "/index.html"
    }
  ],
  "headers": [
    {
      "source": "/api/(.*)",
      "headers": [
        {
          "key": "Access-Control-Allow-Origin",
          "value": "*"
        },
        {
          "key": "Access-Control-Allow-Methods",
          "value": "GET, POST, PUT, DELETE, OPTIONS"
        }
      ]
    }
  ],
  "env": {
    "NODE_ENV": "production"
  },
  "build": {
    "env": {
      "REACT_APP_API_URL": "@api-url"
    }
  }
}
```

### Vercel API Routes
```javascript
// api/hello.js
export default function handler(req, res) {
    const { method, query, body } = req;
    
    // Set CORS headers
    res.setHeader('Access-Control-Allow-Origin', '*');
    res.setHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE');
    res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization');
    
    if (method === 'OPTIONS') {
        res.status(200).end();
        return;
    }
    
    switch (method) {
        case 'GET':
            res.status(200).json({ 
                message: 'Hello from Vercel!',
                query 
            });
            break;
            
        case 'POST':
            res.status(201).json({ 
                message: 'Data received',
                data: body 
            });
            break;
            
        default:
            res.setHeader('Allow', ['GET', 'POST']);
            res.status(405).end(`Method ${method} Not Allowed`);
    }
}

// api/users/[id].js - Dynamic routes
export default function handler(req, res) {
    const { query: { id }, method } = req;
    
    switch (method) {
        case 'GET':
            // Get user by ID
            res.status(200).json({ id, name: `User ${id}` });
            break;
            
        case 'PUT':
            // Update user
            res.status(200).json({ id, updated: true });
            break;
            
        case 'DELETE':
            // Delete user
            res.status(204).end();
            break;
            
        default:
            res.setHeader('Allow', ['GET', 'PUT', 'DELETE']);
            res.status(405).end(`Method ${method} Not Allowed`);
    }
}
```

### Vercel Edge Functions
```javascript
// middleware.js
import { NextResponse } from 'next/server';

export function middleware(request) {
    // Add custom header
    const response = NextResponse.next();
    response.headers.set('x-custom-header', 'my-value');
    
    // Redirect based on conditions
    if (request.nextUrl.pathname.startsWith('/old-path')) {
        return NextResponse.redirect(new URL('/new-path', request.url));
    }
    
    // Rewrite URLs
    if (request.nextUrl.pathname.startsWith('/api-proxy')) {
        return NextResponse.rewrite(new URL('/api/external', request.url));
    }
    
    return response;
}

export const config = {
    matcher: [
        '/((?!api|_next/static|_next/image|favicon.ico).*)',
    ],
};
```

## 🔐 Environment Management

### Environment Variables
```bash
# .env.local (local development)
REACT_APP_API_URL=http://localhost:3001
REACT_APP_ENVIRONMENT=development
REACT_APP_DEBUG=true

# .env.staging
REACT_APP_API_URL=https://api-staging.example.com
REACT_APP_ENVIRONMENT=staging
REACT_APP_DEBUG=false

# .env.production
REACT_APP_API_URL=https://api.example.com
REACT_APP_ENVIRONMENT=production
REACT_APP_DEBUG=false
```

```javascript
// config/environment.js
const config = {
  development: {
    apiUrl: process.env.REACT_APP_API_URL || 'http://localhost:3001',
    debug: true,
    logLevel: 'debug'
  },
  staging: {
    apiUrl: process.env.REACT_APP_API_URL,
    debug: false,
    logLevel: 'warn'
  },
  production: {
    apiUrl: process.env.REACT_APP_API_URL,
    debug: false,
    logLevel: 'error'
  }
};

const environment = process.env.REACT_APP_ENVIRONMENT || 'development';

export default config[environment];

// Usage
import config from './config/environment';

const api = axios.create({
  baseURL: config.apiUrl,
  timeout: 10000
});
```

### Secrets Management
```yaml
# GitHub Actions secrets usage
- name: Deploy to production
  env:
    DATABASE_URL: ${{ secrets.DATABASE_URL }}
    API_KEY: ${{ secrets.API_KEY }}
    JWT_SECRET: ${{ secrets.JWT_SECRET }}
  run: |
    echo "Deploying with secure environment variables"
    
# Conditional secrets based on environment
- name: Set environment variables
  run: |
    if [ "${{ github.ref }}" == "refs/heads/main" ]; then
      echo "API_URL=${{ secrets.PROD_API_URL }}" >> $GITHUB_ENV
    else
      echo "API_URL=${{ secrets.STAGING_API_URL }}" >> $GITHUB_ENV
    fi
```

## 🏗️ Build Optimization

### Multi-stage Docker Builds
```dockerfile
# Dockerfile
# Build stage
FROM node:18-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./
RUN npm ci --only=production

# Copy source code
COPY . .

# Build application
RUN npm run build

# Production stage
FROM nginx:alpine AS production

# Copy built assets
COPY --from=builder /app/build /usr/share/nginx/html

# Copy nginx configuration
COPY nginx.conf /etc/nginx/nginx.conf

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost/ || exit 1

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### Build Caching Strategies
```yaml
# GitHub Actions with caching
- name: Cache node modules
  uses: actions/cache@v3
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-

- name: Cache build output
  uses: actions/cache@v3
  with:
    path: build
    key: ${{ runner.os }}-build-${{ github.sha }}
    restore-keys: |
      ${{ runner.os }}-build-

# Docker layer caching
- name: Set up Docker Buildx
  uses: docker/setup-buildx-action@v2

- name: Build and push Docker image
  uses: docker/build-push-action@v4
  with:
    context: .
    push: true
    tags: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest
    cache-from: type=gha
    cache-to: type=gha,mode=max
```

### Webpack Build Optimization
```javascript
// webpack.prod.js
const path = require('path');
const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer');
const CompressionPlugin = require('compression-webpack-plugin');

module.exports = {
  mode: 'production',
  
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all',
        },
        common: {
          name: 'common',
          minChunks: 2,
          chunks: 'all',
          enforce: true,
        }
      }
    },
    
    // Tree shaking
    usedExports: true,
    sideEffects: false,
    
    // Minimize
    minimize: true
  },
  
  plugins: [
    // Gzip compression
    new CompressionPlugin({
      algorithm: 'gzip',
      test: /\.(js|css|html|svg)$/,
      threshold: 8192,
      minRatio: 0.8
    }),
    
    // Bundle analysis (only when needed)
    process.env.ANALYZE && new BundleAnalyzerPlugin({
      analyzerMode: 'static',
      openAnalyzer: false,
      reportFilename: 'bundle-report.html'
    })
  ].filter(Boolean),
  
  // Output optimization
  output: {
    filename: '[name].[contenthash].js',
    chunkFilename: '[name].[contenthash].chunk.js',
    path: path.resolve(__dirname, 'build'),
    clean: true
  }
};
```

## 🔄 Deployment Strategies

### Blue-Green Deployment
```yaml
# Blue-Green deployment workflow
name: Blue-Green Deployment

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
      
    - name: Build application
      run: npm run build
      
    - name: Deploy to green environment
      run: |
        # Deploy to green environment
        kubectl apply -f k8s/green-deployment.yaml
        
    - name: Wait for green deployment
      run: |
        kubectl rollout status deployment/app-green
        
    - name: Run health checks
      run: |
        # Health check against green environment
        curl -f https://green.example.com/health
        
    - name: Switch traffic to green
      run: |
        # Update service to point to green deployment
        kubectl patch service app-service -p '{"spec":{"selector":{"version":"green"}}}'
        
    - name: Cleanup blue environment
      run: |
        # Remove old blue deployment
        kubectl delete deployment app-blue --ignore-not-found=true
        
    - name: Rename green to blue
      run: |
        # Rename for next deployment
        kubectl patch deployment app-green -p '{"metadata":{"name":"app-blue"},"spec":{"selector":{"matchLabels":{"version":"blue"}},"template":{"metadata":{"labels":{"version":"blue"}}}}}'
```

### Canary Deployment
```yaml
# Canary deployment with gradual traffic shifting
name: Canary Deployment

on:
  push:
    branches: [ main ]

jobs:
  canary-deploy:
    runs-on: ubuntu-latest
    
    steps:
    - name: Deploy canary version
      run: |
        # Deploy canary with 10% traffic
        kubectl apply -f k8s/canary-deployment.yaml
        kubectl patch service app-service -p '{"spec":{"selector":{"version":"canary"}}}'
        
    - name: Monitor canary metrics
      run: |
        # Monitor error rates, response times
        sleep 300 # Wait 5 minutes
        
    - name: Increase canary traffic
      run: |
        # Increase to 50% traffic
        kubectl scale deployment app-canary --replicas=5
        kubectl scale deployment app-stable --replicas=5
        
    - name: Full canary rollout
      run: |
        # If metrics are good, complete rollout
        kubectl scale deployment app-canary --replicas=10
        kubectl scale deployment app-stable --replicas=0
        
    - name: Promote canary to stable
      run: |
        # Replace stable with canary
        kubectl delete deployment app-stable
        kubectl patch deployment app-canary -p '{"metadata":{"name":"app-stable"}}'
```

### Rolling Deployment
```yaml
# Rolling deployment configuration
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
spec:
  replicas: 10
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 2
      maxSurge: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: app
        image: myapp:latest
        ports:
        - containerPort: 3000
        readinessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 10
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
```

## 📊 Monitoring and Alerting

### Health Checks
```javascript
// Express.js health check endpoint
app.get('/health', (req, res) => {
    const healthCheck = {
        uptime: process.uptime(),
        message: 'OK',
        timestamp: Date.now(),
        checks: {
            database: 'OK',
            redis: 'OK',
            external_api: 'OK'
        }
    };
    
    try {
        // Check database connection
        // Check Redis connection
        // Check external API
        
        res.status(200).json(healthCheck);
    } catch (error) {
        healthCheck.message = error.message;
        healthCheck.checks.database = 'ERROR';
        res.status(503).json(healthCheck);
    }
});

// React health check component
const HealthCheck = () => {
    const [health, setHealth] = useState(null);
    
    useEffect(() => {
        const checkHealth = async () => {
            try {
                const response = await fetch('/api/health');
                const data = await response.json();
                setHealth(data);
            } catch (error) {
                setHealth({ status: 'ERROR', message: error.message });
            }
        };
        
        checkHealth();
        const interval = setInterval(checkHealth, 30000);
        
        return () => clearInterval(interval);
    }, []);
    
    return (
        <div className={`health-status ${health?.message === 'OK' ? 'healthy' : 'unhealthy'}`}>
            Status: {health?.message || 'Checking...'}
        </div>
    );
};
```

### Deployment Notifications
```yaml
# Slack notification on deployment
- name: Notify Slack on success
  if: success()
  uses: 8398a7/action-slack@v3
  with:
    status: success
    text: '🚀 Deployment to production successful!'
    webhook_url: ${{ secrets.SLACK_WEBHOOK }}

- name: Notify Slack on failure
  if: failure()
  uses: 8398a7/action-slack@v3
  with:
    status: failure
    text: '❌ Deployment to production failed!'
    webhook_url: ${{ secrets.SLACK_WEBHOOK }}

# Email notification
- name: Send email notification
  uses: dawidd6/action-send-mail@v3
  with:
    server_address: smtp.gmail.com
    server_port: 465
    username: ${{ secrets.EMAIL_USERNAME }}
    password: ${{ secrets.EMAIL_PASSWORD }}
    subject: 'Deployment Status: ${{ job.status }}'
    body: |
      Deployment to production has ${{ job.status }}.
      
      Repository: ${{ github.repository }}
      Commit: ${{ github.sha }}
      Author: ${{ github.actor }}
    to: team@example.com
    from: noreply@example.com
```

## 🚀 Best Practices

### Security in CI/CD
```yaml
# Security scanning in pipeline
- name: Run security audit
  run: npm audit --audit-level high
  
- name: Scan for secrets
  uses: trufflesecurity/trufflehog@main
  with:
    path: ./
    base: main
    head: HEAD

- name: Container security scan
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: 'myapp:latest'
    format: 'sarif'
    output: 'trivy-results.sarif'

- name: Upload Trivy scan results
  uses: github/codeql-action/upload-sarif@v2
  with:
    sarif_file: 'trivy-results.sarif'
```

### Performance Monitoring
```javascript
// Performance monitoring in production
const performanceMonitor = {
    trackDeployment: (version) => {
        // Track deployment metrics
        analytics.track('deployment', {
            version,
            timestamp: Date.now(),
            environment: process.env.NODE_ENV
        });
    },
    
    trackError: (error, context) => {
        // Track errors with context
        errorReporting.captureException(error, {
            tags: {
                deployment_version: process.env.REACT_APP_VERSION,
                environment: process.env.NODE_ENV
            },
            extra: context
        });
    },
    
    trackPerformance: (metric, value) => {
        // Track performance metrics
        metrics.histogram(metric, value, {
            environment: process.env.NODE_ENV,
            version: process.env.REACT_APP_VERSION
        });
    }
};

// Usage in React app
useEffect(() => {
    performanceMonitor.trackDeployment(process.env.REACT_APP_VERSION);
}, []);
```

## 🔧 Quick Reference

### Common CI/CD Commands
```bash
# GitHub CLI for workflow management
gh workflow list
gh workflow run ci.yml
gh workflow view ci.yml

# Netlify CLI
netlify deploy --prod
netlify env:list
netlify functions:list

# Vercel CLI
vercel --prod
vercel env ls
vercel logs

# Docker commands
docker build -t myapp:latest .
docker push myapp:latest
docker run -p 3000:3000 myapp:latest

# Kubernetes deployment
kubectl apply -f deployment.yaml
kubectl rollout status deployment/myapp
kubectl get pods
kubectl logs -f deployment/myapp
```

### Environment Variables Checklist
```markdown
## Required Environment Variables
- [ ] API URLs (dev, staging, prod)
- [ ] Database connection strings
- [ ] Authentication secrets (JWT, OAuth)
- [ ] Third-party API keys
- [ ] CDN URLs
- [ ] Feature flags
- [ ] Monitoring/analytics keys
- [ ] Email service credentials
- [ ] Storage service credentials

## Security Considerations
- [ ] Never commit secrets to version control
- [ ] Use different secrets for each environment
- [ ] Rotate secrets regularly
- [ ] Use secret management services
- [ ] Audit secret access
- [ ] Encrypt secrets at rest
```

---

## 🔗 Related Topics
- [[Git]] - Version control workflows
- [[Testing]] - Automated testing in pipelines
- [[Performance Optimization]] - Build optimization
- [[Node.js]] - Server deployment strategies
- [[React]] - Frontend deployment considerations 