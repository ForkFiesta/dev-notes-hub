# State Management

> **Modern state management patterns and tools for React applications**

## 🎯 Global vs Local State

### When to Use Local State
```typescript
// ✅ Use local state for:
// - Component-specific UI state
// - Form inputs
// - Temporary data
// - Toggle states

const LoginForm = () => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [showPassword, setShowPassword] = useState(false);
  
  return (
    <form>
      <input 
        value={email} 
        onChange={(e) => setEmail(e.target.value)} 
      />
      <input 
        type={showPassword ? 'text' : 'password'}
        value={password} 
        onChange={(e) => setPassword(e.target.value)} 
      />
      <button onClick={() => setShowPassword(!showPassword)}>
        Toggle Password
      </button>
    </form>
  );
};
```

### When to Use Global State
```typescript
// ✅ Use global state for:
// - User authentication data
// - App-wide settings/theme
// - Data shared across multiple components
// - Server state that needs caching
// - Complex state that needs persistence

// Examples of global state needs:
interface GlobalState {
  user: User | null;           // Authentication
  theme: 'light' | 'dark';     // App settings
  notifications: Notification[]; // Shared data
  cart: CartItem[];            // E-commerce data
}
```

## 🔄 Redux Toolkit (RTK)

### Basic Setup

```typescript
// store/index.ts
import { configureStore } from '@reduxjs/toolkit';
import authSlice from './slices/authSlice';
import cartSlice from './slices/cartSlice';

export const store = configureStore({
  reducer: {
    auth: authSlice,
    cart: cartSlice,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        ignoredActions: ['persist/PERSIST'],
      },
    }),
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

### Creating Slices

```typescript
// store/slices/authSlice.ts
import { createSlice, createAsyncThunk, PayloadAction } from '@reduxjs/toolkit';

interface User {
  id: string;
  name: string;
  email: string;
}

interface AuthState {
  user: User | null;
  loading: boolean;
  error: string | null;
}

const initialState: AuthState = {
  user: null,
  loading: false,
  error: null,
};

