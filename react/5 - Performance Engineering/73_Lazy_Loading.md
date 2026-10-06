# Handbook 5 — React Performance Engineering

# Chapter 73 — Lazy Loading

## Core Idea

**Lazy loading** means delaying the loading of code or resources until they are actually needed.

```text
Application starts
      ↓
Load what is needed now
      ↓
User requests feature
      ↓
Load that feature
```

---

## 1. Code Splitting vs Lazy Loading

### Code splitting

Divides code into separate chunks.

```text
Application
     ↓
main.js
admin.js
editor.js
analytics.js
```

### Lazy loading

Controls **when** those chunks are loaded.

```text
Initial load
     ↓
main.js

User opens analytics
     ↓
analytics.js
```

Remember:

```text
Code splitting → creates separate pieces

Lazy loading → delays loading those pieces
```

---

## 2. Why Lazy Loading Helps

Suppose an application has:

```text
Dashboard
AdminPanel
Analytics
PDFViewer
RichTextEditor
```

Most users may initially need only Dashboard.

Lazy loading allows:

```text
Initial:
Dashboard

Later:
AdminPanel → when admin route opens
Analytics → when analytics opens
PDFViewer → when PDF feature is used
Editor → when editor opens
```

This can reduce the initial JavaScript workload.

---

## 3. `React.lazy`

React provides `React.lazy()` for lazy-loading components:

```jsx
const Analytics = React.lazy(
  () => import("./Analytics")
);
```

Compare:

```jsx
import Analytics from "./Analytics";
```

with:

```jsx
const Analytics = React.lazy(
  () => import("./Analytics")
);
```

The lazy version allows the component's module to be loaded asynchronously.

---

## 4. Why `Suspense` Is Needed

A lazy component may not be available immediately.

```jsx
<Suspense fallback={<p>Loading...</p>}>
  <Analytics />
</Suspense>
```

Complete example:

```jsx
const Analytics = React.lazy(
  () => import("./Analytics")
);

function App() {
  return (
    <Suspense fallback={<p>Loading...</p>}>
      <Analytics />
    </Suspense>
  );
}
```

Flow:

```text
Render Analytics
       ↓
Module isn't loaded
       ↓
Component suspends
       ↓
Suspense renders fallback
       ↓
Chunk loads
       ↓
Analytics becomes available
       ↓
React renders Analytics
```

---

## 5. What Does "Suspends" Mean?

When a lazy component isn't ready:

```text
Analytics
    ↓
needs module
    ↓
module still loading
```

React cannot render that component yet.

The component therefore **suspends**, and the nearest appropriate `<Suspense>` boundary can display its fallback.

The fallback is not necessarily an error UI.

It represents:

> This part of the UI isn't ready yet.

---

## 6. Suspense Boundaries

A `<Suspense>` boundary determines which part of the UI should show the fallback.

Large boundary:

```jsx
<Suspense fallback={<PageLoader />}>
  <Dashboard />
</Suspense>
```

Smaller boundary:

```jsx
<Header />

<Suspense fallback={<ChartLoader />}>
  <AnalyticsChart />
</Suspense>
```

The smaller boundary can preserve the rest of the page while only the unavailable section shows loading UI.

```text
Header
  ↓
already available

Chart
  ↓
loading

Fallback
  ↓
only chart area
```

---

## 7. Route-Level Lazy Loading

A common strategy is lazy-loading route components:

```jsx
const Dashboard = React.lazy(
  () => import("./pages/Dashboard")
);

const Settings = React.lazy(
  () => import("./pages/Settings")
);

const Analytics = React.lazy(
  () => import("./pages/Analytics")
);
```

Then:

```jsx
<Suspense fallback={<PageLoader />}>
  <Routes>
    {/* routes */}
  </Routes>
</Suspense>
```

Users do not necessarily download every route's component code during the initial page load.

---

## 8. Component-Level Lazy Loading

Lazy loading is not limited to routes.

Suppose:

```text
Dashboard
├── Header
├── Stats
├── Table
└── AdvancedReportBuilder
```

If the report builder is opened only after a user action:

```jsx
const ReportBuilder = React.lazy(
  () => import("./ReportBuilder")
);
```

Then:

```jsx
<Suspense fallback={<ReportBuilderLoader />}>
  {showBuilder && <ReportBuilder />}
</Suspense>
```

The expensive feature can be deferred until needed.

---

## 9. Lazy Loading Heavy Libraries

Sometimes the component itself isn't huge, but one of its dependencies is.

For example:

```text
PDF feature
     ↓
PDF library
     ↓
large JavaScript dependency
```

Instead of loading it at application startup:

```jsx
async function openPdf() {
  const pdfModule = await import("./pdfFeature");

  pdfModule.open();
}
```

This can keep rarely used dependencies out of the initial load.

---

## 10. Lazy Loading Images

Lazy loading also applies to images:

