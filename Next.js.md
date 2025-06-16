# Next.js - React Production Framework

> **Quick Summary**: Next.js is a production-ready React framework with built-in routing, server-side rendering, and optimization features. Master App Router, Server Components, rendering strategies, and deployment for modern web applications.

## 🏗️ Project Structure

### App Router Structure (Next.js 13+)
```
my-nextjs-app/
├── app/                    # App Router (recommended)
│   ├── globals.css        # Global styles
│   ├── layout.tsx         # Root layout
│   ├── page.tsx           # Home page
│   ├── loading.tsx        # Loading UI
│   ├── error.tsx          # Error UI
│   ├── not-found.tsx      # 404 page
│   ├── about/
│   │   └── page.tsx       # /about route
│   ├── blog/
│   │   ├── page.tsx       # /blog route
│   │   ├── [slug]/
│   │   │   └── page.tsx   # /blog/[slug] dynamic route
│   │   └── loading.tsx    # Blog loading UI
│   └── api/
│       └── users/
│           └── route.ts   # API route
├── components/            # Reusable components
├── lib/                   # Utility functions
├── public/               # Static assets
├── styles/               # Additional styles
├── next.config.js        # Next.js configuration
└── package.json
```

### Pages Router Structure (Legacy)
```
my-nextjs-app/
├── pages/
│   ├── _app.tsx          # Custom App component
│   ├── _document.tsx     # Custom Document
│   ├── index.tsx         # Home page (/)
│   ├── about.tsx         # /about route
│   ├── blog/
│   │   ├── index.tsx     # /blog route
│   │   └── [slug].tsx    # /blog/[slug] dynamic route
│   └── api/
│       └── users.ts      # API route
├── components/
├── styles/
├── public/
└── next.config.js
```

## 🛣️ Routing

### App Router (Next.js 13+)
```tsx
// app/layout.tsx - Root Layout
import './globals.css';

export const metadata = {
  title: 'My Next.js App',
  description: 'Built with Next.js App Router',
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        <header>
          <nav>
            <a href="/">Home</a>
            <a href="/about">About</a>
            <a href="/blog">Blog</a>
          </nav>
        </header>
        <main>{children}</main>
        <footer>© 2024 My App</footer>
      </body>
    </html>
  );
}

// app/page.tsx - Home Page
export default function HomePage() {
  return (
    <div>
      <h1>Welcome to Next.js</h1>
      <p>This is the home page using App Router.</p>
    </div>
  );
}

// app/about/page.tsx - About Page
export default function AboutPage() {
  return (
    <div>
      <h1>About Us</h1>
      <p>Learn more about our company.</p>
    </div>
  );
}

// app/blog/[slug]/page.tsx - Dynamic Route
interface BlogPostProps {
  params: {
    slug: string;
  };
}

export default function BlogPost({ params }: BlogPostProps) {
  return (
    <div>
      <h1>Blog Post: {params.slug}</h1>
      <p>This is a dynamic blog post page.</p>
    </div>
  );
}

// Generate static params for dynamic routes
export async function generateStaticParams() {
  const posts = await fetch('https://api.example.com/posts').then(res => res.json());
  
  return posts.map((post: any) => ({
    slug: post.slug,
  }));
}
```

### Navigation
```tsx
// components/Navigation.tsx
'use client';

import Link from 'next/link';
import { usePathname } from 'next/navigation';

export default function Navigation() {
  const pathname = usePathname();
  
  const links = [
    { href: '/', label: 'Home' },
    { href: '/about', label: 'About' },
    { href: '/blog', label: 'Blog' },
    { href: '/contact', label: 'Contact' },
  ];
  
  return (
    <nav className="flex space-x-4">
      {links.map(({ href, label }) => (
        <Link
          key={href}
          href={href}
          className={`px-3 py-2 rounded-md text-sm font-medium ${
            pathname === href
              ? 'bg-blue-500 text-white'
              : 'text-gray-700 hover:bg-gray-100'
          }`}
        >
          {label}
        </Link>
      ))}
    </nav>
  );
}

// Programmatic navigation
'use client';

import { useRouter } from 'next/navigation';

export default function LoginForm() {
  const router = useRouter();
  
  const handleSubmit = async (formData: FormData) => {
    // Handle login logic
    const success = await login(formData);
    
    if (success) {
      router.push('/dashboard');
      router.refresh(); // Refresh server components
    }
  };
  
  return (
    <form action={handleSubmit}>
      {/* Form fields */}
      <button type="submit">Login</button>
    </form>
  );
}
```

