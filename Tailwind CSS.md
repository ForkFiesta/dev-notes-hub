# Tailwind CSS - Utility-First CSS Framework

> **Quick Summary**: Tailwind CSS is a utility-first framework that provides low-level utility classes to build custom designs quickly without writing custom CSS.

## 🚀 Getting Started

### Installation
```bash
# Install Tailwind CSS
npm install -D tailwindcss
npx tailwindcss init

# With PostCSS and Autoprefixer
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

### Configuration
```javascript
// tailwind.config.js
module.exports = {
  content: [
    "./src/**/*.{html,js,jsx,ts,tsx}",
    "./public/index.html"
  ],
  theme: {
    extend: {
      colors: {
        primary: '#3B82F6',
        secondary: '#64748B'
      }
    },
  },
  plugins: [],
}
```

### CSS Setup
```css
/* styles.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

/* Custom base styles */
@layer base {
  h1 {
    @apply text-2xl font-bold;
  }
}

/* Custom components */
@layer components {
  .btn-primary {
    @apply bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded;
  }
}

/* Custom utilities */
@layer utilities {
  .text-shadow {
    text-shadow: 2px 2px 4px rgba(0,0,0,0.1);
  }
}
```

## 🎨 Layout & Spacing

### Display
```html
<!-- Display utilities -->
<div class="block">Block element</div>
<div class="inline">Inline element</div>
<div class="inline-block">Inline-block element</div>
<div class="flex">Flex container</div>
<div class="inline-flex">Inline flex container</div>
<div class="grid">Grid container</div>
<div class="hidden">Hidden element</div>

<!-- Responsive display -->
<div class="hidden md:block">Hidden on mobile, visible on desktop</div>
<div class="block md:hidden">Visible on mobile, hidden on desktop</div>
```

### Flexbox
```html
<!-- Flex container -->
<div class="flex">
  <div>Item 1</div>
  <div>Item 2</div>
  <div>Item 3</div>
</div>

<!-- Flex direction -->
<div class="flex flex-col">Vertical stack</div>
<div class="flex flex-row">Horizontal stack</div>
<div class="flex flex-col-reverse">Reverse vertical</div>

<!-- Justify content -->
<div class="flex justify-start">Start</div>
<div class="flex justify-center">Center</div>
<div class="flex justify-end">End</div>
<div class="flex justify-between">Space between</div>
<div class="flex justify-around">Space around</div>
<div class="flex justify-evenly">Space evenly</div>

<!-- Align items -->
<div class="flex items-start">Align start</div>
<div class="flex items-center">Align center</div>
<div class="flex items-end">Align end</div>
<div class="flex items-stretch">Stretch</div>

<!-- Flex grow/shrink -->
<div class="flex">
  <div class="flex-1">Grows to fill space</div>
  <div class="flex-none">Fixed size</div>
  <div class="flex-auto">Auto sizing</div>
</div>

<!-- Gap -->
<div class="flex gap-4">Items with 1rem gap</div>
<div class="flex gap-x-2 gap-y-4">Different horizontal/vertical gaps</div>
```

### Grid
```html
<!-- Grid container -->
<div class="grid grid-cols-3 gap-4">
  <div>Item 1</div>
  <div>Item 2</div>
  <div>Item 3</div>
</div>

<!-- Responsive grid -->
<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
  <!-- Items -->
</div>

<!-- Grid column span -->
<div class="grid grid-cols-4 gap-4">
  <div class="col-span-2">Spans 2 columns</div>
  <div>Item</div>
  <div>Item</div>
</div>

<!-- Grid row span -->
<div class="grid grid-rows-3 gap-4">
  <div class="row-span-2">Spans 2 rows</div>
  <div>Item</div>
  <div>Item</div>
</div>

<!-- Auto-fit grid -->
<div class="grid grid-cols-[repeat(auto-fit,minmax(250px,1fr))] gap-4">
  <!-- Responsive items -->
</div>
```

### Spacing
```html
<!-- Margin -->
<div class="m-4">Margin all sides</div>
<div class="mx-4">Margin horizontal</div>
<div class="my-4">Margin vertical</div>
<div class="mt-4 mr-2 mb-4 ml-2">Individual margins</div>
<div class="m-auto">Auto margin (centering)</div>

<!-- Padding -->
<div class="p-4">Padding all sides</div>
<div class="px-4">Padding horizontal</div>
<div class="py-4">Padding vertical</div>
<div class="pt-4 pr-2 pb-4 pl-2">Individual padding</div>