```html
<img
  src="large-image.jpg"
  loading="lazy"
  alt="..."
/>
```

The browser can defer loading images that aren't immediately needed, such as images far below the viewport.

This is different from React component lazy loading, but the underlying idea is the same:

> Don't load the resource until it is useful.

---

## 11. Lazy Loading and Initial Performance

Without lazy loading:

```text
Initial load
    ↓
Dashboard
Admin
Analytics
Editor
PDF viewer
Charts
    ↓
Large initial workload
```

With lazy loading:

```text
Initial load
    ↓
Dashboard
    ↓
User opens Analytics
    ↓
Load Analytics
    ↓
User opens Editor
    ↓
Load Editor
```

The initial workload can be significantly smaller.

---

## 12. The Tradeoff

Lazy loading does not eliminate the cost.

It moves the cost.

Without lazy loading:

```text
Pay loading cost
      ↓
during initial load
```

With lazy loading:

```text
Pay loading cost
      ↓
when feature is first needed
```

If a user immediately needs a feature, they may experience a loading delay.

Therefore loading UX matters.

---

## 13. Prefetching

Sometimes a feature can be loaded before the user explicitly requests it.

```text
User is on Dashboard
        ↓
Likely to open Analytics
        ↓
Prefetch Analytics
        ↓
User clicks Analytics
        ↓
Analytics is already available
```

This combines:

```text
Lazy loading
+
Prefetching
```

The feature does not need to be part of the critical initial load, but it can be prepared before the user requests it.

---

## 14. Choosing What to Lazy Load

Ask:

1. Is it needed immediately?
2. Is it expensive?
3. Is it rarely used?
4. Is it on a route users may not visit?
5. Would delaying it create noticeable loading?
6. Can it be prefetched?

Example:

```text
Navbar
```

Usually needed immediately.

```text
Rarely-used admin report builder
```

Good lazy-loading candidate.

---

## 15. Lazy Loading and User Experience

Poor implementation:

```text
Click
  ↓
Blank screen
  ↓
Network request
  ↓
Wait
  ↓
Content appears
```

Better implementation:

```text
Click
  ↓
Meaningful loading state
  ↓
Network request
  ↓
Content appears
```

Example:

```jsx
<Suspense fallback={<ChartSkeleton />}>
  <AnalyticsChart />
</Suspense>
```

A skeleton or contextual loader communicates what is being loaded.

---

## 16. Lazy Loading vs Eager Loading

### Eager loading

Load immediately:

```text
Application starts
       ↓
Load feature
```

### Lazy loading

Load when needed:

```text
Application starts
       ↓
Do not load feature

Feature requested
       ↓
Load feature
```

The choice depends on usage patterns and performance measurements.

---

## 17. Common Mistake: Lazy Loading Everything

Avoid:

```text
Lazy load
Navbar
Button
Card
Header
Footer
SmallIcon
TinyComponent
...
```

This can create excessive boundaries and unnecessary loading overhead.

Better candidates often include:

- routes
- heavy features
- large editors
- charting functionality
- PDF viewers
- rarely used admin functionality

---

## 18. Interview Scenario

**Question:**

> You have a React dashboard containing a large analytics chart that only 20% of users open. How would you optimize its loading?

Strong answer:

> I would first measure the chart's contribution to the initial bundle and rendering cost. If it is not needed during initial page load, I could lazy-load the chart component using `React.lazy()` and `Suspense`, so its code is fetched when the feature is needed. I would provide an appropriate loading state and consider prefetching if user behavior makes it predictable that the chart will be opened soon. Then I would measure the production performance again.

---

## 19. Interview Mental Model

```text
Feature exists
      ↓
Do we need it immediately?
      ↓
     No
      ↓
Lazy load it
      ↓
Separate chunk
      ↓
User requests feature
      ↓
Load chunk
      ↓
Suspense fallback
      ↓
Feature renders
```

---

# Golden Rules

1. **Lazy loading means delaying resource loading until it is needed.**
2. **`React.lazy()` is designed for lazy-loading React components.**
3. **Lazy components commonly need a `<Suspense>` boundary.**
4. **Suspense fallback represents temporary unavailability, not necessarily an error.**
5. **Route-level lazy loading is a common strategy.**
6. **Heavy, rarely-used components are good candidates.**
7. **Lazy loading moves loading cost later; it does not remove the cost.**
8. **Loading UX matters because deferred code can introduce a visible wait.**
9. **Prefetching can prepare a lazy feature before the user explicitly requests it.**
10. **Do not lazy-load every tiny component; create meaningful boundaries.**

---

## Quick Interview Summary

> **Lazy loading means delaying the loading of code or resources until they are needed. In React, `React.lazy()` can asynchronously load components, while `Suspense` provides a fallback while the component's code is loading. Route-level and heavy feature-level lazy loading are common strategies. Lazy loading reduces initial work but moves the loading cost to the point where the feature is requested, so loading UX, prefetching, and measurement are important.**