// Async thunk for login
export const loginUser = createAsyncThunk(
  'auth/loginUser',
  async (credentials: { email: string; password: string }, { rejectWithValue }) => {
    try {
      const response = await fetch('/api/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(credentials),
      });
      
      if (!response.ok) {
        throw new Error('Login failed');
      }
      
      return await response.json();
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);

const authSlice = createSlice({
  name: 'auth',
  initialState,
  reducers: {
    logout: (state) => {
      state.user = null;
      state.error = null;
    },
    clearError: (state) => {
      state.error = null;
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(loginUser.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(loginUser.fulfilled, (state, action: PayloadAction<User>) => {
        state.loading = false;
        state.user = action.payload;
      })
      .addCase(loginUser.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload as string;
      });
  },
});

export const { logout, clearError } = authSlice.actions;
export default authSlice.reducer;
```

### Typed Hooks

```typescript
// hooks/redux.ts
import { useDispatch, useSelector, TypedUseSelectorHook } from 'react-redux';
import type { RootState, AppDispatch } from '../store';

export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
```

### Using in Components

```typescript
// components/LoginForm.tsx
import { useAppDispatch, useAppSelector } from '../hooks/redux';
import { loginUser, clearError } from '../store/slices/authSlice';

const LoginForm = () => {
  const dispatch = useAppDispatch();
  const { user, loading, error } = useAppSelector((state) => state.auth);
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    const formData = new FormData(e.target as HTMLFormElement);
    
    await dispatch(loginUser({
      email: formData.get('email') as string,
      password: formData.get('password') as string,
    }));
  };

  if (user) {
    return <div>Welcome, {user.name}!</div>;
  }

  return (
    <form onSubmit={handleSubmit}>
      {error && (
        <div className="error">
          {error}
          <button onClick={() => dispatch(clearError())}>×</button>
        </div>
      )}
      <input name="email" type="email" required />
      <input name="password" type="password" required />
      <button type="submit" disabled={loading}>
        {loading ? 'Logging in...' : 'Login'}
      </button>
    </form>
  );
};
```

### RTK Query for Server State

```typescript
// api/apiSlice.ts
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';

interface User {
  id: string;
  name: string;
  email: string;
}

export const apiSlice = createApi({
  reducerPath: 'api',
  baseQuery: fetchBaseQuery({
    baseUrl: '/api',
    prepareHeaders: (headers, { getState }) => {
      const token = (getState() as RootState).auth.token;
      if (token) {
        headers.set('authorization', `Bearer ${token}`);
      }
      return headers;
    },
  }),
  tagTypes: ['User', 'Post'],
  endpoints: (builder) => ({
    getUsers: builder.query<User[], void>({
      query: () => '/users',
      providesTags: ['User'],
    }),
    getUserById: builder.query<User, string>({
      query: (id) => `/users/${id}`,
      providesTags: (result, error, id) => [{ type: 'User', id }],
    }),
    updateUser: builder.mutation<User, Partial<User> & Pick<User, 'id'>>({
      query: ({ id, ...patch }) => ({
        url: `/users/${id}`,
        method: 'PATCH',
        body: patch,
      }),
      invalidatesTags: (result, error, { id }) => [{ type: 'User', id }],
    }),
  }),
});

export const { useGetUsersQuery, useGetUserByIdQuery, useUpdateUserMutation } = apiSlice;
```

## 🐻 Zustand

### Basic Store

```typescript
// store/useAuthStore.ts
import { create } from 'zustand';
import { devtools, persist } from 'zustand/middleware';

interface User {
  id: string;
  name: string;
  email: string;
}

interface AuthState {
  user: User | null;
  loading: boolean;
  error: string | null;
  login: (credentials: { email: string; password: string }) => Promise<void>;
  logout: () => void;
  clearError: () => void;
}

export const useAuthStore = create<AuthState>()(
  devtools(
    persist(
      (set, get) => ({
        user: null,
        loading: false,
        error: null,
        
        login: async (credentials) => {
          set({ loading: true, error: null });
          
          try {
            const response = await fetch('/api/login', {
              method: 'POST',
              headers: { 'Content-Type': 'application/json' },
              body: JSON.stringify(credentials),
            });
            
            if (!response.ok) {
              throw new Error('Login failed');
            }
            
            const user = await response.json();
            set({ user, loading: false });
          } catch (error) {
            set({ error: error.message, loading: false });
          }
        },
        
        logout: () => {
          set({ user: null, error: null });
        },
        
        clearError: () => {
          set({ error: null });
        },
      }),
      {
        name: 'auth-storage',
        partialize: (state) => ({ user: state.user }), // Only persist user
      }
    ),
    {
      name: 'auth-store',
    }
  )
);
```

### Using Zustand in Components

```typescript
// components/LoginForm.tsx
import { useAuthStore } from '../store/useAuthStore';

const LoginForm = () => {
  const { user, loading, error, login, logout, clearError } = useAuthStore();
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    const formData = new FormData(e.target as HTMLFormElement);
    
    await login({
      email: formData.get('email') as string,
      password: formData.get('password') as string,
    });
  };

  if (user) {
    return (
      <div>
        <p>Welcome, {user.name}!</p>
        <button onClick={logout}>Logout</button>
      </div>
    );
  }

  return (
    <form onSubmit={handleSubmit}>
      {error && (
        <div className="error">
          {error}
          <button onClick={clearError}>×</button>
        </div>
      )}
      <input name="email" type="email" required />
      <input name="password" type="password" required />
      <button type="submit" disabled={loading}>
        {loading ? 'Logging in...' : 'Login'}
      </button>
    </form>
  );
};
```

### Zustand with Immer

```typescript
// store/useCartStore.ts
import { create } from 'zustand';
import { immer } from 'zustand/middleware/immer';

interface CartItem {
  id: string;
  name: string;
  price: number;
  quantity: number;
}