<!-- Spacing scale: 0, 0.5, 1, 1.5, 2, 2.5, 3, 3.5, 4, 5, 6, 8, 10, 12, 16, 20, 24, 32, 40, 48, 56, 64 -->
<div class="p-0">0px</div>
<div class="p-1">4px</div>
<div class="p-4">16px</div>
<div class="p-8">32px</div>
```

## 🎯 Typography

### Font Family
```html
<!-- Font families -->
<p class="font-sans">Sans-serif font</p>
<p class="font-serif">Serif font</p>
<p class="font-mono">Monospace font</p>

<!-- Custom fonts (define in config) -->
<p class="font-custom">Custom font family</p>
```

### Font Size
```html
<!-- Font sizes -->
<p class="text-xs">Extra small (12px)</p>
<p class="text-sm">Small (14px)</p>
<p class="text-base">Base (16px)</p>
<p class="text-lg">Large (18px)</p>
<p class="text-xl">Extra large (20px)</p>
<p class="text-2xl">2X large (24px)</p>
<p class="text-3xl">3X large (30px)</p>
<p class="text-4xl">4X large (36px)</p>
<p class="text-5xl">5X large (48px)</p>
<p class="text-6xl">6X large (60px)</p>
```

### Font Weight & Style
```html
<!-- Font weights -->
<p class="font-thin">Thin (100)</p>
<p class="font-light">Light (300)</p>
<p class="font-normal">Normal (400)</p>
<p class="font-medium">Medium (500)</p>
<p class="font-semibold">Semibold (600)</p>
<p class="font-bold">Bold (700)</p>
<p class="font-extrabold">Extra bold (800)</p>
<p class="font-black">Black (900)</p>

<!-- Font style -->
<p class="italic">Italic text</p>
<p class="not-italic">Not italic</p>
```

### Text Alignment & Decoration
```html
<!-- Text alignment -->
<p class="text-left">Left aligned</p>
<p class="text-center">Center aligned</p>
<p class="text-right">Right aligned</p>
<p class="text-justify">Justified</p>

<!-- Text decoration -->
<p class="underline">Underlined text</p>
<p class="line-through">Strikethrough text</p>
<p class="no-underline">No underline</p>

<!-- Text transform -->
<p class="uppercase">UPPERCASE TEXT</p>
<p class="lowercase">lowercase text</p>
<p class="capitalize">Capitalize Text</p>
<p class="normal-case">Normal Case</p>
```

## 🎨 Colors

### Text Colors
```html
<!-- Basic colors -->
<p class="text-black">Black text</p>
<p class="text-white">White text</p>
<p class="text-gray-500">Gray text</p>
<p class="text-red-500">Red text</p>
<p class="text-blue-500">Blue text</p>
<p class="text-green-500">Green text</p>

<!-- Color scale (50, 100, 200, 300, 400, 500, 600, 700, 800, 900) -->
<p class="text-blue-100">Light blue</p>
<p class="text-blue-500">Medium blue</p>
<p class="text-blue-900">Dark blue</p>

<!-- Custom colors (define in config) -->
<p class="text-primary">Primary color</p>
<p class="text-secondary">Secondary color</p>
```

### Background Colors
```html
<!-- Background colors -->
<div class="bg-white">White background</div>
<div class="bg-gray-100">Light gray background</div>
<div class="bg-blue-500">Blue background</div>
<div class="bg-gradient-to-r from-blue-500 to-purple-600">Gradient background</div>

<!-- Opacity -->
<div class="bg-blue-500 bg-opacity-50">50% opacity background</div>
<div class="bg-blue-500/50">50% opacity (modern syntax)</div>
```

## 📱 Responsive Design

### Breakpoints
```html
<!-- Tailwind breakpoints: sm (640px), md (768px), lg (1024px), xl (1280px), 2xl (1536px) -->

<!-- Mobile-first approach -->
<div class="text-sm md:text-base lg:text-lg">
  Responsive text size
</div>

<!-- Hide/show at different breakpoints -->
<div class="block md:hidden">Mobile only</div>
<div class="hidden md:block">Desktop only</div>

<!-- Responsive grid -->
<div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4">
  <!-- Responsive columns -->
</div>

<!-- Responsive spacing -->
<div class="p-4 md:p-8 lg:p-12">
  Responsive padding
</div>
```

### Container
```html
<!-- Responsive container -->
<div class="container mx-auto px-4">
  Centered container with responsive max-width
</div>

<!-- Custom container sizes -->
<div class="max-w-sm mx-auto">Small container</div>
<div class="max-w-md mx-auto">Medium container</div>
<div class="max-w-lg mx-auto">Large container</div>
<div class="max-w-xl mx-auto">Extra large container</div>
<div class="max-w-2xl mx-auto">2X large container</div>
```

## 🎭 Components & Patterns

### Buttons
```html
<!-- Primary button -->
<button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded">
  Primary Button
