# Accessibility (a11y) - Inclusive Web Development

> **Quick Summary**: Accessibility ensures your web applications are usable by everyone, including people with disabilities. Master ARIA, semantic HTML, keyboard navigation, and inclusive design principles.

## 🌐 Semantic HTML Foundation

### Proper HTML Structure
```html
<!-- ✅ Good: Semantic structure -->
<header>
    <nav aria-label="Main navigation">
        <ul>
            <li><a href="/" aria-current="page">Home</a></li>
            <li><a href="/about">About</a></li>
            <li><a href="/contact">Contact</a></li>
        </ul>
    </nav>
</header>

<main>
    <article>
        <header>
            <h1>Article Title</h1>
            <p>Published on <time datetime="2024-01-15">January 15, 2024</time></p>
        </header>
        
        <section>
            <h2>Section Heading</h2>
            <p>Article content...</p>
        </section>
    </article>
    
    <aside>
        <h2>Related Articles</h2>
        <ul>
            <li><a href="/article-1">Related Article 1</a></li>
            <li><a href="/article-2">Related Article 2</a></li>
        </ul>
    </aside>
</main>

<footer>
    <p>&copy; 2024 Company Name. All rights reserved.</p>
</footer>

<!-- ❌ Bad: Non-semantic structure -->
<div class="header">
    <div class="nav">
        <div class="nav-item">Home</div>
        <div class="nav-item">About</div>
    </div>
</div>
```

### Form Accessibility
```html
<!-- Proper form labeling -->
<form>
    <div class="form-group">
        <label for="email">Email Address *</label>
        <input 
            type="email" 
            id="email" 
            name="email" 
            required 
            aria-describedby="email-error email-help"
            aria-invalid="false"
        />
        <div id="email-help" class="help-text">
            We'll never share your email with anyone else.
        </div>
        <div id="email-error" class="error-text" aria-live="polite">
            <!-- Error message appears here -->
        </div>
    </div>
    
    <!-- Fieldset for grouped inputs -->
    <fieldset>
        <legend>Preferred Contact Method</legend>
        <div>
            <input type="radio" id="contact-email" name="contact" value="email" />
            <label for="contact-email">Email</label>
        </div>
        <div>
            <input type="radio" id="contact-phone" name="contact" value="phone" />
            <label for="contact-phone">Phone</label>
        </div>
    </fieldset>
    
    <button type="submit">Submit Form</button>
</form>
```

### Headings Hierarchy
```html
<!-- ✅ Good: Logical heading structure -->
<h1>Page Title</h1>
    <h2>Main Section</h2>
        <h3>Subsection</h3>
        <h3>Another Subsection</h3>
    <h2>Another Main Section</h2>
        <h3>Subsection</h3>

<!-- ❌ Bad: Skipping heading levels -->
<h1>Page Title</h1>
    <h4>Subsection</h4> <!-- Skipped h2 and h3 -->
```

## 🎯 ARIA (Accessible Rich Internet Applications)

### ARIA Roles
```html
<!-- Landmark roles -->
<div role="banner">Header content</div>
<div role="navigation">Navigation menu</div>
<div role="main">Main content</div>
<div role="complementary">Sidebar content</div>
<div role="contentinfo">Footer content</div>

<!-- Widget roles -->
<div role="button" tabindex="0">Custom Button</div>
<div role="checkbox" aria-checked="false" tabindex="0">Custom Checkbox</div>
<div role="slider" aria-valuemin="0" aria-valuemax="100" aria-valuenow="50" tabindex="0">
    Custom Slider
</div>

<!-- Live region roles -->
<div role="alert">Error message</div>
<div role="status">Status update</div>
<div role="log" aria-live="polite">Activity log</div>

<!-- Complex widgets -->
<div role="tablist">
    <button role="tab" aria-selected="true" aria-controls="panel1" id="tab1">
        Tab 1
    </button>
    <button role="tab" aria-selected="false" aria-controls="panel2" id="tab2">
        Tab 2
    </button>
</div>
<div role="tabpanel" id="panel1" aria-labelledby="tab1">
    Panel 1 content
</div>
<div role="tabpanel" id="panel2" aria-labelledby="tab2" hidden>
    Panel 2 content
</div>
```

