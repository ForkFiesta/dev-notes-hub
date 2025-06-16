
>tags: [react, patterns, architecture, components, hooks]
>created: 2025-06-15
>updated: 2025-06-15
>related: [[React]], [[TypeScript]], [[State Management]], [[Best Practices]]
---

# 🧩 Component Design Patterns

Advanced React component patterns for building scalable, maintainable, and reusable UI components.

## 📋 Quick Summary

Component design patterns are proven solutions to common problems in React component architecture. These patterns help create flexible, composable, and maintainable components that scale well in large applications.

**Key Benefits:**
- **Reusability** - Components can be used across different contexts
- **Flexibility** - Easy to customize without modifying core logic
- **Maintainability** - Clear separation of concerns
- **Testability** - Isolated logic is easier to test

---

## 🏗️ Compound Components Pattern

**What it is:** A pattern where multiple components work together to form a complete UI, similar to HTML elements like `<select>` and `<option>`.

### Basic Implementation

```tsx
// Context for sharing state between compound components
const AccordionContext = createContext<{
  openItems: Set<string>;
  toggle: (id: string) => void;
} | null>(null);

// Main compound component
const Accordion = ({ children }: { children: React.ReactNode }) => {
  const [openItems, setOpenItems] = useState<Set<string>>(new Set());
  
  const toggle = (id: string) => {
    setOpenItems(prev => {
      const newSet = new Set(prev);
      if (newSet.has(id)) {
        newSet.delete(id);
      } else {
        newSet.add(id);
      }
      return newSet;
    });
  };

  return (
    <AccordionContext.Provider value={{ openItems, toggle }}>
      <div className="accordion">{children}</div>
    </AccordionContext.Provider>
  );
};

// Sub-components
const AccordionItem = ({ 
  id, 
  children 
}: { 
  id: string; 
  children: React.ReactNode; 
}) => {
  return <div className="accordion-item">{children}</div>;
};

const AccordionHeader = ({ 
  id, 
  children 
}: { 
  id: string; 
  children: React.ReactNode; 
}) => {
  const context = useContext(AccordionContext);
  if (!context) throw new Error('AccordionHeader must be used within Accordion');
  
  const { toggle } = context;
  
  return (
    <button 
      className="accordion-header"
      onClick={() => toggle(id)}
    >
      {children}
    </button>
  );
};

const AccordionPanel = ({ 
  id, 
  children 
}: { 
  id: string; 
  children: React.ReactNode; 
}) => {
  const context = useContext(AccordionContext);
  if (!context) throw new Error('AccordionPanel must be used within Accordion');
  
  const { openItems } = context;
  const isOpen = openItems.has(id);
  
  return isOpen ? (
    <div className="accordion-panel">{children}</div>
  ) : null;
};

// Attach sub-components to main component
Accordion.Item = AccordionItem;
Accordion.Header = AccordionHeader;
Accordion.Panel = AccordionPanel;
```

### Usage Example

```tsx
function App() {
  return (
    <Accordion>
      <Accordion.Item id="item1">
        <Accordion.Header id="item1">
          What is React?
        </Accordion.Header>
        <Accordion.Panel id="item1">
          React is a JavaScript library for building user interfaces.
        </Accordion.Panel>
      </Accordion.Item>
      
      <Accordion.Item id="item2">
        <Accordion.Header id="item2">
          What are hooks?
        </Accordion.Header>
        <Accordion.Panel id="item2">
          Hooks are functions that let you use state and lifecycle features.
        </Accordion.Panel>
      </Accordion.Item>
    </Accordion>
  );
}
```

**Benefits:**
- Intuitive API that mirrors HTML structure
- Flexible composition
- Shared state management
- Easy to extend with new sub-components

---

## 🎭 Render Props Pattern

**What it is:** A pattern where a component receives a function as a prop that returns JSX, allowing the component to share its logic while letting the consumer control the rendering.

### Basic Implementation