### Route Groups and Parallel Routes
```tsx
// app/(marketing)/layout.tsx - Route Group
export default function MarketingLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="marketing-layout">
      <header>Marketing Header</header>
      {children}
    </div>
  );
}

// app/(marketing)/about/page.tsx
// URL: /about (route group doesn't affect URL)

// app/@modal/(.)photo/[id]/page.tsx - Parallel Route with Intercepting
export default function PhotoModal({ params }: { params: { id: string } }) {
  return (
    <div className="modal">
      <h2>Photo {params.id}</h2>
      {/* Modal content */}
    </div>
  );
}

// app/layout.tsx - Consuming parallel routes
export default function RootLayout({
  children,
  modal,
}: {
  children: React.ReactNode;
  modal: React.ReactNode;
}) {
  return (
    <html>
      <body>
        {children}
        {modal}
      </body>
    </html>
  );
}
```

## 🖥️ Server vs Client Components

### Server Components (Default)
```tsx
// app/posts/page.tsx - Server Component
import { getPosts } from '@/lib/api';

// This runs on the server
export default async function PostsPage() {
  const posts = await getPosts(); // Direct database/API calls
  
  return (
    <div>
      <h1>Blog Posts</h1>
      {posts.map(post => (
        <article key={post.id}>
          <h2>{post.title}</h2>
          <p>{post.excerpt}</p>
        </article>
      ))}
    </div>
  );
}

// Server Component benefits:
// - Direct database access
// - No JavaScript sent to client
// - Better SEO and performance
// - Secure API keys usage
```

### Client Components
```tsx
// components/SearchForm.tsx - Client Component
'use client';

import { useState } from 'react';
import { useRouter } from 'next/navigation';

export default function SearchForm() {
  const [query, setQuery] = useState('');
  const router = useRouter();
  
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    router.push(`/search?q=${encodeURIComponent(query)}`);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search..."
      />
      <button type="submit">Search</button>
    </form>
  );
}

// Client Component use cases:
// - Event handlers
// - State management
// - Browser APIs
// - Interactive features
```

### Composition Patterns
```tsx
// app/dashboard/page.tsx - Server Component with Client Components
import { getUser, getStats } from '@/lib/api';
import InteractiveChart from '@/components/InteractiveChart';
import UserProfile from '@/components/UserProfile';

export default async function DashboardPage() {
  const [user, stats] = await Promise.all([
    getUser(),
    getStats()
  ]);
  
  return (
    <div>
      <h1>Dashboard</h1>
      
      {/* Server Component */}
      <UserProfile user={user} />
      
      {/* Client Component with server data */}
      <InteractiveChart data={stats} />
    </div>
  );
}

// components/InteractiveChart.tsx
'use client';

interface Props {
  data: any[]; // Passed from Server Component
}

export default function InteractiveChart({ data }: Props) {
  const [selectedPeriod, setSelectedPeriod] = useState('week');
  
  return (
    <div>
      <select 
        value={selectedPeriod}
        onChange={(e) => setSelectedPeriod(e.target.value)}
      >
        <option value="week">Week</option>
        <option value="month">Month</option>
      </select>
      
      {/* Chart rendering logic */}
    </div>
  );
}
```

## 🔄 Rendering Strategies

### Static Site Generation (SSG)
```tsx
// app/blog/[slug]/page.tsx - Static Generation
interface BlogPostProps {
  params: { slug: string };
}

// Generate static pages at build time
export async function generateStaticParams() {
  const posts = await fetch('https://api.example.com/posts')
    .then(res => res.json());
  
  return posts.map((post: any) => ({
    slug: post.slug,
  }));
}

// Generate metadata for each page
export async function generateMetadata({ params }: BlogPostProps) {
  const post = await fetch(`https://api.example.com/posts/${params.slug}`)
    .then(res => res.json());
  
  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      title: post.title,
      description: post.excerpt,
      images: [post.image],
    },
  };
}

