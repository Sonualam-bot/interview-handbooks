# Chapter 66 — Why React Re-renders

## Handbook

**Handbook 5 — Performance Engineering**

---

## 1. What Is a Re-render?

A React re-render means React **calls the component function again** to calculate the next React element output.

Example:

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  console.log("render");

  return <button>{count}</button>;
}
```

When:

```jsx
setCount(1);
```

React schedules another render.

Mental model:

```text
setState
   ↓
React schedules update
   ↓
Component function executes again
   ↓
new React elements
   ↓
reconciliation
   ↓
commit if necessary
```

---

## 2. Re-render ≠ DOM Update

This is extremely important.

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  return <div>Hello</div>;
}
```

If `count` changes but isn't used in the returned UI:

```text
state changes
 ↓
component re-renders
 ↓
React calculates new output
 ↓
output is effectively the same
 ↓
little/no DOM mutation
```

> **Rendering is not the same thing as updating the DOM.**

Remember:

```text
Render
 ↓
Reconciliation
 ↓
Commit
 ↓
Browser paint
```

---

## 3. Main Causes of Re-renders

### 1. State changes

```jsx
setCount(count + 1);
```

The component that owns the state can be scheduled to render again.

### 2. Parent re-renders

If a parent renders again, its children are normally encountered/rendered again unless React can bail out.

### 3. Consumed context changes

If a component consumes a context value and the relevant value changes, the consumer can update.

### 4. External subscriptions/stores

Components connected to external state systems can update when their subscribed data changes.

---

## 4. Props and Re-renders

Suppose:

```jsx
<Child count={count} />
```

When the parent state changes:

```text
Parent state changes
 ↓
Parent renders
 ↓
new props are calculated
 ↓
Child receives props
```

Don't simplify this to:

> "Props changed → component always re-renders."

The more accurate model is:

> **A parent rendering normally causes its child to be considered/rendered, unless something such as memoization allows React to bail out.**

---

## 5. Parent Re-render ≠ DOM Rebuild

Example:

```jsx
function Parent() {
  const [count, setCount] = useState(0);

  return (
    <>
      <h1>Hello</h1>
      <Child />
      <button onClick={() => setCount(count + 1)}>
        {count}
      </button>
    </>
  );
}
```

When `count` changes:

```text
Parent renders again
       ↓
new React element tree
       ↓
reconciliation
       ↓
React compares old vs new
       ↓
only necessary DOM changes
```

React does not throw away the entire DOM and rebuild it.

---

## 6. `React.memo`

`React.memo` can allow React to skip rendering a child when its relevant props are unchanged.

```jsx
const Child = React.memo(function Child({ count }) {
  return <div>{count}</div>;
});
```

Conceptually:

```text
Parent renders
 ↓
Child props unchanged
 ↓
React.memo bailout
 ↓
Child render can be skipped
```

This is useful when the child is expensive enough for avoiding its render to matter.

---

## 7. Object Props and Reference Identity

Consider:

```jsx
function Parent() {
  const user = {
    name: "Sonu"
  };

  return <Child user={user} />;
}
```

Every parent render creates a new object:

```text
render 1 → object A
render 2 → object B
render 3 → object C
```

Even though the contents look identical:

```js
objectA !== objectB
```

This matters when using `React.memo`, because reference identity is relevant to prop comparison.

---

## 8. Function Props and Reference Identity

Example:

```jsx
function Parent() {
  const handleClick = () => {
    console.log("clicked");
  };

  return <Child onClick={handleClick} />;
}
```

Every render creates a new function reference:

```text
render 1 → function A
render 2 → function B
render 3 → function C
```

Therefore:

```js
functionA !== functionB
```

This is one reason `useCallback` can sometimes be useful.

---

## 9. Don't Memoize Everything

Don't blindly do:

```text
Every component
→ React.memo

Every function
→ useCallback

Every calculation
→ useMemo
```

Memoization has its own:

- comparison cost
- memory cost
- complexity

Use it when there is a meaningful performance reason.

---

## 10. State Updates With the Same Value

For primitive values:

```jsx
setCount(3);
setCount(3);
```

React can recognize that the state value hasn't changed and avoid unnecessary work/bail out.

Be careful with objects:

```jsx
setUser({ name: "Sonu" });
setUser({ name: "Sonu" });
```

