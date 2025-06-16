# Animations & UX

> **Modern animation techniques and UX patterns for engaging web experiences**

## 🎬 Framer Motion Basics

### Installation & Setup

```bash
npm install framer-motion
```

```typescript
// Basic motion component
import { motion } from 'framer-motion';

const AnimatedDiv = () => {
  return (
    <motion.div
      initial={{ opacity: 0, y: 20 }}
      animate={{ opacity: 1, y: 0 }}
      exit={{ opacity: 0, y: -20 }}
      transition={{ duration: 0.3 }}
    >
      Hello World!
    </motion.div>
  );
};
```

### Core Animation Properties

```typescript
// Position animations
<motion.div
  initial={{ x: -100 }}
  animate={{ x: 0 }}
  transition={{ type: "spring", stiffness: 100 }}
/>

// Scale animations
<motion.div
  whileHover={{ scale: 1.05 }}
  whileTap={{ scale: 0.95 }}
  transition={{ type: "spring", stiffness: 400, damping: 17 }}
/>

// Rotation animations
<motion.div
  animate={{ rotate: 360 }}
  transition={{ duration: 2, repeat: Infinity, ease: "linear" }}
/>

// Color animations
<motion.div
  animate={{ backgroundColor: "#ff0000" }}
  transition={{ duration: 1 }}
/>
```

### Variants Pattern

```typescript
// Define animation variants
const containerVariants = {
  hidden: { opacity: 0 },
  visible: {
    opacity: 1,
    transition: {
      delayChildren: 0.3,
      staggerChildren: 0.2
    }
  }
};

const itemVariants = {
  hidden: { y: 20, opacity: 0 },
  visible: {
    y: 0,
    opacity: 1
  }
};

// Use variants in components
const AnimatedList = ({ items }) => {
  return (
    <motion.ul
      variants={containerVariants}
      initial="hidden"
      animate="visible"
    >
      {items.map((item, index) => (
        <motion.li
          key={item.id}
          variants={itemVariants}
        >
          {item.text}
        </motion.li>
      ))}
    </motion.ul>
  );
};
```

### Page Transitions

```typescript
// Layout component with AnimatePresence
import { AnimatePresence, motion } from 'framer-motion';
import { useRouter } from 'next/router';

const pageVariants = {
  initial: {
    opacity: 0,
    x: "-100vw",
    scale: 0.8
  },
  in: {
    opacity: 1,
    x: 0,
    scale: 1
  },
  out: {
    opacity: 0,
    x: "100vw",
    scale: 1.2
  }
};

const pageTransition = {
  type: "tween",
  ease: "anticipate",
  duration: 0.5
};

const Layout = ({ children }) => {
  const router = useRouter();
  
  return (
    <AnimatePresence mode="wait" initial={false}>
      <motion.div
        key={router.asPath}
        initial="initial"
        animate="in"
        exit="out"
        variants={pageVariants}
        transition={pageTransition}
      >
        {children}
      </motion.div>
    </AnimatePresence>
  );
};
```

### Gesture Animations

```typescript
// Drag animations
<motion.div
  drag
  dragConstraints={{ left: -100, right: 100, top: -100, bottom: 100 }}
  dragElastic={0.2}
  whileDrag={{ scale: 1.2 }}
/>

// Hover and tap animations
<motion.button
  whileHover={{ 
    scale: 1.05,
    boxShadow: "0px 5px 10px rgba(0,0,0,0.2)"
  }}
  whileTap={{ scale: 0.95 }}
  transition={{ type: "spring", stiffness: 400, damping: 17 }}
>
  Click me
</motion.button>

// Custom gesture handling
const [isOpen, setIsOpen] = useState(false);

<motion.div
  onHoverStart={() => console.log('Hover started')}
  onHoverEnd={() => console.log('Hover ended')}
  onTap={() => setIsOpen(!isOpen)}
  onPan={(event, info) => console.log(info.point.x, info.point.y)}
/>
```

## 🎯 Common Animation Patterns

### Loading Animations

