# Chapter 55 — Provider Pattern

## Handbook

**Handbook 4 — React Architecture & Design Patterns**

---

## 1. Core Mental Model

The Provider Pattern makes shared state, configuration, or dependencies available to a subtree of components without manually passing them through every level.

```text
Provider
   ↓
shared value
   ↓
Component A
   ↓
Component B
   ↓
Component C
```

> **Provider Pattern = make something available to a component subtree.**

---

## Deep Dive `[NEW]`

### Why Splitting Contexts Actually Reduces Re-renders — The Mechanism Behind Section 17's Advice

Section 17 recommends "splitting contexts when appropriate" without
explaining why that helps. Here's the mechanism: React's context
propagation tracks subscriptions **per distinct Context object**
(whatever `createContext()` returned), not per Provider component or
per value. Each consumer's fiber records which specific Context
objects it reads via `useContext`. When a Provider's value changes,
React's propagation walk only notifies fibers subscribed to *that*
Context object — it has no way to even express "also notify anyone
subscribed to some other, unrelated Context."

This means:

```jsx
// One combined context
<AppContext.Provider value={{ user, theme, setTheme }}>
```

vs.

```jsx
// Two separate contexts
<UserContext.Provider value={user}>
  <ThemeContext.Provider value={{ theme, setTheme }}>
```

With one combined context, a `theme` change re-renders every consumer
that reads `AppContext` — including ones that only ever cared about
`user` — because from React's point of view, the single Context object
changed, and it can't know a consumer only destructured part of it.
With two separate contexts, a `theme` change only propagates through
`ThemeContext`'s own subscriber list; `UserContext`'s consumers are
never touched, because they're subscribed to a different Context
object entirely. Splitting contexts isn't a vague "best practice" —
it's exploiting the fact that propagation granularity in React is
exactly as fine as the number of distinct Context objects you create.

## 2. The Problem: Prop Drilling

Without a provider:

```jsx
function App() {
  const user = { name: "Sonu" };

  return <Dashboard user={user} />;
}

function Dashboard({ user }) {
  return <Sidebar user={user} />;
}

function Sidebar({ user }) {
  return <UserProfile user={user} />;
}

function UserProfile({ user }) {
  return <h1>{user.name}</h1>;
}
```

The data travels through components that may not need it:

```text
App
 ↓ user
Dashboard
 ↓ user
Sidebar
 ↓ user
UserProfile
```

This is prop drilling.

---

## 3. Provider Pattern as the Solution

Instead:

```text
UserProvider
    ↓
Dashboard
    ↓
Sidebar
    ↓
UserProfile
```

`UserProfile` can access the shared value directly through context.

The intermediate components don't need to forward it.

---

## 4. Basic Example

Create a context:

```jsx
const UserContext = createContext(null);
```

Create a provider:

```jsx
function UserProvider({ children }) {
  const user = {
    name: "Sonu"
  };

  return (
    <UserContext.Provider value={user}>
      {children}
    </UserContext.Provider>
  );
}
```

Wrap the application:

```jsx
function App() {
  return (
    <UserProvider>
      <Dashboard />
    </UserProvider>
  );
}
```

Consume it:

```jsx
function UserProfile() {
  const user = useContext(UserContext);

  return <h1>{user.name}</h1>;
}
```

Mental model:

```text
UserProvider
      ↓
   Dashboard
      ↓
   Sidebar
      ↓
 UserProfile
      ↓
 useContext(UserContext)
      ↓
      user
```

---

## 5. Why Is It Called a Provider?

Because the component is **providing something to its descendants**.

Examples:

```text
ThemeProvider
→ provides theme

AuthProvider
→ provides authentication state

UserProvider
→ provides user information

QueryClientProvider
→ provides query client

Redux Provider
→ provides Redux store
```

The value being provided depends on the application.

---

## 6. Provider vs Context

This distinction is important.

### Context

Defines the communication mechanism:

```jsx
const ThemeContext = createContext();
```

### Provider

Supplies a value through that context:

```jsx
<ThemeContext.Provider value={theme}>
  {children}
</ThemeContext.Provider>
```

### `useContext`

Consumes the current value:

```jsx
const theme = useContext(ThemeContext);
```

Mental model:

```text
Context
→ communication mechanism

Provider
→ supplies the value

useContext
→ consumes the value
```

---

## 7. Provider Pattern Is Broader Than Context

The Provider Pattern is an architectural idea, not only a React Context feature.

The general model is:

```text
Provider
 ↓
dependency / state / configuration
 ↓
descendants
```

Things a provider can make available:

```text
API client
Theme
Authentication
State store
Query client
Configuration
```

---

## 8. Provider + State

A provider can own state:

```jsx
function CounterProvider({ children }) {
  const [count, setCount] = useState(0);

  return (
    <CounterContext.Provider
      value={{ count, setCount }}
    >
      {children}
    </CounterContext.Provider>
  );
}
```

Then:

```jsx
function Counter() {
  const { count, setCount } = useContext(CounterContext);

  return (
    <button onClick={() => setCount(c => c + 1)}>
      {count}
    </button>
  );
}
```

The provider becomes a state boundary for its subtree.

---

## 9. Provider + Custom Hook

A common production pattern:

```jsx
const AuthContext = createContext(null);

function AuthProvider({ children }) {
  const [user, setUser] = useState(null);

  return (
    <AuthContext.Provider value={{ user, setUser }}>
      {children}
    </AuthContext.Provider>
  );
}
```

Then:

```jsx
function useAuth() {
  return useContext(AuthContext);
}
```

Consumer:

```jsx
function Profile() {
  const { user } = useAuth();

  return <div>{user.name}</div>;
}
```

Mental model:

```text
Profile
 ↓
useAuth()
 ↓
AuthContext
 ↓
AuthProvider
```

The consumer doesn't need to know the underlying context implementation.

---

## 10. Provider Nesting

Applications can have multiple providers:

```jsx
<App>
  <AuthProvider>
    <ThemeProvider>
      <QueryProvider>
        <Router>
          <Application />
        </Router>
      </QueryProvider>
    </ThemeProvider>
  </AuthProvider>
</App>
```

Each provider can supply a different dependency or state.

---

## 11. Provider Scope

A provider only affects its subtree.

```text
Provider
 ├── A       ← can access
 ├── B       ← can access
 │    └── C  ← can access
 └── D       ← can access
```

A component outside the provider cannot consume that provider's value.

This makes providers useful for creating **scoped state/dependency boundaries**.

---

## 12. Nested Providers

A nested provider can override the value for its subtree.

Conceptually:

```text
Outer Provider
→ dark

    Nested Provider
    → light
```

Components under the nested provider receive the nested value.

---

## 13. Provider Pattern and Dependency Injection

Suppose a component needs an API client.

Instead of constructing it itself:

```jsx
function UserList() {
  const api = new ApiClient();
}
```

you can provide the dependency:

```text
ApiProvider
    ↓
API client
    ↓
UserList
```

The component consumes a dependency supplied by the provider.

> **A provider can inject a dependency into a component subtree.**

---

## 14. Provider Pattern and Redux

Redux uses the Provider Pattern:

```jsx
<Provider store={store}>
  <App />
</Provider>
```

Mental model:

```text
Redux Provider
      ↓
Redux store
      ↓
Application
      ↓
components
```

The provider makes the Redux store available to descendants.

---

## 15. Provider Pattern and TanStack Query

TanStack Query also uses a provider:

```jsx
<QueryClientProvider client={queryClient}>
  <App />
</QueryClientProvider>
```

Mental model:

```text
QueryClientProvider
       ↓
Query client
       ↓
Application
```

This is especially useful to recognize in real codebases.

---

## 16. Provider Pattern vs Prop Drilling

### Prop drilling

```text
Parent
 ↓ props
Child
 ↓ props
Grandchild
 ↓ props
Deep component
```

### Provider

```text
       Provider
          ↓
    ┌─────┼─────┐
    ↓     ↓     ↓
    A     B     C
                ↓
             Consumer
```

The consumer can access the value without every intermediate component receiving it.

---

## 17. Performance Consideration

Context/provider isn't automatically free.

If the provider's value changes, consumers of that context may need to update.

For example:

```jsx
function Provider() {
  const [count, setCount] = useState(0);

  return (
    <Context.Provider value={{ count, setCount }}>
      {children}
    </Context.Provider>
  );
}
```

The object:

```jsx
{ count, setCount }
```

is recreated on renders.

In larger applications, think about:

- provider boundaries
- what state belongs in context
- how frequently the context value changes
- splitting contexts when appropriate

---

## 18. Don't Put Everything in One Provider

Avoid a giant provider containing unrelated concerns:

```text
GlobalProvider
 ├── auth
 ├── theme
 ├── notifications
 ├── cart
 ├── filters
 ├── modal
 ├── user
 └── everything else
```

Prefer meaningful boundaries when domains genuinely need separate shared state:

```text
AuthProvider
ThemeProvider
NotificationProvider
CartProvider
```

This helps avoid unnecessary coupling.

---

# Interview Questions

### Q1. What is the Provider Pattern?

> It is a pattern where a provider makes shared state, configuration, or dependencies available to a subtree of components, commonly through React Context.

### Q2. How does it solve prop drilling?

> Instead of passing a value through every intermediate component, a provider makes it available to descendants so consumers can access it directly.

### Q3. Provider vs Context?

> Context defines the mechanism for sharing a value; the provider supplies the value to the component subtree.

### Q4. Give examples of provider patterns.

Relevant examples:

```text
Redux <Provider>
TanStack Query <QueryClientProvider>
React Context providers
```

### Q5. What happens when a provider's context value changes?

> Components consuming that context may update because they need to receive the new context value.

### Q6. Why shouldn't you put everything into one Context?

> It creates a large shared state boundary and can cause unnecessary coupling and updates. Separate contexts/providers can create cleaner boundaries.

### Q7. How is the Provider Pattern related to Dependency Injection?

> A provider can inject a dependency, such as an API client or service, into a component subtree so consumers don't need to construct it themselves.

---

# Quick Revision

```text
Provider Pattern
→ make something available to a subtree
```

```text
Context
→ communication mechanism

Provider
→ supplies the value
```

```text
useContext
→ consumes the value
```

```text
Provider
→ can provide state
→ configuration
→ services
→ dependencies
```

```text
Provider
→ scoped to its subtree
```

```text
Redux Provider
→ provides Redux store
```

```text
QueryClientProvider
→ provides TanStack Query client
```

```text
Provider
→ can resemble Dependency Injection
```

```text
Avoid one giant provider
→ maintain meaningful boundaries
```

---

# Final Mental Model

```text
                  Provider
                     │
             shared dependency
                     │
          ┌──────────┼──────────┐
          ↓          ↓          ↓
       Child A    Child B    Child C
                                │
                                ↓
                           useContext()
                                │
                                ↓
                         shared value
```

### Interview line

> **The Provider Pattern allows a shared value, state, configuration, or dependency to be made available to an entire component subtree without passing it through every intermediate component.**

**Chapter 55 — COMPLETE**
