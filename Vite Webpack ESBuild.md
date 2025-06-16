---
tags: [tooling, bundler, vite, webpack, esbuild, build, performance]
created: 2025-01-27
updated: 2025-01-27
---

# ⚡ Vite / Webpack / ESBuild

> **Summary**: Comprehensive comparison and setup guide for modern JavaScript build tools - when to use Vite, Webpack, or ESBuild, with practical configurations and performance considerations.

## 📋 Table of Contents

- [Build Tool Comparison](#build-tool-comparison)
- [Vite](#vite)
- [Webpack](#webpack)
- [ESBuild](#esbuild)
- [Performance Comparison](#performance-comparison)
- [Migration Strategies](#migration-strategies)

## Build Tool Comparison

### When to Use Each Tool

| Tool | Best For | Strengths | Weaknesses |
|------|----------|-----------|------------|
| **Vite** | Modern web apps, Vue/React projects | Fast dev server, simple config, modern defaults | Newer ecosystem, some plugin limitations |
| **Webpack** | Complex builds, legacy support, micro-frontends | Mature ecosystem, highly configurable, extensive plugins | Complex config, slower dev builds |
| **ESBuild** | Libraries, simple apps, CI builds | Extremely fast, simple API, small binary | Limited plugins, less configuration options |

### Feature Matrix

| Feature | Vite | Webpack | ESBuild |
|---------|------|---------|---------|
| Dev Server Speed | ⚡⚡⚡ | ⚡ | ⚡⚡⚡ |
| Build Speed | ⚡⚡ | ⚡ | ⚡⚡⚡ |
| Plugin Ecosystem | ⚡⚡ | ⚡⚡⚡ | ⚡ |
| Configuration | ⚡⚡⚡ | ⚡ | ⚡⚡⚡ |
| Bundle Splitting | ⚡⚡ | ⚡⚡⚡ | ⚡⚡ |
| Tree Shaking | ⚡⚡⚡ | ⚡⚡⚡ | ⚡⚡⚡ |

## Vite

### Basic Setup
```bash
# Create new project
npm create vite@latest my-app
cd my-app
npm install

# Or with specific template
npm create vite@latest my-react-app -- --template react-ts
npm create vite@latest my-vue-app -- --template vue
```

### Vite Configuration
```javascript
// vite.config.js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import { resolve } from 'path'

export default defineConfig({
  plugins: [react()],
  
  // Development server
  server: {
    port: 3000,
    open: true,
    cors: true,
    proxy: {
      '/api': {
        target: 'http://localhost:8000',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, '')
      }
    }
  },
  
  // Build configuration
  build: {
    outDir: 'dist',
    sourcemap: true,
    minify: 'esbuild',
    target: 'es2015',
    rollupOptions: {
      input: {
        main: resolve(__dirname, 'index.html'),
        admin: resolve(__dirname, 'admin.html')
      },
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          utils: ['lodash', 'date-fns']
        }
      }
    }
  },
  
  // Path resolution
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src'),
      '@components': resolve(__dirname, 'src/components'),
      '@utils': resolve(__dirname, 'src/utils')
    }
  },
  
  // Environment variables
  define: {
    __APP_VERSION__: JSON.stringify(process.env.npm_package_version),
  },
  
  // CSS configuration
  css: {
    preprocessorOptions: {
      scss: {
        additionalData: `@import "@/styles/variables.scss";`
      }
    },
    modules: {
      localsConvention: 'camelCase'
    }
  }
})
```

### Vite Plugins
```javascript
// Advanced plugin configuration
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import { visualizer } from 'rollup-plugin-visualizer'
import { VitePWA } from 'vite-plugin-pwa'

export default defineConfig({
  plugins: [
    react(),
    
    // Bundle analyzer
    visualizer({
      filename: 'dist/stats.html',
      open: true,
      gzipSize: true
    }),
    
    // PWA support
    VitePWA({
      registerType: 'autoUpdate',
      workbox: {
        globPatterns: ['**/*.{js,css,html,ico,png,svg}']
      },
      manifest: {
        name: 'My App',
        short_name: 'MyApp',
        description: 'My Awesome App description',
        theme_color: '#ffffff',
        icons: [
          {
            src: 'pwa-192x192.png',
            sizes: '192x192',
            type: 'image/png'
          }
        ]
      }
    })
  ]
})
```

### Vite Environment Configuration
```javascript
// vite.config.js - Environment-specific configs
import { defineConfig, loadEnv } from 'vite'

export default defineConfig(({ command, mode }) => {
  const env = loadEnv(mode, process.cwd(), '')
  
  return {
    plugins: [react()],
    
    define: {
      __API_URL__: JSON.stringify(env.VITE_API_URL),
    },
    
    build: {
      minify: command === 'build' ? 'esbuild' : false,
      sourcemap: mode === 'development'
    },
    
    server: {
      port: parseInt(env.VITE_PORT) || 3000
    }
  }
})
```

## Webpack

### Basic Webpack Setup
```javascript
// webpack.config.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

module.exports = {
  entry: './src/index.js',
  
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].[contenthash].js',
    clean: true,
    publicPath: '/'
  },
  
  module: {
    rules: [
      // JavaScript/TypeScript
      {
        test: /\.(js|jsx|ts|tsx)$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: [
              '@babel/preset-env',
              '@babel/preset-react',
              '@babel/preset-typescript'
            ]
          }
        }
      },
      
      // CSS/SCSS
      {
        test: /\.(css|scss)$/,
        use: [
          MiniCssExtractPlugin.loader,
          'css-loader',
          'sass-loader'
        ]
      },
      
      // Assets
      {
        test: /\.(png|jpg|jpeg|gif|svg)$/,
        type: 'asset/resource',
        generator: {
          filename: 'images/[hash][ext][query]'
        }
      },
      
      // Fonts
      {
        test: /\.(woff|woff2|eot|ttf|otf)$/,
        type: 'asset/resource',
        generator: {
          filename: 'fonts/[hash][ext][query]'
        }
      }
    ]
  },
  
  plugins: [
    new HtmlWebpackPlugin({
      template: './public/index.html'
    }),
    
    new MiniCssExtractPlugin({
      filename: '[name].[contenthash].css'
    })
  ],
  
  resolve: {
    extensions: ['.js', '.jsx', '.ts', '.tsx'],
    alias: {
      '@': path.resolve(__dirname, 'src'),
      '@components': path.resolve(__dirname, 'src/components')
    }
  }
};
```

### Development vs Production Config
```javascript
// webpack.dev.js
const { merge } = require('webpack-merge');
const common = require('./webpack.common.js');

module.exports = merge(common, {
  mode: 'development',
  devtool: 'inline-source-map',
  
  devServer: {
    static: './dist',
    port: 3000,
    hot: true,
    open: true,
    historyApiFallback: true,
    proxy: {
      '/api': 'http://localhost:8000'
    }
  },
  
  optimization: {
    runtimeChunk: 'single'
  }
});
```

```javascript
// webpack.prod.js
const { merge } = require('webpack-merge');
const common = require('./webpack.common.js');
const TerserPlugin = require('terser-webpack-plugin');
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin');

module.exports = merge(common, {
  mode: 'production',
  devtool: 'source-map',
  
  optimization: {
    minimize: true,
    minimizer: [
      new TerserPlugin({
        terserOptions: {
          compress: {
            drop_console: true
          }
        }
      }),
      new CssMinimizerPlugin()
    ],
    
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all'
        },
        common: {
          name: 'common',
          minChunks: 2,
          chunks: 'all',
          enforce: true
        }
      }
    }
  }
});
```

### Advanced Webpack Features
```javascript
// webpack.config.js - Advanced features
const webpack = require('webpack');
const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer');

module.exports = {
  // ... other config
  
  plugins: [
    // Environment variables
    new webpack.DefinePlugin({
      'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV),
      'process.env.API_URL': JSON.stringify(process.env.API_URL)
    }),
    
    // Bundle analysis
    new BundleAnalyzerPlugin({
      analyzerMode: process.env.ANALYZE ? 'server' : 'disabled'
    }),
    
    // Module federation (micro-frontends)
    new webpack.container.ModuleFederationPlugin({
      name: 'shell',
      remotes: {
        mfe1: 'mfe1@http://localhost:3001/remoteEntry.js'
      }
    })
  ],
  
  // Performance hints
  performance: {
    hints: 'warning',
    maxEntrypointSize: 512000,
    maxAssetSize: 512000
  },
  
  // Cache configuration
  cache: {
    type: 'filesystem',
    buildDependencies: {
      config: [__filename]
    }
  }
};
```

## ESBuild

### Basic ESBuild Setup
```javascript
// build.js
const esbuild = require('esbuild');

esbuild.build({
  entryPoints: ['src/index.js'],
  bundle: true,
  outfile: 'dist/bundle.js',
  minify: true,
  sourcemap: true,
  target: ['es2020'],
  format: 'esm',
  platform: 'browser',
  
  // External dependencies
  external: ['react', 'react-dom'],
  
  // Define environment variables
  define: {
    'process.env.NODE_ENV': '"production"',
    'process.env.API_URL': '"https://api.example.com"'
  },
  
  // Loader configuration
  loader: {
    '.png': 'file',
    '.svg': 'text'
  }
}).catch(() => process.exit(1));
```

### ESBuild with Watch Mode
```javascript
// dev-server.js
const esbuild = require('esbuild');
const { createServer } = require('http');
const { readFileSync } = require('fs');

// Build with watch
const ctx = await esbuild.context({
  entryPoints: ['src/index.js'],
  bundle: true,
  outdir: 'dist',
  sourcemap: true,
  target: ['es2020'],
  format: 'esm',
  
  plugins: [{
    name: 'rebuild-notify',
    setup(build) {
      build.onEnd(result => {
        console.log(`Build completed with ${result.errors.length} errors`);
      });
    }
  }]
});

// Watch for changes
await ctx.watch();

// Simple dev server
const server = createServer((req, res) => {
  if (req.url === '/') {
    res.writeHead(200, { 'Content-Type': 'text/html' });
    res.end(readFileSync('index.html'));
  } else if (req.url.startsWith('/dist/')) {
    const file = req.url.slice(1);
    try {
      const content = readFileSync(file);
      res.writeHead(200);
      res.end(content);
    } catch {
      res.writeHead(404);
      res.end('Not found');
    }
  }
});

server.listen(3000, () => {
  console.log('Server running on http://localhost:3000');
});
```

### ESBuild Plugins
```javascript
// Custom plugin example
const cssPlugin = {
  name: 'css',
  setup(build) {
    build.onLoad({ filter: /\.css$/ }, async (args) => {
      const css = await fs.readFile(args.path, 'utf8');
      return {
        contents: `
          const style = document.createElement('style');
          style.textContent = ${JSON.stringify(css)};
          document.head.appendChild(style);
        `,
        loader: 'js'
      };
    });
  }
};

// Environment plugin
const envPlugin = {
  name: 'env',
  setup(build) {
    build.onResolve({ filter: /^env$/ }, args => ({
      path: args.path,
      namespace: 'env-ns'
    }));
    
    build.onLoad({ filter: /.*/, namespace: 'env-ns' }, () => ({
      contents: JSON.stringify(process.env),
      loader: 'json'
    }));
  }
};

esbuild.build({
  entryPoints: ['src/index.js'],
  bundle: true,
  outfile: 'dist/bundle.js',
  plugins: [cssPlugin, envPlugin]
});
```

## Performance Comparison

### Build Speed Benchmarks
```bash
# Typical build times for medium React app (1000+ components)

# Cold build
Webpack: ~45s
Vite: ~12s
ESBuild: ~3s

# Hot reload
Webpack: ~2-5s
Vite: ~50-200ms
ESBuild: ~10-50ms

# Bundle size (optimized)
Webpack: Excellent (with proper config)
Vite: Good (Rollup-based)
ESBuild: Good (but less optimization options)
```

### Memory Usage
```javascript
// Monitoring build memory usage
process.on('exit', () => {
  const used = process.memoryUsage();
  console.log('Memory usage:');
  for (let key in used) {
    console.log(`${key}: ${Math.round(used[key] / 1024 / 1024 * 100) / 100} MB`);
  }
});
```

### Bundle Analysis
```javascript
// Package.json scripts for analysis
{
  "scripts": {
    "analyze:webpack": "webpack-bundle-analyzer dist/stats.json",
    "analyze:vite": "vite build && npx vite-bundle-analyzer dist",
    "analyze:esbuild": "esbuild-visualizer --metadata=meta.json --filename=stats.html"
  }
}
```

## Migration Strategies

### Webpack to Vite Migration
```javascript
// 1. Install Vite and plugins
npm install -D vite @vitejs/plugin-react

// 2. Create vite.config.js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  
  // Map webpack aliases
  resolve: {
    alias: {
      '@': '/src',
      'components': '/src/components'
    }
  },
  
  // Map webpack define plugin
  define: {
    global: 'globalThis',
  },
  
  // Handle webpack-specific imports
  optimizeDeps: {
    include: ['react', 'react-dom']
  }
})

// 3. Update package.json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  }
}
```

### Common Migration Issues
```javascript
// Issue 1: CommonJS imports
// Before (Webpack)
const React = require('react');

// After (Vite/ESBuild)
import React from 'react';

// Issue 2: Dynamic imports
// Before
import('./module').then(module => {
  // Webpack returns { default: ... }
  const Component = module.default;
});

// After
import('./module').then(module => {
  // ESM standard
  const Component = module.default;
});

// Issue 3: Environment variables
// Before (Webpack)
process.env.REACT_APP_API_URL

// After (Vite)
import.meta.env.VITE_API_URL

// Issue 4: Asset imports
// Before
import logo from './logo.png';

// After (might need explicit URL)
import logoUrl from './logo.png?url';
```

### Gradual Migration Strategy
```javascript
// 1. Start with development only
{
  "scripts": {
    "dev": "vite",
    "build": "webpack --mode=production"
  }
}

// 2. Add feature flags
const useVite = process.env.USE_VITE === 'true';

// 3. Parallel builds for testing
{
  "scripts": {
    "build:webpack": "webpack --mode=production",
    "build:vite": "vite build",
    "build:compare": "npm run build:webpack && npm run build:vite"
  }
}
```

## 🔗 Related Notes

- [[ESLint & Prettier]] - Code quality tools integration
- [[Performance Optimization]] - Build optimization strategies
- [[CI-CD & Deployment]] - Build tool integration in pipelines
- [[Node.js]] - Build tool runtime environment

## 📚 Additional Resources

- [Vite Documentation](https://vitejs.dev/)
- [Webpack Documentation](https://webpack.js.org/)
- [ESBuild Documentation](https://esbuild.github.io/)
- [Build Tool Performance Comparison](https://github.com/privatenumber/build-tool-performance)
- [Vite vs Webpack Comparison](https://vitejs.dev/guide/comparisons.html)

---
*Last updated: January 27, 2025* 