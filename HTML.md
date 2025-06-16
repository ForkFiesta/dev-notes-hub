# HTML - HyperText Markup Language

> **Quick Summary**: HTML provides the structure and semantic meaning to web content. Focus on semantic elements and accessibility.

## 🏗️ Document Structure

### Basic HTML5 Template
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="description" content="Page description for SEO">
    <title>Page Title</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <header>
        <nav><!-- Navigation --></nav>
    </header>
    
    <main>
        <article><!-- Main content --></article>
        <aside><!-- Sidebar content --></aside>
    </main>
    
    <footer>
        <!-- Footer content -->
    </footer>
    
    <script src="script.js"></script>
</body>
</html>
```

## 🎯 Semantic Elements

### Page Structure
```html
<!-- Use semantic elements for better accessibility and SEO -->
<header>     <!-- Page/section header -->
<nav>        <!-- Navigation links -->
<main>       <!-- Main content area -->
<article>    <!-- Standalone content -->
<section>    <!-- Thematic grouping -->
<aside>      <!-- Sidebar/related content -->
<footer>     <!-- Page/section footer -->
```

### Content Elements
```html
<!-- Headings (h1 should be unique per page) -->
<h1>Main Page Title</h1>
<h2>Section Title</h2>
<h3>Subsection Title</h3>

<!-- Text content -->
<p>Paragraph text with <strong>important</strong> and <em>emphasized</em> content.</p>
<blockquote cite="source-url">
    <p>Quote text here</p>
    <footer>— <cite>Author Name</cite></footer>
</blockquote>

<!-- Lists -->
<ul>
    <li>Unordered list item</li>
    <li>Another item</li>
</ul>

<ol>
    <li>Ordered list item</li>
    <li>Second item</li>
</ol>

<dl>
    <dt>Definition term</dt>
    <dd>Definition description</dd>
</dl>
```

## 🔗 Links and Media

### Links
```html
<!-- Internal links -->
<a href="#section-id">Link to section</a>
<a href="page.html">Link to page</a>

<!-- External links -->
<a href="https://example.com" target="_blank" rel="noopener noreferrer">
    External link
</a>

<!-- Email and phone -->
<a href="mailto:email@example.com">Send email</a>
<a href="tel:+1234567890">Call us</a>
```

### Images
```html
<!-- Always include alt text for accessibility -->
<img src="image.jpg" alt="Descriptive text" width="300" height="200">

<!-- Responsive images -->
<picture>
    <source media="(min-width: 800px)" srcset="large.jpg">
    <source media="(min-width: 400px)" srcset="medium.jpg">
    <img src="small.jpg" alt="Responsive image">
</picture>

<!-- Figure with caption -->
<figure>
    <img src="chart.jpg" alt="Sales data chart">
    <figcaption>Q4 2023 Sales Performance</figcaption>
</figure>
```

## 📝 Forms

### Form Structure
```html
<form action="/submit" method="POST">
    <!-- Text inputs -->
    <div class="form-group">
        <label for="name">Full Name *</label>
        <input type="text" id="name" name="name" required>
    </div>
    
    <!-- Email input -->
    <div class="form-group">
        <label for="email">Email Address *</label>
        <input type="email" id="email" name="email" required>
    </div>
    
    <!-- Select dropdown -->
    <div class="form-group">
        <label for="country">Country</label>
        <select id="country" name="country">
            <option value="">Choose a country</option>
            <option value="us">United States</option>
            <option value="ca">Canada</option>
        </select>
    </div>
    
    <!-- Radio buttons -->
    <fieldset>
        <legend>Preferred Contact Method</legend>
        <input type="radio" id="contact-email" name="contact" value="email">
        <label for="contact-email">Email</label>
        
        <input type="radio" id="contact-phone" name="contact" value="phone">
        <label for="contact-phone">Phone</label>
    </fieldset>
    
    <!-- Checkboxes -->
    <div class="form-group">
        <input type="checkbox" id="newsletter" name="newsletter" value="yes">
        <label for="newsletter">Subscribe to newsletter</label>
    </div>
    
    <!-- Textarea -->
    <div class="form-group">
        <label for="message">Message</label>
        <textarea id="message" name="message" rows="4" cols="50"></textarea>
    </div>
    
    <!-- Submit button -->
    <button type="submit">Send Message</button>