### ARIA Properties and States
```html
<!-- Labels and descriptions -->
<button aria-label="Close dialog">×</button>
<input type="password" aria-labelledby="pwd-label" aria-describedby="pwd-help" />
<div id="pwd-label">Password</div>
<div id="pwd-help">Must be at least 8 characters</div>

<!-- States -->
<button aria-pressed="false">Toggle Button</button>
<div aria-expanded="false" aria-haspopup="true">Dropdown Menu</div>
<input aria-invalid="true" aria-describedby="error-msg" />

<!-- Relationships -->
<div role="group" aria-labelledby="group-label">
    <h3 id="group-label">Personal Information</h3>
    <!-- Form fields -->
</div>

<!-- Live regions -->
<div aria-live="polite" aria-atomic="true" id="status">
    <!-- Status messages appear here -->
</div>
<div aria-live="assertive" role="alert" id="errors">
    <!-- Critical errors appear here -->
</div>
```

### React ARIA Examples
```jsx
// Accessible modal
const Modal = ({ isOpen, onClose, title, children }) => {
    const modalRef = useRef();
    
    useEffect(() => {
        if (isOpen) {
            modalRef.current?.focus();
        }
    }, [isOpen]);
    
    if (!isOpen) return null;
    
    return (
        <div 
            role="dialog" 
            aria-modal="true" 
            aria-labelledby="modal-title"
            ref={modalRef}
            tabIndex={-1}
            className="modal-overlay"
            onClick={onClose}
        >
            <div className="modal-content" onClick={e => e.stopPropagation()}>
                <header>
                    <h2 id="modal-title">{title}</h2>
                    <button 
                        aria-label="Close modal" 
                        onClick={onClose}
                        className="close-button"
                    >
                        ×
                    </button>
                </header>
                <div className="modal-body">
                    {children}
                </div>
            </div>
        </div>
    );
};

// Accessible dropdown
const Dropdown = ({ options, value, onChange, placeholder }) => {
    const [isOpen, setIsOpen] = useState(false);
    const [focusedIndex, setFocusedIndex] = useState(-1);
    
    const handleKeyDown = (e) => {
        switch (e.key) {
            case 'ArrowDown':
                e.preventDefault();
                setFocusedIndex(prev => 
                    prev < options.length - 1 ? prev + 1 : 0
                );
                break;
            case 'ArrowUp':
                e.preventDefault();
                setFocusedIndex(prev => 
                    prev > 0 ? prev - 1 : options.length - 1
                );
                break;
            case 'Enter':
            case ' ':
                e.preventDefault();
                if (focusedIndex >= 0) {
                    onChange(options[focusedIndex]);
                    setIsOpen(false);
                }
                break;
            case 'Escape':
                setIsOpen(false);
                break;
        }
    };
    
    return (
        <div className="dropdown">
            <button
                aria-haspopup="listbox"
                aria-expanded={isOpen}
                aria-labelledby="dropdown-label"
                onClick={() => setIsOpen(!isOpen)}
                onKeyDown={handleKeyDown}
            >
                {value || placeholder}
            </button>
            
            {isOpen && (
                <ul role="listbox" aria-labelledby="dropdown-label">
                    {options.map((option, index) => (
                        <li
                            key={option.id}
                            role="option"
                            aria-selected={option.value === value}
                            className={index === focusedIndex ? 'focused' : ''}
                            onClick={() => {
                                onChange(option);
                                setIsOpen(false);
                            }}
                        >
                            {option.label}
                        </li>
                    ))}
                </ul>
            )}
        </div>
    );
};
```

## ⌨️ Keyboard Navigation

### Focus Management
```css
/* Visible focus indicators */
button:focus,
input:focus,
select:focus,
textarea:focus,
a:focus {
    outline: 2px solid #0066cc;
    outline-offset: 2px;
}

/* Custom focus styles */
.custom-button:focus {
    box-shadow: 0 0 0 3px rgba(0, 102, 204, 0.3);
    outline: none;
}

/* Skip to content link */
.skip-link {
    position: absolute;
    top: -40px;
    left: 6px;
    background: #000;
    color: #fff;
    padding: 8px;
    text-decoration: none;
    z-index: 1000;
}

.skip-link:focus {
    top: 6px;
}
```