```tsx
interface MousePosition {
  x: number;
  y: number;
}

interface MouseTrackerProps {
  render: (mouse: MousePosition) => React.ReactNode;
}

const MouseTracker: React.FC<MouseTrackerProps> = ({ render }) => {
  const [mouse, setMouse] = useState<MousePosition>({ x: 0, y: 0 });

  const handleMouseMove = (event: React.MouseEvent) => {
    setMouse({
      x: event.clientX,
      y: event.clientY,
    });
  };

  return (
    <div 
      style={{ height: '100vh' }} 
      onMouseMove={handleMouseMove}
    >
      {render(mouse)}
    </div>
  );
};
```

### Advanced Render Props with Data Fetching

```tsx
interface DataFetcherProps<T> {
  url: string;
  render: (data: {
    data: T | null;
    loading: boolean;
    error: string | null;
    refetch: () => void;
  }) => React.ReactNode;
}

function DataFetcher<T>({ url, render }: DataFetcherProps<T>) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  const fetchData = useCallback(async () => {
    try {
      setLoading(true);
      setError(null);
      const response = await fetch(url);
      if (!response.ok) throw new Error('Failed to fetch');
      const result = await response.json();
      setData(result);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Unknown error');
    } finally {
      setLoading(false);
    }
  }, [url]);

  useEffect(() => {
    fetchData();
  }, [fetchData]);

  return <>{render({ data, loading, error, refetch: fetchData })}</>;
}
```

### Usage Examples

```tsx
// Mouse tracker usage
function App() {
  return (
    <MouseTracker
      render={({ x, y }) => (
        <div>
          <h2>Mouse Position</h2>
          <p>X: {x}, Y: {y}</p>
          <div
            style={{
              position: 'absolute',
              left: x - 10,
              top: y - 10,
              width: 20,
              height: 20,
              backgroundColor: 'red',
              borderRadius: '50%',
            }}
          />
        </div>
      )}
    />
  );
}

// Data fetcher usage
function UserProfile({ userId }: { userId: string }) {
  return (
    <DataFetcher<User>
      url={`/api/users/${userId}`}
      render={({ data, loading, error, refetch }) => {
        if (loading) return <div>Loading...</div>;
        if (error) return <div>Error: {error} <button onClick={refetch}>Retry</button></div>;
        if (!data) return <div>No user found</div>;
        
        return (
          <div>
            <h2>{data.name}</h2>
            <p>{data.email}</p>
            <button onClick={refetch}>Refresh</button>
          </div>
        );
      }}
    />
  );
}
```

**Benefits:**
- Maximum flexibility in rendering
- Logic reuse across different UIs
- Inversion of control
- Easy to test logic separately

---

## 🎣 Controller Hooks Pattern

**What it is:** Custom hooks that encapsulate component logic and state management, separating business logic from presentation.

### Basic Controller Hook

```tsx
interface UseCounterOptions {
  initialValue?: number;
  min?: number;
  max?: number;
  step?: number;
}

interface UseCounterReturn {
  count: number;
  increment: () => void;
  decrement: () => void;
  reset: () => void;
  setValue: (value: number) => void;
  canIncrement: boolean;
  canDecrement: boolean;
}

const useCounter = ({
  initialValue = 0,
  min = -Infinity,
  max = Infinity,
  step = 1,
}: UseCounterOptions = {}): UseCounterReturn => {
  const [count, setCount] = useState(initialValue);

  const setValue = useCallback((value: number) => {
    setCount(Math.max(min, Math.min(max, value)));
  }, [min, max]);

  const increment = useCallback(() => {
    setValue(count + step);
  }, [count, step, setValue]);

  const decrement = useCallback(() => {
    setValue(count - step);
  }, [count, step, setValue]);

  const reset = useCallback(() => {
    setValue(initialValue);
  }, [initialValue, setValue]);

  return {
    count,
    increment,
    decrement,
    reset,
    setValue,
    canIncrement: count < max,
    canDecrement: count > min,
  };
};
```

### Advanced Form Controller Hook

