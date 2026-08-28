# Chapter 41 — Context API & `useContext`

## 1. Core Mental Model

Context lets a value be available to a subtree without manually passing it through every intermediate component.

```text
Provider
   ↓
   ↓
   ↓
Consumer
```

Create a context:

```jsx
const ThemeContext = createContext(null);
```

Provide a value:

```jsx
<ThemeContext.Provider value="dark">
  <App />
</ThemeContext.Provider>
```

Read it:

```jsx
const theme = useContext(ThemeContext);
```

> **Context solves prop drilling by making a value available to components deeper in a subtree.**

---

## 2. Prop Drilling

Without Context:

```text
App
 ↓
Dashboard
 ↓
UserProfile
 ↓
Avatar
```

If `Avatar` needs `user`, every intermediate component may have to receive and forward it:

```jsx
<App user={user}>
  <Dashboard user={user}>
    <UserProfile user={user}>
      <Avatar user={user} />
    </UserProfile>
  </Dashboard>
</App>
```

This is **prop drilling**.

---

## 3. Provider

A Provider supplies the context value:

```jsx
<ThemeContext.Provider value="dark">
  <App />
</ThemeContext.Provider>
```

Everything underneath that Provider can access the value.

Mental model:

```text
Provider
value = "dark"
      ↓
 ┌────┴────┐
 ↓         ↓
Child     Child
 ↓
useContext()
 ↓
"dark"
```

---

## 4. `useContext`

```jsx
const theme = useContext(ThemeContext);
```

This reads the current value from the nearest matching Provider above the component.

---

## 5. Context Is Not Global State

Context does not create a magical global variable.

Instead:

```text
Context
→ makes a value available to a subtree
```

The value comes from a Provider.

Different Providers can supply different values:

```jsx
<ThemeContext.Provider value="dark">
  <App />
</ThemeContext.Provider>

<ThemeContext.Provider value="light">
  <OtherApp />
</ThemeContext.Provider>
```

Different subtrees can therefore receive different context values.

---

## 6. Nearest Provider Wins

Example:

```jsx
<ThemeContext.Provider value="dark">
  <App>
    <ThemeContext.Provider value="light">
      <Dashboard />
    </ThemeContext.Provider>
  </App>
</ThemeContext.Provider>
```

`Dashboard` receives:

```text
light
```

because it uses the nearest matching Provider above it.

Mental model:

```text
Outer Provider
   ↓
Inner Provider
   ↓
Consumer
```

The inner value wins.

---

## 7. What Happens When Context Changes?

Suppose:

```jsx
const [theme, setTheme] = useState("light");

<ThemeContext.Provider value={theme}>
  <App />
</ThemeContext.Provider>
```

Then:

```jsx
setTheme("dark");
```

The Provider's value changes.

Consumers of that context can update:

```text
theme changes
    ↓
Provider value changes
    ↓
context consumers update
```

---

## 8. `React.memo` Does Not Block Context Updates

Example:

```jsx
const Child = React.memo(function Child() {
  const theme = useContext(ThemeContext);

  return <div>{theme}</div>;
});
```

Even though the component is memoized, a change to the context it consumes can cause it to update.

Important:

> **`React.memo` is not a universal "never render" switch.**

---

## 9. Context + Object Values

This can create unnecessary updates:

```jsx
<ThemeContext.Provider
  value={{
    theme,
    toggleTheme
  }}
>
```

The object is created during rendering.

Conceptually:

```text
Render 1 → Object A
Render 2 → Object B
```

So the context value reference changes.

When appropriate, `useMemo` can stabilize the value:

```jsx
const value = useMemo(() => ({
  theme,
  toggleTheme
}), [theme, toggleTheme]);
```

Now the value can retain the same reference when its dependencies haven't changed.

---

## 10. Good Context Use Cases

Context is useful for values shared across a subtree, such as:

```text
Theme
Authentication/user information
Locale
Feature configuration
Application-level settings
```

The key is that many components genuinely need access to the same value.

---

## 11. Don't Put Everything in Context

Avoid:

```text
Every piece of state
       ↓
Context
```

Context can make dependencies less explicit and can cause broader updates.

For local state:

```jsx
const [count, setCount] = useState(0);
```

is often simpler.

Use Context when a value genuinely needs to be shared across a subtree.

---

## 12. Context vs Props

### Props

```text
Parent
 ↓
Child
 ↓
Grandchild
```

Data flow is explicit.

### Context

```text
Provider
 ↓
 ├── Component
 ├── Component
 └── Deep Component
          ↓
      useContext()
```

Context is useful when many levels would otherwise need to forward the same value.

---

## 13. Context vs `useReducer`

They solve different problems.

### `useReducer`

Answers:

> **How should this state change?**

```text
action
 ↓
reducer
 ↓
new state
```

### Context

Answers:

> **How can components access this value without prop drilling?**

```text
Provider
 ↓
consumer
```

They can be combined:

```jsx
const [state, dispatch] = useReducer(
  reducer,
  initialState
);

<Context.Provider value={{ state, dispatch }}>
  <App />
</Context.Provider>
```

Mental model:

```text
useReducer
→ manages state transitions

Context
→ distributes state/dispatch
```

---

## 14. Context Default Value

You can create:

```jsx
const ThemeContext = createContext("light");
```

The `"light"` value is used when there is no matching Provider above the consumer.

Important:

> The default value is a fallback value, not automatically shared mutable state.

---

## 15. Interview Questions

### Q1. What problem does Context solve?

> It avoids prop drilling by allowing values to be consumed deeper in a subtree without manually passing them through intermediate components.

### Q2. Is Context global state?

> Not exactly. Context provides a value to consumers within a Provider's subtree.

### Q3. Which Provider does a consumer use?

> The nearest matching Provider above it.

### Q4. Does `React.memo` prevent context updates?

> No. A memoized component that consumes changing context can still update.

### Q5. Context vs `useReducer`?

> Context distributes values; `useReducer` manages state transitions. They can be used together.

### Q6. Should all application state go into Context?

> No. Context is best for values that genuinely need to be shared across a subtree.

---

# Quick Revision

```text
createContext()
→ creates context
```

```text
Provider
→ supplies value
```

```text
useContext()
→ reads nearest Provider value
```

```text
Nearest Provider
→ wins
```

```text
Provider value changes
→ consumers can update
```

```text
Context
≠
global variable
```

```text
Context
→ distribution
```

```text
useReducer
→ state transitions
```

```text
React.memo
≠
protection from context updates
```

---

# Final Mental Model

```text
             Provider
          value = X
               ↓
      ┌────────┼────────┐
      ↓        ↓        ↓
   Child    Child    Child
                       ↓
                  useContext()
                       ↓
                       X
```

### Remember:

> **Props pass data explicitly. Context makes shared data available through a subtree.**

> **Context distributes state; it doesn't replace state management.**

**Chapter 41 — COMPLETE**
