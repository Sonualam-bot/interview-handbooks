# Handbook 5 — React Performance Engineering

# Chapter 77 — Bundle Optimization

## 1. What Is a JavaScript Bundle?

When you build a frontend application, your source code is transformed and packaged into files that the browser can download.

For example:

```text
src/
├── App.jsx
├── Dashboard.jsx
├── Settings.jsx
├── utils.js
└── charts.js
```

may become:

```text
dist/
├── main.js
├── vendor.js
└── styles.css
```

The browser downloads JavaScript bundles and then needs to:

```text
Download
   ↓
Parse
   ↓
Compile
   ↓
Execute
```

Therefore, JavaScript performance is not only about download size.

---

## 2. What Is Bundle Optimization?

Bundle optimization means reducing the amount of JavaScript that needs to be downloaded, parsed, compiled, and executed.

The goal is:

```text
Less unnecessary JS
        ↓
Less network work
        ↓
Less parsing
        ↓
Less execution
        ↓
Faster application startup
```

---

## 3. Why Bundle Size Matters

Suppose your application sends:

```text
5 MB JavaScript
```

to the browser.

Even if the user's network is fast, the browser still has to process that JavaScript.

Conceptually:

```text
5 MB JS
  ↓
Download
  ↓
Parse
  ↓
Compile
  ↓
Execute
  ↓
Application becomes interactive
```

Large JavaScript can therefore hurt:

- initial page load
- startup time
- responsiveness
- low-end devices
- mobile networks

---

## 4. Bundle Size vs Runtime Cost

An important interview distinction:

> **Bundle size and runtime execution cost are related, but they are not the same thing.**

Example:

```text
Large library
      ↓
Large download
      ↓
Large parsing/compilation cost
```

But even a small bundle can contain expensive code that performs heavy computation.

So performance analysis should consider:

```text
Network cost
+
Parse/compile cost
+
Execution cost
```

---

## 5. Where Does Bundle Size Come From?

Common sources include:

- application code
- third-party dependencies
- polyfills
- duplicate dependencies
- large utility libraries
- icon libraries
- charting libraries
- date libraries
- unused code
- accidentally imported modules

Example:

```text
Application
   ↓
React
   ↓
Chart library
   ↓
Date library
   ↓
UI library
   ↓
Icon library
   ↓
Other dependencies
```

Your application bundle is often significantly influenced by third-party packages.

---

## 6. Inspect Before Optimizing

Don't start removing dependencies randomly.

First understand what's inside the bundle.

Typical tools include:

- bundle analyzers
- build reports
- browser DevTools
- framework-specific analysis tools

Conceptually:

```text
Build application
      ↓
Analyze bundle
      ↓
Find largest modules
      ↓
Identify unnecessary code
      ↓
Optimize
      ↓
Measure again
```

This is much better than guessing.

---

## 7. Code Splitting

Code splitting divides JavaScript into multiple chunks.

Instead of:

```text
main.js
  ↓
Everything
```

you might have:

```text
main.js
dashboard.js
settings.js
analytics.js
```

Then the application can load some code only when needed.

For example:

```jsx
const Analytics = lazy(() => import("./Analytics"));
```

This prevents the Analytics code from necessarily being part of the initial JavaScript execution path.

Remember:

> **Bundle optimization and code splitting are related, but code splitting specifically changes when pieces of JavaScript are loaded.**

---

## 8. Route-Based Splitting

Routes are natural boundaries for code splitting.

Imagine:

```text
/
 /dashboard
 /settings
 /analytics
```

Instead of loading all route-specific code immediately:

```text
Initial:
dashboard + settings + analytics
```

you can structure it conceptually as:

```text
Initial
  ↓
Dashboard code

Navigate to Settings
  ↓
Load Settings chunk

Navigate to Analytics
  ↓
Load Analytics chunk
```

This can significantly reduce initial JavaScript.

---

## 9. Component-Level Splitting

Code splitting doesn't have to happen only at routes.

A heavy component can also become a separate chunk.

Example:

```jsx
const Chart = lazy(() => import("./Chart"));
```

This can be useful for components that:

- are expensive
- are rarely used
- depend on large libraries
- appear only after a user action

Example:

