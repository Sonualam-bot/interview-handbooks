# React Architecture & Design Patterns
## Chapter 64 — Feature-Based Architecture

> Interview-focused notes. Goal: understand how component-level architecture grows into application-level organization.

---

## 1. Why Feature-Based Architecture?

As a React application grows, a structure such as:

```text
src/
├── components/
├── hooks/
├── services/
└── utils/
```

can become difficult to navigate.

The problem is not that these folders are inherently wrong. The problem is that they organize code mainly by **technical type**, rather than by **business responsibility**.

A large application may have hundreds of components, hooks, and services spread across these folders.

Feature-based architecture groups related code around meaningful application capabilities.

---

## 2. Definition

**Feature-Based Architecture** organizes application code around meaningful business features or domains.

Example:

```text
src/
├── features/
│   ├── auth/
│   ├── products/
│   ├── cart/
│   ├── search/
│   └── notifications/
│
├── components/
├── hooks/
└── utils/
```

A feature can contain the code needed to implement that capability:

```text
features/
└── cart/
    ├── components/
    │   ├── Cart.jsx
    │   ├── CartItem.jsx
    │   └── CartSummary.jsx
    │
    ├── hooks/
    │   └── useCart.js
    │
    ├── services/
    │   └── cartApi.js
    │
    └── utils/
        └── calculateTotal.js
```

The important idea:

> Everything primarily owned by the Cart feature stays close to the Cart feature.

---

## 3. Horizontal vs Vertical Organization

### Horizontal organization

Groups code by technical type:

```text
components/
hooks/
services/
utils/
```

This is convenient for small applications.

### Vertical / feature-oriented organization

Groups code by business capability:

```text
auth/
cart/
products/
search/
```

A feature can then contain its own UI, logic, API access, and utilities.

```text
             APPLICATION
                  │
       ┌──────────┼──────────┐
       ↓          ↓          ↓
      AUTH       CART       SEARCH
       │          │           │
   ┌───┼───┐  ┌───┼───┐   ┌───┼───┐
   UI Logic API UI Logic API UI Logic API
```

---

## 4. Why It Helps

Feature-based organization can improve:

- **Discoverability** — related code is in one place.
- **Maintainability** — feature changes are easier to localize.
- **Ownership** — it is clearer which feature owns a piece of behavior.
- **Isolation** — features can have clearer boundaries.
- **Team collaboration** — teams can work primarily within feature areas.
- **Scalability** — the structure remains understandable as the application grows.

Example:

If asked to add wishlist functionality, a natural home is:

```text
features/
└── wishlist/
```

rather than searching across unrelated global `components/`, `hooks/`, and `services/` folders.

---

## 5. A Feature Owns Its Responsibilities

Example:

```text
features/
└── auth/
    ├── components/
    │   ├── LoginForm.jsx
    │   └── SignupForm.jsx
    │
    ├── hooks/
    │   └── useAuth.js
    │
    ├── services/
    │   └── authApi.js
    │
    └── utils/
        └── authHelpers.js
```

Conceptually:

```text
             AUTH FEATURE
                  │
       ┌──────────┼──────────┐
       ↓          ↓          ↓
      UI         Logic       API
       │          │          │
 LoginForm      useAuth    authApi
```

The feature boundary tells you that these pieces are primarily related to authentication.

---

## 6. Not Everything Belongs Inside a Feature

Feature-based architecture does **not** mean every piece of code must live inside a feature.

Genuinely shared code can remain shared:

```text
src/
├── features/
│   ├── auth/
│   ├── cart/
│   └── products/
│
├── components/
│   ├── Button.jsx
│   ├── Modal.jsx
│   └── Spinner.jsx
│
├── hooks/
│   └── useDebounce.js
│
└── utils/
    └── formatCurrency.js
```

A useful rule:

> Keep code close to the feature that owns it until there is a genuine reason to share it.

Do not move code into `shared/` simply because it *might* be reused someday.

---

## 7. Feature-Based Architecture and State Colocation

This is an architectural extension of the state-colocation principle.

Earlier:

```text
State → keep it close to the components using it.
```

At the feature level:

```text
Feature-specific code → keep it close to the feature that owns it.
```

For example:

```text
features/
└── search/
    ├── components/
    ├── hooks/
    │   └── useSearch.js
    └── state/
        └── searchState.js
```

If search state is only needed by the search feature, there is no reason to immediately make it global.

---

## 8. Feature-Based Architecture and Component Boundaries

Chapter 63 focused on:

> Where should component responsibilities be separated?

Chapter 64 asks:

> Where should application responsibilities be separated?

