# CSS - Cascading Style Sheets

> **Quick Summary**: CSS controls the visual presentation of HTML. Master selectors, layout techniques, and responsive design for modern web development.

## 🎯 Selectors

### Basic Selectors
```css
/* Element selector */
p { color: blue; }

/* Class selector */
.highlight { background: yellow; }

/* ID selector */
#header { font-size: 24px; }

/* Universal selector */
* { box-sizing: border-box; }

/* Attribute selector */
[type="email"] { border: 2px solid blue; }
[href^="https"] { color: green; }
[class*="btn"] { padding: 10px; }
```

### Combinators
```css
/* Descendant selector */
.container p { margin: 16px 0; }

/* Child selector */
.nav > li { display: inline-block; }

/* Adjacent sibling */
h1 + p { font-size: 18px; }

/* General sibling */
h1 ~ p { color: gray; }
```

### Pseudo-classes
```css
/* Link states */
a:link { color: blue; }
a:visited { color: purple; }
a:hover { color: red; }
a:active { color: orange; }

/* Form states */
input:focus { outline: 2px solid blue; }
input:valid { border-color: green; }
input:invalid { border-color: red; }
input:disabled { opacity: 0.5; }

/* Structural pseudo-classes */
li:first-child { font-weight: bold; }
li:last-child { margin-bottom: 0; }
li:nth-child(odd) { background: #f0f0f0; }
li:nth-child(3n) { color: red; }
```

### Pseudo-elements
```css
/* Before and after */
.quote::before { content: """; }
.quote::after { content: """; }

/* First letter and line */
p::first-letter { font-size: 2em; }
p::first-line { font-weight: bold; }

/* Selection styling */
::selection { background: yellow; }
```

## 📦 Box Model

### Understanding the Box Model
```css
.box {
    /* Content area */
    width: 200px;
    height: 100px;
    
    /* Padding (inside the border) */
    padding: 20px;
    
    /* Border */
    border: 2px solid black;
    
    /* Margin (outside the border) */
    margin: 10px;
    
    /* Box-sizing controls how width/height are calculated */
    box-sizing: border-box; /* Includes padding and border in width/height */
}

/* Reset for consistent box model */
*, *::before, *::after {
    box-sizing: border-box;
}
```

## 🏗️ Layout Techniques

### Flexbox
```css
/* Flex container */
.flex-container {
    display: flex;
    
    /* Direction */
    flex-direction: row; /* row | column | row-reverse | column-reverse */
    
    /* Wrapping */
    flex-wrap: wrap; /* nowrap | wrap | wrap-reverse */
    
    /* Shorthand */
    flex-flow: row wrap;
    
    /* Alignment */
    justify-content: center; /* flex-start | flex-end | center | space-between | space-around | space-evenly */
    align-items: center; /* flex-start | flex-end | center | stretch | baseline */
    align-content: center; /* For wrapped lines */
    
    /* Gap */
    gap: 20px; /* Space between items */
}

/* Flex items */
.flex-item {
    /* Grow, shrink, basis */
    flex: 1; /* Shorthand for flex-grow: 1, flex-shrink: 1, flex-basis: 0% */
    flex-grow: 1; /* How much to grow */
    flex-shrink: 0; /* How much to shrink */
    flex-basis: 200px; /* Initial size */
    
    /* Individual alignment */
    align-self: flex-end;
}

/* Common flex patterns */
.center-content {
    display: flex;
    justify-content: center;
    align-items: center;
}

.space-between {
    display: flex;
    justify-content: space-between;
    align-items: center;
}
```

### CSS Grid
```css
/* Grid container */
.grid-container {
    display: grid;
    
    /* Define columns and rows */
    grid-template-columns: 1fr 2fr 1fr; /* Fractional units */
    grid-template-columns: repeat(3, 1fr); /* Repeat function */
    grid-template-columns: 200px auto 100px; /* Mixed units */
    
    grid-template-rows: 100px auto 50px;
    
    /* Gap between grid items */
    gap: 20px;
    grid-gap: 20px; /* Older syntax */
    
    /* Named grid lines */
    grid-template-columns: [start] 1fr [middle] 1fr [end];
    
    /* Grid areas */
    grid-template-areas: 
        "header header header"
        "sidebar main main"
        "footer footer footer";
}

/* Grid items */
.grid-item {
    /* Position by line numbers */
    grid-column: 1 / 3; /* From line 1 to line 3 */
    grid-row: 2 / 4;
    
    /* Shorthand */
    grid-area: 2 / 1 / 4 / 3; /* row-start / col-start / row-end / col-end */
    
    /* Named areas */
    grid-area: header;
    
    /* Span syntax */
    grid-column: span 2; /* Span 2 columns */
}

/* Responsive grid */
.responsive-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 20px;
}
```

### Positioning
```css
/* Static (default) */
.static { position: static; }

/* Relative - positioned relative to normal position */
.relative {
    position: relative;
    top: 10px;
    left: 20px;
}

/* Absolute - positioned relative to nearest positioned ancestor */
.absolute {
    position: absolute;
    top: 0;
    right: 0;
}

/* Fixed - positioned relative to viewport */
.fixed {
    position: fixed;
    bottom: 20px;
    right: 20px;
}

/* Sticky - switches between relative and fixed */
.sticky {
    position: sticky;
    top: 0; /* Sticks when scrolled to top */
}
```

