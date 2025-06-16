# React - JavaScript Library for Building UIs

> **Quick Summary**: React is a component-based library for building user interfaces. Master components, hooks, and state management for modern web applications.

## 🏗️ Components

### Function Components
```jsx
// Basic function component
function Welcome(props) {
    return <h1>Hello, {props.name}!</h1>;
}

// Arrow function component
const Welcome = (props) => {
    return <h1>Hello, {props.name}!</h1>;
};

// Implicit return
const Welcome = ({ name }) => <h1>Hello, {name}!</h1>;

// With destructuring
const UserCard = ({ user: { name, email, avatar } }) => (
    <div className="user-card">
        <img src={avatar} alt={name} />
        <h3>{name}</h3>
        <p>{email}</p>
    </div>
);
```

### JSX Fundamentals
```jsx
// JSX expressions
const name = 'John';
const element = <h1>Hello, {name}!</h1>;

// Conditional rendering
const Greeting = ({ isLoggedIn, user }) => (
    <div>
        {isLoggedIn ? (
            <h1>Welcome back, {user.name}!</h1>
        ) : (
            <h1>Please sign in.</h1>
        )}
    </div>
);

// Lists and keys
const TodoList = ({ todos }) => (
    <ul>
        {todos.map(todo => (
            <li key={todo.id} className={todo.completed ? 'completed' : ''}>
                {todo.text}
            </li>
        ))}
    </ul>
);

// Event handling
const Button = () => {
    const handleClick = (e) => {
        e.preventDefault();
        console.log('Button clicked!');
    };

    return <button onClick={handleClick}>Click me</button>;
};

// Inline styles
const StyledComponent = () => (
    <div style={{ 
        backgroundColor: 'blue', 
        color: 'white', 
        padding: '10px' 
    }}>
        Styled content
    </div>
);
```

## 🎣 Hooks

### useState
```jsx
import { useState } from 'react';

// Basic state
const Counter = () => {
    const [count, setCount] = useState(0);

    return (
        <div>
            <p>Count: {count}</p>
            <button onClick={() => setCount(count + 1)}>+</button>
            <button onClick={() => setCount(count - 1)}>-</button>
            <button onClick={() => setCount(0)}>Reset</button>
        </div>
    );
};

// Object state
const UserForm = () => {
    const [user, setUser] = useState({
        name: '',
        email: '',
        age: 0
    });

    const updateUser = (field, value) => {
        setUser(prevUser => ({
            ...prevUser,
            [field]: value
        }));
    };

    return (
        <form>
            <input
                value={user.name}
                onChange={(e) => updateUser('name', e.target.value)}
                placeholder="Name"
            />
            <input
                value={user.email}
                onChange={(e) => updateUser('email', e.target.value)}
                placeholder="Email"
            />
        </form>
    );
};

// Array state
const TodoApp = () => {
    const [todos, setTodos] = useState([]);

    const addTodo = (text) => {
        setTodos(prev => [...prev, {
            id: Date.now(),
            text,
            completed: false
        }]);
    };

    const toggleTodo = (id) => {
        setTodos(prev => prev.map(todo =>
            todo.id === id ? { ...todo, completed: !todo.completed } : todo
        ));
    };

    const deleteTodo = (id) => {
        setTodos(prev => prev.filter(todo => todo.id !== id));
    };

    return (
        <div>
            {/* Todo list implementation */}
        </div>
    );
};
```

### useEffect
```jsx
import { useState, useEffect } from 'react';

// Basic effect
const Timer = () => {
    const [seconds, setSeconds] = useState(0);

    useEffect(() => {
        const interval = setInterval(() => {
            setSeconds(prev => prev + 1);
        }, 1000);

        // Cleanup function
        return () => clearInterval(interval);
    }, []); // Empty dependency array = run once

    return <div>Seconds: {seconds}</div>;
};

// Effect with dependencies
const UserProfile = ({ userId }) => {
    const [user, setUser] = useState(null);
    const [loading, setLoading] = useState(true);

    useEffect(() => {
        const fetchUser = async () => {
            setLoading(true);
            try {
                const response = await fetch(`/api/users/${userId}`);
                const userData = await response.json();
                setUser(userData);
            } catch (error) {
                console.error('Failed to fetch user:', error);
            } finally {
                setLoading(false);
            }
        };

        fetchUser();
    }, [userId]); // Re-run when userId changes

    if (loading) return <div>Loading...</div>;
    if (!user) return <div>User not found</div>;

    return (
        <div>
            <h1>{user.name}</h1>
            <p>{user.email}</p>
        </div>
    );
};

// Multiple effects
const Component = () => {
    useEffect(() => {
        // Effect for data fetching
        fetchData();
    }, []);

    useEffect(() => {
        // Effect for window resize
        const handleResize = () => {
            console.log('Window resized');
        };

        window.addEventListener('resize', handleResize);
        return () => window.removeEventListener('resize', handleResize);
    }, []);

    return <div>Component</div>;
};
```