A feature boundary is useful when an area has things such as:

- its own business responsibility
- its own state
- its own API interactions
- its own UI
- independent testing
- independent development
- a meaningful reason to change independently

Example:

```text
Dashboard
├── Search
├── Analytics
├── Notifications
└── Profile
```

These may naturally become:

```text
features/
├── search/
├── analytics/
├── notifications/
└── profile/
```

---

## 9. Feature vs Component

They are different architectural levels.

### Component thinking

```text
Button
Modal
ProductCard
SearchBar
```

asks:

> What UI building blocks do I have?

### Feature thinking

```text
Auth
Search
Cart
Checkout
```

asks:

> What capabilities does my application provide?

You normally use both:

```text
Application
   ↓
Features
   ↓
Components
   ↓
Elements
```

Example:

```text
Checkout
   ↓
CheckoutForm
   ↓
Input
Button
Select
```

---

## 10. Feature-Based Architecture and Custom Hooks

Custom Hooks can separate feature behavior from feature UI.

Example:

```text
features/
└── search/
    ├── components/
    │   ├── SearchBox.jsx
    │   └── SearchResults.jsx
    │
    └── hooks/
        └── useSearch.js
```

Conceptually:

```text
SearchBox
    ↓
useSearch()
    ↓
search state / behavior
    ↓
Search API
```

This gives the feature a cohesive internal structure:

```text
Feature
 ├── UI
 ├── behavior
 ├── data access
 └── feature-specific utilities
```

---

## 11. Feature Public API

A feature can expose a stable public interface rather than allowing consumers to depend on its internal folder structure.

Example:

```text
features/cart/
├── components/
├── hooks/
├── services/
└── index.js
```

`index.js`:

```js
export { Cart } from "./components/Cart";
export { useCart } from "./hooks/useCart";
```

Consumers use:

```js
import { Cart, useCart } from "@/features/cart";
```

instead of:

```js
import { Cart } from "@/features/cart/components/Cart";
import { useCart } from "@/features/cart/hooks/useCart";
```

### Why?

Consumers depend on the feature's **public API**, not its internal implementation.

That means the internal structure can change without forcing every consumer to change.

This is a form of **encapsulation**.

---

## 12. Feature Ownership and Teams

Feature boundaries can also make team ownership clearer.

Example:

```text
Team A → Auth
Team B → Checkout
Team C → Search
Team D → Notifications
```

Instead of everyone modifying the same global technical folders, teams can primarily work within:

```text
features/auth/
features/checkout/
features/search/
features/notifications/
```

This can reduce the mental surface area for development.

---

## 13. Dependency Direction

Feature architecture should make dependencies understandable.

A feature can depend on genuinely shared components:

```text
features/cart
      ↓
components/Button
```

But a supposedly shared component should not depend on a specific feature:

```text
components/Button
      ✕
features/cart
```

Otherwise the shared layer becomes coupled to a specific business feature.

A useful mental model:

```text
Feature-specific
      ↓
Shared
```

rather than:

```text
Shared
      ↓
Specific feature
```

---

## 14. Cross-Feature Communication

Features may need to communicate.

For example:

```text
Checkout
   ↓
needs current user
   ↓
Auth
```

That does not mean Checkout should import Auth's internal implementation.

Prefer a stable interface:

```js
import { useCurrentUser } from "@/features/auth";
```

rather than:

```js
import { authStore } from "@/features/auth/internal/store";
```

The first depends on a public contract.

The second depends on an implementation detail.

---

## 15. Avoid Over-Fragmenting Features

Do not turn every small UI element into a feature.

Good feature boundaries:

```text
auth
cart
search
checkout
notifications
```

Bad examples:

```text
button-click
search-icon
profile-name
header-title
```

A feature should represent a meaningful capability or domain responsibility.

---

## 16. Feature-Based Architecture Is Not a Silver Bullet

It has tradeoffs.

### Advantages

- Better discoverability
- Clearer ownership
- Reduced coupling
- Easier feature development
- Better scalability
- Better team ownership
- Related code stays together

### Potential problems

- A feature can become too large.
- Feature boundaries can be arbitrary.
- Shared folders can become dumping grounds.
- Cross-feature dependencies can become messy.
- Overengineering is possible.

Therefore, do not claim:

> "Feature-based architecture is always better."

A better statement:

> "For sufficiently large applications, organizing around meaningful business capabilities can make ownership and dependencies clearer."

---

# 17. Feature-Based Architecture vs Traditional Structure

