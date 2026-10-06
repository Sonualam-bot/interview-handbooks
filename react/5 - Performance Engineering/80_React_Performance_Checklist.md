# Handbook 5 — React Performance Engineering
# Chapter 80 — React Performance Checklist

## 1. The Goal

React performance optimization is not:

> “Make React render as little as possible.”

The goal is:

> **Reduce unnecessary work while keeping the application correct and responsive.**

---

## 2. Start With Measurement

Before optimizing:

```text
User reports slowness
        ↓
Reproduce the problem
        ↓
Profile / measure
        ↓
Find bottleneck
        ↓
Understand cause
        ↓
Optimize
        ↓
Measure again
```

**Never optimize blindly.**

---

## 3. Rendering Checklist

Ask:

- Is this component rendering unnecessarily?
- Is state placed too high?
- Are large subtrees affected by small updates?
- Are props changing unnecessarily?
- Is context causing broad updates?
- Are external store subscriptions too broad?

### Prefer state colocation

Keep state as close as practical to the components that need it.

```text
Bad:

App
 └── state
      └── entire application subtree affected

Better:

SearchBox
 └── search state
```

---

## 4. Props Checklist

Look for unstable references.

### Objects

```jsx
<Child config={{ theme: "dark" }} />
```

### Arrays

```jsx
<Child items={[1, 2, 3]} />
```

### Functions

```jsx
<Child onClick={() => doSomething()} />
```

Every render creates a new reference.

If this prevents a meaningful `React.memo` bailout, investigate whether the reference should be stabilized.

---

## 5. React.memo Checklist

Consider:

```jsx
const Child = React.memo(ChildComponent);
```

when:

```text
Parent renders frequently
        ↓
Child receives same props
        ↓
Child render is expensive
        ↓
Memoization can avoid that work
```

Don't use `React.memo` everywhere.

> `React.memo` is an optimization, not a requirement for correctness.

---

## 6. useMemo Checklist

Use `useMemo` when:

```text
Expensive calculation
        +
Repeated renders
        +
Dependencies often unchanged
        ↓
Potentially worthwhile memoization
```

Example:

```jsx
const filteredUsers = useMemo(
  () => expensiveFilter(users),
  [users]
);
```

Don't use `useMemo` simply because a value exists.

---

## 7. useCallback Checklist

`useCallback` is primarily about **function identity**.

Consider it when:

```text
Parent renders
      ↓
Function prop gets new reference
      ↓
Memoized child cannot bail out
```

Example:

```jsx
const handleSelect = useCallback((item) => {
  selectItem(item);
}, [selectItem]);
```

Remember:

> `useCallback` does not make the function execute faster.

---

## 8. Code Splitting Checklist

For large applications:

- Split large routes
- Split rarely used features
- Split heavy components
- Lazy-load expensive features

Example:

```jsx
const Analytics = React.lazy(
  () => import("./Analytics")
);
```

Use a `Suspense` boundary:

```jsx
<Suspense fallback={<Loader />}>
  <Analytics />
</Suspense>
```

---

## 9. Bundle Checklist

Inspect the production bundle.

Look for:

- Large dependencies
- Duplicate dependencies
- Unused code
- Heavy libraries
- Accidentally imported modules
- Large initial JavaScript

Remember:

```text
Download
   ↓
Parse
   ↓
Compile
   ↓
Execute
```

Reducing JavaScript can improve startup performance.

---

## 10. Tree Shaking Checklist

Use statically analyzable ES modules:

```js
export function add() {}
export function subtract() {}
```

Import only what is needed:

```js
import { add } from "./math";
```

Modern production bundlers can remove unused exports when conditions allow.

Check package side effects carefully before using:

```json
{
  "sideEffects": false
}
```

Don't mark modules as side-effect-free if they actually perform required side effects.

---

## 11. Lazy Loading Checklist

Ask:

> Does this resource need to load immediately?

Good candidates:

- Below-the-fold images
- Rarely used routes
- Heavy charts
- PDF viewers
- Rich text editors
- Large maps
- Rarely opened dialogs

Don't lazy-load critical content unnecessarily.

---

## 12. Suspense Checklist

Use appropriate boundaries:

```jsx
<Suspense fallback={<PageLoader />}>
  <Dashboard />
</Suspense>
```

Consider:

- What part of the UI is waiting?
- Is the fallback useful?
- Is the boundary too broad?
- Would a smaller boundary provide better UX?

Remember:

```text
Code splitting → creates separate chunks

Lazy loading → delays loading

Suspense → provides fallback while something suspends
```

---

## 13. Large Lists Checklist

For very large lists:

```text
100,000 items
       ↓
Don't mount all 100,000 DOM nodes
```

Consider **virtualization**.

