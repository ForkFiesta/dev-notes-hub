# TypeScript

> **Essential TypeScript concepts and patterns for modern web development**

## 🎯 Core Concepts

### Basic Types

```typescript
// Primitive types
let name: string = "John";
let age: number = 30;
let isActive: boolean = true;
let data: null = null;
let value: undefined = undefined;

// Arrays
let numbers: number[] = [1, 2, 3];
let names: Array<string> = ["Alice", "Bob"];

// Tuples
let person: [string, number] = ["John", 30];

// Any (avoid when possible)
let anything: any = "could be anything";

// Unknown (safer than any)
let userInput: unknown;
if (typeof userInput === "string") {
  console.log(userInput.toUpperCase()); // Type narrowing
}
```

### Interfaces vs Types

```typescript
// Interface - extensible, declaration merging
interface User {
  id: number;
  name: string;
  email?: string; // Optional property
  readonly createdAt: Date; // Read-only
}

// Extending interfaces
interface AdminUser extends User {
  permissions: string[];
}

// Type alias - more flexible for unions/intersections
type Status = "pending" | "approved" | "rejected";
type UserWithStatus = User & { status: Status };

// Function types
interface ApiCall {
  (endpoint: string, data?: any): Promise<any>;
}

type ApiCallType = (endpoint: string, data?: any) => Promise<any>;
```

### Enums

```typescript
// Numeric enum (auto-incrementing)
enum Direction {
  Up,    // 0
  Down,  // 1
  Left,  // 2
  Right  // 3
}

// String enum (recommended)
enum Theme {
  LIGHT = "light",
  DARK = "dark",
  AUTO = "auto"
}

// Const enum (compile-time optimization)
const enum HttpStatus {
  OK = 200,
  NOT_FOUND = 404,
  SERVER_ERROR = 500
}

// Usage
const currentTheme: Theme = Theme.DARK;
```

### Generics

```typescript
// Generic functions
function identity<T>(arg: T): T {
  return arg;
}

const stringResult = identity<string>("hello"); // string
const numberResult = identity(42); // Type inferred as number

// Generic interfaces
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}

interface User {
  id: number;
  name: string;
}

const userResponse: ApiResponse<User> = {
  data: { id: 1, name: "John" },
  status: 200,
  message: "Success"
};

// Generic constraints
interface Lengthwise {
  length: number;
}

function logLength<T extends Lengthwise>(arg: T): T {
  console.log(arg.length);
  return arg;
}

// Generic classes
class DataStore<T> {
  private data: T[] = [];
  
  add(item: T): void {
    this.data.push(item);
  }
  
  get(index: number): T | undefined {
    return this.data[index];
  }
}

const stringStore = new DataStore<string>();
stringStore.add("hello");
```

### Utility Types

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  age: number;
}

// Partial - makes all properties optional
type PartialUser = Partial<User>;
// { id?: number; name?: string; email?: string; age?: number; }

// Required - makes all properties required
type RequiredUser = Required<PartialUser>;

// Pick - select specific properties
type UserSummary = Pick<User, "id" | "name">;
// { id: number; name: string; }

// Omit - exclude specific properties
type UserWithoutId = Omit<User, "id">;
// { name: string; email: string; age: number; }

// Record - create object type with specific keys
type UserRoles = Record<"admin" | "user" | "guest", string[]>;
// { admin: string[]; user: string[]; guest: string[]; }

// Exclude/Extract with union types
type Status = "pending" | "approved" | "rejected" | "cancelled";
type ActiveStatus = Exclude<Status, "cancelled">; // "pending" | "approved" | "rejected"
type FinalStatus = Extract<Status, "approved" | "rejected">; // "approved" | "rejected"

// ReturnType - get function return type
function getUser() {
  return { id: 1, name: "John" };
}
type UserReturnType = ReturnType<typeof getUser>; // { id: number; name: string; }
```

## ⚛️ TypeScript with React

### Component Props

```typescript
import React from 'react';

// Basic props interface
interface ButtonProps {
  children: React.ReactNode;
  onClick: () => void;
  variant?: "primary" | "secondary";
  disabled?: boolean;
}

// Using the interface
const Button: React.FC<ButtonProps> = ({ 
  children, 
  onClick, 
  variant = "primary", 
  disabled = false 
}) => {
  return (
    <button 
      onClick={onClick} 
      disabled={disabled}
      className={`btn btn-${variant}`}
    >
      {children}
    </button>
  );
};

// Extending HTML attributes
interface InputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  label: string;
  error?: string;
}

const Input: React.FC<InputProps> = ({ label, error, ...props }) => {
  return (
    <div>
      <label>{label}</label>
      <input {...props} />
      {error && <span className="error">{error}</span>}
    </div>
  );
};
```

### Hooks with TypeScript

```typescript
import { useState, useEffect, useRef, useCallback } from 'react';

// useState with explicit typing
const [user, setUser] = useState<User | null>(null);
const [loading, setLoading] = useState<boolean>(false);

// useRef with DOM elements
const inputRef = useRef<HTMLInputElement>(null);

// useRef with mutable values
const countRef = useRef<number>(0);

// useCallback with proper typing
const handleSubmit = useCallback((data: FormData) => {
  // Handle form submission
}, []);

// Custom hook with TypeScript
function useApi<T>(url: string) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState<boolean>(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    fetch(url)
      .then(response => response.json())
      .then((data: T) => {
        setData(data);
        setLoading(false);
      })
      .catch((err: Error) => {
        setError(err.message);
        setLoading(false);
      });
  }, [url]);

  return { data, loading, error };
}

// Usage
const { data: users, loading, error } = useApi<User[]>('/api/users');
```

### Event Handlers

```typescript
import React from 'react';