| Traditional | Feature-Based |
|---|---|
| Organized by technical type | Organized by business capability |
| Components are globally grouped | Feature components stay near the feature |
| Hooks are globally grouped | Feature hooks can live inside the feature |
| Services are globally grouped | Feature APIs can live inside the feature |
| Can become hard to navigate at scale | Easier to locate feature-specific code |
| Simple for small apps | Useful as applications grow |

Neither structure is universally correct.

The architecture should match the application's size and complexity.

---

# 18. Interview Scenario

### Question

> "You have a React application with 500 components. How would you structure it?"

### Strong answer

> "For a larger application, I'd generally organize code around business features or domains. Each feature can contain its own components, hooks, state, API logic, and utilities. Truly shared UI and utilities can live in shared modules. I'd also establish clear feature boundaries and public interfaces so features don't depend on each other's internal implementation."

The key is that you explain **why**, not just the folder structure.

---

# 19. Data-Flow Exercise 🧠

Imagine:

```text
App
├── Dashboard
│   ├── SearchBox
│   ├── SearchResults
│   ├── Notifications
│   └── UserProfile
│
└── Settings
```

Requirements:

- Search has its own API.
- Notifications poll for new notifications.
- UserProfile uses authenticated user information.
- Settings modifies user preferences.
- Search state is only needed by SearchBox + SearchResults.
- Auth state is needed by UserProfile and Settings.

### Step 1 — Identify features

Design:

```text
features/
├── ?
├── ?
├── ?
└── ?
```

Think about meaningful business capabilities, not individual components.

### Step 2 — Identify state ownership

Ask:

- Where should search state live?
- Where should auth state live?
- Should search state be global?
- Should notification state belong to the Dashboard?

### Step 3 — Identify logic ownership

Ask:

- Where should search API logic live?
- Where should notification polling logic live?
- Where should authentication behavior live?
- Where should user-preference logic live?

### Step 4 — Identify shared UI

Ask which pieces might belong in:

```text
components/
```

rather than inside a feature.

### Step 5 — Trace search data flow

Complete:

```text
User types "react"
        ↓
?
        ↓
?
        ↓
Search API
        ↓
?
        ↓
SearchResults
```

The important part is not memorizing a folder structure.

The important part is being able to explain:

> **Who owns the state? Who owns the behavior? Who owns the API? How does the data move? Which boundaries prevent unrelated code from becoming coupled?**

---

# 20. Connection to Previous Chapters

The architecture progression is:

```text
59 Controlled vs Uncontrolled
        ↓
Who owns state?

60 State Colocation
        ↓
Where should state live?

61 Lifting State Up
        ↓
What if multiple components need it?

62 Prop Drilling & Alternatives
        ↓
What if data travels too far?

63 Component Boundaries
        ↓
Where should responsibilities be separated?

64 Feature-Based Architecture
        ↓
How should larger responsibilities be organized?

65 Scaling Large React Applications
        ↓
How do we scale the whole system?
```

This is the key architectural story of Handbook 4.

---

# 21. Interview Cheat Sheet

### What is Feature-Based Architecture?

> Organizing an application around meaningful business features or domains rather than only technical file types.

### Why use it?

> It improves discoverability, ownership, maintainability, and scalability by keeping related code together.

### Does everything belong inside a feature?

> No. Truly shared components, hooks, and utilities can remain shared.

### How do you decide whether something is feature-specific?

Ask:

> Who owns its meaning and behavior?

### What should features expose?

> Prefer stable public interfaces rather than exposing internal implementation details.

### How does it relate to state colocation?

> Both follow the principle of keeping responsibility close to where it is actually needed.

### Is it always better?

> No. For small applications, a simpler structure may be preferable. Feature-based architecture becomes increasingly useful as the application and team grow.

---

# 22. The One Mental Model to Remember

```text
                 APPLICATION
                      │
          ┌───────────┼───────────┐
          ↓           ↓           ↓
        AUTH         CART       SEARCH
          │           │           │
       ┌──┼──┐     ┌──┼──┐     ┌──┼──┐
       UI Logic     UI Logic     UI Logic
          │           │           │
          └───────────┼───────────┘
                      ↓
                 SHARED UI
```

The goal is **not** simply a particular folder structure.

The goal is:

```text
Meaningful boundaries
        ↓
Clear ownership
        ↓
Controlled dependencies
        ↓
Lower coupling
        ↓
Easier scaling
```

---

## ⭐ Final Interview Takeaway

> **Feature-Based Architecture organizes a React application around meaningful business capabilities, with each feature owning its related UI, state, behavior, and data-access logic while genuinely shared code remains in shared modules. The goal is to create clear ownership and dependency boundaries so the application remains understandable as it grows.**