These are different object references:

```js
{ name: "Sonu" } !== { name: "Sonu" }
```

So React can see a different state value.

---

## 11. Context and Re-renders

Example:

```jsx
function Child() {
  const theme = useContext(ThemeContext);

  return <div>{theme}</div>;
}
```

If the provider's relevant value changes:

```text
Provider value changes
       ↓
Child consumes changed context
       ↓
Child updates
```

This is one reason a giant Context containing frequently changing unrelated state can become problematic.

---

## 12. Re-render Doesn't Mean Everything Below Changed

Suppose:

```text
App
 ↓
Dashboard
 ├── Header
 ├── Sidebar
 └── Results
```

`Dashboard` renders again.

That does not mean:

```text
Header DOM changed
Sidebar DOM changed
Results DOM changed
```

React still reconciles the output to determine what actually needs to change.

Remember:

```text
Render
→ calculate

Reconciliation
→ determine differences

Commit
→ apply required DOM changes
```

---

## 13. Re-render vs Remount

### Re-render

The component function executes again while the component retains its identity/state.

```text
same component identity
 ↓
render again
```

### Remount

React removes the old component and creates a new one.

```text
old component
 ↓
removed
 ↓
new component
```

A remount can reset state and trigger the relevant cleanup/setup lifecycle.

> **Re-render does not mean the component was destroyed.**

---

## 14. Why Re-render If the DOM Doesn't Change?

React needs to calculate what the UI **should be** for the new state, props, or context.

Example:

```jsx
function Counter({ count }) {
  return <h1>{count}</h1>;
}
```

Old output:

```jsx
<h1>1</h1>
```

New output:

```jsx
<h1>2</h1>
```

React needs the new render output before reconciliation can determine the necessary DOM change.

---

## 15. Debugging Re-renders

If asked:

> "How would you find why a component is re-rendering?"

Think:

```text
React DevTools
      ↓
Profiler
      ↓
Which component rendered?
      ↓
What changed?
      ↓
props?
state?
context?
parent?
external subscription?
      ↓
reference identity?
```

Inspect:

- changing object props
- changing function props
- context updates
- state updates
- unnecessary parent renders

---

# Interview Questions

### Q1. What causes a React component to re-render?

> Its state changes, its parent renders, a consumed context value changes, or an external subscription/store causes an update.

### Q2. Does re-render mean the DOM is updated?

> No. A component can re-render and produce the same output. Reconciliation determines whether DOM changes are actually necessary.

### Q3. If a parent re-renders, does the child re-render?

> Normally the child is considered/rendered as part of the parent's rendering process, unless React can bail out, for example through `React.memo`.

### Q4. Why can `React.memo` fail to prevent a child render?

Because props can have new references:

```jsx
<Child user={{ name: "Sonu" }} />
```

or:

```jsx
<Child onClick={() => doSomething()} />
```

Each render creates a new object/function reference.

### Q5. Re-render vs remount?

> A re-render executes the component again while preserving its identity/state. A remount removes the old component and creates a new one.

### Q6. Does React rebuild the whole DOM after every render?

> No. React calculates a new React element tree, reconciles it against the previous one, and commits only the necessary DOM changes.

---

# Quick Revision

```text
State change
→ can trigger render
```

```text
Parent render
→ child normally considered/rendered
```

```text
Context value change
→ consumers can update
```

```text
Re-render
≠
DOM update
```

```text
Render
→ calculate new UI
```

```text
Reconciliation
→ determine necessary changes
```

```text
Commit
→ apply DOM changes
```

```text
React.memo
→ can bail out when props are unchanged
```

```text
New object/function reference
→ can defeat memoization
```

```text
Re-render
≠
remount
```

---

# Final Mental Model

```text
             UPDATE
                ↓
       ┌────────┼────────┐
       ↓        ↓        ↓
     State     Props   Context
       │
       └────────┬────────┘
                ↓
             Render
                ↓
       component executes
                ↓
        new React elements
                ↓
         Reconciliation
                ↓
       "What actually changed?"
                ↓
             Commit
                ↓
        required DOM changes
```

### Interview line

> **"A React re-render means React executes the component again to calculate the next UI. It does not mean the DOM is rebuilt. React reconciles the new output with the previous one and commits only the necessary changes."**

**Chapter 66 — COMPLETE ✅**