```typescript
// Spinner component
const Spinner = () => {
  return (
    <motion.div
      className="w-8 h-8 border-4 border-blue-500 border-t-transparent rounded-full"
      animate={{ rotate: 360 }}
      transition={{
        duration: 1,
        repeat: Infinity,
        ease: "linear"
      }}
    />
  );
};

// Skeleton loader
const SkeletonCard = () => {
  return (
    <div className="p-4 border rounded-lg">
      <motion.div
        className="h-4 bg-gray-300 rounded mb-2"
        animate={{ opacity: [0.5, 1, 0.5] }}
        transition={{ duration: 1.5, repeat: Infinity }}
      />
      <motion.div
        className="h-4 bg-gray-300 rounded w-3/4"
        animate={{ opacity: [0.5, 1, 0.5] }}
        transition={{ duration: 1.5, repeat: Infinity, delay: 0.2 }}
      />
    </div>
  );
};

// Progress bar
const ProgressBar = ({ progress }) => {
  return (
    <div className="w-full bg-gray-200 rounded-full h-2">
      <motion.div
        className="bg-blue-500 h-2 rounded-full"
        initial={{ width: 0 }}
        animate={{ width: `${progress}%` }}
        transition={{ duration: 0.5, ease: "easeOut" }}
      />
    </div>
  );
};
```

### Modal Animations

```typescript
// Modal with backdrop
const Modal = ({ isOpen, onClose, children }) => {
  return (
    <AnimatePresence>
      {isOpen && (
        <motion.div
          className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50"
          initial={{ opacity: 0 }}
          animate={{ opacity: 1 }}
          exit={{ opacity: 0 }}
          onClick={onClose}
        >
          <motion.div
            className="bg-white rounded-lg p-6 max-w-md w-full mx-4"
            initial={{ scale: 0.8, opacity: 0 }}
            animate={{ scale: 1, opacity: 1 }}
            exit={{ scale: 0.8, opacity: 0 }}
            transition={{ type: "spring", damping: 15 }}
            onClick={(e) => e.stopPropagation()}
          >
            {children}
          </motion.div>
        </motion.div>
      )}
    </AnimatePresence>
  );
};
```

### List Animations

```typescript
// Animated list with add/remove
const AnimatedTodoList = ({ todos, onRemove }) => {
  return (
    <AnimatePresence>
      {todos.map((todo) => (
        <motion.div
          key={todo.id}
          layout
          initial={{ opacity: 0, height: 0 }}
          animate={{ opacity: 1, height: "auto" }}
          exit={{ opacity: 0, height: 0 }}
          transition={{ duration: 0.3 }}
          className="border-b py-2"
        >
          <div className="flex justify-between items-center">
            <span>{todo.text}</span>
            <button onClick={() => onRemove(todo.id)}>Remove</button>
          </div>
        </motion.div>
      ))}
    </AnimatePresence>
  );
};
```

### Scroll Animations

```typescript
// Scroll-triggered animations
import { useInView } from 'framer-motion';
import { useRef } from 'react';

const ScrollReveal = ({ children }) => {
  const ref = useRef(null);
  const isInView = useInView(ref, { once: true });

  return (
    <motion.div
      ref={ref}
      initial={{ opacity: 0, y: 75 }}
      animate={isInView ? { opacity: 1, y: 0 } : { opacity: 0, y: 75 }}
      transition={{ duration: 0.5, delay: 0.25 }}
    >
      {children}
    </motion.div>
  );
};

// Parallax scrolling
import { useScroll, useTransform } from 'framer-motion';

const ParallaxSection = ({ children }) => {
  const { scrollYProgress } = useScroll();
  const y = useTransform(scrollYProgress, [0, 1], ['0%', '50%']);

  return (
    <motion.div style={{ y }}>
      {children}
    </motion.div>
  );
};
```

## ⚡ CSS Animation Performance

### Hardware Acceleration

```css
/* ✅ Use transform and opacity for smooth animations */
.smooth-animation {
  transform: translateX(0);
  opacity: 1;
  transition: transform 0.3s ease, opacity 0.3s ease;
}

.smooth-animation:hover {
  transform: translateX(10px);
  opacity: 0.8;
}

/* ❌ Avoid animating layout properties */
.janky-animation {
  left: 0;
  width: 100px;
  transition: left 0.3s ease, width 0.3s ease;
}

.janky-animation:hover {
  left: 10px;
  width: 110px;
}
```

### Will-Change Property

```css
/* Use will-change to optimize animations */
.will-animate {
  will-change: transform, opacity;
}

.will-animate:hover {
  transform: scale(1.1);
  opacity: 0.9;
}

/* Remove will-change after animation */
.animation-complete {
  will-change: auto;
}
```

### CSS Custom Properties for Dynamic Animations

```css
.dynamic-animation {
  --duration: 0.3s;
  --delay: 0s;
  --easing: cubic-bezier(0.4, 0, 0.2, 1);
  
  transition: transform var(--duration) var(--easing) var(--delay);
}

.fast-animation {
  --duration: 0.15s;
}

.slow-animation {
  --duration: 0.6s;
}
```

### Performance-Optimized Keyframes

