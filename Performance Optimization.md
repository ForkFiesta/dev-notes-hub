# Performance Optimization - Frontend & Backend

> **Quick Summary**: Performance optimization ensures fast, responsive web applications. Master lazy loading, bundle splitting, Core Web Vitals, DevTools profiling, and server optimization techniques.

## 🚀 Core Web Vitals

### Understanding Web Vitals
```javascript
// Largest Contentful Paint (LCP) - Loading performance
// Target: < 2.5 seconds
const observer = new PerformanceObserver((list) => {
    for (const entry of list.getEntries()) {
        if (entry.entryType === 'largest-contentful-paint') {
            console.log('LCP:', entry.startTime);
        }
    }
});
observer.observe({ entryTypes: ['largest-contentful-paint'] });

// First Input Delay (FID) - Interactivity
// Target: < 100 milliseconds
const fidObserver = new PerformanceObserver((list) => {
    for (const entry of list.getEntries()) {
        if (entry.entryType === 'first-input') {
            console.log('FID:', entry.processingStart - entry.startTime);
        }
    }
});
fidObserver.observe({ entryTypes: ['first-input'] });

// Cumulative Layout Shift (CLS) - Visual stability
// Target: < 0.1
let clsValue = 0;
const clsObserver = new PerformanceObserver((list) => {
    for (const entry of list.getEntries()) {
        if (!entry.hadRecentInput) {
            clsValue += entry.value;
        }
    }
    console.log('CLS:', clsValue);
});
clsObserver.observe({ entryTypes: ['layout-shift'] });
```

### Measuring Performance
```javascript
// Performance API
const navigationTiming = performance.getEntriesByType('navigation')[0];
console.log('DOM Content Loaded:', navigationTiming.domContentLoadedEventEnd - navigationTiming.domContentLoadedEventStart);
console.log('Load Complete:', navigationTiming.loadEventEnd - navigationTiming.loadEventStart);

// Custom performance marks
performance.mark('component-start');
// ... component rendering
performance.mark('component-end');
performance.measure('component-render', 'component-start', 'component-end');

const measures = performance.getEntriesByType('measure');
console.log('Component render time:', measures[0].duration);

// Web Vitals library
import { getCLS, getFID, getFCP, getLCP, getTTFB } from 'web-vitals';

getCLS(console.log);
getFID(console.log);
getFCP(console.log);
getLCP(console.log);
getTTFB(console.log);
```

## 📦 Bundle Optimization

### Code Splitting
```javascript
// Dynamic imports for route-based splitting
import { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';

const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));
const Dashboard = lazy(() => import('./pages/Dashboard'));

function App() {
    return (
        <BrowserRouter>
            <Suspense fallback={<div>Loading...</div>}>
                <Routes>
                    <Route path="/" element={<Home />} />
                    <Route path="/about" element={<About />} />
                    <Route path="/dashboard" element={<Dashboard />} />
                </Routes>
            </Suspense>
        </BrowserRouter>
    );
}

// Component-based splitting
const HeavyComponent = lazy(() => import('./HeavyComponent'));

function MyComponent() {
    const [showHeavy, setShowHeavy] = useState(false);
    
    return (
        <div>
            <button onClick={() => setShowHeavy(true)}>
                Load Heavy Component
            </button>
            {showHeavy && (
                <Suspense fallback={<div>Loading heavy component...</div>}>
                    <HeavyComponent />
                </Suspense>
            )}
        </div>
    );
}

// Webpack bundle splitting
// webpack.config.js
module.exports = {
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
        }
    }
};
```

### Tree Shaking
```javascript
// ✅ Good: Import only what you need
import { debounce } from 'lodash/debounce';
import { format } from 'date-fns/format';

// ❌ Bad: Imports entire library
import _ from 'lodash';
import * as dateFns from 'date-fns';

// ES6 modules for tree shaking
// utils.js
export const formatCurrency = (amount) => {
    return new Intl.NumberFormat('en-US', {
        style: 'currency',
        currency: 'USD'
    }).format(amount);
};

export const formatDate = (date) => {
    return new Intl.DateTimeFormat('en-US').format(date);
};

// main.js - Only imports what's used
import { formatCurrency } from './utils';

// Webpack analyzer to visualize bundle
npm install --save-dev webpack-bundle-analyzer

// package.json
{
    "scripts": {
        "analyze": "npm run build && npx webpack-bundle-analyzer build/static/js/*.js"
    }
}
```