export default async function BlogPost({ params }: BlogPostProps) {
  const post = await fetch(`https://api.example.com/posts/${params.slug}`)
    .then(res => res.json());
  
  return (
    <article>
      <h1>{post.title}</h1>
      <div dangerouslySetInnerHTML={{ __html: post.content }} />
    </article>
  );
}
```

### Server-Side Rendering (SSR)
```tsx
// app/profile/page.tsx - Server-Side Rendering
import { cookies } from 'next/headers';
import { getUser } from '@/lib/api';

// This page will be rendered on each request
export const dynamic = 'force-dynamic';

export default async function ProfilePage() {
  const cookieStore = cookies();
  const token = cookieStore.get('auth-token');
  
  if (!token) {
    redirect('/login');
  }
  
  // Fetch user data on each request
  const user = await getUser(token.value);
  
  return (
    <div>
      <h1>Welcome, {user.name}</h1>
      <p>Last login: {new Date().toLocaleString()}</p>
    </div>
  );
}
```

### Incremental Static Regeneration (ISR)
```tsx
// app/products/page.tsx - ISR
export const revalidate = 3600; // Revalidate every hour

export default async function ProductsPage() {
  const products = await fetch('https://api.example.com/products', {
    next: { revalidate: 3600 } // Per-request revalidation
  }).then(res => res.json());
  
  return (
    <div>
      <h1>Products</h1>
      {products.map(product => (
        <div key={product.id}>
          <h2>{product.name}</h2>
          <p>${product.price}</p>
        </div>
      ))}
    </div>
  );
}

// On-demand revalidation
// app/api/revalidate/route.ts
import { revalidatePath } from 'next/cache';
import { NextRequest } from 'next/server';

export async function POST(request: NextRequest) {
  const path = request.nextUrl.searchParams.get('path');
  
  if (path) {
    revalidatePath(path);
    return Response.json({ revalidated: true });
  }
  
  return Response.json({ revalidated: false });
}
```

## 🔌 API Routes

### App Router API Routes
```tsx
// app/api/users/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { z } from 'zod';

const userSchema = z.object({
  name: z.string().min(1),
  email: z.string().email(),
});

// GET /api/users
export async function GET() {
  try {
    const users = await db.user.findMany();
    return NextResponse.json(users);
  } catch (error) {
    return NextResponse.json(
      { error: 'Failed to fetch users' },
      { status: 500 }
    );
  }
}

// POST /api/users
export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const validatedData = userSchema.parse(body);
    
    const user = await db.user.create({
      data: validatedData,
    });
    
    return NextResponse.json(user, { status: 201 });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { error: 'Invalid data', details: error.errors },
        { status: 400 }
      );
    }
    
    return NextResponse.json(
      { error: 'Failed to create user' },
      { status: 500 }
    );
  }
}

// app/api/users/[id]/route.ts - Dynamic API routes
interface RouteParams {
  params: { id: string };
}

export async function GET(
  request: NextRequest,
  { params }: RouteParams
) {
  const user = await db.user.findUnique({
    where: { id: params.id },
  });
  
  if (!user) {
    return NextResponse.json(
      { error: 'User not found' },
      { status: 404 }
    );
  }
  
  return NextResponse.json(user);
}

export async function DELETE(
  request: NextRequest,
  { params }: RouteParams
) {
  await db.user.delete({
    where: { id: params.id },
  });
  
  return new NextResponse(null, { status: 204 });
}
```

### Middleware
```tsx
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  // Authentication check
  const token = request.cookies.get('auth-token');
  
  if (request.nextUrl.pathname.startsWith('/dashboard')) {
    if (!token) {
      return NextResponse.redirect(new URL('/login', request.url));
    }
  }
  
  // Add custom headers
  const response = NextResponse.next();
  response.headers.set('x-custom-header', 'my-value');
  
  // Rate limiting
  const ip = request.ip ?? '127.0.0.1';
  const rateLimitKey = `rate-limit:${ip}`;
  
  // Implement rate limiting logic here
  
  return response;
}

export const config = {
  matcher: [
    '/((?!api|_next/static|_next/image|favicon.ico).*)',
  ],
};
```

## 🎨 Styling with Tailwind CSS

### Setup
```bash
# Install Tailwind CSS
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

```javascript
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    './pages/**/*.{js,ts,jsx,tsx,mdx}',
    './components/**/*.{js,ts,jsx,tsx,mdx}',
    './app/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  theme: {
    extend: {
      colors: {
        primary: {
          50: '#eff6ff',
          500: '#3b82f6',
          900: '#1e3a8a',
        },
      },
      fontFamily: {
        sans: ['Inter', 'sans-serif'],
      },
    },
  },
  plugins: [
    require('@tailwindcss/forms'),
    require('@tailwindcss/typography'),
  ],
}
```

