# Handbook 5 — React Performance Engineering

# Chapter 78 — Tree Shaking

## 1. What Is Tree Shaking?

**Tree shaking** is the process of removing code from the production bundle that is imported/exported but never actually used.

```text
Source code
    ↓
Analyze imports/exports
    ↓
Find unused code
    ↓
Remove it
    ↓
Smaller bundle
```

The name comes from the idea of shaking a tree so dead branches fall off.

---

## 2. Simple Example

```js
export function add(a, b) {
  return a + b;
}

export function subtract(a, b) {
  return a - b;
}

export function multiply(a, b) {
  return a * b;
}
```

If the application only uses:

```js
import { add } from "./math";
```

a production bundler can potentially remove:

```text
subtract()
multiply()
```

---

## 3. Why Tree Shaking Matters

Without tree shaking:

```text
Library
 ├── feature A
 ├── feature B
 ├── feature C
 ├── feature D
 └── feature E

Application needs:
feature A

Bundle:
A + B + C + D + E
```

With effective tree shaking:

```text
Application needs:
feature A

Bundle:
A
```

This can reduce:

- JavaScript download size
- parsing cost
- compilation cost
- execution overhead

---

## 4. ES Modules Help Tree Shaking

Tree shaking works especially well with **ES modules**:

```js
import { add } from "./math.js";
```

and:

```js
export function add() {}
export function subtract() {}
```

ES module imports/exports are statically analyzable, so the bundler can understand the module dependency graph before runtime.

---

## 5. Static vs Dynamic Dependencies

A static import:

```js
import { add } from "./math";
```

can be analyzed ahead of time.

Dynamic import:

```js
import("./math");
```

is primarily a **code-splitting/loading** mechanism.

Remember:

```text
Tree shaking
→ Remove unused code

Code splitting
→ Separate code into chunks
```

They can work together.

---

## 6. Tree Shaking vs Minification

### Tree shaking

Removes unused code.

```text
Unused function
      ↓
Removed
```

### Minification

Makes remaining code smaller.

```text
Readable code
      ↓
Minified representation
```

Therefore:

```text
Tree shaking → What code can be removed?
Minification → How can remaining code be smaller?
```

---

## 7. Tree Shaking vs Code Splitting

### Tree shaking

```text
Remove code that isn't used
```

### Code splitting

```text
Separate code so it can load independently
```

Example:

```text
5 MB total application

Tree shaking
    ↓
3 MB necessary code

Code splitting
    ↓
Initial: 1 MB
Lazy chunks: 2 MB
```

These are different optimizations that can work together.

---

## 8. Common Interview Trap

Don't say:

> "Tree shaking removes unused files."

More accurately:

> **Tree shaking removes unused code from the final bundle when the bundler can statically determine that the code is not needed.**

It is about **unused code**, not simply unused files.

---

## 9. What Can Prevent Effective Tree Shaking?

Tree shaking depends on the bundler being able to safely determine what code is unused.

Problems can arise from:

- modules with side effects
- CommonJS modules
- dynamic patterns
- poorly structured dependencies
- package configuration that prevents effective elimination

For example:

```js
require("./some-module");
```

is generally harder to analyze statically than a straightforward ES module import.

---

## 10. Side Effects Matter

Suppose a module contains:

```js
console.log("Module initialized");
```

or modifies global state during import.

Even if none of its exported functions are used, removing the entire module could change application behavior.

Therefore bundlers need to account for **side effects**.

Package metadata such as:

```json
{
  "sideEffects": false
}
```

can matter for bundling.

It should only be declared when it is actually safe for the package.

---

## 11. Interview Example

Suppose:

```js
import {
  formatDate,
  parseDate,
  validateDate
} from "./date-utils";
```

but the application only uses:

```js
formatDate();
```

A tree-shakable build can potentially eliminate:

```text
parseDate
validateDate
```

The exact result depends on the module structure and bundler.

---

## 12. Mental Model

```text
Application
     ↓
Module dependency graph
     ↓
Determine used exports
     ↓
Remove unreachable/unused code
     ↓
Minify remaining code
     ↓
Produce smaller bundle
```

---

## 13. Golden Rules

1. Tree shaking removes unused code from production bundles.
2. It works especially well with statically analyzable ES modules.
3. Tree shaking ≠ minification.
4. Tree shaking ≠ code splitting.
5. Side effects can prevent safe removal.
6. CommonJS can be harder to analyze than ES modules.
7. `sideEffects` metadata can help bundlers when correctly declared.
8. Tree shaking reduces unnecessary JavaScript before delivery.
9. Always verify bundle output rather than assuming tree shaking worked.
10. Tree shaking, code splitting, minification, and compression can complement each other.

---

## Quick Interview Answer

> **Tree shaking is a build-time optimization that removes unused, statically analyzable code from the production bundle, reducing the JavaScript that users need to download and process.**