```css
/* ✅ Efficient keyframe animation */
@keyframes slideIn {
  from {
    transform: translateX(-100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

/* ✅ Use transform3d for hardware acceleration */
@keyframes bounce {
  0%, 20%, 53%, 80%, to {
    transform: translate3d(0, 0, 0);
  }
  40%, 43% {
    transform: translate3d(0, -30px, 0);
  }
  70% {
    transform: translate3d(0, -15px, 0);
  }
  90% {
    transform: translate3d(0, -4px, 0);
  }
}
```

## 🎨 UX Animation Principles

### 1. **Purposeful Motion**
```typescript
// ✅ Animations should serve a purpose
const FeedbackButton = () => {
  const [clicked, setClicked] = useState(false);
  
  return (
    <motion.button
      onClick={() => setClicked(true)}
      animate={clicked ? { scale: [1, 1.2, 1] } : {}}
      transition={{ duration: 0.3 }}
      onAnimationComplete={() => setClicked(false)}
    >
      {clicked ? '✓ Saved!' : 'Save'}
    </motion.button>
  );
};
```

### 2. **Respect User Preferences**
```typescript
// Check for reduced motion preference
const useReducedMotion = () => {
  const [prefersReducedMotion, setPrefersReducedMotion] = useState(false);
  
  useEffect(() => {
    const mediaQuery = window.matchMedia('(prefers-reduced-motion: reduce)');
    setPrefersReducedMotion(mediaQuery.matches);
    
    const handleChange = () => setPrefersReducedMotion(mediaQuery.matches);
    mediaQuery.addEventListener('change', handleChange);
    
    return () => mediaQuery.removeEventListener('change', handleChange);
  }, []);
  
  return prefersReducedMotion;
};

// Use in components
const RespectfulAnimation = ({ children }) => {
  const prefersReducedMotion = useReducedMotion();
  
  return (
    <motion.div
      initial={prefersReducedMotion ? {} : { opacity: 0, y: 20 }}
      animate={prefersReducedMotion ? {} : { opacity: 1, y: 0 }}
      transition={prefersReducedMotion ? { duration: 0 } : { duration: 0.5 }}
    >
      {children}
    </motion.div>
  );
};
```

### 3. **Timing and Easing**
```typescript
// Natural easing curves
const easings = {
  easeOut: [0, 0.55, 0.45, 1],
  easeIn: [0.55, 0, 1, 0.45],
  easeInOut: [0.645, 0.045, 0.355, 1],
  anticipate: [0.175, 0.885, 0.32, 1.275],
};

// Duration guidelines
const durations = {
  fast: 0.15,      // Micro-interactions
  normal: 0.3,     // Standard transitions
  slow: 0.5,       // Complex animations
  slower: 0.8,     // Page transitions
};
```

## 📱 Responsive Animations

```typescript
// Conditional animations based on screen size
const ResponsiveAnimation = ({ children }) => {
  const [isMobile, setIsMobile] = useState(false);
  
  useEffect(() => {
    const checkMobile = () => setIsMobile(window.innerWidth < 768);
    checkMobile();
    window.addEventListener('resize', checkMobile);
    return () => window.removeEventListener('resize', checkMobile);
  }, []);
  
  return (
    <motion.div
      initial={{ opacity: 0 }}
      animate={{ opacity: 1 }}
      transition={{ 
        duration: isMobile ? 0.2 : 0.5,
        ease: isMobile ? "easeOut" : "anticipate"
      }}
    >
      {children}
    </motion.div>
  );
};
```

## 📋 Best Practices

### 1. **Performance First**
- Use `transform` and `opacity` for smooth animations
- Avoid animating `width`, `height`, `top`, `left`
- Use `will-change` sparingly and remove after animation
- Prefer CSS animations for simple transitions

### 2. **Accessibility**
- Respect `prefers-reduced-motion`
- Provide alternative feedback for users who disable animations
- Ensure animations don't cause seizures (avoid rapid flashing)
- Don't rely solely on animation to convey information

### 3. **User Experience**
- Keep animations fast (< 500ms for most interactions)
- Use consistent easing throughout your app
- Provide immediate feedback for user actions
- Don't animate everything - be selective

### 4. **Code Organization**
- Create reusable animation variants
- Extract common animations into custom hooks
- Use TypeScript for animation props
- Test animations on various devices

## 🔗 Related Topics
- [[React]] - Component patterns and hooks
- [[Performance Optimization]] - Animation performance
- [[Accessibility]] - Motion accessibility guidelines
- [[CSS]] - CSS animation fundamentals

---
*Great animations enhance user experience without getting in the way. Focus on purposeful motion that guides users and provides helpful feedback.* 