interface CartState {
  items: CartItem[];
  total: number;
  addItem: (item: Omit<CartItem, 'quantity'>) => void;
  removeItem: (id: string) => void;
  updateQuantity: (id: string, quantity: number) => void;
  clearCart: () => void;
}

export const useCartStore = create<CartState>()(
  immer((set) => ({
    items: [],
    total: 0,
    
    addItem: (newItem) =>
      set((state) => {
        const existingItem = state.items.find(item => item.id === newItem.id);
        
        if (existingItem) {
          existingItem.quantity += 1;
        } else {
          state.items.push({ ...newItem, quantity: 1 });
        }
        
        state.total = state.items.reduce(
          (sum, item) => sum + item.price * item.quantity,
          0
        );
      }),
    
    removeItem: (id) =>
      set((state) => {
        state.items = state.items.filter(item => item.id !== id);
        state.total = state.items.reduce(
          (sum, item) => sum + item.price * item.quantity,
          0
        );
      }),
    
    updateQuantity: (id, quantity) =>
      set((state) => {
        const item = state.items.find(item => item.id === id);
        if (item) {
          item.quantity = quantity;
          state.total = state.items.reduce(
            (sum, item) => sum + item.price * item.quantity,
            0
          );
        }
      }),
    
    clearCart: () =>
      set((state) => {
        state.items = [];
        state.total = 0;
      }),
  }))
);
```

## ⚛️ Jotai

### Basic Atoms

```typescript
// store/atoms.ts
import { atom } from 'jotai';

// Primitive atoms
export const countAtom = atom(0);
export const nameAtom = atom('');

// Derived atoms (read-only)
export const doubleCountAtom = atom((get) => get(countAtom) * 2);

// Derived atoms (read-write)
export const uppercaseNameAtom = atom(
  (get) => get(nameAtom).toUpperCase(),
  (get, set, newValue: string) => {
    set(nameAtom, newValue.toLowerCase());
  }
);

// Async atoms
export const userAtom = atom<User | null>(null);
export const userAsyncAtom = atom(
  async (get) => {
    const userId = get(userIdAtom);
    if (!userId) return null;
    
    const response = await fetch(`/api/users/${userId}`);
    return response.json();
  }
);

// Atom with async write
export const loginAtom = atom(
  null,
  async (get, set, credentials: { email: string; password: string }) => {
    set(loadingAtom, true);
    
    try {
      const response = await fetch('/api/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(credentials),
      });
      
      if (!response.ok) {
        throw new Error('Login failed');
      }
      
      const user = await response.json();
      set(userAtom, user);
      set(errorAtom, null);
    } catch (error) {
      set(errorAtom, error.message);
    } finally {
      set(loadingAtom, false);
    }
  }
);
```

### Using Atoms in Components

```typescript
// components/Counter.tsx
import { useAtom, useAtomValue, useSetAtom } from 'jotai';
import { countAtom, doubleCountAtom } from '../store/atoms';

const Counter = () => {
  const [count, setCount] = useAtom(countAtom);
  const doubleCount = useAtomValue(doubleCountAtom);
  
  return (
    <div>
      <p>Count: {count}</p>
      <p>Double: {doubleCount}</p>
      <button onClick={() => setCount(c => c + 1)}>+</button>
      <button onClick={() => setCount(c => c - 1)}>-</button>
    </div>
  );
};

// components/LoginForm.tsx
import { useSetAtom, useAtomValue } from 'jotai';
import { loginAtom, userAtom, loadingAtom, errorAtom } from '../store/atoms';