```css
/* app/globals.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  html {
    font-family: 'Inter', system-ui, sans-serif;
  }
}

@layer components {
  .btn-primary {
    @apply bg-primary-500 hover:bg-primary-600 text-white font-medium py-2 px-4 rounded-lg transition-colors;
  }
  
  .card {
    @apply bg-white rounded-lg shadow-md p-6 border border-gray-200;
  }
}
```

### Component Examples
```tsx
// components/Button.tsx
import { cn } from '@/lib/utils';

interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary' | 'outline';
  size?: 'sm' | 'md' | 'lg';
}

export default function Button({
  className,
  variant = 'primary',
  size = 'md',
  ...props
}: ButtonProps) {
  return (
    <button
      className={cn(
        'inline-flex items-center justify-center rounded-md font-medium transition-colors',
        'focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-offset-2',
        'disabled:pointer-events-none disabled:opacity-50',
        {
          'bg-primary-500 text-white hover:bg-primary-600': variant === 'primary',
          'bg-gray-100 text-gray-900 hover:bg-gray-200': variant === 'secondary',
          'border border-gray-300 bg-transparent hover:bg-gray-50': variant === 'outline',
        },
        {
          'h-8 px-3 text-sm': size === 'sm',
          'h-10 px-4': size === 'md',
          'h-12 px-6 text-lg': size === 'lg',
        },
        className
      )}
      {...props}
    />
  );
}

// components/Card.tsx
interface CardProps {
  children: React.ReactNode;
  className?: string;
}

export default function Card({ children, className }: CardProps) {
  return (
    <div className={cn('card', className)}>
      {children}
    </div>
  );
}

// Usage
export default function HomePage() {
  return (
    <div className="container mx-auto px-4 py-8">
      <h1 className="text-4xl font-bold text-gray-900 mb-8">
        Welcome to Next.js
      </h1>
      
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
        <Card>
          <h2 className="text-xl font-semibold mb-4">Feature 1</h2>
          <p className="text-gray-600 mb-4">
            Description of the first feature.
          </p>
          <Button>Learn More</Button>
        </Card>
        
        <Card>
          <h2 className="text-xl font-semibold mb-4">Feature 2</h2>
          <p className="text-gray-600 mb-4">
            Description of the second feature.
          </p>
          <Button variant="outline">Get Started</Button>
        </Card>
      </div>
    </div>
  );
}
```

## 📊 Data Fetching

### Server Components Data Fetching
```tsx
// app/posts/page.tsx
async function getPosts() {
  const res = await fetch('https://api.example.com/posts', {
    next: { revalidate: 3600 }, // Cache for 1 hour
  });
  
  if (!res.ok) {
    throw new Error('Failed to fetch posts');
  }
  
  return res.json();
}

export default async function PostsPage() {
  const posts = await getPosts();
  
  return (
    <div>
      <h1>Blog Posts</h1>
      {posts.map(post => (
        <article key={post.id}>
          <h2>{post.title}</h2>
          <p>{post.excerpt}</p>
        </article>
      ))}
    </div>
  );
}

// Parallel data fetching
async function getUser(id: string) {
  const res = await fetch(`https://api.example.com/users/${id}`);
  return res.json();
}

async function getUserPosts(id: string) {
  const res = await fetch(`https://api.example.com/users/${id}/posts`);
  return res.json();
}

export default async function UserPage({ params }: { params: { id: string } }) {
  // Fetch data in parallel
  const [user, posts] = await Promise.all([
    getUser(params.id),
    getUserPosts(params.id),
  ]);
  
  return (
    <div>
      <h1>{user.name}</h1>
      <div>
        {posts.map(post => (
          <article key={post.id}>{post.title}</article>
        ))}
      </div>
    </div>
  );
}
```

### Client-Side Data Fetching
```tsx
// components/UserProfile.tsx
'use client';

import { useState, useEffect } from 'react';

interface User {
  id: string;
  name: string;
  email: string;
}