### useContext
```jsx
import { createContext, useContext, useState } from 'react';

// Create context
const ThemeContext = createContext();

// Provider component
const ThemeProvider = ({ children }) => {
    const [theme, setTheme] = useState('light');

    const toggleTheme = () => {
        setTheme(prev => prev === 'light' ? 'dark' : 'light');
    };

    return (
        <ThemeContext.Provider value={{ theme, toggleTheme }}>
            {children}
        </ThemeContext.Provider>
    );
};

// Consumer component
const ThemedButton = () => {
    const { theme, toggleTheme } = useContext(ThemeContext);

    return (
        <button
            onClick={toggleTheme}
            className={`btn btn-${theme}`}
        >
            Current theme: {theme}
        </button>
    );
};

// App with provider
const App = () => (
    <ThemeProvider>
        <div>
            <ThemedButton />
        </div>
    </ThemeProvider>
);
```

### useReducer
```jsx
import { useReducer } from 'react';

// Reducer function
const todoReducer = (state, action) => {
    switch (action.type) {
        case 'ADD_TODO':
            return [
                ...state,
                {
                    id: Date.now(),
                    text: action.payload,
                    completed: false
                }
            ];
        case 'TOGGLE_TODO':
            return state.map(todo =>
                todo.id === action.payload
                    ? { ...todo, completed: !todo.completed }
                    : todo
            );
        case 'DELETE_TODO':
            return state.filter(todo => todo.id !== action.payload);
        default:
            return state;
    }
};

// Component using useReducer
const TodoApp = () => {
    const [todos, dispatch] = useReducer(todoReducer, []);

    const addTodo = (text) => {
        dispatch({ type: 'ADD_TODO', payload: text });
    };

    const toggleTodo = (id) => {
        dispatch({ type: 'TOGGLE_TODO', payload: id });
    };

    const deleteTodo = (id) => {
        dispatch({ type: 'DELETE_TODO', payload: id });
    };

    return (
        <div>
            {todos.map(todo => (
                <div key={todo.id}>
                    <span
                        onClick={() => toggleTodo(todo.id)}
                        style={{
                            textDecoration: todo.completed ? 'line-through' : 'none'
                        }}
                    >
                        {todo.text}
                    </span>
                    <button onClick={() => deleteTodo(todo.id)}>Delete</button>
                </div>
            ))}
        </div>
    );
};
```

### Custom Hooks
```jsx
// Custom hook for API calls
const useApi = (url) => {
    const [data, setData] = useState(null);
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState(null);

    useEffect(() => {
        const fetchData = async () => {
            try {
                setLoading(true);
                const response = await fetch(url);
                if (!response.ok) throw new Error('Failed to fetch');
                const result = await response.json();
                setData(result);
            } catch (err) {
                setError(err.message);
            } finally {
                setLoading(false);
            }
        };

        fetchData();
    }, [url]);

    return { data, loading, error };
};

// Custom hook for local storage
const useLocalStorage = (key, initialValue) => {
    const [storedValue, setStoredValue] = useState(() => {
        try {
            const item = window.localStorage.getItem(key);
            return item ? JSON.parse(item) : initialValue;
        } catch (error) {
            return initialValue;
        }
    });

    const setValue = (value) => {
        try {
            setStoredValue(value);
            window.localStorage.setItem(key, JSON.stringify(value));
        } catch (error) {
            console.error('Error saving to localStorage:', error);
        }
    };

    return [storedValue, setValue];
};

// Usage of custom hooks
const UserProfile = ({ userId }) => {
    const { data: user, loading, error } = useApi(`/api/users/${userId}`);
    const [preferences, setPreferences] = useLocalStorage('userPrefs', {});

    if (loading) return <div>Loading...</div>;
    if (error) return <div>Error: {error}</div>;

    return (
        <div>
            <h1>{user.name}</h1>
            <p>{user.email}</p>
        </div>
    );
};
```

