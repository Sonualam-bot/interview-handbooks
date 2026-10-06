# Handbook 5 — React Performance Engineering

# Chapter 74 — Suspense

## Core Idea

`Suspense` is a React mechanism that lets a component declare:

> "I'm not ready to render yet. Show this fallback UI for now."

```jsx
<Suspense fallback={<p>Loading...</p>}>
  <Profile />
</Suspense>
```

Conceptually:

```text
Component ready
    ↓
Render component

Component not ready
    ↓
Suspend
    ↓
Show fallback
    ↓
Resource becomes ready
    ↓
React retries rendering
```

## 1. Suspense Is Not a Loading Spinner

Suspense is the **mechanism** for handling suspended rendering.

The fallback can be:

```jsx
<Suspense fallback={<Spinner />}>
```

or:

```jsx
<Suspense fallback={<Skeleton />}>
```

or:

```jsx
<Suspense fallback={<p>Loading profile...</p>}>
```

The important concept is the **boundary**, not the particular loading UI.

## 2. Suspense Boundary

A Suspense boundary is created with:

```jsx
<Suspense fallback={<PageLoader />}>
  <Dashboard />
</Suspense>
```

If something inside the boundary suspends, React can show the fallback.

```text
Suspense Boundary
        ↓
     Dashboard
        ↓
  Something suspends
        ↓
   Show fallback
```

## 3. Suspense + React.lazy

A major performance use case is lazy-loaded components:

```jsx
const Settings = React.lazy(
  () => import("./Settings")
);

<Suspense fallback={<p>Loading settings...</p>}>
  <Settings />
</Suspense>
```

Flow:

```text
Render Settings
      ↓
Settings code isn't loaded
      ↓
Settings suspends
      ↓
Suspense boundary found
      ↓
Fallback displayed
      ↓
Chunk loads
      ↓
React retries rendering
      ↓
Settings renders
```

## 4. What Does "Suspend" Mean?

Think:

> "I cannot complete this render yet."

For example:

```text
Settings
   ↓
Needs Settings.js
   ↓
Module isn't available
   ↓
Cannot finish rendering
   ↓
Suspend
```

React finds the appropriate Suspense boundary and can display its fallback.

## 5. Suspense Is Declarative

Manual loading often looks like:

```jsx
const [loading, setLoading] = useState(true);

if (loading) {
  return <Spinner />;
}
```

Suspense expresses the loading boundary declaratively:

```jsx
<Suspense fallback={<Spinner />}>
  <SomeSuspendingComponent />
</Suspense>
```

The component that suspends does not need to manually coordinate the fallback with its parent.

## 6. Boundary Placement Matters

Large boundary:

```jsx
<Suspense fallback={<PageLoader />}>
  <Dashboard />
</Suspense>
```

Smaller boundary:

```jsx
<DashboardHeader />

<Suspense fallback={<ChartSkeleton />}>
  <AnalyticsChart />
</Suspense>

<DataTable />
```

A smaller boundary can allow the rest of the page to remain visible while only the unavailable section shows fallback UI.

## 7. Nested Suspense Boundaries

Boundaries can be nested:

```jsx
<Suspense fallback={<PageLoader />}>
  <Dashboard>
    <Suspense fallback={<ChartSkeleton />}>
      <AnalyticsChart />
    </Suspense>
  </Dashboard>
</Suspense>
```

The inner boundary can provide a more specific fallback for the chart.

This enables progressive loading experiences.

## 8. Suspense + Code Splitting + Lazy Loading

These concepts work together:

```text
Code splitting
      ↓
Separate chunks
      ↓
Lazy loading
      ↓
Load chunk when needed
      ↓
Suspense
      ↓
Show fallback while waiting
```

Remember:

- **Code splitting** creates separate loading units.
- **Lazy loading** delays when they are loaded.
- **Suspense** handles the UI while a suspended subtree is not ready.

## 9. Suspense and Data

Suspense is not only about lazy-loaded JavaScript.

Suspense-enabled data sources can conceptually work like:

```text
Component needs data
       ↓
Data isn't ready
       ↓
Component suspends
       ↓
Suspense fallback
       ↓
Data becomes available
       ↓
React renders content
```

However, ordinary data fetching does **not** automatically integrate with Suspense.

For example:

```jsx
useEffect(() => {
  fetch("/api/users");
}, []);
```

does not automatically mean Suspense will handle that request.

A data source or framework must participate in React's suspension model.

## 10. Suspense vs Error Boundary

They solve different problems.

### Suspense

> "This UI isn't ready yet."

```text
Not ready
   ↓
Suspense
   ↓
Loading fallback
```

### Error Boundary

> "Something went wrong."

```text
Error
   ↓
Error Boundary
   ↓
Error UI
```

A production application may use both:

```jsx
<ErrorBoundary fallback={<ErrorUI />}>
  <Suspense fallback={<LoadingUI />}>
    <Analytics />
  </Suspense>
</ErrorBoundary>
```

## 11. Suspense Does Not Automatically Make Everything Asynchronous

This:

```jsx
<Suspense fallback={<Spinner />}>
  <NormalComponent />
</Suspense>
```

does not automatically show the fallback.

Something inside the subtree must actually **suspend**.

For example:

```jsx
const Component = React.lazy(
  () => import("./Component")
);
```

creates a common suspension scenario.

## 12. Rendering Mental Model

```text
React starts rendering subtree
          ↓
Can subtree finish?
       /            Yes        No
      ↓          ↓
   Render      Suspend
                 ↓
          Find Suspense
                 ↓
          Show fallback
```

When the resource becomes ready:

```text
Resource ready
     ↓
React retries
     ↓
Subtree renders
```

## 13. Good Boundary Design

Use boundaries around meaningful UI units.

For example:

```text
Page
├── Header
├── User information
├── Analytics
└── Footer
```

A useful design might allow:

```text
Header          → available
User information → available
Analytics       → loading
Footer          → available
```

Avoid one giant boundary when a small suspended section does not need to hide the entire page.

Also avoid dozens of tiny boundaries that make the loading experience fragmented.

## 14. Interview Questions

### What is Suspense?

> Suspense is a declarative mechanism for displaying fallback UI when part of a React tree suspends because it isn't ready to render yet.

### Does Suspense fetch data?

> Suspense itself is not a data-fetching API. It provides the rendering mechanism for components or data sources that participate in React's suspension model.

### Does Suspense replace loading state?

> It can replace manual loading-state coordination for Suspense-enabled operations, but ordinary asynchronous operations do not automatically work with Suspense.

### Is Suspense the same as an Error Boundary?

> No. Suspense handles content that isn't ready yet; Error Boundaries handle errors.

## Golden Rules

1. **Suspense provides fallback UI when part of the tree suspends.**
2. **`React.lazy()` is a common source of suspension.**
3. **Suspense is a mechanism, not a loading spinner.**
4. **The fallback can be any React UI.**
5. **Suspense boundaries determine where fallback UI appears.**
6. **Nested boundaries allow more granular loading experiences.**
7. **Suspense is different from Error Boundaries.**
8. **Wrapping something in Suspense does not automatically make it suspend.**
9. **Ordinary `useEffect` fetching does not automatically integrate with Suspense.**
10. **Use meaningful boundaries rather than one giant boundary or dozens of tiny ones.**

## Quick Interview Summary

> **Suspense is React's declarative mechanism for handling a subtree that isn't ready to render yet. When something suspends, the nearest appropriate Suspense boundary can display fallback UI, and React can retry rendering once the resource becomes available. `React.lazy()` is a common use case. Suspense is not an error handler, and it does not automatically make arbitrary asynchronous operations suspend.**