## 🖼️ Image Optimization

### Modern Image Formats
```html
<!-- WebP with fallback -->
<picture>
    <source srcset="image.webp" type="image/webp">
    <source srcset="image.avif" type="image/avif">
    <img src="image.jpg" alt="Description" loading="lazy">
</picture>

<!-- Responsive images -->
<img 
    src="image-800w.jpg"
    srcset="
        image-400w.jpg 400w,
        image-800w.jpg 800w,
        image-1200w.jpg 1200w
    "
    sizes="(max-width: 600px) 400px, (max-width: 1000px) 800px, 1200px"
    alt="Description"
    loading="lazy"
>
```

```javascript
// React image optimization
const OptimizedImage = ({ src, alt, ...props }) => {
    const [imageSrc, setImageSrc] = useState(src);
    const [isLoading, setIsLoading] = useState(true);
    
    useEffect(() => {
        const img = new Image();
        img.onload = () => {
            setImageSrc(img.src);
            setIsLoading(false);
        };
        
        // Try WebP first, fallback to original
        const webpSrc = src.replace(/\.(jpg|jpeg|png)$/, '.webp');
        img.src = webpSrc;
        
        // Fallback if WebP fails
        img.onerror = () => {
            img.src = src;
        };
    }, [src]);
    
    return (
        <div className="image-container">
            {isLoading && <div className="image-placeholder">Loading...</div>}
            <img
                src={imageSrc}
                alt={alt}
                loading="lazy"
                style={{ display: isLoading ? 'none' : 'block' }}
                {...props}
            />
        </div>
    );
};

// Intersection Observer for lazy loading
const useLazyLoading = () => {
    const [isVisible, setIsVisible] = useState(false);
    const ref = useRef();
    
    useEffect(() => {
        const observer = new IntersectionObserver(
            ([entry]) => {
                if (entry.isIntersecting) {
                    setIsVisible(true);
                    observer.disconnect();
                }
            },
            { threshold: 0.1 }
        );
        
        if (ref.current) {
            observer.observe(ref.current);
        }
        
        return () => observer.disconnect();
    }, []);
    
    return [ref, isVisible];
};
```

## ⚡ React Performance

### Memoization
```javascript
// React.memo for component memoization
const ExpensiveComponent = React.memo(({ data, onUpdate }) => {
    return (
        <div>
            {data.map(item => (
                <div key={item.id}>{item.name}</div>
            ))}
        </div>
    );
}, (prevProps, nextProps) => {
    // Custom comparison function
    return prevProps.data.length === nextProps.data.length &&
           prevProps.onUpdate === nextProps.onUpdate;
});

// useMemo for expensive calculations
const ExpensiveCalculation = ({ items }) => {
    const expensiveValue = useMemo(() => {
        return items.reduce((sum, item) => {
            return sum + complexCalculation(item);
        }, 0);
    }, [items]);
    
    return <div>Result: {expensiveValue}</div>;
};

// useCallback for function memoization
const ParentComponent = ({ items }) => {
    const [filter, setFilter] = useState('');
    
    const handleItemClick = useCallback((id) => {
        // Handle click logic
        console.log('Clicked item:', id);
    }, []); // Empty dependency array since logic doesn't depend on props/state
    
    const filteredItems = useMemo(() => {
        return items.filter(item => 
            item.name.toLowerCase().includes(filter.toLowerCase())
        );
    }, [items, filter]);
    
    return (
        <div>
            <input 
                value={filter}
                onChange={(e) => setFilter(e.target.value)}
                placeholder="Filter items..."
            />
            {filteredItems.map(item => (
                <ItemComponent
                    key={item.id}
                    item={item}
                    onClick={handleItemClick}
                />
            ))}
        </div>
    );
};
```