## 🔄 State Management

### Lifting State Up
```jsx
// Parent component manages shared state
const TodoApp = () => {
    const [todos, setTodos] = useState([]);
    const [filter, setFilter] = useState('all');

    const addTodo = (text) => {
        setTodos(prev => [...prev, {
            id: Date.now(),
            text,
            completed: false
        }]);
    };

    const toggleTodo = (id) => {
        setTodos(prev => prev.map(todo =>
            todo.id === id ? { ...todo, completed: !todo.completed } : todo
        ));
    };

    const filteredTodos = todos.filter(todo => {
        if (filter === 'active') return !todo.completed;
        if (filter === 'completed') return todo.completed;
        return true;
    });

    return (
        <div>
            <TodoForm onAddTodo={addTodo} />
            <TodoList todos={filteredTodos} onToggleTodo={toggleTodo} />
            <TodoFilter filter={filter} onFilterChange={setFilter} />
        </div>
    );
};
```

### Context for Global State
```jsx
// Global state context
const AppContext = createContext();

const AppProvider = ({ children }) => {
    const [user, setUser] = useState(null);
    const [theme, setTheme] = useState('light');
    const [notifications, setNotifications] = useState([]);

    const login = async (credentials) => {
        try {
            const response = await fetch('/api/login', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(credentials)
            });
            const userData = await response.json();
            setUser(userData);
        } catch (error) {
            console.error('Login failed:', error);
        }
    };

    const logout = () => {
        setUser(null);
    };

    const addNotification = (message, type = 'info') => {
        const notification = {
            id: Date.now(),
            message,
            type
        };
        setNotifications(prev => [...prev, notification]);
    };

    return (
        <AppContext.Provider value={{
            user,
            theme,
            notifications,
            login,
            logout,
            setTheme,
            addNotification
        }}>
            {children}
        </AppContext.Provider>
    );
};

// Hook to use app context
const useApp = () => {
    const context = useContext(AppContext);
    if (!context) {
        throw new Error('useApp must be used within AppProvider');
    }
    return context;
};
```

## 🧩 Component Patterns

### Compound Components
```jsx
// Compound component pattern
const Tabs = ({ children, defaultTab = 0 }) => {
    const [activeTab, setActiveTab] = useState(defaultTab);

    return (
        <div className="tabs">
            {React.Children.map(children, (child, index) =>
                React.cloneElement(child, { activeTab, setActiveTab, index })
            )}
        </div>
    );
};

const TabList = ({ children, activeTab, setActiveTab }) => (
    <div className="tab-list">
        {React.Children.map(children, (child, index) =>
            React.cloneElement(child, { 
                isActive: index === activeTab,
                onClick: () => setActiveTab(index)
            })
        )}
    </div>
);

const Tab = ({ children, isActive, onClick }) => (
    <button
        className={`tab ${isActive ? 'active' : ''}`}
        onClick={onClick}
    >
        {children}
    </button>
);

const TabPanels = ({ children, activeTab }) => (
    <div className="tab-panels">
        {React.Children.toArray(children)[activeTab]}
    </div>
);

const TabPanel = ({ children }) => (
    <div className="tab-panel">{children}</div>
);

// Usage
const App = () => (
    <Tabs defaultTab={0}>
        <TabList>
            <Tab>Tab 1</Tab>
            <Tab>Tab 2</Tab>
            <Tab>Tab 3</Tab>
        </TabList>
        <TabPanels>
            <TabPanel>Content 1</TabPanel>
            <TabPanel>Content 2</TabPanel>
            <TabPanel>Content 3</TabPanel>
        </TabPanels>
    </Tabs>
);
```

### Render Props
```jsx
// Render prop component
const DataFetcher = ({ url, render }) => {
    const [data, setData] = useState(null);
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState(null);

    useEffect(() => {
        const fetchData = async () => {
            try {
                const response = await fetch(url);
                const result = await response.json();
                setData(result);
            } catch (err) {
                setError(err.message);
            } finally {
                setLoading(false);
            }
        };

        fetchData();
    }, [url]);

    return render({ data, loading, error });
};

// Usage
const UserList = () => (
    <DataFetcher
        url="/api/users"
        render={({ data, loading, error }) => {
            if (loading) return <div>Loading...</div>;
            if (error) return <div>Error: {error}</div>;
            return (
                <ul>
                    {data.map(user => (
                        <li key={user.id}>{user.name}</li>
                    ))}
                </ul>
            );
        }}
    />
);
```

