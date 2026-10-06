# Handbook 5 — React Performance Engineering

# Chapter 72 — Code Splitting

## Core Idea

**Code splitting** means breaking an application's JavaScript into multiple independently loadable chunks so users do not have to download all application code during the initial load.

Instead of:

```text
main.js
└── entire application
```

we can have:

```text
main.js
dashboard.js
settings.js
analytics.js
admin.js
editor.js
```

---

## 1. Why Code Splitting?

A large React application may contain:

```text
App
├── Dashboard
├── Settings
├── AdminPanel
├── Analytics
├── Reports
└── RichTextEditor
```

Users may initially need only Dashboard.

Without splitting:

```text
Browser loads page
       ↓
Downloads large JavaScript bundle
       ↓
Parses / compiles / executes JavaScript
       ↓
Application becomes usable
```

With splitting:

```text
Browser loads page
       ↓
Downloads required initial code
       ↓
Loads other feature code later
       ↓
Less initial JavaScript work
```

---

## 2. Bundling vs Code Splitting

### Bundling

Combines modules into files that can be served to the browser.

```text
100 modules
    ↓
bundler
    ↓
bundle(s)
```

### Code splitting

Creates boundaries where code can be separated into independently loadable chunks.

```text
Large application
       ↓
bundler
       ↓
multiple chunks
```

**Key distinction:**

> Bundling packages code together; code splitting determines useful boundaries where code can be separated.

---

## 3. Static Imports

```jsx
import Dashboard from "./Dashboard";
import Settings from "./Settings";
import Analytics from "./Analytics";
```

These are static imports.

The bundler can see these dependencies during the build. Depending on the build configuration, they may be included in the initial bundle or other build-generated chunks.

If a feature is always needed immediately, loading it upfront may be reasonable.

---

## 4. Dynamic Imports

JavaScript supports dynamic imports:

```js
import("./Analytics");
```

A dynamic import returns a Promise:

```text
import("./Analytics")
        ↓
      Promise
        ↓
load module asynchronously
        ↓
module available
```

Example:

```jsx
async function openAnalytics() {
  const module = await import("./Analytics");

  module.open();
}
```

Dynamic imports create natural split points that bundlers can use to generate separate chunks.

---

## 5. React.lazy

React provides `React.lazy()` for lazy-loading components:

```jsx
const Settings = React.lazy(() => import("./Settings"));
```

Instead of:

```jsx
import Settings from "./Settings";
```

the component can be loaded asynchronously when needed.

---

## 6. React.lazy + Suspense

A lazy component may not be available immediately, so React needs a fallback:

```jsx
const Settings = React.lazy(() => import("./Settings"));

function App() {
  return (
    <Suspense fallback={<p>Loading...</p>}>
      <Settings />
    </Suspense>
  );
}
```

Conceptual flow:

```text
Render Settings
      ↓
Settings module not loaded
      ↓
React suspends rendering
      ↓
Suspense displays fallback
      ↓
chunk loads
      ↓
Settings becomes available
      ↓
React renders Settings
```

---

## 7. Code Splitting vs Lazy Loading

### Code splitting

Divides application code into separate chunks.

### Lazy loading

Loads a chunk/module later instead of immediately.

Therefore:

```text
Code splitting
     ↓
creates separate loading units
     ↓
Lazy loading
     ↓
loads a unit when needed
```

Code splitting makes lazy loading possible.

---

## 8. Route-Based Code Splitting

One of the most common strategies is splitting by route.

For example:

```text
/
 /dashboard
 /settings
 /analytics
 /admin
```

Instead of loading every page component immediately:

```text
Initial load
    ↓
Dashboard code

Later:
Settings → settings chunk
Analytics → analytics chunk
Admin → admin chunk
```

This works well because users typically visit only a subset of routes during a session.

---

## 9. Component-Level Code Splitting

You can also split large components.

```text
Dashboard
├── Header
├── Stats
├── Charts
├── DataTable
└── AdvancedEditor
```

If `AdvancedEditor` is opened only after a user action:

```jsx
const AdvancedEditor = React.lazy(
  () => import("./AdvancedEditor")
);
```

Its code can be loaded only when needed.

---

## 10. When Code Splitting Helps

Strong candidates include:

- large applications
- large initial JavaScript bundles
- rarely used features
- routes with heavy dependencies
- expensive libraries used only by certain features

Examples:

```text
Main application
      +
Charting library
      +
Rich text editor
      +
PDF viewer
      +
Admin tools
```

These can be good candidates for deferred loading.

---

## 11. Code Splitting Is Not Automatically Better