```jsx
// Focus trap for modals
const useFocusTrap = (isActive) => {
    const containerRef = useRef();
    
    useEffect(() => {
        if (!isActive) return;
        
        const container = containerRef.current;
        const focusableElements = container.querySelectorAll(
            'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
        );
        
        const firstElement = focusableElements[0];
        const lastElement = focusableElements[focusableElements.length - 1];
        
        const handleTabKey = (e) => {
            if (e.key !== 'Tab') return;
            
            if (e.shiftKey) {
                if (document.activeElement === firstElement) {
                    e.preventDefault();
                    lastElement.focus();
                }
            } else {
                if (document.activeElement === lastElement) {
                    e.preventDefault();
                    firstElement.focus();
                }
            }
        };
        
        container.addEventListener('keydown', handleTabKey);
        firstElement?.focus();
        
        return () => {
            container.removeEventListener('keydown', handleTabKey);
        };
    }, [isActive]);
    
    return containerRef;
};

// Keyboard event handling
const KeyboardAccessibleButton = ({ onClick, children, ...props }) => {
    const handleKeyDown = (e) => {
        if (e.key === 'Enter' || e.key === ' ') {
            e.preventDefault();
            onClick(e);
        }
    };
    
    return (
        <div
            role="button"
            tabIndex={0}
            onClick={onClick}
            onKeyDown={handleKeyDown}
            {...props}
        >
            {children}
        </div>
    );
};
```

### Tab Order Management
```html
<!-- Natural tab order -->
<input type="text" /> <!-- tabindex 1 -->
<button>Submit</button> <!-- tabindex 2 -->
<a href="/link">Link</a> <!-- tabindex 3 -->

<!-- Custom tab order -->
<input type="text" tabindex="3" />
<button tabindex="1">Submit</button>
<a href="/link" tabindex="2">Link</a>

<!-- Remove from tab order -->
<div tabindex="-1">Not focusable via keyboard</div>

<!-- Add to tab order -->
<div tabindex="0" role="button">Focusable div</div>
```

## 🎨 Color and Contrast

### WCAG Color Contrast Guidelines
```css
/* WCAG AA compliance (4.5:1 ratio for normal text) */
.text-normal {
    color: #333333; /* Dark gray on white background */
    background-color: #ffffff;
}

/* WCAG AAA compliance (7:1 ratio for normal text) */
.text-high-contrast {
    color: #000000; /* Black on white background */
    background-color: #ffffff;
}

/* Large text (18pt+ or 14pt+ bold) needs 3:1 ratio */
.text-large {
    font-size: 18pt;
    color: #666666; /* Lighter gray acceptable for large text */
    background-color: #ffffff;
}

/* Error states with sufficient contrast */
.error-text {
    color: #d32f2f; /* Red with good contrast */
    background-color: #ffffff;
}

.error-background {
    color: #ffffff;
    background-color: #d32f2f;
}
```

### Color-Blind Friendly Design
```css
/* Don't rely solely on color */
.status-success {
    color: #2e7d32;
    background-color: #e8f5e8;
}

.status-success::before {
    content: "✓ ";
    font-weight: bold;
}

.status-error {
    color: #d32f2f;
    background-color: #ffeaea;
}

.status-error::before {
    content: "⚠ ";
    font-weight: bold;
}

/* Use patterns and shapes, not just color */
.chart-bar-1 {
    background: linear-stripes(45deg, #blue, transparent);
}

.chart-bar-2 {
    background: dots(#red);
}
```

### CSS Custom Properties for Themes
```css
:root {
    --color-primary: #0066cc;
    --color-primary-contrast: #ffffff;
    --color-secondary: #6c757d;
    --color-secondary-contrast: #ffffff;
    --color-success: #28a745;
    --color-success-contrast: #ffffff;
    --color-error: #dc3545;
    --color-error-contrast: #ffffff;
    --color-warning: #ffc107;
    --color-warning-contrast: #000000;
}

/* High contrast theme */
[data-theme="high-contrast"] {
    --color-primary: #000000;
    --color-primary-contrast: #ffffff;
    --color-secondary: #ffffff;
    --color-secondary-contrast: #000000;
}

/* Dark theme with proper contrast */
[data-theme="dark"] {
    --color-primary: #4dabf7;
    --color-primary-contrast: #000000;
    --color-background: #121212;
    --color-text: #ffffff;
}
```

## 🔊 Screen Reader Support