## 🎨 Colors and Typography

### Colors
```css
/* Color formats */
.colors {
    /* Named colors */
    color: red;
    
    /* Hex */
    color: #ff0000;
    color: #f00; /* Shorthand */
    
    /* RGB */
    color: rgb(255, 0, 0);
    color: rgba(255, 0, 0, 0.5); /* With alpha */
    
    /* HSL */
    color: hsl(0, 100%, 50%);
    color: hsla(0, 100%, 50%, 0.5); /* With alpha */
    
    /* CSS Variables */
    color: var(--primary-color);
}

/* CSS Custom Properties (Variables) */
:root {
    --primary-color: #007bff;
    --secondary-color: #6c757d;
    --font-size-base: 16px;
}

.component {
    color: var(--primary-color);
    font-size: var(--font-size-base);
}
```

### Typography
```css
/* Font properties */
.typography {
    /* Font family with fallbacks */
    font-family: 'Helvetica Neue', Arial, sans-serif;
    
    /* Font size */
    font-size: 16px; /* Absolute */
    font-size: 1.2em; /* Relative to parent */
    font-size: 1.2rem; /* Relative to root */
    
    /* Font weight */
    font-weight: normal; /* 400 */
    font-weight: bold; /* 700 */
    font-weight: 300; /* Light */
    
    /* Font style */
    font-style: italic;
    
    /* Line height */
    line-height: 1.5; /* Unitless preferred */
    line-height: 24px; /* Absolute */
    
    /* Letter and word spacing */
    letter-spacing: 0.1em;
    word-spacing: 0.2em;
    
    /* Text properties */
    text-align: center;
    text-decoration: underline;
    text-transform: uppercase;
    
    /* Text shadow */
    text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
}

/* Web fonts */
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap');

.custom-font {
    font-family: 'Inter', sans-serif;
}
```

## 📱 Responsive Design

### Media Queries
```css
/* Mobile-first approach */
.responsive {
    padding: 10px;
}

/* Tablet */
@media (min-width: 768px) {
    .responsive {
        padding: 20px;
    }
}

/* Desktop */
@media (min-width: 1024px) {
    .responsive {
        padding: 30px;
    }
}

/* Common breakpoints */
@media (max-width: 767px) { /* Mobile */ }
@media (min-width: 768px) and (max-width: 1023px) { /* Tablet */ }
@media (min-width: 1024px) { /* Desktop */ }

/* Orientation */
@media (orientation: landscape) { }
@media (orientation: portrait) { }

/* High DPI displays */
@media (-webkit-min-device-pixel-ratio: 2) { }
```

### Responsive Units
```css
.responsive-units {
    /* Viewport units */
    width: 100vw; /* 100% of viewport width */
    height: 100vh; /* 100% of viewport height */
    font-size: 4vw; /* 4% of viewport width */
    
    /* Relative units */
    padding: 2rem; /* Relative to root font size */
    margin: 1.5em; /* Relative to element font size */
    
    /* Percentage */
    width: 50%; /* 50% of parent width */
    
    /* Calc function */
    width: calc(100% - 40px);
    font-size: calc(16px + 1vw);
}
```

## ✨ Animations and Transitions

### Transitions
```css
.transition-element {
    background-color: blue;
    transform: scale(1);
    
    /* Transition property */
    transition: all 0.3s ease;
    transition: background-color 0.3s ease, transform 0.2s ease-out;
    
    /* Transition properties */
    transition-property: background-color, transform;
    transition-duration: 0.3s, 0.2s;
    transition-timing-function: ease, ease-out;
    transition-delay: 0s, 0.1s;
}

.transition-element:hover {
    background-color: red;
    transform: scale(1.1);
}

/* Common timing functions */
.timing-functions {
    transition-timing-function: ease; /* Default */
    transition-timing-function: linear;
    transition-timing-function: ease-in;
    transition-timing-function: ease-out;
    transition-timing-function: ease-in-out;
    transition-timing-function: cubic-bezier(0.25, 0.1, 0.25, 1);
}
```

### Keyframe Animations
```css
/* Define keyframes */
@keyframes fadeIn {
    from { opacity: 0; }
    to { opacity: 1; }
}

@keyframes slideIn {
    0% { transform: translateX(-100%); }
    50% { transform: translateX(10px); }
    100% { transform: translateX(0); }
}

/* Apply animation */
.animated-element {
    animation: fadeIn 1s ease-in-out;
    animation: slideIn 0.5s ease-out forwards;
    
    /* Animation properties */
    animation-name: fadeIn;
    animation-duration: 1s;
    animation-timing-function: ease-in-out;
    animation-delay: 0.2s;
    animation-iteration-count: infinite; /* or number */
    animation-direction: alternate; /* normal | reverse | alternate | alternate-reverse */
    animation-fill-mode: forwards; /* none | forwards | backwards | both */
    animation-play-state: paused; /* running | paused */
}

/* Common animations */
@keyframes spin {
    from { transform: rotate(0deg); }
    to { transform: rotate(360deg); }
}

@keyframes bounce {
    0%, 20%, 53%, 80%, 100% { transform: translate3d(0,0,0); }
    40%, 43% { transform: translate3d(0,-30px,0); }
    70% { transform: translate3d(0,-15px,0); }
    90% { transform: translate3d(0,-4px,0); }
}
```