export default function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  
  useEffect(() => {
    async function fetchUser() {
      try {
        const response = await fetch(`/api/users/${userId}`);
        if (!response.ok) {
          throw new Error('Failed to fetch user');
        }
        const userData = await response.json();
        setUser(userData);
      } catch (err) {
        setError(err instanceof Error ? err.message : 'An error occurred');
      } finally {
        setLoading(false);
      }
    }
    
    fetchUser();
  }, [userId]);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!user) return <div>User not found</div>;
  
  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}

// Using SWR for client-side data fetching
import useSWR from 'swr';

const fetcher = (url: string) => fetch(url).then(res => res.json());

export default function PostsList() {
  const { data: posts, error, isLoading } = useSWR('/api/posts', fetcher);
  
  if (error) return <div>Failed to load posts</div>;
  if (isLoading) return <div>Loading...</div>;
  
  return (
    <div>
      {posts.map(post => (
        <article key={post.id}>
          <h2>{post.title}</h2>
        </article>
      ))}
    </div>
  );
}
```

## 🚀 Deployment

### Vercel Deployment
```bash
# Install Vercel CLI
npm install -g vercel

# Deploy to Vercel
vercel

# Deploy to production
vercel --prod

# Environment variables
vercel env add NEXT_PUBLIC_API_URL
vercel env add DATABASE_URL
```

```json
// vercel.json
{
  "framework": "nextjs",
  "buildCommand": "npm run build",
  "devCommand": "npm run dev",
  "installCommand": "npm install",
  "env": {
    "NEXT_PUBLIC_API_URL": "@api-url"
  },
  "build": {
    "env": {
      "DATABASE_URL": "@database-url"
    }
  }
}
```

### Docker Deployment
```dockerfile
# Dockerfile
FROM node:18-alpine AS base

# Install dependencies only when needed
FROM base AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app

COPY package.json yarn.lock* package-lock.json* pnpm-lock.yaml* ./
RUN \
  if [ -f yarn.lock ]; then yarn --frozen-lockfile; \
  elif [ -f package-lock.json ]; then npm ci; \
  elif [ -f pnpm-lock.yaml ]; then yarn global add pnpm && pnpm i --frozen-lockfile; \
  else echo "Lockfile not found." && exit 1; \
  fi

# Rebuild the source code only when needed
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

ENV NEXT_TELEMETRY_DISABLED 1

RUN npm run build

# Production image, copy all the files and run next
FROM base AS runner
WORKDIR /app

ENV NODE_ENV production
ENV NEXT_TELEMETRY_DISABLED 1

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public

# Set the correct permission for prerender cache
RUN mkdir .next
RUN chown nextjs:nodejs .next

# Automatically leverage output traces to reduce image size
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000
ENV HOSTNAME "0.0.0.0"

CMD ["node", "server.js"]
```

### Static Export
```javascript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  output: 'export',
  trailingSlash: true,
  images: {
    unoptimized: true,
  },
};

module.exports = nextConfig;
```

```bash
# Build and export static files
npm run build

# Serve static files
npx serve out
```

## ⚙️ Configuration

### Next.js Configuration
```javascript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  // Enable experimental features
  experimental: {
    appDir: true,
    serverComponentsExternalPackages: ['mongoose'],
  },
  
  // Image optimization
  images: {
    domains: ['example.com', 'cdn.example.com'],
    formats: ['image/webp', 'image/avif'],
  },
  
  // Environment variables
  env: {
    CUSTOM_KEY: process.env.CUSTOM_KEY,
  },
  
  // Redirects
  async redirects() {
    return [
      {
        source: '/old-path',
        destination: '/new-path',
        permanent: true,
      },
    ];
  },
  
  // Rewrites
  async rewrites() {
    return [
      {
        source: '/api/proxy/:path*',
        destination: 'https://api.example.com/:path*',
      },
    ];
  },
  
  // Headers
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          {
            key: 'X-Content-Type-Options',
            value: 'nosniff',
          },
        ],
      },
    ];
  },
  
  // Webpack configuration
  webpack: (config, { buildId, dev, isServer, defaultLoaders, webpack }) => {
    // Custom webpack config
    return config;
  },
};

module.exports = nextConfig;
```

## 🚀 Best Practices

### Performance Optimization
```tsx
// Image optimization
import Image from 'next/image';