### Screen Reader Friendly Content
```html
<!-- Descriptive link text -->
<!-- ✅ Good -->
<a href="/products/laptop-123">View details for MacBook Pro 16-inch</a>

<!-- ❌ Bad -->
<a href="/products/laptop-123">Click here</a>

<!-- Image alt text -->
<img src="chart.png" alt="Sales increased 25% from January to March 2024" />
<img src="decorative-border.png" alt="" /> <!-- Decorative images -->
<img src="logo.png" alt="Company Name" />

<!-- Complex images -->
<figure>
    <img src="complex-chart.png" alt="Quarterly sales data" />
    <figcaption>
        Sales data showing Q1: $100k, Q2: $150k, Q3: $200k, Q4: $175k
    </figcaption>
</figure>

<!-- Screen reader only content -->
<span class="sr-only">Current page: </span>
<a href="/" aria-current="page">Home</a>

<!-- Skip repetitive content -->
<a href="#main-content" class="skip-link">Skip to main content</a>
```

```css
/* Screen reader only class */
.sr-only {
    position: absolute;
    width: 1px;
    height: 1px;
    padding: 0;
    margin: -1px;
    overflow: hidden;
    clip: rect(0, 0, 0, 0);
    white-space: nowrap;
    border: 0;
}

/* Show on focus for skip links */
.sr-only:focus {
    position: static;
    width: auto;
    height: auto;
    padding: inherit;
    margin: inherit;
    overflow: visible;
    clip: auto;
    white-space: inherit;
}
```

### Live Regions and Announcements
```jsx
// Announcement component
const Announcer = ({ message, priority = 'polite' }) => {
    return (
        <div
            aria-live={priority}
            aria-atomic="true"
            className="sr-only"
        >
            {message}
        </div>
    );
};

// Usage in forms
const ContactForm = () => {
    const [status, setStatus] = useState('');
    const [errors, setErrors] = useState({});
    
    const handleSubmit = async (e) => {
        e.preventDefault();
        
        try {
            await submitForm(formData);
            setStatus('Form submitted successfully!');
        } catch (error) {
            setErrors(error.fieldErrors);
            setStatus('Please correct the errors below.');
        }
    };
    
    return (
        <form onSubmit={handleSubmit}>
            <Announcer message={status} />
            
            <div>
                <label htmlFor="email">Email</label>
                <input
                    id="email"
                    type="email"
                    aria-invalid={!!errors.email}
                    aria-describedby={errors.email ? 'email-error' : undefined}
                />
                {errors.email && (
                    <div id="email-error" role="alert">
                        {errors.email}
                    </div>
                )}
            </div>
            
            <button type="submit">Submit</button>
        </form>
    );
};
```

## 📱 Mobile Accessibility

### Touch Target Sizing
```css
/* Minimum 44px touch targets */
.touch-target {
    min-height: 44px;
    min-width: 44px;
    padding: 12px;
}

/* Adequate spacing between touch targets */
.button-group button {
    margin: 8px;
}

/* Larger touch targets for critical actions */
.primary-button {
    min-height: 48px;
    padding: 16px 24px;
}
```

### Responsive Text and Zoom
```css
/* Support 200% zoom without horizontal scrolling */
.container {
    max-width: 100%;
    overflow-x: auto;
}

/* Relative units for scalable text */
body {
    font-size: 1rem; /* 16px base */
    line-height: 1.5;
}

h1 { font-size: 2rem; }
h2 { font-size: 1.5rem; }
h3 { font-size: 1.25rem; }

/* Prevent text from being too small */
.small-text {
    font-size: max(0.875rem, 14px);
}
```

## 🧪 Testing Accessibility

### Automated Testing Tools
```javascript
// Jest + jest-axe
import { axe, toHaveNoViolations } from 'jest-axe';
import { render } from '@testing-library/react';

expect.extend(toHaveNoViolations);

test('should not have accessibility violations', async () => {
    const { container } = render(<MyComponent />);
    const results = await axe(container);
    expect(results).toHaveNoViolations();
});

// Testing with specific rules
test('should pass color contrast checks', async () => {
    const { container } = render(<Button>Click me</Button>);
    const results = await axe(container, {
        rules: {
            'color-contrast': { enabled: true }
        }
    });
    expect(results).toHaveNoViolations();
});
```