### Virtual Scrolling
```javascript
// React Window for large lists
import { FixedSizeList as List } from 'react-window';

const VirtualizedList = ({ items }) => {
    const Row = ({ index, style }) => (
        <div style={style}>
            <div className="list-item">
                {items[index].name}
            </div>
        </div>
    );
    
    return (
        <List
            height={600}
            itemCount={items.length}
            itemSize={50}
            width="100%"
        >
            {Row}
        </List>
    );
};

// Custom virtual scrolling hook
const useVirtualScroll = (items, itemHeight, containerHeight) => {
    const [scrollTop, setScrollTop] = useState(0);
    
    const visibleStart = Math.floor(scrollTop / itemHeight);
    const visibleEnd = Math.min(
        visibleStart + Math.ceil(containerHeight / itemHeight) + 1,
        items.length
    );
    
    const visibleItems = items.slice(visibleStart, visibleEnd);
    const totalHeight = items.length * itemHeight;
    const offsetY = visibleStart * itemHeight;
    
    return {
        visibleItems,
        totalHeight,
        offsetY,
        onScroll: (e) => setScrollTop(e.target.scrollTop)
    };
};
```

### State Management Optimization
```javascript
// Context optimization with multiple contexts
const UserContext = createContext();
const ThemeContext = createContext();
const DataContext = createContext();

// Split contexts to prevent unnecessary re-renders
const AppProvider = ({ children }) => {
    const [user, setUser] = useState(null);
    const [theme, setTheme] = useState('light');
    const [data, setData] = useState([]);
    
    return (
        <UserContext.Provider value={{ user, setUser }}>
            <ThemeContext.Provider value={{ theme, setTheme }}>
                <DataContext.Provider value={{ data, setData }}>
                    {children}
                </DataContext.Provider>
            </ThemeContext.Provider>
        </UserContext.Provider>
    );
};

// Reducer for complex state
const dataReducer = (state, action) => {
    switch (action.type) {
        case 'ADD_ITEM':
            return {
                ...state,
                items: [...state.items, action.payload]
            };
        case 'UPDATE_ITEM':
            return {
                ...state,
                items: state.items.map(item =>
                    item.id === action.payload.id
                        ? { ...item, ...action.payload.updates }
                        : item
                )
            };
        case 'DELETE_ITEM':
            return {
                ...state,
                items: state.items.filter(item => item.id !== action.payload)
            };
        default:
            return state;
    }
};
```

## 🌐 Network Optimization

### HTTP/2 and Resource Hints
```html
<!-- Preload critical resources -->
<link rel="preload" href="/fonts/main.woff2" as="font" type="font/woff2" crossorigin>
<link rel="preload" href="/css/critical.css" as="style">
<link rel="preload" href="/js/main.js" as="script">

<!-- Prefetch likely next resources -->
<link rel="prefetch" href="/js/dashboard.js">
<link rel="prefetch" href="/api/user-data">

<!-- Preconnect to external domains -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://api.example.com">

<!-- DNS prefetch for external domains -->
<link rel="dns-prefetch" href="//cdn.example.com">
```

### Caching Strategies
```javascript
// Service Worker caching
// sw.js
const CACHE_NAME = 'app-v1';
const urlsToCache = [
    '/',
    '/static/css/main.css',
    '/static/js/main.js'
];

self.addEventListener('install', (event) => {
    event.waitUntil(
        caches.open(CACHE_NAME)
            .then((cache) => cache.addAll(urlsToCache))
    );
});

self.addEventListener('fetch', (event) => {
    event.respondWith(
        caches.match(event.request)
            .then((response) => {
                // Return cached version or fetch from network
                return response || fetch(event.request);
            })
    );
});

// HTTP caching headers
// Express.js example
app.use('/static', express.static('public', {
    maxAge: '1y', // Cache static assets for 1 year
    etag: true,
    lastModified: true
}));

app.get('/api/data', (req, res) => {
    res.set({
        'Cache-Control': 'public, max-age=300', // 5 minutes
        'ETag': generateETag(data)
    });
    res.json(data);
});
```

