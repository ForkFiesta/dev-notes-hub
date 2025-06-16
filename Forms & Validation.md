# Forms & Validation

> **Modern form handling and validation patterns for React applications**

## 🎯 React Hook Form Basics

### Installation & Setup

```bash
npm install react-hook-form
npm install @hookform/resolvers zod  # For Zod validation
npm install @hookform/resolvers yup  # For Yup validation
```

### Basic Form Setup

```typescript
import { useForm } from 'react-hook-form';

interface FormData {
  email: string;
  password: string;
  rememberMe: boolean;
}

const LoginForm = () => {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm<FormData>();

  const onSubmit = async (data: FormData) => {
    try {
      await loginUser(data);
    } catch (error) {
      console.error('Login failed:', error);
    }
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <label htmlFor="email">Email</label>
        <input
          id="email"
          type="email"
          {...register('email', {
            required: 'Email is required',
            pattern: {
              value: /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i,
              message: 'Invalid email address'
            }
          })}
        />
        {errors.email && (
          <span className="error">{errors.email.message}</span>
        )}
      </div>

      <div>
        <label htmlFor="password">Password</label>
        <input
          id="password"
          type="password"
          {...register('password', {
            required: 'Password is required',
            minLength: {
              value: 8,
              message: 'Password must be at least 8 characters'
            }
          })}
        />
        {errors.password && (
          <span className="error">{errors.password.message}</span>
        )}
      </div>

      <div>
        <label>
          <input
            type="checkbox"
            {...register('rememberMe')}
          />
          Remember me
        </label>
      </div>

      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Logging in...' : 'Login'}
      </button>
    </form>
  );
};
```

## 🔍 Zod Validation

### Schema Definition

```typescript
import { z } from 'zod';
import { zodResolver } from '@hookform/resolvers/zod';

// Define validation schema
const loginSchema = z.object({
  email: z
    .string()
    .min(1, 'Email is required')
    .email('Invalid email address'),
  password: z
    .string()
    .min(8, 'Password must be at least 8 characters')
    .regex(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/, 
      'Password must contain uppercase, lowercase, and number'),
  rememberMe: z.boolean().optional(),
});

type LoginFormData = z.infer<typeof loginSchema>;

// Use with React Hook Form
const LoginForm = () => {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm<LoginFormData>({
    resolver: zodResolver(loginSchema),
  });

  const onSubmit = (data: LoginFormData) => {
    console.log(data); // Type-safe and validated
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {/* Form fields */}
    </form>
  );
};
```

### Complex Validation Schemas

```typescript
// User registration schema
const registrationSchema = z.object({
  personalInfo: z.object({
    firstName: z.string().min(2, 'First name must be at least 2 characters'),
    lastName: z.string().min(2, 'Last name must be at least 2 characters'),
    dateOfBirth: z.string().refine((date) => {
      const age = new Date().getFullYear() - new Date(date).getFullYear();
      return age >= 18;
    }, 'Must be at least 18 years old'),
  }),
  account: z.object({
    email: z.string().email('Invalid email address'),
    password: z.string().min(8, 'Password must be at least 8 characters'),
    confirmPassword: z.string(),
  }).refine((data) => data.password === data.confirmPassword, {
    message: "Passwords don't match",
    path: ["confirmPassword"],
  }),
  preferences: z.object({
    newsletter: z.boolean(),
    notifications: z.enum(['all', 'important', 'none']),
    theme: z.enum(['light', 'dark', 'auto']).default('auto'),
  }),
});

// Array validation
const todoListSchema = z.object({
  title: z.string().min(1, 'Title is required'),
  todos: z.array(z.object({
    id: z.string(),
    text: z.string().min(1, 'Todo text is required'),
    completed: z.boolean(),
    priority: z.enum(['low', 'medium', 'high']),
  })).min(1, 'At least one todo is required'),
});

// Conditional validation
const paymentSchema = z.object({
  amount: z.number().positive('Amount must be positive'),
  paymentMethod: z.enum(['card', 'paypal', 'bank']),
  cardDetails: z.object({
    cardNumber: z.string().regex(/^\d{16}$/, 'Invalid card number'),
    expiryDate: z.string().regex(/^\d{2}\/\d{2}$/, 'Invalid expiry date'),
    cvv: z.string().regex(/^\d{3,4}$/, 'Invalid CVV'),
  }).optional(),
}).refine((data) => {
  if (data.paymentMethod === 'card') {
    return data.cardDetails !== undefined;
  }
  return true;
}, {
  message: 'Card details are required for card payments',
  path: ['cardDetails'],
});
```

## 📝 Yup Validation

### Basic Yup Schema