Do not split everything.

Excessive splitting can produce:

```text
main.js
a.js
b.js
c.js
d.js
e.js
f.js
...
```

This can create additional loading coordination and overhead.

The goal is:

> Create meaningful boundaries around code that does not need to be loaded immediately.

---

## 12. Initial Load Tradeoff

Without splitting:

```text
Large initial bundle
        ↓
More JavaScript downloaded
        ↓
More parsing / compilation / execution
        ↓
Potentially slower startup
```

With excessive splitting:

```text
Tiny chunks
    ↓
Many requests
    ↓
More loading coordination
    ↓
Potential overhead
```

Good performance engineering balances these tradeoffs.

---

## 13. Practical Example

Instead of:

```jsx
import Dashboard from "./Dashboard";
import AdminPanel from "./AdminPanel";
import Analytics from "./Analytics";
import RichTextEditor from "./RichTextEditor";
```

you might use:

```jsx
import Dashboard from "./Dashboard";

const AdminPanel = React.lazy(
  () => import("./AdminPanel")
);

const Analytics = React.lazy(
  () => import("./Analytics")
);

const RichTextEditor = React.lazy(
  () => import("./RichTextEditor")
);
```

Conceptually:

```text
Initial
  ↓
Dashboard

Admin route
  ↓
AdminPanel chunk

Analytics route
  ↓
Analytics chunk

Editor opened
  ↓
RichTextEditor chunk
```

---

## 14. Why It Can Improve Performance

Code splitting can reduce the amount of JavaScript initially:

- downloaded
- parsed
- compiled
- executed

Think:

```text
Download
   ↓
Parse
   ↓
Compile
   ↓
Execute
   ↓
Render
```

Reducing unnecessary initial JavaScript can reduce work across this pipeline.

---

## 15. Code Splitting Does Not Necessarily Reduce Total Code

Suppose the application contains:

```text
500 KB JavaScript
```

Code splitting could produce:

```text
main.js       200 KB
analytics.js  150 KB
editor.js     150 KB
```

The total is still approximately:

```text
500 KB
```

The optimization is **when** the code is loaded:

```text
Before:
500 KB immediately

After:
200 KB immediately
+150 KB later
+150 KB later
```

---

## 16. Code Splitting vs Tree Shaking

### Tree shaking

Removes unused code.

```text
Library
├── A ✓
├── B ✗
├── C ✗
└── D ✓

Tree shaking
     ↓

Bundle
├── A
└── D
```

### Code splitting

Separates code into independently loadable chunks.

```text
Application
     ↓
main.js
admin.js
analytics.js
```

Remember:

```text
Tree shaking → remove unused code

Code splitting → separate code into chunks
```

They can be used together.

---

## 17. Interview Scenario

**Question:**

> Your React application's initial JavaScript bundle is very large. It has an admin dashboard, analytics section, and rich text editor that most users rarely use. What would you do?

Strong answer:

> I would first measure the bundle and identify which dependencies and routes contribute most to the initial JavaScript. Then I would introduce meaningful code-splitting boundaries, commonly at routes and for heavy features such as the rich text editor. React.lazy with Suspense can load components asynchronously. I would also examine tree shaking and dependency optimization, then measure the production application again.

Avoid simply saying:

> "I'll use `React.lazy()` everywhere."

---

## 18. Interview Mental Model

```text
Large React application
          ↓
Identify rarely-needed code
          ↓
Create split boundary
          ↓
Separate chunk
          ↓
Load when needed
          ↓
Smaller initial JS
```

---

# Golden Rules

1. **Code splitting divides application code into independently loadable chunks.**
2. **Dynamic `import()` is a common mechanism for creating split points.**
3. **`React.lazy()` can lazy-load React components.**
4. **Suspense provides a fallback while a lazy component is loading.**
5. **Route-based code splitting is one of the most common strategies.**
6. **Component-level splitting is useful for heavy, rarely-used features.**
7. **Code splitting primarily changes when code is loaded, not necessarily total application code size.**
8. **Tree shaking removes unused code; code splitting separates code into chunks.**
9. **Too much splitting can create unnecessary complexity and loading overhead.**
10. **Measure the production application before and after optimization.**

---

## Quick Interview Summary

> **Code splitting means dividing a JavaScript application into smaller chunks so users do not have to download all application code during the initial load. In React, dynamic imports and `React.lazy()` are common mechanisms for creating lazy-loaded boundaries, with `Suspense` providing the loading fallback. Route-based splitting is common, but splitting should be based on meaningful boundaries rather than creating excessive chunks.**