### API Optimization
```javascript
// Request batching
class APIBatcher {
    constructor(batchSize = 10, delay = 100) {
        this.batchSize = batchSize;
        this.delay = delay;
        this.queue = [];
        this.timeoutId = null;
    }
    
    request(endpoint, options) {
        return new Promise((resolve, reject) => {
            this.queue.push({ endpoint, options, resolve, reject });
            
            if (this.queue.length >= this.batchSize) {
                this.flush();
            } else if (!this.timeoutId) {
                this.timeoutId = setTimeout(() => this.flush(), this.delay);
            }
        });
    }
    
    async flush() {
        if (this.queue.length === 0) return;
        
        const batch = this.queue.splice(0, this.batchSize);
        clearTimeout(this.timeoutId);
        this.timeoutId = null;
        
        try {
            const response = await fetch('/api/batch', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({
                    requests: batch.map(({ endpoint, options }) => ({
                        endpoint,
                        options
                    }))
                })
            });
            
            const results = await response.json();
            
            batch.forEach((item, index) => {
                item.resolve(results[index]);
            });
        } catch (error) {
            batch.forEach(item => item.reject(error));
        }
    }
}

// GraphQL query optimization
const OPTIMIZED_QUERY = gql`
    query GetUserDashboard($userId: ID!) {
        user(id: $userId) {
            id
            name
            avatar
            # Only fetch needed fields
        }
        posts(userId: $userId, limit: 10) {
            id
            title
            excerpt # Don't fetch full content
            createdAt
        }
        notifications(userId: $userId, unreadOnly: true) {
            id
            message
            createdAt
        }
    }
`;
```

## 🔧 DevTools Profiling

### Chrome DevTools Performance
```javascript
// Performance profiling in code
const profileComponent = (WrappedComponent) => {
    return function ProfiledComponent(props) {
        useEffect(() => {
            performance.mark('component-mount-start');
            
            return () => {
                performance.mark('component-mount-end');
                performance.measure(
                    'component-mount',
                    'component-mount-start',
                    'component-mount-end'
                );
                
                const measure = performance.getEntriesByName('component-mount')[0];
                console.log(`${WrappedComponent.name} mount time:`, measure.duration);
            };
        }, []);
        
        return <WrappedComponent {...props} />;
    };
};

// React DevTools Profiler
import { Profiler } from 'react';

function onRenderCallback(id, phase, actualDuration, baseDuration, startTime, commitTime) {
    console.log('Profiler:', {
        id,
        phase,
        actualDuration,
        baseDuration,
        startTime,
        commitTime
    });
}

function App() {
    return (
        <Profiler id="App" onRender={onRenderCallback}>
            <Router>
                <Routes>
                    <Route path="/" element={<Home />} />
                </Routes>
            </Router>
        </Profiler>
    );
}
```

### Memory Leak Detection
```javascript
// Detect memory leaks
const useMemoryMonitor = () => {
    useEffect(() => {
        const checkMemory = () => {
            if (performance.memory) {
                console.log('Memory usage:', {
                    used: Math.round(performance.memory.usedJSHeapSize / 1048576),
                    total: Math.round(performance.memory.totalJSHeapSize / 1048576),
                    limit: Math.round(performance.memory.jsHeapSizeLimit / 1048576)
                });
            }
        };
        
        const interval = setInterval(checkMemory, 5000);
        return () => clearInterval(interval);
    }, []);
};

// Cleanup event listeners
const useEventListener = (event, handler, element = window) => {
    useEffect(() => {
        element.addEventListener(event, handler);
        
        return () => {
            element.removeEventListener(event, handler);
        };
    }, [event, handler, element]);
};
```

## 🖥️ Server-Side Optimization

### Response Optimization
```javascript
// Express.js optimizations
const express = require('express');
const compression = require('compression');
const helmet = require('helmet');

const app = express();

// Enable gzip compression
app.use(compression());

// Security headers
app.use(helmet());

// Response caching middleware
const cache = (duration) => {
    return (req, res, next) => {
        res.set('Cache-Control', `public, max-age=${duration}`);
        next();
    };
};

// API response optimization
app.get('/api/users', cache(300), async (req, res) => {
    try {
        const users = await User.find()
            .select('id name email') // Only select needed fields
            .limit(100) // Limit results
            .lean(); // Return plain objects, not Mongoose documents
        
        res.json({
            data: users,
            meta: {
                count: users.length,
                cached: true
            }
        });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// Database query optimization
const getOptimizedUserData = async (userId) => {
    // Use database indexes
    const user = await User.findById(userId)
        .populate('posts', 'title createdAt') // Only populate needed fields
        .select('-password -__v') // Exclude sensitive/unnecessary fields
        .lean();
    
    return user;
};
```