```typescript
import * as yup from 'yup';
import { yupResolver } from '@hookform/resolvers/yup';

const loginSchema = yup.object({
  email: yup
    .string()
    .required('Email is required')
    .email('Invalid email address'),
  password: yup
    .string()
    .required('Password is required')
    .min(8, 'Password must be at least 8 characters')
    .matches(
      /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/,
      'Password must contain uppercase, lowercase, and number'
    ),
  rememberMe: yup.boolean(),
});

// Advanced Yup patterns
const profileSchema = yup.object({
  name: yup.string().required('Name is required'),
  age: yup
    .number()
    .required('Age is required')
    .positive('Age must be positive')
    .integer('Age must be a whole number')
    .min(18, 'Must be at least 18 years old'),
  website: yup
    .string()
    .url('Must be a valid URL')
    .nullable(),
  tags: yup
    .array()
    .of(yup.string().required())
    .min(1, 'At least one tag is required'),
});
```

## 🏗️ Common Form Structures

### Multi-Step Form

```typescript
import { useState } from 'react';
import { useForm, FormProvider } from 'react-hook-form';

const multiStepSchema = z.object({
  step1: z.object({
    firstName: z.string().min(1, 'First name is required'),
    lastName: z.string().min(1, 'Last name is required'),
  }),
  step2: z.object({
    email: z.string().email('Invalid email'),
    phone: z.string().min(10, 'Phone number is required'),
  }),
  step3: z.object({
    address: z.string().min(1, 'Address is required'),
    city: z.string().min(1, 'City is required'),
    zipCode: z.string().min(5, 'Valid zip code required'),
  }),
});

const MultiStepForm = () => {
  const [currentStep, setCurrentStep] = useState(1);
  const methods = useForm({
    resolver: zodResolver(multiStepSchema),
    mode: 'onChange',
  });

  const { handleSubmit, trigger, formState: { errors } } = methods;

  const nextStep = async () => {
    const stepField = `step${currentStep}` as keyof typeof multiStepSchema.shape;
    const isValid = await trigger(stepField);
    
    if (isValid) {
      setCurrentStep(prev => prev + 1);
    }
  };

  const prevStep = () => {
    setCurrentStep(prev => prev - 1);
  };

  const onSubmit = (data: any) => {
    console.log('Form submitted:', data);
  };

  return (
    <FormProvider {...methods}>
      <form onSubmit={handleSubmit(onSubmit)}>
        {currentStep === 1 && <Step1 />}
        {currentStep === 2 && <Step2 />}
        {currentStep === 3 && <Step3 />}
        
        <div className="flex justify-between mt-4">
          {currentStep > 1 && (
            <button type="button" onClick={prevStep}>
              Previous
            </button>
          )}
          
          {currentStep < 3 ? (
            <button type="button" onClick={nextStep}>
              Next
            </button>
          ) : (
            <button type="submit">
              Submit
            </button>
          )}
        </div>
      </form>
    </FormProvider>
  );
};

// Step components
const Step1 = () => {
  const { register, formState: { errors } } = useFormContext();
  
  return (
    <div>
      <h2>Personal Information</h2>
      <input
        {...register('step1.firstName')}
        placeholder="First Name"
      />
      {errors.step1?.firstName && (
        <span className="error">{errors.step1.firstName.message}</span>
      )}
      
      <input
        {...register('step1.lastName')}
        placeholder="Last Name"
      />
      {errors.step1?.lastName && (
        <span className="error">{errors.step1.lastName.message}</span>
      )}
    </div>
  );
};
```

### Dynamic Form Fields