```text
Dashboard
   ↓
User clicks "View Analytics"
   ↓
Load chart code
   ↓
Render chart
```

---

## 10. Large Dependencies

Third-party dependencies are often major contributors to bundle size.

The important idea is:

> **Always investigate large dependencies instead of assuming application code is the main problem.**

---

## 11. Import Only What You Need

A broad import can potentially pull in more code than necessary depending on the library and build tooling.

Prefer APIs that allow the bundler to eliminate unused code.

The principle:

```text
Need one function
      ↓
Avoid unnecessarily including unrelated functionality
```

The exact import style depends on the package.

---

## 12. Tree Shaking

Tree shaking removes code that is not used by the application.

Example:

```js
export function add() {}
export function subtract() {}
export function multiply() {}
```

If the application only imports:

```js
import { add } from "./math";
```

a capable production build can remove unused exports such as:

```text
subtract
multiply
```

This reduces the final bundle.

Tree shaking is especially effective with statically analyzable code such as ES modules.

---

## 13. Bundle Optimization vs Tree Shaking

These are not identical.

### Bundle optimization

A broad category:

```text
Reduce unnecessary JavaScript
```

### Tree shaking

One specific technique:

```text
Remove unused statically analyzable code
```

Other bundle optimization techniques include:

- code splitting
- dependency reduction
- minification
- compression
- lazy loading
- avoiding duplicate packages

---

## 14. Minification

Production JavaScript is usually minified.

Before:

```js
function calculateTotal(price, quantity) {
  return price * quantity;
}
```

After minification, unnecessary whitespace and some syntax overhead can be removed.

Conceptually:

```text
Readable source
      ↓
Minification
      ↓
Smaller production bundle
```

Minification reduces transferred bytes but does not replace architectural optimization.

---

## 15. Compression

After bundling/minification, servers can compress resources during transfer.

Common compression technologies include:

- gzip
- Brotli

Conceptually:

```text
Large production bundle
      ↓
Compression
      ↓
Smaller network payload
```

Important distinction:

```text
Minification → transforms code
Compression  → compresses transferred bytes
```

---

## 16. Duplicate Dependencies

Sometimes different parts of an application can result in multiple versions or copies of a dependency being included.

Conceptually:

```text
App
 ├── library v1
 └── library v2
```

This can increase bundle size.

Dependency analysis can reveal these situations.

Possible solutions depend on the package manager and dependency tree.

---

## 17. Heavy Libraries

Some libraries are simply large.

Examples of potentially heavy categories:

- charting
- rich text editors
- maps
- syntax highlighting
- PDF rendering
- animation
- data visualization

Instead of loading them during startup:

```text
Application starts
      ↓
Load chart library
      ↓
User may never use chart
```

you can often use:

```text
Application starts
      ↓
User requests chart
      ↓
Load chart library
      ↓
Render chart
```

This combines:

```text
Code splitting
+
Lazy loading
```

---

## 18. Don't Optimize Only by File Size

A 1 MB chunk isn't automatically a problem.

Ask:

- Is it loaded on the critical path?
- Is it needed immediately?
- How frequently is it used?
- How expensive is it to parse?
- How expensive is it to execute?
- Does it block interactivity?

The important question is:

> **How much unnecessary work reaches the user before the application becomes useful?**

---

## 19. Initial Bundle vs Total Bundle

Suppose:

```text
Total application JS = 5 MB
```

After code splitting:

```text
Initial JS = 1 MB
Lazy chunks = 4 MB
```

The total application code hasn't necessarily decreased.

But the initial experience may improve because the browser doesn't need to process all 5 MB immediately.

Important distinction:

```text
Total JS
≠
Initial JS
```

---

## 20. Bundle Optimization and Code Splitting

Think of them together:

```text
Bundle optimization
        ↓
Reduce unnecessary JS
        +
Code splitting
        ↓
Don't load all remaining JS immediately
```

For example:

```text
Before:

main.js
  ├── Dashboard
  ├── Settings
  ├── Analytics
  └── PDF editor

After:

main.js
dashboard.js
settings.js
analytics.js
pdf-editor.js
```

The initial page may only need:

```text
main.js
+
dashboard.js
```

---

## 21. Performance Is About the Critical Path

Imagine:

