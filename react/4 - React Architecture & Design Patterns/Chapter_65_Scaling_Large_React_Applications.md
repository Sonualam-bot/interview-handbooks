# Chapter 65 — Scaling Large React Applications

## Handbook

**Handbook 4 — React Architecture & Design Patterns**

Chapter 65 is the final high-ROI chapter we are covering from Handbook 4 for the current interview preparation.

---

## 1. The Problem

A small React application can start with:

```text
src/
├── App.jsx
├── components/
├── hooks/
└── utils/
```

As it grows, you may have:

```text
100+ components
50+ hooks
30+ API modules
multiple features
multiple teams
```

The problem is not simply the number of files.

The real problem is maintaining:

- clear boundaries
- clear responsibilities
- predictable dependencies
- maintainability

---

## 2. Main Principle

A scalable React application should have:

```text
Clear responsibilities
        +
Clear boundaries
        +
Predictable dependencies
        +
Reusable shared code
```

> **Good architecture makes growing complexity easier to manage.**

---

## 3. Feature-Based Organization

Instead of organizing only by technical type:

```text
components/
hooks/
services/
utils/
```

organize around business features:

```text
src/
├── features/
│   ├── auth/
│   ├── dashboard/
│   ├── orders/
│   └── profile/
│
├── shared/
│   ├── components/
│   ├── hooks/
│   └── utils/
│
└── app/
```

Mental model:

```text
Application
   ↓
Features
   ↓
Feature-specific UI + logic + data
```

---

## 4. Example Feature Structure

For an ecommerce application:

```text
features/
└── products/
    ├── components/
    │   ├── ProductCard.jsx
    │   └── ProductList.jsx
    │
    ├── hooks/
    │   └── useProducts.js
    │
    ├── api/
    │   └── productApi.js
    │
    └── index.js
```

Everything related to the products feature stays close together.

---

## 5. Shared vs Feature-Specific Code

Ask:

> **Does this belong to one feature, or does the application genuinely need it everywhere?**

Feature-specific:

```text
features/
└── orders/
    ├── OrderList
    ├── useOrders
    └── orderApi
```

Shared:

```text
shared/
├── Button
├── Modal
├── useDebounce
└── formatDate
```

Important rule:

> **Share something because multiple features genuinely need it, not because it might be reusable someday.**

---

## 6. Separation of Responsibilities

Avoid putting everything inside one component:

```jsx
function Dashboard() {
  // fetch data
  // transform data
  // business logic
  // state management
  // WebSocket logic
  // rendering
  // error handling
  // formatting
}
```

Instead:

```text
Component
→ UI

Hook
→ component-specific behavior

API/service
→ backend communication

State layer
→ shared application state

Utility
→ generic pure logic
```

Mental model:

```text
UI
 ↓
Hook / state
 ↓
Service / API
 ↓
Backend
```

---

## 7. Avoid the "God Component"

A God Component accumulates too many responsibilities:

```text
One component
 ├── UI
 ├── API calls
 ├── business logic
 ├── state
 ├── effects
 ├── WebSocket
 ├── validation
 └── everything else
```

Problems:

- difficult to test
- difficult to understand
- difficult to modify
- difficult to reuse

Instead, split responsibilities:

```text
Dashboard
 ├── DashboardHeader
 ├── DashboardFilters
 ├── DashboardContent
 └── DashboardFooter

useDashboard()
dashboardApi
dashboardUtils
```

---

## 8. State Ownership

A major scaling question is:

> **Where should this state live?**

Not every piece of state belongs in Redux or Context.

Examples:

```text
Input value
→ local state

Modal open/closed
→ local state

Shared authenticated user
→ shared state/context

Server data
→ server-state solution such as TanStack Query
```

Rule:

> **Keep state as close as possible to where it is needed, while lifting or sharing it when necessary.**

---

## 9. Client State vs Server State

### Client state

State owned by the UI/application:

```text
theme
modal state
selected tab
local filters
UI preferences
```

### Server state

Data originating from the backend:

```text
users
orders
products
dashboard data
```

Server state often requires:

```text
caching
refetching
stale handling
synchronization
loading states
error states
```

This is why a tool such as TanStack Query can be more appropriate for server state than putting all API data into Redux.

---

## 10. Clear Dependency Direction

Keep dependencies predictable.

Conceptually:

```text
UI
 ↓
feature logic
 ↓
data/API layer
```

Avoid situations where everything imports everything else:

```text
A → B
↑   ↓
D ← C
```

This can lead to:

- circular dependencies
- tight coupling
- difficult refactoring

---

## 11. Shared Components Should Stay Generic

A shared component should ideally avoid business-specific assumptions.

Good:

```jsx
<Button variant="primary">
  Save
</Button>
```

Less ideal as a generic shared component:

```jsx
<ProductCheckoutButton />
```

Shared components should remain broadly reusable.

---

## 12. Reuse Through Composition

This connects directly to Chapter 51.

Instead of:

```jsx
<Card
  showHeader
  showFooter
  showActions
  showAvatar
/>
```

compose:

```jsx
<Card>
  <CardHeader />
  <CardBody />
  <CardFooter />
</Card>
```

Scaling architecture is not only about folder structure.

It is also about designing flexible components.

---

## 13. Custom Hooks as Boundaries

Custom Hooks can separate behavior from presentation.

```jsx
function Dashboard() {
  const {
    data,
    isLoading,
    error
  } = useDashboard();

  // render UI
}
```

Behavior can live in:

```jsx
function useDashboard() {
  // fetching
  // state
  // subscriptions
  // transformation
}
```

Mental model:

```text
Component
→ primarily rendering

Custom Hook
→ behavior / state / effects
```

---

## 14. API / Data Layer

Avoid scattering API calls across unrelated components:

```jsx
function User() {
  fetch("/api/user");
}

function Profile() {
  fetch("/api/user");
}

function Settings() {
  fetch("/api/user");
}
```

Instead, centralize feature-related data access where appropriate:

```text
features/
└── users/
    └── api/
        └── userApi.js
```

Benefits:

- consistency
- easier testing
- maintainability
- easier API changes

---

## 15. Error Boundaries

Large applications should consider failure isolation.

Conceptually:

```text
Application
 ├── Header
 ├── Error Boundary
 │    └── Dashboard
 │
 └── Error Boundary
      └── Reports
```

A failure in one section can be isolated rather than necessarily taking down the entire application UI.

---

## 16. Performance Architecture

Scaling also involves performance.

Important areas include:

```text
Code splitting
Lazy loading
Memoization
Virtualization
Caching
Bundle size
Rendering cost
```

These are covered more deeply in Handbook 5.

---

## 17. Testing Architecture

Separating responsibilities makes testing easier:

```text
UI
Business logic
API/data access
Utilities
```

Possible test boundaries:

```text
Component test
→ rendering/UI behavior

Hook test
→ state/behavior

Utility test
→ pure logic

API layer test
→ data communication
```

---

## 18. Don't Over-Engineer Early

Scalable architecture does **not** mean:

```text
Every component
→ abstraction

Every function
→ service

Every value
→ global state

Every feature
→ 20 folders
```

That creates unnecessary complexity.

Good architecture evolves with the application.

> **Start simple, identify real boundaries, and introduce abstractions when complexity justifies them.**

---

# Interview Questions

### Q1. How would you scale a React application?

> I'd organize the application around features, keep feature-specific logic together, maintain a small shared layer for genuinely reusable code, establish clear state ownership, separate client state from server state, and keep UI, business logic and data access reasonably separated.

### Q2. Feature-based vs type-based organization?

**Type-based:**

```text
components/
hooks/
services/
utils/
```

**Feature-based:**

```text
features/
  auth/
  orders/
  dashboard/
```

Feature-based organization often scales better because related code stays together.

### Q3. Where should state live?

> As close as possible to the components that need it. Lift it when multiple components need it, and use appropriate shared-state or server-state tools when the scope requires them.

### Q4. Should all state go into Redux?

> No. Local UI state should usually remain local. Redux or another shared-state solution is useful when state genuinely needs broader application scope. Server state can often be handled better by tools such as TanStack Query.

### Q5. What's a God Component?

> A component that has accumulated too many responsibilities such as UI, API calls, business logic, state management and side effects. It becomes difficult to test and maintain.

### Q6. How do you decide whether something belongs in `shared/`?

> It should be genuinely reusable across multiple features and ideally have minimal business-specific assumptions. Don't move something to shared merely because it might be reused later.

---

# Quick Revision

```text
Large React App
→ clear boundaries
→ clear responsibilities
→ predictable dependencies
```

```text
Feature-based architecture
→ organize around business features
```

```text
Feature-specific code
→ keep close to its feature
```

```text
Shared code
→ genuinely reusable
```

```text
State
→ keep as local as possible
→ lift/share when necessary
```

```text
Client state
→ UI/application state
```

```text
Server state
→ backend data
→ caching/refetching/synchronization
```

```text
God Component
→ too many responsibilities
→ split it
```

```text
Custom Hook
→ behavior/logic boundary
```

```text
API layer
→ data-access boundary
```

```text
Good architecture
→ don't over-engineer
```

---

# Final Mental Model

```text
                 Large React App
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
       Feature A    Feature B    Feature C
          │            │            │
       UI/Logic     UI/Logic     UI/Logic
          │            │            │
          └────────────┼────────────┘
                       ↓
                    Shared
                       │
                 Generic utilities
                 Generic components
```

And underneath:

```text
UI
 ↓
Hooks / State
 ↓
API / Data layer
 ↓
Backend
```

### Interview line

> **For a large React application, I'd organize around features, keep feature-specific logic together, maintain clear state and data boundaries, share only genuinely reusable code, and avoid components or global stores accumulating unrelated responsibilities.**

**Chapter 65 — COMPLETE ✅**