```typescript
import { useFieldArray } from 'react-hook-form';

const dynamicFormSchema = z.object({
  projectName: z.string().min(1, 'Project name is required'),
  tasks: z.array(z.object({
    title: z.string().min(1, 'Task title is required'),
    description: z.string().optional(),
    priority: z.enum(['low', 'medium', 'high']),
    assignee: z.string().min(1, 'Assignee is required'),
  })).min(1, 'At least one task is required'),
});

const DynamicForm = () => {
  const { register, control, handleSubmit, formState: { errors } } = useForm({
    resolver: zodResolver(dynamicFormSchema),
    defaultValues: {
      projectName: '',
      tasks: [{ title: '', description: '', priority: 'medium', assignee: '' }],
    },
  });

  const { fields, append, remove } = useFieldArray({
    control,
    name: 'tasks',
  });

  const onSubmit = (data: any) => {
    console.log(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <label>Project Name</label>
        <input {...register('projectName')} />
        {errors.projectName && (
          <span className="error">{errors.projectName.message}</span>
        )}
      </div>

      <div>
        <h3>Tasks</h3>
        {fields.map((field, index) => (
          <div key={field.id} className="border p-4 mb-4">
            <div>
              <label>Task Title</label>
              <input {...register(`tasks.${index}.title`)} />
              {errors.tasks?.[index]?.title && (
                <span className="error">
                  {errors.tasks[index].title.message}
                </span>
              )}
            </div>

            <div>
              <label>Description</label>
              <textarea {...register(`tasks.${index}.description`)} />
            </div>

            <div>
              <label>Priority</label>
              <select {...register(`tasks.${index}.priority`)}>
                <option value="low">Low</option>
                <option value="medium">Medium</option>
                <option value="high">High</option>
              </select>
            </div>

            <div>
              <label>Assignee</label>
              <input {...register(`tasks.${index}.assignee`)} />
              {errors.tasks?.[index]?.assignee && (
                <span className="error">
                  {errors.tasks[index].assignee.message}
                </span>
              )}
            </div>

            <button
              type="button"
              onClick={() => remove(index)}
              disabled={fields.length === 1}
            >
              Remove Task
            </button>
          </div>
        ))}

        <button
          type="button"
          onClick={() => append({ 
            title: '', 
            description: '', 
            priority: 'medium', 
            assignee: '' 
          })}
        >
          Add Task
        </button>
      </div>

      <button type="submit">Create Project</button>
    </form>
  );
};
```

## 🎛️ Controlled vs Uncontrolled Inputs

### Controlled Components

```typescript
// Controlled input - React manages the state
const ControlledForm = () => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    console.log({ email, password });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
      />
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="Password"
      />
      <button type="submit">Submit</button>
    </form>
  );
};

// Controlled with React Hook Form
const ControlledRHF = () => {
  const { control, handleSubmit } = useForm();

  return (
    <form onSubmit={handleSubmit(console.log)}>
      <Controller
        name="email"
        control={control}
        rules={{ required: 'Email is required' }}
        render={({ field, fieldState: { error } }) => (
          <div>
            <input
              {...field}
              type="email"
              placeholder="Email"
            />
            {error && <span className="error">{error.message}</span>}
          </div>
        )}
      />
    </form>
  );
};
```

### Uncontrolled Components

```typescript
// Uncontrolled input - DOM manages the state
const UncontrolledForm = () => {
  const emailRef = useRef<HTMLInputElement>(null);
  const passwordRef = useRef<HTMLInputElement>(null);

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    console.log({
      email: emailRef.current?.value,
      password: passwordRef.current?.value,
    });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        ref={emailRef}
        type="email"
        placeholder="Email"
        defaultValue=""
      />
      <input
        ref={passwordRef}
        type="password"
        placeholder="Password"
        defaultValue=""
      />
      <button type="submit">Submit</button>
    </form>
  );
};

// Uncontrolled with React Hook Form (recommended)
const UncontrolledRHF = () => {
  const { register, handleSubmit, formState: { errors } } = useForm();

  return (
    <form onSubmit={handleSubmit(console.log)}>
      <input
        {...register('email', { required: 'Email is required' })}
        type="email"
        placeholder="Email"
      />
      {errors.email && (
        <span className="error">{errors.email.message}</span>
      )}
      
      <input
        {...register('password', { required: 'Password is required' })}
        type="password"
        placeholder="Password"
      />
      {errors.password && (
        <span className="error">{errors.password.message}</span>
      )}
      
      <button type="submit">Submit</button>
    </form>
  );
};
```

## 🎨 Advanced Form Patterns

### File Upload with Validation

```typescript
const fileUploadSchema = z.object({
  files: z
    .instanceof(FileList)
    .refine((files) => files.length > 0, 'At least one file is required')
    .refine(
      (files) => Array.from(files).every(file => file.size <= 5000000),
      'Each file must be less than 5MB'
    )
    .refine(
      (files) => Array.from(files).every(file => 
        ['image/jpeg', 'image/png', 'image/gif'].includes(file.type)
      ),
      'Only JPEG, PNG, and GIF files are allowed'
    ),
});

const FileUploadForm = () => {
  const { register, handleSubmit, watch, formState: { errors } } = useForm({
    resolver: zodResolver(fileUploadSchema),
  });

  const watchedFiles = watch('files');

  const onSubmit = async (data: any) => {
    const formData = new FormData();
    Array.from(data.files).forEach((file: File) => {
      formData.append('files', file);
    });

    try {
      await fetch('/api/upload', {
        method: 'POST',
        body: formData,
      });
    } catch (error) {
      console.error('Upload failed:', error);
    }
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <label>Upload Images</label>
        <input
          {...register('files')}
          type="file"
          multiple
          accept="image/*"
        />
        {errors.files && (
          <span className="error">{errors.files.message}</span>
        )}
      </div>

      {watchedFiles && watchedFiles.length > 0 && (
        <div>
          <h4>Selected Files:</h4>
          <ul>
            {Array.from(watchedFiles).map((file: File, index) => (
              <li key={index}>
                {file.name} ({(file.size / 1024 / 1024).toFixed(2)} MB)
              </li>
            ))}
          </ul>
        </div>
      )}

      <button type="submit">Upload</button>
    </form>
  );
};
```