const EventHandlers: React.FC = () => {
  // Form events
  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    // Handle form submission
  };

  const handleInputChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    console.log(e.target.value);
  };

  // Mouse events
  const handleClick = (e: React.MouseEvent<HTMLButtonElement>) => {
    console.log('Button clicked');
  };

  // Keyboard events
  const handleKeyDown = (e: React.KeyboardEvent<HTMLInputElement>) => {
    if (e.key === 'Enter') {
      // Handle enter key
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input 
        onChange={handleInputChange}
        onKeyDown={handleKeyDown}
      />
      <button onClick={handleClick}>Submit</button>
    </form>
  );
};
```

## 🎨 TypeScript with Tailwind CSS

### Typed Tailwind Classes

```typescript
// Create type-safe Tailwind class utilities
type TailwindColor = 
  | "red" | "blue" | "green" | "yellow" | "purple" 
  | "pink" | "indigo" | "gray";

type TailwindShade = 
  | "50" | "100" | "200" | "300" | "400" 
  | "500" | "600" | "700" | "800" | "900";

type BackgroundColor = `bg-${TailwindColor}-${TailwindShade}`;
type TextColor = `text-${TailwindColor}-${TailwindShade}`;

interface StyledComponentProps {
  bgColor?: BackgroundColor;
  textColor?: TextColor;
  className?: string;
}

const StyledDiv: React.FC<StyledComponentProps> = ({ 
  bgColor = "bg-white", 
  textColor = "text-gray-900",
  className = "",
  children 
}) => {
  return (
    <div className={`${bgColor} ${textColor} ${className}`}>
      {children}
    </div>
  );
};

// Usage with type safety
<StyledDiv 
  bgColor="bg-blue-500" 
  textColor="text-white"
  className="p-4 rounded-lg"
>
  Content
</StyledDiv>
```

### Class Variance Authority (CVA) with TypeScript

```typescript
import { cva, type VariantProps } from "class-variance-authority";

// Define button variants
const buttonVariants = cva(
  "inline-flex items-center justify-center rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:opacity-50 disabled:pointer-events-none ring-offset-background",
  {
    variants: {
      variant: {
        default: "bg-primary text-primary-foreground hover:bg-primary/90",
        destructive: "bg-destructive text-destructive-foreground hover:bg-destructive/90",
        outline: "border border-input hover:bg-accent hover:text-accent-foreground",
        secondary: "bg-secondary text-secondary-foreground hover:bg-secondary/80",
        ghost: "hover:bg-accent hover:text-accent-foreground",
        link: "underline-offset-4 hover:underline text-primary",
      },
      size: {
        default: "h-10 py-2 px-4",
        sm: "h-9 px-3 rounded-md",
        lg: "h-11 px-8 rounded-md",
        icon: "h-10 w-10",
      },
    },
    defaultVariants: {
      variant: "default",
      size: "default",
    },
  }
);

// Create component with proper typing
interface ButtonProps 
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  asChild?: boolean;
}

const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, asChild = false, ...props }, ref) => {
    return (
      <button
        className={buttonVariants({ variant, size, className })}
        ref={ref}
        {...props}
      />
    );
  }
);
```

## 🛠️ Advanced Patterns

### Discriminated Unions

```typescript
// API response types
interface LoadingState {
  status: "loading";
}

interface SuccessState {
  status: "success";
  data: any;
}

interface ErrorState {
  status: "error";
  error: string;
}

type ApiState = LoadingState | SuccessState | ErrorState;

// Type-safe handling
function handleApiState(state: ApiState) {
  switch (state.status) {
    case "loading":
      return <div>Loading...</div>;
    case "success":
      return <div>{state.data}</div>; // TypeScript knows data exists
    case "error":
      return <div>Error: {state.error}</div>; // TypeScript knows error exists
  }
}
```

### Conditional Types

```typescript
// Create conditional types based on conditions
type NonNullable<T> = T extends null | undefined ? never : T;

type ApiResponse<T> = T extends string 
  ? { message: T } 
  : { data: T };

// Mapped types
type Optional<T> = {
  [K in keyof T]?: T[K];
};

type Readonly<T> = {
  readonly [K in keyof T]: T[K];
};
```

## 📋 Best Practices

### 1. **Strict TypeScript Configuration**
```json
// tsconfig.json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "noImplicitReturns": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true
  }
}
```

### 2. **Prefer Interfaces for Object Shapes**
```typescript
// ✅ Good - use interface for object shapes
interface User {
  id: number;
  name: string;
}

// ✅ Good - use type for unions/computed types
type Status = "active" | "inactive";
type UserWithStatus = User & { status: Status };
```

### 3. **Use Type Guards**
```typescript
function isString(value: unknown): value is string {
  return typeof value === "string";
}

function isUser(obj: any): obj is User {
  return obj && typeof obj.id === "number" && typeof obj.name === "string";
}

// Usage
if (isUser(data)) {
  console.log(data.name); // TypeScript knows this is safe
}
```

### 4. **Avoid `any`, Use `unknown`**
```typescript
// ❌ Avoid
function processData(data: any) {
  return data.someProperty; // No type safety
}

// ✅ Better
function processData(data: unknown) {
  if (isUser(data)) {
    return data.name; // Type-safe after guard
  }
}
```

## 🔗 Related Topics
- [[React]] - Component patterns and hooks
- [[State Management]] - Typed state management solutions
- [[Testing]] - Testing TypeScript code
- [[Performance Optimization]] - TypeScript compilation optimization

---
*TypeScript provides compile-time type safety and excellent developer experience. Start with basic types and gradually adopt advanced patterns as your codebase grows.* 