# Chapter 56 — ES Modules

> **Collections Handbook**

---

# What You'll Learn

- Why ES Modules were introduced
- Exporting values
- Named Exports
- Default Exports
- Importing Modules
- Re-exporting
- Dynamic Imports
- Tree Shaking
- CommonJS vs ES Modules
- Interview Questions

---

# Introduction

As JavaScript applications grew larger, keeping all code in a single file became difficult.

Modules solve this problem by allowing code to be split into reusable files with well-defined exports and imports.

ES6 introduced the **ECMAScript Module (ESM)** system, which is now the standard module format for modern JavaScript.

---

# What is a Module?

A module is simply a JavaScript file with its own scope.

Variables declared inside one module are **not** automatically available in another module.

```js
// math.js
export const PI = 3.14159;
```

```js
// app.js
import { PI } from "./math.js";

console.log(PI);
```

---

# Named Exports

A module can export multiple named values.

```js
// utils.js
export function add(a, b) {
  return a + b;
}

export function subtract(a, b) {
  return a - b;
}
```

Import them by name:

```js
import { add, subtract } from "./utils.js";
```

You can also rename imports:

```js
import { add as sum } from "./utils.js";
```

---

# Default Export

A module can have **one** default export.

```js
// logger.js
export default function log(message) {
  console.log(message);
}
```

Import it without braces:

```js
import log from "./logger.js";
```

The imported name can be chosen freely.

---

# Mixing Named and Default Exports

```js
// api.js
export const BASE_URL = "https://api.example.com";

export default function request() {}
```

```js
import request, { BASE_URL } from "./api.js";
```

---

# Re-exporting

Modules can forward exports from other modules.

```js
export * from "./math.js";
```

Or:

```js
export { add } from "./math.js";
```

This is commonly used to create an `index.js` barrel file.

---

# Dynamic Imports

Modules can be loaded on demand.

```js
const math = await import("./math.js");

console.log(math.add(2, 3));
```

Dynamic imports return a Promise.

Use cases:

- Lazy loading
- Code splitting
- Feature-based loading

---

# Tree Shaking

Tree shaking removes unused exports during the build process.

```js
// math.js
export function add() {}
export function subtract() {}
export function multiply() {}
```

```js
import { add } from "./math.js";
```

A bundler can remove the unused exports from the final bundle.

---

# CommonJS vs ES Modules

| CommonJS | ES Modules |
|----------|------------|
| `require()` | `import` |
| `module.exports` | `export` |
| Loaded synchronously | Supports static analysis |
| Primarily Node.js (legacy) | Modern JavaScript standard |

Example:

```js
// CommonJS
const fs = require("fs");
```

```js
// ES Module
import fs from "fs";
```

---

# Common Mistakes

### Forgetting Curly Braces

Named export:

```js
export const name = "Sonu";
```

Correct import:

```js
import { name } from "./file.js";
```

---

### Multiple Default Exports

A module can only have **one** default export.

---

# Interview Questions

### What is the difference between a named export and a default export?

- Named exports require matching names and use `{}`.
- A default export does not use `{}` and there can only be one per module.

---

### What is tree shaking?

A build optimization that removes unused exports from the final bundle.

---

### Why use dynamic imports?

To load code only when it is needed, improving application performance.

---

# Key Takeaways

- ES Modules are the modern JavaScript module system.
- Use named exports for multiple exported values.
- Use a default export for the primary exported value.
- Dynamic imports enable lazy loading and code splitting.
- Tree shaking reduces bundle size by eliminating unused code.
- Understanding ESM is essential for modern React, Node.js, and frontend development.

---

# Handbook Complete ✅

You have completed the **Collections Handbook**.

Next Handbook:

**Performance Patterns Handbook**