### Search Form with Debouncing

```typescript
import { useDebouncedCallback } from 'use-debounce';

const SearchForm = () => {
  const [results, setResults] = useState([]);
  const [loading, setLoading] = useState(false);

  const { register, watch } = useForm({
    defaultValues: {
      query: '',
      category: 'all',
      sortBy: 'relevance',
    },
  });

  const watchedQuery = watch('query');
  const watchedCategory = watch('category');

  const debouncedSearch = useDebouncedCallback(
    async (query: string, category: string) => {
      if (!query.trim()) {
        setResults([]);
        return;
      }

      setLoading(true);
      try {
        const response = await fetch(
          `/api/search?q=${encodeURIComponent(query)}&category=${category}`
        );
        const data = await response.json();
        setResults(data.results);
      } catch (error) {
        console.error('Search failed:', error);
      } finally {
        setLoading(false);
      }
    },
    300
  );

  useEffect(() => {
    debouncedSearch(watchedQuery, watchedCategory);
  }, [watchedQuery, watchedCategory, debouncedSearch]);

  return (
    <div>
      <form>
        <input
          {...register('query')}
          type="text"
          placeholder="Search..."
        />
        
        <select {...register('category')}>
          <option value="all">All Categories</option>
          <option value="products">Products</option>
          <option value="articles">Articles</option>
          <option value="users">Users</option>
        </select>

        <select {...register('sortBy')}>
          <option value="relevance">Relevance</option>
          <option value="date">Date</option>
          <option value="popularity">Popularity</option>
        </select>
      </form>

      {loading && <div>Searching...</div>}
      
      <div>
        {results.map((result: any) => (
          <div key={result.id}>
            <h3>{result.title}</h3>
            <p>{result.description}</p>
          </div>
        ))}
      </div>
    </div>
  );
};
```

## 📋 Best Practices

### 1. **Form Performance**
```typescript
// ✅ Use mode: 'onBlur' for better performance
const { register } = useForm({
  mode: 'onBlur', // Validate on blur instead of onChange
});

// ✅ Optimize re-renders with useCallback
const onSubmit = useCallback((data: FormData) => {
  // Handle submission
}, []);

// ✅ Use Controller only when necessary
// Prefer register() for simple inputs
```

### 2. **Error Handling**
```typescript
// ✅ Comprehensive error handling
const onSubmit = async (data: FormData) => {
  try {
    await submitForm(data);
  } catch (error) {
    if (error.response?.status === 422) {
      // Handle validation errors from server
      const serverErrors = error.response.data.errors;
      Object.keys(serverErrors).forEach(field => {
        setError(field, {
          type: 'server',
          message: serverErrors[field][0],
        });
      });
    } else {
      // Handle other errors
      setError('root', {
        type: 'server',
        message: 'Something went wrong. Please try again.',
      });
    }
  }
};
```

### 3. **Accessibility**
```typescript
// ✅ Proper form accessibility
<div>
  <label htmlFor="email">Email Address</label>
  <input
    id="email"
    type="email"
    {...register('email')}
    aria-describedby={errors.email ? 'email-error' : undefined}
    aria-invalid={errors.email ? 'true' : 'false'}
  />
  {errors.email && (
    <span id="email-error" role="alert" className="error">
      {errors.email.message}
    </span>
  )}
</div>
```

### 4. **TypeScript Integration**
```typescript
// ✅ Strong typing for forms
interface FormData {
  email: string;
  password: string;
  preferences: {
    newsletter: boolean;
    theme: 'light' | 'dark';
  };
}

const { register, handleSubmit } = useForm<FormData>({
  defaultValues: {
    email: '',
    password: '',
    preferences: {
      newsletter: false,
      theme: 'light',
    },
  },
});
```

## 🔗 Related Topics
- [[React]] - Component patterns and hooks
- [[TypeScript]] - Type-safe form handling
- [[Accessibility]] - Form accessibility guidelines
- [[State Management]] - Managing form state globally

---
*Well-designed forms are crucial for user experience. Focus on clear validation, good error handling, and accessibility to create forms users actually want to fill out.* 