export default function Gallery() {
  return (
    <div>
      <Image
        src="/hero.jpg"
        alt="Hero image"
        width={800}
        height={600}
        priority // Load immediately
        placeholder="blur"
        blurDataURL="data:image/jpeg;base64,..."
      />
      
      <Image
        src="/gallery-1.jpg"
        alt="Gallery image"
        width={400}
        height={300}
        loading="lazy" // Default behavior
      />
    </div>
  );
}

// Font optimization
import { Inter, Roboto_Mono } from 'next/font/google';

const inter = Inter({
  subsets: ['latin'],
  display: 'swap',
});

const robotoMono = Roboto_Mono({
  subsets: ['latin'],
  display: 'swap',
});

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en" className={inter.className}>
      <body>
        {children}
        <code className={robotoMono.className}>
          console.log('Hello World');
        </code>
      </body>
    </html>
  );
}

// Dynamic imports for code splitting
import dynamic from 'next/dynamic';

const DynamicComponent = dynamic(() => import('../components/HeavyComponent'), {
  loading: () => <p>Loading...</p>,
  ssr: false, // Disable server-side rendering
});

export default function Page() {
  return (
    <div>
      <h1>My Page</h1>
      <DynamicComponent />
    </div>
  );
}
```

### SEO Optimization
```tsx
// app/layout.tsx
import type { Metadata } from 'next';

export const metadata: Metadata = {
  title: {
    template: '%s | My App',
    default: 'My App',
  },
  description: 'The best Next.js app',
  keywords: ['Next.js', 'React', 'JavaScript'],
  authors: [{ name: 'Your Name' }],
  creator: 'Your Name',
  openGraph: {
    type: 'website',
    locale: 'en_US',
    url: 'https://example.com',
    siteName: 'My App',
    images: [
      {
        url: 'https://example.com/og-image.jpg',
        width: 1200,
        height: 630,
        alt: 'My App',
      },
    ],
  },
  twitter: {
    card: 'summary_large_image',
    title: 'My App',
    description: 'The best Next.js app',
    creator: '@yourusername',
    images: ['https://example.com/twitter-image.jpg'],
  },
  robots: {
    index: true,
    follow: true,
    googleBot: {
      index: true,
      follow: true,
      'max-video-preview': -1,
      'max-image-preview': 'large',
      'max-snippet': -1,
    },
  },
};

// Dynamic metadata
export async function generateMetadata({ params }): Promise<Metadata> {
  const post = await fetch(`https://api.example.com/posts/${params.slug}`)
    .then(res => res.json());
  
  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      title: post.title,
      description: post.excerpt,
      images: [post.image],
    },
  };
}

// Structured data
export default function BlogPost({ post }) {
  const jsonLd = {
    '@context': 'https://schema.org',
    '@type': 'Article',
    headline: post.title,
    description: post.excerpt,
    author: {
      '@type': 'Person',
      name: post.author.name,
    },
    datePublished: post.publishedAt,
    dateModified: post.updatedAt,
  };
  
  return (
    <>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(jsonLd) }}
      />
      <article>
        <h1>{post.title}</h1>
        <div dangerouslySetInnerHTML={{ __html: post.content }} />
      </article>
    </>
  );
}
```

## 🔧 Quick Reference

### Common Commands
```bash
# Create new Next.js app
npx create-next-app@latest my-app --typescript --tailwind --eslint --app

# Development
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Lint code
npm run lint

# Type checking
npx tsc --noEmit
```

### Useful Hooks and APIs
```tsx
// App Router hooks
import { 
  useRouter,      // Navigation
  usePathname,    // Current path
  useSearchParams // URL search params
} from 'next/navigation';

// Server-side APIs
import { 
  cookies,        // Read cookies
  headers,        // Read headers
  redirect,       // Server redirect
  notFound        // 404 page
} from 'next/headers';

// Cache and revalidation
import { 
  revalidatePath,
  revalidateTag,
  unstable_cache
} from 'next/cache';
```

### Environment Variables
```bash
# .env.local
NEXT_PUBLIC_API_URL=http://localhost:3001  # Available in browser
DATABASE_URL=postgresql://...              # Server-side only
NEXTAUTH_SECRET=your-secret-key
NEXTAUTH_URL=http://localhost:3000
```

---

## 🔗 Related Topics
- [[React]] - Core React concepts and patterns
- [[Tailwind CSS]] - Styling and design system
- [[Testing]] - Testing Next.js applications
- [[Performance Optimization]] - Next.js performance features
- [[CI/CD & Deployment]] - Deployment strategies 