```tsx
interface UseFormOptions<T> {
  initialValues: T;
  validate?: (values: T) => Partial<Record<keyof T, string>>;
  onSubmit?: (values: T) => void | Promise<void>;
}

interface UseFormReturn<T> {
  values: T;
  errors: Partial<Record<keyof T, string>>;
  touched: Partial<Record<keyof T, boolean>>;
  isSubmitting: boolean;
  isValid: boolean;
  setValue: (field: keyof T, value: T[keyof T]) => void;
  setFieldTouched: (field: keyof T) => void;
  handleSubmit: (e: React.FormEvent) => void;
  reset: () => void;
}

function useForm<T extends Record<string, any>>({
  initialValues,
  validate,
  onSubmit,
}: UseFormOptions<T>): UseFormReturn<T> {
  const [values, setValues] = useState<T>(initialValues);
  const [errors, setErrors] = useState<Partial<Record<keyof T, string>>>({});
  const [touched, setTouched] = useState<Partial<Record<keyof T, boolean>>>({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  const validateForm = useCallback((formValues: T) => {
    if (!validate) return {};
    return validate(formValues);
  }, [validate]);

  const setValue = useCallback((field: keyof T, value: T[keyof T]) => {
    setValues(prev => ({ ...prev, [field]: value }));
    
    // Clear error when user starts typing
    if (errors[field]) {
      setErrors(prev => ({ ...prev, [field]: undefined }));
    }
  }, [errors]);

  const setFieldTouched = useCallback((field: keyof T) => {
    setTouched(prev => ({ ...prev, [field]: true }));
  }, []);

  const handleSubmit = useCallback(async (e: React.FormEvent) => {
    e.preventDefault();
    
    const formErrors = validateForm(values);
    setErrors(formErrors);
    
    // Mark all fields as touched
    const allTouched = Object.keys(values).reduce((acc, key) => ({
      ...acc,
      [key]: true,
    }), {});
    setTouched(allTouched);

    if (Object.keys(formErrors).length === 0 && onSubmit) {
      setIsSubmitting(true);
      try {
        await onSubmit(values);
      } finally {
        setIsSubmitting(false);
      }
    }
  }, [values, validateForm, onSubmit]);

  const reset = useCallback(() => {
    setValues(initialValues);
    setErrors({});
    setTouched({});
    setIsSubmitting(false);
  }, [initialValues]);

  const isValid = Object.keys(validateForm(values)).length === 0;

  return {
    values,
    errors,
    touched,
    isSubmitting,
    isValid,
    setValue,
    setFieldTouched,
    handleSubmit,
    reset,
  };
}
```

### Usage Examples

```tsx
// Counter component using controller hook
function Counter() {
  const { 
    count, 
    increment, 
    decrement, 
    reset, 
    canIncrement, 
    canDecrement 
  } = useCounter({ 
    initialValue: 0, 
    min: 0, 
    max: 10 
  });

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={decrement} disabled={!canDecrement}>
        -
      </button>
      <button onClick={increment} disabled={!canIncrement}>
        +
      </button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}

// Form component using controller hook
interface LoginForm {
  email: string;
  password: string;
}

function LoginForm() {
  const form = useForm<LoginForm>({
    initialValues: { email: '', password: '' },
    validate: (values) => {
      const errors: Partial<Record<keyof LoginForm, string>> = {};
      if (!values.email) errors.email = 'Email is required';
      if (!values.password) errors.password = 'Password is required';
      return errors;
    },
    onSubmit: async (values) => {
      console.log('Submitting:', values);
      // Handle login logic
    },
  });

  return (
    <form onSubmit={form.handleSubmit}>
      <div>
        <input
          type="email"
          value={form.values.email}
          onChange={(e) => form.setValue('email', e.target.value)}
          onBlur={() => form.setFieldTouched('email')}
          placeholder="Email"
        />
        {form.touched.email && form.errors.email && (
          <span className="error">{form.errors.email}</span>
        )}
      </div>
      
      <div>
        <input
          type="password"
          value={form.values.password}
          onChange={(e) => form.setValue('password', e.target.value)}
          onBlur={() => form.setFieldTouched('password')}
          placeholder="Password"
        />
        {form.touched.password && form.errors.password && (
          <span className="error">{form.errors.password}</span>
        )}
      </div>
      
      <button 
        type="submit" 
        disabled={!form.isValid || form.isSubmitting}
      >
        {form.isSubmitting ? 'Logging in...' : 'Login'}
      </button>
    </form>
  );
}
```