</button>

<!-- Secondary button -->
<button class="bg-transparent hover:bg-blue-500 text-blue-700 font-semibold hover:text-white py-2 px-4 border border-blue-500 hover:border-transparent rounded">
  Secondary Button
</button>

<!-- Button sizes -->
<button class="bg-blue-500 text-white px-2 py-1 text-xs rounded">Small</button>
<button class="bg-blue-500 text-white px-4 py-2 text-sm rounded">Medium</button>
<button class="bg-blue-500 text-white px-6 py-3 text-lg rounded">Large</button>

<!-- Disabled button -->
<button class="bg-blue-500 text-white px-4 py-2 rounded opacity-50 cursor-not-allowed" disabled>
  Disabled
</button>
```

### Cards
```html
<!-- Basic card -->
<div class="max-w-sm rounded overflow-hidden shadow-lg">
  <img class="w-full" src="image.jpg" alt="Card image">
  <div class="px-6 py-4">
    <div class="font-bold text-xl mb-2">Card Title</div>
    <p class="text-gray-700 text-base">
      Card description goes here.
    </p>
  </div>
  <div class="px-6 pt-4 pb-2">
    <span class="inline-block bg-gray-200 rounded-full px-3 py-1 text-sm font-semibold text-gray-700 mr-2 mb-2">
      #tag1
    </span>
  </div>
</div>

<!-- Hover effects -->
<div class="max-w-sm rounded overflow-hidden shadow-lg hover:shadow-xl transition-shadow duration-300">
  <!-- Card content -->
</div>
```

### Forms
```html
<!-- Form elements -->
<form class="w-full max-w-sm">
  <!-- Text input -->
  <div class="mb-4">
    <label class="block text-gray-700 text-sm font-bold mb-2" for="username">
      Username
    </label>
    <input class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline" 
           id="username" type="text" placeholder="Username">
  </div>

  <!-- Password input -->
  <div class="mb-6">
    <label class="block text-gray-700 text-sm font-bold mb-2" for="password">
      Password
    </label>
    <input class="shadow appearance-none border border-red-500 rounded w-full py-2 px-3 text-gray-700 mb-3 leading-tight focus:outline-none focus:shadow-outline" 
           id="password" type="password" placeholder="******************">
    <p class="text-red-500 text-xs italic">Please choose a password.</p>
  </div>

  <!-- Submit button -->
  <div class="flex items-center justify-between">
    <button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline" 
            type="button">
      Sign In
    </button>
  </div>
</form>

<!-- Select dropdown -->
<select class="block appearance-none w-full bg-white border border-gray-400 hover:border-gray-500 px-4 py-2 pr-8 rounded shadow leading-tight focus:outline-none focus:shadow-outline">
  <option>Option 1</option>
  <option>Option 2</option>
  <option>Option 3</option>
</select>
```

### Navigation
```html
<!-- Horizontal navigation -->
<nav class="flex items-center justify-between flex-wrap bg-blue-500 p-6">
  <div class="flex items-center flex-shrink-0 text-white mr-6">
    <span class="font-semibold text-xl tracking-tight">Brand</span>
  </div>
  <div class="w-full block flex-grow lg:flex lg:items-center lg:w-auto">
    <div class="text-sm lg:flex-grow">
      <a href="#" class="block mt-4 lg:inline-block lg:mt-0 text-blue-200 hover:text-white mr-4">
        Home
      </a>
      <a href="#" class="block mt-4 lg:inline-block lg:mt-0 text-blue-200 hover:text-white mr-4">
        About
      </a>
      <a href="#" class="block mt-4 lg:inline-block lg:mt-0 text-blue-200 hover:text-white">
        Contact
      </a>
    </div>
  </div>
</nav>

<!-- Mobile-responsive navigation -->
<nav class="bg-gray-800">
  <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
    <div class="flex items-center justify-between h-16">
      <div class="flex items-center">
        <div class="hidden md:block">
          <div class="ml-10 flex items-baseline space-x-4">
            <a href="#" class="text-gray-300 hover:bg-gray-700 hover:text-white px-3 py-2 rounded-md text-sm font-medium">Home</a>
            <a href="#" class="text-gray-300 hover:bg-gray-700 hover:text-white px-3 py-2 rounded-md text-sm font-medium">About</a>
          </div>
        </div>
      </div>
    </div>
  </div>
</nav>
```

## ✨ Animations & Transitions

### Transitions
```html
<!-- Basic transitions -->
<button class="bg-blue-500 text-white px-4 py-2 rounded transition-colors duration-300 hover:bg-blue-700">
  Hover me