const LoginForm = () => {
  const login = useSetAtom(loginAtom);
  const user = useAtomValue(userAtom);
  const loading = useAtomValue(loadingAtom);
  const error = useAtomValue(errorAtom);
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    const formData = new FormData(e.target as HTMLFormElement);
    
    await login({
      email: formData.get('email') as string,
      password: formData.get('password') as string,
    });
  };

  if (user) {
    return <div>Welcome, {user.name}!</div>;
  }

  return (
    <form onSubmit={handleSubmit}>
      {error && <div className="error">{error}</div>}
      <input name="email" type="email" required />
      <input name="password" type="password" required />
      <button type="submit" disabled={loading}>
        {loading ? 'Logging in...' : 'Login'}
      </button>
    </form>
  );
};
```

### Jotai with Persistence

```typescript
// store/persistedAtoms.ts
import { atom } from 'jotai';
import { atomWithStorage } from 'jotai/utils';

// Persisted atoms
export const themeAtom = atomWithStorage<'light' | 'dark'>('theme', 'light');
export const settingsAtom = atomWithStorage('settings', {
  notifications: true,
  autoSave: false,
});

// Custom storage
const customStorage = {
  getItem: (key: string) => {
    return localStorage.getItem(key);
  },
  setItem: (key: string, value: string) => {
    localStorage.setItem(key, value);
  },
  removeItem: (key: string) => {
    localStorage.removeItem(key);
  },
};

export const userPreferencesAtom = atomWithStorage(
  'userPreferences',
  { language: 'en', timezone: 'UTC' },
  customStorage
);
```

## 📊 Comparison Matrix

| Feature | Redux RTK | Zustand | Jotai |
|---------|-----------|---------|-------|
| **Bundle Size** | ~47kb | ~2.5kb | ~5kb |
| **Learning Curve** | Steep | Gentle | Moderate |
| **Boilerplate** | High | Low | Low |
| **DevTools** | Excellent | Good | Good |
| **TypeScript** | Excellent | Excellent | Excellent |
| **Persistence** | Via middleware | Built-in | Via utils |
| **Server State** | RTK Query | External libs | External libs |
| **Performance** | Good | Excellent | Excellent |
| **Ecosystem** | Large | Growing | Growing |

## 🎯 When to Choose What

### Choose Redux RTK when:
- Large, complex applications
- Need time-travel debugging
- Team familiar with Redux patterns
- Heavy server state management needs
- Strict predictable state updates required

### Choose Zustand when:
- Medium-sized applications
- Want minimal boilerplate
- Need simple, flexible API
- Performance is critical
- Quick prototyping

### Choose Jotai when:
- Component-level state composition
- Bottom-up state architecture
- Atomic state updates
- Fine-grained reactivity needed
- React Suspense integration important

## 📋 Best Practices

### 1. **State Normalization**
```typescript
// ✅ Normalized state structure
interface NormalizedState {
  users: {
    byId: Record<string, User>;
    allIds: string[];
  };
  posts: {
    byId: Record<string, Post>;
    allIds: string[];
  };
}

// ❌ Nested/denormalized state
interface BadState {
  users: Array<{
    id: string;
    posts: Post[]; // Duplicated data
  }>;
}
```

### 2. **Separate Concerns**
```typescript
// ✅ Separate UI state from business logic
const useUIStore = create((set) => ({
  sidebarOpen: false,
  modalOpen: false,
  toggleSidebar: () => set((state) => ({ sidebarOpen: !state.sidebarOpen })),
}));

const useDataStore = create((set) => ({
  users: [],
  loading: false,
  fetchUsers: async () => { /* ... */ },
}));
```

### 3. **Optimize Selectors**
```typescript
// ✅ Memoized selectors
const selectUserById = (id: string) => (state: RootState) => 
  state.users.byId[id];

const selectUserPosts = createSelector(
  [selectUserById, (state: RootState) => state.posts.byId],
  (user, postsById) => 
    user?.postIds.map(id => postsById[id]).filter(Boolean) || []
);
```

## 🔗 Related Topics
- [[React]] - Component patterns and hooks
- [[TypeScript]] - Type-safe state management
- [[Performance Optimization]] - State optimization techniques
- [[Testing]] - Testing state management logic

---
*Choose the right state management solution based on your app's complexity, team expertise, and performance requirements. Start simple and scale up as needed.* 