### Higher-Order Components (HOCs)
```jsx
// HOC for authentication
const withAuth = (WrappedComponent) => {
    return (props) => {
        const { user } = useApp();

        if (!user) {
            return <div>Please log in to access this page.</div>;
        }

        return <WrappedComponent {...props} />;
    };
};

// HOC for loading states
const withLoading = (WrappedComponent) => {
    return ({ isLoading, ...props }) => {
        if (isLoading) {
            return <div>Loading...</div>;
        }

        return <WrappedComponent {...props} />;
    };
};

// Usage
const Dashboard = () => <div>Dashboard Content</div>;
const ProtectedDashboard = withAuth(withLoading(Dashboard));
```

## 🚦 React Router
```jsx
import { BrowserRouter, Routes, Route, Link, useNavigate, useParams } from 'react-router-dom';

// Basic routing setup
const App = () => (
    <BrowserRouter>
        <nav>
            <Link to="/">Home</Link>
            <Link to="/about">About</Link>
            <Link to="/users">Users</Link>
        </nav>

        <Routes>
            <Route path="/" element={<Home />} />
            <Route path="/about" element={<About />} />
            <Route path="/users" element={<Users />} />
            <Route path="/users/:id" element={<UserDetail />} />
            <Route path="*" element={<NotFound />} />
        </Routes>
    </BrowserRouter>
);

// Component with URL parameters
const UserDetail = () => {
    const { id } = useParams();
    const navigate = useNavigate();

    const goBack = () => {
        navigate(-1); // Go back one page
    };

    return (
        <div>
            <h1>User {id}</h1>
            <button onClick={goBack}>Go Back</button>
        </div>
    );
};

// Programmatic navigation
const LoginForm = () => {
    const navigate = useNavigate();

    const handleLogin = async (credentials) => {
        try {
            await login(credentials);
            navigate('/dashboard'); // Redirect after login
        } catch (error) {
            console.error('Login failed');
        }
    };

    return (
        <form onSubmit={handleLogin}>
            {/* Form fields */}
        </form>
    );
};
```

## 🎯 Performance Optimization

### React.memo
```jsx
// Memoize component to prevent unnecessary re-renders
const ExpensiveComponent = React.memo(({ data, onUpdate }) => {
    console.log('ExpensiveComponent rendered');
    
    return (
        <div>
            {data.map(item => (
                <div key={item.id}>{item.name}</div>
            ))}
        </div>
    );
});

// Custom comparison function
const MyComponent = React.memo(({ user, settings }) => {
    return <div>{user.name}</div>;
}, (prevProps, nextProps) => {
    // Return true if props are equal (skip re-render)
    return prevProps.user.id === nextProps.user.id;
});
```

### useMemo and useCallback
```jsx
const ExpensiveCalculation = ({ items, multiplier }) => {
    // Memoize expensive calculations
    const expensiveValue = useMemo(() => {
        console.log('Calculating expensive value...');
        return items.reduce((sum, item) => sum + item.value, 0) * multiplier;
    }, [items, multiplier]);

    // Memoize callback functions
    const handleClick = useCallback((id) => {
        console.log('Item clicked:', id);
    }, []);

    return (
        <div>
            <p>Total: {expensiveValue}</p>
            {items.map(item => (
                <Item
                    key={item.id}
                    item={item}
                    onClick={handleClick}
                />
            ))}
        </div>
    );
};
```

## ✅ Quick Checklist

- [ ] Use function components with hooks
- [ ] Implement proper key props for lists
- [ ] Handle loading and error states
- [ ] Use useEffect cleanup functions
- [ ] Implement proper form handling
- [ ] Use React.memo for performance optimization
- [ ] Structure components logically
- [ ] Handle edge cases and validation
- [ ] Use TypeScript for better development experience
- [ ] Test components thoroughly

## 🔗 Related Topics
- [[JavaScript]] - Foundation for React development
- [[Tailwind CSS]] - Styling React components
- [[Node.js]] - Backend for React applications

---
*Remember: React is about building reusable, composable components. Think in components and data flow!* 