```text
User opens page
      ↓
Download JS
      ↓
Parse JS
      ↓
Execute JS
      ↓
Render application
      ↓
User interacts
```

Anything unnecessarily placed before the user can interact with the application increases startup work.

Therefore bundle optimization should focus heavily on the **critical path**.

---

## 22. Example: E-commerce Application

Suppose an ecommerce app has:

```text
Home
Products
Cart
Checkout
Admin Dashboard
Analytics
PDF Invoice Generator
```

A poor architecture might load everything immediately:

```text
Initial bundle
=
Home
+
Products
+
Cart
+
Checkout
+
Admin
+
Analytics
+
PDF
```

A better strategy could be:

```text
Initial
  ↓
Home + common application code

Products
  ↓
Load when route is visited

Admin
  ↓
Load only for admin routes

Analytics
  ↓
Load only when analytics is opened

PDF generator
  ↓
Load when invoice generation is requested
```

This reduces unnecessary startup work.

---

## 23. Common Bundle Optimization Checklist

When a bundle is too large:

```text
1. Analyze the bundle
2. Find the largest dependencies
3. Remove unused dependencies
4. Check duplicate dependencies
5. Ensure tree shaking works
6. Split large routes
7. Lazy-load heavy components
8. Minify production builds
9. Enable Brotli/gzip
10. Measure again
```

---

## 24. Common Mistakes

### Mistake 1 — Removing random packages

Without measuring, you may optimize something irrelevant.

### Mistake 2 — Lazy-loading everything

Too many chunks can create unnecessary loading complexity and network overhead.

### Mistake 3 — Ignoring third-party dependencies

A dependency may be much larger than your own code.

### Mistake 4 — Confusing total bundle size with initial bundle size

Code splitting may reduce initial JavaScript without reducing total application code.

### Mistake 5 — Assuming minification solves everything

Minification reduces bytes but doesn't fix poor architecture or unnecessary dependencies.

### Mistake 6 — Ignoring runtime cost

Smaller JavaScript can still perform expensive computations.

---

## 25. Interview Scenario

### Question

> Your React application has a large initial JavaScript bundle and users complain that the first page feels slow. What would you do?

Strong answer:

```text
1. Analyze the production bundle
2. Identify the largest modules/dependencies
3. Remove unused or unnecessary dependencies
4. Check for duplicate dependencies
5. Ensure tree shaking/minification are working
6. Split routes and heavy components
7. Lazy-load rarely used features
8. Compress assets with Brotli/gzip
9. Measure initial JS and startup performance again
```

The important part is:

> **Measure first, identify the actual bottleneck, then optimize.**

---

## 26. Bundle Optimization Mental Model

```text
Application
      ↓
Analyze bundle
      ↓
Find unnecessary / expensive JS
      ↓
Remove
      ↓
Tree shake
      ↓
Split
      ↓
Lazy-load
      ↓
Minify
      ↓
Compress
      ↓
Measure again
```

---

## 27. Golden Rules

1. Bundle size affects network and browser processing cost.
2. JavaScript must be downloaded, parsed, compiled, and executed.
3. Analyze bundles before optimizing them.
4. Third-party dependencies can dominate bundle size.
5. Code splitting reduces how much JS must be loaded initially.
6. Route boundaries are natural code-splitting boundaries.
7. Heavy rarely-used components are good lazy-loading candidates.
8. Tree shaking removes unused statically analyzable code.
9. Minification reduces code size.
10. Compression reduces transferred bytes.
11. Duplicate dependencies can unnecessarily increase bundle size.
12. Total JavaScript and initial JavaScript are different.
13. Smaller bundle size does not automatically mean lower runtime cost.
14. Don't create excessive chunks without a reason.
15. Measure again after optimization.

---

## 28. Quick Interview Summary

```text
Bundle Optimization
        ↓
Analyze
        ↓
Remove unnecessary code
        ↓
Tree shake
        ↓
Code split
        ↓
Lazy-load heavy features
        ↓
Minify
        ↓
Compress
        ↓
Smaller critical JS
        ↓
Faster startup
```

### One-line interview answer

> **Bundle optimization is the process of reducing and restructuring the JavaScript delivered to users so the browser downloads, parses, compiles, and executes less unnecessary code, especially on the critical startup path.**
