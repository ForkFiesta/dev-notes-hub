---
tags: [backend, auth, jwt, oauth, security, nextauth, clerk]
created: 2025-06-15
updated: 2025-06-15
---

# 🔐 Authentication

> **Summary**: Complete guide to authentication patterns including JWT, OAuth2, and third-party solutions like NextAuth and Clerk for secure user management.

## 📋 Table of Contents

- [Authentication Fundamentals](#authentication-fundamentals)
- [JWT (JSON Web Tokens)](#jwt-json-web-tokens)
- [OAuth2 & OpenID Connect](#oauth2--openid-connect)
- [NextAuth.js](#nextauthjs)
- [Clerk Authentication](#clerk-authentication)
- [Session Management](#session-management)
- [Security Best Practices](#security-best-practices)

## Authentication Fundamentals

### Authentication vs Authorization
```javascript
// Authentication: "Who are you?"
const user = await authenticateUser(token);

// Authorization: "What can you do?"
const canAccess = await authorizeUser(user, resource, action);
```

### Common Auth Flows
1. **Username/Password** - Traditional login
2. **Social Login** - OAuth with Google, GitHub, etc.
3. **Magic Links** - Passwordless email authentication
4. **Multi-Factor Authentication** - Additional security layer
5. **Single Sign-On (SSO)** - One login for multiple apps

## JWT (JSON Web Tokens)

### JWT Structure
```javascript
// Header.Payload.Signature
const jwt = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c";

// Decoded Header
{
  "alg": "HS256",
  "typ": "JWT"
}

// Decoded Payload
{
  "sub": "1234567890",
  "name": "John Doe",
  "iat": 1516239022,
  "exp": 1516242622,
  "role": "user"
}
```

### JWT Implementation
```javascript
const jwt = require('jsonwebtoken');
const bcrypt = require('bcryptjs');

// Generate JWT
const generateToken = (user) => {
  const payload = {
    id: user.id,
    email: user.email,
    role: user.role
  };
  
  return jwt.sign(payload, process.env.JWT_SECRET, {
    expiresIn: '24h',
    issuer: 'your-app-name',
    audience: 'your-app-users'
  });
};

// Verify JWT
const verifyToken = (token) => {
  try {
    return jwt.verify(token, process.env.JWT_SECRET);
  } catch (error) {
    throw new Error('Invalid token');
  }
};

// Login endpoint
app.post('/api/auth/login', async (req, res) => {
  try {
    const { email, password } = req.body;
    
    // Find user
    const user = await User.findOne({ where: { email } });
    if (!user) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }
    
    // Verify password
    const isValid = await bcrypt.compare(password, user.password);
    if (!isValid) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }
    
    // Generate tokens
    const accessToken = generateToken(user);
    const refreshToken = generateRefreshToken(user);
    
    // Store refresh token
    await user.update({ refreshToken });
    
    res.json({
      accessToken,
      refreshToken,
      user: {
        id: user.id,
        email: user.email,
        name: user.name,
        role: user.role
      }
    });
  } catch (error) {
    res.status(500).json({ error: 'Login failed' });
  }
});
```

### JWT Middleware
```javascript
const authenticateToken = (req, res, next) => {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1];

  if (!token) {
    return res.status(401).json({ error: 'Access token required' });
  }

  try {
    const decoded = verifyToken(token);
    req.user = decoded;
    next();
  } catch (error) {
    return res.status(403).json({ error: 'Invalid or expired token' });
  }
};

// Usage
app.get('/api/protected', authenticateToken, (req, res) => {
  res.json({ message: 'Protected data', user: req.user });
});
```

### Refresh Token Pattern
```javascript
const generateRefreshToken = (user) => {
  return jwt.sign(
    { id: user.id, type: 'refresh' },
    process.env.REFRESH_TOKEN_SECRET,
    { expiresIn: '7d' }
  );
};

app.post('/api/auth/refresh', async (req, res) => {
  const { refreshToken } = req.body;
  
  if (!refreshToken) {
    return res.status(401).json({ error: 'Refresh token required' });
  }
  
  try {
    const decoded = jwt.verify(refreshToken, process.env.REFRESH_TOKEN_SECRET);
    const user = await User.findByPk(decoded.id);
    
    if (!user || user.refreshToken !== refreshToken) {
      return res.status(403).json({ error: 'Invalid refresh token' });
    }
    
    const newAccessToken = generateToken(user);
    const newRefreshToken = generateRefreshToken(user);
    
    await user.update({ refreshToken: newRefreshToken });
    
    res.json({
      accessToken: newAccessToken,
      refreshToken: newRefreshToken
    });
  } catch (error) {
    res.status(403).json({ error: 'Invalid refresh token' });
  }
});
```

## OAuth2 & OpenID Connect

### OAuth2 Flow
```javascript
const passport = require('passport');
const GoogleStrategy = require('passport-google-oauth20').Strategy;

passport.use(new GoogleStrategy({
  clientID: process.env.GOOGLE_CLIENT_ID,
  clientSecret: process.env.GOOGLE_CLIENT_SECRET,
  callbackURL: "/auth/google/callback"
}, async (accessToken, refreshToken, profile, done) => {
  try {
    // Check if user exists
    let user = await User.findOne({ googleId: profile.id });
    
    if (user) {
      return done(null, user);
    }
    
    // Create new user
    user = await User.create({
      googleId: profile.id,
      name: profile.displayName,
      email: profile.emails[0].value,
      avatar: profile.photos[0].value
    });
    
    done(null, user);
  } catch (error) {
    done(error, null);
  }
}));

// Routes
app.get('/auth/google',
  passport.authenticate('google', { scope: ['profile', 'email'] })
);

app.get('/auth/google/callback',
  passport.authenticate('google', { failureRedirect: '/login' }),
  (req, res) => {
    const token = generateToken(req.user);
    res.redirect(`${process.env.CLIENT_URL}?token=${token}`);
  }
);
```

### GitHub OAuth Implementation
```javascript
const GitHubStrategy = require('passport-github2').Strategy;

passport.use(new GitHubStrategy({
  clientID: process.env.GITHUB_CLIENT_ID,
  clientSecret: process.env.GITHUB_CLIENT_SECRET,
  callbackURL: "/auth/github/callback"
}, async (accessToken, refreshToken, profile, done) => {
  try {
    let user = await User.findOne({ githubId: profile.id });
    
    if (!user) {
      user = await User.create({
        githubId: profile.id,
        username: profile.username,
        name: profile.displayName,
        email: profile.emails?.[0]?.value,
        avatar: profile.photos[0].value
      });
    }
    
    done(null, user);
  } catch (error) {
    done(error, null);
  }
}));
```

## NextAuth.js

### Basic Setup
```javascript
// pages/api/auth/[...nextauth].js
import NextAuth from 'next-auth'
import GoogleProvider from 'next-auth/providers/google'
import GitHubProvider from 'next-auth/providers/github'
import CredentialsProvider from 'next-auth/providers/credentials'

export default NextAuth({
  providers: [
    GoogleProvider({
      clientId: process.env.GOOGLE_CLIENT_ID,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET,
    }),
    GitHubProvider({
      clientId: process.env.GITHUB_ID,
      clientSecret: process.env.GITHUB_SECRET,
    }),
    CredentialsProvider({
      name: "credentials",
      credentials: {
        email: { label: "Email", type: "email" },
        password: { label: "Password", type: "password" }
      },
      async authorize(credentials) {
        const user = await authenticateUser(credentials.email, credentials.password);
        return user ? { id: user.id, email: user.email, name: user.name } : null;
      }
    })
  ],
  callbacks: {
    async jwt({ token, user, account }) {
      if (user) {
        token.role = user.role;
      }
      return token;
    },
    async session({ session, token }) {
      session.user.id = token.sub;
      session.user.role = token.role;
      return session;
    },
  },
  pages: {
    signIn: '/auth/signin',
    signUp: '/auth/signup',
    error: '/auth/error',
  },
  session: {
    strategy: 'jwt',
    maxAge: 24 * 60 * 60, // 24 hours
  },
})
```

### Using NextAuth in Components
```jsx
import { useSession, signIn, signOut } from 'next-auth/react'

export default function Component() {
  const { data: session, status } = useSession()

  if (status === "loading") return <p>Loading...</p>

  if (session) {
    return (
      <>
        <p>Signed in as {session.user.email}</p>
        <button onClick={() => signOut()}>Sign out</button>
      </>
    )
  }
  
  return (
    <>
      <p>Not signed in</p>
      <button onClick={() => signIn()}>Sign in</button>
    </>
  )
}
```

### Protected API Routes
```javascript
// pages/api/protected.js
import { getServerSession } from "next-auth/next"
import { authOptions } from "./auth/[...nextauth]"

export default async function handler(req, res) {
  const session = await getServerSession(req, res, authOptions)
  
  if (!session) {
    res.status(401).json({ message: "You must be logged in." })
    return
  }
  
  res.json({
    message: "This is protected content.",
    user: session.user
  })
}
```

## Clerk Authentication

### Setup
```bash
npm install @clerk/nextjs
```

```javascript
// _app.js
import { ClerkProvider } from '@clerk/nextjs'

function MyApp({ Component, pageProps }) {
  return (
    <ClerkProvider {...pageProps}>
      <Component {...pageProps} />
    </ClerkProvider>
  )
}

export default MyApp
```

### Using Clerk Components
```jsx
import {
  SignedIn,
  SignedOut,
  SignInButton,
  UserButton,
  useUser
} from "@clerk/nextjs";

export default function Header() {
  const { user } = useUser();

  return (
    <header>
      <SignedOut>
        <SignInButton />
      </SignedOut>
      <SignedIn>
        <p>Welcome, {user?.firstName}!</p>
        <UserButton />
      </SignedIn>
    </header>
  );
}
```

### Protecting Pages
```jsx
import { withClerkMiddleware, requireAuth } from "@clerk/nextjs/api";

export default requireAuth((req, res) => {
  res.json({ message: "This is a protected API route" });
});

export const config = {
  matcher: "/api/protected/:path*",
};
```

## Session Management

### Cookie-Based Sessions
```javascript
const session = require('express-session');
const MongoStore = require('connect-mongo');

app.use(session({
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  store: MongoStore.create({
    mongoUrl: process.env.MONGODB_URI
  }),
  cookie: {
    secure: process.env.NODE_ENV === 'production',
    httpOnly: true,
    maxAge: 24 * 60 * 60 * 1000 // 24 hours
  }
}));
```

### Redis Sessions
```javascript
const redis = require('redis');
const RedisStore = require('connect-redis')(session);
const client = redis.createClient();

app.use(session({
  store: new RedisStore({ client }),
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: {
    secure: false,
    httpOnly: true,
    maxAge: 1000 * 60 * 60 * 24
  }
}));
```

## Security Best Practices

### Password Security
```javascript
const bcrypt = require('bcryptjs');
const zxcvbn = require('zxcvbn');

// Hash password
const hashPassword = async (password) => {
  const saltRounds = 12;
  return await bcrypt.hash(password, saltRounds);
};

// Validate password strength
const validatePassword = (password) => {
  const result = zxcvbn(password);
  return {
    isStrong: result.score >= 3,
    feedback: result.feedback,
    score: result.score
  };
};

// Registration with validation
app.post('/api/auth/register', async (req, res) => {
  const { email, password, name } = req.body;
  
  // Validate password strength
  const passwordValidation = validatePassword(password);
  if (!passwordValidation.isStrong) {
    return res.status(400).json({
      error: 'Password too weak',
      feedback: passwordValidation.feedback
    });
  }
  
  const hashedPassword = await hashPassword(password);
  
  const user = await User.create({
    email,
    password: hashedPassword,
    name
  });
  
  res.status(201).json({ message: 'User created successfully' });
});
```

### Rate Limiting
```javascript
const rateLimit = require('express-rate-limit');

const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // Limit each IP to 5 requests per windowMs
  message: 'Too many login attempts, please try again later',
  standardHeaders: true,
  legacyHeaders: false,
});

app.use('/api/auth/login', authLimiter);
```

### CSRF Protection
```javascript
const csrf = require('csurf');

const csrfProtection = csrf({
  cookie: {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'strict'
  }
});

app.use(csrfProtection);

app.get('/api/csrf-token', (req, res) => {
  res.json({ csrfToken: req.csrfToken() });
});
```

### Environment Variables
```bash
# .env
JWT_SECRET=your-super-secret-jwt-key-min-32-chars
REFRESH_TOKEN_SECRET=your-refresh-token-secret
SESSION_SECRET=your-session-secret

# OAuth
GOOGLE_CLIENT_ID=your-google-client-id
GOOGLE_CLIENT_SECRET=your-google-client-secret
GITHUB_ID=your-github-client-id
GITHUB_SECRET=your-github-client-secret

# NextAuth
NEXTAUTH_URL=http://localhost:3000
NEXTAUTH_SECRET=your-nextauth-secret

# Clerk
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_...
CLERK_SECRET_KEY=sk_test_...
```

## 🔗 Related Notes

- [[API Design]] - Securing API endpoints
- [[Node.js]] - Server-side implementation
- [[Next.js]] - Full-stack authentication
- [[Testing]] - Testing authentication flows

## 📚 Additional Resources

- [JWT.io](https://jwt.io/) - JWT debugger and library list
- [OAuth 2.0 RFC](https://tools.ietf.org/html/rfc6749)
- [NextAuth.js Documentation](https://next-auth.js.org/)
- [Clerk Documentation](https://clerk.com/docs)
- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)

---
*Last updated: January 27, 2025* 