### CDN and Asset Optimization
```javascript
// Webpack production optimizations
// webpack.prod.js
const path = require('path');
const TerserPlugin = require('terser-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin');

module.exports = {
    mode: 'production',
    optimization: {
        minimize: true,
        minimizer: [
            new TerserPlugin({
                terserOptions: {
                    compress: {
                        drop_console: true, // Remove console.log in production
                    },
                },
            }),
            new CssMinimizerPlugin(),
        ],
        splitChunks: {
            chunks: 'all',
            cacheGroups: {
                vendor: {
                    test: /[\\/]node_modules[\\/]/,
                    name: 'vendors',
                    chunks: 'all',
                },
            },
        },
    },
    plugins: [
        new MiniCssExtractPlugin({
            filename: '[name].[contenthash].css',
        }),
    ],
    output: {
        filename: '[name].[contenthash].js',
        path: path.resolve(__dirname, 'dist'),
        clean: true,
    },
};
```

## 🚀 Best Practices

### Performance Budget
```javascript
// webpack-bundle-analyzer configuration
// package.json
{
    "scripts": {
        "analyze": "npm run build && npx webpack-bundle-analyzer build/static/js/*.js",
        "size-limit": "size-limit"
    },
    "size-limit": [
        {
            "path": "build/static/js/*.js",
            "limit": "250 KB"
        },
        {
            "path": "build/static/css/*.css",
            "limit": "50 KB"
        }
    ]
}

// Performance monitoring
const performanceObserver = new PerformanceObserver((list) => {
    for (const entry of list.getEntries()) {
        // Send metrics to analytics
        analytics.track('performance', {
            name: entry.name,
            duration: entry.duration,
            type: entry.entryType
        });
    }
});

performanceObserver.observe({ entryTypes: ['measure', 'navigation'] });
```

### Critical CSS
```css
/* Critical CSS - inline in <head> */
body {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
    margin: 0;
    padding: 0;
}

.header {
    background: #fff;
    border-bottom: 1px solid #eee;
    padding: 1rem;
}

.main-content {
    max-width: 1200px;
    margin: 0 auto;
    padding: 2rem 1rem;
}

/* Non-critical CSS - load asynchronously */
.footer,
.sidebar,
.modal {
    /* Styles loaded later */
}
```

## 🔧 Quick Reference

### Performance Checklist
```markdown
## Frontend Performance
- [ ] Enable gzip/brotli compression
- [ ] Optimize images (WebP, lazy loading)
- [ ] Implement code splitting
- [ ] Use React.memo, useMemo, useCallback appropriately
- [ ] Minimize bundle size (tree shaking)
- [ ] Implement virtual scrolling for large lists
- [ ] Use service workers for caching
- [ ] Optimize fonts (preload, font-display)

## Backend Performance
- [ ] Database query optimization
- [ ] API response caching
- [ ] Request batching
- [ ] Connection pooling
- [ ] CDN for static assets
- [ ] Server-side compression
- [ ] HTTP/2 implementation
- [ ] Database indexing

## Monitoring
- [ ] Core Web Vitals tracking
- [ ] Performance budgets
- [ ] Real User Monitoring (RUM)
- [ ] Synthetic monitoring
- [ ] Error tracking
- [ ] Memory leak detection
```

### Performance Tools
```bash
# Lighthouse CLI
npm install -g lighthouse
lighthouse https://example.com --output html --output-path ./report.html

# WebPageTest
# Use webpagetest.org for detailed analysis

# Bundle analyzers
npm install --save-dev webpack-bundle-analyzer
npm install --save-dev source-map-explorer

# Performance monitoring
npm install web-vitals
npm install @sentry/browser # For error and performance tracking
```

---

## 🔗 Related Topics
- [[React]] - Component optimization strategies
- [[JavaScript]] - Core performance concepts
- [[CSS]] - Styling performance
- [[Testing]] - Performance testing
- [[CI/CD & Deployment]] - Build optimization 