</button>

<!-- Multiple properties -->
<div class="transform transition-all duration-500 hover:scale-110 hover:rotate-3">
  Transform on hover
</div>

<!-- Transition timing -->
<div class="transition-opacity duration-75 ease-in">Fast fade</div>
<div class="transition-opacity duration-300 ease-out">Medium fade</div>
<div class="transition-opacity duration-700 ease-in-out">Slow fade</div>
```

### Transforms
```html
<!-- Scale -->
<div class="transform scale-75">75% scale</div>
<div class="transform scale-110 hover:scale-125">Scale on hover</div>

<!-- Rotate -->
<div class="transform rotate-45">45 degree rotation</div>
<div class="transform hover:rotate-180 transition-transform duration-500">Rotate on hover</div>

<!-- Translate -->
<div class="transform translate-x-4 translate-y-2">Translate</div>
<div class="transform hover:translate-y-1 transition-transform">Lift on hover</div>

<!-- Skew -->
<div class="transform skew-x-12 skew-y-6">Skewed element</div>
```

### Animations
```html
<!-- Built-in animations -->
<div class="animate-spin">Spinning</div>
<div class="animate-ping">Pinging</div>
<div class="animate-pulse">Pulsing</div>
<div class="animate-bounce">Bouncing</div>

<!-- Custom animations (define in config) -->
<div class="animate-fade-in">Custom fade in</div>
```

## 🔧 Customization

### Extending Theme
```javascript
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      colors: {
        'brand-blue': '#1fb6ff',
        'brand-purple': '#7e5bef',
        'brand-pink': '#ff49db',
      },
      fontFamily: {
        'custom': ['Custom Font', 'sans-serif'],
      },
      spacing: {
        '72': '18rem',
        '84': '21rem',
        '96': '24rem',
      },
      animation: {
        'fade-in': 'fadeIn 0.5s ease-in-out',
        'slide-up': 'slideUp 0.3s ease-out',
      },
      keyframes: {
        fadeIn: {
          '0%': { opacity: '0' },
          '100%': { opacity: '1' },
        },
        slideUp: {
          '0%': { transform: 'translateY(100%)' },
          '100%': { transform: 'translateY(0)' },
        },
      }
    }
  }
}
```

### Custom Components
```css
@layer components {
  .btn {
    @apply px-4 py-2 rounded font-semibold text-sm;
  }
  
  .btn-primary {
    @apply bg-blue-500 text-white hover:bg-blue-700;
  }
  
  .btn-secondary {
    @apply bg-gray-200 text-gray-800 hover:bg-gray-300;
  }
  
  .card {
    @apply bg-white rounded-lg shadow-md overflow-hidden;
  }
  
  .card-header {
    @apply px-6 py-4 border-b border-gray-200;
  }
  
  .card-body {
    @apply px-6 py-4;
  }
}
```

## 🎯 Best Practices

### Utility Organization
```html
<!-- Group related utilities -->
<div class="
  flex items-center justify-between
  w-full max-w-md mx-auto
  p-6 bg-white rounded-lg shadow-lg
  border border-gray-200
">
  Content
</div>

<!-- Use component classes for repeated patterns -->
<button class="btn btn-primary">Primary Button</button>
<button class="btn btn-secondary">Secondary Button</button>
```

### Responsive Design Patterns
```html
<!-- Mobile-first responsive design -->
<div class="
  grid grid-cols-1 gap-4
  sm:grid-cols-2 sm:gap-6
  lg:grid-cols-3 lg:gap-8
  xl:grid-cols-4
">
  <!-- Grid items -->
</div>

<!-- Responsive typography -->
<h1 class="
  text-2xl font-bold
  sm:text-3xl
  lg:text-4xl
  xl:text-5xl
">
  Responsive Heading
</h1>
```

## ✅ Quick Checklist

- [ ] Configure Tailwind for your project
- [ ] Use mobile-first responsive design
- [ ] Leverage utility classes instead of custom CSS
- [ ] Create component classes for repeated patterns
- [ ] Use consistent spacing scale
- [ ] Implement proper color contrast
- [ ] Optimize for performance with PurgeCSS
- [ ] Test across different screen sizes
- [ ] Use semantic HTML with Tailwind classes
- [ ] Document custom utilities and components

## 🔗 Related Topics
- [[CSS]] - Understanding CSS fundamentals
- [[React]] - Using Tailwind with React components
- [[HTML]] - Semantic markup with Tailwind styling

---
*Remember: Tailwind is about rapid development with utility classes. Embrace the utility-first approach for maximum productivity!* 