### Transforms
```css
.transform-examples {
    /* 2D Transforms */
    transform: translate(50px, 100px);
    transform: translateX(50px);
    transform: translateY(100px);
    transform: scale(1.5);
    transform: scaleX(2);
    transform: rotate(45deg);
    transform: skew(30deg, 20deg);
    
    /* 3D Transforms */
    transform: translate3d(50px, 100px, 0);
    transform: rotateX(45deg);
    transform: rotateY(45deg);
    transform: rotateZ(45deg);
    
    /* Multiple transforms */
    transform: translate(50px, 100px) rotate(45deg) scale(1.2);
    
    /* Transform origin */
    transform-origin: center center; /* Default */
    transform-origin: top left;
    transform-origin: 50% 50%;
}
```

## 🎨 Common Patterns

### Button Styles
```css
.btn {
    display: inline-block;
    padding: 12px 24px;
    border: none;
    border-radius: 4px;
    font-size: 16px;
    font-weight: 600;
    text-decoration: none;
    text-align: center;
    cursor: pointer;
    transition: all 0.2s ease;
    
    /* Primary button */
    background-color: #007bff;
    color: white;
}

.btn:hover {
    background-color: #0056b3;
    transform: translateY(-1px);
    box-shadow: 0 4px 8px rgba(0,0,0,0.2);
}

.btn:active {
    transform: translateY(0);
}

.btn:disabled {
    opacity: 0.6;
    cursor: not-allowed;
}
```

### Card Component
```css
.card {
    background: white;
    border-radius: 8px;
    box-shadow: 0 2px 10px rgba(0,0,0,0.1);
    overflow: hidden;
    transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.card:hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 20px rgba(0,0,0,0.15);
}

.card-header {
    padding: 20px;
    border-bottom: 1px solid #eee;
}

.card-body {
    padding: 20px;
}

.card-footer {
    padding: 15px 20px;
    background-color: #f8f9fa;
    border-top: 1px solid #eee;
}
```

### Navigation Menu
```css
.nav {
    display: flex;
    list-style: none;
    margin: 0;
    padding: 0;
}

.nav-item {
    margin-right: 20px;
}

.nav-link {
    display: block;
    padding: 10px 15px;
    color: #333;
    text-decoration: none;
    border-radius: 4px;
    transition: background-color 0.2s ease;
}

.nav-link:hover,
.nav-link.active {
    background-color: #007bff;
    color: white;
}

/* Mobile menu */
@media (max-width: 767px) {
    .nav {
        flex-direction: column;
    }
    
    .nav-item {
        margin-right: 0;
        margin-bottom: 5px;
    }
}
```

## 🔧 CSS Architecture

### BEM Methodology
```css
/* Block */
.card { }

/* Element */
.card__header { }
.card__body { }
.card__footer { }

/* Modifier */
.card--featured { }
.card--large { }
.card__header--dark { }

/* Example usage in HTML */
/*
<div class="card card--featured">
    <div class="card__header card__header--dark">Header</div>
    <div class="card__body">Body content</div>
    <div class="card__footer">Footer</div>
</div>
*/
```

### CSS Custom Properties (Variables)
```css
:root {
    /* Colors */
    --color-primary: #007bff;
    --color-secondary: #6c757d;
    --color-success: #28a745;
    --color-danger: #dc3545;
    
    /* Spacing */
    --spacing-xs: 4px;
    --spacing-sm: 8px;
    --spacing-md: 16px;
    --spacing-lg: 24px;
    --spacing-xl: 32px;
    
    /* Typography */
    --font-family-base: 'Inter', sans-serif;
    --font-size-sm: 14px;
    --font-size-base: 16px;
    --font-size-lg: 18px;
    --font-size-xl: 24px;
    
    /* Breakpoints */
    --breakpoint-sm: 576px;
    --breakpoint-md: 768px;
    --breakpoint-lg: 992px;
    --breakpoint-xl: 1200px;
}

/* Usage */
.component {
    color: var(--color-primary);
    padding: var(--spacing-md);
    font-family: var(--font-family-base);
}
```

## ✅ Quick Checklist

- [ ] Use consistent naming conventions (BEM recommended)
- [ ] Implement mobile-first responsive design
- [ ] Use CSS custom properties for theming
- [ ] Optimize for performance (minimize repaints/reflows)
- [ ] Test across different browsers
- [ ] Validate CSS with W3C validator
- [ ] Use semantic class names
- [ ] Implement proper focus states for accessibility

## 🔗 Related Topics
- [[HTML]] - Structure your content
- [[Tailwind CSS]] - Utility-first CSS framework
- [[JavaScript]] - Add interactivity to styled elements

---
*Remember: CSS is about presentation and user experience. Keep it maintainable and performant!* 