**Benefits:**
- Clean separation of logic and presentation
- Highly reusable across components
- Easy to test in isolation
- Follows React's composition model

---

## 🧠 Smart vs Dumb Components

**What it is:** An architectural pattern that separates components into two categories: Smart (Container) components that handle logic and state, and Dumb (Presentational) components that focus purely on rendering.

### Smart Components (Containers)

Smart components are responsible for:
- Managing state
- Handling side effects
- Data fetching
- Business logic
- Connecting to external systems

```tsx
// Smart Component - UserListContainer
interface User {
  id: string;
  name: string;
  email: string;
  avatar: string;
}

const UserListContainer: React.FC = () => {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  const [searchTerm, setSearchTerm] = useState('');

  // Data fetching logic
  useEffect(() => {
    const fetchUsers = async () => {
      try {
        setLoading(true);
        const response = await fetch('/api/users');
        if (!response.ok) throw new Error('Failed to fetch users');
        const data = await response.json();
        setUsers(data);
      } catch (err) {
        setError(err instanceof Error ? err.message : 'Unknown error');
      } finally {
        setLoading(false);
      }
    };

    fetchUsers();
  }, []);

  // Business logic
  const filteredUsers = useMemo(() => {
    return users.filter(user =>
      user.name.toLowerCase().includes(searchTerm.toLowerCase()) ||
      user.email.toLowerCase().includes(searchTerm.toLowerCase())
    );
  }, [users, searchTerm]);

  const handleDeleteUser = async (userId: string) => {
    try {
      await fetch(`/api/users/${userId}`, { method: 'DELETE' });
      setUsers(prev => prev.filter(user => user.id !== userId));
    } catch (err) {
      console.error('Failed to delete user:', err);
    }
  };

  // Render dumb component with data and handlers
  return (
    <UserList
      users={filteredUsers}
      loading={loading}
      error={error}
      searchTerm={searchTerm}
      onSearchChange={setSearchTerm}
      onDeleteUser={handleDeleteUser}
    />
  );
};
```

### Dumb Components (Presentational)

Dumb components are responsible for:
- Rendering UI based on props
- Handling user interactions via callbacks
- Focusing on appearance and user experience
- Being highly reusable

```tsx
// Dumb Component - UserList
interface UserListProps {
  users: User[];
  loading: boolean;
  error: string | null;
  searchTerm: string;
  onSearchChange: (term: string) => void;
  onDeleteUser: (userId: string) => void;
}

const UserList: React.FC<UserListProps> = ({
  users,
  loading,
  error,
  searchTerm,
  onSearchChange,
  onDeleteUser,
}) => {
  if (loading) {
    return <LoadingSpinner />;
  }

  if (error) {
    return <ErrorMessage message={error} />;
  }

  return (
    <div className="user-list">
      <SearchInput
        value={searchTerm}
        onChange={onSearchChange}
        placeholder="Search users..."
      />
      
      {users.length === 0 ? (
        <EmptyState message="No users found" />
      ) : (
        <div className="user-grid">
          {users.map(user => (
            <UserCard
              key={user.id}
              user={user}
              onDelete={() => onDeleteUser(user.id)}
            />
          ))}
        </div>
      )}
    </div>
  );
};

// Even more granular dumb components
const UserCard: React.FC<{
  user: User;
  onDelete: () => void;
}> = ({ user, onDelete }) => (
  <div className="user-card">
    <img src={user.avatar} alt={user.name} className="user-avatar" />
    <h3>{user.name}</h3>
    <p>{user.email}</p>
    <button onClick={onDelete} className="delete-btn">
      Delete
    </button>
  </div>
);

const SearchInput: React.FC<{
  value: string;
  onChange: (value: string) => void;
  placeholder: string;
}> = ({ value, onChange, placeholder }) => (
  <input
    type="text"
    value={value}
    onChange={(e) => onChange(e.target.value)}
    placeholder={placeholder}
    className="search-input"
  />
);

const LoadingSpinner: React.FC = () => (
  <div className="loading-spinner">Loading...</div>
);

const ErrorMessage: React.FC<{ message: string }> = ({ message }) => (
  <div className="error-message">Error: {message}</div>
);

const EmptyState: React.FC<{ message: string }> = ({ message }) => (
  <div className="empty-state">{message}</div>
);
```