### Manual Testing Checklist
```javascript
// Keyboard navigation test
const testKeyboardNavigation = () => {
    // 1. Tab through all interactive elements
    // 2. Ensure focus is visible
    // 3. Test Enter/Space on buttons
    // 4. Test Escape to close modals
    // 5. Test arrow keys in menus/tabs
};

// Screen reader test
const testScreenReader = () => {
    // 1. Turn on screen reader (NVDA, JAWS, VoiceOver)
    // 2. Navigate with screen reader shortcuts
    // 3. Verify all content is announced
    // 4. Check heading navigation
    // 5. Test form labels and errors
};
```

## 🚀 Best Practices

### Progressive Enhancement
```jsx
// Start with accessible HTML, enhance with JavaScript
const ProgressiveButton = ({ onClick, children, ...props }) => {
    const [isEnhanced, setIsEnhanced] = useState(false);
    
    useEffect(() => {
        setIsEnhanced(true);
    }, []);
    
    if (!isEnhanced) {
        // Fallback for no-JS
        return (
            <button type="submit" {...props}>
                {children}
            </button>
        );
    }
    
    return (
        <button
            type="button"
            onClick={onClick}
            {...props}
        >
            {children}
        </button>
    );
};
```

### Inclusive Design Patterns
```jsx
// Flexible component APIs
const Button = ({ 
    variant = 'primary',
    size = 'medium',
    disabled = false,
    'aria-label': ariaLabel,
    'aria-describedby': ariaDescribedBy,
    children,
    ...props 
}) => {
    return (
        <button
            className={`btn btn-${variant} btn-${size}`}
            disabled={disabled}
            aria-label={ariaLabel}
            aria-describedby={ariaDescribedBy}
            {...props}
        >
            {children}
        </button>
    );
};

// Accessible form validation
const useAccessibleValidation = () => {
    const [errors, setErrors] = useState({});
    const [touched, setTouched] = useState({});
    
    const validateField = (name, value, rules) => {
        const fieldErrors = [];
        
        rules.forEach(rule => {
            if (!rule.test(value)) {
                fieldErrors.push(rule.message);
            }
        });
        
        setErrors(prev => ({
            ...prev,
            [name]: fieldErrors
        }));
        
        return fieldErrors.length === 0;
    };
    
    const getFieldProps = (name) => ({
        'aria-invalid': errors[name]?.length > 0,
        'aria-describedby': errors[name]?.length > 0 ? `${name}-error` : undefined,
        onBlur: () => setTouched(prev => ({ ...prev, [name]: true }))
    });
    
    return { errors, touched, validateField, getFieldProps };
};
```

## 🔧 Quick Reference

### ARIA Attributes Cheat Sheet
```html
<!-- Labels -->
aria-label="Close button"
aria-labelledby="heading-id"
aria-describedby="help-text-id"

<!-- States -->
aria-checked="true|false|mixed"
aria-selected="true|false"
aria-expanded="true|false"
aria-hidden="true|false"
aria-disabled="true|false"
aria-pressed="true|false"

<!-- Properties -->
aria-required="true|false"
aria-readonly="true|false"
aria-multiline="true|false"
aria-autocomplete="none|inline|list|both"
aria-haspopup="true|false|menu|listbox|tree|grid|dialog"

<!-- Live regions -->
aria-live="off|polite|assertive"
aria-atomic="true|false"
aria-relevant="additions|removals|text|all"

<!-- Relationships -->
aria-owns="id-list"
aria-controls="id-list"
aria-flowto="id-list"
```

### Testing Tools
```bash
# Browser extensions
# - axe DevTools
# - WAVE Web Accessibility Evaluator
# - Lighthouse accessibility audit

# Command line tools
npm install -g @axe-core/cli
axe https://example.com

# Screen readers
# - NVDA (Windows, free)
# - JAWS (Windows, paid)
# - VoiceOver (macOS, built-in)
# - Orca (Linux, free)
```

### Color Contrast Tools
- WebAIM Contrast Checker
- Colour Contrast Analyser
- Stark (Figma/Sketch plugin)
- Chrome DevTools Contrast Ratio

---

## 🔗 Related Topics
- [[HTML]] - Semantic markup foundation
- [[CSS]] - Styling for accessibility
- [[React]] - Accessible component patterns
- [[Testing]] - Automated accessibility testing
- [[Best Practices]] - Inclusive development guidelines 