</form>
```

### Input Types
```html
<!-- HTML5 input types for better UX and validation -->
<input type="text">        <!-- Plain text -->
<input type="email">       <!-- Email validation -->
<input type="password">    <!-- Hidden text -->
<input type="number">      <!-- Numeric input -->
<input type="tel">         <!-- Phone number -->
<input type="url">         <!-- URL validation -->
<input type="date">        <!-- Date picker -->
<input type="time">        <!-- Time picker -->
<input type="color">       <!-- Color picker -->
<input type="range">       <!-- Slider -->
<input type="file">        <!-- File upload -->
<input type="search">      <!-- Search box -->
```

## ♿ Accessibility Best Practices

### Essential Attributes
```html
<!-- Always provide alt text for images -->
<img src="logo.png" alt="Company logo">

<!-- Use labels for form inputs -->
<label for="username">Username</label>
<input type="text" id="username" name="username">

<!-- Provide context with aria-label when needed -->
<button aria-label="Close dialog">×</button>

<!-- Use headings hierarchically -->
<h1>Main Title</h1>
<h2>Section Title</h2>
<h3>Subsection Title</h3>

<!-- Group related form elements -->
<fieldset>
    <legend>Personal Information</legend>
    <!-- form fields -->
</fieldset>
```

### ARIA Attributes
```html
<!-- Describe the purpose of elements -->
<nav aria-label="Main navigation">
<button aria-describedby="help-text">Submit</button>
<div id="help-text">This will send your form data</div>

<!-- Indicate current state -->
<button aria-pressed="true">Toggle Button</button>
<div aria-expanded="false">Collapsible content</div>

<!-- Hide decorative elements from screen readers -->
<span aria-hidden="true">👍</span>

<!-- Live regions for dynamic content -->
<div aria-live="polite" id="status"></div>
```

## 📊 Tables

### Data Tables
```html
<table>
    <caption>Monthly Sales Data</caption>
    <thead>
        <tr>
            <th scope="col">Month</th>
            <th scope="col">Sales</th>
            <th scope="col">Growth</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <th scope="row">January</th>
            <td>$10,000</td>
            <td>+5%</td>
        </tr>
        <tr>
            <th scope="row">February</th>
            <td>$12,000</td>
            <td>+20%</td>
        </tr>
    </tbody>
</table>
```

## 🔧 Meta Tags and SEO

### Essential Meta Tags
```html
<head>
    <!-- Character encoding -->
    <meta charset="UTF-8">
    
    <!-- Viewport for responsive design -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    
    <!-- SEO meta tags -->
    <meta name="description" content="Page description (150-160 characters)">
    <meta name="keywords" content="keyword1, keyword2, keyword3">
    <meta name="author" content="Author Name">
    
    <!-- Open Graph for social media -->
    <meta property="og:title" content="Page Title">
    <meta property="og:description" content="Page description">
    <meta property="og:image" content="https://example.com/image.jpg">
    <meta property="og:url" content="https://example.com/page">
    
    <!-- Twitter Card -->
    <meta name="twitter:card" content="summary_large_image">
    <meta name="twitter:title" content="Page Title">
    <meta name="twitter:description" content="Page description">
    <meta name="twitter:image" content="https://example.com/image.jpg">
</head>
```

## ✅ Quick Checklist

- [ ] Use semantic HTML5 elements
- [ ] Include proper meta tags
- [ ] Add alt text to all images
- [ ] Label all form inputs
- [ ] Use headings hierarchically (h1 → h2 → h3)
- [ ] Validate HTML with W3C validator
- [ ] Test with screen reader
- [ ] Ensure keyboard navigation works

## 🔗 Related Topics
- [[CSS]] - Styling your HTML
- [[JavaScript]] - Adding interactivity
- [[Accessibility]] - Making content accessible to all users

---
*Remember: HTML is about structure and meaning, not appearance. Use [[CSS]] for styling!* 