```text
Viewport
 ├── Item 101
 ├── Item 102
 ├── Item 103
 └── ...

Only visible items + overscan
```

Useful for:

- Large tables
- Logs
- Search results
- File explorers
- Chat histories
- Large feeds

Remember:

> Virtualization optimizes rendering; pagination/infinite scrolling controls data loading.

---

## 14. Image Checklist

Ask:

### What size?

Don't send a 4000px image when a 400px image is required.

### What format?

Consider modern formats such as:

- WebP
- AVIF

### When should it load?

Use lazy loading for appropriate non-critical images.

```html
<img
  src="image.jpg"
  loading="lazy"
/>
```

### Can layout shift?

Reserve space:

```html
<img
  width="800"
  height="600"
  src="image.jpg"
/>
```

### Critical image?

Don't blindly lazy-load above-the-fold/critical images.

---

## 15. Network Checklist

Performance isn't only React rendering.

Check:

- Number of requests
- Request size
- API latency
- Waterfalls
- Caching
- Compression
- CDN usage
- Unnecessary requests

Think about the complete path:

```text
User
 ↓
Network
 ↓
Server
 ↓
API
 ↓
JavaScript
 ↓
React
 ↓
DOM
 ↓
Paint
```

---

## 16. Context Checklist

Context updates can affect many consumers.

Ask:

> Does changing this context value cause a large number of components to update?

Avoid putting unrelated frequently changing state into a broad context when unnecessary.

Possible approaches:

- Split contexts
- Colocate state
- Reduce consumer count
- Use a more appropriate state/store architecture

---

## 17. Performance Anti-Patterns

Avoid:

```text
❌ React.memo everywhere
❌ useMemo everywhere
❌ useCallback everywhere
❌ Lazy-loading everything
❌ Virtualizing tiny lists
❌ Optimizing without profiling
❌ Assuming fewer renders always means faster
❌ Ignoring network/image costs
```

Performance engineering is about **measured bottlenecks**, not optimization cargo cults.

---

## 18. Practical React Performance Checklist

### Rendering

- [ ] Is unnecessary rendering happening?
- [ ] Is state colocated appropriately?
- [ ] Are expensive subtrees isolated?

### Props

- [ ] Are object props unnecessarily recreated?
- [ ] Are array props unnecessarily recreated?
- [ ] Are function props unnecessarily recreated?

### Memoization

- [ ] Is `React.memo` actually useful here?
- [ ] Is `useMemo` preventing meaningful expensive work?
- [ ] Is `useCallback` needed for function identity?

### JavaScript

- [ ] Is the initial bundle unnecessarily large?
- [ ] Are heavy dependencies required?
- [ ] Is tree shaking working?
- [ ] Are routes/features split appropriately?

### Loading

- [ ] Can large features be lazy-loaded?
- [ ] Are Suspense boundaries appropriate?
- [ ] Are images loaded at the right time?

### Lists

- [ ] Is the list large enough to need virtualization?
- [ ] Is overscan reasonable?

### Images

- [ ] Correct dimensions?
- [ ] Appropriate format?
- [ ] Compressed?
- [ ] Lazy-loaded when appropriate?
- [ ] Layout space reserved?

### Measurement

- [ ] Did I profile before optimizing?
- [ ] Did I measure after optimizing?
- [ ] Did the optimization actually improve the user experience?

---

## 19. The Complete Mental Model

```text
                 React Performance
                       │
        ┌──────────────┼──────────────┐
        ↓              ↓              ↓
    Rendering       JavaScript      Network
        │              │              │
    State          Bundle size       APIs
    Props          Dependencies      Requests
    Context        Code splitting    Caching
    Memoization    Tree shaking      CDN
        │
        ↓
      DOM
        │
        ↓
     Images
        │
        ↓
      Paint
```

React performance is **not only about React**.

---

## 20. Interview Answer

### “How do you optimize a slow React application?”

> I would first reproduce and profile the problem rather than immediately adding memoization. I would identify whether the bottleneck is rendering, expensive calculations, JavaScript bundle size, network requests, large lists, or images. For rendering issues I would examine state placement, prop stability, context updates, and expensive subtrees, then consider techniques such as state colocation, `React.memo`, `useMemo`, or `useCallback`. For loading performance I would use code splitting, lazy loading, bundle optimization, tree shaking, image optimization, and virtualization where appropriate. Finally, I would measure again to verify that the change actually improved performance.

---

# Final Mental Model

```text
Measure
   ↓
Find the bottleneck
   ↓
Understand why it happens
   ↓
Choose the smallest useful optimization
   ↓
Measure again
```

> **Don't optimize React because you can. Optimize because you measured a meaningful problem.**

---

## Handbook 5 Complete

**Chapters 66–80 — React Performance Engineering — COMPLETE.**