### Modern Approach with Hooks

With hooks, the distinction becomes more nuanced. You can create "smart hooks" and "dumb components":

```tsx
// Smart Hook
const useUserManagement = () => {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  const fetchUsers = useCallback(async () => {
    try {
      setLoading(true);
      const response = await fetch('/api/users');
      const data = await response.json();
      setUsers(data);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Unknown error');
    } finally {
      setLoading(false);
    }
  }, []);

  const deleteUser = useCallback(async (userId: string) => {
    try {
      await fetch(`/api/users/${userId}`, { method: 'DELETE' });
      setUsers(prev => prev.filter(user => user.id !== userId));
    } catch (err) {
      console.error('Failed to delete user:', err);
    }
  }, []);

  useEffect(() => {
    fetchUsers();
  }, [fetchUsers]);

  return { users, loading, error, deleteUser, refetch: fetchUsers };
};

// Dumb Component using smart hook
const UserManagementPage: React.FC = () => {
  const { users, loading, error, deleteUser } = useUserManagement();
  const [searchTerm, setSearchTerm] = useState('');

  const filteredUsers = users.filter(user =>
    user.name.toLowerCase().includes(searchTerm.toLowerCase())
  );

  return (
    <UserList
      users={filteredUsers}
      loading={loading}
      error={error}
      searchTerm={searchTerm}
      onSearchChange={setSearchTerm}
      onDeleteUser={deleteUser}
    />
  );
};
```

### Benefits Comparison

| Smart Components | Dumb Components |
|------------------|-----------------|
| Handle complex logic | Simple and predictable |
| Manage state | Stateless (mostly) |
| Connect to APIs | Pure functions of props |
| Harder to test | Easy to test |
| Less reusable | Highly reusable |
| Business logic focused | UI focused |

---

## 🎯 When to Use Each Pattern

### Compound Components
- **Use when:** Building complex UI components with multiple related parts
- **Examples:** Accordions, Tabs, Modals, Form builders
- **Benefits:** Intuitive API, flexible composition

### Render Props
- **Use when:** You need maximum flexibility in rendering while sharing logic
- **Examples:** Data fetching, Mouse tracking, Form validation
- **Benefits:** Ultimate flexibility, logic reuse

### Controller Hooks
- **Use when:** You want to separate business logic from presentation
- **Examples:** Form handling, Data management, Complex state logic
- **Benefits:** Testable logic, reusable across components

### Smart vs Dumb Components
- **Use when:** Building large applications with complex data flow
- **Examples:** Pages with data fetching, Complex forms, Dashboard components
- **Benefits:** Clear separation of concerns, easier testing

---

## 🔗 Related Topics

- [[React]] - Core React concepts and hooks
- [[TypeScript]] - Type safety for component patterns
- [[State Management]] - Advanced state patterns
- [[Testing]] - Testing strategies for different patterns
- [[Best Practices]] - General React best practices

---

## 📚 Additional Resources

- [React Patterns](https://reactpatterns.com/)
- [Advanced React Patterns](https://kentcdodds.com/blog/advanced-react-patterns)
- [Component Composition](https://reactjs.org/docs/composition-vs-inheritance.html)
- [Custom Hooks](https://reactjs.org/docs/hooks-custom.html)

---

*Last updated: January 27, 2025* 